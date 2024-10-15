# RISC-V Hart-Level Interrupt Controller (HLIC)のためのデバイスツリーエントリ

オリジナル: `linux-sources/Documentation/devicetree/bindings/interrupt-controller/riscv,cpu-intc.txt`

RISC-Vコアには各CPUコア（RISC-V用語ではHART）にローカルなソフトウェアで
読み書きできる制御・状態レジスタ (CSR: Control Status Register)が含まれて
います。これらのCSRの中にはコアに接続されているローカル割り込みの制御に
使用されるものがあります。すべての割り込みは最終的にはハートのHLICを経由
してそのハートに割り込みます。

RISC-VスーパバイザISAマニュアルではすべてのHLICに接続される3つの割り込み
ソースを指定しています: ソフトウェア割り込み、タイマー割り込み、外部割り
込みです。ソフトウェア割り込みはコア間でIPIを送信するために使用されます。
タイマー割り込みはSBI（Supervisor Binary Interface）コールとCSR Readで
で制御されるアーキテクチャに義務付けられたリアルタイムタイマーから発生します。
外部割り込みは他のすべてのデバイス割り込みをHLICに接続し、PLC (Platform-Lebel
Interrupt Controller)を介してルーティングされます。

スーパバイザISA仕様に準拠するすべてのRISC-Vシステムにはこれら3つの割り込み
ソースを持つHLICが必要とされます。割り込みマップはISAによって定義されるため、
HLICのデバイスツリーエントリにはリストされませんが（たとえばPLICなどの）外部
割り込みコントローラはその割り込みがどのように関連するHLICにマップされるかを
定義する必要があります。これはPLICの割り込みプロパティは通常、システムにある
すべてのHARTのHLICをリストすることを意味します。

## 必須プロパティ

- **compatible**: "riscv,cpu-intc」
- **#interrupt-cells**: <1>でなければなりません。 割り込みソースはRISC-Vスーパー
  バイザISAマニュアルで定義されており、スーパーバイザモードでは以下の3つの
  割り込みだけが定義されています。
    - ソース`1`はスーパーバイザソフトウェア割り込みです。SBIコールにより
      送信することができ、ソフトウェアの使用に予約されています。
    - ソース`5`はスーパーバイザタイマー割り込みです。SBIコールにより構成する
      ことができ、ワンショットタイマーを実装しています。
    - ソース`9`はスーパーバイザ外部割り込みです。他のすべてのデバイス割り込みに
      チェーンされます。
- **interrupt-controller**: ノードを割り込みコントローラとして識別します。

さらに、この割り込みコントローラはそのCSRがこれらのローカル割り込みを制御する
HARTのCPU定義内に組み込まれていなければなりません。

HLICのデバイスツリーエントリーの例を以下に示します。

```
cpu1: cpu@1 {
                compatible = "riscv";
                ...
                cpu1-intc: interrupt-controller {
                        #interrupt-cells = <1>;
                        compatible = "sifive,fu540-c000-cpu-intc", "riscv,cpu-intc";
                        interrupt-controller;
                };
        };
```
