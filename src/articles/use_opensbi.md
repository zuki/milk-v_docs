# OpenSBIを使って自作OSを起動する

[オリジナル](https://forum.sophgo.com/t/use-opensbi-to-boot-your-own-operating-system/340)

以前、[U-Bootを使って自作OSをブートストラップする](https://community.milkv.io/t/uboot/181)と
いう記事において、OpenSBIを使って自作OSをブートストラップしようとしました。しかし、FIP
(Firmware Image Package)を作成する際、および、ATF (Arm Trusted Firmware) をスタートアップ
する際にbl33イメージのフォーマットに要件があることがわかりました。

この間、RISC-Vのアセンブリとリンカスクリプトを掘り下げ、ATFとU-Bootのコードを再検討しました。
そして独自のbl33イメージを作るのは実はとても簡単だということに気づきました。必要なのは
プログラムの冒頭に以下の情報を含めることだけでした。

```
_start:
	/* BL33 information */
	j real_start
	.balign 4
	.word 0x33334c42  /* b'BL33' */
	.word 0xdeadbeea  /* CKSUM */
	.word 0xdeadbeeb  /* SIZE */
	.quad 0x80200000  /* RUNADDR */
	.word 0xdeadbeec
	.balign 4
	j real_start
	.balign 4
	/* BL33 end */
```

簡単な例を示します。

SDKをコンパイルしたことがない場合は、次の手順でFSBL (First Stage Bootloader)をコンパイルして、
bl2イメージを取得する必要があります。

```bash
export MILKV_BOARD=milkv-duo
source milkv/boardconfig-milkv-duo.sh

source build/milkvsetup.sh
defconfig cv1800b_milkv_duo_sd

build_fsbl
```

次に、uart8250を使って文字列をプリントする簡単なプログラムを書きます。

```
#define UART0_THR 0x04140000
#define UART0_LSR 0x04140014

	.section .text
	.global _start
_start:
	/* BL33 information */
	j real_start
	.balign 4
	.word 0x33334c42  /* b'BL33' */
	.word 0xdeadbeea  /* CKSUM */
	.word 0xdeadbeeb  /* SIZE */
	.quad 0x80200000  /* RUNADDR */
	.word 0xdeadbeec
	.balign 4
	j real_start
	.balign 4
	/* Information end */

real_start:
	la s0, str
1:
	lbu a0, (s0)
	beqz a0, exit
	jal ra, uart_send
	addi s0, s0, 1
	j 1b

exit:
	j exit

uart_send:
	/* Wait for tx idle */
	li t0, UART0_LSR
	lw t1, (t0)
	andi t1, t1, 0x20
	beqz t1, uart_send
	/* Send a char */
	li t0, UART0_THR
	sw a0, (t0)
	ret

.section .rodata
str: 
	.asciz "Hello Milkv-duo!\n"
```

コンパイルします。

```bash
riscv64-unknown-elf-gcc -nostdlib -fno-builtin -march=rv64gc -mabi=lp64f -g -Wall -Ttext=0x80200000 -o bl33.elf start.S

riscv64-unknown-elf-objcopy -O binary bl33.elf bl33.bin
```

`hd`を使って生成された`bin`ファイルを調べると、検証用のデータが2行あることがわかります。
一方、OpenSBIは実際に0x80200020にジャンプしてプログラムを実行します。

```
00000000  05 a0 01 00 42 4c 33 33  ea be ad de eb be ad de  |....BL33........|
00000010  00 00 20 80 00 00 00 00  ec be ad de 11 a0 01 00  |.. .............|
```

`fsbl`ディレクトリに移動して`fip.bin`ファイルを生成します。

```bash
cd fsbl/

./plat/cv180x/fiptool.py -v genfip \
    'build/cv1800b_milkv_duo_sd/fip.bin' \
    --MONITOR_RUNADDR="0x0000000080000000" \
    --CHIP_CONF='build/cv1800b_milkv_duo_sd/chip_conf.bin' \
    --NOR_INFO='FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF' \
    --NAND_INFO='00000000'\
    --BL2='build/cv1800b_milkv_duo_sd/bl2.bin' \
    --DDR_PARAM='test/cv181x/ddr_param.bin' \
    --MONITOR='../opensbi/build/platform/generic/firmware/fw_dynamic.bin' \
    --LOADER_2ND='/path/to/bl33.bin' \
    --compress='lzma'
```

最後に`fip.bin`ファイルをTFカードに置き、ボーレート128000でボードのシリアルポートに接続すれば
次の情報を見ることができます。

![boot情報](boot_info.png)
