# 1.5: アドレス空間マッピング

| 開始アドレス<br/>[31:0] | 終了アドレス<br/>[31:0] | 機能 | サイズ |
|:------------------------|:------------------------|:-----|-------:|
| 0x01000000 | 0x018FFFFF | reserve |  |
| 0x01900000 | 0x01900FFF | ap_mailbox | 4K |
| 0x01901000 | 0x01901FFF | ap_system_ctrl | 4K |
| 0x01902000 | 0x019EFFFF | reserve |  |
| 0x01A00000 | 0x01FFFFFF | reserve |  |
| 0x02000000 | 0x02FFFFFF | reserve | 64K |
| 0x03000000 | 0x03000FFF | TOP_MISC control register | 4K |
| 0x03001000 | 0x03001FFF | PINMUX control register | 4K |
| 0x03002000 | 0x03002FFF | CLKGEN/PLL control register | 4K |
| 0x03003000 | 0x03003FFF | RSTGEN control register | 4K |
| 0x03004000 | 0x03005FFF | reserve |  |
| 0x03006000 | 0x03006FFF | reserve | 4K |
| 0x03007000 | 0x03008FFF | reserve |  |
| 0x03009000 | 0x03009FFF | reserve | 4K |
| 0x0300A000 | 0x0300AFFF | reserve | 4K |
| 0x0300B000 | 0x0300FFFF | reserve |  |
| 0x03010000 | 0x03010FFF | WATCH DOG0 control register | 4K |
| 0x03011000 | 0x03011FFF | WATCH DOG1 control register | 4K |
| 0x03012000 | 0x03012FFF | WATCH DOG2 control register | 4K |
| 0x03020000 | 0x03020FFF | GPIO0 control register | 4K |
| 0x03021000 | 0x03021FFF | GPIO1 control register | 4K |
| 0x03022000 | 0x03022FFF | GPIO2 control register | 4K |
| 0x03023000 | 0x03023FFF | GPIO3 control register | 4K |
| 0x03024000 | 0x0302FFFF | reserve |  |
| 0x03030000 | 0x03030FFF | WGN0 control register | 4K |
| 0x03031000 | 0x03031FFF | WGN1 control register | 4K |
| 0x03032000 | 0x03032FFF | WGN2 control register | 4K |
| 0x03033000 | 0x0303FFFF | reserve |  |
| 0x03040000 | 0x0304FFFF | KEYSCAN control register | 64K |
| 0x03050000 | 0x0305FFFF | EFUSE control register | 64K |
| 0x03060000 | 0x03060FFF | PWM0 control register | 4K |
| 0x03061000 | 0x03061FFF | PWM1 control register | 4K |
| 0x03062000 | 0x03062FFF | PWM2 control register | 4K |
| 0x03063000 | 0x03063FFF | PWM3 control register | 4K |
| 0x03064000 | 0x0309FFFF | reserve |  |
| 0x030A0000 | 0x030AFFFF | TIMER control register | 64K |
| 0x030C0000 | 0x030CFFFF | reserve |  |
| 0x030D0000 | 0x030D0FFF | reserve | 4K |
| 0x030D1000 | 0x030D1FFF | reserve | 4K |
| 0x030D2000 | 0x030D2FFF | reserve | 4K |
| 0x030D3000 | 0x030DFFFF | reserve |  |
| 0x030E0000 | 0x030EFFFF | TEMPSEN control register | 64K |
| 0x030F0000 | 0x030FFFFF | SARADC control register | 64K |
| 0x04000000 | 0x0400FFFF | I2C0 control register | 64K |
| 0x04010000 | 0x0401FFFF | I2C1 control register | 64K |
| 0x04020000 | 0x0402FFFF | I2C2 control register | 64K |
| 0x04030000 | 0x0403FFFF | I2C3 control register | 64K |
| 0x04040000 | 0x0404FFFF | I2C4 control register | 64K |
| 0x04050000 | 0x0405FFFF | reserve |  |
| 0x04060000 | 0x0406FFFF | SPI_NAND control register | 64K |
| 0x04070000 | 0x0407FFFF | ETH0 control register | 64K |
| 0x04080000 | 0x040FFFFF | reserve |  |
| 0x04100000 | 0x04107FFF | I2S0 control register | 64K |
| 0x04108000 | 0x0410FFFF | I2S Global control register | 64K |
| 0x04110000 | 0x0411FFFF | I2S1 control register | 64K |
| 0x04120000 | 0x0412FFFF | I2S2 control register | 64K |
| 0x04130000 | 0x0413FFFF | I2S3 control register | 64K |
| 0x04140000 | 0x0414FFFF | UART0 control register | 64K |
| 0x04150000 | 0x0415FFFF | UART1 control register | 64K |
| 0x04160000 | 0x0416FFFF | UART2 control register | 64K |
| 0x04170000 | 0x0417FFFF | UART3 control register | 64K |
| 0x04180000 | 0x0418FFFF | SPI0 control register | 64K |
| 0x04190000 | 0x0419FFFF | SPI1 control register | 64K |
| 0x041A0000 | 0x041AFFFF | SPI2 control register | 64K |
| 0x041B0000 | 0x041BFFFF | SPI3 control register | 64K |
| 0x041C0000 | 0x041CFFFF | UART4 control register | 64K |
| 0x041D0000 | 0x041DFFFF | AUDSRC control register | 64K |
| 0x041E0000 | 0x042FFFFF | reserve |  |
| 0x04300000 | 0x0430FFFF | reserve |  |
| 0x04310000 | 0x0431FFFF | SD0 control register | 64K |
| 0x04320000 | 0x0432FFFF | SD1 control register | 64K |
| 0x04330000 | 0x0433FFFF | DMA control register | 64K |
| 0x04340000 | 0x0434FFFF | USB control register | 64K |
| 0x04350000 | 0x043FFFFF | reserve |  |
| 0x04400000 | 0x0440FFFF | ROM 内存空间 | 64K |
| 0x04410000 | 0x04FFFFFF | reserve |  |
| 0x05000000 | 0x05000FFF | reserve | 4K |
| 0x05020000 | 0x05020FFF | RTCSYS_Timer control register | 4K |
| 0x05021000 | 0x05021FFF | RTCSYS_GPIO control register | 4K |
| 0x05022000 | 0x05022FFF | RTCSYS_UART control register | 4K |
| 0x05023000 | 0x05023FFF | RTCSYS_INTR control register | 4K |
| 0x05024000 | 0x05024FFF | RTCSYS_MBOX control register | 4K |
| 0x05025000 | 0x05025FFF | RTCSYS_CTRL control register | 4K |
| 0x05026000 | 0x05026FFF | RTCSYS_CORE | 4K |
| 0x05027000 | 0x05027FFF | RTCSYS_IO control register | 4K |
| 0x05028000 | 0x05028FFF | RTCSYS_OSC control register | 4K |
| 0x05029000 | 0x05029FFF | reserve | 4K |
| 0x0502A000 | 0x0502AFFF | RTCSYS_32kless control register | 4K |
| 0x0502B000 | 0x0502BFFF | RTCSYS_I2C control register | 4K |
| 0x0502C000 | 0x0502CFFF | RTCSYS_SAR control register | 4K |
| 0x0502D000 | 0x0502DFFF | RTCSYS_WDT control register | 4K |
| 0x0502E000 | 0x0502EFFF | RTCSYS_IRRXcontrol register | 4K |
| 0x05200000 | 0x053FFFFF | RTCSYS_SRAM | 8K |
| 0x05400000 | 0x057FFFFF | RTCSYS_SPINOR | 4MB |
| 0x08000000 | 0x08001FFF | reserve | 8K |
| 0x08004000 | 0x08005FFF | DDR Controler control register | 8K |
| 0x08006000 | 0x08007FFF | reserve | 8K |
| 0x08008000 | 0x08009FFF | DDR AXI Monitor control register | 8K |
| 0x0800A000 | 0x0800BFFF | DDR Global control register | 8K |
| 0x08010000 | 0x08011FFF | reserve | 8K |
| 0x08012000 | 0x08013FFF | reserve | 8K |
| 0x08014000 | 0x09FFFFFF | reserve |  |
| 0x0A000000 | 0x0A07FFFF | ISP control register | 512K |
| 0x0A080000 | 0x0A0803FF | sc_top control register | 1K |
| 0x0A080400 | 0x0A080BFF | reserve | 2K |
| 0x0A080C00 | 0x0A080CFF | osd enc control register | 256B |
| 0x0A080D00 | 0x0A080FFF | reserve | 768B |
| 0x0A081000 | 0x0A081FFF | reserve | 4K |
| 0x0A082000 | 0x0A082FFF | img_v control register | 4K |
| 0x0A083000 | 0x0A083FFF | img_d control register | 4K |
| 0x0A084000 | 0x0A084FFF | sc_d control register | 4K |
| 0x0A085000 | 0x0A085FFF | sc_v1 control register | 4K |
| 0x0A086000 | 0x0A086FFF | sc_v2 control register | 4K |
| 0x0A087000 | 0x0A087FFF | reserve | 4K |
| 0x0A088000 | 0x0A088FFF | reserve | 4K |
| 0x0A089000 | 0x0A089FFF | reserve | 4K |
| 0x0A08A000 | 0x0A08AFFF | reserve | 4K |
| 0x0A08B000 | 0x0A08BFFF | cmdq control register | 4K |
| 0x0A08C000 | 0x0A08CFFF | reserve | 4K |
| 0x0A08D000 | 0x0A08DFFF | reserve | 4K |
| 0x0A08E000 | 0x0A09FFFF | reserve | 72K |
| 0x0A0A0000 | 0x0A0AFFFF | reserve | 64K |
| 0x0A0A0000 | 0x0A0BFFFF | reserve | 64K |
| 0x0A0C0000 | 0x0A0C1FFF | ldc control register | 8K |
| 0x0A0C2000 | 0x0A0C3FFF | VI0/MIPI_RX0 control register | 8K |
| 0x0A0C4000 | 0x0A0C5FFF | reserve | 8K |
| 0x0A0C6000 | 0x0A0C7FFF | reserve | 8K |
| 0x0A0C8000 | 0x0A0C9FFF | VIPSYS control register | 8K |
| 0x0A0CA000 | 0x0A0CFFFF | reserve | 24K |
| 0x0A0D0000 | 0x0A0D0FFF | CSI_PHY control register | 4K |
| 0x0A0D1000 | 0x0A0D1FFF | reserve | 4K |
| 0x0A0D2000 | 0x0AFFFFFF | reserve |  |
| 0x0B000000 | 0x0B00FFFF | JPEG codec control register | 64K |
| 0x0B010000 | 0x0B01FFFF | H.264 codec control register | 64K |
| 0x0B020000 | 0x0B02FFFF | H.265 codec control register | 64K |
| 0x0B030000 | 0x0BFFFFFF | reserve |  |
| 0x0C000000 | 0x0FFFFFFF | reserve |  |
| 0x10000000 | 0x1FFFFFFF | SPI_NOR 内存空间 | 256M |
| 0x20000000 | 0x7FFFFFFF | reserve |  |
| 0x80000000 | 0xFFFFFFFF | DDR 内存空间 | 2G |
