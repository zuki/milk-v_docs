# 3. 端子制御レジスタ

## 3.1 レジスタ一覧

<table>
<thead>
<tr><th>ピン番号</th><th>ピン名</th><th>レジスタ名</th><th>アドレス</th></tr>
</thead>
<tbody>
<tr><td>3</td><td>SD0_CLK</td><td>IOBLK_G10_REG_SD0_CLK</td><td>0x0300_1A00</td></tr>
<tr><td>4</td><td>SD0_CMD</td><td>IOBLK_G10_REG_SD0_CMD</td><td>0x0300_1A04</td></tr>
<tr><td>5</td><td>SD0_D0</td><td>IOBLK_G10_REG_SD0_D0</td><td>0x0300_1A08</td></tr>
<tr><td>7</td><td>SD0_D1</td><td>IOBLK_G10_REG_SD0_D1</td><td>0x0300_1A0c</td></tr>
<tr><td>8</td><td>SD0_D2</td><td>IOBLK_G10_REG_SD0_D2</td><td>0x0300_1A10</td></tr>
<tr><td>9</td><td>SD0_D3</td><td>IOBLK_G10_REG_SD0_D3</td><td>0x0300_1A14</td></tr>
<tr><td>11</td><td>SD0_CD</td><td>IOBLK_G7_REG_SD0_CD</td><td>0x0300_1900</td></tr>
<tr><td>12</td><td>SD0_PWR_EN</td><td>IOBLK_G7_REG_SD0_PWR_EN</td><td>0x0300_1904</td></tr>
</tbody>
</table>

## 3.2 フィールド一覧

<table>
<thead>
<tr><th>ピン番号</th><th>ピン名</th><th>フィールド名</th><th>ビット</th><th>でファルと</th><th>フィールド説明</th></tr>
</thead>
<tbody>
<tr><td>3</td><td>SD0_CLK</td><td>IOBLK_G10_REG_SD0_CLK_PU</td><td>2</td><td>0x0</td><td>Pull up enable<br/>0=disable; 1=enable</td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_CLK_PD</td><td>3</td><td>0x1</td><td>Pull down enable<br/>0=disable; 1=enable</td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_CLK_DS0</td><td>5</td><td>0x0</td><td>Output buffer Driving Strength bit 0 </td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_CLK_DS1</td><td>6</td><td>0x1</td><td>Output buffer Driving Strength bit 1 </td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_CLK_DS2</td><td>7</td><td>0x0</td><td>Output buffer Driving Strength bit 2 </td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_CLK_ST0</td><td>8</td><td>0x0</td><td>Input buffer Schmitt trigger level bit 0</td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_CLK_SL</td><td>11</td><td>0x0</td><td>Output Buffer Slew Rate Limit<br/>0=disable; 1=enable</td></tr>
<tr><td>4</td><td>SD0_CMD</td><td>IOBLK_G10_REG_SD0_CMD_PU</td><td>2</td><td>0x0</td><td>Pull up enable<br/>0=disable; 1=enable</td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_CMD_PD</td><td>3</td><td>0x1</td><td>Pull down enable<br/>0=disable; 1=enable</td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_CMD_DS0</td><td>5</td><td>0x0</td><td>Output buffer Driving Strength bit 0 </td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_CMD_DS1</td><td>6</td><td>0x1</td><td>Output buffer Driving Strength bit 1 </td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_CMD_DS2</td><td>7</td><td>0x0</td><td>Output buffer Driving Strength bit 2 </td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_CMD_ST0</td><td>8</td><td>0x0</td><td>Input buffer Schmitt trigger level bit 0</td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_CMD_SL</td><td>11</td><td>0x0</td><td>Output Buffer Slew Rate Limit<br/>0=disable; 1=enable</td></tr>
<tr><td>5</td><td>SD0_D0</td><td>IOBLK_G10_REG_SD0_D0_PU</td><td>2</td><td>0x0</td><td>Pull up enable<br/>0=disable; 1=enable</td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D0_PD</td><td>3</td><td>0x1</td><td>Pull down enable<br/>0=disable; 1=enable</td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D0_DS0</td><td>5</td><td>0x0</td><td>Output buffer Driving Strength bit 0 </td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D0_DS1</td><td>6</td><td>0x1</td><td>Output buffer Driving Strength bit 1 </td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D0_DS2</td><td>7</td><td>0x0</td><td>Output buffer Driving Strength bit 2 </td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D0_ST0</td><td>8</td><td>0x0</td><td>Input buffer Schmitt trigger level bit 0</td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D0_SL</td><td>11</td><td>0x0</td><td>Output Buffer Slew Rate Limit<br/>0=disable; 1=enable</td></tr>
<tr><td>7</td><td>SD0_D1</td><td>IOBLK_G10_REG_SD0_D1_PU</td><td>2</td><td>0x0</td><td>Pull up enable<br/>0=disable; 1=enable</td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D1_PD</td><td>3</td><td>0x1</td><td>Pull down enable<br/>0=disable; 1=enable</td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D1_DS0</td><td>5</td><td>0x0</td><td>Output buffer Driving Strength bit 0 </td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D1_DS1</td><td>6</td><td>0x1</td><td>Output buffer Driving Strength bit 1 </td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D1_DS2</td><td>7</td><td>0x0</td><td>Output buffer Driving Strength bit 2 </td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D1_ST0</td><td>8</td><td>0x0</td><td>Input buffer Schmitt trigger level bit 0</td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D1_SL</td><td>11</td><td>0x0</td><td>Output Buffer Slew Rate Limit<br/>0=disable; 1=enable</td></tr>
<tr><td>8</td><td>SD0_D2</td><td>IOBLK_G10_REG_SD0_D2_PU</td><td>2</td><td>0x0</td><td>Pull up enable<br/>0=disable; 1=enable</td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D2_PD</td><td>3</td><td>0x1</td><td>Pull down enable<br/>0=disable; 1=enable</td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D2_DS0</td><td>5</td><td>0x0</td><td>Output buffer Driving Strength bit 0 </td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D2_DS1</td><td>6</td><td>0x1</td><td>Output buffer Driving Strength bit 1 </td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D2_DS2</td><td>7</td><td>0x0</td><td>Output buffer Driving Strength bit 2 </td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D2_ST0</td><td>8</td><td>0x0</td><td>Input buffer Schmitt trigger level bit 0</td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D2_SL</td><td>11</td><td>0x0</td><td>Output Buffer Slew Rate Limit<br/>0=disable; 1=enable</td></tr>
<tr><td>9</td><td>SD0_D3</td><td>IOBLK_G10_REG_SD0_D3_PU</td><td>2</td><td>0x0</td><td>Pull up enable<br/>0=disable; 1=enable</td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D3_PD</td><td>3</td><td>0x1</td><td>Pull down enable<br/>0=disable; 1=enable</td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D3_DS0</td><td>5</td><td>0x0</td><td>Output buffer Driving Strength bit 0 </td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D3_DS1</td><td>6</td><td>0x1</td><td>Output buffer Driving Strength bit 1 </td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D3_DS2</td><td>7</td><td>0x0</td><td>Output buffer Driving Strength bit 2 </td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D3_ST0</td><td>8</td><td>0x0</td><td>Input buffer Schmitt trigger level bit 0</td></tr>
<tr><td></td><td></td><td>IOBLK_G10_REG_SD0_D3_SL</td><td>11</td><td>0x0</td><td>Output Buffer Slew Rate Limit<br/>0=disable; 1=enable</td></tr>
<tr><td>11</td><td>SD0_CD</td><td>IOBLK_G7_REG_SD0_CD_PU</td><td>2</td><td>0x1</td><td>Pull up enable<br/>0=disable; 1=enable</td></tr>
<tr><td></td><td></td><td>IOBLK_G7_REG_SD0_CD_PD</td><td>3</td><td>0x0</td><td>Pull down enable<br/>0=disable; 1=enable</td></tr>
<tr><td></td><td></td><td>IOBLK_G7_REG_SD0_CD_DS0</td><td>5</td><td>0x0</td><td>Output buffer Driving Strength bit 0 </td></tr>
<tr><td></td><td></td><td>IOBLK_G7_REG_SD0_CD_DS1</td><td>6</td><td>0x1</td><td>Output buffer Driving Strength bit 1 </td></tr>
<tr><td></td><td></td><td>IOBLK_G7_REG_SD0_CD_DS2</td><td>7</td><td>0x0</td><td>Output buffer Driving Strength bit 2 </td></tr>
<tr><td></td><td></td><td>IOBLK_G7_REG_SD0_CD_ST0</td><td>8</td><td>0x0</td><td>Input buffer Schmitt trigger level bit 0</td></tr>
<tr><td></td><td></td><td>IOBLK_G7_REG_SD0_CD_SL</td><td>11</td><td>0x0</td><td>Output Buffer Slew Rate Limit<br/>0=disable; 1=enable</td></tr>
<tr><td>12</td><td>SD0_PWR_EN</td><td>IOBLK_G7_REG_SD0_PWR_EN_PU</td><td>2</td><td>0x0</td><td>Pull up enable<br/>0=disable; 1=enable</td></tr>
<tr><td></td><td></td><td>IOBLK_G7_REG_SD0_PWR_EN_PD</td><td>3</td><td>0x1</td><td>Pull down enable<br/>0=disable; 1=enable</td></tr>
<tr><td></td><td></td><td>IOBLK_G7_REG_SD0_PWR_EN_DS0</td><td>5</td><td>0x0</td><td>Output buffer Driving Strength bit 0 </td></tr>
<tr><td></td><td></td><td>IOBLK_G7_REG_SD0_PWR_EN_DS1</td><td>6</td><td>0x1</td><td>Output buffer Driving Strength bit 1 </td></tr>
<tr><td></td><td></td><td>IOBLK_G7_REG_SD0_PWR_EN_DS2</td><td>7</td><td>0x0</td><td>Output buffer Driving Strength bit 2 </td></tr>
<tr><td></td><td></td><td>IOBLK_G7_REG_SD0_PWR_EN_ST0</td><td>8</td><td>0x0</td><td>Input buffer Schmitt trigger level bit 0</td></tr>
<tr><td></td><td></td><td>IOBLK_G7_REG_SD0_PWR_EN_SL</td><td>11</td><td>0x0</td><td>Output Buffer Slew Rate Limit<br/>0=disable; 1=enable</td></tr>
</tbody>
</table>
