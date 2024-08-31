# Milk-V Duo用のLinuxシステムの作成

## host-tools

```bash
$ wget https://sophon-file.sophon.cn/sophon-prod-s3/drive/23/03/07/16/host-tools.tar.gz
$ tar xf host-tools.tar.gz
$ cd host-tools ; ls
gcc
$ cd gcc ; ls
riscv64-elf-x86_64  riscv64-linux-musl-x86_64  riscv64-linux-x86_64

# これが使われている
$ riscv64-elf-x86_64/bin/riscv64-unknown-elf-gcc --version
riscv64-unknown-elf-gcc (V2.0.3.0-xialf) 10.2.0
$ ls riscv64-elf-x86_64/
bin  include  lib  lib64  libexec  riscv64-unknown-elf  share

$ riscv64-linux-musl-x86_64/bin/riscv64-unknown-linux-musl-gcc --version
riscv64-unknown-linux-musl-gcc (Xuantie-900 linux-5.10.4 musl gcc Toolchain V2.6.1 B-20220906) 10.2.0
$ ls riscv64-linux-musl-x86_64/
bin  include  lib  lib64  libexec  riscv64-unknown-linux-musl  share  sysroot

$ riscv64-linux-x86_64/bin/riscv64-unknown-linux-gnu-gcc --version
riscv64-unknown-linux-gnu-gcc (Xuantie-900 linux-5.10.4 glibc gcc Toolchain V2.6.1 B-20220906) 10.2.0
$ ls riscv64-linux-x86_64/
bin  include  lib  lib64  libexec  riscv64-unknown-linux-gnu  share  sysroot
```

## Linuxのビルド

- [Quick Start](https://github.com/milkv-duo/duo-buildroot-sdk)による
- Ubuntu 22.04.4 LTSを使用

```bash
$ sudo apt install -y pkg-config build-essential ninja-build automake autoconf libtool wget curl git gcc libssl-dev bc slib squashfs-tools android-sdk-libsparse-utils jq python3-distutils scons parallel tree python3-dev python3-pip device-tree-compiler ssh cpio fakeroot libncurses5 flex bison libncurses5-dev genext2fs rsync unzip dosfstools mtools tcl openssh-client cmake expect

$ git clone https://github.com/milkv-duo/duo-buildroot-sdk.git --depth=1
$ cd duo-buildroot-sdk
$ mv ../host-tools .        # ダウンロード済のhost-toolsを規定の場所に移動
$ ./build.sh lunch
Select a target to build:
1. milkv-duo-sd
2. milkv-duo-spinand
3. milkv-duo-spinor
4. milkv-duo256m-sd
5. milkv-duo256m-spinand
6. milkv-duo256m-spinor
7. milkv-duos-emmc
8. milkv-duos-sd
Which would you like: 1
```

## `milkv-duo-sd`を選択した場合の`build.sh`の処理内容

1. MILKV_BOARD=milkv-duo-sd
2. MILKV_BOARD_CONFIG=device/milkv-duo-sd/boardconfig.sh
3. ツールチェインの取得(host-toolsディレクトリがあれば何もしない)
4. 環境変数の設定

    ```bash
    source device/milkv-duo-sd/boardconfig.sh
        export MV_BOARD=milkv-duo-sd
        export MV_BOARD_CPU=cv1800b
        export MV_VENDOR=milkv
        export MV_BUILD_ENV=milkvsetup.sh
        export MV_BOARD_LINK=cv1800b_milkv_duo_sd
    source build/milkvsetup.sh > /dev/null 2>&1
        # 各種関数の定義
        build/scripts/boards_scan.py --gen-build-kconfig  # build/output/Kconfigを生成
        build/scripts/gen_sensor_config.py                # build/output/Kconfig.sensorsの生成
        build/scripts/gen_panel_config.py                 # build/output/Kconfig.panelsの生成

        # 共通関数のインポート
        source build/common_functions.sh

    defconfig cv1800b_milkv_duo_sd > /dev/null 2>&1
    MILKV_IMAGE_CONFIG=device/milkv-duo-sd/genimage.cfg
    ```
5. 選択情報の表示
6. ビルド: clean \& rebuild

    ```bash
    # clean_all
    clean_uboot
    clean_kernel
    clean_ramdisk
    clean_osdrv
    clean_middleware
    # build_all
    build_uboot
    build_kernel
    build_osdrv
    build_middleware
    pack_access_guard_turnkey_app
    pack_ipc_turnkey_app
    pack_boot
    pack_cfg
    pack_rootfs
    pack_data
    pack_system
    copy_tools
    pack_upgrade
    ```

7. imageの作成

## ビルド概要

```bash
$ ./build.sh lunch
...
 Run clean_uboot() function 
  [TARGET] opensbi-clean
  [TARGET] rtos-clean
  [TARGET] fsbl-clean
  [TARGET] u-boot-clean
 Run clean_kernel() function
  [TARGET] kernel-clean
 Run clean_osdrv() function
 Run clean_middleware() function
 Run build_uboot() function
 Run _link_uboot_logo() function
  [TARGET] /home/vagrant/duo-buildroot-sdk/u-boot-2021.10/build/cv1800b_milkv_duo_sd/.config
  [TARGET] u-boot-dts
  [TARGET] u-boot-build
  [TARGET] opensbi
  [TARGET] rtos
  [TARGET] fsbl-build
  [TARGET] u-boot-dep
 Run build_kernel() function
  [TARGET] /home/vagrant/duo-buildroot-sdk/linux_5.10/build/cv1800b_milkv_duo_sd/.config 
  [TARGET] kernel-build
  [TARGET] kernel
 Run pack_boot() function
  [TARGET] kernel-dts
  [TARGET] boot
 Run build_osdrv()  function 
 Run build_middleware() function
 Run pack_boot() function
  [TARGET] kernel-dts
  [TARGET] boot
 Run pack_cfg_sd() function
 Run pack_rootfs_sd() function 
  [TARGET] br-rootfs-prepare
  [TARGET] br-rootfs-pack
 Run pack_data_sd() function
  [TARGET] jffs2
 Run pack_system_sd() function
  [TARGET] sd_image 
```

## [`build.sh`](build.sh.txt)

## ビルドイメージの内容

```bash
$ parted milkv-duo-sd-20240828-1520.img unit B print
WARNING: You are not superuser.  Watch out for permissions.
Model:  (file)
Disk /home/vagrant/duo-buildroot-sdk/out/milkv-duo-sd-20240828-1520.img: 939524608B
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start       End         Size        Type     File system  Flags
 1      512B        134218239B  134217728B  primary  fat16        boot, lba
 2      134218240B  939524607B  805306368B  primary  ext4

$ mkdir mnt
$ sudo mount -o ro,loop,offset=512 milkv-duo-sd-20240828-1520.img mnt/
$ ls mnt/
boot.sd  fip.bin
$ file mnt/boot.sd 
mnt/boot.sd: Device Tree Blob version 17, size=2981832, boot CPU=0, string block size=100, DT structure block size=2980744
$ file mnt/fip.bin
mnt/fip.bin: data
$ sudo umount mnt

$ sudo mount -o ro,loop,offset=134218240 milkv-duo-sd-20240828-1520.img mnt/
$ ls mnt/
bin  etc  lib64    lost+found  mnt  proc  run   sys  usr
dev  lib  linuxrc  media       opt  root  sbin  tmp  var
$ cat mnt/etc/os-release
NAME=Buildroot
VERSION=20240828-1517
ID=buildroot
VERSION_ID=2021.05
PRETTY_NAME="Buildroot 2021.05"
$ file mnt/bin/busybox
mnt/bin/busybox: ELF 64-bit LSB executable, UCB RISC-V, RVC, double-float ABI, version 1 (SYSV), statically linked, stripped
$ readelf -A mnt/bin/busybox
Attribute Section: riscv
File Attributes
  Tag_RISCV_arch: "rv64i2p0_m2p0_a2p0_f2p0_d2p0_c2p0_v0p7_zfh1p0_zvamo0p7_zvlsseg0p7_xtheadc2p0"
  Tag_RISCV_priv_spec: 1
  Tag_RISCV_priv_spec_minor: 11
$ ls mnt/dev
fd  log  pts  shm  stderr  stdin  stdout
$ ls mnt/etc
dhcpcd.conf   hosts        mtab        profile.d    shells
dnsmasq.conf  init.d       network     protocols    ssl
dropbear      inittab      ntp.conf    resolv.conf  uhubon.sh
fstab         issue        os-release  run_usb.sh   wgetrc
group         libnl        passwd      services     wpa_supplicant.conf
hostname      mke2fs.conf  profile     shadow

$ sudo umount mnt
```

## [ビルドログ](build_log.txt)
