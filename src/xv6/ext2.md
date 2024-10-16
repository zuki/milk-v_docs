# ext2ファイルシステムを作成

## 作成

```bash
$ mkdir ext2 ; cd ext2 ; mkdir mnt
$ dd if=/dev/zero of=disk bs=1M count=30
30+0 records in
30+0 records out
31457280 bytes (31 MB, 30 MiB) copied, 0.0198903 s, 1.6 GB/s
$ sudo losetup /dev/loop20 disk
$ sudo mkfs.ext2 -v /dev/loop20
mke2fs 1.46.5 (30-Dec-2021)
fs_types for mke2fs.conf resolution: 'ext2', 'small'
Discarding device blocks: done
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
7680 inodes, 7680 blocks
384 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=8388608
1 block group
32768 blocks per group, 32768 fragments per group
7680 inodes per group

Allocating group tables: done
Writing inode tables: done
Writing superblocks and filesystem accounting information: done

$ sudo mount -t ext2 /dev/loop20 mnt
$ sudo mkdir -p mnt/bin
$ sudo cp ~/busybox-1.37.0/busybox mnt/bin
$ sudo cp ../README.md .
$ sudo umount mnt
$ sudo dd if=/dev/loop20 bs=1M count=30 of=ext2fs.img
30+0 records in
30+0 records out
31457280 bytes (31 MB, 30 MiB) copied, 0.0657495 s, 478 MB/s
$ sudo losetup -d /dev/loop20
$ sudo chown vagrant:vagrant ext2fs.img
$ cp ext2fs.img ../duo-imgtools
```

## SDに書き込む

```diff
$ git diff xv6.cfg
diff --git a/duo-imgtools/xv6.cfg b/duo-imgtools/xv6.cfg
index 1e14bcb..5b3549b 100644
--- a/duo-imgtools/xv6.cfg
+++ b/duo-imgtools/xv6.cfg
@@ -8,6 +8,13 @@ image boot.vfat {
 	size = 64M
 }

+image rootfs.ext2 {
+	ext2 {
+		label = "rootfs"
+	}
+	size = 30M
+}
+
 image milkv-duo_sdcard.img {
 	hdimage {
 	}
@@ -18,4 +25,8 @@ image milkv-duo_sdcard.img {
 		image = "boot.vfat"
 	}

+    partition rootfs {
+        partition-type = 0x83
+        image = "ext2fs.img"
+    }
 }
```

## 実行

```bash
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
000001b0: 0000 0000 0000 0000 0000 0000 0000 8000
000001c0: 0200 0c28 2108 0100 0000 0000 0200 0028
000001d0: 2208 83fa 300b 0100 0200 00f0 0000 0000
000001e0: 0000 0000 0000 0000 0000 0000 0000 0000
000001f0: 0000 0000 0000 0000 0000 0000 0000 55aa
[0]sd_init: partition[0]: TYPE: 12, LBA = 0x1, #SECS = 0x20000
[0]sd_init: partition[1]: TYPE: 131, LBA = 0x20001, #SECS = 0xf000  // <= 30 MB
```

## ext2fs.imgにファイルを追加する

```bash
$ sudo losetup /dev/loop20 ext2fs.img
$ sudo mount -t ext2 /dev/loop20 mnt

// ここで必要なファイルをmnt徘徊にコピー

$ sudo umount mnt
$ sudo losetup -d /dev/loop20
$ ls -l
total 30724
-rw-r--r-- 1 vagrant vagrant 31457280 Oct 16 17:07 ext2fs.img
drwxrwxr-x 2 vagrant vagrant     4096 Oct 16 14:56 mnt
```

## busyboxをビルドする

```bash
$ wget https://busybox.net/downloads/busybox-1.37.0.tar.bz2
$ tar xf busybox-1.37.0.tar.bz2
$ cd busybox-1.37.0
$ cp ../duo-
$ make CROSS_COMPILE=/home/vagrant/duo-buildroot-sdk/host-tools/gcc/riscv64-linux-musl-x86_64/bin/riscv64-unknown-linux-musl- menuconfig
$ make CROSS_COMPILE=/home/vagrant/duo-buildroot-sdk/host-tools/gcc/riscv64-linux-musl-x86_64/bin/riscv64-unknown-linux-musl-
...
libbb/hash_md5_sha.c: In function 'sha1_end':
libbb/hash_md5_sha.c:1316:28: error: 'sha1_process_block64_shaNI' undeclared (first use in this function); did you mean 'sha1_process_block64'?
 1316 |   || ctx->process_block == sha1_process_block64_shaNI
      |                            ^~~~~~~~~~~~~~~~~~~~~~~~~~
      |                            sha1_process_block64
libbb/hash_md5_sha.c:1316:28: note: each undeclared identifier is reported only once for each function it appears in
make[1]: *** [scripts/Makefile.build:198: libbb/hash_md5_sha.o] Error 1
make: *** [Makefile:744: libbb] Error 2
$ vi libbb/hash_md5_sha.c
// https://lists.busybox.net/pipermail/busybox/2024-September/090899.html のパッチを適用
$ make CROSS_COMPILE=/home/vagrant/duo-buildroot-sdk/host-tools/gcc/riscv64-linux-musl-x86_64/bin/riscv64-unknown-linux-musl-
...
  AR      util-linux/volume_id/lib.a
  LINK    busybox_unstripped
Trying libraries: crypt m rt
 Library crypt is not needed, excluding it
 Library m is not needed, excluding it
 Library rt is not needed, excluding it
Final link with: <none>
  DOC     busybox.pod
  DOC     BusyBox.txt
  DOC     busybox.1
  DOC     BusyBox.html
$ ls -l busybox busybox_unstripped
-rwxrwxr-x 1 vagrant vagrant  505488 Oct 16 17:02 busybox
-rwxrwxr-x 1 vagrant vagrant 2452120 Oct 16 17:02 busybox_unstripped
```
