# Pageシステム (BuddyとSlab) を導入する

- kallocの代わりにBuddyシステムとSlabキャッシュを導入する
- 第1次移行処置として`kalloc()`, `krfrre()`を`buddy_alloc()`, `buddy_free()`へのスタブとして残した。

## `Load access fault`が発生してシステムが無限リセット

```bash
tarting kernel ...


xv6 kernel is booting

SBI specification v0.3 detected
SBI TIME extension detected
SBI IPI extension detected
SBI RFNC extension detected
SBI HSM extension detected

[0]mappages: va: 0x04140000, size: 0x00001000, pa: 0x04140000
[0]mappages: a: 0x04140000, pa: 0x04140000, last: 0x04140000
[0]mappages: *pte: 0x010500c7

[0]mappages: va: 0x03000000, size: 0x00001000, pa: 0x03000000
[0]mappages: a: 0x03000000, pa: 0x03000000, last: 0x03000000
[0]mappages: *pte: 0x00c000c7

[0]mappages: va: 0x03002000, size: 0x00001000, pa: 0x03002000
[0]mappages: a: 0x03002000, pa: 0x03002000, last: 0x03002000
[0]mappages: *pte: 0x00c008c7

[0]mappages: va: 0x03001000, size: 0x00001000, pa: 0x03001000
[0]mappages: a: 0x03001000, pa: 0x03001000, last: 0x03001000
[0]mappages: *pte: 0x00c004c7

[0]mappages: va: 0x03003000, size: 0x00001000, pa: 0x03003000
[0]mappages: a: 0x03003000, pa: 0x03003000, last: 0x03003000
[0]mappages: *pte: 0x00c00cc7

[0]mappages: va: 0x70000000, size: 0x00400000, pa: 0x70000000 <= PLICのmapping
[0]mappages: a: 0x70000000, pa: 0x70000000, last: 0x703ff000
[0]mappages: *pte: 0x1c0000c7
[0]mappages: a: 0x70001000, pa: 0x70001000
[0]mappages: *pte: 0x1c0004c7
[0]mappages: a: 0x70002000, pa: 0x70002000

// buddy版
[0]walk: va: 0x70001000, level: 2, PX: 1, pte: 809ff008, *pte: 0x00000000
[0]walk: 2: pagetable: 0x809ff000, pte: 0x809ff008, *pte: 0x2027fc01
[0]walk: va: 0x70001000, level: 1, PX: 384, pte: 809ffc00, *pte: 0x00000000
[0]walk: 2: pagetable: 0x809ff000, pte: 0x809ffc00, *pte: 0x2027fc01
[0]walk: va: 0x70002000, level: 2, PX: 1, pte: 809ff008, *pte: 0x1c0004c7
[0]walk: 1: pagetable: 0x70001000

// kalloc版
[0]walk: va: 0x70000000, level: 2, PX: 1, pte: 83dff008, *pte: 0x00000000
[0]walk: 2: pagetable: 0x83dfb000, pte: 0x83dff008, *pte: 0x20f7ec01
[0]walk: va: 0x70000000, level: 1, PX: 384, pte: 83dfbc00, *pte: 0x00000000
[0]walk: 2: pagetable: 0x83dfa000, pte: 0x83dfbc00, *pte: 0x20f7e801
[0]walk: va: 0x70001000, level: 2, PX: 1, pte: 83dff008, *pte: 0x20f7ec01
[0]walk: 1: pagetable: 0x83dfb000
[0]walk: va: 0x70001000, level: 1, PX: 384, pte: 83dfbc00, *pte: 0x20f7e801
[0]walk: 1: pagetable: 0x83dfa000
[0]walk: va: 0x70002000, level: 2, PX: 1, pte: 83dff008, *pte: 0x20f7ec01
[0]walk: 1: pagetable: 0x83dfb000
[0]walk: va: 0x70002000, level: 1, PX: 384, pte: 83dfbc00, *pte: 0x20f7e801
[0]walk: 1: pagetable: 0x83dfa000


// u-bootが出しているエラー: arch/riscv/libinterrupts.c
Unhandled exception: Load access fault      // <=
// EOC=epc, RA=regs->ra, tval=例外が発生したメモリアドレス
EPC: 0000000080200f6e RA: 00000000802011d6 TVAL: 0000000070001c00 <= PLICのアドレス
// 再配置により調整されたアドレス
EPC: 000000007c508f6e RA: 000000007c5091d6 reloc adjusted

Code: 7913 1ff9 090e 9926 3483 0009 f793 0014 (cb95)


resetting ...
C.SCS/0/0.WD.URPL.SDI/25000000/6000000.BS/SD.PS.SD/0x0/0x1000/0x1000/0.PE.BS.SD/0x1000/0xba00/0xba00/0.BE.J.
```

## エラー箇所のobjdump: `vm.c#mappages()`関数

```bash
  a = PGROUNDDOWN(va);
    802010fc:   77fd                    lui     a5,0xfffff
  last = PGROUNDDOWN(va + size - 1);
    802010fe:   fff58a93                add     s5,a1,-1
    80201102:   9ab2                    add     s5,s5,a2
  a = PGROUNDDOWN(va);
    80201104:   00f5f4b3                and     s1,a1,a5
    80201108:   8a2a                    mv      s4,a0
    8020110a:   8b3a                    mv      s6,a4
  last = PGROUNDDOWN(va + size - 1);
    8020110c:   00fafab3                and     s5,s5,a5
    80201110:   409689b3                sub     s3,a3,s1
    a += PGSIZE;
    80201114:   6b85                    lui     s7,0x1
    if((pte = walk(pagetable, a, 1)) == 0)
    80201116:   4605                    li      a2,1
    80201118:   85a6                    mv      a1,s1
    8020111a:   8552                    mv      a0,s4
    8020111c:   00998933                add     s2,s3,s1
    80201120:   00000097                auipc   ra,0x0
    80201124:   e08080e7                jalr    -504(ra) # 80200f28 <walk>
    80201128:   c521                    beqz    a0,80201170 <mappages+0x9c>
```

## buddy.cのバグのようだ

```bash
[0]buddy_init: pages: 0x805ac000, size: 0x00054000, PAGE_START: 0x80600000
[0]buddy_init: sizeof(struct page): 0x0018

[0]buddy_init: pages[0]: address: 0x805ac000, order: 10, flags: 0x00000001
[0]buddy_init: pages[1024]: address: 0x805b2000, order: 10, flags: 0x00000001
[0]buddy_init: pages[2048]: address: 0x805b8000, order: 10, flags: 0x00000001
[0]buddy_init: pages[3072]: address: 0x805be000, order: 10, flags: 0x00000001
[0]buddy_init: pages[4096]: address: 0x805c4000, order: 10, flags: 0x00000001
[0]buddy_init: pages[5120]: address: 0x805ca000, order: 10, flags: 0x00000001
[0]buddy_init: pages[6144]: address: 0x805d0000, order: 10, flags: 0x00000001
[0]buddy_init: pages[7168]: address: 0x805d6000, order: 10, flags: 0x00000001
[0]buddy_init: pages[8192]: address: 0x805dc000, order: 10, flags: 0x00000001
[0]buddy_init: pages[9216]: address: 0x805e2000, order: 10, flags: 0x00000001
[0]buddy_init: pages[10240]: address: 0x805e8000, order: 10, flags: 0x00000001
[0]buddy_init: pages[11264]: address: 0x805ee000, order: 10, flags: 0x00000001
[0]buddy_init: pages[12288]: address: 0x805f4000, order: 10, flags: 0x00000001
[0]buddy_init: pages[13312]: address: 0x805fa000, order: 10, flags: 0x00000001
[0]buddy_init:
free_lists[10].head: 0x805ac000

[0]slab_cache_init: SLAB_HEADER_SIZE: 28

[0]slab_cache_init: i: 0, page_size: 4096, limits: 504
[0]slab_cache_init: i: 1, page_size: 8192, limits: 1016
[0]slab_cache_init: i: 2, page_size: 16384, limits: 2040
[0]slab_cache_init: i: 3, page_size: 32768, limits: 4088
[0]slab_cache_init: i: 4, page_size: 65536, limits: 8184
[0]slab_cache_init: i: 5, page_size: 131072, limits: 16376
[0]slab_cache_init: i: 6, page_size: 262144, limits: 32760
[0]slab_cache_init: i: 7, page_size: 524288, limits: 65528
[0]slab_cache_init: i: 8, page_size: 1048576, limits: 131064
[0]slab_cache_init: i: 9, page_size: 2097152, limits: 262136
[0]slab_cache_init: i: 10, page_size: 4194304, limits: 524280
[0]buddy_alloc: match order 0 list
[0]buddy_alloc: no page in order 0 list
[0]buddy_pull_block: from 0, to: 9
[0]buddy_alloc: size: 4096, page: index: 1023, flags: 0x00000002, order: 0, addr: 0x809ff000
[0]kalloc: addr: 0x809ff000
[0]buddy_alloc: match order 0 list
[0]buddy_alloc: no page in order 0 list
[0]buddy_pull_block: from 0, to: 9
[0]buddy_alloc: size: 4096, page: index: 1023, flags: 0x00000002, order: 0, addr: 0x809ff000
[0]kalloc: addr: 0x809ff000
[0]buddy_alloc: match order 0 list
[0]buddy_alloc: no page in order 0 list
[0]buddy_pull_block: from 0, to: 9
[0]buddy_alloc: size: 4096, page: index: 1023, flags: 0x00000002, order: 0, addr: 0x809ff000
[0]kalloc: addr: 0x809ff000
[0]buddy_alloc: match order 0 list
[0]buddy_alloc: no page in order 0 list
[0]buddy_pull_block: from 0, to: 9
[0]buddy_alloc: size: 4096, page: index: 1023, flags: 0x00000002, order: 0, addr: 0x809ff000
[0]kalloc: addr: 0x809ff000
[0]buddy_alloc: match order 0 list
[0]buddy_alloc: no page in order 0 list
[0]buddy_pull_block: from 0, to: 9
[0]buddy_alloc: size: 4096, page: index: 1023, flags: 0x00000002, order: 0, addr: 0x809ff000
[0]kalloc: addr: 0x809ff000
[0]buddy_alloc: match order 0 list
[0]buddy_alloc: no page in order 0 list
[0]buddy_pull_block: from 0, to: 9
[0]buddy_alloc: size: 4096, page: index: 1023, flags: 0x00000002, order: 0, addr: 0x809ff000
[0]kalloc: addr: 0x809ff000
[0]buddy_alloc: match order 0 list
[0]buddy_alloc: no page in order 0 list
[0]buddy_pull_block: from 0, to: 9
[0]buddy_alloc: size: 4096, page: index: 1023, flags: 0x00000002, order: 0, addr: 0x809ff000
[0]kalloc: addr: 0x809ff000
[0]buddy_alloc: match order 0 list
[0]buddy_alloc: no page in order 0 list
[0]buddy_pull_block: from 0, to: 9
[0]buddy_alloc: size: 4096, page: index: 1023, flags: 0x00000002, order: 0, addr: 0x809ff000
[0]kalloc: addr: 0x809ff000
[0]buddy_alloc: match order 0 list
[0]buddy_alloc: no page in order 0 list
[0]buddy_pull_block: from 0, to: 9
[0]buddy_alloc: size: 4096, page: index: 1023, flags: 0x00000002, order: 0, addr: 0x809ff000
[0]kalloc: addr: 0x809ff000
[0]buddy_alloc: match order 0 list
[0]buddy_alloc: no page in order 0 list
[0]buddy_pull_block: from 0, to: 9
[0]buddy_alloc: size: 4096, page: index: 1023, flags: 0x00000002, order: 0, addr: 0x809ff000
[0]kalloc: addr: 0x809ff000
[0]walk: va: 0x70000000, level: 2, PX: 1, pte: 809ff008, *pte: 0x00000000
[0]buddy_alloc: match order 0 list
[0]buddy_alloc: no page in order 0 list
[0]buddy_pull_block: from 0, to: 9
[0]buddy_alloc: size: 4096, page: index: 1023, flags: 0x00000002, order: 0, addr: 0x809ff000
[0]kalloc: addr: 0x809ff000
[0]walk: 2: pagetable: 0x809ff000, pte: 0x809ff008, *pte: 0x2027fc01
[0]walk: va: 0x70000000, level: 1, PX: 384, pte: 809ffc00, *pte: 0x00000000
[0]buddy_alloc: match order 0 list
[0]buddy_alloc: no page in order 0 list
[0]buddy_pull_block: from 0, to: 9
[0]buddy_alloc: size: 4096, page: index: 1023, flags: 0x00000002, order: 0, addr: 0x809ff000
[0]kalloc: addr: 0x809ff000
[0]walk: 2: pagetable: 0x809ff000, pte: 0x809ffc00, *pte: 0x2027fc01
[0]walk: va: 0x70001000, level: 2, PX: 1, pte: 809ff008, *pte: 0x00000000
[0]buddy_alloc: match order 0 list
[0]buddy_alloc: no page in order 0 list
[0]buddy_pull_block: from 0, to: 9
[0]buddy_alloc: size: 4096, page: index: 1023, flags: 0x00000002, order: 0, addr: 0x809ff000
[0]kalloc: addr: 0x809ff000
[0]walk: 2: pagetable: 0x809ff000, pte: 0x809ff008, *pte: 0x2027fc01
[0]walk: va: 0x70001000, level: 1, PX: 384, pte: 809ffc00, *pte: 0x00000000
[0]buddy_alloc: match order 0 list
[0]buddy_alloc: no page in order 0 list
[0]buddy_pull_block: from 0, to: 9
[0]buddy_alloc: size: 4096, page: index: 1023, flags: 0x00000002, order: 0, addr: 0x809ff000
[0]kalloc: addr: 0x809ff000
[0]walk: 2: pagetable: 0x809ff000, pte: 0x809ffc00, *pte: 0x2027fc01
[0]walk: va: 0x70002000, level: 2, PX: 1, pte: 809ff008, *pte: 0x1c0004c7
[0]walk: 1: pagetable: 0x70001000
[0]walk: Unhandled exception: Load access fault
EPC: 00000000802006f0 RA: 000000008020103e TVAL: 0000000070001c00
EPC: 000000007c5086f0 RA: 000000007c50903e reloc adjusted

Code: b7e5 0097 0000 80e7 b340 2905 b599 7159 (f022)
```

## バグが判明

- `buddy.c`ではなく使う側 (`kalloc.c`) の問題だった
- `buddy_alloc()`で取得したpageに`_page_cleanup_`をつけていたため、関数を抜けた段階で`buddy_free()`されていた
- `_page_cleanup_`の使い方を学習すること

```bash
Starting kernel ...

xv6 kernel is booting

SBI specification v0.3 detected
SBI TIME extension detected
SBI IPI extension detected
SBI RFNC extension detected
SBI HSM extension detected

[0]emmc_issue_command_int: waiting for command complete interrupt 1: irpts=0b0000_0000_0000_0001_1000_0000_0000_0000
[0]emmc_card_reset: found valid version 3.0x SD card

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
000001c0: 0200 0c28 2108 0100 0000 0000 0200 0000
000001d0: 0000 0000 0000 0000 0000 0000 0000 0000
000001e0: 0000 0000 0000 0000 0000 0000 0000 0000
000001f0: 0000 0000 0000 0000 0000 0000 0000 55aa
[0]sd_init: partition[0]: TYPE: 12, LBA = 0x1, #SECS = 0x20000

sd_init ok
init: starting sh
$ ls
.              1 1 1024
..             1 1 1024
README.md      2 2 493
cat            2 3 36456
echo           2 4 35064
forktest       2 5 16392
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
```
