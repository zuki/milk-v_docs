
# [fiptool](https://github.com/sophgo/fiptool)を試す

## 1. macで試す

```bash
$ git clone -o upstream https://github.com/u-boot/u-boot.git
$ git clone -o upstream https://github.com/riscv-software-src/opensbi.git
$ git clone -o upstream https://github.com/sophgo/fiptool.git
$ brew install riscv64-elf-gcc riscv64-elf-binutils riscv64-elf-gdb

$ cd u-boot
$ make milkv_duo_defconfig
Makefile:40: *** missing separator.  Stop.
$ gmake milkv_duo_defconfig
  HOSTCC  scripts/basic/fixdep
  HOSTCC  scripts/kconfig/conf.o
  YACC    scripts/kconfig/zconf.tab.c
  LEX     scripts/kconfig/zconf.lex.c
  HOSTCC  scripts/kconfig/zconf.tab.o
  HOSTLD  scripts/kconfig/conf
#
# configuration written to .config
#
$ gmake
...
  LD      lib/efi_loader/boothart_efi.so
riscv64-elf-ld.bfd: warning: -z nocombreloc ignored
riscv64-elf-ld.bfd: warning: -z norelro ignored
riscv64-elf-ld.bfd: -shared not supported
gmake[2]: *** [scripts/Makefile.lib:515: lib/efi_loader/boothart_efi.so] Error 1
gmake[1]: *** [scripts/Makefile.build:397: lib/efi_loader] Error 2
gmake: *** [Makefile:1895: lib] Error 2
$ vi scripts/Makefile.lib
cmd_efi_ld = $(LD) $(KBUILD_EFILDFLAGS) -T $(EFI_LDS_PATH) \
        -shared -Bsymbolic -s $^ -o $@
        ^^^^^^^ この-sharedを削除
$ gmake
...
  LD      lib/efi_loader/boothart_efi.so
riscv64-elf-ld.bfd: warning: -z nocombreloc ignored
riscv64-elf-ld.bfd: warning: -z norelro ignored
riscv64-elf-ld.bfd: warning: cannot find entry symbol _start; defaulting to 0000000000000000
riscv64-elf-ld.bfd: lib/efi_loader/efi_crt0.o: in function `section_table':
(.text.head+0x1010): undefined reference to `_DYNAMIC'
gmake[2]: *** [scripts/Makefile.lib:515: lib/efi_loader/boothart_efi.so] Error 1
gmake[1]: *** [scripts/Makefile.build:397: lib/efi_loader] Error 2
gmake: *** [Makefile:1895: lib] Error 2
```

- `riscv-gnu-toolchain`はコンパイルエラーが出てインストールできず。
- `air m3`では別のエラー (`openssl/evp.h`が見つからない)でやはりビルドできず。

**Macでの処理は諦める**

# ubuntuで試す

```bash
$ sudo apt install gcc-riscv64-unknown-elf
# /usr/bin/riscv64-unknown-elf-gcc      # v.10.2.0
$ sudo apt install gcc-riscv64-linux-gnu
# /usr/bin/riscv64-linux-gnu-gcc        # 11.4.0
$ export CROSS_COMPLE=riscv64-unknown-elf-
$ git clone -o upstream https://github.com/u-boot/u-boot.git
$ git clone -o upstream https://github.com/riscv-software-src/opensbi.git
$ git clone -o upstream https://github.com/sophgo/fiptool.git
$ cd u-boot
$ make milkv_duo_defconfig
  HOSTCC  scripts/basic/fixdep
  HOSTCC  scripts/kconfig/conf.o
  HOSTCC  scripts/kconfig/zconf.tab.o
  HOSTLD  scripts/kconfig/conf
#
# configuration written to .config
#
$ make
...
tools/mkeficapsule.c:21:10: fatal error: gnutls/gnutls.h: No such file or directory
   21 | #include <gnutls/gnutls.h>
      |          ^~~~~~~~~~~~~~~~~
compilation terminated.
make[1]: *** [scripts/Makefile.host:95: tools/mkeficapsule] Error 1
make: *** [Makefile:1895: tools] Error 2
$ sudo apt install libgnutls28-dev
$ make
  LD      lib/efi_loader/boothart_efi.so
riscv64-unknown-elf-ld.bfd: warning: -z nocombreloc ignored
riscv64-unknown-elf-ld.bfd: warning: -z norelro ignored
riscv64-unknown-elf-ld.bfd: -shared not supported
make[2]: *** [scripts/Makefile.lib:515: lib/efi_loader/boothart_efi.so] Error 1
make[1]: *** [scripts/Makefile.build:397: lib/efi_loader] Error 2
make: *** [Makefile:1895: lib] Error 2

$ make clean
# ツールチェーンを変える
$ export CROSS_COMPLE=riscv64-linux-gnu-
$ cd u-boot
$ make milkv_duo_defconfig
$ make
$ ls
u-boot.bin u-boot-dtb.bin
$ cd ../opensbi
$ make PLATFORM=generic FW_FDT_PATH=../u-boot/u-boot.dtb
$ ls build/platform/generic/firmware/
fw_dynamic.bin      fw_dynamic.o     fw_jump.elf.ld  fw_payload.elf.dep
fw_dynamic.dep      fw_jump.bin      fw_jump.o       fw_payload.elf.ld
fw_dynamic.elf      fw_jump.dep      fw_payload.bin  fw_payload.o
fw_dynamic.elf.dep  fw_jump.elf      fw_payload.dep  payloads
fw_dynamic.elf.ld   fw_jump.elf.dep  fw_payload.elf
$ cd ../hiptool
$ cp ../u-boot/u-boot.bin data
$ cp ../opensbi/build/platform/generic/firmware/fw_dynamic.bin data
$ make BOARD=64M
build fip for board: 64M
./fiptool \
	--fsbl data/fsbl/cv180x.bin \
	--ddr_param data/ddr_param.bin \
	--opensbi data/fw_dynamic.bin \
	--uboot data/u-boot.bin \
	--rtos data/cvirtos.bin
Packing Param1...
Param1:    0x0
FSBL:      0x1000
Packing Param2...
Param2:    0xca00
DDR Param: 0xda00
RTOS:      0xfa00
OpenSBI:   0x13000
U-Boot:    0x55a00
Success
$ ls
data  fip.bin  fiptool  LICENSE  Makefile  README.md
```

- [u-bootビルドログ](uboot_build_log.txt)
- [OpenSBIビルドログ](opensbi_build_log.txt)

## 実行

```bash
C.SCS/0/0.WD.URPL.SDI/25000000/6000000.BS/SD.PS.SD/0x0/0x1000/0x1000/0.PE.BS.SD.
FSBL Jb28g9:gc737560cc-dirty:2024-03-04T15:33:50+08:00
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
[M1S./409x71331040]0P/r0ex 8s0y0s0t0e0m0 0i/n0ixt4 2dao0n0e.

RT: [1.503634]CVIRTOS Build Date:Apr  7 2024  (Time :18:32:34)
RT: [1.509637]Post system init done
RT: [1.512952]create cvi task
RT: [1.515779][cvi_spinlock_init] succeess
RT: [1.519672]prvCmdQuRunTask run
SD/0x13000/0x42a00/0x42a00/0.ME.
L2/0x55a00.
SD/0x55a00/0x200/0x200/0.L2/0x414d3342/0xcafe4f59/0x801fffe0/0x39a00/0x39a00
COMP/1.
SD/0x55a00/0x39a00/0x39a00/0.DCP/0x80200000/0x1000000/0x81500020/0x39a00/1.
DCP/0x773d6/0.
Loader_2nd loaded.
Use internal 32k
Jump to monitor at 0x80000000.
OPENSBI: next_addr=0x80200000 arg1=0x80080000
OpenSBI v1.5-52-gc4940a9
   ____                    _____ ____ _____
  / __ \                  / ____|  _ \_   _|
 | |  | |_ __   ___ _ __ | (___ | |_) || |
 | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
 | |__| | |_) |  __/ | | |____) | |_) || |_
  \____/| .__/ \___|_| |_|_____/|____/_____|
        | |
        |_|

Platform Name             : Milk-V Duo
Platform Features         : medeleg
Platform HART Count       : 1
Platform IPI Device       : aclint-mswi
Platform Timer Device     : aclint-mtimer @ 25000000Hz
Platform Console Device   : uart8250
Platform HSM Device       : ---
Platform PMU Device       : thead,c900-pmu
Platform Reboot Device    : ---
Platform Shutdown Device  : ---
Platform Suspend Device   : ---
Platform CPPC Device      : ---
Firmware Base             : 0x80000000
Firmware Size             : 327 KB
Firmware RW Offset        : 0x40000
Firmware RW Size          : 71 KB
Firmware Heap Offset      : 0x49000
Firmware Heap Size        : 35 KB (total), 2 KB (reserved), 11 KB (used), 21 KB)
Firmware Scratch Size     : 4096 B (total), 416 B (used), 3680 B (free)
Runtime SBI Version       : 2.0

Domain0 Name              : root
Domain0 Boot HART         : 0
Domain0 HARTs             : 0*
Domain0 Region00          : 0x0000000004140000-0x0000000004140fff M: (I,R,W) S/)
Domain0 Region01          : 0x0000000074008000-0x000000007400bfff M: (I,R,W) S/)
Domain0 Region02          : 0x0000000074000000-0x0000000074007fff M: (I,R,W) S/)
Domain0 Region03          : 0x0000000080040000-0x000000008005ffff M: (R,W) S/U:)
Domain0 Region04          : 0x0000000080000000-0x000000008003ffff M: (R,X) S/U:)
Domain0 Region05          : 0x0000000070000000-0x0000000073ffffff M: (I,R,W) S/)
Domain0 Region06          : 0x0000000000000000-0xffffffffffffffff M: () S/U: (R)
Domain0 Next Address      : 0x0000000080200000
Domain0 Next Arg1         : 0x0000000080080000
Domain0 Next Mode         : S-mode
Domain0 SysReset          : yes
Domain0 SysSuspend        : yes

Boot HART ID              : 0
Boot HART Domain          : root
Boot HART Priv Version    : v1.11
Boot HART Base ISA        : rv64imafdcvx
Boot HART ISA Extensions  : zicntr,zihpm,sdtrig
Boot HART PMP Count       : 8
Boot HART PMP Granularity : 12 bits
Boot HART PMP Address Bits: 38
Boot HART MHPM Info       : 8 (0x000007f8)
Boot HART Debug Triggers  : 6 triggers
Boot HART MIDELEG         : 0x0000000000020222
Boot HART MEDELEG         : 0x000000000000b109


U-Boot 2024.10-rc3-00015-g57949a99b7bd (Sep 02 2024 - 10:41:27 +0900)milkv_duo

DRAM:  63.3 MiB
Core:  25 devices, 15 uclasses, devicetree: separate
MMC:   mmc@4310000: 0
Loading Environment from nowhere... OK
In:    serial@4140000
Out:   serial@4140000
Err:   serial@4140000
Net:
Warning: ethernet@4070000 (eth0) using random MAC address - a6:42:f8:ca:7b:bb
eth0: ethernet@4070000
milkv_duo# help
?         - alias for 'help'
base      - print or set address offset
bdinfo    - print Board Info structure
blkcache  - block cache diagnostics and control
boot      - boot default, i.e., run 'bootcmd'
bootd     - boot default, i.e., run 'bootcmd'
bootefi   - Boots an EFI payload from memory
bootelf   - Boot from an ELF image in memory
bootflow  - Boot flows
booti     - boot Linux kernel 'Image' format from memory
bootm     - boot application image from memory
bootp     - boot image via network using BOOTP/TFTP protocol
bootvx    - Boot vxWorks from an ELF image
cmp       - memory compare
coninfo   - print console devices and information
cp        - memory copy
cpu       - display information about CPUs
crc32     - checksum calculation
dhcp      - boot image via network using DHCP/TFTP protocol
dm        - Driver model low level access
echo      - echo args to console
editenv   - edit environment variable
eficonfig - provide menu-driven UEFI variable maintenance interface
env       - environment handling commands
exit      - exit script
false     - do nothing, unsuccessfully
fatinfo   - print information about filesystem
fatload   - load binary file from a dos filesystem
fatls     - list files in a directory (default /)
fatmkdir  - create a directory
fatrm     - delete a file
fatsize   - determine a file's size
fatwrite  - write file into a dos filesystem
fdt       - flattened device tree utility commands
fstype    - Look up a filesystem type
fstypes   - List supported filesystem types
go        - start application at address 'addr'
gzwrite   - unzip and write memory to block device
help      - print command description/usage
iminfo    - print header information for application image
imxtract  - extract a part of a multi-image
itest     - return true/false on integer compare
ln        - Create a symbolic link
load      - load binary file from a filesystem
loadb     - load binary file over serial line (kermit mode)
loads     - load S-Record file over serial line
loadx     - load binary file over serial line (xmodem mode)
loady     - load binary file over serial line (ymodem mode)
loop      - infinite loop on address range
ls        - list files in a directory (default /)
lzmadec   - lzma uncompress a memory region
md        - memory display
mm        - memory modify (auto-incrementing address)
mmc       - MMC sub system
mmcinfo   - display MMC info
mw        - memory write (fill)
net       - NET sub-system
nm        - memory modify (constant address)
panic     - Panic with optional message
part      - disk partition related commands
poweroff  - Perform POWEROFF of the device
printenv  - print environment variables
pxe       - get and boot from pxe files
random    - fill memory with random pattern
reset     - Perform RESET of the CPU
run       - run commands in an environment variable
save      - save file to a filesystem
setenv    - set environment variables
setexpr   - set environment variable as the result of eval expression
sf        - SPI flash sub-system
showvar   - print local hushshell variables
size      - determine a file's size
sleep     - delay execution for some time
source    - run script from memory
test      - minimal test like /bin/sh
tftpboot  - load file via network using TFTP protocol
true      - do nothing, successfully
unlz4     - lz4 uncompress a memory region
unzip     - unzip a memory region
version   - print monitor, compiler and linker version
milkv_duo# bdinfo
boot_params = 0x0000000000000000
DRAM bank   = 0x0000000000000000
-> start    = 0x0000000080000000
-> size     = 0x0000000003f40000
flashstart  = 0x0000000000000000
flashsize   = 0x0000000000000000
flashoffset = 0x0000000000000000
baudrate    = 115200 bps
relocaddr   = 0x0000000083ec4000
reloc off   = 0x0000000003cc4000
Build       = 64-bit
current eth = ethernet@4070000
ethaddr     = a6:42:f8:ca:7b:bb
IP addr     = <NULL>
fdt_blob    = 0x00000000836814c0
new_fdt     = 0x00000000836814c0
fdt_size    = 0x0000000000002940
lmb_dump_all:
 memory.cnt = 0x1 / max = 0x10
 memory[0]      [0x80000000-0x83f3ffff], 0x03f40000 bytes flags: 0
 reserved.cnt = 0x2 / max = 0x10
 reserved[0]    [0x80000000-0x8005ffff], 0x00060000 bytes flags: 4
 reserved[1]    [0x8267d000-0x83f3ffff], 0x018c3000 bytes flags: 0
devicetree  = separate
serial addr = 0x0000000004140000
 width      = 0x0000000000000004
 shift      = 0x0000000000000002
 offset     = 0x0000000000000000
 clock      = 0x00000000017d7840
boot hart   = 0x0000000000000000
firmware fdt= 0x0000000080080000

```
