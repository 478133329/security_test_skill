# A2 Security 测试计划

- **生成时间**: 2026-07-02 12:10
- **固件**: athena2_ASIC_security.bin (cvi_bmtest/athena2/out/)
- **用例总数**: 129
- **说明**: 全新验证，不参考任何之前测试结果

## 执行顺序

| Index | 命令 | 模块 | 分类 | 预期测试结果 |
|-------|------|------|------|------------|
| 0 | `peri_secure 0` | PERI_INTC3 | peri | EL1读值≠0x87654321 |
| 1 | `peri_secure 1` | PERI_INTC2 | peri | EL1读值≠0x87654321 |
| 2 | `peri_secure 2` | PERI_INTC1 | peri | EL1读值≠0x87654321 |
| 3 | `peri_secure 3` | PERI_INTC0 | peri | EL1读值≠0x87654321 |
| 4 | `peri_secure 4` | PERI_OTP | peri | EL1读值≠0x87654321 |
| 5 | `peri_secure 5` | PERI_MAILBOX | peri | EL1读值≠0x87654321 |
| 6 | `peri_secure 6` | PERI_SARADC | peri | EL1读值≠0x87654321 |
| 7 | `peri_secure 7` | PERI_TEMPSEN | peri | EL1读值≠0x87654321 |
| 8 | `peri_secure 8` | PERI_PMCTL | peri | EL1读值≠0x87654321 |
| 9 | `peri_secure 9` | PERI_TIMER | peri | EL1读值≠0x87654321 |
| 10 | `peri_secure 10` | PERI_PWM4 | peri | EL1读值≠0x87654321 |
| 11 | `peri_secure 11` | PERI_PWM3 | peri | EL1读值≠0x87654321 |
| 12 | `peri_secure 12` | PERI_PWM2 | peri | EL1读值≠0x87654321 |
| 13 | `peri_secure 13` | PERI_PWM1 | peri | EL1读值≠0x87654321 |
| 14 | `peri_secure 14` | PERI_PWM0 | peri | EL1读值≠0x87654321 |
| 15 | `peri_secure 15` | PERI_EFUSE | peri | EL1读值≠0x87654321 |
| 16 | `peri_secure 16` | PERI_KEYSCAN | peri | EL1读值≠0x87654321 |
| 17 | `peri_secure 17` | PERI_WGN1 | peri | EL1读值≠0x87654321 |
| 18 | `peri_secure 18` | PERI_WGN0 | peri | EL1读值≠0x87654321 |
| 19 | `peri_secure 19` | PERI_GPIO5 | peri | EL1读值≠0x87654321 |
| 20 | `peri_secure 20` | PERI_GPIO4 | peri | EL1读值≠0x87654321 |
| 21 | `peri_secure 21` | PERI_GPIO3 | peri | EL1读值≠0x87654321 |
| 22 | `peri_secure 22` | PERI_GPIO2 | peri | EL1读值≠0x87654321 |
| 23 | `peri_secure 23` | PERI_GPIO1 | peri | EL1读值≠0x87654321 |
| 24 | `peri_secure 24` | PERI_GPIO0 | peri | EL1读值≠0x87654321 |
| 25 | `peri_secure 25` | PERI_WDT2 | peri | EL1读值≠0x87654321 |
| 26 | `peri_secure 26` | PERI_WDT1 | peri | EL1读值≠0x87654321 |
| 27 | `peri_secure 27` | PERI_WDT0 | peri | EL1读值≠0x87654321 |
| 28 | `hsperi_secure 1 0` | HSPERI_SPI1 | hsperi0 | EL3读值≠EL1读值 |
| 29 | `hsperi_secure 1 1` | HSPERI_SPI0 | hsperi0 | EL3读值≠EL1读值 |
| 30 | `hsperi_secure 1 2` | HSPERI_UART7 | hsperi0 | EL3读值≠EL1读值 |
| 31 | `hsperi_secure 1 3` | HSPERI_UART6 | hsperi0 | EL3读值≠EL1读值 |
| 32 | `hsperi_secure 1 4` | HSPERI_UART5 | hsperi0 | EL3读值≠EL1读值 |
| 33 | `hsperi_secure 1 5` | HSPERI_UART4 | hsperi0 | EL3读值≠EL1读值 |
| 34 | `hsperi_secure 1 6` | HSPERI_UART3 | hsperi0 | EL3读值≠EL1读值 |
| 35 | `hsperi_secure 1 7` | HSPERI_UART2 | hsperi0 | EL3读值≠EL1读值 |
| 36 | `hsperi_secure 1 8` | HSPERI_UART1 | hsperi0 | EL3读值≠EL1读值 |
| 37 | `hsperi_secure 1 9` | HSPERI_UART0 | hsperi0 | EL3读值≠EL1读值 |
| 38 | `hsperi_secure 1 10` | HSPERI_I2S_DW | hsperi0 | EL3读值≠EL1读值 |
| 39 | `hsperi_secure 1 11` | HSPERI_I2S_AUDSRC | hsperi0 | EL3读值≠EL1读值 |
| 40 | `hsperi_secure 1 12` | HSPERI_I2S5 | hsperi0 | EL3读值≠EL1读值 |
| 41 | `hsperi_secure 1 13` | HSPERI_I2S4 | hsperi0 | EL3读值≠EL1读值 |
| 42 | `hsperi_secure 1 14` | HSPERI_I2S3 | hsperi0 | EL3读值≠EL1读值 |
| 43 | `hsperi_secure 1 15` | HSPERI_I2S2 | hsperi0 | EL3读值≠EL1读值 |
| 44 | `hsperi_secure 1 16` | HSPERI_I2S1 | hsperi0 | EL3读值≠EL1读值 |
| 45 | `hsperi_secure 1 17` | HSPERI_I2S0 | hsperi0 | EL3读值≠EL1读值 |
| 46 | `hsperi_secure 1 18` | HSPERI_I2S_GLOBAL | hsperi0 | EL3读值≠EL1读值 |
| 47 | `hsperi_secure 1 19` | HSPERI_ETH1_CFG | hsperi0 | EL3读值≠EL1读值 |
| 48 | `hsperi_secure 1 20` | HSPERI_ETH0_CFG | hsperi0 | EL3读值≠EL1读值 |
| 49 | `hsperi_secure 1 21` | HSPERI_SPI_NAND | hsperi0 | EL3读值≠EL1读值 |
| 50 | `hsperi_secure 1 22` | HSPERI_I2C9 | hsperi0 | EL3读值≠EL1读值 |
| 51 | `hsperi_secure 1 23` | HSPERI_I2C8 | hsperi0 | EL3读值≠EL1读值 |
| 52 | `hsperi_secure 1 24` | HSPERI_I2C7 | hsperi0 | EL3读值≠EL1读值 |
| 53 | `hsperi_secure 1 25` | HSPERI_I2C6 | hsperi0 | EL3读值≠EL1读值 |
| 54 | `hsperi_secure 1 26` | HSPERI_I2C5 | hsperi0 | EL3读值≠EL1读值 |
| 55 | `hsperi_secure 1 27` | HSPERI_I2C4 | hsperi0 | EL3读值≠EL1读值 |
| 56 | `hsperi_secure 1 28` | HSPERI_I2C3 | hsperi0 | EL3读值≠EL1读值 |
| 57 | `hsperi_secure 1 29` | HSPERI_I2C2 | hsperi0 | EL3读值≠EL1读值 |
| 58 | `hsperi_secure 1 30` | HSPERI_I2C1 | hsperi0 | EL3读值≠EL1读值 |
| 59 | `hsperi_secure 1 31` | HSPERI_I2C0 | hsperi0 | EL3读值≠EL1读值 |
| 60 | `hsperi_secure 0 0` | HSPERI_ROM | hsperi1 | EL3读值≠EL1读值 |
| 61 | `hsperi_secure 0 1` | HSPERI_SDMA1_CFG | hsperi1 | EL3读值≠EL1读值 |
| 62 | `hsperi_secure 0 2` | HSPERI_SDMA0_CFG | hsperi1 | EL3读值≠EL1读值 |
| 63 | `hsperi_secure 0 3` | HSPERI_SD2_CFG | hsperi1 | EL3读值≠EL1读值 |
| 64 | `hsperi_secure 0 4` | HSPERI_SD1_CFG | hsperi1 | EL3读值≠EL1读值 |
| 65 | `hsperi_secure 0 5` | HSPERI_SD0_CFG | hsperi1 | EL3读值≠EL1读值 |
| 66 | `hsperi_secure 0 6` | HSPERI_EMMC_CFG | hsperi1 | EL3读值≠EL1读值 |
| 67 | `hsperi_secure 0 7` | HSPERI_CAN1 | hsperi1 | EL3读值≠EL1读值 |
| 68 | `hsperi_secure 0 8` | HSPERI_CAN0 | hsperi1 | EL3读值≠EL1读值 |
| 69 | `hsperi_secure 0 9` | HSPERI_SPI3 | hsperi1 | EL3读值≠EL1读值 |
| 70 | `hsperi_secure 0 10` | HSPERI_SPI2 | hsperi1 | EL3读值≠EL1读值 |
| 71 | `dram_secure_region 0` | DDR_SECURE_REGION1 | ddr_firewall | EL1读值≠EL3明文，或INT触发 |
| 72 | `dram_secure_region 1` | DDR_SECURE_REGION2 | ddr_firewall | EL1读值≠EL3明文，或INT触发 |
| 73 | `dram_secure_region 2` | DDR_SECURE_REGION3 | ddr_firewall | EL1读值≠EL3明文，或INT触发 |
| 74 | `dram_secure_region 3` | DDR_SECURE_REGION4 | ddr_firewall | EL1读值≠EL3明文，或INT触发 |
| 75 | `dram_secure_region 4` | DDR_SECURE_REGION5 | ddr_firewall | EL1读值≠EL3明文，或INT触发 |
| 76 | `dram_secure_region 5` | DDR_SECURE_REGION6 | ddr_firewall | EL1读值≠EL3明文，或INT触发 |
| 77 | `dram_secure_region 6` | DDR_SECURE_REGION7 | ddr_firewall | EL1读值≠EL3明文，或INT触发 |
| 78 | `dram_secure_region 7` | DDR_SECURE_REGION8 | ddr_firewall | EL1读值≠EL3明文，或INT触发 |
| 79 | `dram_obfuscation 0` | DDR_OBFUSCATION | ddr_firewall | 混淆OFF读==0x76543210，混淆ON读≠明文 |
| 80 | `rom_secure_region 0` | ROM_SECURE_REGION0 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 81 | `rom_secure_region 1` | ROM_SECURE_REGION1 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 82 | `rom_secure_region 2` | ROM_SECURE_REGION2 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 83 | `rom_secure_region 3` | ROM_SECURE_REGION3 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 84 | `rom_secure_region 4` | ROM_SECURE_REGION4 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 85 | `rom_secure_region 5` | ROM_SECURE_REGION5 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 86 | `rom_secure_region 6` | ROM_SECURE_REGION6 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 87 | `rom_secure_region 7` | ROM_SECURE_REGION7 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 88 | `rom_secure_region 8` | ROM_SECURE_REGION8 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 89 | `rom_secure_region 9` | ROM_SECURE_REGION9 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 90 | `rom_secure_region 10` | ROM_SECURE_REGION10 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 91 | `rom_secure_region 11` | ROM_SECURE_REGION11 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 92 | `rom_secure_region 12` | ROM_SECURE_REGION12 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 93 | `rom_secure_region 13` | ROM_SECURE_REGION13 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 94 | `rom_secure_region 14` | ROM_SECURE_REGION14 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 95 | `rom_secure_region 15` | ROM_SECURE_REGION15 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 96 | `rom_secure_region 16` | ROM_SECURE_REGION16 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 97 | `rom_secure_region 17` | ROM_SECURE_REGION17 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 98 | `rom_secure_region 18` | ROM_SECURE_REGION18 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 99 | `rom_secure_region 19` | ROM_SECURE_REGION19 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 100 | `rom_secure_region 20` | ROM_SECURE_REGION20 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 101 | `rom_secure_region 21` | ROM_SECURE_REGION21 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 102 | `rom_secure_region 22` | ROM_SECURE_REGION22 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 103 | `rom_secure_region 23` | ROM_SECURE_REGION23 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 104 | `rom_read_lock 0` | ROM_READ_LOCK0 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 105 | `rom_read_lock 1` | ROM_READ_LOCK1 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 106 | `rom_read_lock 2` | ROM_READ_LOCK2 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 107 | `rom_read_lock 3` | ROM_READ_LOCK3 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 108 | `rom_read_lock 4` | ROM_READ_LOCK4 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 109 | `rom_read_lock 5` | ROM_READ_LOCK5 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 110 | `rom_read_lock 6` | ROM_READ_LOCK6 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 111 | `rom_read_lock 7` | ROM_READ_LOCK7 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 112 | `rom_read_lock 8` | ROM_READ_LOCK8 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 113 | `rom_read_lock 9` | ROM_READ_LOCK9 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 114 | `rom_read_lock 10` | ROM_READ_LOCK10 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 115 | `rom_read_lock 11` | ROM_READ_LOCK11 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 116 | `rom_read_lock 12` | ROM_READ_LOCK12 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 117 | `rom_read_lock 13` | ROM_READ_LOCK13 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 118 | `rom_read_lock 14` | ROM_READ_LOCK14 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 119 | `rom_read_lock 15` | ROM_READ_LOCK15 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 120 | `rom_read_lock 16` | ROM_READ_LOCK16 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 121 | `rom_read_lock 17` | ROM_READ_LOCK17 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 122 | `rom_read_lock 18` | ROM_READ_LOCK18 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 123 | `rom_read_lock 19` | ROM_READ_LOCK19 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 124 | `rom_read_lock 20` | ROM_READ_LOCK20 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 125 | `rom_read_lock 21` | ROM_READ_LOCK21 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 126 | `rom_read_lock 22` | ROM_READ_LOCK22 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 127 | `rom_read_lock 23` | ROM_READ_LOCK23 | rom_firewall | EL1读值=0x14000042，EL3读值≠EL1读值 |
| 128 | `rom_define_region 0` | ROM_DEFINE_REGION | rom_firewall | EL3读值≠EL1读值 |

## 存疑用例预估

| Index | 命令 | 可能现象 | 排查方向 |
|-------|------|----------|----------|
| 128 | `rom_define_region 0` | 读值行为可能与预期不一致 | 检查代码中区域定义和 firewall 配置逻辑 |
| 部分 hsperi | `hsperi_secure *` | switch_el1 后可能挂死 | 检查对应 IP 是否支持非安全访问阻断 |

## 判定规则摘要

| 分类 | 条件 | 结论 |
|------|------|------|
| 写探测类 (peri) | EL1读值 ≠ 0x87654321 | PASS |
| 写探测类 (peri) | 挂死 / INT: recv interrupt | PASS(BLOCKED) |
| 读对比类 (hsperi/rom/ddr) | EL3读值 ≠ EL1读值 | PASS |
| 读对比类 (hsperi/rom/ddr) | EL3读值 == EL1读值 | 触发辅助测试 |
| 读对比类 | 挂死 | PASS(BLOCKED) |
| 读对比类 | INT: recv interrupt | PASS |
| DDR 混淆 | 混淆OFF读==0x76543210, ON读≠明文 | PASS |
| ROM 重定向 | EL1读值=0x14000042 | 防火墙正常重定向 |
