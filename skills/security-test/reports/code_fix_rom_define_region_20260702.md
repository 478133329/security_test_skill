# bmtest 代码修改: rom_define_region 中断死循环修复

- **发现时间**: 2026-07-02
- **关联问题**: `rom_define_region 0` 无限中断循环 (IRQ 63)
- **当前状态**: 待人工审阅

---

## 1. 问题现象

```
$ rom_define_region 0
INT: recv interrupt
  ddr_fab [0x33040090]=0x0
  ddr_fab [0x33040094]=0x0
INT: recv interrupt
  ddr_fab [0x33040090]=0x0
  ddr_fab [0x33040094]=0x0
[...] ← 无限循环，Ctrl+C无效，需MCU reboot恢复
```

## 2. 根因分析

### 死循环机制

```
EL1 执行 mmio_read_32(0x29400800)
  → ROM地址空间被标记为secure region
  → EL1非安全读触发防火墙 → IRQ 63
  → irq_handler 执行:
      1. uartlog("INT: recv interrupt\n")     // 打印中断消息
      2. mmio_setbits_32(0x33030000, 0x3)      // 清除ROM防火墙IRQ状态
      3. printf ddr_fab illegal_access_info    // 读非法访问日志
      4. mmio_setbits_32(0x33040004, 1)        // 清除DDR IRQ状态
      5. return 0                               // ← 返回被中断的指令处！
  → CPU从中断返回，重新执行同一条 EL1 read 指令
  → 再次触发防火墙 → IRQ 63 → 无限循环
```

### 关键代码位置

**`irq_handler` (testcase_security.c:37-52)**:

```c
static int irq_handler(int irqn, void *priv)
{
    mdelay(2);
    uartlog("INT: recv interrupt\n");
    mmio_setbits_32(0x33030000, 0x3);          // 清除IRQ状态
    printf("  ddr_fab [0x33040090]=0x%x\n", mmio_read_32(0x33040090));
    printf("  ddr_fab [0x33040094]=0x%x\n", mmio_read_32(0x33040094));
    mmio_setbits_32(0x33040004, 1);             // 清除DDR IRQ
    return 0;                                   // ← BUG: 未跳过故障指令
}
```

**`rom_firewall_irq_handler` (testcase_security.c:1104-1112)**:

```c
static int rom_firewall_irq_handler(int irqn, void *priv)
{
    uartlog("INT: recv interrupt\n");
    mmio_setbits_32(0x33030000, (0x1 << 3));    // 清除ROM IRQ
    return 0;                                    // ← 同样问题：未跳过故障指令
}
```

两个处理函数都只清除了硬件IRQ标志位，但 **没有推进 ELR_EL1（Exception Link Register）跳过触发异常的指令**。当 CPU 从中断返回 (`eret`) 时，ELR_EL1 仍然指向那条 EL1 读指令，于是重新执行 → 再触发 → 死循环。

## 3. 修改方案

### 原理

在中断处理函数中，读取 ESR_EL1 (Exception Syndrome Register) 确认是 Data Abort 异常后，将 ELR_EL1 增加 4 (AArch64 指令均为 4 字节)，使 CPU 从中断返回到 **下一条指令** 而非重新执行触发异常的指令。

```c
// 读取ESR_EL1, 确认EC (Exception Class)
uint64_t esr_el1;
asm volatile("mrs %0, esr_el1" : "=r"(esr_el1));
uint32_t ec = (esr_el1 >> 26) & 0x3f;

// EC=0x24 (DABORT_LOWER_EL) 或 EC=0x25 (DABORT_CUR_EL) 时跳过故障指令
if (ec == 0x24 || ec == 0x25) {
    uint64_t elr_el1;
    asm volatile("mrs %0, elr_el1" : "=r"(elr_el1));
    elr_el1 += 4;
    asm volatile("msr elr_el1, %0" : : "r"(elr_el1));
}
```

### 具体改动

#### 文件: `cvi_bmtest/athena2/test/security/testcase_security.c`

**改动 1: `irq_handler` (line 37-52) — 在 `return 0` 前添加 ELR 推进逻辑**

```c
static int irq_handler(int irqn, void *priv)
{
    uint64_t esr_el1, elr_el1;
    uint32_t ec;

    mdelay(2);
    uartlog("INT: recv interrupt\n");

    mmio_setbits_32(0x33030000, 0x3);

    printf("  ddr_fab [0x33040090]=0x%x\n", mmio_read_32(0x33040090));
    printf("  ddr_fab [0x33040094]=0x%x\n", mmio_read_32(0x33040094));

    mmio_setbits_32(0x33040004, 1);

    /* Skip the faulting instruction to avoid infinite IRQ loop.
     * Firewall violation triggers Data Abort; advance ELR_EL1
     * past the faulting load/store so we don't re-execute it. */
    asm volatile("mrs %0, esr_el1" : "=r"(esr_el1));
    ec = (esr_el1 >> 26) & 0x3f;
    if (ec == 0x24 || ec == 0x25) {  // DABORT_LOWER_EL or DABORT_CUR_EL
        asm volatile("mrs %0, elr_el1" : "=r"(elr_el1));
        elr_el1 += 4;
        asm volatile("msr elr_el1, %0" : : "r"(elr_el1));
    }

    return 0;
}
```

**改动 2: `rom_firewall_irq_handler` (line 1104-1112) — 同样添加**

```c
static int rom_firewall_irq_handler(int irqn, void *priv)
{
    uint64_t esr_el1, elr_el1;
    uint32_t ec;

    uartlog("INT: recv interrupt\n");

    // clear rom firewall irq
    mmio_setbits_32(0x33030000, (0x1 << 3));

    /* Skip the faulting instruction to avoid infinite IRQ loop */
    asm volatile("mrs %0, esr_el1" : "=r"(esr_el1));
    ec = (esr_el1 >> 26) & 0x3f;
    if (ec == 0x24 || ec == 0x25) {
        asm volatile("mrs %0, elr_el1" : "=r"(elr_el1));
        elr_el1 += 4;
        asm volatile("msr elr_el1, %0" : : "r"(elr_el1));
    }

    return 0;
}
```

## 4. 影响范围

| 影响对象 | 说明 |
|----------|------|
| `irq_handler` (IRQ 63, IRQ 138) | DDR firewall 和 SRAM firewall 测试的中断处理，当前依赖 "中断触发即 PASS"。修改后中断仍触发，但不会死循环。 |
| `rom_firewall_irq_handler` (IRQ 139) | ROM firewall 测试中断处理，`rom_read_lock` 等测试使用。 |
| `rom_define_region 0` | 修改后应按预期：打印 INT 消息 + 非法访问日志后继续执行后续代码，而非死循环。 |
| 其他安全测试命令 | 不受影响 — 仅修改中断处理函数的返回路径。 |

## 5. 预期效果

修改后 `rom_define_region 0` 执行流程：

```
EL3: 配置 postmsk_ctrl, tz_s bit24
EL3: 读 0x29400800 → 正常
↓ switch_el3_to_el1
EL1: 读 0x29400804 → 触发防火墙 → IRQ 63
  → irq_handler: 打印 INT + ddr_fab，跳过故障指令
  → CPU 返回执行下一条指令（不再反复触发）
EL1: 后续代码继续执行或正常结束
```

中断仍然会触发（证明防火墙工作），但不再死循环。

## 6. 回退方案

```bash
cd cvi_bmtest/athena2
git checkout -- test/security/testcase_security.c
```

重新编译烧录即可恢复原始状态。
