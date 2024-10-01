# 3.5: システムコントローラ

## 3.5.1 概要

システムコントローラはシステムのソフトリセット、クロック制御などの
レジスタを通じてチップを制御します。リセットとクロックについては
他の章で説明していまう。この章ではその他のシステム機能モジュールの
構成レジストとステータスレジスタについて説明します。

## 3.5.2 機能説明

### 3.5.2.1 グローバルリセットの有効化

システムのグローバルソフトリセット、デバッグのリセット、
ウォッチドッグのリセットは`reg_sw_root_reset_en`レジスタを
構成することで発行できます。詳細は`reg_sw_root_reset_en`で
説明されています。

### 3.5.2.2 システムDMAチャネルのマッピング

このDMAには8つのチャネルがあり、0から7のdma要求インタフェースを
個別に構成することができます。0から7のdma要求インタフェースは
システム制御レジスタ`sdma_dma_ch_remap0`と`sdma_dma_ch_remap1`に
より、以下の表のペリフェラルインタフェースのいずれかにマッピング
することができます。1つのペリフェラルインタフェースを複数の
チャンネルに割り当ててはいけません。構成の手順は次のとおりです。

1. DMAチャネルイメージレジスタの`sdma_dma_ch_remap0`と
   `sdma_dma_ch_remap1`を構成します。
2. `update_dma_remap_0_3`と`update_dma_remap_4_7`に1を書き込んで
   マッピングを有効にします。

| 番号 | DMAインタフェース | 番号 | DMAインタフェース |
|-----:|:------------------|-----:|:------------------|
| 0 | dma_rx_req_i2s0 | 24 | dma_rx_req_i2c0 |
| 1 | dma_tx_req_i2s0 | 25 | dma_tx_req_i2c0 |
| 2 | dma_rx_req_i2s1 | 26 | dma_rx_req_i2c1 |
| 3 | dma_tx_req_i2s1 | 27 | dma_tx_req_i2c1 |
| 4 | dma_rx_req_i2s2 | 28 | dma_rx_req_i2c2 |
| 5 | dma_tx_req_i2s2 | 29 | dma_tx_req_i2c2 |
| 6 | dma_rx_req_i2s3 | 30 | dma_rx_req_i2c3 |
| 7 | dma_tx_req_i2s3 | 31 | dma_tx_req_i2c3 |
| 8 | dma_rx_req_n_uart0 | 32 | dma_rx_req_i2c4 |
| 9 | dma_tx_req_n_uart0 | 33 | dma_tx_req_i2c4 |
| 10 | dma_rx_req_n_uart1 | 34 | dma_rx_req_tdm0 |
| 11 | dma_tx_req_n_uart1 | 35 | dma_tx_req_tdm0 |
| 12 | dma_rx_req_n_uart2 | 36 | dma_rx_req_tdm1 |
| 13 | dma_tx_req_n_uart2 | 37 | dma_req_audsrc |
| 14 | dma_rx_req_n_uart3 | 38 | dma_req_spi_nand |
| 15 | dma_tx_req_n_uart3 | 39 | dma_req_spi_nor |
| 16 | dma_rx_req_spi0 | 40 | dma_rx_req_n_uart4 |
| 17 | dma_tx_req_spi0 | 41 | dma_tx_req_n_uart4 |
| 18 | dma_rx_req_spi1 | 42 | dma_req_spi_nor1 |
| 19 | dma_tx_req_spi1 |  |  |
| 20 | dma_rx_req_spi2 |  |  |
| 21 | dma_tx_req_spi2 |  |  |
| 22 | dma_rx_req_spi3 |  |  |
| 23 | dma_tx_req_spi3 |  |  |

### 3.5.2.3 DDR AXI Urgent/Qosの構成

DDR AXIの優先順位は`ddr_axi_urgent_o`, `ddr_axi_urgen`,
`ddr_axi_qos_0`, `ddr_axi_qos_1`により制御できます。詳細は
DDRコントローラのセクションを参照してください。

## 3.5.3 システム制御レジスタ

### 3.5.3.1 システム制御レジスタの概要

**基底アドレス: 0x0300_0000**

| 名前 | オフセット | 記述 |
|:---- |-----------:|:-----|
| conf_info | 0x004 | conf_info |
| sys_ctrl_reg | 0x008 | sys_ctrl_reg |
| usb_phy_ctrl_reg | 0x048 | usb_phy_ctrl_reg |
| sdma_dma_ch_remap0 | 0x154 | sdma_dma_ch_remap0 |
| sdma_dma_ch_remap1 | 0x158 | sdma_dma_ch_remap1 |
| top_timer_clk_sel | 0x1a0 | top_timer_clk_sel |
| top_wdt_ctrl | 0x1a8 | top_timer_clk_sel |
| ddr_axi_urgent_ow | 0x1b8 | ddr_axi_urgent_ow |
| ddr_axi_urgent | 0x1bc | ddr_axi_urgent |
| ddr_axi_qos_0 | 0x1d8 | ddr_axi_qos_0 |
| ddr_axi_qos_1 | 0x1dc | ddr_axi_qos_1 |
| sd_pwrsw_ctrl | 0x1f4 | sd_pwrsw_ctrl |
| sd_pwrsw_time | 0x1f8 | sd_pwrsw_time |
| ddr_axi_qos_ow | 0x23c | ddr_axi_qos_ow |
| sd_ctrl_opt | 0x294 | additional control register for sd |
| sdma_dma_int_mux | 0x298 | Mux sdma channel interrupt to different processors |

### 3.5.3.2 システム制御レジスタの詳細

#### sd_pwrsw_ctrl

| ビット | 名前 | アクセス | 記述 | リセット値 |
|-------:|:---- |:---------|:-----|:-----------|
| 0 | reg_en_pwrsw | R/W | 18/33 IO power switch enable | 0x0 |
| 1 | reg_pwrsw_vsel | R/W | 18/33 IO power switch enable<br/>0: 3.3v<br/>1: 1.8v | 0x1 |
| 2 | reg_pwrsw_disc | R/W | 18/33 IO power switch discharge enable | 0x0 |
| 3 | reg_pwrsw_auto | R/W | 18/33 IO power switch auto protect enable | 0x1 |
| 31:4 | Reserved | | |

#### sd_pwrsw_time

| ビット | 名前 | アクセス | 記述 | リセット値 |
|-------:|:---- |:---------|:-----|:-----------|
| 15:0 | reg_tpwrup | R/W | 18/33 IO power switch, power up<br/>protection time is 500x40ns = 20us | 0x1f4<br/>(500) |
| 31:16 | reg_tpwrdn | R/W | 18/33 IO power switch, power down<br/>protection time is 500x40ns = 20us | 0x1f4 |

#### sd_ctrl_opt

| ビット | 名前 | アクセス | 記述 | リセット値 |
|-------:|:---- |:---------|:-----|:-----------|
| 0 | reg_sd0_carddet_ow | R/W | sd0 card detect over write enable | 0x0 |
| 1 | reg_sd0_carddet_sw | R/W | sd0 card detect over write value | 0x0 |
| 7:2 | Reserved | | |
| 8 | reg_sd1_carddet_ow | R/W | sd1 card detect over write enable | 0x0 |
| 9 | reg_sd1_carddet_sw | R/W | sd1 card detect over write value | 0x0 |
| 15:10 | Reserved | | |
| 16 | reg_sd0_pwr_en_polarity | R/W | off chip sd0 pwr en polarity<br/>0: SD_LDO power ctrl high is power on,<br/>low is power off<br/>1: SD_LDO power ctrl high is power off,<br/>low is power on | 0x0 |
| 31:17 | Reserved | | |
