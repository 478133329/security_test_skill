# Security 测试常见问题与解决方法

当测试结果异常时，先在此文档中匹配现象，按对应流程排查，避免盲目重试。

---

## 1. 现象速查表

| 现象 | 常见原因 | 结论倾向 | 详见 |
|------|---------|---------|------|
| EL3读值==EL1读值 | 只读寄存器复位值相同 / ar_ns未清除 / tz_s未置位 / clock未开 | 需辅助验证，勿直接判FAIL | §2.1 |
| 命令执行后挂死 | 非安全访问触发硬件block / clock未开导致总线hang | PASS(BLOCKED) | §2.2 |
| INT: recv interrupt | 防火墙以中断方式阻断非安全访问 | PASS | §2.3 |
| EL1读到0x14000042 | 防火墙重定向到保留地址（ROM/peri/hspei均可能） | PASS（需区分来源确认是阻断） | §2.4 |
| reset后current_el!=3 | 上一轮挂死导致reset未正常执行 / EL1残留 | 再次reset或MCU reboot | §2.5 |
| 读值与预期模式不一致 | clock未开 / pinmux未配 / 地址错误 / 寄存器只读 | 逐项排查环境配置 | §2.6 |

---

## 2. 各现象详细分析

### 2.1 EL3读值==EL1读值

**可能原因**：

1. **只读寄存器，复位值相同**：某些IP寄存器为只读状态寄存器，EL3和EL1读到的都是硬件复位值（如0x0），无法体现防火墙差异。此时需换备用地址（+0x4/+0x8/+0x10）找可读写的寄存器。
2. **ar_ns未清除**：CA53(master1)的`fabFW_hsperi_m_ar_ns` bit1仍为1（强制secure读），EL1发出的读被当作secure，穿透防火墙。检查`rm 0x33030004`，确认bit1=0。
3. **tz_s对应bit未置位**：目标外设的secure位未设为1，防火墙未生效。检查对应tz_s寄存器（peri→0x3303003C, hspei0→0x33030040, hspei1→0x33030044, rom→0x33030058/5C）。
4. **ROM psmsk 已被系统预配置**（仅 ROM 案例）：`fabFW_ROM_psmsk` (0x3303005C) 可能在 FSBL 阶段就被置位，此寄存器阻断**所有访问（含EL3）**，读操作被重定向到 ROM 基址 0x29400000 返回 0x14000042。**EL3 也读到 0x14000042 时，必须先查 psmsk**。详见 [reference.md §辅助验证前置基线测试]。
5. **Clock未开**：IP的时钟门控未使能，EL3和EL1读到的可能都是总线默认返回值（如0x0或随机值），看起来相同。检查clkgen中对应IP的时钟位（见§3.1）。

**排查顺序**：

```
1. rm 0x33030004 → 确认bit1==0（清除CA53强制secure读）
2. rm <tz_s寄存器> → 确认目标bit已置1
3. 【ROM案例专属】rm 0x3303005C → 检查psmsk是否有bit被预置位（非零即被配置）
   psmsk的bit=1会阻断EL3+EL1的访问 → 若psmsk非零，先wm 0x3303005C 0x0清除
   → 若无法清除（回读仍为非零），则psmsk被硬件锁定，此区域不可用于辅助验证
4. rm <探测地址+0x4/+0x8/+0x10> → 换地址对比
5. rm 0x3303004C / 0x33030050 → 查非法访问日志
6. 检查IP clock是否使能（见§3.1）
7. 全部EL3==EL1且配置正确 → FAIL
```

> 对应的 tz_s 寄存器必须严格按 [reference.md](reference.md)「套件→寄存器强制映射」选择，**禁止跨套件查寄存器**。

### 2.2 命令执行后挂死

**可能原因**：

1. **硬件访问阻断**：非安全(EL1)访问安全区域时总线无响应，CPU挂死。
2. **Clock未开**：IP时钟关闭时，总线访问该IP会无限等待ACK，导致挂死。
3. **IP本身不支持非安全访问**：某些IP在硬件设计上对非安全访问无响应。

**判定**：

| 挂死时机 | 判定 | 说明 |
|---------|------|------|
| switch_el1后立即挂死 | PASS(BLOCKED) | 防火墙成功阻断 |
| EL3阶段就挂死 | 异常，需排查 | 可能是clock未开或地址错误 |
| EL1读特定地址挂死 | PASS(BLOCKED) | 该地址被防火墙保护 |

**恢复**：在 soc_session 发送 `reset` → 等待 `$` prompt → `current_el` 确认==3。无响应则MCU reboot。

### 2.3 INT: recv interrupt

**可能原因**：防火墙配置为中断模式阻断非安全访问，而非挂死模式。不同IP的防火墙响应机制不同（挂死 vs 中断），取决于硬件实现。

**判定**：**PASS** — log中出现 `INT: recv interrupt` 即表示防火墙检测到非法访问并以中断方式上报。

### 2.4 EL1读到0x14000042

**可能原因**：该值来自总线保留地址的默认返回值，当非安全访问被防火墙拦截后：
- **ROM防火墙**：硬件重定向到ROM首地址0x29400000，其第一个word为0x14000042
- **PERI/HSPERI防火墙**：访问被拒绝后总线返回默认值0x14000042

**判定**：**PASS** — 只要EL3读值≠0x14000042而EL1读到0x14000042，就是防火墙正常阻断。

### 2.5 reset后current_el!=3

**可能原因**：上一轮测试挂死时reset命令未被处理，系统仍残留在EL1状态。

**处理**：
```
1. 再次执行 reset → 等待reboot → 检查current_el
2. 仍!=3 → MCU reboot → 等待 "athena2 bmtest start" → 等待2s → 按任意键
3. 仍!=3 → 请用户手动断电重启
```

### 2.6 读值与预期模式不一致

**可能原因（按概率排序）**：

1. **Clock未开**（见§3.1）
2. **Pinmux未配**（见§3.2）
3. **寄存器地址错误**（见§3.3）
4. **只读寄存器**，读到的就是真实值且EL3/EL1无差异
5. **ar_ns/tz_s配置错误**（见§4）

---

## 3. 环境排查清单

异常用例在辅助验证之前，先完成以下三项环境检查。

### 3.1 Clock 配置

IP的时钟门控未使能会导致：寄存器访问挂死、返回随机值、EL3/EL1读值相同。

**检查方法**：

```
# clkgen基址 0x27020000（PERI APB clkgen），具体偏移需参考时钟树文档
# 确认目标IP的时钟bit已置1（使能）

rm 0x27020000    # 读clkgen控制寄存器，确认对应bit
```

**常见需要检查clk的IP**：

| 分类 | IP | 备注 |
|------|-----|------|
| hsperi0 | I2C0-9, SPI0-1, UART0-7, I2S0-5, ETH0-1 | 所有hsperi外设依赖独立时钟门控 |
| hsperi1 | CAN0-1, SD0-2, EMMC, SPI2-3, SDMA0-1 | 同上 |
| peri | PWM0-4, WDT0-2, Timer, KEYSCAN, SARADC, TEMPSEN | 部分依赖PERI公用时钟 |

> 若时钟未开，EL3访问也可能异常（挂死或返回值不合理），此时测试结果不可信。

### 3.2 Pinmux 配置

带外部引脚的外设，若pinmux未配或配置错误，IP虽能被访问，但功能状态可能异常。

**检查方法**：

```
# pinmux基址 0x27001000（PERI pinmux），具体偏移需参考pinmux文档
# 确认目标IP的引脚功能已选择

rm 0x27001000    # 读pinmux寄存器，确认对应引脚功能位
```

**需检查pinmux的IP**：

| IP | 引脚类型 | 备注 |
|-----|---------|------|
| UART0-7 | TX/RX/RTS/CTS | UART0常用于调试，可能被复用 |
| I2C0-9 | SCL/SDA | 多个I2C共享引脚，需确认选择 |
| SPI0-3, SPI_NAND | CLK/MOSI/MISO/CS | |
| SD0-2, EMMC | CLK/CMD/DATA | |
| ETH0-1 | RGMII/MDC/MDIO | |
| I2S0-5 | BCLK/LRCK/DATA | |
| CAN0-1 | TX/RX | |
| PWM0-4 | 输出引脚 | |
| KEYSCAN | 行列引脚 | |
| GPIO0-5 | 对应GPIO引脚 | |

> pinmux未配一般不会导致防火墙测试结果反转，但若EL3/EL1读到的寄存器值与预期差异很大，应检查pinmux排除硬件状态异常。

### 3.3 寄存器地址确认

地址错误会导致读到其他IP的寄存器或无效地址空间。

**检查方法**：

1. 以 `test_cases.md` 中「内部步骤」和 [security_reg.md](security_reg.md) §4 的探测地址表为基准
2. 正式log中的 `peri_fw [0x........]` 地址优先级最高
3. 辅助验证换备用偏移时，不得跳出该IP的64KB窗口
4. 写探测类地址在security_reg.md §4.2，读对比类地址在§4.3/§4.4

**常见错误**：

| 错误 | 纠正 |
|------|------|
| `hsperi_secure 0 0` (HSPERI_ROM) 去查 0x3303005C (ROM psmsk) | 应查 0x33030044 (hsperi1 tz_s)，ROM psmsk仅用于 rom_secure_region/rom_read_lock |
| peri_secure 用 hsperi 的 tz_s 寄存器 | 应查 0x3303003C (peri tz_s) |
| 备用偏移跳到其他IP窗口 | 保持在同一个64KB窗口内（+0x4/+0x8/+0x10） |

---

## 4. 配置排查

### ar_ns未清除

**现象**：EL3读值==EL1读值，tz_s配置正确但防火墙似乎未生效。

**原因**：`fabFW_hsperi_m_ar_ns` (0x33030004) bit1 仍为1（复位默认值），CA53的读被强制为secure，EL1下的读穿透了防火墙。

**检查**：`rm 0x33030004`，预期bit1==0。若不是，`wm 0x33030004 0x7FD` 后重测。

### tz_s对应bit未置位

**现象**：EL3读值==EL1读值，ar_ns配置正确。

**原因**：目标IP的secure bit未设为1，防火墙对该IP未生效。

**检查**：`rm <tz_s寄存器>`，确认目标bit==1。对应关系见下表：

| 套件 | tz_s寄存器 | 示例 |
|------|-----------|------|
| peri | 0x3303003C | peri_secure 6 → SARADC bit28 |
| hsperi0 | 0x33030040 | hspei_secure 1 0 → SPI1 bit18 |
| hsperi1 | 0x33030044 | hspei_secure 0 0 → ROM bit26 |
| rom (secure region) | 0x33030058 | rom_secure_region 0 → bit0 |
| rom (read lock) | 0x3303005C | rom_read_lock 0 → bit0 |

### 跨套件查错寄存器

**现象**：辅助验证时配置了正确的tz_s但问题依旧，因为查的根本不是当前套件的寄存器。

**最容易出错的组合**：

| 命令 | 错误做法 | 正确做法 |
|------|---------|---------|
| `hsperi_secure 0 0` (HSPERI_ROM) | 查 0x3303005C (ROM psmsk) | 查 0x33030044 (hsperi1) |
| `hsperi_secure 1 21` (SPI_NAND) | 查 0x33030044 (hsperi1) | 查 0x33030040 (hsperi0) |
| `peri_secure 6` (SARADC) | 查 0x33030040 (hsperi0) | 查 0x3303003C (peri) |

> 口诀：`hsperi_secure` 第一个参数=1查hsperi0(0x40)，=0查hsperi1(0x44)；`peri_secure` 不管参数只查peri(0x3C)；`rom_*` 查ROM(0x58/0x5C)。

---

## 5. 历史问题案例

记录历史测试中遇到的典型案例，供后续参考。

### 案例1: HSPERI_SD2_CFG (hsperi_secure 0 3) EL3==EL1==0x0

- **日期**: 2026-07-01
- **现象**: EL3和EL1均读到0x0，辅助验证换+0x4/+0x8/+0x10也全部相同
- **排查**: ar_ns/tz_s配置正确，illegal_slave日志为零
- **结论**: FAIL（SD2控制器寄存器可能为只读状态寄存器，复位值就是0x0）
- **待验证**: 是否因clock未开导致

### 案例2: HSPERI_ROM (hsperi_secure 0 0) EL3==EL1

- **日期**: 2026-07-01
- **现象**: EL3读值==EL1读值，但ROM本身有独立的防火墙管控
- **排查**: 确认是hsperi1防火墙测试，ROM被hspei1防火墙覆盖，需辅助验证
- **结论**: PASS（辅助验证确认hsperi1防火墙生效）

### 案例3: ROM_DEFINE_REGION (rom_define_region 0) BLOCKED

- **日期**: 2026-07-01
- **现象**: 命令执行后触发interrupt无限循环，无法正常完成
- **排查**: 用户自定义ROM区的非安全访问触发了硬件中断循环
- **结论**: PASS(BLOCKED)
