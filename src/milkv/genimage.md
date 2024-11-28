# `Genimage - イメージ作成ツール

- [Github](https://github.com/pengutronix/genimage)

genimageは指定したルートファイルシステムツリーから複数のファイルシステムや
フラッシュ/ディスクイメージを生成するツールです。genimageはfakeroot環境で
実行されることを想定しています。genimageは異なるファイルシステムイメージや
ファイルからフラッシュ/ディスクイメージを作成することもできます。

構成は`libconfuse`により解析されるconfigファイルで行います。ツールへの
パスなどのオプションは環境変数、configファイル、コマンドラインスイッチで
指定することができます。

## configファイル

genimageのconfigファイルには`libconfuse`が提供するシンプルな構成言語を
使用します。この言語は単純なキーと値のペアだけでなく、ネストされた
セクションもサポートしています。

一行コメントは`#`または`//`で、複数行コメントは`/* ... */` で記述します
（C言語と同じ）。

configファイルはメインセクションである`image`、`flash`、`config`に
分かれており、`include`プリミティブを提供しています。

### imageセクション

imageセクションには構築するファイルシステムまたはディスクイメージを1つ
記述します。複数のイメージを生成するためにimageセクションは複数回指定
することができます。イメージはイメージ自身を参照する複数のパーティションを
持つこともできます。各イメージにはタイプを指定する必要があり、タイプに
よって異なるサブオプションを指定することができます。

例を見てみましょう。

```bash
image nand-pcm038.img {
        flash {
        }
        flashtype = "nand-64M-512"
        partition barebox {
                image = "barebox-pcm038.bin"
                size = 512K
        }
        partition root {
                image = "root-nand.jffs2"
                size = 24M
        }
}
```

これは"nand-64M-512"タイプのフラッシュである`nand-pcm038.img`を生成します。
このイメージは"barebox-pcm038.bin"と"root-nand.jffs2"という2つの
パーティションを含んでおり、これらのパーティションはconfigファイルの他の
場所で記述されているイメージを参照している必要があります。たとえば、
"root-nand.jffs2"パーティションは次のように記述できます。

```bash
image root-nand.jffs2 {
        name = "root"
        jffs2 {}
        size = 24M
        mountpoint = "/"
}
```

この場合は単一のjffs2イメージがルートマウントポイントから生成されます。

以下はimageセクションで使用するすべてのオプションです。

| オプション | 説明 |
|:----------:|:-----|
| name: |  |
| size: |  |
| mountpoint: |  |
| srcpath: |  |
| empty: |  |
| temporary: |  |
| exec-pre: |  |
| exec-post: |  |
| flashtype: |  |
| partition: |  |

さらに各イメージにはイメージの種類を示す次のセクションのいずれかを指定できます。

`cpio`, `cramfs`, `ext2`, `ext3`, `ext4`, `file`, `flash`, `hdimage`, `iso`,
`jffs2`, `qemu`, `squashfs`, `tar`, `ubi`, `ubifs`, `vfat`

`Partition`オプション

| オプション | 説明 |
|:----------:|:-----|
| offset: |  |
| size: |  |
| align: |  |
| partition-type: |  |
| image: |  |
| fill: |  |
| autoresize: |  |
| bootable: |  |
| in-partition-table: |  |
| forced-primary: |  |
| partition-uuid: |  |
| partition-type-uuid: |  |

各パーティションについて、最終的なアライメント、オフセット、サイズは以下のように
決定されます。

- `align`オプションが指定されていない場合はそのパーティションがパーティション
  テーブル内にある場合はそのイメージの`align`オプションの値が、 そうでなければ
  1がデフォルトとなります。
- `offset`オプションが指定されていないか0で、かつ`in-partition-table`がtrueの
  場合、そのパーティションはそれ以前に定義されているすべてのパーティションの
  後に配置され、最終的なオフセットはそのパーティションの`align`値に丸められます。
- そうでない場合は`offset`オプションはそのまま使用されます。指定されない場合の
  デフォルトは0です。したがって、実際上、パーティションテーブルにないパーティション
  （ブートローダなどの例外を除いて）にはオフセットを指定しなければなりません。
- パーティションに`autoresize`フラグが設定されている場合、そのオフセットから
  イメージに残っているスペースを計算（GPTイメージの場合、バックアップGPTテーブル
  用に最後にスペースが確保されます）し、パーティションの`align`値に丸められます。
  パーティションに`size`オプションも指定されている場合、計算された値がそのサイズ
  より小さくならないことが保証されます。
- そうでない場合でゼロでない`size`オプションが指定された場合はその値がそのまま
  使われます。
- そうでない場合でパーティションに`image`オプションが指定されている場合は、
  そのイメージのサイズがパーティションの`align`値に切り上げられパーティションの
  サイズの決定に使用されます。

これらの最終値に対して以下のサニティチェックが行われます（多くの場合、明示的に
値を指定するのではなく、上記のルールのいずれかを使って値を決定した場合、これらは
自動的に満たされます）。

- パーティションテーブルのパーティションでは、パーティションの`align`値は
  イメージの`align`値以上でなければなりません。
- パーティションの`offset`と`size`はどちらも`align`の倍数でなければなりません。
- `size`は0であってはなりません。
- パーティションは他のパーティションやパーティションテーブルの領域と重なっては
  いけません。

### image構成オプション

#### file

これはそのまま使用する既存のイメージを表します。`partition`セクションが`config`ファイルの
他の場所で定義されていないイメージを参照する場合、`file`ルールが暗黙的に生成されます。
イメージが入力ディレクトリに存在すること、または、イメージへの絶対パスを使用することは
ユーザの責任です。

`file`イメージを明示的に追加することも可能であり、 その場合は自動的には推測できない
イメージに関する情報を`genimage`に与えることができます。現在のところ、そのための
オプションが1つだけ存在します。

| オプション | 説明 |
|:----------:|:-----|
| holes: | "(<start>;<end>)"ペアのリストで意味のあるデータを含まないファイルの範囲を指定し、他のパーティションやイメージメタデータとの重複を許します |

例:

```
image foo {
        hdimage {
                partition-table-type = "gpt"
                gpt-location = 64K
        }

        partition bootloader {
                in-partition-table = false
                offset = 0
                image = "/path/to/bootloader.img"
        }

        partition rootfs {
                offset = 1M
                image = "rootfs.ext4"
        }
}

image /path/to/bootloader.img {
        file {
                holes = {"(440; 1K)", "(64K; 80K)"}
        }
}
```

この例は`genimage`に`bootloader`パーティションが`MBR`（DOS パーティションテーブルがある）
の最後の72バイトとオフセット512から始まるセクタにあるGPTヘッダが重なっていても、
問題ありません。`bootloader.img`はその範囲に有用なデータを含んでいないからです。さらに、
この例では`bootloader`イメージがGPTアレイをオフセット64Kに配置できるように注意深く
作成されています（GPTヘッダは常にオフセット512にあります）。

`bootloader`イメージが明示的に宣言されず、一度しか使用されない場合、ホールも
パーティションで設定できます。これにより単純なユースケースのconfigファイルを簡素化できます。

例:

```
image bar {
        hdimage {}

        partition bootloader {
                in-partition-table = false
                offset = 0
                image = "/path/to/bootloader.img"
                holes = {"(440; 512)"}
        }

        partition rootfs {
                offset = 1M
                image = "rootfs.ext4"
        }
}
```
