---
name: security-test
description: >-
  Runs Athena2 security firewall tests one-by-one by sending bmtest CLI commands
  over UART (_secure suites plus primitive commands rm/wm/switch_el1/current_el
  for flexible auxiliary verification). Resets after each case. Infers PASS/FAIL
  from register reads. Use for security test, firewall testing, or flexible
  memory/EL verification on bmtest.
---

# Athena2 Security 测试

| 对象 | 说明 |
|------|------|
| 测试环境 | 裸机环境，只能使用 bmtest 中的命令 |
| bmtest | 封装 `peri_secure` / `hsperi_secure` 等命令，内部完成寄存器配置 + EL 切换 |
| 辅助测试 | `rm`/`wm`/`switch_el1`/`current_el`/`reset`；配置规律见 [references/testcase_security.c](references/testcase_security.c) |

## 关联文档

| 文档 | 用途 |
|------|------|
| [uart_skill.md](uart_skill.md) | 串口连接与烧录 |
| [bmtest_build.md](bmtest_build.md) | bmtest 代码修改与编译 |
| [reference.md](references/reference.md) | **判定规则**、log 字段、寄存器位语义 |
| [troubleshooting.md](references/troubleshooting.md) | **常见问题与解决方法**：异常现象→原因→排查步骤→结论 |
| [security_reg.md](references/security_reg.md) | 寄存器绝对地址、index→bit 映射 |
| [security_testcase.md](references/security_testcase.md) | 官方验证表格用例总结 |
| [test_cases.md](test_cases.md) | 批量测试命令清单 |
| [report_template.md](reports/report_template.md) | 报告模板（测试报告 / 辅助测试报告 / 代码修改记录） |
| [reports/auxiliary_test_plan.md](reports/auxiliary_test_plan.md) | **辅助验证计划**（EL3==EL1/挂死等存疑用例的逐步骤复验方案） |

## 测试套件（bmtest）

| 套件 | 命令 | 数量 | 判定类型 |
|------|------|------|----------|
| peri | `peri_secure <i>` | 28 | **写探测** |
| hsperi | `hsperi_secure <g> <i>` | 42 可测 | **读对比** |
| dram_region | `dram_secure_region <0-7>` | 8 | 读对比 |
| dram_obf | `dram_obfuscation 0` | 1 | 混淆验证 |
| rom_region | `rom_secure_region <0-23>` | 24 | 读对比 |
| rom_lock | `rom_read_lock <0-23>` | 24 | 读对比 |
| rom_define | `rom_define_region 0` | 1 | 读对比 |

**推荐使用 bmtest_automation 脚本自动执行。** 脚本通过 pexpect + sshpass 管理交互式 SSH 会话，自动处理 prompt 等待、挂死检测和 MCU 恢复。

> 运行方式：`python3 -m security-test.bmtest_automation.main` (从 skills 目录)
>
> 脚本特性：
> - 逐条发送命令，等待 `$` prompt 后才发下一条
> - 15s 超时检测挂死，自动通过 MCU 2223 reboot 恢复
> - 每条用例后自动 reset + 等待 reboot + EL3 验证
> - 每 5 条保存一次进度到 `results_progress.json`
> - 支持断点续跑（通过 `--start-from` 参数）
>
> 详见 [bmtest_automation/](bmtest_automation/) 目录下各模块源码

## Superpowers 技能对接

安全测试的四个关键环节对接 Superpowers 技能，确保流程规范、可追溯、可验证：

| 环节 | Superpowers 技能 | 说明 |
|------|-----------------|------|
| **测试计划** | `superpowers:writing-plans` | 正式开始前生成测试计划文档，包含用例顺序和预期测试结果，**所有用例全部执行，不设 SKIP** |
| **测试过程** | `superpowers:executing-plans` | 严格按照计划逐条执行，每条用例单步交互，不跳过不批量 |
| **错误/异常** | `superpowers:systematic-debugging` | 挂死、读值异常、EL3==EL1 存疑时触发，系统性排查根因而非盲目重试 |
| **测试报告** | `superpowers:verification-before-completion` | 报告生成后验证：所有值是否实际 hex、备注是否完整、log 是否齐全、判定是否正确 |

### 0. 测试计划（superpowers:writing-plans）

在连接串口、开始测试之前，Agent 必须先调用 `superpowers:writing-plans` 生成测试计划文档 `reports/test_plan.md`：

1. 解析 [test_cases.md](test_cases.md) 中所有用例
2. 为每条用例设定预期测试结果（描述具体预期现象而非 PASS/FAIL，参考 [reference.md](references/reference.md) 判定规则）：
   - 写探测类：`EL1读值≠0x87654321`
   - 读对比类：`EL3读值≠EL1读值`
   - DDR 类：`EL1读值≠EL3明文，或INT触发`
   - 混淆类：`混淆OFF读==0x76543210，混淆ON读≠明文`
   - ROM 类：`EL1读值=0x14000042（重定向），EL3读值≠EL1读值`
   - 预期挂死或中断的IP：`EL1访问挂死` 或 `INT: recv interrupt`
3. 确定执行顺序（按 Index 递增）
4. 预估可能存疑的用例（如已知某些 IP 易挂死或读值特殊）

## Agent 执行步骤

Agent 通过串口向 bmtest **直接发送 CLI 命令**，通信方式见 [uart_skill.md](uart_skill.md)。

**固件 log 不会输出 PASS/FAIL**，判定规则见 [reference.md](references/reference.md)。

### 1. 硬约束：单步交互 (one-step-at-a-time)

**每条测试用例必须完整执行以下循环后，才能开始下一条：**

```
发送命令 → 等待 $ prompt → 收集 log → 判定 → reset → 等待 reboot → 等待 "athena2 bmtest start" → 等待2s → 按任意键进入CLI → 等待 $ prompt → current_el 确认为3 → 下一条
```

以下行为**绝对禁止**，出现即视为违规：

| 禁止行为 | 为什么不行 |
|----------|-----------|
| 用 `sshpass`/`expect`/bash 管道一次发送多条命令 | 无法感知 prompt 是否就绪，时间不可控 |
| 用 bash `for` 循环 + `sleep` 批量发送 | reboot 时长不固定，`sleep` 不可靠 |
| 把命令放到 background task 执行 | 失去对每步输出的实时判定能力 |
| 在 `reset` 后不等待 `$` 就发送下一条命令 | 串口输入首字符会被 "Input any key" 吞掉导致命令损坏 |
| 因为"测试结果看起来一致"就跳过中间用例 | 每条用例对应不同的硬件 IP，必须全部覆盖 |
| **不检查 current_el 就直接发送测试命令** | 上一条用例可能残留 EL1 状态，在 EL1 下执行会导致异常循环挂死 |

**正确做法**：Agent 使用交互式终端工具，一条一条发送命令，每次等待 `$` 出现后再发下一条。速度慢是正常的，安全测试就是这样。

### 2. 连接串口

Agent 需要创建**一个交互式终端会话**（SoC 主通道），MCU 备用通道使用 SSH exec 模式：

| 会话 | 连接方式 | 用途 |
|------|----------|------|
| `soc_session` | MCP 交互终端 `ssh -p 2222 admin@172.25.84.190` | **主通道**，bmtest CLI，逐条执行安全测试命令 |
| MCU 备用通道 | SSH exec：`sshpass -p 123456 ssh -T -o StrictHostKeyChecking=no -p 2223 admin@172.25.84.190 <命令>` | **备用通道**，SoC 挂死时发送 `reboot` 重启 SoC |

正常测试流程中只使用 `soc_session`，MCU 通道仅在需要重启 SoC 时通过 SSH exec 发送命令。架构详见 [uart_skill.md](uart_skill.md)。

**开机或reset后，log输出`athena2 bmtest start`后，请等待2s在进行命令行操作，避免串口乱码**

**重要**：[uart_skill.md](uart_skill.md) 中描述的 `uart-upgrade` / `scp fip.bin` 流程**仅用于首次固件部署**（如刚拿到板子需要烧录 bmtest 固件）。**安全测试过程中绝对不要触发烧录**——系统卡死时请按「挂死恢复」流程处理，不要尝试重烧固件。

### 3. 测试循环（superpowers:executing-plans）

严格按照 [Superpowers 测试计划](#0-测试计划superpowerswriting-plans) 生成的 `reports/test_plan.md` 逐条执行：

```
按计划取下一用例 → 发送 bmtest 命令 → 收集 log → 判定 → reset → 下一条
```

**每条用例后必须 `reset`**，否则 EL 切换后可能挂死且无法恢复。不要用 Ctrl+C。

**reset 后必须验证 EL 状态**，否则上一条用例的 EL1 残留会导致下一条命令在 EL1 执行而异常挂死：

```
# 每条用例开始前：
1. 确认已看到 $ prompt
2. 执行 current_el
3. 若 current_el != 3 → 再次 reset，等待 reboot，重新检查
4. 若 current_el == 3 → 继续执行测试命令
```

**为什么需要这个检查**：`peri_secure` / `hsperi_secure` 等命令内部会 `switch_el1`，测试完成后虽然执行了 `reset`，但如果在 reset 前系统已挂死（命令超时/异常），reset 可能未被处理，导致下次开机仍处于 EL1 残留状态。

### 4. 结果判定（摘要；存疑时使用 superpowers:systematic-debugging）

完整规则见 [reference.md](references/reference.md)。

**当遇到以下情况时，必须调用 `superpowers:systematic-debugging` 进行系统性排查，而非盲目重试或直接判 FAIL**：

| 触发条件 | 说明 |
|----------|------|
| 命令执行后挂死/无限循环 | 排查挂死原因：EL 残留、IP 访问阻断、配置错误 |
| EL3读值 == EL1读值 | 读对比类存疑，需查防火墙配置是否正确、换地址验证 |
| 读值与预期模式不一致 | 如 peri_secure 的 EL1 不是 0x14000042，或 dram 未触发 INT |
| reset 后 current_el != 3 持续多轮 | 排查是否为硬件异常 |
| 任何无法直接判定 PASS/FAIL 的情况 | 需要系统性分析的边界情况 |

#### 写探测类：`peri_secure`

```
EL3 写入标记值 0x87654321 → switch_el1 → EL1 读回
EL1读值 ≠ 0x87654321  →  PASS（写未穿透）
EL1读值 == 0x87654321  →  FAIL
EL1访问后挂死 / INT: recv interrupt  →  PASS(BLOCKED) / PASS
```

> **重要**：挂死后必须先做基线测试（见 [reference.md](references/reference.md)「辅助验证前置基线测试」）：reset 后不配置 tz_s，EL3 直接读探测地址。EL3 挂死 → **INCONCLUSIVE(HW_LIMIT)**，非防火墙问题。

#### 读对比类：`hsperi_secure` / `rom_*` / `dram_secure_region`

```
EL3读值 ≠ EL1读值  →  PASS
EL3读值 == EL1读值  →  触发辅助测试（勿直接 FAIL）→ 见 reference.md「辅助测试流程」
EL1访问后挂死  →  先做基线测试（EL3无配置直接读），EL3挂死→INCONCLUSIVE，仅EL1挂死→PASS(BLOCKED)
INT: recv interrupt  →  PASS
```

#### 辅助测试（EL3==EL1 时）

**铁律**: 辅助验证的每一步必须连接设备实际执行并记录返回值，禁止用机制分析替代实测。无实测 log 的结论一律标记为 **未验证**。详见 [reference.md](references/reference.md)「辅助验证铁律（强制）」。

**第一步必须确认正确的 tz_s 寄存器**（见 [reference.md](references/reference.md)「套件→寄存器强制映射」），**严禁跨套件查错寄存器**。

正式用例 `reset` 后，按 [reference.md](references/reference.md) 辅助测试流程：

1. 从 log 或 [security_reg.md](references/security_reg.md) §4 取探测基址
2. **确认 tz_s 寄存器**：`hsperi_secure 1`→`0x33030040`, `hsperi_secure 0`→`0x33030044`, `peri_secure`→`0x3303003C`, `rom_*`→`0x3303005C`
3. `wm 0x33030004 0x7FD` + 置位对应 `tz_s` → `rm` 主地址 → `switch_el1` → `rm` 对比
4. 仍相同则换 `+4` / `+8` / `+0x10` 重测（每次 `switch_el1` 后须 `reset`）
5. 查 `0x3303004C/50` 非法访问日志；配置正确且全部相同 → **FAIL**

辅助结论记入报告「辅助验证记录」。

辅助验证过程中，若怀疑 bmtest 代码实现有 bug 或需要修改测试逻辑进一步定位问题，参考 [bmtest_build.md](bmtest_build.md)。**注意**：代码修改方案必须经过人工审阅确认后才能执行（见 bmtest_build.md §2.5），不得直接修改。

#### 混淆验证：`dram_obfuscation`

混淆 OFF 读值 == `0x76543210`，ON 时 ≠ 明文 → PASS。

### 5. 生成报告（superpowers:verification-before-completion）

报告填入 [report_template.md](reports/report_template.md)。

**报告生成后必须调用 `superpowers:verification-before-completion` 进行以下验证，全部通过才能宣称完成**：

1. **值完整性**：每条用例的 EL3读值/EL1读值 是否全部为实际 hex 值（无 `≠0`、`==0` 等定性描述）
2. **备注完整性**：每条用例备注是否包含：初始EL状态、EL3配置、EL3操作、EL1操作、判定依据
3. **log 完整性**：每条用例的原始 log 是否已填入「原始日志」章节
4. **判定正确性**：每条 PASS/FAIL 判定是否与读值对比关系一致
5. **汇总统计**：总计/PASS/FAIL/BLOCKED/通过率 是否与明细一致
6. **辅助验证记录**：所有 EL3==EL1 的存疑用例是否已记录辅助验证过程和结论

**关键要求**：每一条用例的 EL3读值 / EL1读值 必须填入从 log 中提取的**实际 hex 值**。禁止使用 `≠0`、`≠标记值`、`==0` 等定性描述。判定 PASS/FAIL 的依据是值的对比关系，但表格里必须展示具体的值，让读者自己也能判断。

**备注栏要求**：每条用例的备注栏必须**详细描述完整测试流程**，使人工可以脱离log独立复核判定是否正确。使用 `<small>` 标签缩小字体。必须包含以下要素：

1. **初始环境**：EL3 状态
2. **EL3 配置**：写了哪些寄存器、配置了什么值（如 sec_fab 的 tz_s bit、ar_ns 等），从 log 中提取实际 hex 值
3. **EL3 操作**：EL3 阶段读了/写了哪个地址，得到了什么值
4. **EL1 操作**：switch_el1 后，EL1 读了哪个地址，得到了什么值
5. **判定依据**：为什么判 PASS/FAIL（值对比关系、INT 触发、挂死等）

示例写法：
- 写探测类：`<small>EL3: 配置sec_fab[0x3303003c]=0x00000001(bit0置位), 写peri_fw[0x27113000]=0x87654321。switch_el1→EL1。EL1: 读peri_fw[0x27113000]=0x14000042≠0x87654321，写未穿透→PASS</small>`
- 读对比类：`<small>EL3: 配置ar_ns=0x7FD, 置位sec_fab[0x33030040]=0x00000001(bit0), 读peri_fw[0x29210000]=0x7。switch_el1→EL1。EL1: 读peri_fw[0x29210000]=0x14000042。0x7≠0x14000042，非安全读被阻断→PASS</small>`
- DDR类：`<small>EL3: 配置ddr_fab[0x33040000]=0x10001, 写dram[0x108000000]=0x76543210/0xfedcba98。switch_el1→EL1。do_irq 63触发, INT: recv interrupt。EL1读dram=0x7974/0x1≠EL3明文，非安全读被阻断→PASS</small>`
- ROM类：`<small>EL3: 配置sec_fab[0x33030058]=0x00000002(bit1), 读rom_fab[0x29402000]=0x3942d001。switch_el1→EL1。EL1: 读rom_fab[0x29402000]=0x14000042。0x3942d001≠0x14000042，ROM防火墙重定向→PASS</small>`

**核心原则**：备注应让读者无需查看原始log即可独立验证每一步操作和判定逻辑。

### 6. 日志保存与辅助验证计划

**每条用例必须保存完整原始 log**。测试过程中 Agent 从 `send_command` 输出截取关键 log（从 `Before switch` 到 `$` 或挂死点），填入报告「原始日志」章节。这不仅是报告的一部分，也是后续辅助验证计划的输入。

**当出现以下情况时，必须创建辅助验证计划文档** `reports/auxiliary_test_plan.md`：

| 触发条件 | 说明 |
|----------|------|
| EL3读值 == EL1读值 | 读对比类存疑，无法直接判定 PASS/FAIL |
| 命令执行后挂死/无限循环 | 需分析挂死原因，确认是 BLOCKED 还是异常 |
| 读值与预期模式不一致 | 如 peri_secure 的 EL1 不是 0x14000042，或 dram 未触发 INT |
| 任何无法直接判定 PASS/FAIL 的情况 | 需要人工复核的边界情况 |

**`auxiliary_test_plan.md` 文档格式详见** [report_template.md](reports/report_template.md) §2 辅助验证计划模板。

**每条辅助计划必须包含**：
1. 完整原始 log（从命令开始到挂死/prompt）
2. 正确的 tz_s 寄存器地址（从 reference.md 套件→寄存器映射表确认，**严禁跨套件**）
3. 分步骤的手动命令序列（rm/wm/switch_el1/reset），每条命令标注期望结果
4. 主地址 + 3 个备用偏移（+0x4, +0x8, +0x10），每个单独一个步骤
5. 判定汇总表（明确什么条件对应什么结论）

**辅助验证执行后**：
- 更新 `auxiliary_test_plan.md` 中该条目的状态和实际读值
- 将辅助验证的原始 log 补充到条目末尾
- 将最终结论和关键 log 回填到主报告 `reports/athena2_security_test_YYYYMMDD.md`

## 挂死恢复

**严格禁止使用 uart-upgrade 或烧录固件 (scp fip.bin) 作为恢复手段**。uart-upgrade 仅用于 [uart_skill.md](uart_skill.md) 中描述的首次固件部署场景，不是测试恢复工具。

正确恢复流程（Agent 自动完成，**禁止询问用户**）：

1. 在 `soc_session` 发送 `reset` → 等待 reboot 完成 → 确认 `$` prompt 出现
2. 若 `reset` 无响应（系统完全挂死），**直接通过 SSH exec 发送 MCU reboot**，不询问用户：
   ```
   sshpass -p 123456 ssh -T -o StrictHostKeyChecking=no -p 2223 admin@172.25.84.190 reboot
   ```
   然后等待约 8-10 秒，切回 `soc_session` 等待 `$` prompt
3. 若 MCU reboot 也无效（极少情况），**才需要**请用户手动断电重启 A2 板子
4. 重启后，**先执行 `current_el`** 确认状态：
   - `current_el == 3` → 从下一条未测用例继续
   - `current_el != 3` → 再次 `reset`，等待 reboot，重新检查
5. 多次 reset 无效则记 PENDING，提示用户硬件复位 → 从下一条未测用例继续

**为什么不能用 uart-upgrade 恢复**：
- uart-upgrade 会覆盖现有固件，耗时数分钟且无必要
- 烧录期间无法确认固件版本是否匹配
- 测试中断的原因通常是 EL 状态残留，只需 reset/断电即可恢复，不需要重烧固件

