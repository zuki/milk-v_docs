# RISC-V Platform-Level Interrupt Controller (PLIC)

**未定稿** [オリジナル](https://patchwork.kernel.org/project/linux-kbuild/patch/20170912215715.4186-5-palmer@dabbelt.com/)

RISC-VスーパバイザISA仕様ではPLIC（プラットフォームレベルの割り込みコントローラ）の
存在を許しています。PLICは各ハートのHLIC（ハートローカルの割り込みコントローラ）の
外部割り込みソースを介してシステムのすべての外部割り込みをシステムのすべてのハート
コンテキストに接続します。ハートコンテキストはハードウェア実行スレッドの特権モードです。
たとえば、2ウェイSMTの4コアシステムでは、8つのハートがあり、おそらく1つのハートにつき
少なくとも2つの特権モード （マシンモードとスーパバイザモード）があります。

各割り込みはコンテキストごとに有効にできます。どのコンテキストも保留中の有効な割り込みを
要求し、それを処理して解放することができます。

各割り込みには構成可能な優先度があります。優先順位の高い割り込みが最初に処理されます。
各コンテキストは優先度のしきい値を指定できます。このしきい値以下の優先順位の割り込みは
PLICがそのコンテキストにつながる割り込みラインを上げる原因になりません。

PLICはエッジトリガ割り込みとレベルトリガ割り込みを共にサポートしていますが、割り込み
ハンドラはこれを区別しないのでPLICのデバイスツリーバインディングでは指定されていません。

RISC-V ISAはPLICのメモリレイアウトを規定していませんが”riscv,plic0"デバイスは特定の
メモリレイアウトを持つPLICの具体的な実装です。"riscv,plic0"デバイスのメモリレイアウトに
関する詳細はデバイスドライバのコメントで見ることできます。また、
[SiFive U5 Coreplexシリーズマニュアル](https://www.sifive.com/documentation/coreplex/u5-coreplex-series-manual/)（バージョン1.0のPDF文書の22ページ）に文書化されています。

## 必須の属性

- compatible : "riscv,plic0"
- #address-cells : <0>である必要があります
- #interrupt-cells : <1>である必要があります
- interrupt-controller : ノードを割り込みコントローとして認識します
- reg : レジスタ範囲（アドレスとサイズ）を1つ含む必要があります
- interrupts-extended : PLICに接続されているコンテキストを指定します。
      "-1"はコンテキストが存在しないことを示します。

## 指定例

```
plic: interrupt-controller@c000000 {
    #address-cells = <0>;
    #interrupt-cells = <1>;
    compatible = "riscv,plic0";
    interrupt-controller;
    interrupts-extended = <
        &cpu0-intc 11
        &cpu1-intc 11 &cpu1-intc 9
        &cpu2-intc 11 &cpu2-intc 9
        &cpu3-intc 11 &cpu3-intc 9
        &cpu4-intc 11 &cpu4-intc 9>;
    reg = <0xc000000 0x4000000>;
    riscv,ndev = <10>;
};
```

## milkvの例

```
interrupt-controller@70000000 {
    riscv,ndev = <0x65>;
    riscv,max-priority = <0x7>;
    reg-names = "control";
    reg = <0x0 0x70000000 0x0 0x4000000>;
    interrupts-extended = <0x12 0xffffffff 0x12 0x9>;
    interrupt-controller;
    compatible = "riscv,plic0";
    #interrupt-cells = <0x2>;
    #address-cells = <0x0>;
    phandle = <0x4>;
};
```
