# 3.9 リアルタイムクロック

## 3.9.1 概要

リアルタイムクロック (RTC) はチップ内の独立したパワードメインブロックです。
32KHzの発振器とパワーオンリセット (POR) モジュールを内蔵しており、日付
クロック表示とアラームクロック生成に使用できます。さらに、内部にある
ハードウェア有限状態マシンはチップのパワーオン、パワーオフ、システムリセットを
トリガーするためのタイミングシーケンス制御を提供します。

## 3.9.2 機能

RTCの特徴は以下の通りです。

- システムリセットソースを提供
- 32768Hzのクロックソースを提供（ミスマッチ < ±1）
- 32ビットの秒カウンタとハードウェア較正回路を提供。5ppmの秒精度を実現
- アラームクロックの構成をサポートし、アラーム割り込みを発生
- ソフトウェアコードまたは一時的なデータの保存用に2KBのSRAM空庵を提供
- バッテリ電圧の低下を検出し、割り込みを発生
- ボタン操作によるチップのスリープ解除をサポート
- ソフトウェアによるチップのスリープ、リセット、過熱時再起動のトリガをサポート
- ウォッチドッグによるチップのシステムリセットをトリガ
- アラームトリガによるチップのスリープ解除をサポート
- パワーアップ/ダウンのタイミング、リセット時間間隔を設定可能
- 超低消費電力のアナログクロックカウンタ（32ビット、1970年～2106年をカウント）を
  1つ搭載

## 3.9.3 機能説明

RTCは常時オンのパワードメインモジュールです。RTCに初めて電源が投入されるとPOR回路が
ローレベルのパルスを発生し、32KHz発振器が振動を開始します。PORがハイレベルになると
RTCは初期状態に入り、イベントトリガーを待ちます。

ステートマシンはバッテリー電圧が正常な状態であることを検出すると、デフォルトの
タイミングに従ってチップのパワーオンプロセスを完了し、システムリセット信号をリリース
します。ソフトウェアは初回起動後にRTCを初期化し、初期カウント値を設定する必要があります。
システムのシャットダウンやスリープモードへの移行が必要な場合、RTCステートマシンは
システム制御レジスタ (`RTC_CTRL`) で構成した構成タイミングに基づいてチップ電源を切断する
プロセスを完了させるトリガーを発することができます。

チップがパワーオフまたはスリープモードの状態にあっても、RTCは動作し続け、必要な
ソフトウェアコードやユーザデータをRTC SRAMと情報レジスタ (`RTC_INFO0-3`,
`RTC_NOPOR_INFO0-3`) に保存することができます。秒カウンタはカウントを続け、
同時にステートマシンはチップのパワーオンまたはスリープからのウェイクアップのトリガーと
なるキーイベントを検出し続けます。システムが再開または再起動した後、ソフトウェアは
RTCステータスレジスタまたは以前に情報レジスタに書き込んだ内容を読み戻すことにより
チップの状態を判断することができます。2つのステータスレジスタ (`RTC_ST_ON_REASON`,
`RTC_ST_OFF_REASON`) がチップの最新のパワーダウン、パワーアップ、リセットのトリガー
条件を記録するために提供されています。また、強制リセット、チップ過熱（サーマルシャット
ダウン）、バッテリー低下、電源低下などの予期せぬイベントが発生したか否かかなどの
より詳細な情報を提供することもできます。さらに、RTCはウォッチドッグイベントを受信すると
システムリセットを発生させます。

RTC秒カウンタは32KHzクロックで計数され、32ビット加算器に基づいています。カウンタの
初期値はレジスタ`RTC_SET_SEC_CNTR_VALUE`でロードできます。秒の値は特定の年、月、日、時、
分に変換できます。

32KHzクロックと秒パルス周期はソフトウェアプロセスで較正することも、ハードウェア
モジュールを有効にして定期的な自動較正を実行することもできます。

ソフトウェアは32ビットレジスタ`RTC_ALARM_TIME`を構成し、レジスタビット`RTC_ALARM_ENABLE`に
1をセットしてアラーム機能を有効にすることにより、アラーム時間を指定することができます。
秒カウンタ値`RTC_SEC_CNTR_VALUE`が`RTC_ALARM_TIME`と等しくなるとアラーム割り込みが
発生します。割り込み状態は`RTC_ALARM_ENABLE`に0がセットされるまで保持されます。

さらに、RTCはバッテリ低電圧検出機能を備えています。バッテリ電圧が一定レベルより低く
なると割り込みが発生します。異常なエラーの発生を防ぐために、ソフトウエアは割り込みを
受信したら直ちにシャットダウン手順を実行し、RTCをトリガーしてパワーダウンプロセスを
完了させることができます。

## 3.9.4 操作

### 3.9.4.1 計数クロック

RTC秒カウンタの最大計数時間は次の通りです。

    2^32 = 49710日 ＝136年

#### 3.9.4.2 RTCのリセット

RTCはチップのパワーオン/パワーオフ制御ユニットとして機能し、ソフトウェアにより個別に
リセットすることはできません。初回電源投入時のPORを除き、例外が発生した際にはRSTN
ボタンによりチップ全体（RTCを含む）を強制的にリセットすることができます。RSTNボタンを
離すと、すべてのRTCレジスタはデフォルト値に戻り、ステートマシンは初期状態に戻ります。
ステートマシンはバッテリ電圧が正常状態であることを検出すると、チップのパワーオン
プロセスを開始し、デフォルトのタイミングに従ってシステムリセット信号をリリースします。

#### 3.9.4.3 RTCの初期化

チップの初回電源投入後、システムはRTCを初期化する必要があります。まず、32KHzクロックと
秒計時周期を較正する必要があります。較正回路は25MHzの水晶クロックを使用して
32KHzクロックをサンプリングします。粗調整モードでは25MHzの水晶クロックが32KHzクロックの
1周期をサンプリングして計数結果を報告します。ソフトウェアはこの計数結果に応じて構成
レジスタ`RTC_ANA_CALIB[8:0]`を調整して32KHzクロックレートを高速または低速にして32KHz
クロックの精度を向上させます。粗調整の完了後に微調整を行うことができます。デフォルトでは
25MHzの水晶クロックを使用して32KHzクロックを256回サンプリングします。ソフトウェアはこの
計数結果に応じて平均値を計算し、32KHzクロックで1秒を計数するのに必要なパルス数を得ます。
この平均値をレジスタ`RTC_SEC_PULSE_GEN_INT`と`RTC_SEC_PULSE_GEN_FRAC`に書き込むことで
秒較正が完了します。

粗調整較正のプロセスは以下のように要約されます。

1. レジスタ`RTC_ANA_SEL_FTUNE`に0をセットし、`RTC_ANA_CALIB`の初期値を`0x100`に設定します。
2. 次のようにバイナリサーチを用いて較正を実施します。

    FTUNE = RTC_ANA_CALIB; offset = 0x80

3. `RTC_FC_COARSE_EN`に1をセットし、粗調整を開始します。`RTC_FC_COARSE_TIME`の値が前回の
   読み取り値より大きくなるまでポーリングし、その後、`RTC_FC_COARSE_EN`に0をセットします。
4. `RTC_FC_COARSE_VALUE`を読み取り、25MHzクロックでサンプリングされた32KHzクロックの
   1サイクルのカウントを取得します。
    ```c
    if (RTC_FC_COARSE_VALUE > 770) FTUNE = FTUNE + offset;
    if (RTC_FC_COARSE_VALUE < 755) FTUNE = FTUNE - offset;
    FTUNE値をRTC_ANA_CALIBレジスタに書き戻します。
    offset = offset >> 1;
    ```

5. `RTC_FC_COARSE_VALUE`の値が755-770の間であれば、32KHzクロックの精度は32,768Hzの
   ±1%以内に達しており、粗調整は完了です。そうでない場合は、0.5ms待ってからステップ3～5を
   最大8回まで繰り返します。

微調整較正のプロセスは以下のように要約されます。

1. `RTC_SEL_SEC_PULSE`に0をセットします。さらに、`RTC_FC_FINE_EN`に1をセットし、
   微調整を開始します。
2. `RTC_FC_FINE_TIME`の値が前回の読み取り値より大きくなるまでポーリングします。
3. `RTC_FC_FINE_VALUE`を読み取り、25MHzクロックでサンプリングされた32KHzクロックの
   256サイクルのカウントを取得します。
4. 32KHzクロックの周波数は以下の式から求めることができます。
    ```c
    Frequency = 256 / (RTC_FC_FINE_VALUE x 40ns)

    例: 256 / (195310 x 40) = 32768.4194357
    ```

5. 整数部32768を取り、レジスタ`RTC_SEC_PULSE_GEN_INT`に書き込みます。そして、
   少数部分の 8-bit = 0.4194357 x 256 = 107 を取り、レジスタ`RTC_SEC_PULSE_GEN_FRAC`に
   書き込みます。
6. `RTC_FC_FINE_EN`に0をセットして微調整を終了します。

クロック較正プロセスはシステムのニーズに応じてソフトウェアで1回だけ実行することも、
定期的に実行することもできます。ソフトウェア較正プロセスに加えて、RTCはハードウェアに
よって定期的に実行される自動較正もサポートしています。

クロックを較正した後、さらにRTCを初期化する必要があります。必要な初期化のみを以下に
示します。その他のパラメータレジスタのほとんどは電源シーケンスのタイミング間隔を最適化
する必要がある場合にのみ設定する必要があります。それ以外の場合は一般にデフォルト値を
使用することを推奨します。

1. `RTC_POR_DB_MAGIC_KEY`レジスタの値に`0x5AF0`をセットしてパワーオンリセット（POR）
   デバウンスを有効にし、RTCモジュール電源の短時間の電圧降下によるPORの誤トリガを
   防止します。デバウンス時間は約1msです。
2. `RTC_SET_SEC_CNTR_VALUE`レジスタをセットしてRTC計時カウンタを初期化します。
3. `RTC_SET_SEC_CNTR_TRIG`に1を書き込み、初期カウンタ値をRTC秒カウンタにロードします。
4. 読み取り値が`RTC_SET_SEC_CNTR_VALUE`の値と等しくなるまで`RTC_SEC_CNTR_VALUE`レジスタを
   ポーリングします。
5. `RTC_PWR_DET_COMP[0]`に1をセットしてバッテリ低電圧検出を有効にし、`RTC_PWR_DET_SEL[0]`
   に1をセットしてバッテリ電圧が閾値を下回ったときに低電圧検出割り込みを生成するように
   します。閾値は`RTC_PWR_DET_COMP[12:8]`レジスタを構成することで調整できます。
6. 初回電源投入後、チップの電源切断（停止またはパワーダウン）後もRTCサブシス テムの動作
   状態を維持するように`reg_rtcsys_rstn_src_sel`レジスタをデフォルト値の0から1に構成する
   ことにより、RTCサブシステムを構成する必要があります。このレジスタに0をセットすると
   チップのパワーダウン時にRTCサブシステムがリセットされます。
7. 初回電源投入後、`RTC_EN_AUTO_POWER_UP`レジスタをデフォルト値の1から0に構成する必要が
   あります。デフォルト値のままだとチップがパワーダウン状態（パワーダウン）になり、`PWR_VBAT_DET`がhighと検出されるとRTCは自動的に電源投入されます。

#### 3.9.4.4 アナログ秒クロックの初期化

1. `RTC_MACRO_RG_DEFD`に16'hC80 (32000 32KHz clock cycles)をセットします
2. `RTC_MACRO_DA_SOC_READY`に1をセットします
3. `RTC_MACRO_DA_CLEAR_ALL`に1をセットします
4. `RTC_MACRO_DA_CLEAR_ALL`に0をセットします
5. `RTC_MACRO_RG_SET_T`に必要なカウンタ値をセットします
6. `RTC_MACRO_DA_LATCH_PASS`に1をセットします
7. `RTC_MACRO_DA_LATCH_PASS`に0をセットします
8. `RTC_MACRO_DA_SOC_READY`に0をセットします
9. `RTC_MACRO_RO_T`を読み取りカウンタ値を取得します

#### 3.9.4.5 割り込み処理

RTCはアラーム割り込みと低電圧割り込みを発行することができます。アラーム割り込みを
受信したらレジスタビット`RTC_ALARM_ENABLE`に0をセットしてアラームを無効にし、割り込み
状態をクリアします。新しいアラーム時間が必要な場合はレジスタ`RTC_ALARM_TIME`に新しい
値を指定して`RTC_ALARM_ENABLE`に再度1をセットします。

#### 3.9.4.6 サスペンドと起床

`RTC_DB_REQ_SUSPEND`レジスタの`req_suspend`ビットに1をセットするとチップをサスペンド/
スリープモードにするトリガーとなります。`RTC_EN_PWR_WAKEUP`レジスタに指定することにより
チップに起床をトリガーするソースを選択することができます。DDRインタフェースの誤動作を
防止し、チップのパワーダウンまたはパワーアップ期間中にDDRコンテンツを保護するために、`req_suspend`をセットする前に`RTC_PG_REG`レジスタに0をセットしてDDR IO状態を保持する
必要があることに注意してください。チップ再開後、DDRにアクセスする前に`RTC_PG_REG`
レジスタに1を書き込んでDDR IO保持を解除します。

#### 3.9.4.7 パワーオンとパワーオフ

`req_shdn`レジスタに1をセットすることにより、システムソフトウェアはDDRを含むチップを
電源オフ状態（パワーオフ）にすることができます。`RTC_EN_PWR_UP`レジスタを設定することで
チップのパワーアップと起動(powerup)をトリガーするソースを選択することができます。

## 3.9.5 RTCレジスタ概要

RTCレジスタは複数の部分: `RTC_CORE`, `RTC_MACRO`, `RTC_CTRL`で構成されて
おり、それぞれ異なるベースアドレスを持ちます。すべてのレジスタはバスを介してアクセス
されます。

`RTC_CORE`レジスタの概要を表3.9に示します。

表3.9 `RTC_CORE`レジスタの概要（ベースアドレス: `0x0502_6000`）

| レジスタ名 | アドレス<br/>オフセット | 説明 |
|:-----------|-------------------------|:-----|
| RTC_ANA_CALIB | 0x000 |  32KHz発振器制御 |
| RTC_SEC_PULSE_GEN | 0x004 | 秒パルスジェネレータの整数/小数点以下の桁数 |
| RTC_ALARM_TIME | 0x008 | アラーム時刻の設定 |
| RTC_ALARM_ENABLE | 0x00c | アラームを有効にする |
| RTC_SET_SEC_CNTR_VALUE | 0x010 | 秒カウンタ値の設定 |
| RTC_SET_SEC_CNTR_TRIG | 0x014 | 秒カウンタの値を読み込む |
| RTC_SEC_CNTR_VALUE | 0x018 | 現在の秒カウンタ値を読み込む |
| RTC_INFO0 | 0x01c | 情報レジスタ 0 |
| RTC_INFO1 | 0x020 | 情報レジスタ 1 |
| RTC_INFO2 | 0x024 | 情報レジスタ 2 |
| RTC_INFO3 | 0x028 | 情報レジスタ 3 |
| RTC_NOPOR_INFO0 | 0x02c | リセットなし情報レジスタ0 |
| RTC_NOPOR_INFO1 | 0x030 | リセットなし情報レジスタ1 |
| RTC_NOPOR_INFO2 | 0x034 | リセットなし情報レジスタ2 |
| RTC_NOPOR_INFO3 | 0x038 | リセットなし情報レジスタ3 |
| RTC_DB_PWR_VBAT_DET | 0x040 | PWR_VBAT_DETジッター除去時間 |
| RTC_DB_BUTTON1 | 0x048 | PWR_BUTTON1 ジッター除去時間 |
| RTC_DB_PWR_ON | 0x04c | PWR_ON ジッター除去時間 |
| RTC_7SEC_RESET | 0x050 | PWR_BUTTON長押しで強制リセットする秒数を設定 |
| RTC_THM_SHDN_AUTO_REBOOT | 0x064 | REQ_THM_SHDNアクションを選択 |
| RTC_POR_DB_MAGIC_KEY | 0x068 | PORをイネーブル 長時間デジッタリング |
| RTC_DB_SEL_PWR | 0x06c | PWR_BUTTONディジッターモードの選択 |
| RTC_UP_SEQ0 | 0x070 | 電源投入時 PWR_SEQ0 出力タイミング |
| RTC_UP_SEQ1 | 0x074 | 電源投入時 PWR_SEQ1 出力タイミング |
| RTC_UP_SEQ2 | 0x078 | 電源投入時 PWR_SEQ2 出力タイミング |
| RTC_UP_SEQ3 | 0x07c | 電源投入時 PWR_SEQ3 出力タイミング |
| RTC_UP_IF_EN | 0x080 | 電源投入時ISO解除タイミング |
| RTC_UP_RSTN | 0x084 | 電源投入時システムリセット解除タイミング |
| RTC_UP_MAX | 0x088 | 電源投入時処理完了タイミング |
| RTC_DN_SEQ0 | 0x090 | 電源切断時 PWR_SEQ0 出力タイミング |
| RTC_DN_SEQ1 | 0x094 | 電源切断時 PWR_SEQ1 出力タイミング |
| RTC_DN_SEQ2 | 0x098 | 電源切断時 PWR_SEQ2 出力タイミング |
| RTC_DN_SEQ3 | 0x09c | 電源切断時 PWR_SEQ3 出力タイミング |
| RTC_DN_IF_EN | 0x0a0 | パワーダウンISOターンオンのタイミング |
| RTC_DN_RSTN | 0x0a4 | パワーダウン・システムのリセット発行タイミング |
| RTC_DN_MAX | 0x0a8 | パワーダウンプロセス完了タイミング |
| RTC_PWR_CYC_MAX | 0x0b0 | パワーサイクル完了タイミング |
| RTC_WARM_RST_MAX | 0x0b4 | Warm-reset完了タイミング |
| RTC_EN_7SEC_RST | 0x0b8 | PWR_BUTTON1 7SECリセットモードの設定 |
| RTC_EN_PWR_WAKEUP | 0x0bc | スリープ解除ソースの設定 |
| RTC_EN_SHDN_REQ | 0x0c0 | REQ_SHDNをイネーブル |
| RTC_EN_THM_SHDN | 0x0c4 | REQ_THM_SHDNをイネーブル |
| RTC_EN_PWR_CYC_REQ | 0x0c8 | REQ_PWR_CYCをイネーブル |
| RTC_EN_WARM_RST_REQ | 0x0cc | REQ_WARM_RSTをイネーブル |
| RTC_EN_PWR_VBAT_DET | 0x0d0 | PWR_VBAT_DETステートマシンリファレンスをイネーブル |
| FSM_STATE | 0x0d4 | RTCステートマシン値 |
| RTC_EN_WDG_RST_REQ | 0x0e0 | REQ_WDG_RSTをイネーブル |
| RTC_EN_SUSPEND_REQ | 0x0e4 | REQ_SUSPENDをイネーブル |
| RTC_DB_REQ_WDG_RST | 0x0e8 | REQ_WDG_RST デジッタ時間 |
| RTC_DB_REQ_SUSPEND | 0x0ec | REQ_SUSPEND デジッタ時間 |
| RTC_PG_REG | 0x0f0 | Power Goodレジスタ |
| RTC_ST_ON_REASON | 0x0f8 | パワーアップステータスレジスタ |
| RTC_ST_OFF_REASON | 0x0fc | パワーオフステータスレジスタ |
| RTC_EN_WAKEUP_REQ | 0x120 | REQ_WAKEUPをイネーブル |
| RTC_PWR_WAKEUP_POLARITY | 0x128 | PWR_WAKEUP Lowを選択 |
| RTC_DB_SEL_REQ | 0x130 | ジッター除去モードの選択 |
| RTC_PWR_DET_SEL | 0x140 | 低電圧検出信号源の選択 |

`RTC_MACRO`レジスタの概要を表3.10に示します。

表3.10 `RTC_MACRO`レジスタの概要（ベースアドレス: `0x0502_6400`）

| レジスタ名 | アドレス<br/>オフセット | 説明 |
|:-----------|-------------------------|:-----|
| RTC_PWR_DET_COMP | 0x044 | 低電圧検出制御 |
| RTC_MACRO_DA_CLEAR_ALL | 0x080 | DA_CLEAR_ALL |
| RTC_MACRO_DA_SET_ALL | 0x084 | DA_SEL_ALL |
| RTC_MACRO_DA_LATCH_PASS | 0x088 | DA_LATCH_PASS |
| RTC_MACRO_DA_SOC_READY | 0x08c | DA_SOC_READY |
| RTC_MACRO_PD_SLDO | 0x090 | PD_SLDO |
| RTC_MACRO_RG_DEFD | 0x094 | RG_DEFD |
| RTC_MACRO_RG_SET_T | 0x098 | RG_SET_T |
| RTC_MACRO_RO_CLK_STOP | 0x0a0 | RO_CLK_STOP |
| RTC_MACRO_RO_DEFQ | 0x0a4 | RO_DEFQ |
| RTC_MACRO_RO_T | 0x0a8 | RO_T |

`RTC_CTRL`レジスタの概要を表3.11に示します。

表3.11 `RTC_CTRL`レジスタの概要（ベースアドレス: `0x05025000`）

訳注: ベースアドレスはデータシートでは`0x0300_4000`とあるが、u-bootのincludeファイルでは
`0x05025000`となっている。また、データシートの「1.5 アドレス空間マッピング」でも
`RTCSYS_CTRL`のベースアドレスは`0x05025000`であり、`0x0300_4000`は予約領域となっている。
実際に動かしたところ`0x05025000`が正しかった。

| レジスタ名 | アドレス<br/>オフセット | 説明 |
|:-----------|-------------------------|:-----|
| rtc_ctrl_unlockkey | 0x004 | rtc_ctrl_unlockkey |
| rtc_ctrl0 | 0x008 | rtc_ctrl0 |
| rtc_ctrl_status0 | 0x00c | rtc_ctrl_status0 |
| rtc_ctrl_status1 | 0x010 | rtc_ctrl_status1 |
| rtc_ctrl_status2gpio | 0x014 | rtc_ctrl_status2gpio |
| rtcsys_rst_ctrl | 0x018 | rtcsys_rst_ctrl |
| rtcsys_clkmux | 0x01c | rtcsys_clkmux |
| rtcsys_mcu51_ctrl0 | 0x020 | rtcsys_mcu51_ctrl0 |
| rtcsys_mcu51_ctrl1 | 0x024 | rtcsys_mcu51_ctrl1 |
| rtcsys_pmu | 0x028 | rtcsys_pmu |
| rtcsys_status | 0x02c | rtcsys_status |
| rtcsys_clkbyp | 0x030 | rtcsys_clkbyp |
| rtcsys_clk_en | 0x034 | rtcsys_clk_en |
| rtcsys_wkup_ctrl | 0x038 | rtcsys_wkup_ctrl |
| rtcsys_clkdiv | 0x03c | rtcsys_clkdiv |
| fc_coarse_en | 0x040 | fc_coarse_en |
| fc_coarse_cal | 0x044 | fc_coarse_cal |
| fc_fine_en | 0x048 | fc_fine_en |
| fc_fine_period | 0x04c | fc_fine_period |
| fc_fine_cal | 0x050 | fc_fine_cal |
| rtcsys_pmu2 | 0x054 | rtcsys_pmu2 |
| rtcsys_clkdiv1 | 0x058 | rtcsys_clkdiv1 |
| rtcsys_mcu51_dbg | 0x05c | rtcsys_mcu51_dbg |
| sw_reg0 | 0x060 | sw_reg0 |
| sw_reg1_por | 0x064 | sw_reg1_por |
| fab_lp_ctrl | 0x068 | fab_lp_ctrl |
| fab_option | 0x06c | fab_option |
| rtcsys_mcu51_ictrl1 | 0x07c | rtcsys_mcu51_ictrl1 |
| rtc_ip_pwr_req | 0x080 | rtc_ip_pwr_req |
| rtc_ip_iso_ctrl | 0x084 | rtc_ip_iso_ctrl |
| rtcsys_spare_reg0 | 0x088 | rtcsys_spare_reg0 |
| rtcsys_spare_reg1 | 0x08c | rtcsys_spare_reg1 |
| rtcsys_spare_ro | 0x090 | rtcsys_spare_ro |
| rtcsys_wkup_ctrl1 | 0x094 | rtcsys_wkup_ctrl1 |
| rtcsys_sram_ctrl | 0x098 | rtcsys_sram_ctrl |
| rtcsys_io_ctrl | 0x09c | rtcsys_io_ctrl |
| rtcsys_wdt_ctrl | 0x0a0 | rtcsys_wdt_ctrl |
| rtcsys_irrx_clk_ctrl | 0x0a4 | rtcsys_irrx_clk_ctrl |
| rtcsys_rtc_wkup_ctrl | 0x0a8 | rtcsys_rtc_wkup_ctrl |
| rtcsys_por_rst_ctrl | 0x0ac | rtcsys_por_rst_ctrl |
