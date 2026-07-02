# bmtest 代码修改与编译

| 属性 | 说明 |
|------|------|
| **触发场景** | 辅助验证阶段，agent 怀疑 bmtest 代码有 bug，或需要修改 bmtest 行为作进一步验证 |
| **前提条件** | SDK 源码已就绪，编译环境（Docker 容器）可用 |

---

## 1. 源码定位

bmtest 源码位于 SDK 根目录下的 `cvi_bmtest/` 仓库。A2 security 测试相关代码路径：

| 路径（相对于 SDK 根目录） | 说明 |
|------|------|
| `cvi_bmtest/athena2/test/security/` | A2 security 测试用例目录 |
| `cvi_bmtest/athena2/test/security/testcase_security.c` | 测试用例主文件，命令注册 + 测试逻辑 |
| `cvi_bmtest/athena2/test/security/reg_sec_fab_firewall.h` | 安全防火墙寄存器定义 |
| `cvi_bmtest/athena2/test/security/switch_el.S` | EL3/EL1 切换汇编 |
| `cvi_bmtest/athena2/test/security/arch.h` | 架构宏定义（SPSR、SCR、HCR 等） |
| `cvi_bmtest/athena2/test/security/config.mk` | 模块编译配置 |

### 代码结构

`testcase_security.c` 的核心结构：

- **`test_func_table[]`** (第138行)：测试函数表，数组中的每个函数对应一个可通过 CLI 调用的测试命令
- **`test_func_t`**：测试函数签名 `int (*)(void)`，返回 0 为 PASS
- 辅助函数：`switch_el3_to_el1()` / `switch_el3_to_s_el1()` 用于 EL 切换，`get_arm_current_el()` 获取当前异常级别

### 如何定位与搜索

Agent 需要修改代码时，应当先用 `grep` 搜索相关符号，确认要改的逻辑位置：

```bash
# 搜索特定 CLI 命令的实现
grep -rn "命令名" cvi_bmtest/athena2/test/security/

# 搜索特定寄存器操作
grep -rn "寄存器地址" cvi_bmtest/athena2/test/security/

# 搜索 firewall 配置宏
grep -rn "宏名" cvi_bmtest/athena2/test/security/reg_sec_fab_firewall.h
```

---

## 2. 修改约束

### 允许的操作

| 操作 | 说明 |
|------|------|
| 修改 `testcase_security.c` 中的测试逻辑 | 如增减寄存器检查、调整读值对比方式、修改测试地址 |
| 修改 `reg_sec_fab_firewall.h` 中的寄存器定义 | 如修正地址偏移、mask 值 |
| 修改 `switch_el.S` 中的 EL 切换逻辑 | 如调整 SCR/HCR 配置、SPSR 设置 |
| 修改 `arch.h` 中的宏定义 | 如调整异常级别配置值 |
| 修改 `config.mk` 中的编译选项 | 如增减 CFLAGS、包含路径 |

### 禁止的操作

| 操作 | 原因 |
|------|------|
| 修改 cvi_bmtest 仓库外的 SDK 代码 | 影响范围不可控，可能导致其他模块异常 |
| 引入外部库或新依赖 | 裸机环境无法支持 |
| 修改编译脚本（`build/envsetup_soc.sh` 等） | 编译环境是共享的 |
| 修改 Makefile 顶层逻辑 | 可能破坏其他平台的编译 |

### 修改原则

- **最小改动**：只改必要的位置，不要重构或优化无关代码
- **保持兼容**：不改动现有命令的接口（命令名、参数格式）
- **修改后说明**：每次修改后，记录改了什么、为什么改，便于回溯

---

## 2.5 代码修改审阅（人工闸门）

**在 agent 确定代码修改方案后、实际修改代码前，必须中断流程，等待人工审阅确认。**

### 触发条件

辅助验证过程中，agent 判定问题根因可能与 bmtest 代码实现有关（如逻辑 bug、配置错误），需要修改代码来修复或进一步定位时。

### 审阅流程

1. **Agent 输出修改方案**：包括涉及的文件、具体改动内容、改动理由（关联哪个测试问题/现象）
2. **中断等待**：明确告知用户需要审阅，**不要直接修改代码**
3. **人工确认后**：用户同意方案后，agent 再执行代码修改和编译

### 审阅内容模板

```markdown
### 代码修改审阅请求

- **关联问题**: <哪个用例/什么现象，关联哪个辅助验证条目>
- **问题分析**: <为什么认为是代码问题，排除硬件/环境因素的依据>
- **修改文件**: <文件路径>
- **修改方案**: 
  - <具体改动1及理由>
  - <具体改动2及理由>
- **影响范围**: <是否影响其他测试命令>
- **回退方案**: <如何撤销修改>
```

> **严格禁止**：在人工确认前，agent 不得执行任何代码编辑操作。即使 agent 高度确信修改正确，也必须等待审阅。

---

## 3. 编译步骤

分两步：**先编译 bmtest.bin**，**再把 bmtest.bin 打包进 fip.bin**。

> **注意**：以下所有路径相对于 SDK 根目录。

### 3.0 编译前预检

在编译前，确认 `cvi_bmtest/athena2/Makefile` 中 `TOOL_PATH` 指向正确：

```makefile
TOOL_PATH := $(shell pwd)/../..
```

`TOOL_PATH` 应指向 SDK 根目录，交叉编译工具链位于 `$(TOOL_PATH)/host-tools/gcc/...`。

验证 host-tools 存在：

```bash
ls <SDK_ROOT>/host-tools/gcc/
```

如果 `cvi_bmtest` 不在 SDK 根目录下（例如在 SDK 外部独立维护），需要相应修改 `TOOL_PATH` 使其指向 SDK 根目录。

### 3.1 编译 bmtest.bin

在 `cvi_bmtest/athena2` 目录下执行 security 构建脚本：

```bash
cd cvi_bmtest/athena2
./build_scripts/build_security.sh
```

构建脚本实际执行 `make clean; make TEST_CASE=security RUN_ENV=DDR BOARD=ASIC`。

产物路径：`cvi_bmtest/athena2/out/bmtest.bin`

### 3.2 更新 fip.mk 中的 MONITOR_PATH

编辑 `fsbl/make_helpers/fip.mk`，将 `MONITOR_PATH` 改为新编译的 `bmtest.bin` 路径。

当前 `fip.mk` 中 `BOOT_CPU=aarch64` 分支有两处 `MONITOR_PATH`（`RELEASE_VER=1` 和 `else` 分支），**两处都需修改**为新路径：

```makefile
MONITOR_PATH = <SDK_ROOT>/cvi_bmtest/athena2/out/bmtest.bin
```

> **重要**：必须使用绝对路径，且路径指向刚编译生成的 `bmtest.bin`。

### 3.3 编译 fip.bin

回到 SDK 根目录，执行 fsbl 编译：

```bash
cd <SDK_ROOT>
source build/envsetup_soc.sh && defconfig edge_wevb_emmc && build_fsbl
```

`fip-all` 目标会通过 `--MONITOR='${MONITOR_PATH}'` 将 `bmtest.bin` 打包进 `fip.bin`。

最终产物路径：`install/soc_edge_wevb_emmc/fip.bin`

> 产物已直接生成在 [uart_skill.md](uart_skill.md) SCP 命令引用的路径下，无需额外 cp。

### 3.4 验证编译成功

```bash
ls -la install/soc_edge_wevb_emmc/fip.bin
```

### 编译失败排查

| 现象 | 可能原因 | 排查方向 |
|------|----------|----------|
| `build_security.sh` 找不到编译器 | `TOOL_PATH` 不正确 | 检查 `Makefile` 第4行 `TOOL_PATH` 是否指向 SDK 根目录，确认 `host-tools/gcc/` 存在 |
| `build_security.sh` 报 syntax error | 修改引入了 C 语法错误 | 检查修改行的语法，确认括号配对、分号 |
| `build_security.sh` 报 undefined reference | 引用了不存在的符号 | 检查是否拼写错误，或符号是否在链接范围内 |
| 寄存器宏未定义 | `reg_sec_fab_firewall.h` 中缺少对应宏 | 检查宏名拼写，或补充定义 |
| `build_fsbl` 报找不到 MONITOR_PATH | `fip.mk` 中路径不正确 | 确认 `MONITOR_PATH` 指向的新 `bmtest.bin` 绝对路径存在 |
| 编译通过但烧录后行为异常 | 逻辑错误 | 复查修改逻辑，必要时回退到原始版本对比 |

---

## 4. 与烧录流程衔接

编译完成后，按 [uart_skill.md](uart_skill.md) 第4节的流程完成烧录和验证：

```
问题分析 → 提出修改方案 → 【人工审阅确认 §2.5】 → 修改代码 → 编译 bmtest.bin(§3.1) → 更新 fip.mk(§3.2) → build_fsbl(§3.3) → SCP 上传 → A2 重启 → uart-upgrade → 验证
                                                                                                    ↑
                                                                                             uart_skill.md §4
```

> **关键闸门**：流程在「人工审阅确认」处中断，必须等待用户确认修改方案后才能继续执行后续步骤。

**完整命令序列**（服务器端执行）：

```bash
# 1. 编译 bmtest.bin
cd cvi_bmtest/athena2 && ./build_scripts/build_security.sh

# 2. 确认 bmtest.bin
ls -la out/bmtest.bin

# 3. 更新 fip.mk 中的 MONITOR_PATH（需用绝对路径）
#    编辑 fsbl/make_helpers/fip.mk，修改两处 MONITOR_PATH

# 4. 编译 fip.bin
cd <SDK_ROOT>
source build/envsetup_soc.sh && defconfig edge_wevb_emmc && build_fsbl

# 5. 确认 fip.bin
ls -la install/soc_edge_wevb_emmc/fip.bin

# 6. SCP 上传到跳板机（IP 需确认）
sshpass -p 123456 scp -P 2222 -o StrictHostKeyChecking=no \
  install/soc_edge_wevb_emmc/fip.bin admin@172.25.84.190:.

# 7. A2 重启 + 触发 uart-upgrade（见 uart_skill.md §4 步骤4-6）
```

---

## 5. 修改记录

每次修改 bmtest 代码后，Agent 应当在测试报告中记录：

```markdown
### bmtest 代码修改记录

- **时间**: YYYY-MM-DD HH:MM
- **修改文件**: <修改的源码文件路径>
- **修改内容**: <简要描述改了什么>
- **修改原因**: <为什么需要这个修改，关联哪个测试问题>
- **fip.mk 变更**: MONITOR_PATH 从 <原路径> 改为 <新路径>
- **验证结果**: <重新编译烧录后的验证结论>
```
