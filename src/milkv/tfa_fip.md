# TF-Aドキュメント: FIP

[オリジナル](https://trustedfirmware-a.readthedocs.io/en/latest/design/firmware-design.html#firmware-image-package-fip)

## 5.4.12 FIP: Firmaware Image Package

FIP (Firmaware Image Package)を使用することにより、ブートローダイメ ージ（と潜在的には
他のペイロード）を不揮発性プラットフォームストレージからTF-Aによりロード可能な単一の
アーカイブにパックすることができます。FIPからイメージをロードするドライバがストレージ層に
追加され、サポートされたプラットフォームストレージからパッケージを読み込むことができる
ようになりました。FIPを作成するツールも提供されています。それについては以下で説明されます。

### 5.4.12.1. FIPのレイアウト

FIPのレイアウトは目次（ToC）とそれに続くペイロードデータで構成されています。ToCには
ヘッダーがあり、その後に1つ以上のテーブルエントリが続きます。ToCはエンドマーカー
エントリで終了し、ToCのサイズは0バイトであるため、オフセットはFIPファイルの合計サイズに
等しくなります。すべてのToCエントリはバイナリパッケージの最後に追加されたペイロード
データを記述します。ToCエントリで提供される情報により、対応するペイロードデータを検索
することができます。

```
------------------
| ToC Header     |
|----------------|
| ToC Entry 0    |
|----------------|
| ToC Entry 1    |
|----------------|
| ToC End Marker |
|----------------|
|                |
|     Data 0     |
|                |
|----------------|
|                |
|     Data 1     |
|                |
------------------
```

ToCヘッダーとエントリーのフォーマットはヘッダーファイル`include/tools_share/firmware_image_package.h`に
記述されています。このファイルはツールとTF-Aがともに使用しています。

ToCヘッダーは次のフィールドを持ちます。

```
`name`: ToCの名前。現在はヘッダーのバリデータとして使用されています。
`serial_number`: 作成ツール二提供される非ゼロの数値
`flags`: このデータに関連するフラグ
    Bits 0-31: 予約
    Bits 32-47: プラットフォーム定義
    Bits 48-63: 予約
```

Tocエントリは次のフィールドを持ちます。

```
`uuid`: すべてのファイルは事前に定義されたUUID (Universally Unique IDentifier) [UUID]に
    より参照されます。UUIDは`include/tools_share/firmware_image_package.h`で定義されて
    います。プラットフォームはパケージにアクセスする際、指定されたイメージ名を対応する
    UUIDに変換します。
`offset_address`: 対応するペイロードデータを見つけることができるオフセットアドレスです。
    オフセットはToCのベースアドレスから計算されます。
`size`: 対応するペイロードデータのバイト単位のサイズです。
`flags`: このエントリに関連するフラグ。まだ定義されていません。
```

### 5.4.12.2. FIP作成ツール

FIP作成ツールを使って指定されたイメージをプラットフォームストレージからTF-Aがロード可能な
バイナリパッケージにパックすることができます。このツールは現在のところ、ブートローダ
イメージのパックにしか対応していません。必要に応じて、追加のイメージ定義をツールに追加する
ことができます。

このツールは`tools/fiptool`にあります。

### 5.4.12.3. FIPからのロード

FIPドライバは不揮発性プラットフォーム ストレージに保存されたバイナリパッケージから
イメージをロードすることができます。Arm開発プラットフォームではこれは`NOR FLASH`です。

ブートローダイメージは`plat_get_image_source()`関数で指定されたプラットフォームポリシーに
従ってロードされます。Arm開発用プラットフォームではこれは`NOR FLASH0`の先頭にある
FIPからイメージをロードすることを意味します。

Arm開発用プラットフォームのポリシーでは既知のイメージセットのロードのみを許可しています。
プラットフォームのポリシーで追加イメージを許可するように変更できます。

## 参考

### firmware_image_package.h

```c
/* This is used as a signature to validate the blob header */
#define TOC_HEADER_NAME	0xAA640001

typedef struct fip_toc_header {
	uint32_t	name;
	uint32_t	serial_number;
	uint64_t	flags;
} fip_toc_header_t;             // 16バイト

typedef struct fip_toc_entry {
	uuid_t		uuid;           // 16バイト
	uint64_t	offset_address;
	uint64_t	size;
	uint64_t	flags;
} fip_toc_entry_t;              // 40バイト

#endif /* FIRMWARE_IMAGE_PACKAGE_H */
```
