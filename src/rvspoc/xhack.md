# xhack版

```bash
$ diff -qr xv6-org xv6-xhack
Files xv6-org/Makefile and xv6-xhack/Makefile differ
Files xv6-org/README and xv6-xhack/README differ
Files xv6-org/kernel/bio.c and xv6-xhack/kernel/bio.c differ
Files xv6-org/kernel/console.c and xv6-xhack/kernel/console.c differ
Files xv6-org/kernel/defs.h and xv6-xhack/kernel/defs.h differ
Files xv6-org/kernel/entry.S and xv6-xhack/kernel/entry.S differ
Files xv6-org/kernel/exec.c and xv6-xhack/kernel/exec.c differ
Files xv6-org/kernel/file.c and xv6-xhack/kernel/file.c differ
Files xv6-org/kernel/file.h and xv6-xhack/kernel/file.h differ
Files xv6-org/kernel/kalloc.c and xv6-xhack/kernel/kalloc.c differ
Files xv6-org/kernel/kernelvec.S and xv6-xhack/kernel/kernelvec.S differ
Files xv6-org/kernel/main.c and xv6-xhack/kernel/main.c differ
Files xv6-org/kernel/memlayout.h and xv6-xhack/kernel/memlayout.h differ
Files xv6-org/kernel/param.h and xv6-xhack/kernel/param.h differ
Files xv6-org/kernel/plic.c and xv6-xhack/kernel/plic.c differ
Files xv6-org/kernel/printf.c and xv6-xhack/kernel/printf.c differ
Files xv6-org/kernel/proc.c and xv6-xhack/kernel/proc.c differ
Files xv6-org/kernel/ramdisk.c and xv6-xhack/kernel/ramdisk.c differ
Files xv6-org/kernel/riscv.h and xv6-xhack/kernel/riscv.h differ
Files xv6-org/kernel/start.c and xv6-xhack/kernel/start.c differ
Files xv6-org/kernel/swtch.S and xv6-xhack/kernel/swtch.S differ
Files xv6-org/kernel/syscall.c and xv6-xhack/kernel/syscall.c differ
Files xv6-org/kernel/syscall.h and xv6-xhack/kernel/syscall.h differ
Files xv6-org/kernel/sysfile.c and xv6-xhack/kernel/sysfile.c differ
Files xv6-org/kernel/trampoline.S and xv6-xhack/kernel/trampoline.S differ
Files xv6-org/kernel/trap.c and xv6-xhack/kernel/trap.c differ
Files xv6-org/kernel/types.h and xv6-xhack/kernel/types.h differ
Files xv6-org/kernel/uart.c and xv6-xhack/kernel/uart.c differ
Files xv6-org/kernel/vm.c and xv6-xhack/kernel/vm.c differ
Files xv6-org/user/user.h and xv6-xhack/user/user.h differ
Files xv6-org/user/usertests.c and xv6-xhack/user/usertests.c differ
Files xv6-org/user/usys.pl and xv6-xhack/user/usys.pl differ

Only in xv6-xhack: build_for_milkv_duo.sh
Only in xv6-xhack: duo-imgtools
Only in xv6-xhack/kernel: adc.c
Only in xv6-xhack/kernel: adc.h
Only in xv6-xhack/kernel: config.h
Only in xv6-xhack/kernel: config.h.cv1800b
Only in xv6-xhack/kernel: config.h.th1520
Only in xv6-xhack/kernel: gpio.c
Only in xv6-xhack/kernel: gpio.h
Only in xv6-xhack/kernel: i2c.c
Only in xv6-xhack/kernel: i2c.h
Only in xv6-xhack/kernel: io.h
Only in xv6-org/kernel: kernel.ld
Only in xv6-xhack/kernel: kernel.ld.S
Only in xv6-xhack/kernel: pwm.c
Only in xv6-xhack/kernel: pwm.h
Only in xv6-xhack/kernel: ramdisk_data.S
Only in xv6-xhack/kernel: sbi.c
Only in xv6-xhack/kernel: sbi.h
Only in xv6-xhack/kernel: spi.c
Only in xv6-xhack/kernel: spi.h
Only in xv6-xhack/user: adc.c
Only in xv6-xhack/user: blink.c
Only in xv6-xhack/user: i2c.c
Only in xv6-xhack/user: pwm.c
Only in xv6-xhack/user: spi.c
Only in xv6-xhack/user: strtoul.c
```

## ビルド

```bash
$ ./build_for_milkv_duo.sh

 # duo-buildroot-sdk/host-tools/gcc/riscv64-elf-x86_64/binはLDFLAGSのno-warn-rwx-segmentsがなくてエラー
 # riscv-gnu-toolchain/binを使用
./genimage.sh: line 44: genimage: command not found
$ cp duo-buildroot-sdk/buildroot-2021.05/output/milkv-duo-sd_musl_riscv64/build/host-genimage-14/genimage ~/bin

$ ./build_for_milkv_duo.sh
```

[ビルドログ](xhack_log.txt)

```bash
cp kernel/config.h.cv1800b kernel/config.h
# xv6カーネルの作成
make
make kernel/kernel.bin
cp kernel/kernel.bin duo-imgtools/
cd duo-imgtools
# u-bootの作成
mkimage -f xv6.its boot.sd
# sdイメージの作成
./genimage.sh -c xv6.cfg
```

## 実行

```bash
# 以降がxv6の出力
xv6 kernel is booting

SBI specification v0.3 detected
SBI TIME extension detected
SBI IPI extension detected
SBI RFNC extension detected
SBI HSM extension detected

usertrap(): unexpected scause 0xd pid=1
            sepc=0 stval=0x505050505
panic: init exiting
```

- パッチ適用後

```bash
C.SCS/0/0.WD.URPL.SDI/25000000/6000000.BS/SD.PS.SD/0x0/0x1000/0x1000/0.PE.BS.SD.
FSBL Jb28g9:gf2df47913:2024-02-29T16:35:38+08:00
st_on_reason=d0000
st_off_reason=0
P2S/0x1000/0x3bc0da00.
SD/0xca00/0x1000/0x1000/0.P2E.
DPS/0xda00/0x2000.
SD/0xda00/0x2000/0x2000/0.DPE.
DDR init.
ddr_param[0]=0x78075562.
pkg_type=3
D3_1_4
DDR2-512M-QFN68
Data rate=1333.
DDR BIST PASS
PLLS.
PLLE.
C2S/0xfa00/0x83f40000/0x3600.
 2RET.:00/0x3600/0x3600/0.RSC.
[M1S./405x01939080]0P/r0ex 8s0y0s0t0e0m0 0i/n0ixt1 bd0o0n0e.

RT: [1.457318]CVIRTOS Build Date:Feb 29 2024  (Time :16:35:37)
RT: [1.463321]Post system init done
RT: [1.466636]create cvi task
RT: [1.469462][cvi_spinlock_init] succeess
RT: [1.473356]prvCmdQuRunTask run
SD/0x13000/0x1b000/0x1b000/0.ME.
L2/0x2e000.
SD/0x2e000/0x200/0x200/0.L2/0x414d3342/0xcafebfd8/0x80200000/0x23000/0x23000
COMP/1.
SD/0x2e000/0x23000/0x23000/0.DCP/0x80200020/0x1000000/0x81500020/0x23000/1.
DCP/0x4a307/0.
Loader_2nd loaded.
Use internal 32k
Jump to monitor at 0x80000000.
OPENSBI: next_addr=0x80200020 arg1=0x80080000
OpenSBI v0.9
   ____                    _____ ____ _____
  / __ \                  / ____|  _ \_   _|
 | |  | |_ __   ___ _ __ | (___ | |_) || |
 | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
 | |__| | |_) |  __/ | | |____) | |_) || |_
  \____/| .__/ \___|_| |_|_____/|____/_____|
        | |
        |_|

Platform Name             : Milk-V Duo
Platform Features         : mfdeleg
Platform HART Count       : 1
Platform IPI Device       : clint
Platform Timer Device     : clint
Platform Console Device   : uart8250
Platform HSM Device       : ---
Platform SysReset Device  : ---
Firmware Base             : 0x80000000
Firmware Size             : 132 KB
Runtime SBI Version       : 0.3

Domain0 Name              : root
Domain0 Boot HART         : 0
Domain0 HARTs             : 0*
Domain0 Region00          : 0x0000000074000000-0x000000007400ffff (I)
Domain0 Region01          : 0x0000000080000000-0x000000008003ffff ()
Domain0 Region02          : 0x0000000000000000-0xffffffffffffffff (R,W,X)
Domain0 Next Address      : 0x0000000080200020
Domain0 Next Arg1         : 0x0000000080080000
Domain0 Next Mode         : S-mode
Domain0 SysReset          : yes

Boot HART ID              : 0
Boot HART Domain          : root
Boot HART ISA             : rv64imafdcvsux
Boot HART Features        : scounteren,mcounteren,time
Boot HART PMP Count       : 16
Boot HART PMP Granularity : 4096
Boot HART PMP Address Bits: 38
Boot HART MHPM Count      : 8
Boot HART MHPM Count      : 8
Boot HART MIDELEG         : 0x0000000000000222
Boot HART MEDELEG         : 0x000000000000b109


U-Boot 2021.10 (Feb 29 2024 - 16:53:07 +0800)cvitek_cv180x

DRAM:  63.3 MiB
gd->relocaddr=0x83ef8000. offset=0x3cf8000
MMC:   cv-sd@4310000: 0
Loading Environment from <NULL>... OK
In:    serial
Out:   serial
Err:   serial
Net:
Warning: ethernet@4070000 (eth0) using random MAC address - 5a:68:14:99:8e:c2
eth0: ethernet@4070000
Hit any key to stop autoboot:  0
Boot from SD ...
switch to partitions #0, OK
mmc0 is current device
1578445 bytes read in 71 ms (21.2 MiB/s)
## Loading kernel from FIT Image at 81400000 ...
   Using 'config-cv1800b_milkv_duo_sd' configuration
   Trying 'kernel-1' kernel subimage
   Verifying Hash Integrity ... crc32+ OK
## Loading fdt from FIT Image at 81400000 ...
   Using 'config-cv1800b_milkv_duo_sd' configuration
   Trying 'fdt-1' fdt subimage
   Verifying Hash Integrity ... sha256+ OK
   Booting using the fdt blob at 0x8158015c
   Loading Kernel Image
   Decompressing 1572864 bytes used 2ms
   Loading Device Tree to 00000000836ac000, end 00000000836aff73 ... OK

Starting kernel ...

# 以降がxv6の出力
xv6 kernel is booting

SBI specification v0.3 detected
SBI TIME extension detected
SBI IPI extension detected
SBI RFNC extension detected
SBI HSM extension detected

init: starting sh
$ ls
.              1 1 1024
..             1 1 1024
README         2 2 4571
cat            2 3 36392
echo           2 4 34992
forktest       2 5 16336
grep           2 6 40104
init           2 7 35504
kill           2 8 35016
ln             2 9 34896
ls             2 10 38496
mkdir          2 11 35048
rm             2 12 35032
sh             2 13 58696
stressfs       2 14 36080
usertests      2 15 166600
grind          2 16 49408
wc             2 17 37128
zombie         2 18 34520
blink          2 19 36000
pwm            2 20 35784
adc            2 21 35344
i2c            2 22 36216
spi            2 23 36224
console        3 24 0
$ forktest
fork test
fork test OK
$ blink
./blink LOOPS_CNT
$ blink 5
$
```
