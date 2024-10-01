# 2. SD0機能選択

<table class="tbl-cv180x">
<thead>
<tr><th>番号</th><th>名前</th><th>機能選択レジスタ</th><th >機能選択</th></tr>
</thead>
<tbody>
<tr><td>3</td><td>SD0_CLK</td><td>FMUX_GPIO_REG_IOCTRL_SD0_CLK<br/>0x0300_1000</td><td> IO SD0_CLK :<br/><font color="red">0 : SDIO0_CLK</font><br/>1 : IIC1_SDA<br/>2 : SPI0_SCK<br/>3 : XGPIOA[7]<br/>5 : PWM[15]<br/>6 : EPHY_LNK_LED<br/>7 : DBG[0]<br/>Others : Reserved </td></tr>
<tr><td>4</td><td>SD0_CMD</td><td>FMUX_GPIO_REG_IOCTRL_SD0_CMD<br/>0x0300_1004</td><td> IO SD0_CMD :<br/><font color="red">0 : SDIO0_CMD</font><br/>1 : IIC1_SCL<br/>2 : SPI0_SDO<br/>3 : XGPIOA[8]<br/>5 : PWM[14]<br/>6 : EPHY_SPD_LED<br/>7 : DBG[1]<br/>Others : Reserved </td></tr>
<tr><td>5</td><td>SD0_D0</td><td>FMUX_GPIO_REG_IOCTRL_SD0_D0<br/>0x0300_1008</td><td> IO SD0_D0 :<br/><font color="red">0 : SDIO0_D[0]</font><br/>1 : CAM_MCLK1<br/>2 : SPI0_SDI<br/>3 : XGPIOA[9]<br/>4 : UART3_TX<br/>5 : PWM[13]<br/>6 : WG0_D0<br/>7 : DBG[2]<br/>Others : Reserved </td></tr>
<tr><td>7</td><td>SD0_D1</td><td>FMUX_GPIO_REG_IOCTRL_SD0_D1<br/>0x0300_100c</td><td> IO SD0_D1 :<br/><font color="red">0 : SDIO0_D[1]</font><br/>1 : IIC1_SDA<br/>2 : AUX0<br/>3 : XGPIOA[10]<br/>4 : UART1_TX<br/>5 : PWM[12]<br/>6 : WG0_D1<br/>7 : DBG[3]<br/>Others : Reserved </td></tr>
<tr><td>8</td><td>SD0_D2</td><td>FMUX_GPIO_REG_IOCTRL_SD0_D2<br/>0x0300_1010</td><td> IO SD0_D2 :<br/><font color="red">0 : SDIO0_D[2]</font><br/>1 : IIC1_SCL<br/>2 : AUX1<br/>3 : XGPIOA[11]<br/>4 : UART1_RX<br/>5 : PWM[11]<br/>6 : WG1_D0<br/>7 : DBG[4]<br/>Others : Reserved </td></tr>
<tr><td>9</td><td>SD0_D3</td><td>FMUX_GPIO_REG_IOCTRL_SD0_D3<br/>0x0300_1014</td><td> IO SD0_D3 :<br/><font color="red">0 : SDIO0_D[3]</font><br/>1 : CAM_MCLK0<br/>2 : SPI0_CS_X<br/>3 : XGPIOA[12]<br/>4 : UART3_RX<br/>5 : PWM[10]<br/>6 : WG1_D1<br/>7 : DBG[5]<br/>Others : Reserved </td></tr>
<tr><td>11</td><td>SD0_CD</td><td>FMUX_GPIO_REG_IOCTRL_SD0_CD<br/>0x0300_1018</td><td> IO SD0_CD :<br/><font color="red">0 : SDIO0_CD</font><br/>3 : XGPIOA[13]<br/>Others : Reserved </td></tr>
<tr><td>12</td><td>SD0_PWR_EN</td><td>FMUX_GPIO_REG_IOCTRL_SD0_PWR_EN<br/>0x0300_101c</td><td> IO SD0_PWR_EN :<br/>0 : SDIO0_PWR_EN<br/><font color="red">3 : XGPIOA[14]</font><br/>Others : Reserved </td></tr>
</tbody>
</table>

- IOタイプ: 18OD33 IO
- IOグループ: G7 (SD0CD, SD0_PWR_EN), G10 (それ以外)
- パワードメイン: VDDIO_SD0_SPI
- 機能選択欄の赤字はデフォルト
