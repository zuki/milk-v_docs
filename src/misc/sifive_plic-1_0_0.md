# SiFive Platform-Level Interrupt Controller (PLIC)

```
# SPDX-License-Identifier: GPL-2.0-only OR BSD-2-Clause
# Copyright (C) 2020 SiFive, Inc.
%YAML 1.2
---
$id: http://devicetree.org/schemas/interrupt-controller/sifive,plic-1.0.0.yaml#
$schema: http://devicetree.org/meta-schemas/core.yaml#
```

## 説明

SiFive SOCにはRISC-V Privileged Architecture仕様のPLIC（Platform-Level Interrupt
Controller）ハイレベル仕様の実装が含まれています。PLICは各ハートの外部割り込みソースを
介してシステムのすべての外部割り込みをシステムのすべてのハートコンテキストに接続します。

ハートコンテキストはハードウェア実行スレッドでは特権モードです。たとえば、2ウェイSMTの
4コアシステムでは、8つのハートがあり、おそらく1つのハートにつき少なくとも2つの特権モード
（マシンモードとスーパバイザモード）があります。

各割り込みはコンテキストごとに有効にできます。どのコンテキストも保留中の有効な割り込みを
要求し、それを処理して解放することができます。

各割り込みには構成可能な優先度があります。優先順位の高い割り込みが最初に処理されます。
各コンテキストは優先度のしきい値を指定できます。このしきい値以下の優先順位の割り込みは
PLICがそのコンテキストにつながる割り込みラインを上げる原因になりません。

PLICはエッジトリガ割り込みとレベルトリガ割り込みを共にサポートしていますが、割り込み
ハンドラはこれを区別しないのでPLICのデバイスツリーバインディングでは指定されていません。

RISC-V ISAはPLICのメモリレイアウトを規定していませんが”sifive,plic-1.0.0"デバイスは
特定のメモリレイアウトを持つPLICの具体的な実装であり[SiFive U5 Coreplexシリーズマニュアル Manual](https://static.dev.sifive.com/U54-MC-RVCoreIP.pdf)の第8章に文書化されています。

## メンテナ

- Sagar Kadam <sagar.kadam@sifive.com>
- Paul Walmsley  <paul.walmsley@sifive.com>
- Palmer Dabbelt <palmer@dabbelt.com>

## 属性

```
compatible:
  items:
      - const: sifive,fu540-c000-plic
      - const: sifive,plic-1.0.0

  reg:
    maxItems: 1

  '#address-cells':
    const: 0

  '#interrupt-cells':
    const: 1

  interrupt-controller: true

  interrupts-extended:
    minItems: 1
    description:
      PLICに接続されているコンテキストを指定します。"-1"はコンテキストが存在しない
      ことを示しています。指定されるノードは`riscv`ノードを親に持つ`riscv,cpu-intc`
      ノードでなければなりません。

  riscv,ndev:
    $ref: "/schemas/types.yaml#/definitions/uint32"
    description:
      このコントローラがサポートする外部割込みの数を指定します。

required:
  - compatible
  - '#address-cells'
  - '#interrupt-cells'
  - interrupt-controller
  - reg
  - interrupts-extended
  - riscv,ndev

additionalProperties: false
```

## 指定例

```
  - |
    plic: interrupt-controller@c000000 {
      #address-cells = <0>;
      #interrupt-cells = <1>;
      compatible = "sifive,fu540-c000-plic", "sifive,plic-1.0.0";
      interrupt-controller;
      interrupts-extended = <
        &cpu0_intc 11
        &cpu1_intc 11 &cpu1_intc 9
        &cpu2_intc 11 &cpu2_intc 9
        &cpu3_intc 11 &cpu3_intc 9
        &cpu4_intc 11 &cpu4_intc 9>;
      reg = <0xc000000 0x4000000>;
      riscv,ndev = <10>;
    };
```
