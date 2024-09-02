# bbj版

```bash
$ diff -qr xv6-org xv6-bbj
Files xv6-org/Makefile xv6-bbj/Makefile differ
Files xv6-org/kernel/bio.c xv6-bbj/kernel/bio.c differ
Files xv6-org/kernel/defs.h xv6-bbj/kernel/defs.h differ
Files xv6-org/kernel/entry.S xv6-bbj/kernel/entry.S differ
Files xv6-org/kernel/file.c xv6-bbj/kernel/file.c differ
Files xv6-org/kernel/file.h xv6-bbj/kernel/file.h differ
Files xv6-org/kernel/kernel.ld xv6-bbj/kernel/kernel.ld differ
Files xv6-org/kernel/kernelvec.S xv6-bbj/kernel/kernelvec.S differ
Files xv6-org/kernel/main.c xv6-bbj/kernel/main.c differ
Files xv6-org/kernel/memlayout.h xv6-bbj/kernel/memlayout.h differ
Files xv6-org/kernel/plic.c xv6-bbj/kernel/plic.c differ
Files xv6-org/kernel/ramdisk.c xv6-bbj/kernel/ramdisk.c differ
Files xv6-org/kernel/riscv.h xv6-bbj/kernel/riscv.h differ
Files xv6-org/kernel/start.c xv6-bbj/kernel/start.c differ
Files xv6-org/kernel/syscall.c xv6-bbj/kernel/syscall.c differ
Files xv6-org/kernel/syscall.h xv6-bbj/kernel/syscall.h differ
Files xv6-org/kernel/sysfile.c xv6-bbj/kernel/sysfile.c differ
Files xv6-org/kernel/trap.c xv6-bbj/kernel/trap.c differ
Files xv6-org/kernel/uart.c xv6-bbj/kernel/uart.c differ
Files xv6-org/kernel/vm.c xv6-bbj/kernel/vm.c differ
Files xv6-org/user/user.h xv6-bbj/user/user.h differ
Files xv6-org/user/usertests.c xv6-bbj/user/usertests.c differ
Files xv6-org/user/usys.pl xv6-bbj/user/usys.pl differ

Only in xv6-bbj/kernel: adc.h
Only in xv6-bbj/kernel: adc_dev.c
Only in xv6-bbj/kernel: gpio.h
Only in xv6-bbj/kernel: gpio_dev.c
Only in xv6-bbj/kernel: i2c.h
Only in xv6-bbj/kernel: i2c_designware.c
Only in xv6-bbj/kernel: i2c_designware.h
Only in xv6-bbj/kernel: i2c_dev.c
Only in xv6-bbj/kernel: pwm.h
Only in xv6-bbj/kernel: pwm_dev.c
Only in xv6-bbj/kernel: uart_dev.c
Only in xv6-bbj/kernel: uart_dev.h
Only in xv6-bbj/kernel: spi.h
Only in xv6-bbj/kernel: spi_dev.c
Only in xv6-bbj: port
Only in xv6-bbj: script
Only in xv6-bbj/user: adctest.c
Only in xv6-bbj/user: gpiotest.c
Only in xv6-bbj/user: i2ctest.c
Only in xv6-bbj/user: pwmtest.c
Only in xv6-bbj/user: sleep.c
Only in xv6-bbj/user: spitest.c
Only in xv6-bbj/user: uarttest.c
Only in xv6-bbj/user: uptime.c
```

## 特徴

- `fs.img`は`ramdisk.h`としてバイト配列としてカーネルに組み込んでいる。

## ビルド

```
$ vi script
# SDカードへの書き込み部分をコメントアウト

$ ./script
...
generated fip.bin
./genimage: error while loading shared libraries: libconfuse.so.2: cannot open shared object file: No such file or directory

$ vi port/image/makeig.sh
./genimageの./を取り、~/bin/genimageを使うように変更

$ ./script
...
generated milkv-duo.img

$ ls port/image/output
boot.vfat  milkv-duo.img
```

[ビルドログ](bbj_log.txt)

```bash
make clean
make fs.img
# xxd -i fs.img > kernel/ramdisk.h
make kernel/kernel && echo -e "${RED}generated kernel.bin${RESET}"
# $(OBJCOPY) -S -O binary $K/kernel $K/kernel.bin

cd port/fip
./makefip.sh && echo -e "${RED}generated fip.bin${RESET}"

/* makefip.shの内容
./fiptool.py -v genfip \
        'fip.bin' \
        --MONITOR_RUNADDR="0x0000000080000000" \
        --BLCP_2ND_RUNADDR="0x0000000083f40000" \
        --CHIP_CONF='chip_conf.bin' \
        --NOR_INFO='FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF' \
        --NAND_INFO='00000000'\
        --BL2='bl2_debug.bin' \
        --BLCP_IMG_RUNADDR=0x05200200 \
        --BLCP_PARAM_LOADADDR=0 \
        --BLCP=empty.bin \
        --DDR_PARAM='ddr_param.bin' \
        --BLCP_2ND='empty.bin' \
        --MONITOR='../../kernel/kernel.bin' \
        --LOADER_2ND='bl33.bin' \
        --compress='lzma' ;
*/

cd ../image
PATH=$PATH:/sbin
rm output/* -f
./makeimg.sh && echo -e "${RED}generated milkv-duo.img${RESET}"

/* port/image/genimage.cfg
image boot.vfat {
	vfat {
		label = "boot"
		files = {
			"fip.bin",
		}
	}
	size = 10M
}

image milkv-duo.img {
	hdimage {
	}

	partition boot {
		partition-type = 0xC
		bootable = "true"
		image = "boot.vfat"
	}

}
*/
#sudo dd if=output/milkv-duo.img of=$1 status=progress &&
#sync && echo -e "${RED}sd card burnt${RESET}"
```

## 実行

```bash
C.SCS/0/0.WD.URPL.SDI/25000000/6000000.BS/SD.PS.SD/0x0/0x1000/0x1000/0.PE.BS.SD.
FSBL Jb28g9:gf91cf4331-dirty:2024-03-01T11:06:19+08:00
st_on_reason=d0000
st_off_reason=0
P2S/0x1000/0x3bc0dc00.
SD/0xcc00/0x1000/0x1000/0.P2E.
DPS/0xdc00/0x2000.
SD/0xdc00/0x2000/0x2000/0.DPE.
DDR init.
ddr_param[0]=0x78075562.
pkg_type=3
D3_1_4
DDR2-512M-QFN68
Data rate=1333.
DDR BIST PASS
PLLS.
PLLE.
C2S/0x0/0x0/0x0.
No C906L image.
MS/0xfc00/0x80000000/0x1fde00.
SD/0xfc00/0x1fde00/0x1fde00/0.ME.
L2/0x20da00.
SD/0x20da00/0x200/0x200/0.L2/0x414d3342/0xcafe5f0b/0x80500000/0x200/0x200
COMP/1.
SD/0x20da00/0x200/0x200/0.DCP/0x80500020/0x1000000/0x81500020/0x200/1.
DCP/0x0/0.
Loader_2nd loaded.
Use internal 32k
Jump to monitor at 0x80000000.
OPENSBI: next_addr=0x80500020 arg1=0x80080000

# 以降がxv6の出力
This system is ported by OStar Team.
xv6 kernel is booting on hart 0

pll_g6_ctrl:   0 0x0000000000000000
pll_g6_status: 70000 0x0000000000070000
fpll_csr:      7788101 0x0000000007788101
pll_g6_ssc_syn_ctrl: 2 0x0000000000000002
clk_en_0:      -1 0xffffffffffffffff
clk_en_1:      -1 0xffffffffffffffff
clk_en_2:      -1 0xffffffffffffffff
clk_en_3:      -1 0xffffffffffffffff
clk_en_4:      -1 0xffffffffffffffff
clk_byp_0:     0 0x0000000000000000
div_clk_axi6:  1 0x0000000000000001
div_clk_i2c:   1 0x0000000000000001
div_clk_pwm_src_0: f0009 0x00000000000f0009
div_clk_gpio_db: 1 0x0000000000000001
div_clk_spi: 1 0x0000000000000001

Initialized physical page allocator
Created kernel page table
Turned on paging
Initialized process table
Installed kernel trap vector
Set up interrupt controller
Set up buffer cache
Set up inode table
Set up file table
Initialized disk
Initialized i2c controller
Initialized uart controller
Initialized adc controller
Initialized pwm controller
Initialized gpio controller
Initialized spi controller
init: starting sh
$ ls
.              1 1 1024
..             1 1 1024
README         2 2 2305
cat            2 3 32952
echo           2 4 31832
forktest       2 5 15880
grep           2 6 36392
init           2 7 32296
kill           2 8 31744
ln             2 9 31568
ls             2 10 34896
mkdir          2 11 31808
rm             2 12 31800
sh             2 13 54376
stressfs       2 14 32680
usertests      2 15 180512
grind          2 16 47560
wc             2 17 33880
zombie         2 18 31176
sleep          2 19 31616
uptime         2 20 31584
i2ctest        2 21 35752
uarttest       2 22 33472
adctest        2 23 32568
pwmtest        2 24 32328
gpiotest       2 25 34936
spitest        2 26 33128
console        3 27 0
$ gpiotest
open gpio-2 success!
start flashing led
open gpio-0 success!
please change input at GP0
input: 1
input: 1
input: 1
input: 1
input: 1
input: 1
input: 1
input: 1
input: 1
input: 1
$ forktest
fork test
fork test OK
$ usertests
usertests starting
test copyin: OK
test copyout: OK
test copyinstr1: OK
test copyinstr2: OK
test copyinstr3: OK
test rwsbrk: OK
test truncate1: OK
test truncate2: OK
test truncate3: OK
test openiput: child exit
parent waiting
OK
test exitiput: OK
test iput: OK
test opentest: OK
test writetest: OK
test writebig: OK
test createtest: OK
test dirtest: OK
test exectest: OK
test pipe1: OK
test killstatus: OK
test preempt: kill... wait... OK
test exitwait: OK
test reparent: OK
test twochildren: OK
test forkfork: OK
test forkforkfork: OK
test reparent2: OK
test mem: OK
test sharedfd: OK
test fourfiles: OK
test createdelete: OK
test unlinkread: OK
test linktest: OK
test concreate: OK
test linkunlink: OK
test subdir: OK
test bigwrite: OK
test bigfile: OK
test fourteen: OK
test rmdot: OK
test dirfile: OK
test iref: OK
test forktest: OK
test sbrkbasic: OK
test sbrkmuch: OK
test kernmem: usertrap(): unexpected scause 0x000000000000000d pid=6539
            sepc=0x00000000000021ee stval=0x0000000080000000
usertrap(): unexpected scause 0x000000000000000d pid=6540
            sepc=0x00000000000021ee stval=0x000000008000c350
usertrap(): unexpected scause 0x000000000000000d pid=6541
            sepc=0x00000000000021ee stval=0x00000000800186a0
usertrap(): unexpected scause 0x000000000000000d pid=6542
            sepc=0x00000000000021ee stval=0x00000000800249f0
usertrap(): unexpected scause 0x000000000000000d pid=6543
            sepc=0x00000000000021ee stval=0x0000000080030d40
usertrap(): unexpected scause 0x000000000000000d pid=6544
            sepc=0x00000000000021ee stval=0x000000008003d090
usertrap(): unexpected scause 0x000000000000000d pid=6545
            sepc=0x00000000000021ee stval=0x00000000800493e0
usertrap(): unexpected scause 0x000000000000000d pid=6546
            sepc=0x00000000000021ee stval=0x0000000080055730
usertrap(): unexpected scause 0x000000000000000d pid=6547
            sepc=0x00000000000021ee stval=0x0000000080061a80
usertrap(): unexpected scause 0x000000000000000d pid=6548
            sepc=0x00000000000021ee stval=0x000000008006ddd0
usertrap(): unexpected scause 0x000000000000000d pid=6549
            sepc=0x00000000000021ee stval=0x000000008007a120
usertrap(): unexpected scause 0x000000000000000d pid=6550
            sepc=0x00000000000021ee stval=0x0000000080086470
usertrap(): unexpected scause 0x000000000000000d pid=6551
            sepc=0x00000000000021ee stval=0x00000000800927c0
usertrap(): unexpected scause 0x000000000000000d pid=6552
            sepc=0x00000000000021ee stval=0x000000008009eb10
usertrap(): unexpected scause 0x000000000000000d pid=6553
            sepc=0x00000000000021ee stval=0x00000000800aae60
usertrap(): unexpected scause 0x000000000000000d pid=6554
            sepc=0x00000000000021ee stval=0x00000000800b71b0
usertrap(): unexpected scause 0x000000000000000d pid=6555
            sepc=0x00000000000021ee stval=0x00000000800c3500
usertrap(): unexpected scause 0x000000000000000d pid=6556
            sepc=0x00000000000021ee stval=0x00000000800cf850
usertrap(): unexpected scause 0x000000000000000d pid=6557
            sepc=0x00000000000021ee stval=0x00000000800dbba0
usertrap(): unexpected scause 0x000000000000000d pid=6558
            sepc=0x00000000000021ee stval=0x00000000800e7ef0
usertrap(): unexpected scause 0x000000000000000d pid=6559
            sepc=0x00000000000021ee stval=0x00000000800f4240
usertrap(): unexpected scause 0x000000000000000d pid=6560
            sepc=0x00000000000021ee stval=0x0000000080100590
usertrap(): unexpected scause 0x000000000000000d pid=6561
            sepc=0x00000000000021ee stval=0x000000008010c8e0
usertrap(): unexpected scause 0x000000000000000d pid=6562
            sepc=0x00000000000021ee stval=0x0000000080118c30
usertrap(): unexpected scause 0x000000000000000d pid=6563
            sepc=0x00000000000021ee stval=0x0000000080124f80
usertrap(): unexpected scause 0x000000000000000d pid=6564
            sepc=0x00000000000021ee stval=0x00000000801312d0
usertrap(): unexpected scause 0x000000000000000d pid=6565
            sepc=0x00000000000021ee stval=0x000000008013d620
usertrap(): unexpected scause 0x000000000000000d pid=6566
            sepc=0x00000000000021ee stval=0x0000000080149970
usertrap(): unexpected scause 0x000000000000000d pid=6567
            sepc=0x00000000000021ee stval=0x0000000080155cc0
usertrap(): unexpected scause 0x000000000000000d pid=6568
            sepc=0x00000000000021ee stval=0x0000000080162010
usertrap(): unexpected scause 0x000000000000000d pid=6569
            sepc=0x00000000000021ee stval=0x000000008016e360
usertrap(): unexpected scause 0x000000000000000d pid=6570
            sepc=0x00000000000021ee stval=0x000000008017a6b0
usertrap(): unexpected scause 0x000000000000000d pid=6571
            sepc=0x00000000000021ee stval=0x0000000080186a00
usertrap(): unexpected scause 0x000000000000000d pid=6572
            sepc=0x00000000000021ee stval=0x0000000080192d50
usertrap(): unexpected scause 0x000000000000000d pid=6573
            sepc=0x00000000000021ee stval=0x000000008019f0a0
usertrap(): unexpected scause 0x000000000000000d pid=6574
            sepc=0x00000000000021ee stval=0x00000000801ab3f0
usertrap(): unexpected scause 0x000000000000000d pid=6575
            sepc=0x00000000000021ee stval=0x00000000801b7740
usertrap(): unexpected scause 0x000000000000000d pid=6576
            sepc=0x00000000000021ee stval=0x00000000801c3a90
usertrap(): unexpected scause 0x000000000000000d pid=6577
            sepc=0x00000000000021ee stval=0x00000000801cfde0
usertrap(): unexpected scause 0x000000000000000d pid=6578
            sepc=0x00000000000021ee stval=0x00000000801dc130
OK
test MAXVAplus: usertrap(): unexpected scause 0x000000000000000f pid=6580
            sepc=0x000000000000229a stval=0x0000004000000000
usertrap(): unexpected scause 0x000000000000000f pid=6581
            sepc=0x000000000000229a stval=0xffffff8000000000
usertrap(): unexpected scause 0x000000000000000f pid=6582
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6583
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6584
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6585
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6586
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6587
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6588
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6589
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6590
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6591
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6592
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6593
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6594
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6595
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6596
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6597
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6598
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6599
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6600
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6601
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6602
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6603
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6604
            sepc=0x000000000000229a stval=0x0000000000000000
usertrap(): unexpected scause 0x000000000000000f pid=6605
            sepc=0x000000000000229a stval=0x0000000000000000
OK
test sbrkfail: usertrap(): unexpected scause 0x000000000000000d pid=6617
            sepc=0x00000000000049b0 stval=0x0000000000013000
OK
test sbrkarg: OK
test validatetest: OK
test bsstest: OK
test bigargtest: OK
test argptest: OK
test stacktest: usertrap(): unexpected scause 0x000000000000000d pid=6625
            sepc=0x000000000000240c stval=0x0000000000010ed0
OK
test textwrite: usertrap(): unexpected scause 0x000000000000000f pid=6627
            sepc=0x000000000000248c stval=0x0000000000000000
OK
test pgbug: OK
test sbrkbugs: usertrap(): unexpected scause 0x000000000000000c pid=6630
            sepc=0x0000000000005c3c stval=0x0000000000005c3c
usertrap(): unexpected scause 0x000000000000000c pid=6631
            sepc=0x0000000000005c3c stval=0x0000000000005c3c
OK
test sbrklast: OK
test sbrk8000: OK
test badarg: OK
usertests slow tests starting
test bigdir: OK
test manywrites: OK
test badwrite: OK
test execout: OK
test diskfull: balloc: out of blocks
ialloc: no inodes
ialloc: no inodes
OK
test outofinodes: ialloc: no inodes
OK
ALL TESTS PASSED
$
```

# `bl33.bin`を作成して`milkv-duo.img`を再作成

"OpenSBIを使って自作OSを起動する"にしたがい`bl33`を作成し、`port/fip/bl33.bin`を置き換えて
`milkv-duo.img`を再作成する。

```bash
$ vi start.S
$ cat start.S
    .section .text
    .global _start
_start:
    /* BL33 information */
    j real_start
    .balign 4
    .word 0x33334c42  /* b'BL33' */
    .word 0xdeadbeea  /* CKSUM */
    .word 0xdeadbeeb  /* SIZE */
    .quad 0x80500000  /* RUNADDR */
    .word 0xdeadbeec
    .balign 4
    j real_start
    .balign 4
    /* BL33 end */
real_start:
$ ~/riscv-gnu-toolchain/bin/riscv64-unknown-elf-gcc -nostdlib -fno-builtin -march=rv64gc -mabi=lp64f -g -Wall -Ttext=0x80200000 -o bl33.elf start.S
$ ~/riscv-gnu-toolchain/bin/riscv64-unknown-elf-objcopy -O binary bl33.elf bl33.bin
$ xxd bl33.bin
00000000: 05a0 0100 424c 3333 eabe adde ebbe adde  ....BL33........
00000010: 0000 5080 0000 0000 ecbe adde 11a0 0100  ..P.............
$ cp xv6-bbj/port/fip/bl33.bin xv6-bbj/port/fip/bl33.bin.org
$ cp bl33.bin xv6-bbj/port/fip/
$ cd xv6-bbj
$ ./script
# port/image/out/milkv-duo.img をSDカードに
$ minicom
C.SCS/0/0.WD.URPL.SDI/25000000/6000000.BS/SD.PS.SD/0x0/0x1000/0x1000/0.PE.BS.SD.
FSBL Jb28g9:gf91cf4331-dirty:2024-03-01T11:06:19+08:00
st_on_reason=d0000
st_off_reason=0
P2S/0x1000/0x3bc0dc00.
SD/0xcc00/0x1000/0x1000/0.P2E.
DPS/0xdc00/0x2000.
SD/0xdc00/0x2000/0x2000/0.DPE.
DDR init.
ddr_param[0]=0x78075562.
pkg_type=3
D3_1_4
DDR2-512M-QFN68
Data rate=1333.
DDR BIST PASS
PLLS.
PLLE.
C2S/0x0/0x0/0x0.
No C906L image.
MS/0xfc00/0x80000000/0x1fde00.
SD/0xfc00/0x1fde00/0x1fde00/0.ME.
L2/0x20da00.
SD/0x20da00/0x200/0x200/0.L2/0x414d3342/0xcafe5f0b/0x80500000/0x200/0x200
COMP/1.
SD/0x20da00/0x200/0x200/0.DCP/0x80500020/0x1000000/0x81500020/0x200/1.
DCP/0x0/0.
Loader_2nd loaded.
Use internal 32k
Jump to monitor at 0x80000000.
OPENSBI: next_addr=0x80500020 arg1=0x80080000

This system is ported by OStar Team.
xv6 kernel is booting on hart 0

pll_g6_ctrl:   0 0x0000000000000000
pll_g6_status: 70000 0x0000000000070000
fpll_csr:      7788101 0x0000000007788101
pll_g6_ssc_syn_ctrl: 2 0x0000000000000002
clk_en_0:      -1 0xffffffffffffffff
clk_en_1:      -1 0xffffffffffffffff
clk_en_2:      -1 0xffffffffffffffff
clk_en_3:      -1 0xffffffffffffffff
clk_en_4:      -1 0xffffffffffffffff
clk_byp_0:     0 0x0000000000000000
div_clk_axi6:  1 0x0000000000000001
div_clk_i2c:   1 0x0000000000000001
div_clk_pwm_src_0: f0009 0x00000000000f0009
div_clk_gpio_db: 1 0x0000000000000001
div_clk_spi: 1 0x0000000000000001

Initialized physical page allocator
Created kernel page table
Turned on paging
Initialized process table
Installed kernel trap vector
Set up interrupt controller
Set up buffer cache
Set up inode table
Set up file table
Initialized disk
Initialized i2c controller
Initialized uart controller
Initialized adc controller
Initialized pwm controller
Initialized gpio controller
Initialized spi controller
init: starting sh
$ ls
.              1 1 1024
..             1 1 1024
README         2 2 2305
...
```

オリジナル同様、問題なく動いた。
