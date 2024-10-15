# デバイス割り込み情報の指定

## 1) 割り込みクライアントノード

割り込みを発生させるデバイスを記述するノードは"`interrupts`"プロパティ、
"`interrupts-extended`"プロパティ、またはその両方を持たなければなりません。
両方が存在する場合は後者が優先されます。前者は単に後者を認識しないソフトウェアとの
互換性のために提供される場合があります。これらのプロパティには割り込み指定子の
リストが含まれ、出力する割り込みにつき1つ指定されます。割り込み指定子の形式は
割り込みがルーティングされる割り込みコントローラによって決まります。詳細は以下の
第2項を参照してください。

```
  指定例:
	interrupt-parent = <&intc1>;
	interrupts = <5 0>, <6 0>;
```

"`interrupt-parent`"プロパティは割り込みがルーティングされるコントローラの指定に
使用され、割り込みコントローラノードを参照するphandleを1つ含みます。このプロパティは
継承されるので割り込みクライアントノードやその親ノードで指定することができます。
"`interrupts`"プロパティにリストされている割り込みは常にそのノードの割り込み親ノードを
参照しています。

"`interrupts-extended`"プロパティは特別な形式です。ノードが複数の割り込み親を参照する
必要がある場合や継承された割り込み親とは異なる割り込み親を参照する必要がある場合に
便利です。このプロパティの各エントリには親のphandleと割り込み指定子を含みます。

```
  指定例:
	interrupts-extended = <&intc1 5 1>, <&intc2 1 0>;
```
## 2) 割り込みコントローラノード

デバイスは"`interrupt-controller`"プロパティで割り込みコントローラとしてマークされます。
これは空のブーリアンプロパティです。追加の"`#interrupt-cells`"プロパティは1つの割り込みを
指定するために必要なセル数を定義します。

割り込み指定子の長さと形式を定義するのは割り込みコントローラのバインディングの責任です。
次の2つの形式が通常使用されています。

### a) 1セル

"`#interrupt-cells`"プロパティは1にセットされており、シングルセルはコントローラ内の
割り込みのインデックスを定義します。

```
  指定例:

	vic: intc@10140000 {
		compatible = "arm,versatile-vic";
		interrupt-controller;
		#interrupt-cells = <1>;
		reg = <0x10140000 0x1000>;
	};

	sic: intc@10003000 {
		compatible = "arm,versatile-sic";
		interrupt-controller;
		#interrupt-cells = <1>;
		reg = <0x10003000 0x1000>;
		interrupt-parent = <&vic>;
		interrupts = <31>; /* Cascaded to vic */
	};
```

### b) 2セル

"`#interrupt-cells`"プロパティは2にセットされており、最初のセルはコントローラ内の
割り込みのインデックスを定義し、2番目のセルは次のフラグのいずれかを指定するために
使用されます。

```
	1 = low-to-highのエッジトリガ
	2 = high-to-lowのエッジトリガ
	4 = アクティブhighのレベルトリガ
	8 = アクティブlowのレベルトリガ
```

```
  指定例:

	i2c@7000c000 {
		gpioext: gpio-adnp@41 {
			compatible = "ad,gpio-adnp";
			reg = <0x41>;

			interrupt-parent = <&gpio>;
			interrupts = <160 1>;

			gpio-controller;
			#gpio-cells = <1>;

			interrupt-controller;
			#interrupt-cells = <2>;

			nr-gpios = <64>;
		};

		sx8634@2b {
			compatible = "smtc,sx8634";
			reg = <0x2b>;

			interrupt-parent = <&gpioext>;
			interrupts = <3 0x8>;

			#address-cells = <1>;
			#size-cells = <0>;

			threshold = <0x40>;
			sensitivity = <7>;
		};
	};
```

3) 割り込み起床親

SoCの割り込みコントローラの中には常に電源が入っており、選択された割り込みが
ルーティングされているものがあり、これはSoCを一時停止から起床させることができます。
これらの割り込みコントローラは親割り込みコントローラの範疇には入らず、"`wakeup-parent`"
プロパティで指定することができ、起床可能な割り込みコントローラを参照する1つのphandle
を含みます。

```
   指定例:
	wakeup-parent = <&pdc_intc>;
```
