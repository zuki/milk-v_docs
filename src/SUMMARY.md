# Summary

[目次](toc.md)

- [RISC-V ソフトウェアの移植と最適化チャンピオンシップ](rvspoc/README.md)

    - [教義概要](rvspoc/p2308.md)
    - [応募作品概要](rvspoc/application.md)
        - [virus-V氏](rvspoc/virusv.md)
        - [xhackerustc氏](rvspoc/xhack.md)
        - [BigBrotherJu氏](rvspoc/bbj.md)

- [Milk-V Duo関係資料](milkv/README.md)

    - [ビルドメモ](milkv/linux.md)
    - [fip_binについて](milkv/fip_bin.md)
    - [fiptoolを試す](milkv/fiptool.md)
    - [u-bootのbootmコマンド定義](milkv/bootm.md)
    - [U-Bootドキュメント](milkv/u-boot.md)
    - [OpenSBI: Genericプラットフォーム](milkv/sbi_generic.md)
    - [TF-Aドキュメント: FIP](milkv/tfa_fip.md)
    - [genimage](milkv/genimage.md)
    - [ベアボーンおよび非ベアボーンアップグレードユーザーズマニュアル](milkv/barebone.md)
    - [RISC-V Linuxのブートイメージヘッダ](milkv/boot_img_header.md)

- [Web上の情報](articles/README.md)

    - [RISC-Vブートフロー入門](articles/Summit_bootflow.md)
    - [ubootを使って自作OSを起動する](articles/use_uboot.md)
    - [OpenSBIを使って自作OSを起動する](articles/use_opensbi.md)
    - [milk-v duo純正Linux_SDKコンパイルプロセス](articles/linux_sdk_compile.md)
    - [小型コアでRTOSをブートする - RISC-V C906](articles/boot_rtos_on_c906.md)

- [CV180x](cv180x/README.md)
    - [データシート](cv180x/datasheet/README.md)
        - [1.5: アドレス空間マッピング](cv180x/datasheet/address_space.md)
        - [3.1: リセット](cv180x/datasheet/reset.md)
        - [3.2: クロック](cv180x/datasheet/clock.md)
        - [3.4: 割り込みシステム](cv180x/datasheet/interrupt.md)
        - [3.5: システムコントローラ](cv180x/datasheet/system_controller.md)
        - [3.7: タイマー](cv180x/datasheet/timer.md)
        - [3.9: リアルタイムクロック](cv180x/datasheet/rtc.md)
        - [12.4: SD/SDIOコントローラ](cv180x/datasheet/sd_sdio.md)
    - [Pinout-v1](cv180x/pinout/README.md)
        - [1. SD0関係レジスタ一覧](cv180x/pinout/table_0.md)
        - [2. SD0機能選択レジスタ](cv180x/pinout/table_1.md)
        - [3. SD0制御レジスタ](cv180x/pinout/table_3.md)
        - [4. SD0デフォルト値](cv180x/pinout/table_4.md)

- [その他の資料](misc/README.md)

    - [HLICデバイスツリーエントリ](misc/hlic_devtree_entry.md)
    - [U-Boot MMC関連の関数と構造体](misc/uboot_mmc.md)
    - [emmc.cとmmc.cの比較](misc/emmc.md)

- [xv6実装時のメモ](xv6/README.md)

    - [SDカードを使えるようにする](xv6/sdcard.md)
