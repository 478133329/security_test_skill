# Athena2 Security Test Skill 总结

## 1. 概述

本 skill 用于在 **Athena2 (BM1688)** 裸机环境下，通过串口桥接连接，向 bmtest 固件发送 CLI 命令，逐条执行安全防火墙测试。测试覆盖 **peri**、**hsperi**、**dram**、**rom** 四类防火墙及 DDR 混淆验证，共 **129 条用例**。

---

## 2. 系统架构（三节点两层串口桥接）

```
┌──────────────────┐         SSH/SCP         ┌──────────────────┐      物理串口 (x2)      ┌─────────────────┐
│   服务器          │ ◄─────────────────────► │   跳板机          │ ◄───────────────────► │   A2 (BM1688)   │
│  (运行Claude)     │                         │  (桥接程序)       │                         │                 │
│                  │   ssh -p 2222           │                  │   COM1 ──────────────► │   SoC (bmtest)  │
│   soc_session ───┼───────────────────────► │   SSH:2222       │   交互式透传            │   CLI 交互       │
│                  │   (交互终端)             │   转发SoC串口     │                        │                 │
│                  │                         │                  │                        │                 │
│                  │   ssh -p 2223           │                  │   COM2 ──────────────► │   MCU           │
│   MCU备用 ───────┼───────────────────────► │   SSH:2223       │   SSH exec发送命令     │   reboot控制    │
│                  │   (SSH exec模式)         │   转发MCU串口     │                        │                 │
└──────────────────┘                         └──────────────────┘                        └─────────────────┘

网络隔离                       桥接程序 (单串口设计，需两个进程)             A2 板上两颗芯片
```

### 2.1 节点职责

| 节点 | IP/账号 | 角色 |
|------|--------|------|
| **服务器** | 本机 | 运行 Claude Agent，维护代码仓库，编译固件 |
| **跳板机** | `admin@172.25.84.190` (示例) | 物理串口接入，运行两个桥接进程转发串口到 SSH |
| **A2 开发板** | — | 目标测试设备，SoC 运行 bmtest，MCU 可控制 SoC 复位 |

### 2.2 双串口通道

| 通道 | SSH 端口 | 串口 | 目标 | 通信方式 | 用途 |
|------|---------|------|------|---------|------|
| **soc_session** | 2222 | COM1 | A2 SoC | MCP 交互终端 | **主通道**：发送 bmtest 命令，逐条交互 |
| **MCU 备用** | 2223 | COM2 | A2 MCU | SSH exec (`sshpass ssh ... command`) | **备用通道**：SoC 挂死时发送 `reboot` 重启 SoC |

### 2.3 跳板机能力边界

| 操作 | 2222 (SoC) | 2223 (MCU) |
|------|-----------|-----------|
| SSH 交互式连接 | = A2 串口控制台（**不是shell**） | 不可靠 |
| SSH exec 命令 | 触发 uart-upgrade 等特殊操作 | 发送 MCU 命令 (reboot) |
| SCP 上传文件 | 可上传 fip.bin | — |

---

## 3. 测试流程

```
1. 建立测试计划 ──→ 2. 建立串口通信 ──→ 3. 烧录固件 ──→ 4. 开始测试
                                                           │
                                                    ┌──────┘
                                                    ▼
                                             5. 异常测试：建立辅助测试计划
                                                    │
                                                    ├── 环境/配置问题 → 修复后继续
                                                    │
                                                    └── 怀疑bmtest问题 → 6. AI修改代码
                                                           → 人工审阅 → 重新烧录验证
                                                           │
                                                    ┌──────┘
                                                    ▼
                                             7. 生成测试报告
```

### 3.1 建立测试计划

解析 `test_cases.md` 中所有 129 条用例，生成 `reports/test_plan.md`：
- 为每条用例设定预期结果
- 确定执行顺序（按 Index 递增）
- 标注存疑用例

### 3.2 建立串口通信

按 [uart_skill.md](uart_skill.md) 建立双串口通道：
- **主通道 (2222)**：MCP 交互终端，发送 bmtest 命令
- **备用通道 (2223)**：SSH exec，SoC 挂死时发送 MCU reboot

### 3.3 烧录固件

按 [uart_skill.md](uart_skill.md) §4 将 fip.bin 烧录到 A2 开发板。开机后看到 `athena2 bmtest start` 等待 2 秒再操作。

### 3.4 开始测试

严格逐条执行，**每条用例必须完整走完以下循环**：

```
发送命令 → 等待 $ prompt → 收集 log → 判定 → reset → 等待 reboot
  → 等待 "athena2 bmtest start" → 等待2s → 按任意键进入CLI
  → 等待 $ prompt → current_el 确认为3 → 下一条
```

**禁止**：批量发送命令、跳过中间用例、不检查 current_el 就发下一条。

固件不输出 PASS/FAIL，Agent 根据 log 自行判定（详见 [reference.md](references/reference.md)）：
- 写探测类：EL1读值 ≠ 标记值 → PASS
- 读对比类：EL3读值 ≠ EL1读值 → PASS；EL3 == EL1 → 触发辅助测试
- 挂死/INT interrupt → PASS(BLOCKED) / PASS

### 3.5 异常测试：建立辅助测试计划

遇到 EL3 == EL1、挂死、读值异常等情况时，生成 `reports/auxiliary_test_plan.md`：

1. 检查 ar_ns (0x33030004 bit1) 和 tz_s 配置
2. 换 +4/+8/+0x10 备用地址重测
3. 查非法访问日志 (0x3303004C/50)
4. 检查 IP clock 和 pinmux
5. 配置正确且全部 EL3==EL1 → FAIL

**铁律**：每一步必须连接设备实际执行并记录返回值，禁止用机制分析替代实测。

### 3.6 怀疑 bmtest 问题：AI 修改代码，重新烧录验证

当辅助测试排除环境/配置因素后，根因指向 bmtest 代码时：

1. AI 提出修改方案（定位文件、行号、改动内容、理由）
2. **人工审阅确认**（必须等待，不能跳过）
3. AI 修改源码
4. 重新编译 bmtest.bin → 打包 fip.bin → 烧录到 A2
5. 重新验证相关用例
6. 结果写入报告（含代码修改记录，格式见 [bmtest_build.md](bmtest_build.md) §5）

> 详细编译/烧录步骤见 [bmtest_build.md](bmtest_build.md) 和 [uart_skill.md](uart_skill.md) §4。

### 3.7 生成测试报告

填入 `templates/report_template.md`，所有值必须为实际 hex，禁止定性描述（如 `≠0`）。

---

## 4. 测试套件总览

| 套件 | 命令 | 数量 | 测试目标 | 判定类型 |
|------|------|------|----------|----------|
| **peri** | `peri_secure <0-27>` | 28 | PERI 外设 secure 防火墙写保护 | 写探测 |
| **hsperi0** | `hsperi_secure 1 <0-31>` | 32 | 高速外设组0 非安全读阻断 | 读对比 |
| **hsperi1** | `hsperi_secure 0 <0-10>` | 11 | 高速外设组1 非安全读阻断 | 读对比 |
| **dram_region** | `dram_secure_region <0-7>` | 8 | DDR secure region 非安全访问阻断 | 读对比 |
| **dram_obf** | `dram_obfuscation 0` | 1 | DDR 数据混淆功能 | 混淆验证 |
| **rom_region** | `rom_secure_region <0-23>` | 24 | ROM secure region 非安全读阻断 | 读对比 |
| **rom_lock** | `rom_read_lock <0-23>` | 24 | ROM 读锁定 (psmsk) | 读对比 |
| **rom_define** | `rom_define_region 0` | 1 | 用户自定义 ROM 区保护 | 读对比 |
| **合计** | | **129** | | |

---

## 5. 辅助命令

| 命令 | 说明 |
|------|------|
| `rm <addr>` | 读内存/寄存器 |
| `wm <addr> <data>` | 写内存/寄存器 |
| `switch_el1` | EL3 跳入非安全 EL1 |
| `current_el` | 查看当前异常级别 |
| `reset` | 系统复位 |
| `help` | 命令列表 |

---

## 6. 文件说明

### 核心文件

| 文件 | 说明 |
|------|------|
| `SKILL.md` | **主入口**：skill 元数据、完整测试流程、Superpowers 对接、判定规则摘要、报告格式要求 |
| `test_cases.md` | **用例清单**：129 条测试用例的 Index/模块/名称/命令/目标/步骤 |
| `uart_skill.md` | **串口连接与烧录**：双串口架构、跳板机能力边界、fip.bin 升级流程、系统数据流总览 |

### 参考文档 (references/)

| 文件 | 说明 |
|------|------|
| `reference.md` | **判定规则**：所有套件的详细判定表、辅助测试流程、EL3==EL1 处理方法、基线测试、地址空间约束、套件→寄存器强制映射 |
| `security_reg.md` | **寄存器绝对地址**：防火墙寄存器完整清单、index→bit 映射、探测地址表 |
| `troubleshooting.md` | **常见问题与解决方法**：异常现象→原因→排查步骤→结论 |

### 构建相关

| 文件 | 说明 |
|------|------|
| `bmtest_build.md` | **bmtest 代码修改与编译**：源码定位、修改约束、人工审阅闸门、编译步骤、fip.bin 打包流程 |

### 模板

| 文件 | 说明 |
|------|------|
| `templates/report_template.md` | **报告模板**：汇总表 + 各套件明细表 + 辅助验证记录 + 失败项详情 + 原始日志 |

### 测试产出 (reports/)

| 文件 | 说明 |
|------|------|
| `test_plan.md` | **测试计划** (writing-plans 生成)：执行顺序、预期结果 |
| `auxiliary_test_plan.md` | **辅助验证计划**：EL3==EL1 存疑用例的逐步骤复验方案 |
| `athena2_security_test_YYYYMMDD.md` | **测试报告**：实际执行结果 |
| `progress.md` | 测试进度跟踪 |
| `code_fix_*.md` | bmtest 代码修改记录 |

---

## 7. 防火墙配置模型

```
EL3 (Secure World)                    EL1 (Non-Secure World)
        │                                      │
        ▼                                      ▼
  配置 Master ar_ns (清除bit1)            switch_el1 后
  配置 Slave tz_s (置位目标bit)           以非安全身份发起读/写
  EL3 读/写目标地址                       读/写同一目标地址
        │                                      │
        ▼                                      ▼
   读到真值 / 写入成功                   被防火墙阻断:
                                          - 返回 0x14000042 (重定向)
                                          - 触发 INT interrupt
                                          - 系统挂死 (BLOCKED)
```

---

## 8. 关键约束速查

| 约束 | 说明 |
|------|------|
| **单步交互** | 每条命令必须等 `$` prompt 后才能发下一条，绝对禁止批量 |
| **逐条 reset** | 每条用例后必须 reset |
| **EL3 验证** | 每条用例前必须 `current_el` 确认为 3 |
| **禁止烧录恢复** | 挂死时用 reset/MCU reboot，不要用 uart-upgrade |
| **禁止跨套件查寄存器** | `hsperi_secure 1` 查 `0x33030040`，`hsperi_secure 0` 查 `0x33030044`，不能互换 |
| **辅助必须实测** | 禁止用机制分析替代设备实际输出 |
| **代码修改需审阅** | bmtest 代码修改方案必须经人工确认后才能执行（闸门在修改前，非编译前） |
| **代码修改后必须重烧验证** | 怀疑 bmtest 问题的分支：修改代码 → 编译 → 烧录 → 重测 → 写入报告，缺一不可 |
| **报告值必须实际 hex** | 禁止 `≠0`、`≠标记值` 等定性描述 |
