# userコマンドのビルドライブラリにmuslを使用する

## muslをインストール

```bash
$ mkdir ~/musl
$ cd ~/source
$ git clone git://git.musl-libc.org/musl
$ cd musl
$ export CROSS_COMPILE=riscv64-linux-gnu- && ./configure --target=riscv64 --prefix=$HOME/musl
$ make
$ make install
$ ls ~/musl
bin  include  lib
```

## user/*.cをmusl apiで書き換える

- userコマンドのビルドにrpi-os/usr/Makefileを使用する
- userディレクトリのファイル編成もrpi-osに合わせる
- `struct stat`などの定義をlinuxにあわせる
- syscall番号をmusl (linux)のsyscall番号に合わせてカーネルソースを変更

## 実行

```bash
SUPER-BLOCK: magic: 0x10203040, size: 1500, nblks: 1451, ninode: 256, nlog: 30
             logstart: 2, inodestart: 32, bmapstart: 48
FSSIZE: 1500, nmeta: 49

[WARN ] sys_openat: expect O_LARGEFILE in open flags
usertrap(): unexpected scause 0xf pid=1
            sepc=0x486 stval=0x3fffffff5b
[DEBUG] usertrap: killed pid: 1
panic: init exiting
```

- debug行を追加
- 現象的にはsyscall後にerrnoをtp相対で-164のアドレスに保管する
  ようになっているが、tpに正しい値が入ってない(-1)ので保管アドレスが
  奇数となりアクセスエラーになっているようだ。

```bash
[INFO ] initlog: init log ok
[INFO ] fsinit: fsinit ok
[DEBUG] usertrap: scause: 8, which_dev: 0
[DEBUG] usertrap: call syscall
[DEBUG] syscall: num: 221, ret: 1               // execve
[DEBUG] usertrap: syscall called: a0 = 0x0000000000000001
[DEBUG] usertrap: scause: 8, which_dev: 0
[DEBUG] usertrap: call syscall
[DEBUG] sys_openat: dirfd: -100, path: console, mode: 0x00000000, flags: 0x00008002
[WARN ] fileopen: console is not found
[DEBUG] syscall: num: 56, ret: -2               // openat
[DEBUG] usertrap: syscall called: a0 = 0x00000000fffffffe
[DEBUG] usertrap: scause: 15, which_dev: 0
usertrap(): unexpected scause 0xf pid=1
            sepc=0x4e8 stval=0xffffffffffffff5b     // stval値が奇数
[DEBUG] usertrap: killed pid: 1
panic: init exiting
```

- `obj/usr/src/init.asm`

```bash
00000000000004c6 <__syscall_ret>:
     4c6:       1141                    add     sp,sp,-16
     4c8:       e022                    sd      s0,0(sp)
     4ca:       e406                    sd      ra,8(sp)
     4cc:       77fd                    lui     a5,0xfffff
     4ce:       842a                    mv      s0,a0
     4d0:       00a7e663                bltu    a5,a0,4dc <__syscall_ret+0x16>
     4d4:       60a2                    ld      ra,8(sp)
     4d6:       6402                    ld      s0,0(sp)
     4d8:       0141                    add     sp,sp,16
     4da:       8082                    ret
     4dc:       1fb000ef                jal     ed6 <__errno_location>
     4e0:       4080043b                negw    s0,s0
     4e4:       c100                    sw      s0,0(a0)        // sepc=0x4e8: wordを奇数番地にstoreでエラー
     4e6:       557d                    li      a0,-1
     4e8:       b7f5                    j       4d4 <__syscall_ret+0xe>

0000000000000ed6 <__errno_location>:
     ed6:       8512                    mv      a0,tp            // このtpの値が想定した値でない  tp = -1
     ed8:       f5c50513                add     a0,a0,-164       // この-164 は?
     edc:       8082                    ret
```

- [Incorrect RISC-V assembly with -fno-omit-frame-pointer](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=110634) を
参考にCFLAGSの`-fno-omit-frame-pointer`を削除したが結果は同じだった

```bash
$ ./musl/bin/musl-gcc -v
Using built-in specs.
Reading specs from /home/vagrant/musl/lib/musl-gcc.specs
rename spec cpp_options to old_cpp_options
COLLECT_GCC=riscv64-linux-gnu-gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc-cross/riscv64-linux-gnu/11/lto-wrapper
Target: riscv64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 11.4.0-1ubuntu1~22.04' --with-bugurl=file:///usr/share/doc/gcc-11/README.Bugs --enable-languages=c,ada,c++,go,d,fortran,objc,obj-c++,m2 --prefix=/usr --with-gcc-major-version-only --program-suffix=-11 --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --with-sysroot=/ --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-libitm --disable-libquadmath --disable-libquadmath-support --enable-plugin --enable-default-pie --with-system-zlib --enable-libphobos-checking=release --without-target-system-zlib --enable-multiarch --disable-werror --disable-multilib --with-arch=rv64gc --with-abi=lp64d --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=riscv64-linux-gnu --program-prefix=riscv64-linux-gnu- --includedir=/usr/riscv64-linux-gnu/include --with-build-config=bootstrap-lto-lean --enable-link-serialization=2
Thread model: posix
Supported LTO compression algorithms: zlib zstd
gcc version 11.4.0 (Ubuntu 11.4.0-1ubuntu1~22.04)       // <=
```

- usr用のリンカスクリプトを外してmusl(gcc?)のデフォルト
  リンカスクリプトを使用するようにした。
- プログラムヘッダーが複数になり、2番目のヘッダーからは
  ページサイズにアラインしていないアドレスにセットされる
  のでexec()でエラーとなった

```bash
[DEBUG] usertrap: scause: 8, which_dev: 0
[DEBUG] usertrap: call syscall
[DEBUG] exec: vaddr not align: 0x
[DEBUG] syscall: num: 221, ret: -1
[DEBUG] usertrap: syscall called: a0 = 0x00000000ffffffff
[DEBUG] usertrap: scause: 8, which_dev: 0
[DEBUG] usertrap: call syscall
panic: init exiting
```

- 2番目のプログラムヘッダーはアライン指定なくてもエラーと
  しないようにexec.cを修正した。
- __init_libc()でエラーが発生。a5 = *envp[] が0のためか?

```bash
[DEBUG] usertrap: scause: 8, which_dev: 0
[DEBUG] usertrap: call syscall
[DEBUG] syscall: num: 221, ret: 1
[DEBUG] usertrap: syscall called: a0 = 0x0000000000000001
[DEBUG] usertrap: scause: 15, which_dev: 0
usertrap(): unexpected scause 0xf pid=1           // 0000000000010368 <__init_libc>:
            sepc=0x1038e stval=0                  // 1038e:       e380                    sd      s0,0(a5)
[DEBUG] usertrap: killed pid: 1
panic: init exiting
```

- exec.vにenvpを追加
- envpのアドレスが正しく渡されていない
- エラーはinitcode.Sの実行時のもので、initはまだ実行されていなかった。

```bash[INFO ] initlog: init log ok
[INFO ] fsinit: fsinit ok
[DEBUG] usertrap: scause: 8, which_dev: 0
[DEBUG] usertrap: call syscall
[DEBUG] syscall: a0: 0x0000000000000024, a1: 0x000000000000002b, a2: 0x00000000fdf77fbf, a3: 0x00000000ffe3dfff, a4: 0x00000000fefffe7f
[DEBUG] sys_execve: uargv: 0x000000000000002b, uenvp: 0x00000000ffffffbf
[DEBUG] fetchaddr: addr; 0x00000000ffffffbf, p->sz: 0x0000000000001000
[ERROR] sys_execve: fetchaddr error: uenv[0]
[DEBUG] syscall: num: 221, ret: -14
[DEBUG] usertrap: syscall called: a0 = 0x00000000fffffff2
[DEBUG] usertrap: scause: 8, which_dev: 0
[DEBUG] usertrap: call syscall
panic: init exiting
```

- initcode.Sにa2 = 0を追加
- __init_libc()のエラーに戻る

```bash
[DEBUG] usertrap: scause: 8, which_dev: 0
[DEBUG] usertrap: call syscall
[DEBUG] syscall: a0: 0x0000000000000028, a1: 0x000000000000002f, a2: 0x0000000000000000, a3: 0x00000000fff3ffff, a4f
[DEBUG] sys_execve: uargv: 0x000000000000002f, uenvp: 0x0000000000000000
[DEBUG] fetchstr: error in copyinstr()
[ERROR] sys_execve: fetchstr error: envp[0]
[DEBUG] syscall: num: 221, ret: -5
[DEBUG] usertrap: syscall called: a0 = 0x00000000fffffffb
[DEBUG] usertrap: scause: 8, which_dev: 0
[DEBUG] usertrap: call syscall
panic: init exiting
```

- xv6ではtpにhartidを保存する仕様になっていた。
- milkvにはcpuは1つしかないので保存せずr_tp()を使っていた
  箇所は直接0を返すようにした。
- これまで加えた変更をすべてもとに戻した。
- 結果は変わらなかった。そもそも1つだけあるcpuのhartidは
  0なのでtpは0になっているはずだが、その場合、`add a0,a0,-164`に
  よりa0は`0xffffffffffffff5b`ではなく`0xffffffffffffff5c`になるはず。

```bash
[DEBUG] usertrap: scause: 8, which_dev: 0
[DEBUG] usertrap: call syscall
[DEBUG] sys_execve: uargv: 0x000000000000002b
[DEBUG] syscall: num: 221, ret: 1
[DEBUG] usertrap: syscall called: a0 = 0x0000000000000001
[DEBUG] usertrap: scause: 8, which_dev: 0
[DEBUG] usertrap: call syscall
[DEBUG] sys_openat: dirfd: -100, path: console, mode: 0x00000000, flags: 0x00008002
[WARN ] fileopen: console is not found
[DEBUG] syscall: num: 56, ret: -2
[DEBUG] usertrap: syscall called: a0 = 0x00000000fffffffe
[DEBUG] usertrap: scause: 15, which_dev: 0
usertrap(): unexpected scause 0xf pid=1
            sepc=0x4ec stval=0xffffffffffffff5b        // 最初と同じエラー
[DEBUG] usertrap: killed pid: 1
panic: init exiting
```

- tpをデバッグ出力

```bash
[DEBUG] usertrapret: tp: 0x000000008021ed30         # initcode.Sからのusertrapret. entry.Sでセットした値と思われる
[DEBUG] usertrap: scause: 9, which_dev: 1
[DEBUG] usertrapret: tp: 0x0000000000000000         # hartid ? に変化している
[DEBUG] usertrap: scause: 9, which_dev: 1
...                                                 # 以後 tp = 0, scause 9で無限ループ

[DEBUG] usertrap: scause: 8, which_dev: 0           # usertrapret()内のdebgu行をはずした
[DEBUG] usertrap: call syscall
[DEBUG] syscall: tp: 0x0000000000000000             # 最初のyscall時にすでに tpは0
```
