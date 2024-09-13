# 3.7 タイマー

## 3.7.1 概要

システムは8つのタイマーモジュールを備えています。これは計時や
計数機能として使用することができ、アプリケーションプログラムに
計時/計数機能を提供したり、オペレーティングシステムにシステム
クロックの実装を提供したりすることができます。

## 3.7.2 特徴

タイマーには次の特徴があります。

- 32ビットのカウントダウンタイマー/カウンタ
- 2つの計数モード: フリーランニングモードとユーザ定義の計数モードの
  サポート
- カウンタの現在値を読み取り可能

カウンタ値が0にカウントダウンすると割り込みが発生します。

## 3.7.3 機能説明

タイマーは32ビットのカウントダウンカウンタに基づいています。カウンタの値は
計数クロックの立ち上がりエッジごとに1ずつ減算されます。カウンタがゼロに
カウントダウンされるとタイマーは割り込みを発生します。

タイマーには次の2つの計数モードがあります：

1. フリーランニングモード
    タイマーは連続的にカウントします。カウンタ値が0になると自動的に
    最大値に戻り、カウントダウンを続けます。カウンタ値の最大初期値は
    `0xFFFF_FFFF`です。
2. ユーザー定義の計数モード
    タイマーはカウントダウンを続けます。カウンタ値が0になると
    `TimerNLoadCount (N = 1-8)`レジスタからカウンタ初期値をリロードし、
    カウントダウンを続けます。

カウンタの初期値をタイマーにロードする方法は次の通りです。

- `TimerNLoadCount (N = 1-8)`レジスタに書き込むことでタイマーのカウンタ
初期値をロードすることができます。

## 3.7.4 操作モード

### 3.7.4.1 初期化

1. `TimerNLoadCount (N = 1-8)`レジスタにタイマーのカウンタ初期値を書き込んで
   ロードします。
2. `TimerNControlReg [2:0] (N = 1-8)`レジスタを構成して、タイマーの計数モードの
   選択、タイマー割り込みのマスク、タイマーを有効にしてカウントダウンを
   開始します。

### 3.7.4.2 割り込み処理

タイマーが割り込みを発生させた際の操作手順は次のとおりです。

1. `TimerNEOI (N = 1-8)`レジスタを読み取り、タイマーN割り込みをクリアします。
2. 割り込みを待っていたプロセスを実行します。
3. 実行が完了したら、割り込みされたプログラムは再開されます。

### 3.7.4.3 クロックの選択

システムタイマーは計数に25MHzまたは32KHzのクロックを使用することができます。
選択には`reg_timer_clk_sel`レジスタを使います。

## 3.7.5 タイマーレジスタの概要

タイマーレジスタはバスを通じてアクセスされます。タイマーレジスタの概要を
表3-7に示します。

**基底アドレス: 0x030A_0000**

<center>表3-7: タイマーレジスタ</center>

| 名前 | オフセット | 記述 |
|:---- |:-----------|:-----|
| Timer1LoadCount | 0x000 | Value to be loaded into Timer1 |
| Timer1CurrentValue | 0x004 | Current Value of Timer1 |
| Timer1ControlReg | 0x008 | Control Register for Timer1 |
| Timer1EOI | 0x00c | Clears the interrupt from Timer1 |
| Timer1IntStatus | 0x010 | Contains the interrupt status for Timer1 |
| Timer2 | 0x014-0x024 | Timer1と同じレジスタが5つある |
| Timer3 | 0x028-0x038 | Timer1と同じレジスタが5つある |
| Timer4 | 0x03c-0x04c | Timer1と同じレジスタが5つある |
| Timer5 | 0x050-0x060 | Timer1と同じレジスタが5つある |
| Timer6 | 0x064-0x074 | Timer1と同じレジスタが5つある |
| Timer7 | 0x078-0x088 | Timer1と同じレジスタが5つある |
| Timer8 | 0x08c-0x09c | Timer1と同じレジスタが5つある |
| TimersIntStatus | 0x0a0 | Contains | the interrupt status of all timers in the component. |
| TimersEOI | 0x0a4 | Returns | all zeroes (0) and clears all active interrupts. |
| TimersRawIntStatus | 0x0a8 | Contains | the unmasked interrupt status of all timers in the component. |
