# Security Test Reference

## 常用命令

| 命令 | 参数 | 说明 | 示例 |
|------|------|------|------|
| `peri_secure` | `<index>` | PERI 防火墙：EL3 写入标记值，EL1 读回 | `peri_secure 0` |
| `hsperi_secure` | `<is_hsperi0> <index>` | HSPERI 防火墙：EL3/EL1 读对比 | `hsperi_secure 1 0` |
| `dram_secure_region` | `<0-7>` | DDR secure region | `dram_secure_region 0` |
| `dram_obfuscation` | `0` | DDR 混淆（仅 EL3） | `dram_obfuscation 0` |
| `rom_secure_region` | `<0-23>` | ROM secure region | `rom_secure_region 0` |
| `rom_read_lock` | `<0-23>` | ROM 读锁定 | `rom_read_lock 0` |
| `rom_define_region` | `0` | 用户自定义 ROM 区 | `rom_define_region 0` |
| `reset` | 无 | 系统复位 | `reset` |
| `current_el` | 无 | 查看当前异常级别 | `current_el` |
| `switch_el1` | 无 | EL3 跳入非安全 EL1 | `switch_el1` |
| `help` | 无 | 命令列表 | `help` |
| `wm` | `<addr> <data>` | 写内存/寄存器 | `wm 0x33030004 0x7FD` |
| `rm` | `<addr>` | 读内存/寄存器 | `rm 0x3303003C` |

### EL3 → EL1 约束

- `switch_el1` 跳入非安全 EL1 后，部分 IP 访问会导致系统挂死
- **每个用例独立一轮**：测试 → 解析 log → `reset` → 等待重启 → 重新进入 CLI
- 挂死后不要用 Ctrl+C，只能 `reset` 或硬件复位

---

## 防火墙通用配置（源自 `testcase_security.c`）

典型顺序（`peri_secure` / `hsperi_secure` / `dram_secure_region` 等 bmtest 命令内部亦遵循）：

1. 配置 **Master** `fabFW_hsperi_m_ar_ns`（偏移 `+0x04`，CA53 为 **bit1**）
2. 配置 **Slave secure** 位图（`reg_peri_fw_s_tz_s` / `reg_hsperi0_fw_tz_s` 等）
3. EL3 访问目标地址（写入标记值或读真值）
4. `switch_el1`
5. EL1 再次访问，检查是否被阻断

### Master `fabFW_hsperi_m_ar_ns` 位语义（以源码为准）

头文件 `reg_sec_fab_firewall.h` 中该字段复位值为 `h1`（各 bit 默认为 1）。

`testcase_security.c` 注释为 **Remove CA53 forced secure read**，写入：

```c
mmio_write_32(FAB_FIREWALL_BASE + SEC_FAB_FIREWALL_FABFW_HSPERI_M1_AR_NS, 0x000007FD);
```

即 **清除 bit1**（`0x7FF & ~0x2`），使 CA53（master1）读通道不再被强制为 secure。

| bit 值 | 含义（本芯片实现） |
|--------|-------------------|
| **1** | 强制 secure 读（master 以 secure 身份发起读） |
| **0** | 允许非安全读（配合 `switch_el1` 后由非安全 CPU 访问） |

> 注意：字段名虽带 `_ns`，实现上 **bit=1 表示强制 secure**，与字面相反；**以 `testcase_security.c` 注释和写值为准**。

### Slave `*_tz_s` 位语义

| bit 值 | 含义 |
|--------|------|
| **0** | 非 secure（默认） |
| **1** | secure 区域/外设 |

---

## 结果判定

固件 **不打印 PASS/FAIL**，Agent 根据 log 自行推断。

### 共同 log 字段

```
Before switch, CurrentEL=3
  peri_fw [0x........]=0x........    ← EL3 阶段外设/防火墙读值
After switch, CurrentEL=1
  peri_fw [0x........]=0x........    ← EL1 阶段读值
```

SRAM/DDR 测试还会打印：

```
  sec_fab [0x........]=0x........    ← 防火墙寄存器
  sram/dram [0x........]=0x........   ← 探测地址读值
```

---

### 写探测类：`peri_secure`

- EL3 向探测地址写 **`0x87654321`**
- `switch_el1` 后 EL1 读同一地址

| 结果 | 条件 |
|------|------|
| **PASS** | EL1 读值 **≠ `0x87654321`**（写未穿透） |
| **FAIL** | EL1 读值 **== `0x87654321`** |
| **PASS(BLOCKED)** | 命令超时无 prompt（访问挂死） |
| **PASS** | log 含 `INT: recv interrupt` |

辅助验证寄存器：`reg_peri_fw_s_tz_s`（Athena2 偏移见 [security_reg.md](security_reg.md) §4.2）。

---

### 读对比类：`hsperi_secure` / `rom_*` / `dram_secure_region`

- EL3 读取目标寄存器/内存得 **EL3读值**
- `switch_el1` 后 EL1 再读得 **EL1读值**

| 结果 | 条件 |
|------|------|
| **PASS** | EL3读值 **≠** EL1读值 |
| **FAIL** | EL3读值 **==** EL1读值（非安全读穿透） |
| **PASS(BLOCKED)** | 命令超时无 prompt |
| **PASS** | log 含 `INT: recv interrupt` |

存疑时（EL3 == EL1）：**不要立刻判 FAIL**，按下方 [辅助测试流程](#辅助测试流程el3el1-存疑) 复核；任一备用地址 EL3≠EL1、或非法访问日志非零 → 改判 PASS。

#### ROM 非安全访问重定向

ROM 有两个独立的保护机制，作用范围不同，**绝不能混淆**：

| 机制 | 寄存器 | 阻断范围 | EL3 能否读真值 | 测试命令 |
|------|--------|---------|---------------|----------|
| **tz_s (secure only)** | `reg_rom_fw_s_tz_s` (0x33030058) | 仅非安全访问 (EL1) | **能** — EL3 读真值，EL1 被重定向 | `rom_secure_region` |
| **psmsk (post-mask lock)** | `fabFW_ROM_psmsk` (0x3303005C) | **所有访问 (含 EL3)** | **不能** — EL3 和 EL1 均被重定向 | `rom_read_lock` |

当这两个机制的任一 bit 置位时，被阻断的读操作不会挂死或返回随机值，而是被硬件重定向到 ROM 首地址 `0x29400000`，该地址的第一个 word 值为 `0x14000042`。

**关键区别**：
- tz_s 下 EL3 可以读到 ROM 真实内容，EL1 读到 0x14000042（重定向）→ 读对比可判定 PASS
- psmsk 下 EL3 和 EL1 都读到 0x14000042（重定向）→ 读对比无法区分，需其他方式验证（如 do_irq 触发）
- psmsk 可能被 FSBL 预配置且硬件锁定，辅助验证前必须先 `rm 0x3303005C` 检查

辅助验证寄存器：

- HSPERI0：`reg_hsperi0_fw_tz_s`（`0x33030040`）
- HSPERI1：`reg_hsperi1_fw_tz_s`（`0x33030044`）
- ROM psmsk：`rom_fab_psmsk`（`0x3303005C`，**仅 rom_secure_region/rom_read_lock 使用**）
- 非法访问日志：`illegal_slave_space_access_info_l/h`（`0x3303004C` / `0x33030050`）
- 探测基址：见 [security_reg.md](security_reg.md) §4.3 / §4.4

### 套件→寄存器强制映射（辅助验证时必须查对应的寄存器）

**绝对禁止跨套件查寄存器**。每条测试命令对应唯一的 tz_s 寄存器：

| 测试命令 | 应查的 tz_s 寄存器 | 说明 |
|----------|-------------------|------|
| `hsperi_secure 1 <i>` | `0x33030040` (hsperi0) | hspei0 防火墙 |
| `hsperi_secure 0 <i>` | `0x33030044` (hsperi1) | hspei1 防火墙 |
| `peri_secure <i>` | `0x3303003C` (peri) | peri 防火墙 |
| `rom_secure_region <i>` | `0x3303005C` (ROM psmsk) | ROM 内置防火墙 |
| `rom_read_lock <i>` | `0x3303005C` (ROM psmsk) | ROM 读锁定 |
| `dram_secure_region <i>` | `0x33030048` (ddr tz_s) | DDR secure region |

> **常见错误**：对于 `hsperi_secure 0 0` (HSPERI_ROM)，Agent 可能会错误地查看 `0x3303005C` (ROM psmsk) 来判定。**这是错误的**——`hsperi_secure` 测试的是 hspei 防火墙，必须查对应的 `0x33030044` (hsperi1)，而非 ROM psmsk。ROM psmsk 仅用于 `rom_secure_region` / `rom_read_lock` 测试。

---

## 辅助测试流程（EL3==EL1 存疑）

正式命令跑完后，若读对比类 **EL3读值 == EL1读值**，或写探测类结果难以解释，Agent **在同一用例内** 追加辅助步骤（仍须 `reset` 后从 EL3 重来）。

### 何时触发

| 套件 | 触发条件 | 说明 |
|------|----------|------|
| `hsperi_secure` / `rom_*` / `dram_secure_region` | 正式 log 中 EL3读值 **==** EL1读值 | 可能是只读寄存器复位值相同（如均为 `0x0`），需换地址或查日志 |
| `peri_secure` | EL1读值 **==** `0x87654321` | 直接 **FAIL**，一般无需辅助 |
| `peri_secure` | EL1读值 ≠ 标记值，但与 EL3 读值相同且存疑 | 可手动复现写入标记值确认 EL3 是否写成功 |

### 辅助验证铁律（强制）

**辅助验证的每一步命令和输出必须来自设备实际执行，绝对禁止用"机制分析"或"同原理推断"替代实测。**

违反此铁律的典型错误模式：

| 错误模式 | 示例 | 为什么是错的 |
|----------|------|-------------|
| 同机制推断 | "与案例A相同机制，+4偏移验证可确认" | 不同案例的探测地址不同，ROM/O寄存器内容可能不同，必须实测 |
| 类比推理 | "参照 rom_secure_region 0 的结果，区域15也是 PASS" | rom_region 0 的 +4 偏移有可区分值(0x0)，区域15全为 0x14000042，情况完全不同 |
| 空壳结论 | 辅助计划中写了完整步骤序列，但从未连接设备执行 | 计划和执行是两回事，计划写得再详细也不能替代实际输出 |
| 批量推断 | "24个 rom_read_lock 案例机制一致" | 每个 index 对应不同 tz_s bit 和不同地址范围，批量分析不能替代逐条验证 |

**硬性规则**：

1. **每一步辅助验证命令必须连接设备实际发送并记录返回值**。`rm`/`wm` 的返回值必须是设备实际输出的 hex 值，不能是 "≠0x14000042"、"同上" 等推断描述
2. **无实测设备 log 的辅助结论一律标记为 `未验证`**，不得标 PASS、FAIL、BLOCKED 或 INCONCLUSIVE
3. **辅助验证计划文档 (`auxiliary_test_plan.md`) 中每条案例必须包含**：
   - 完整的原始设备输出（从 `reset`/`current_el` 到最后一个 `rm` 的逐行 log）
   - 每个 `rm` 的实际返回值（如 `value = 0x0`，不得写 `≠0x14000042`）
   - 实测完成后才能写最终结论
4. **禁止在辅助计划中写 "步骤 1: rm xxx → 期望值 ≠ yyy" 然后直接跳到"判定: PASS"**。必须先执行、拿到实际返回值、再根据实际值判定

**自检清单**（辅助验证完成前必须逐条确认）：

- [ ] 每个 `rm`/`wm` 命令都有设备返回的实际 hex 值
- [ ] 没有任何 "同上机制"、"与XX相同"、"可确认" 等推断性措辞
- [ ] EL3 和 EL1 的读值都是实际的完整 32-bit hex（如 `0x3f68c832`，不是 `≠0x14000042`）
- [ ] 结论是基于实际读值对比得出的，不是基于机制分析

### 地址来源（优先级）

1. **正式 log** 中 `peri_fw [0x........]` / 同类行里的地址（最准）
2. [security_reg.md](security_reg.md) §4.2（peri 探测地址表）
3. [security_reg.md](security_reg.md) §4.3（hsperi0 探测基址，从 memory map 解析）
4. 以上基址的 **备用偏移**（见下表）

### 备用探测偏移

在同一 IP 窗口内，按顺序最多试 **3 个** 备用地址（4 字节对齐）：

| 次序 | 偏移 | 示例（基址 `0x04190000`） |
|------|------|---------------------------|
| 主地址 | `+0x0` | `0x04190000` |
| 备用 1 | `+0x4` | `0x04190004` |
| 备用 2 | `+0x8` | `0x04190008` |
| 备用 3 | `+0x10` | `0x04190010` |

> 禁止跳出该 IP 的 64KB/窗口范围；若全部 EL3==EL1 再查配置与非法访问日志。

### 标准辅助序列（读对比类，以 `hsperi_secure` 为例）

每条辅助尝试前执行 **`reset`**，等待 `$ ` prompt，确认 `current_el` 为 3。

```
# --- 1. 查 index 对应寄存器与位（security_reg.md §4.3）---
# 例：hsperi_secure 1 0 → reg_hsperi0_fw_tz_s bit18，探测基址 0x04190000

# --- 2. 复现防火墙配置（EL3）---
current_el
rm 0x3303004C                    # 读非法日志基线（可选）
wm 0x33030004 0x7FD              # 清除 CA53 强制 secure 读
rm 0x33030040                    # 读当前 hsperi0 tz_s
wm 0x33030040 <old|(1<<bit)>     # 置位目标 slave secure；bit 未知时可写 (1<<bit) 覆盖
rm 0x33030040                    # 确认 tz_s 已置位

# --- 3. 【ROM案例强制】检查psmsk ---
rm 0x3303005C                    # 检查 fabFW_ROM_psmsk，任何bit非零都可能重定向EL3访问
# 若 psmsk 中对应被测试region的bit已置位：
#   → 尝试清除: wm 0x3303005C <清除对应bit后的值>
#   → 回读确认: rm 0x3303005C
#   → 若无法清除（回读值未变），psmsk被硬件锁定
#   → 结论: INCONCLUSIVE(PSMSK_LOCKED)，psmsk阻断EL3+EL1，tz_s效果无法验证

# --- 4. EL3 读探测地址 ---
rm <probe_addr>                  # 记录 EL3读值（必须是实际hex值）

# --- 5. 切 EL1 再读 ---
switch_el1
current_el                         # 应为 1
rm <probe_addr>                  # 记录 EL1读值（必须是实际hex值）

# --- 6. 收尾 ---
reset
```

若主地址 EL3==EL1，**reset 后**对备用地址 `+4`、`+8`、`+0x10` 各重复步骤 2–5（配置可只做一次，但每次 switch_el1 后须 reset 再测下一地址）。

### 写探测类辅助（`peri_secure`）

```
reset
wm 0x33030004 0x7FD
rm 0x3303003C
wm 0x3303003C <old|(1<<bit)>     # bit 见 security_reg.md §4.2
wm <probe_addr> 0x87654321       # EL3 写入标记值
rm <probe_addr>                  # 确认 EL3 写成功（应读到 0x87654321）
switch_el1
rm <probe_addr>                  # EL1 读回：≠0x87654321 → PASS
reset
```

### 辅助验证前置基线测试（强制）

**任何导致挂死的测试用例，在辅助验证时必须先做对照实验**：不配置任何防火墙（tz_s 全 0），EL3 直接访问探测地址。

```
reset → current_el=3
rm 0x33030004 0x7FD              # 仅清除强制secure读，不配置 tz_s
rm <probe_addr>                  # EL3直接读
rm <probe_addr+0x40>             # 多试几个偏移
```

| 基线结果 | 含义 | 后续操作 |
|----------|------|----------|
| EL3 可正常读写 | 硬件本身可访问，可以继续辅助验证防火墙 | 进入标准辅助序列 |
| **EL3 直接挂死** | 硬件级别保护，与安全防火墙无关 | → **INCONCLUSIVE(HW_LIMIT)** |

**为什么必须做基线测试**：
- 如果 EL3 不配置 tz_s 都能挂死，说明挂死是硬件特性，不是防火墙阻断
- EL3 是最高权限级别，只应该受硬件级保护（如 OTP 出厂锁定）影响
- 缺少基线测试会误判：将 "硬件不可访问" 错判为 "防火墙 PASE(BLOCKED)"

### 辅助验证地址空间约束（强制）

**绝对禁止**用 sec_fab 配置寄存器空间 (0x33030000) 的保护状态来推断外设地址空间的保护状态。两者是完全独立的地址范围：

| 地址空间 | 范围 | 保护机制 |
|----------|------|----------|
| sec_fab 配置寄存器 | 0x33030000 | 自身默认 secure-only，与 tz_s 配置无关 |
| 外设地址空间 | 0x2xxxxxxx | 由对应防火墙 tz_s 位控制 |

**典型错误**：EL1 读 sec_fab 寄存器触发 SYNC exception，就认为"防火墙阻断了外设访问"。sec_fab 寄存器默认就是 secure-only 的，即使不配置任何 tz_s 位，EL1 也无法访问。**这个异常不能证明任何外设被保护了。**

**正确做法**：当外设寄存器全部读回 0x0 无法对比时：
1. 在该外设地址空间内扫描更大偏移量（如 +0x40, +0x100 等），寻找硬件定义的非零寄存器（Capabilities ID、Version、Status 等只读寄存器通常在较高偏移）
2. 在该外设地址空间内找可写寄存器：EL3 写入已知值 → switch_el1 → EL1 读回，若被重定向到 0x14000042 则证明阻断
3. 以上均失败且无法判定时，标记为 **INCONCLUSIVE**，不要强行用无关地址空间的数据来判定

### 辅助判定（汇总）

| 条件 | 正式结果修正为 |
|------|----------------|
| 任一地址 EL3读值 **≠** EL1读值 | **PASS**（注明辅助地址和实际hex值） |
| `illegal_slave_space_access_info_l/h` 非零 | **PASS**（访问被记录阻断） |
| `fabFW_hsperi_m_ar_ns` bit1 仍为 1（未清） | **待确认** / 配置错误，先 `wm 0x33030004 0x7FD` 重测 |
| `tz_s` 对应 bit 未置 1 | **待确认** / 配置错误，修正后重测 |
| 主地址 + 3 个备用地址均 EL3==EL1，且配置正确 | **FAIL** |
| 上述 FAIL 但原因是测试框架限制（如防火墙对目标地址无管辖权） | **FAIL(TL)** — 非硬件缺陷，备注说明框架限制 |
| `switch_el1` 后无 prompt（挂死） | **PASS(BLOCKED)** |
| 挂死但基线测试 EL3 无 tz_s 直接读亦挂死 | **INCONCLUSIVE(HW_LIMIT)** — 非防火墙问题 |
| ROM案例：psmsk对应bit已置位且无法清除 | **INCONCLUSIVE(PSMSK_LOCKED)** — psmsk阻断EL3+EL1，tz_s效果被掩盖 |
| **辅助结论无设备实测 log** | **未验证** — 禁止标 PASS/FAIL/BLOCKED/INCONCLUSIVE，必须先实测 |

### 报告要求

在 [report_template.md](../templates/report_template.md)「辅助验证记录」中填写：正式命令、尝试地址列表、各地址 EL3/EL1 读值、`tz_s`/`ar_ns`/illegal_slave 读值、最终结论。

---

### 混淆验证：`dram_obfuscation`

- 仅 EL3，不 `switch_el1`
- 混淆 **OFF**：读值 == 明文 `0x76543210`
- 混淆 **ON**：读值 ≠ 明文

---

## 参考文档

| 名称 | 说明 | 路径 |
|------|------|------|
| bmtest 参考源码 | SRAM/DDR 辅助测试范例（非完整 peri/hsperi 实现） | [testcase_security.c](testcase_security.c) |
| 寄存器头文件 | 符号名与 bit 偏移（**偏移以 Athena2 xls 为准**） | [reg_sec_fab_firewall.h](reg_sec_fab_firewall.h) |
| 寄存器映射 | 绝对地址、bmtest index→bit | [security_reg.md](security_reg.md) |
| 测试用例 | 官方表格总结 | [security_testcase.md](security_testcase.md) |
| 批量命令 | Agent 执行清单 | [test_cases.md](../test_cases.md) |

> `references/testcase_security.c` 仅含 `test_ns_access_secure_sram` / `dram` 范例；**`peri_secure` / `hsperi_secure` 的完整实现以固件 bmtest 为准**，寄存器配置规律与范例一致。

---

## 故障排查

**遇到挂死、读值异常、EL3==EL1 存疑等情况时，必须调用 `superpowers:systematic-debugging` 系统性排查根因，禁止盲目重试。**

| 现象 | 处理 |
|------|------|
| 串口无响应 | 按 uart_skill 检查 SSH 2222 桥接或 COM 口 |
| reset 后未回到 CLI | 等待更长时间，确认固件自动启动 |
| 系统挂死 | 调用 `superpowers:systematic-debugging` 排查：EL状态→配置寄存器→硬件复位；记 PASS(BLOCKED) 或 PENDING |
| EL3==EL1 存疑 | 调用 `superpowers:systematic-debugging` 后按辅助测试流程：换 `+4/+8/+0x10` 地址重测，查 illegal_slave 日志 |
| 编译失败 | 确认交叉编译工具链和 BOARD=ASIC 环境 |
