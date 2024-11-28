# V6 FSをSD上に作成

## genimageのvf.cfgを編集

```bash
image fs.img {
    ext2 {
        label = "v6fs"
    }
    size = 150K
}

image milkv-duo_sdcard.img {
    hdimage {
    }

    partition boot {
        partition-type = 0xC
        bootable = "true"
        image = "boot.vfat"
    }

    partition fs {
        partition-type = 0x83
        image = "fs.img"
    }
}
```

## `buf.h`, `bio.c`, `sd.c`を変更, `ramdisk`をビルド対象から外す

```bash
[0]sd_init: partition[0]: TYPE: 12, LBA = 0x1, #SECS = 0x20000

[0]sd_init: partition[1]: TYPE: 131, LBA = 0x20001, #SECS = 0x12c


00000000: 0000 0000 0000 0000 0000 0000 0000 0000
00000010: 0000 0000 0000 0000 0000 0000 0000 0000
00000020: 0000 0000 0000 0000 0000 0000 0000 0000
00000030: 0000 0000 0000 0000 0000 0000 0000 0000
00000040: 0000 0000 0000 0000 0000 0000 0000 0000
00000050: 0000 0000 0000 0000 0000 0000 0000 0000
00000060: 0000 0000 0000 0000 0000 0000 0000 0000
00000070: 0000 0000 0000 0000 0000 0000 0000 0000
00000080: 0000 0000 0000 0000 0000 0000 0000 0000
00000090: 0000 0000 0000 0000 0000 0000 0000 0000
000000a0: 0000 0000 0000 0000 0000 0000 0000 0000
000000b0: 0000 0000 0000 0000 0000 0000 0000 0000
000000c0: 0000 0000 0000 0000 0000 0000 0000 0000
000000d0: 0000 0000 0000 0000 0000 0000 0000 0000
000000e0: 0000 0000 0000 0000 0000 0000 0000 0000
000000f0: 0000 0000 0000 0000 0000 0000 0000 0000
00000100: 0000 0000 0000 0000 0000 0000 0000 0000
00000110: 0000 0000 0000 0000 0000 0000 0000 0000
00000120: 0000 0000 0000 0000 0000 0000 0000 0000
00000130: 0000 0000 0000 0000 0000 0000 0000 0000
00000140: 0000 0000 0000 0000 0000 0000 0000 0000
00000150: 0000 0000 0000 0000 0000 0000 0000 0000
00000160: 0000 0000 0000 0000 0000 0000 0000 0000
00000170: 0000 0000 0000 0000 0000 0000 0000 0000
00000180: 0000 0000 0000 0000 0000 0000 0000 0000
00000190: 0000 0000 0000 0000 0000 0000 0000 0000
000001a0: 0000 0000 0000 0000 0000 0000 0000 0000
000001b0: 0000 0000 0000 0000 0000 0000 0000 0000
000001c0: 0000 0000 0000 0000 0000 0000 0000 0000
000001d0: 0000 0000 0000 0000 0000 0000 0000 0000
000001e0: 0000 0000 0000 0000 0000 0000 0000 0000
000001f0: 0000 0000 0000 0000 0000 0000 0000 0000
sd_init ok
[0]readsb: supber-block: 0x00020001, start_block: 0x00010001
[0]printsb: magic: 0x00080005, size: 524294, blk: 524295, inode: 29889270, log: 262144
[0]printsb: num block: 524293, inode: 524294, log: 524295
[0]printsb: start log: 0, inode: 0, bmap: 1079640520
panic: invalid file system
```

## xv6.cfgでfs.imgをext2と指定しているのでext2としてスーパーブロックが作成されている

```bash
04000600: 1000 0000 9600 0000 0700 0000 7d00 0000  ............}...     // LBA: 0x20003
04000610: 0500 0000 0100 0000 0000 0000 0000 0000  ................
04000620: 9800 0000 9800 0000 1000 0000 0000 0000  ................
04000630: 758d 2867 0000 1400 53ef 0100 0000 0000  u.(g....S.......     // 0xef53: ext2のmagic
04000640: 758d 2867 0000 0000 0000 0000 0100 0000  u.(g............
04000650: 0000 0000 0b00 0000 8000 0000 0000 0000  ................
04000660: 0000 0000 0000 0000 b183 268c 74a8 406a  ..........&.t.@j
04000670: b468 6166 069f 086f 7636 6673 0000 0000  .haf...ov6fs....
```

## 諸々間違っていた

- genimageの`image fs.img`のオプションに`ext2`をつけていたため`ext2`で作成された
    - オプション`file`を使用する
- `fs.img`を先に作成していたテスト用のext2ファイルシステムを使用するようになっていた
    - mkfsで作成したv6ファイルを使用するように変更
- スーパーブロックが正しく読み込まれていなかった。
    - MBRのパーティションテーブルエントリからv6ファイルシステムの先頭ブロック番号を取得
    - これにブロックサイズ(1024)を考慮して読み込む位置を調整

```bash
[0]rtc_init: rtc_init ok!
[0]emmc_issue_command_int: waiting for command complete interrupt 1: irpts=0b0000_0000_0000_0001_1000_0000_0000_0000
[0]emmc_card_reset: found valid version 3.0x SD card
[0]sd_init: partition[0]: TYPE: 12, LBA = 0x1, #SECS = 0x20000

[0]sd_init: partition[1]: TYPE: 131, LBA = 0x20001, #SECS = 0x12c

sd_init ok
[0]readsb: supber-block: 0x00020001, start_block: 0x00020003
[0]sd_start: blks: 1024, b->blockno: 0x00020003, addr: 0x04000600   // super-block
[0]sd_start: blks: 1024, b->blockno: 0x00000002, addr: 0x00000400   // これは?
[0]sd_start: blks: 1024, b->blockno: 0x00000002, addr: 0x00000400
[0]sd_start: blks: 1024, b->blockno: 0x00000020, addr: 0x00004000   // これは?
panic: ilock: no type
```

スーパーブロックは正しく読めるようになったがinodeのブロック番号の解析がまちがってるようだ。

## これはlog, inodeのスタートブロックが正しく計算されていなかったため

```bash
sd_init ok
[0]readsb: supber-block: 0x00020001, start_block: 0x00020003
[0]sd_start: buf blockno: 0x00020003, flags: 0x00000001
[0]printsb: magic: 0x10203040, size: 1500, blk: 1466, inode: 256, log: 30
[0]printsb: num block: 270544960, inode: 1500, log: 1466
[0]printsb: start log: 2, inode: 32, bmap: 33
[0]sd_start: buf blockno: 0x00020005, flags: 0x00000001
[0]sd_start: buf blockno: 0x00020005, flags: 0x00000007
[0]sd_start: buf blockno: 0x00020023, flags: 0x00000001

8021b6c0: 0000 0000 0000 0000 0000 0000 0000 0000
8021b6d0: 0000 0000 0000 0000 0000 0000 0000 0000
8021b6e0: 0000 0000 0000 0000 0000 0000 0000 0000
8021b6f0: 0000 0000 0000 0000 0000 0000 0000 0000
panic: ilock: no type
```

## ブロック番号の処理法を変える

- ファイルシステム内の相対ブロック番号（ただし、ブロックサイズ単位）とする
- bgetで一元的にSD内の絶対LBAに変換する

```bash
nmeta 34 (boot, super, log blocks 30 inode blocks 1, bitmap blocks 1) blocks 1466 total 1500

sd_init ok
[0]sd_start: buf blockno: 0x00020003, flags: 0x00000001
[0]printsb: magic: 0x10203040, size: 1500, blk: 1466, inode: 256, log: 30
[0]printsb: num block: 270544960, inode: 1500, log: 1466
[0]printsb: start log: 2, inode: 32, bmap: 33
scause 0xf
sepc=0x80205a22 stval=0x809f3000    // <= アドレスが2倍?
panic: kerneltrap
```

- ページアクセス例外
- superblockは以下の通り

```bash
04000600: 4030 2010 dc05 0000 ba05 0000 0001 0000
04000610: 1e00 0000 0200 0000 2000 0000 2100 0000

struct superblock {
    uint32_t magic;        // 0x10203040 : Must be FSMAGIC
    uint32_t size;         // 0x5dc (1500) : ファイルシステムのサイズ（ブロック単位）
    uint32_t nblocks;      // 0x5ba (1466) : データブロックの総数
    uint32_t ninodes;      // 0x100 (256)  : inodeの総数.
    uint32_t nlog;         // 0x1e  (30)   : ログブロックの総数
    uint32_t logstart;     // 0x2          : 先頭のログブロックのブロック番号
    uint32_t inodestart;   // 0x20  (32)   : 先頭のinodeブロックのブロック番号
    uint32_t bmapstart;    // 0x21  (33)   : 先頭のビットマップブロックのブロック番号
};
```

## 各ブロックの先頭ブロック番号にセクタ番号を指定していた

- データの読み込みはできるようになった

```bash
sd_init ok
[0]bget: bno: 1, blockno: 0x00020003
[0]sd_start: buf blockno: 0x00020003, flags: 0x00000001
[0]emmc_read: nblock: 0x20003, offset: 0x4000600
[0]printsb: magic: 0x10203040, size: 1500
[0]printsb: num of block: 1466, inode: 256, log: 30
[0]printsb: start at log: 2, inode: 32, bmap: 33
[0]initlog: start: 2, size: 30, dev: 1
[0]read_head: log dev: 1, start: 2
[0]bget: bno: 2, blockno: 0x00020005
[0]sd_start: buf blockno: 0x00020005, flags: 0x00000001
[0]emmc_read: nblock: 0x20005, offset: 0x4000a00
[0]bget: bno: 2, blockno: 0x00020005
[0]sd_start: buf blockno: 0x00020005, flags: 0x00000007
[0]bget: bno: 32, blockno: 0x00020041
[0]sd_start: buf blockno: 0x00020041, flags: 0x00000001
[0]emmc_read: nblock: 0x20041, offset: 0x4008200

[0]bget: bno: 34, blockno: 0x00020045
[0]sd_start: buf blockno: 0x00020045, flags: 0x00000001
[0]emmc_read: nblock: 0x20045, offset: 0x4008a00
[0]bget: bno: 34, blockno: 0x00020045
[0]bget: bno: 34, blockno: 0x00020045
[0]bget: bno: 34, blockno: 0x00020045
[0]bget: bno: 34, blockno: 0x00020045
[0]bget: bno: 34, blockno: 0x00020045
[0]bget: bno: 34, blockno: 0x00020045
[0]bget: bno: 34, blockno: 0x00020045
[0]bget: bno: 32, blockno: 0x00020041
[0]sd_start: buf blockno: 0x00020041, flags: 0x00000001
[0]emmc_read: nblock: 0x20041, offset: 0x4008200

[0]bget: bno: 168, blockno: 0x00020151
[0]sd_start: buf blockno: 0x00020151, flags: 0x00000001
[0]emmc_read: nblock: 0x20151, offset: 0x402a200
panic: init exiting         // initプロセスがexit()を呼び出した


[0]rtc_init: rtc_init ok!
[0]emmc_issue_command_int: waiting for command complete interrupt 1: irpts=0b0000_0000_0000_0001_1000_0000_0000_0000
[0]emmc_card_reset: found valid version 3.0x SD card
[0]sd_init: partition[0]: TYPE: 12, LBA = 0x1, #SECS = 0x20000
[0]sd_init: partition[1]: TYPE: 131, LBA = 0x20001, #SECS = 0x12c
sd_init ok
[0]allocproc: pid: 1
[0]userinit: pid: 1
userinit ok
set started 1
[0]main: start scheduler
[0]scheduler: switch to pid: 1
[0]forkret: call fsinit
[0]initlog: init log ok
[0]fsinit: fsinit ok
[0]scheduler: switch to pid: 1

[0]forkret: fall through to usertrapret
[0]forkret: call usertrapret
[0]scheduler: switch to pid: 1

0]sys_v6_exec: exec: /init
[0]exec: bad magic: 0x00000000      // elf magic error
[0]sys_v6_exec: exec ok: ret=-1     // <= execでエラー
[0]scheduler: switch to pid: 1

[0]exit: pid: 1
panic: init exiting
```

## xv6.cfgでfs.imgのサイズの指定が間違っていた

- 1500Kとところを150Kとしていたため、データが削除されていた
- 別のエラーが発生

```bash
sd_init ok
[0]allocproc: pid: 1
[0]userinit: pid: 1
userinit ok
set started 1
[0]main: start scheduler
[0]scheduler: switch to pid: 1
[0]forkret: call fsinit
[0]initlog: init log ok
[0]fsinit: [0]scheduler: switch to pid: 1
fsinit ok
[0]forkret: fall through to usertrapret
[0]forkret: call usertrapret[0]scheduler: switch to pid: 1

[0]sys_v6_exec: exec: /init
[0]scheduler: switch to pid: 1
[0]sys_v6_exec: exec ok: ret=1
[0]scheduler: switch to pid: 1
[0]scheduler: switch to pid: 1
[0]scheduler: switch to pid: 1
[0]emmc_issue_command_int: error occured whilst waiting for transfer complete interrupt 3
[0]emmc_do_data_command: error: 0x1200000 when sending CMD25
[0]emmc_issue_command_int: waiting for command complete interrupt 1: irpts=0b0000_0001_0000_0000_1000_0000_0001_0001
[0]emmc_do_data_command: error: 0x1000000 when sending CMD25
[0]emmc_issue_command_int: waiting for command complete interrupt 1: irpts=0b0000_0001_0000_0000_1000_0000_0001_0001
[0]emmc_do_data_command: error: 0x1000000 when sending CMD25
[0]emmc_do_data_command: Giving up
kernel/sd.c:126: assertion failed.
panic:
```

## auto command エラー

- 転送処理でエラー時に自動的にCMD12を発行する処理でエラーが発生

```bash
[DEBUG] sd_init: partition[0]: TYPE: 12, LBA = 0x1, #SECS = 0x20000
[DEBUG] sd_init: partition[1]: TYPE: 131, LBA = 0x20001, #SECS = 0xbb8
sd_init ok
[DEBUG] allocproc: pid: 1
[DEBUG] userinit: pid: 1
userinit ok
set started 1
[DEBUG] main: start scheduler
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x00020003
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x00020003), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x00020005
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x00020005), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: write 2 blocks from 0x00020005
[DEBUG] emmc_issue_command: 3.2.2 cmd: 25 (0x00020005), success
[DEBUG] scheduler: switch to pid: 1
[INFO ] initlog: init log ok
[INFO ] fsinit: fsinit ok
[DEBUG] scheduler: switch to pid: 1
[DEBUG] sys_v6_exec: exec: /init
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x00020041
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x00020041), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x00020063
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x00020063), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x00020041
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x00020041), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x0002016f
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x0002016f), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x00020177
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x00020177), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x00020179
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x00020179), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x0002017b
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x0002017b), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x0002017d
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x0002017d), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x0002016f
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x0002016f), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x0002017f
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x0002017f), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x0002016f
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x0002016f), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] sys_v6_exec: exec ok: ret=1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x00020063
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x00020063), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x00020041
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x00020041), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x00020043
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x00020043), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x00020063
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x00020063), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x00020041
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x00020041), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x00020007
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x00020007), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x00060087
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x00060087), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: write 2 blocks from 0x00020007
[DEBUG] emmc_issue_command: 3.2.2 cmd: 25 (0x00020007), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x00020009
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x00020009), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: read 2 blocks from 0x000600c7
[DEBUG] emmc_issue_command: 3.2.2 cmd: 18 (0x000600c7), success
[DEBUG] scheduler: switch to pid: 1
[DEBUG] emmc_issue_command: 3.2.2 cmd: 13 (0x00010000), success
[DEBUG] emmc_do_data_command: write 2 blocks from 0x00020009
[ERROR] emmc_issue_command_int: error occured whilst waiting for transfer complete interrupt 3: irpts=0x01208000
[DEBUG] emmc_issue_command: 3.2.2 cmd: 25 (0x00020009), failure: interrupt=0x01208000, error=0x01200000
[ERROR] emmc_do_data_command: error: 0x01200000 when sending CMD25
[ERROR] emmc_issue_command_int: waiting for command complete interrupt 1: irpts=0x01008011
[DEBUG] emmc_issue_command: 3.2.2 cmd: 25 (0x00020009), failure: interrupt=0x01008011, error=0x01000000
[ERROR] emmc_do_data_command: error: 0x01000000 when sending CMD25
[ERROR] emmc_issue_command_int: waiting for command complete interrupt 1: irpts=0x01008011
[DEBUG] emmc_issue_command: 3.2.2 cmd: 25 (0x00020009), failure: interrupt=0x01008011, error=0x01000000   // auto command error
[ERROR] emmc_do_data_command: error: 0x01000000 when sending CMD25
[ERROR] emmc_do_data_command: Giving up
kernel/sd.c:126: assertion failed.
panic:

$ ./dump o
Block     : LBA     Address    (#block)
Boot      : 0x20001 0x04000200 (1)
SuperBlock: 0x20003 0x04000600 (1)
Log       : 0x20005 0x04000a00 (30)
INode     : 0x20041 0x04008200 (16)
Bitmap    : 0x20061 0x0400c200 (1)
Data      : 0x20063 0x0400c600
```

```bash
sd_init ok
[DEBUG] allocproc: pid: 1
[DEBUG] userinit: pid: 1
userinit ok
set started 1
[DEBUG] main: start scheduler
[DEBUG] scheduler: switch to pid: 1
[INFO ] initlog: init log ok
[INFO ] fsinit: fsinit ok
[DEBUG] scheduler: switch to pid: 1

[DEBUG] sys_v6_exec: exec: /init
[DEBUG] scheduler: switch to pid: 1
[DEBUG] sys_v6_exec: exec ok: ret=1
[DEBUG] scheduler: switch to pid: 1

[DEBUG] scheduler: switch to pid: 1
[DEBUG] write_log: write 0x00020007
[DEBUG] scheduler: switch to pid: 1

[DEBUG] write_log: write 0x00020009
[ERROR] emmc_issue_command_int: error occured whilst waiting for transfer complete interrupt 3: irpts=0x01208000
[DEBUG] emmc_issue_command: 3.2.2 cmd: 25 (0x00020009), failure: interrupt=0x01208000, error=0x01200000     // 15: err int, 21: data-crc, 24: auto-cmd
[ERROR] emmc_do_data_command: error: 0x01200000 when sending CMD25
[DEBUG] emmc_issue_command_int: auto cmd err: 0x03                      // AUTOCMD12_NO_EXE & AUTO_CMD_TOUT_ERR
[ERROR] emmc_issue_command_int: waiting for command complete interrupt 1: irpts=0x01008011
[DEBUG] emmc_issue_command: 3.2.2 cmd: 25 (0x00020009), failure: interrupt=0x01008011, error=0x01000000     // 0: cmd comp, 1: trans comp, 15: err int, 24: auto-cmd
[ERROR] emmc_do_data_command: error: 0x01000000 when sending CMD25
[DEBUG] emmc_issue_command_int: auto cmd err: 0x03
[ERROR] emmc_issue_command_int: waiting for command complete interrupt 1: irpts=0x01008011
[DEBUG] emmc_issue_command: 3.2.2 cmd: 25 (0x00020009), failure: interrupt=0x01008011, error=0x01000000
[ERROR] emmc_do_data_command: error: 0x01000000 when sending CMD25
[ERROR] emmc_do_data_command: Giving up
kernel/sd.c:126: assertion failed.
panic:
```

## カードの問題であった

- カードを変えたらエラーなく正常に動いた

```bash
Starting kernel ...


xv6 kernel is booting in hart 0

SBI specification v0.3 detected
SBI TIME extension detected
SBI IPI extension detected
SBI RFNC extension detected
SBI HSM extension detected

[INFO ] rtc_init: rtc_init ok!
[ERROR] emmc_issue_command_int: waiting for command complete interrupt 1: irpts=0x00018000
[DEBUG] emmc_issue_command: 3.2.2 cmd: 5 (0x00000000), failure: interrupt=0x00018000, error=0x00010000
[DEBUG] emmc_issue_command: issuing command ACMD41
[DEBUG] emmc_issue_command: 3.1.2 cmd: 55 (0x00000000), result: success
[DEBUG] emmc_issue_command: 3.1.3 cmd: 41 (0x00000000), result: success
[DEBUG] emmc_issue_command: issuing command ACMD41
[DEBUG] emmc_issue_command: 3.1.2 cmd: 55 (0x00000000), result: success
[DEBUG] emmc_issue_command: 3.1.3 cmd: 41 (0x40ff8000), result: success
[DEBUG] emmc_issue_command: issuing command ACMD41
[DEBUG] emmc_issue_command: 3.1.2 cmd: 55 (0x00000000), result: success
[DEBUG] emmc_issue_command: 3.1.3 cmd: 41 (0x40ff8000), result: success
[DEBUG] emmc_issue_command: issuing command ACMD51
[DEBUG] emmc_issue_command: 3.1.2 cmd: 55 (0x12340000), result: success
[DEBUG] emmc_issue_command: 3.1.3 cmd: 51 (0x00000000), result: success
[DEBUG] emmc_issue_command: issuing command ACMD6
[DEBUG] emmc_issue_command: 3.1.2 cmd: 55 (0x12340000), result: success
[DEBUG] emmc_issue_command: 3.1.3 cmd: 6 (0x00000002), result: success
[INFO ] emmc_card_reset: found valid version 3.0x SD card
[DEBUG] sd_init: partition[0]: TYPE: 12, LBA = 0x1, #SECS = 0x20000
[DEBUG] sd_init: partition[1]: TYPE: 131, LBA = 0x20001, #SECS = 0xbb8
sd_init ok
[DEBUG] allocproc: pid: 1
[DEBUG] userinit: pid: 1
userinit ok
set started 1
[DEBUG] main: start scheduler
[DEBUG] scheduler: switch to pid: 1
[DEBUG] scheduler: switch to pid: 1
[INFO ] initlog: init log ok
[INFO ] fsinit: fsinit ok
[DEBUG] sys_v6_exec: [DEBUG] scheduler: switch to pid: 1
exec: /init
[DEBUG] exec: inum: 7
[DEBUG] scheduler: switch to pid: 1
[DEBUG] exec: end_op start for inum: 7
[DEBUG] exec: [DEBUG] scheduler: switch to pid: 1
end_op ok
[DEBUG] sys_v6_exec: exec ok: ret=1
[DEBUG] scheduler: switch to pid: 1
[DEBUG] scheduler: switch to pid: 1
[DEBUG] commit: lh.n: 3
[DEBUG] write_log: [DEBUG] scheduler: switch to pid: 1
tail[0]: write 0x00020007
[DEBUG] scheduler: switch to pid: 1
[DEBUG] write_log: tail[0] ok
[DEBUG] write_log: tail[1]: write 0x00020009[DEBUG] scheduler: switch to pid: 1

[DEBUG] scheduler: switch to pid: 1
[DEBUG] write_log: tail[1] ok
[DEBUG] write_log: tail[2]: write 0x0002000b[DEBUG] scheduler: switch to pid: 1

[DEBUG] scheduler: switch to pid: 1
[DEBUG] write_log: tail[2] ok
[DEBUG] scheduler: switch to pid: 1
[DEBUG] scheduler: switch to pid: 1
[DEBUG] scheduler: switch to pid: 1
[DEBUG] scheduler: switch to pid: 1
[DEBUG] scheduler: switch to pid: 1
init: st[DEBUG] allocproc: pid: 2
ar[DEBUG] fork: pid: 2
[DEBUG] scheduler: switch to pid: 2
[DEBUG] scheduler: switch to pid: 2
[DEBUG] sys_v6_exec: exec: sh
ting sh
[DEBUG] exec: inum: 13
[DEBUG] scheduler: switch to pid: 2
[DEBUG] exec: end_op start for inum: 13[DEBUG] scheduler: switch to pid: 2

[DEBUG] exec: end_op ok
[DEBUG] sys_v6_exec: exec ok: ret=1
$ [DEBUG] scheduler: switch to pid: 2

[DEBUG] scheduler: switch to pid: 2
[DEBUG] scheduler: switch to pid: 2
[DEBUG] allocproc: pid: 3
[DEBUG] fork: pid: 3
[DEBUG] scheduler: switch to pid: 3
[DEBUG] scheduler: switch to pid: 3
[DEBUG] exit: pid: 3
[DEBUG] scheduler: switch to pid: 2
[DEBUG] freeproc: pid: 3
[DEBUG] scheduler: switch to pid: 2
$ ls
[DEBUG] scheduler: switch to pid: 2
[DEBUG] allocproc: pid: 4
[DEBUG] scheduler: switch to pid: 2
[DEBUG] fork: pid: 4
[DEBUG] scheduler: switch to pid: 4
[DEBUG] sys_v6_exec: [DEBUG] scheduler: switch to pid: 4
exec: ls
[DEBUG] exec: inum: 10
[DEBUG] exec: [DEBUG] scheduler: switch to pid: 4
end_op start for inum: 10
[DEBUG] exec: end_op ok
[DEBUG] sys_v6_exec: exec ok: ret=1[DEBUG] scheduler: switch to pid: 4

.              1 1 1024
..             1 1 1024
README.md      2 2 7[DEBUG] scheduler: switch to pid: 4
71
cat          [DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
2[DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
3[DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
 36456
echo       [DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
2[DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
4[DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
 35064
forktest   [DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
2[DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
5[DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
 16424
grep         [DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
2[DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
6[DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
 40160
init         [DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
2[DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
7[DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
 35560
kill          [DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
2[DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
8[DEBUG] scheduler: switch to pid: 4
 35080
ln             [DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
2[DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
9[DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
34960
ls             2 [DEBUG] scheduler: switch to pid: 4
1[DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
0[DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
38568
mkdir          2 11[DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
35104
rm             2 12[DEBUG] scheduler: switch to pid: 4
 [DEBUG] scheduler: switch to pid: 4
[DEBUG] scheduler: switch to pid: 4
35096
sh             2 13 [DEBUG] scheduler: switch to pid: 4
58776
stressf[DEBUG] scheduler: switch to pid: 4
s       2 14 36[DEBUG] scheduler: switch to pid: 4
144
usertests      2 15[DEBUG] scheduler: switch to pid: 4
 166656
grind          2 16 49472
wc             2 17 37200
zombie         [DEBUG] scheduler: switch to pid: 4
2 18 34584
blink          2 19 36064
console        3 20 0
[DEBUG] scheduler: switch to pid: 4
[DEBUG] exit: pid: 4
[DEBUG] scheduler: switch to pid: 2
[DEBUG] freeproc: pid: 4
[DEBUG] scheduler: switch to pid: 2
$ exit
[DEBUG] scheduler: switch to pid: 2
[DEBUG] allocproc: pid: 5
[DEBUG] fork: [DEBUG] scheduler: switch to pid: 5
[DEBUG] sys_v6_exec: exec: exit
[DEBUG] exec: path: exit couldn't find[DEBUG] scheduler: switch to pid: 2
pid: 5
[DEBUG] scheduler: switch to pid: 5

[DEBUG] sys_v6_exec: exec ok: ret=-1[DEBUG] scheduler: switch to pid: 5

exec e[DEBUG] exit: pid: 5
x[DEBUG] scheduler: switch to pid: 2
[DEBUG] freeproc: pid: 5
i[DEBUG] scheduler: switch to pid: 2
t failed
$
```

## デバッグ行を削除

```bash
[INFO ] rtc_init: rtc_init ok!
[ERROR] emmc_issue_command_int: waiting for command complete interrupt 1: irpts=0x00018000
[INFO ] emmc_card_reset: found valid version 3.0x SD card
[INFO ] sd_init: partition[0]: TYPE: 12, LBA = 0x1, #SECS = 0x20000
[INFO ] sd_init: partition[1]: TYPE: 131, LBA = 0x20001, #SECS = 0xbb8
[INFO ] sd_init: sd_init ok

[INFO ] initlog: init log ok
[INFO ] fsinit: fsinit ok
init: starting sh
$ ls
.              1 1 1024
..             1 1 1024
README.md      2 2 771
cat            2 3 36456
echo           2 4 35064
forktest       2 5 16424
grep           2 6 40160
init           2 7 35560
kill           2 8 35080
ln             2 9 34960
ls             2 10 38568
mkdir          2 11 35104
rm             2 12 35096
sh             2 13 58776
stressfs       2 14 36144
usertests      2 15 166656
grind          2 16 49472
wc             2 17 37200
zombie         2 18 34584
blink          2 19 36064
console        3 20 0
$
Meta-Z for help | 115200 8N1 | NOR | Minicom 2.8 | VT102 | Offline | cu.usbserial-AI057C9L
dspace@mini:~/xv6-milkv/xv6-milkv-duo/duo-imgtools$
```
