# virus_v版

```bash
$ diff -qr xv6-org/kernel xv6-virusv/kernel
Files xv6-org/kernel/bio.c and xv6-virusv/kernel/bio.c differ
Files xv6-org/kernel/console.c and xv6-virusv/kernel/console.c differ
Files xv6-org/kernel/defs.h and xv6-virusv/kernel/defs.h differ
Files xv6-org/kernel/entry.S and xv6-virusv/kernel/entry.S differ
Files xv6-org/kernel/file.h and xv6-virusv/kernel/file.h differ
Files xv6-org/kernel/kernel.ld and xv6-virusv/kernel/kernel.ld differ
Files xv6-org/kernel/kernelvec.S and xv6-virusv/kernel/kernelvec.S differ
Files xv6-org/kernel/main.c and xv6-virusv/kernel/main.c differ
Files xv6-org/kernel/memlayout.h and xv6-virusv/kernel/memlayout.h differ
Files xv6-org/kernel/param.h and xv6-virusv/kernel/param.h differ
Files xv6-org/kernel/plic.c and xv6-virusv/kernel/plic.c differ
Files xv6-org/kernel/proc.c and xv6-virusv/kernel/proc.c differ
Files xv6-org/kernel/ramdisk.c and xv6-virusv/kernel/ramdisk.c differ
Files xv6-org/kernel/riscv.h and xv6-virusv/kernel/riscv.h differ
Files xv6-org/kernel/start.c and xv6-virusv/kernel/start.c differ
Files xv6-org/kernel/trampoline.S and xv6-virusv/kernel/trampoline.S differ
Files xv6-org/kernel/trap.c and xv6-virusv/kernel/trap.c differ
Files xv6-org/kernel/types.h and xv6-virusv/kernel/types.h differ
Files xv6-org/kernel/vm.c and xv6-virusv/kernel/vm.c differ

Only in xv6-virusv/kernel: device.c
Only in xv6-virusv/kernel: gpio.c
Only in xv6-virusv/kernel: hal_dw_i2c.c
Only in xv6-virusv/kernel: hal_dw_i2c.h
Only in xv6-virusv/kernel: i2c.c
Only in xv6-virusv/kernel: i2c.h
Only in xv6-virusv/kernel: intr_conf.h
Only in xv6-virusv/kernel: pwm.c
Only in xv6-virusv/kernel: sbi.h
Only in xv6-virusv/kernel: spi.c
Only in xv6-virusv/kernel: spilcd.c
Only in xv6-virusv/kernel: top_reg.h
Only in xv6-virusv/kernel: uart8250.c
Only in xv6-virusv/kernel: uart8250.h
```

## 主な変更点

- シングルコア
- timerはSBIを使用
- レジスタアドレスが異なる (データシート: 1.5 アドレス空間マッピングを参照)
- `riscv.h`を大幅改定

## ビルド

```
$ make duo
...
kernel/vm.c: Assembler messages:
kernel/vm.c:77: Error: unrecognized opcode `dcache.ciall'
kernel/vm.c:78: Error: unrecognized opcode `icache.iall'
make: *** [<builtin>: kernel/vm.o] Error 1

# toolchainをhost-toolsのものに変更

$ make clean
$ make duo
...
./genimage: error while loading shared libraries: libconfuse.so.2: cannot open shared object file: No such file or directory
make: *** [Makefile:191: duo] Error 127

$ vi vendor/milkv-duo/images/mgen_images.sh
./genimageの./を取り、~/bin/genimageを使うように変更

$ make clean ; make duo
...
INFO: hdimage(milkv-duo.img): adding partition 'boot' (in MBR) from 'boot.vfat' ...
INFO: hdimage(milkv-duo.img): writing MBR

$ ls vendor/milkv-duo/images
boot.vfat  genimage-milkv-duo.cfg  milkv-duo.img  tmp
genimage   gen_images.sh           root
```

[ビルドログ](virusv_log.txt)

```makefile
duo: $K/kernel fs.img
    cp $K/kernel.bin vendor/milkv-duo/boot.sd/kernel.bin
    cp fs.img vendor/milkv-duo/boot.sd/fs.img
    cd vendor/milkv-duo/fip && ./gen_fip.sh
    cd vendor/milkv-duo/boot.sd && ./gen_bootsd.sh
    cd vendor/milkv-duo/images && ./gen_images.sh
```

## 実行

```bash
C.SCS/0/0.WD.URPL.SDI/25000000/6000000.BS/SD.PS.SD/0x0/0x1000/0x1000/0.PE.BS.SD.
FSBL Jb28g9:gd1d59d9af-dirty:2023-12-22T16:12:06+08:00
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
DDR BIST PASS
PLLS.
PLLE.
C2S/0xfa00/0x83f40000/0x3600.
 2RET.:00/0x3600/0x3600/0.RSC.
[M1S./403x71039080]0P/r0ex 8s0y0s0t0e0m0 0i/n0ixt1 bd0o0n0e.

RT: [1.443418]CVIRTOS Build Date:Dec 20 2023  (Time :17:25:52)
RT: [1.449421]Post system init done
RT: [1.452737]create cvi task
RT: [1.455563][cvi_spinlock_init] succeess
RT: [1.459457]prvCmdQuRunTask run
SD/0x13000/0x1b000/0x1b000/0.ME.
L2/0x2e000.
SD/0x2e000/0x200/0x200/0.L2/0x414d3342/0xcafeb9d6/0x80200000/0x1f800/0x1f800
COMP/1.
SD/0x2e000/0x1f800/0x1f800/0.DCP/0x80200020/0x1000000/0x81500020/0x1f800/1.
DCP/0x425e9/0.
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

Platform Name             : Cvitek. CV180X ASIC. C906.
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


U-Boot 2021.10 (Dec 20 2023 - 17:25:42 +0800)cvitek_cv180x

DRAM:  63.3 MiB
gd->relocaddr=0x83f02000. offset=0x3d02000
MMC:   cv-sd@4310000: 0
Loading Environment from <NULL>... OK
In:    serial
Out:   serial
Err:   serial
Net:
Warning: ethernet@4070000 (eth0) using random MAC address - ae:27:f2:17:c2:fb
eth0: ethernet@4070000
Hit any key to stop autoboot:  0
Boot from SD ...
switch to partitions #0, OK
mmc0 is current device
2087112 bytes read in 94 ms (21.2 MiB/s)
## Loading kernel from FIT Image at 81400000 ...
   Using 'config-cv1800b_milkv_duo_sd' configuration
   Trying 'kernel-1' kernel subimage
   Verifying Hash Integrity ... crc32+ OK
## Loading ramdisk from FIT Image at 81400000 ...
   Using 'config-cv1800b_milkv_duo_sd' configuration
   Trying 'ramdisk-1' ramdisk subimage
   Verifying Hash Integrity ... crc32+ OK
   Loading ramdisk from 0x814046d4 to 0x80280000
## Loading fdt from FIT Image at 81400000 ...
   Using 'config-cv1800b_milkv_duo_sd' configuration
   Trying 'fdt-cv1800b_milkv_duo_sd' fdt subimage
   Verifying Hash Integrity ... sha256+ OK
   Booting using the fdt blob at 0x815f87d4
   Uncompressing Kernel Image
   Decompressing 39452 bytes used 6ms
   Loading Ramdisk to 834c6000, end 836ba000 ... OK
   Loading Device Tree to 00000000834be000, end 00000000834c5b80 ... OK

Starting kernel ...

# 以降がxv6の出力
xv6 kernel is booting

helloworld! IEEE80211 TEAM
SPI CTRLR0: 0x7
init: starting sh
$ ls
.              1 1 1024
..             1 1 1024
README         2 2 2305
cat            2 3 33528
echo           2 4 32368
forktest       2 5 16040
grep           2 6 36856
init           2 7 32832
kill           2 8 32384
ln             2 9 32200
ls             2 10 35408
mkdir          2 11 32464
rm             2 12 32448
sh             2 13 54848
stressfs       2 14 33168
usertests      2 15 182488
grind          2 16 48368
wc             2 17 34472
zombie         2 18 31840
gpiotest       2 19 32984
pwmtest        2 20 33056
spilcdtest     2 21 39592
console        3 22 0
$ forktest
fork test
fork test OK
$
```
