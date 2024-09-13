# 3.4 割り込みシステム

表3-5にこのチップの割り込みソースを示します。

| 番号 | ソース | 番号 | ソース | 番号 | ソース |
|:----:|:-------|:----:|:-------|:----:|:-------|
| 16 | TEMPSENS | 48 | UART4 | 80 | Timer1 |
| 17 | RTC Alarm | 49 | I2C0 | 81 | Timer2 |
| 18 | RTC Longpress | 50 | I2C1 | 82 | Timer3 |
| 19 | VBAT DET | 51 | I2C2 | 83 | Timer4 |
| 20 | JPEG | 52 | I2C3 | 84 | Timer5 |
| 21 | H264 | 53 | I2C4 | 85 | Timer6 |
| 22 | H265 | 54 | SPI1 | 86 | Timer7 |
| 23 | VC SBM | 55 | SPI2 | 87 | peri_firewall |
| 24 | ISP | 56 | SPI3 | 88 | hsperi_firewall |
| 25 | SC_TOP | 57 | SPI4 | 89 | ddr_fw |
| 26 | CSI_MAC0 | 58 | Watch Dog1 | 90 | rom_firewall |
| 27 | CSI_MAC1 | 59 | KEYSCAN | 91 | SPACC |
| 28 | LDC | 60 | GPIO0 | 92 | TRNG |
| 29 | System DMA | 61 | GPIO1 | 93 | ddr_axi_mon |
| 30 | USB | 62 | GPIO2 | 94 | ddr_pi_phy |
| 31 | Ethnet0 | 63 | GPIO3 | 95 | SPI_NOR |
| 32 | Ethnet0 | 64 | Wiegand0 | 96 | EPHY |
| 33 | - | 65 | Wiegand1 | 97 | IVE |
| 34 | - | 66 | Wiegand2 | 98 | - |
| 35 | SD0 Wakup | 67 | RTC MBOX | 99 | - |
| 36 | SD0 | 68 | | 100 | SARADC |
| 37 | SD1 Wakup | 69 | RTC IRRX | 101 | mbox |
| 38 | SD1 | 70 | RTC GPIO | | |
| 39 | SPI_NAND | 71 | RTC UART | | |
| 40 | I2S0 | 72 | RTC SPI_NOR | | |
| 41 | I2S1 | 73 | RTC I2C | | |
| 42 | I2S2 | 74 | RTC WDG | | |
| 43 | I2S3 | 75 | TPU | | |
| 44 | UART0 | 76 | TDMA | | |
| 45 | UART1 | 77 | - | | |
| 46 | UART2 | 78 | - | | |
| 47 | UART3 | 79 | Timer0 | | |
