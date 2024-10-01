# 4. SDOデフォルト値

<table>
<thead>
<tr><th>ピン番号</th><th>ピン名</th><th>機能</th><th>機能選択</th><th>Pull UP/DOWN/NO<th>方向</th><th>Trap</th><th>IOタイプ</th><th>パワードメイン</th><th>ブロックグループ</th><th>Fall Safe</th></tr>
</thead>
<tbody>
<tr><td>3</td><td>SD0_CLK</td><td>SDIO0_CLK</td><td>0</td><td>PD (61Kohm)  →DRV_LO + PD</td><td>Output (Low)</td><td></td><td>18OD33</td><td>VDDIO_SD0_SPI</td><td>G10</td><td></td></tr>
<tr><td>4</td><td>SD0_CMD</td><td>SDIO0_CMD</td><td>0</td><td>PD (61Kohm)</td><td>Input</td><td></td><td>18OD33</td><td>VDDIO_SD0_SPI</td><td>G10</td><td></td></tr>
<tr><td>5</td><td>SD0_D0</td><td>SDIO0_D[0]</td><td>0</td><td>PD (61Kohm)</td><td>Input</td><td></td><td>18OD33</td><td>VDDIO_SD0_SPI</td><td>G10</td><td></td></tr>
<tr><td>7</td><td>SD0_D1</td><td>SDIO0_D[1]</td><td>0</td><td>PD (61Kohm)</td><td>Input</td><td></td><td>18OD33</td><td>VDDIO_SD0_SPI</td><td>G10</td><td></td></tr>
<tr><td>8</td><td>SD0_D2</td><td>SDIO0_D[2]</td><td>0</td><td>PD (61Kohm)</td><td>Input</td><td></td><td>18OD33</td><td>VDDIO_SD0_SPI</td><td>G10</td><td></td></tr>
<tr><td>9</td><td>SD0_D3</td><td>SDIO0_D[3]</td><td>0</td><td>PD (61Kohm)</td><td>Input</td><td></td><td>18OD33</td><td>VDDIO_SD0_SPI</td><td>G10</td><td></td></tr>
<tr><td>11</td><td>SD0_CD</td><td>SDIO0_CD</td><td>0</td><td>PU (60Kohm)</td><td>Input</td><td></td><td>18OD33</td><td>VDDIO_SD0_SPI</td><td>G7</td><td></td></tr>
<tr><td>12</td><td>SD0_PWR_EN</td><td>XGPIOA[14] </td><td>3</td><td>PD (61Kohm)  → DRV_HI/LO + PD</td><td>Input (Trap)  → Output</td><td>V</td><td>18OD33</td><td>VDDIO_SD0_SPI</td><td>G7</td><td>1</td></tr>
</tbody>
</table>
