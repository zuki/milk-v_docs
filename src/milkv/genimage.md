# `genimage`

- [Github](https://github.com/pengutronix/genimage)

以下は`README`の冒頭の翻訳

## Genimage - イメージ作成ツール

genimageは指定したルートファイルシステムツリーから複数のファイルシステムと
フラッシュ/ディスクイメージを生成するツールです。genimageはfakeroot環境で
実行されることを想定しています。genimageは異なるファイルシステムイメージや
ファイルからフラッシュ/ディスクイメージを作成することもできます。

構成は`libconfuse`により解析されるconfigファイルで行います。ツールへの
パスなどのオプションは環境変数、configファイル、コマンドラインスイッチで
指定することができます。

### configファイル

genimageのconfigファイルには`libconfuse`が提供するシンプルな構成言語を
使用します。この言語は単純なキーと値のペアだけでなく、ネストされた
セクションもサポートしています。

一行コメントは`#`または`//`で、複数行コメントは`/* ... */` で記述します
（C言語と同じ）。

configファイルはメインセクションである`image`、`flash`、`config`に
分かれており、`include`プリミティブを提供しています。

#### imageセクション

imageセクションには構築するファイルシステムまたはディスクイメージを1つ
記述します。複数のイメージを生成するためにimageセクションは複数回指定
することができます。イメージはイメージ自身を参照する複数のパーティションを
持つこともできます。各イメージにはタイプを指定する必要があり、タイプに
よって異なるサブオプショ ンを指定することができます。

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
