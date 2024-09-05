# u-bootのbootmコマンド定義

## sdbootコマンド

```
setenv bootargs ${reserved_mem} ${root} ${mtdparts} console=$consoledev,$baudrate $othbootargs;
echo Boot from SD ...;
mmc dev 0 && fatload mmc 0 ${uImage_addr} boot.sd;
bootm ${uImage_addr}#config-cv1800b_milkv_duo_sd;
fi;
```

- `config-cv1800b_milkv_duo_sd`: mkimageで-fで指定する*.itsファイルで定義したconfigurations名
- `${uImage_addr}`: UIMAG_ADDR = CVIMMAP_UIMAG_ADDR = 0x81400000
- bootmコマンドの詳細については[bootコマンド](file:///Users/dspace/pine64/docs/book/uboot/commands/bootm.html)を参照

## xv6-virusvで実行

```bash
# milkv-duo.imgをSDに焼く
$ mv /Volume/boot/boot.sd /Volume/boot/boot.sd.org	# u-bootで止めるため
$ minicom
# 以下はfip.binが実行されている

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
[M1S./307x61834030]0P/r0ex 8s0y0s0t0e0m0 0i/n0ixt1 bd0o0n0e.

RT: [1.383163]CVIRTOS Build Date:Dec 20 2023  (Time :17:25:52)
RT: [1.389166]Post system init done
RT: [1.392481]create cvi task
RT: [1.395307][cvi_spinlock_init] succeess
RT: [1.399202]prvCmdQuRunTask run
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
Warning: ethernet@4070000 (eth0) using random MAC address - 26:96:f5:f7:d2:d1
eth0: ethernet@4070000
Hit any key to stop autoboot:  0
Boot from SD ...
switch to partitions #0, OK
mmc0 is current device
** Unable to read file boot.sd **
Failed to load 'boot.sd'		# fatload mmc 0 ${uImage_addr} boot.sdでエラー
cv180x_c906#					# bootの処理が停止してコマンドプロントが現れる

cv180x_c906# printenv
baudrate=115200
bootargs=root=/dev/mmcblk0p2 rootwait rw console=ttyS0,115200 earlycon=sbi logl0
bootcmd=run sdboot
bootdelay=1
consoledev=ttyS0
fdtcontroladdr=836bbe70
gatewayip=192.168.0.11
ipaddr=192.168.0.3
netdev=eth0
netmask=255.255.255.0
othbootargs=earlycon=sbi loglevel=9 riscv.fwsz=0x80000
root=root=/dev/mmcblk0p2 rootwait rw
sdboot=setenv bootargs ${root} ${mtdparts} console=$consoledev,$baudrate $othbo;
serverip=192.168.56.101
stderr=serial
stdin=serial
stdout=serial
uImage_addr=0x81400000
update_addr=0x83f40000

Environment size: 708/131068 bytes

# boot.sd.orgを指定してsdbootをやり直す
# 1. boot.sd.orgのロード
cv180x_c906# mmc dev 0 && fatload mmc 0 ${uImage_addr} boot.sd.org
switch to partitions #0, OK
mmc0 is current device
2087112 bytes read in 95 ms (21 MiB/s)
# 2. boot.sd.orgの実行
cv180x_c906# bootm ${uImage_addr}#config-cv1800b_milkv_duo_sd
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


xv6 kernel is booting

helloworld! IEEE80211 TEAM
SPI CTRLR0: 0x7
init: starting sh
$ ls
.              1 1 1024
..             1 1 1024
README         2 2 2305
...
```

## `duo-buildroot-sdk/u-boot-2021.10/include/configs/cv180x-asic.h`

```c
#include <../../../board/cvitek/cv180x/cv180x_reg.h>

/* cvi_board_memmap.h build/boards/{CHIP_ARCH}/{BOARD}/memmap.py により作成される*/
#include "cvi_board_memmap.h"
    #define CONFIG_SYS_TEXT_BASE 0x80200000  /* offset 2.0MiB */
    #define CVIMMAP_ATF_SIZE 0x80000  /* 512.0KiB */
    #define CVIMMAP_BOOTLOGO_ADDR 0x82473000  /* offset 36.44921875MiB */
    #define CVIMMAP_BOOTLOGO_SIZE 0x0  /* 0.0KiB */
    #define CVIMMAP_CONFIG_SYS_INIT_SP_ADDR 0x82300000  /* offset 35.0MiB */
    #define CVIMMAP_CVI_UPDATE_HEADER_ADDR 0x813ffc00  /* offset 19.9990234375MiB */
    #define CVIMMAP_CVI_UPDATE_HEADER_SIZE 0x400  /* 1.0KiB */
    #define CVIMMAP_DRAM_BASE 0x80000000  /* offset 0.0KiB */
    #define CVIMMAP_DRAM_SIZE 0x4000000  /* 64.0MiB */
    #define CVIMMAP_FREERTOS_ADDR 0x83f40000  /* offset 63.25MiB */
    #define CVIMMAP_FREERTOS_RESERVED_ION_SIZE 0x0  /* 0.0KiB */
    #define CVIMMAP_FREERTOS_SIZE 0xc0000  /* 768.0KiB */
    #define CVIMMAP_FSBL_C906L_START_ADDR 0x83f40000  /* offset 63.25MiB */
    #define CVIMMAP_FSBL_UNZIP_ADDR 0x81400000  /* offset 20.0MiB */
    #define CVIMMAP_FSBL_UNZIP_SIZE 0xf00000  /* 15.0MiB */
    #define CVIMMAP_H26X_BITSTREAM_ADDR 0x82473000  /* offset 36.44921875MiB */
    #define CVIMMAP_H26X_BITSTREAM_SIZE 0x0  /* 0.0KiB */
    #define CVIMMAP_H26X_ENC_BUFF_ADDR 0x82473000  /* offset 36.44921875MiB */
    #define CVIMMAP_H26X_ENC_BUFF_SIZE 0x0  /* 0.0KiB */
    #define CVIMMAP_ION_ADDR 0x82473000  /* offset 36.44921875MiB */
    #define CVIMMAP_ION_SIZE 0x1acd000  /* 26.80078125MiB */
    #define CVIMMAP_ISP_MEM_BASE_ADDR 0x82473000  /* offset 36.44921875MiB */
    #define CVIMMAP_ISP_MEM_BASE_SIZE 0x0  /* 0.0KiB */
    #define CVIMMAP_KERNEL_MEMORY_ADDR 0x80000000  /* offset 0.0KiB */
    #define CVIMMAP_KERNEL_MEMORY_SIZE 0x3f40000  /* 63.25MiB */
    #define CVIMMAP_MONITOR_ADDR 0x80000000  /* offset 0.0KiB */
    #define CVIMMAP_OPENSBI_FDT_ADDR 0x80080000  /* offset 512.0KiB */
    #define CVIMMAP_OPENSBI_SIZE 0x80000  /* 512.0KiB */
    #define CVIMMAP_UIMAG_ADDR 0x81400000  /* offset 20.0MiB */
    #define CVIMMAP_UIMAG_SIZE 0xf00000  /* 15.0MiB */
/* mkcvipart.py により作成されたパーティション定義ヘッダー */
#include "cvipart.h"
    #ifndef CVIPART_H
    #define CVIPART_H
    #ifndef CONFIG_ENV_IS_NOWHERE
    #define CONFIG_ENV_IS_NOWHERE
    #endif
    #define CONFIG_ENV_SIZE 0x20000
    #define PART_LAYOUT ""
    #define ROOTFS_DEV "/dev/mmcblk0p2"
    #define PARTS_OFFSET \
    "BOOT_PART_OFFSET=0x0\0" \
    "BOOT_PART_SIZE=0x40000\0" \
    "ROOTFS_PART_OFFSET=0x40000\0" \
    "ROOTFS_PART_SIZE=0x180100\0"
    #define SPL_BOOT_PART_OFFSET 0x0

#    include "cvi_panels/cvi_panel_diffs.h"

#define CONFIG_REMAKE_ELF

/* 物理メモリマップ */
#define CONFIG_SYS_RESVIONSZ		CVIMMAP_ION_SIZE            // 0x1acd000
#define CONFIG_SYS_BOOTMAPSZ		CVIMMAP_KERNEL_MEMORY_SIZE  // 0x3f40000

#define PHYS_SDRAM_1			    CVIMMAP_KERNEL_MEMORY_ADDR      // 0x80000000
#define PHYS_SDRAM_1_SIZE		    CVIMMAP_KERNEL_MEMORY_SIZE      // 0x3f40000
#define CONFIG_SYS_SDRAM_BASE		PHYS_SDRAM_1                // 0x80000000

/* リンク定義 */
#define CONFIG_SYS_INIT_SP_ADDR		CVIMMAP_CONFIG_SYS_INIT_SP_ADDR     // 0x82300000

/* 引数がないばあのbootmコマンドのデフォルトアドレス */
#define CONFIG_SYS_LOAD_ADDR		0x80080000
#define CONFIG_SYS_BOOTM_LEN		(64 << 20)

/* Generic割り込みコントローラの定義 */
#define GICD_BASE			(0x01F01000)
#define GICC_BASE			(0x01F02000)

/* malloc() プールのサイズ */
#define CONFIG_SYS_MALLOC_LEN		(CONFIG_ENV_SIZE + (8 << 20))

#if defined(__riscv)
#define CONFIG_SYS_CACHELINE_SIZE	64
#endif

//  RISC-V rdtimeの周波数
#define SYS_COUNTER_FREQ_IN_SECOND 25000000

/* 16550シリアルの構成 */
#define CONFIG_SYS_NS16550_COM1		0x04140000
#define CONFIG_SYS_NS16550_SERIAL
#define CONFIG_SYS_NS16550_REG_SIZE	(-4)
#define CONFIG_SYS_NS16550_MEM32
#define CONFIG_SYS_NS16550_CLK		25000000

/* include/generated/autoconf.h would define CONFIG_BAUDRATE from drivers/serial/Kconfig (default 115200) */

/*#define CONFIG_MENU_SHOW*/

/* Download related definitions */
#define UPGRADE_SRAM_ADDR	0x0e000030
#define UBOOT_PID_SRAM_ADDR  0x0e000030
#define UPDATE_ADDR	CVIMMAP_ION_ADDR
#define HEADER_ADDR	UPDATE_ADDR
#define USB_UPDATE_MAGIC MAGIC_NUM_USB_DL

/* Monitor Command Prompt */
#define CONFIG_SYS_CBSIZE		512	/* Console I/O Buffer Size */
#define CONFIG_SYS_PBSIZE		(CONFIG_SYS_CBSIZE + \
					sizeof(CONFIG_SYS_PROMPT) + 16)
#define CONFIG_SYS_BARGSIZE		CONFIG_SYS_CBSIZE
#define CONFIG_SYS_MAXARGS		64	/* max command args */

#ifndef CONFIG_ENV_SECT_SIZE
#define CONFIG_ENV_SECT_SIZE		0x00040000
#endif

#define CONFIG_ENV_OVERWRITE

#define CONFIG_MMC_UHS_SUPPORT
#define CONFIG_MMC_HS200_SUPPORT
#define CONFIG_MMC_SUPPORTS_TUNING

#define CONFIG_IPADDR			192.168.0.3
#define CONFIG_NETMASK			255.255.255.0
#define CONFIG_GATEWAYIP		192.168.0.11
#define CONFIG_SERVERIP			192.168.56.101


/* CONFIG_USE_DEFAULT_ENV=1: コンパイル時の=Dオプションで定義 */

/* 以下の設定はチップ依存 */
/******************************************************************************/
	#define UIMAG_ADDR	CVIMMAP_UIMAG_ADDR      // 0x81400000

/******************************************************************************/
/* 共通envの定義 */
/*******************************************************************************/
	/* Config FDT_NO */
	#define FDT_NO __stringify(CVICHIP) "_" __stringify(CVIBOARD)

	/* config root */
	#define ROOTARGS "rootfstype=squashfs rootwait ro root=" ROOTFS_DEV

	/* BOOTARGS */
	#define PARTS  PART_LAYOUT

	/* config uart */
	#define CONSOLEDEV "ttyS0\0"

	/* config loglevel */
	#ifdef RELEASE
		#define CONSOLE_LOGLEVEL   " loglevel=0"
		#define EARLYCON_RELEASE   " release "
	#else
		#define CONSOLE_LOGLEVEL   " loglevel=9"
		#define EARLYCON_RELEASE   " "
	#endif

	#define OTHERBOOTARGS   "earlycon=sbi riscv.fwsz="  __stringify(CVIMMAP_OPENSBI_SIZE) \
		EARLYCON_RELEASE CONSOLE_LOGLEVEL

	#define CONFIG_EXTRA_ENV_SETTINGS	\
		"netdev=eth0\0"		\
		"consoledev=" CONSOLEDEV  \
		"baudrate=115200\0" \
		"uImage_addr=" __stringify(UIMAG_ADDR) "\0" \       // 0x81400000
		"update_addr=" __stringify(UPDATE_ADDR) "\0" \      // 0x82473000
		"mtdparts=" PARTS "\0" \
		"mtdids=" MTDIDS_DEFAULT "\0" \
		"root=" ROOTARGS "\0" \
		"sdboot=" SD_BOOTM_COMMAND "\0" \
		"othbootargs=" OTHERBOOTARGS "\0"\
		PARTS_OFFSET

/********************************************************************************/
	/* UBOOT_VBOOT commands : UBOOT?VBOOT = 0*/
	#define UBOOT_VBOOT_BOOTM_COMMAND "bootm ${uImage_addr}#config-" FDT_NO ";"

	/* BOOTLOGO */
	#define SHOWLOGOCMD

	#define SET_BOOTARGS "setenv bootargs ${reserved_mem} ${root} ${mtdparts} " \
					"console=$consoledev,$baudrate $othbootargs;"

	#define SD_BOOTM_COMMAND \
				SET_BOOTARGS \
				"echo Boot from SD ...;" \
				"mmc dev 0 && fatload mmc 0 ${uImage_addr} boot.sd; " \
				"if test $? -eq 0; then " \
				UBOOT_VBOOT_BOOTM_COMMAND \
				"fi;"

    // setenv bootargs ${reserved_mem} ${root} ${mtdparts} console=$consoledev,$baudrate $othbootargs;
    // echo Boot from SD ...;
    // mmc dev 0 && fatload mmc 0 ${uImage_addr} boot.sd;
    // bootm ${uImage_addr}#config-;
    // fi;

	#define CONFIG_BOOTCOMMAND	SHOWLOGOCMD "cvi_update || run norboot || run nandboot ||run emmcboot"

#define CVI_SPL_BOOTAGRS \
	PARTS " "  \
	ROOTARGS " " \
	"console=ttyS0,115200 " \
	OTHERBOOTARGS "\0"
```

## `duo-buildroot-sdk/u-boot-2021.10/build/cv1800b_milkv_duo_sd/.config`

```bash
CONFIG_CREATE_ARCH_SYMLINK=y
CONFIG_LINKER_LIST_ALIGN=4
CONFIG_RISCV=y
CONFIG_SYS_ARCH="riscv"
CONFIG_SYS_CPU="generic"
CONFIG_SYS_VENDOR="cvitek"
CONFIG_SYS_BOARD="cv180x"
CONFIG_SYS_CONFIG_NAME="cv180x-asic"
CONFIG_DMA_ADDR_T_64BIT=y
CONFIG_USE_ARCH_MEMCPY=y
CONFIG_USE_ARCH_MEMSET=y
CONFIG_SYS_MALLOC_F_LEN=0x2000
CONFIG_NR_DRAM_BANKS=1
CONFIG_DEFAULT_DEVICE_TREE="cv180x_asic"
CONFIG_ERR_PTR_OFFSET=0x0
CONFIG_SPL_SIZE_LIMIT_PROVIDE_STACK=0
CONFIG_BOOTSTAGE_STASH_ADDR=0
CONFIG_IDENT_STRING="cvitek_cv180x"
CONFIG_BUILD_TARGET=""
CONFIG_64BIT=y
#
# RISC-V architecture
#
CONFIG_TARGET_CVITEK_RISCV=y
CONFIG_GENERIC_RISCV=y
CONFIG_ARCH_RV64I=y
CONFIG_CMODEL_MEDLOW=y
CONFIG_RISCV_SMODE=y
CONFIG_RISCV_ISA_C=y
CONFIG_RISCV_ISA_A=y
CONFIG_SBI=y
CONFIG_SBI_V02=y
CONFIG_STACK_SIZE_SHIFT=14
CONFIG_OF_BOARD_FIXUP=y
#
# Use assembly optimized implementation of memory routines
#
CONFIG_USE_ARCH_MEMMOVE=y
CONFIG_TARGET_CVITEK=y
CONFIG_TARGET_CVITEK_CV180X=y
CONFIG_TARGET_CVITEK_CV180X_ASIC=y
#
# General setup
#
CONFIG_LOCALVERSION=""
CONFIG_LOCALVERSION_AUTO=y
CONFIG_CC_IS_GCC=y
CONFIG_GCC_VERSION=100200
CONFIG_CLANG_VERSION=0
CONFIG_CC_OPTIMIZE_FOR_SIZE=y
CONFIG_CC_HAS_ASM_INLINE=y
CONFIG_SYS_MALLOC_F=y
CONFIG_EXPERT=y
CONFIG_SYS_MALLOC_CLEAR_ON_INIT=y
CONFIG_PHYS_64BIT=y
CONFIG_PLATFORM_ELFENTRY="_start"
CONFIG_STACK_SIZE=0x1000000
CONFIG_SYS_SRAM_BASE=0x0
CONFIG_SYS_SRAM_SIZE=0x0
#
# Boot images
#
CONFIG_FIT=y
CONFIG_FIT_EXTERNAL_OFFSET=0x0
CONFIG_FIT_FULL_CHECK=y
CONFIG_SYS_EXTRA_OPTIONS=""
#
# Boot timing
#
CONFIG_BOOTSTAGE_STASH_SIZE=0x1000
#
# Autoboot options
#
CONFIG_AUTOBOOT=y
CONFIG_BOOTDELAY=1
CONFIG_USE_BOOTCOMMAND=y
CONFIG_BOOTCOMMAND="run distro_bootcmd"
CONFIG_DEFAULT_FDT_FILE=""
#
# Console
#
CONFIG_LOGLEVEL=4
CONFIG_SPL_LOGLEVEL=4
CONFIG_TPL_LOGLEVEL=4
#
# Start-up hooks
#
CONFIG_ARCH_EARLY_INIT_R=y
#
# Security support
#
CONFIG_HASH=y
#
# SPL / TPL
#
CONFIG_SPL_SYS_STACK_F_CHECK_BYTE=0xaa
#
# Command line interface
#
CONFIG_CMDLINE=y
CONFIG_HUSH_PARSER=y
CONFIG_SYS_PROMPT="cv180x_c906# "
CONFIG_SYS_PROMPT_HUSH_PS2="> "
#
# Boot commands
#
CONFIG_CMD_BOOTM=y
CONFIG_BOOTM_LINUX=y
CONFIG_BOOTM_OPENRTOS=y
CONFIG_CMD_RUN=y
#
# Memory commands
#
CONFIG_CMD_MEMORY=y
CONFIG_CMD_RANDOM=y
#
# Device access commands
#
CONFIG_CMD_MMC=y
CONFIG_CMD_PART=y
#
# Shell scripting commands
#
CONFIG_CMD_ECHO=y
#
# Android support commands
#
CONFIG_CMD_NET=y
#
# Filesystem commands
#
CONFIG_CMD_FAT=y
CONFIG_CMD_FS_GENERIC=y
CONFIG_MTDIDS_DEFAULT=""
CONFIG_MTDPARTS_DEFAULT=""
#
# Debug commands
#
CONFIG_CMD_CVI_UPDATE=y
#
# Partition Types
#
CONFIG_PARTITIONS=y
CONFIG_DOS_PARTITION=y
CONFIG_PARTITION_UUIDS=y
CONFIG_SUPPORT_OF_CONTROL=y
CONFIG_DTC=y
#
# Device Tree Control
#
CONFIG_OF_CONTROL=y
CONFIG_OF_SEPARATE=y
#
# Environment
#
CONFIG_ENV_SUPPORT=y
CONFIG_ENV_IS_NOWHERE=y
CONFIG_NET=y
CONFIG_NET_RANDOM_ETHADDR=y
CONFIG_TFTP_BLOCKSIZE=1468
CONFIG_TFTP_WINDOWSIZE=1
CONFIG_SERVERIP_FROM_PROXYDHCP_DELAY_MS=100
#
# Generic Driver Options
#
CONFIG_DM=y
CONFIG_DM_WARN=y
CONFIG_DM_DEVICE_REMOVE=y
CONFIG_DM_STDIO=y
CONFIG_DM_SEQ_ALIAS=y
CONFIG_SIMPLE_BUS=y
CONFIG_OF_TRANSLATE=y
CONFIG_DM_DEV_READ_INLINE=y
#
# Bus devices
#
CONFIG_BLK=y
CONFIG_HAVE_BLOCK_DEVICE=y
CONFIG_BLOCK_CACHE=y
#
# Clock
#
CONFIG_CPU=y
CONFIG_CPU_RISCV=y
#
# Hardware crypto devices
#
CONFIG_CAAM_64BIT=y
#
# Hardware Spinlock Support
#
CONFIG_I2C=y
CONFIG_INPUT=y
#
# MMC Host controller Support
#
CONFIG_MMC=y
CONFIG_MMC_WRITE=y
CONFIG_DM_MMC=y
CONFIG_MMC_QUIRKS=y
CONFIG_MMC_HW_PARTITIONING=y
CONFIG_MMC_VERBOSE=y
CONFIG_MMC_SDHCI=y
CONFIG_MMC_SDHCI_SDMA=y
CONFIG_MMC_SDHCI_CVITEK=y
#
# MTD Support
#
CONFIG_MTD=y
#
# Multiplexer drivers
#
CONFIG_PHYLIB=y
CONFIG_PHY_CVITEK=y
CONFIG_DM_ETH=y
CONFIG_NETDEVICES=y
CONFIG_ETH_DESIGNWARE=y
#
# Serial drivers
#
CONFIG_BAUDRATE=115200
CONFIG_SPECIFY_CONSOLE_INDEX=y
CONFIG_CONS_INDEX=1
CONFIG_SYS_NS16550=y
#
# SOC (System On Chip) specific Drivers
#
CONFIG_SPI=y
#
# Timer Support
#
CONFIG_TIMER=y
CONFIG_RISCV_TIMER=y
#
# Watchdog Timer Support
#
CONFIG_WATCHDOG_TIMEOUT_MSECS=60000
#
# File systems
#
CONFIG_FS_FAT=y
CONFIG_FAT_WRITE=y
CONFIG_FS_FAT_MAX_CLUSTSIZE=65536
#
# Library routines
#
CONFIG_LIB_UUID=y
CONFIG_PRINTF=y
CONFIG_SPRINTF=y
CONFIG_STRTO=y
CONFIG_SYS_HZ=1000
CONFIG_LIB_RAND=y
#
# Hashing Support
#
CONFIG_SHA1=y
CONFIG_SHA256=y
CONFIG_MD5=y
#
# Compression Support
#
CONFIG_LZMA=y
CONFIG_ZLIB=y
CONFIG_OF_LIBFDT=y
CONFIG_OF_LIBFDT_ASSUME_MASK=0
CONFIG_SPL_OF_LIBFDT_ASSUME_MASK=0xff
CONFIG_TPL_OF_LIBFDT_ASSUME_MASK=0xff
#
# System tables
#
CONFIG_LMB=y
CONFIG_LMB_USE_MAX_REGIONS=y
CONFIG_LMB_MAX_REGIONS=8
#
# Tools options
#
CONFIG_MKIMAGE_DTC_PATH="dtc"
```

## コンパイルフラグ

`-D__KERNEL__ -D__UBOOT__ -DCONFIG_SKIP_RAMDISK=y -DCONFIG_USE_DEFAULT_ENV=y -DCVICHIP=cv1800b -DCVIBOARD=milkv_duo_sd -DCV1800B_MILKV_DUO_SD -DMIPI_PANEL_HX8394`
