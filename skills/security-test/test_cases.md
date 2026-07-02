# Security 测试命令清单
（内容回到测试用例的大表）
(agent根据spec、驱动代码设计额外的用例)

| Index | 模块           | 名称                  | 命令                     | 测试目标                            | 测试步骤                                                                                           |
| ----- | ------------ | ------------------- | ---------------------- | ------------------------------- | ---------------------------------------------------------------------------------------------- |
| 0     | peri         | PERI_INTC3          | `peri_secure 0`        | 验证 PERI_INTC3 的 secure 防火墙写保护   | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 1     | peri         | PERI_INTC2          | `peri_secure 1`        | 验证 PERI_INTC2 的 secure 防火墙写保护   | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 2     | peri         | PERI_INTC1          | `peri_secure 2`        | 验证 PERI_INTC1 的 secure 防火墙写保护   | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 3     | peri         | PERI_INTC0          | `peri_secure 3`        | 验证 PERI_INTC0 的 secure 防火墙写保护   | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 4     | peri         | PERI_OTP            | `peri_secure 4`        | 验证 PERI_OTP 的 secure 防火墙写保护     | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 5     | peri         | PERI_MAILBOX        | `peri_secure 5`        | 验证 PERI_MAILBOX 的 secure 防火墙写保护 | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 6     | peri         | PERI_SARADC         | `peri_secure 6`        | 验证 PERI_SARADC 的 secure 防火墙写保护  | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 7     | peri         | PERI_TEMPSEN        | `peri_secure 7`        | 验证 PERI_TEMPSEN 的 secure 防火墙写保护 | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 8     | peri         | PERI_PMCTL          | `peri_secure 8`        | 验证 PERI_PMCTL 的 secure 防火墙写保护   | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 9     | peri         | PERI_TIMER          | `peri_secure 9`        | 验证 PERI_TIMER 的 secure 防火墙写保护   | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 10    | peri         | PERI_PWM4           | `peri_secure 10`       | 验证 PERI_PWM4 的 secure 防火墙写保护    | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 11    | peri         | PERI_PWM3           | `peri_secure 11`       | 验证 PERI_PWM3 的 secure 防火墙写保护    | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 12    | peri         | PERI_PWM2           | `peri_secure 12`       | 验证 PERI_PWM2 的 secure 防火墙写保护    | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 13    | peri         | PERI_PWM1           | `peri_secure 13`       | 验证 PERI_PWM1 的 secure 防火墙写保护    | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 14    | peri         | PERI_PWM0           | `peri_secure 14`       | 验证 PERI_PWM0 的 secure 防火墙写保护    | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 15    | peri         | PERI_EFUSE          | `peri_secure 15`       | 验证 PERI_EFUSE 的 secure 防火墙写保护   | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 16    | peri         | PERI_KEYSCAN        | `peri_secure 16`       | 验证 PERI_KEYSCAN 的 secure 防火墙写保护 | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 17    | peri         | PERI_WGN1           | `peri_secure 17`       | 验证 PERI_WGN1 的 secure 防火墙写保护    | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 18    | peri         | PERI_WGN0           | `peri_secure 18`       | 验证 PERI_WGN0 的 secure 防火墙写保护    | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 19    | peri         | PERI_GPIO5          | `peri_secure 19`       | 验证 PERI_GPIO5 的 secure 防火墙写保护   | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 20    | peri         | PERI_GPIO4          | `peri_secure 20`       | 验证 PERI_GPIO4 的 secure 防火墙写保护   | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 21    | peri         | PERI_GPIO3          | `peri_secure 21`       | 验证 PERI_GPIO3 的 secure 防火墙写保护   | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 22    | peri         | PERI_GPIO2          | `peri_secure 22`       | 验证 PERI_GPIO2 的 secure 防火墙写保护   | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 23    | peri         | PERI_GPIO1          | `peri_secure 23`       | 验证 PERI_GPIO1 的 secure 防火墙写保护   | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 24    | peri         | PERI_GPIO0          | `peri_secure 24`       | 验证 PERI_GPIO0 的 secure 防火墙写保护   | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 25    | peri         | PERI_WDT2           | `peri_secure 25`       | 验证 PERI_WDT2 的 secure 防火墙写保护    | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 26    | peri         | PERI_WDT1           | `peri_secure 26`       | 验证 PERI_WDT1 的 secure 防火墙写保护    | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 27    | peri         | PERI_WDT0           | `peri_secure 27`       | 验证 PERI_WDT0 的 secure 防火墙写保护    | EL3 写探测地址 `0x87654321` → switch_el1 → EL1 读回，预期 ≠ `0x87654321`                                 |
| 28    | hsperi0      | HSPERI_SPI1         | `hsperi_secure 1 0`    | 验证 HSPERI_SPI1 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit18 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 29    | hsperi0      | HSPERI_SPI0         | `hsperi_secure 1 1`    | 验证 HSPERI_SPI0 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit17 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 30    | hsperi0      | HSPERI_UART7        | `hsperi_secure 1 2`    | 验证 HSPERI_UART7 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit16 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 31    | hsperi0      | HSPERI_UART6        | `hsperi_secure 1 3`    | 验证 HSPERI_UART6 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit15 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 32    | hsperi0      | HSPERI_UART5        | `hsperi_secure 1 4`    | 验证 HSPERI_UART5 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit14 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 33    | hsperi0      | HSPERI_UART4        | `hsperi_secure 1 5`    | 验证 HSPERI_UART4 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit13 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 34    | hsperi0      | HSPERI_UART3        | `hsperi_secure 1 6`    | 验证 HSPERI_UART3 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit12 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 35    | hsperi0      | HSPERI_UART2        | `hsperi_secure 1 7`    | 验证 HSPERI_UART2 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit11 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 36    | hsperi0      | HSPERI_UART1        | `hsperi_secure 1 8`    | 验证 HSPERI_UART1 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit10 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 37    | hsperi0      | HSPERI_UART0        | `hsperi_secure 1 9`    | 验证 HSPERI_UART0 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit9 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1           |
| 38    | hsperi0      | HSPERI_I2S_DW       | `hsperi_secure 1 10`   | 验证 HSPERI_I2S_DW 的非安全读阻断        | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit8 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1           |
| 39    | hsperi0      | HSPERI_I2S_AUDSRC   | `hsperi_secure 1 11`   | 验证 HSPERI_I2S_AUDSRC 的非安全读阻断    | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit7 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1           |
| 40    | hsperi0      | HSPERI_I2S5         | `hsperi_secure 1 12`   | 验证 HSPERI_I2S5 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit6 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1           |
| 41    | hsperi0      | HSPERI_I2S4         | `hsperi_secure 1 13`   | 验证 HSPERI_I2S4 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit5 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1           |
| 42    | hsperi0      | HSPERI_I2S3         | `hsperi_secure 1 14`   | 验证 HSPERI_I2S3 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit4 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1           |
| 43    | hsperi0      | HSPERI_I2S2         | `hsperi_secure 1 15`   | 验证 HSPERI_I2S2 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit3 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1           |
| 44    | hsperi0      | HSPERI_I2S1         | `hsperi_secure 1 16`   | 验证 HSPERI_I2S1 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit2 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1           |
| 45    | hsperi0      | HSPERI_I2S0         | `hsperi_secure 1 17`   | 验证 HSPERI_I2S0 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit1 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1           |
| 46    | hsperi0      | HSPERI_I2S_GLOBAL   | `hsperi_secure 1 18`   | 验证 HSPERI_I2S_GLOBAL 的非安全读阻断    | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit0 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1           |
| 47    | hsperi0      | HSPERI_ETH1_CFG     | `hsperi_secure 1 19`   | 验证 HSPERI_ETH1_CFG 的非安全读阻断      | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit28 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 48    | hsperi0      | HSPERI_ETH0_CFG     | `hsperi_secure 1 20`   | 验证 HSPERI_ETH0_CFG 的非安全读阻断      | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit27 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 49    | hsperi0      | HSPERI_SPI_NAND     | `hsperi_secure 1 21`   | 验证 HSPERI_SPI_NAND 的非安全读阻断      | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit21 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 50    | hsperi0      | HSPERI_I2C9         | `hsperi_secure 1 22`   | 验证 HSPERI_I2C9 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit31 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 51    | hsperi0      | HSPERI_I2C8         | `hsperi_secure 1 23`   | 验证 HSPERI_I2C8 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit30 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 52    | hsperi0      | HSPERI_I2C7         | `hsperi_secure 1 24`   | 验证 HSPERI_I2C7 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit29 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 53    | hsperi0      | HSPERI_I2C6         | `hsperi_secure 1 25`   | 验证 HSPERI_I2C6 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit26 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 54    | hsperi0      | HSPERI_I2C5         | `hsperi_secure 1 26`   | 验证 HSPERI_I2C5 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit25 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 55    | hsperi0      | HSPERI_I2C4         | `hsperi_secure 1 27`   | 验证 HSPERI_I2C4 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit24 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 56    | hsperi0      | HSPERI_I2C3         | `hsperi_secure 1 28`   | 验证 HSPERI_I2C3 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit23 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 57    | hsperi0      | HSPERI_I2C2         | `hsperi_secure 1 29`   | 验证 HSPERI_I2C2 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit22 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 58    | hsperi0      | HSPERI_I2C1         | `hsperi_secure 1 30`   | 验证 HSPERI_I2C1 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit20 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 59    | hsperi0      | HSPERI_I2C0         | `hsperi_secure 1 31`   | 验证 HSPERI_I2C0 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi0 tz_s bit19 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 60    | hsperi1      | HSPERI_ROM          | `hsperi_secure 0 0`    | 验证 HSPERI_ROM 的非安全读阻断           | 配置 ar_ns=0x7FD → 置位 hsperi1 tz_s bit26 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 61    | hsperi1      | HSPERI_SDMA1_CFG    | `hsperi_secure 0 1`    | 验证 HSPERI_SDMA1_CFG 的非安全读阻断     | 配置 ar_ns=0x7FD → 置位 hsperi1 tz_s bit25 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 62    | hsperi1      | HSPERI_SDMA0_CFG    | `hsperi_secure 0 2`    | 验证 HSPERI_SDMA0_CFG 的非安全读阻断     | 配置 ar_ns=0x7FD → 置位 hsperi1 tz_s bit24 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 63    | hsperi1      | HSPERI_SD2_CFG      | `hsperi_secure 0 3`    | 验证 HSPERI_SD2_CFG 的非安全读阻断       | 配置 ar_ns=0x7FD → 置位 hsperi1 tz_s bit23 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 64    | hsperi1      | HSPERI_SD1_CFG      | `hsperi_secure 0 4`    | 验证 HSPERI_SD1_CFG 的非安全读阻断       | 配置 ar_ns=0x7FD → 置位 hsperi1 tz_s bit22 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 65    | hsperi1      | HSPERI_SD0_CFG      | `hsperi_secure 0 5`    | 验证 HSPERI_SD0_CFG 的非安全读阻断       | 配置 ar_ns=0x7FD → 置位 hsperi1 tz_s bit21 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 66    | hsperi1      | HSPERI_EMMC_CFG     | `hsperi_secure 0 6`    | 验证 HSPERI_EMMC_CFG 的非安全读阻断      | 配置 ar_ns=0x7FD → 置位 hsperi1 tz_s bit22 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 67    | hsperi1      | HSPERI_CAN1         | `hsperi_secure 0 7`    | 验证 HSPERI_CAN1 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi1 tz_s bit20 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 68    | hsperi1      | HSPERI_CAN0         | `hsperi_secure 0 8`    | 验证 HSPERI_CAN0 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi1 tz_s bit19 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 69    | hsperi1      | HSPERI_SPI3         | `hsperi_secure 0 9`    | 验证 HSPERI_SPI3 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi1 tz_s bit20 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 70    | hsperi1      | HSPERI_SPI2         | `hsperi_secure 0 10`   | 验证 HSPERI_SPI2 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 hsperi1 tz_s bit19 → EL3 读探测地址 → switch_el1 → EL1 读回对比，预期 EL3≠EL1          |
| 71    | ddr_firewall | DDR_SECURE_REGION1  | `dram_secure_region 0` | 验证 DDR region0 的非安全读/写阻断        | 配置 DDR region0 start/end → EL3 写 0x76543210/0xFEDCBA98 → 读回 → switch_el1 → EL1 读回对比，预期 EL3≠EL1 |
| 72    | ddr_firewall | DDR_SECURE_REGION2  | `dram_secure_region 1` | 验证 DDR region1 的非安全读/写阻断        | 配置 DDR region1 start/end → EL3 写 0x76543210/0xFEDCBA98 → 读回 → switch_el1 → EL1 读回对比，预期 EL3≠EL1 |
| 73    | ddr_firewall | DDR_SECURE_REGION3  | `dram_secure_region 2` | 验证 DDR region2 的非安全读/写阻断        | 配置 DDR region2 start/end → EL3 写 0x76543210/0xFEDCBA98 → 读回 → switch_el1 → EL1 读回对比，预期 EL3≠EL1 |
| 74    | ddr_firewall | DDR_SECURE_REGION4  | `dram_secure_region 3` | 验证 DDR region3 的非安全读/写阻断        | 配置 DDR region3 start/end → EL3 写 0x76543210/0xFEDCBA98 → 读回 → switch_el1 → EL1 读回对比，预期 EL3≠EL1 |
| 75    | ddr_firewall | DDR_SECURE_REGION5  | `dram_secure_region 4` | 验证 DDR region4 的非安全读/写阻断        | 配置 DDR region4 start/end → EL3 写 0x76543210/0xFEDCBA98 → 读回 → switch_el1 → EL1 读回对比，预期 EL3≠EL1 |
| 76    | ddr_firewall | DDR_SECURE_REGION6  | `dram_secure_region 5` | 验证 DDR region5 的非安全读/写阻断        | 配置 DDR region5 start/end → EL3 写 0x76543210/0xFEDCBA98 → 读回 → switch_el1 → EL1 读回对比，预期 EL3≠EL1 |
| 77    | ddr_firewall | DDR_SECURE_REGION7  | `dram_secure_region 6` | 验证 DDR region6 的非安全读/写阻断        | 配置 DDR region6 start/end → EL3 写 0x76543210/0xFEDCBA98 → 读回 → switch_el1 → EL1 读回对比，预期 EL3≠EL1 |
| 78    | ddr_firewall | DDR_SECURE_REGION8  | `dram_secure_region 7` | 验证 DDR region7 的非安全读/写阻断        | 配置 DDR region7 start/end → EL3 写 0x76543210/0xFEDCBA98 → 读回 → switch_el1 → EL1 读回对比，预期 EL3≠EL1 |
| 79    | ddr_firewall | DDR_OBFUSCATION     | `dram_obfuscation 0`   | 验证 DDR 数据混淆功能                   | EL3 混淆 OFF: 读 DRAM == 0x76543210 → 混淆 ON: 读 DRAM ≠ 明文                                          |
| 80    | rom_firewall | ROM_SECURE_REGION0  | `rom_secure_region 0`  | 验证 ROM region0 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 rom tz_s bit0 → EL3 读 ROM region0 → switch_el1 → EL1 读回对比，预期 EL3≠EL1       |
| 81    | rom_firewall | ROM_SECURE_REGION1  | `rom_secure_region 1`  | 验证 ROM region1 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 rom tz_s bit1 → EL3 读 ROM region1 → switch_el1 → EL1 读回对比，预期 EL3≠EL1       |
| 82    | rom_firewall | ROM_SECURE_REGION2  | `rom_secure_region 2`  | 验证 ROM region2 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 rom tz_s bit2 → EL3 读 ROM region2 → switch_el1 → EL1 读回对比，预期 EL3≠EL1       |
| 83    | rom_firewall | ROM_SECURE_REGION3  | `rom_secure_region 3`  | 验证 ROM region3 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 rom tz_s bit3 → EL3 读 ROM region3 → switch_el1 → EL1 读回对比，预期 EL3≠EL1       |
| 84    | rom_firewall | ROM_SECURE_REGION4  | `rom_secure_region 4`  | 验证 ROM region4 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 rom tz_s bit4 → EL3 读 ROM region4 → switch_el1 → EL1 读回对比，预期 EL3≠EL1       |
| 85    | rom_firewall | ROM_SECURE_REGION5  | `rom_secure_region 5`  | 验证 ROM region5 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 rom tz_s bit5 → EL3 读 ROM region5 → switch_el1 → EL1 读回对比，预期 EL3≠EL1       |
| 86    | rom_firewall | ROM_SECURE_REGION6  | `rom_secure_region 6`  | 验证 ROM region6 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 rom tz_s bit6 → EL3 读 ROM region6 → switch_el1 → EL1 读回对比，预期 EL3≠EL1       |
| 87    | rom_firewall | ROM_SECURE_REGION7  | `rom_secure_region 7`  | 验证 ROM region7 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 rom tz_s bit7 → EL3 读 ROM region7 → switch_el1 → EL1 读回对比，预期 EL3≠EL1       |
| 88    | rom_firewall | ROM_SECURE_REGION8  | `rom_secure_region 8`  | 验证 ROM region8 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 rom tz_s bit8 → EL3 读 ROM region8 → switch_el1 → EL1 读回对比，预期 EL3≠EL1       |
| 89    | rom_firewall | ROM_SECURE_REGION9  | `rom_secure_region 9`  | 验证 ROM region9 的非安全读阻断          | 配置 ar_ns=0x7FD → 置位 rom tz_s bit9 → EL3 读 ROM region9 → switch_el1 → EL1 读回对比，预期 EL3≠EL1       |
| 90    | rom_firewall | ROM_SECURE_REGION10 | `rom_secure_region 10` | 验证 ROM region10 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 rom tz_s bit10 → EL3 读 ROM region10 → switch_el1 → EL1 读回对比，预期 EL3≠EL1     |
| 91    | rom_firewall | ROM_SECURE_REGION11 | `rom_secure_region 11` | 验证 ROM region11 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 rom tz_s bit11 → EL3 读 ROM region11 → switch_el1 → EL1 读回对比，预期 EL3≠EL1     |
| 92    | rom_firewall | ROM_SECURE_REGION12 | `rom_secure_region 12` | 验证 ROM region12 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 rom tz_s bit12 → EL3 读 ROM region12 → switch_el1 → EL1 读回对比，预期 EL3≠EL1     |
| 93    | rom_firewall | ROM_SECURE_REGION13 | `rom_secure_region 13` | 验证 ROM region13 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 rom tz_s bit13 → EL3 读 ROM region13 → switch_el1 → EL1 读回对比，预期 EL3≠EL1     |
| 94    | rom_firewall | ROM_SECURE_REGION14 | `rom_secure_region 14` | 验证 ROM region14 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 rom tz_s bit14 → EL3 读 ROM region14 → switch_el1 → EL1 读回对比，预期 EL3≠EL1     |
| 95    | rom_firewall | ROM_SECURE_REGION15 | `rom_secure_region 15` | 验证 ROM region15 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 rom tz_s bit15 → EL3 读 ROM region15 → switch_el1 → EL1 读回对比，预期 EL3≠EL1     |
| 96    | rom_firewall | ROM_SECURE_REGION16 | `rom_secure_region 16` | 验证 ROM region16 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 rom tz_s bit16 → EL3 读 ROM region16 → switch_el1 → EL1 读回对比，预期 EL3≠EL1     |
| 97    | rom_firewall | ROM_SECURE_REGION17 | `rom_secure_region 17` | 验证 ROM region17 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 rom tz_s bit17 → EL3 读 ROM region17 → switch_el1 → EL1 读回对比，预期 EL3≠EL1     |
| 98    | rom_firewall | ROM_SECURE_REGION18 | `rom_secure_region 18` | 验证 ROM region18 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 rom tz_s bit18 → EL3 读 ROM region18 → switch_el1 → EL1 读回对比，预期 EL3≠EL1     |
| 99    | rom_firewall | ROM_SECURE_REGION19 | `rom_secure_region 19` | 验证 ROM region19 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 rom tz_s bit19 → EL3 读 ROM region19 → switch_el1 → EL1 读回对比，预期 EL3≠EL1     |
| 100   | rom_firewall | ROM_SECURE_REGION20 | `rom_secure_region 20` | 验证 ROM region20 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 rom tz_s bit20 → EL3 读 ROM region20 → switch_el1 → EL1 读回对比，预期 EL3≠EL1     |
| 101   | rom_firewall | ROM_SECURE_REGION21 | `rom_secure_region 21` | 验证 ROM region21 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 rom tz_s bit21 → EL3 读 ROM region21 → switch_el1 → EL1 读回对比，预期 EL3≠EL1     |
| 102   | rom_firewall | ROM_SECURE_REGION22 | `rom_secure_region 22` | 验证 ROM region22 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 rom tz_s bit22 → EL3 读 ROM region22 → switch_el1 → EL1 读回对比，预期 EL3≠EL1     |
| 103   | rom_firewall | ROM_SECURE_REGION23 | `rom_secure_region 23` | 验证 ROM region23 的非安全读阻断         | 配置 ar_ns=0x7FD → 置位 rom tz_s bit23 → EL3 读 ROM region23 → switch_el1 → EL1 读回对比，预期 EL3≠EL1     |
| 104   | rom_firewall | ROM_READ_LOCK0      | `rom_read_lock 0`      | 验证 ROM region0 的读锁定             | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit0 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1              |
| 105   | rom_firewall | ROM_READ_LOCK1      | `rom_read_lock 1`      | 验证 ROM region1 的读锁定             | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit1 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1              |
| 106   | rom_firewall | ROM_READ_LOCK2      | `rom_read_lock 2`      | 验证 ROM region2 的读锁定             | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit2 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1              |
| 107   | rom_firewall | ROM_READ_LOCK3      | `rom_read_lock 3`      | 验证 ROM region3 的读锁定             | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit3 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1              |
| 108   | rom_firewall | ROM_READ_LOCK4      | `rom_read_lock 4`      | 验证 ROM region4 的读锁定             | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit4 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1              |
| 109   | rom_firewall | ROM_READ_LOCK5      | `rom_read_lock 5`      | 验证 ROM region5 的读锁定             | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit5 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1              |
| 110   | rom_firewall | ROM_READ_LOCK6      | `rom_read_lock 6`      | 验证 ROM region6 的读锁定             | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit6 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1              |
| 111   | rom_firewall | ROM_READ_LOCK7      | `rom_read_lock 7`      | 验证 ROM region7 的读锁定             | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit7 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1              |
| 112   | rom_firewall | ROM_READ_LOCK8      | `rom_read_lock 8`      | 验证 ROM region8 的读锁定             | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit8 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1              |
| 113   | rom_firewall | ROM_READ_LOCK9      | `rom_read_lock 9`      | 验证 ROM region9 的读锁定             | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit9 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1              |
| 114   | rom_firewall | ROM_READ_LOCK10     | `rom_read_lock 10`     | 验证 ROM region10 的读锁定            | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit10 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1             |
| 115   | rom_firewall | ROM_READ_LOCK11     | `rom_read_lock 11`     | 验证 ROM region11 的读锁定            | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit11 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1             |
| 116   | rom_firewall | ROM_READ_LOCK12     | `rom_read_lock 12`     | 验证 ROM region12 的读锁定            | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit12 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1             |
| 117   | rom_firewall | ROM_READ_LOCK13     | `rom_read_lock 13`     | 验证 ROM region13 的读锁定            | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit13 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1             |
| 118   | rom_firewall | ROM_READ_LOCK14     | `rom_read_lock 14`     | 验证 ROM region14 的读锁定            | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit14 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1             |
| 119   | rom_firewall | ROM_READ_LOCK15     | `rom_read_lock 15`     | 验证 ROM region15 的读锁定            | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit15 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1             |
| 120   | rom_firewall | ROM_READ_LOCK16     | `rom_read_lock 16`     | 验证 ROM region16 的读锁定            | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit16 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1             |
| 121   | rom_firewall | ROM_READ_LOCK17     | `rom_read_lock 17`     | 验证 ROM region17 的读锁定            | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit17 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1             |
| 122   | rom_firewall | ROM_READ_LOCK18     | `rom_read_lock 18`     | 验证 ROM region18 的读锁定            | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit18 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1             |
| 123   | rom_firewall | ROM_READ_LOCK19     | `rom_read_lock 19`     | 验证 ROM region19 的读锁定            | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit19 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1             |
| 124   | rom_firewall | ROM_READ_LOCK20     | `rom_read_lock 20`     | 验证 ROM region20 的读锁定            | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit20 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1             |
| 125   | rom_firewall | ROM_READ_LOCK21     | `rom_read_lock 21`     | 验证 ROM region21 的读锁定            | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit21 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1             |
| 126   | rom_firewall | ROM_READ_LOCK22     | `rom_read_lock 22`     | 验证 ROM region22 的读锁定            | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit22 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1             |
| 127   | rom_firewall | ROM_READ_LOCK23     | `rom_read_lock 23`     | 验证 ROM region23 的读锁定            | 配置 ar_ns=0x7FD → 置位 ROM psmsk bit23 → EL3 读 ROM → switch_el1 → EL1 读回对比，预期 EL3≠EL1             |
| 128   | rom_firewall | ROM_DEFINE_REGION   | `rom_define_region 0`  | 验证用户自定义 ROM 区的非安全读阻断            | 配置用户 ROM 区 → EL3 读自定义区 → switch_el1 → EL1 读回对比，预期 EL3≠EL1                                      |


