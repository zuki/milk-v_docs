# errnoが設定されるエラーになる件

```
// xv6-milkv-musl/obj/usr/src/init/init.asm
000000000000046c <__syscall_ret>:
     46c:    1141           add    sp,sp,-16
     46e:    e022           sd    s0,0(sp)
     470:    e406           sd    ra,8(sp)
     472:    77fd           lui    a5,0xfffff
     474:    842a           mv    s0,a0
     476:    00a7e663       bltu    a5,a0,482 <__syscall_ret+0x16>
     47a:    60a2           ld    ra,8(sp)
     47c:    6402           ld    s0,0(sp)
     47e:    0141           add    sp,sp,16
     480:    8082           ret
     482:    0b3000ef       jal    d34 <__errno_location>
     486:    4080043b       negw    s0,s0
     48a:    c100           sw    s0,0(a0)  // ここでアクセス例外が発生
     48c:    557d           li    a0,-1
     48e:    b7f5           j    47a <__syscall_ret+0xe>

0000000000000d34 <__errno_location>:
     d34:    8512           mv    a0,tp     // このtpに適切な値が設定されない
     d36:    f5c50513       add   a0,a0,-164
     d3a:    8082           ret
```

## `__errno_location`で-164を足している件

```c
// arch/riscv64/pthread_arch.h
#define TLS_ABOVE_TP        // tlsはtpの上にある
#define GAP_ABOVE_TP 0

// musr/src/internal/pthread/impl.h

struct pthread {
    /* Part 1 -- these fields may be external or
     * internal (accessed via asm) ABI. Do not change. */
[0]    struct pthread *self;
#ifndef TLS_ABOVE_TP
[_]    uintptr_t *dtv;
#endif
[8]    struct pthread *prev, *next; /* non-ABI */
[24] uintptr_t sysinfo;
#ifndef TLS_ABOVE_TP
#ifdef CANARY_PAD
    uintptr_t canary_pad;
#endif
[_]    uintptr_t canary;
#endif

    /* Part 2 -- implementation details, non-ABI. */
[32]    int tid;
[36]    int errno_val;
[40]    volatile int detach_state;
[44]    volatile int cancel;
[48]    volatile unsigned char canceldisable, cancelasync;
[?]    unsigned char tsd_used:1;
[?]    unsigned char dlerror_flag:1;
[64]    unsigned char *map_base;
[72]    size_t map_size;
[80]    void *stack;
[88]    size_t stack_size;
[96]    size_t guard_size;
[104]    void *result;
[112]    struct __ptcb *cancelbuf;
[120]    void **tsd;
[_]    struct {
[128]        volatile void *volatile head;
[136]        long off;
[144]        volatile void *volatile pending;
    } robust_list;
[152]    int h_errno_val;
[156]    volatile int timer_id;
[160]    locale_t locale;
[164]    volatile int killlock[1];
[168]    char *dlerror_buf;
[176]    void *stdio_locks;

    /* Part 3 -- the positions of these fields relative to
     * the end of the structure is external and internal ABI. */
#ifdef TLS_ABOVE_TP
[184]    uintptr_t canary;
[192]    uintptr_t *dtv;
#endif
};
```

linux上で構造体のサイズとerrno_valの相対位置を調べた

- sizeof(struct pthread) = 200
- pos of errno_val = 36
- 200 - 164 = 36

## `tp`に関わる構造体などの定義

```c
// linux/arch/riscv/kernel/entry.S

ENTRY(handle_exception)
    /*
     * ユーザ空間から来た場合は、ユーザスレッドポインタを保持し、
     * カーネルスレッドポインタをロードする。 カーネルから来た場合は、
     * スクラッチレジスタに0が格納され、現在のTPを継続する
     * If coming from userspace, preserve the user thread pointer and load
     * the kernel thread pointer.  If we came from the kernel, the scratch
     * register will contain 0, and we should continue on the current TP.
     */
    csrrw tp, CSR_SCRATCH, tp
    bnez tp, _save_context

_restore_kernel_tpsp:
    csrr tp, CSR_SCRATCH
    REG_S sp, TASK_TI_KERNEL_SP(tp)

#ifdef CONFIG_VMAP_STACK
    addi sp, sp, -(PT_SIZE_ON_STACK)
    srli sp, sp, THREAD_SHIFT
    andi sp, sp, 0x1
    bnez sp, handle_kernel_stack_overflow
    REG_L sp, TASK_TI_KERNEL_SP(tp)
#endif

_save_context:
    REG_S sp, TASK_TI_USER_SP(tp)
    REG_L sp, TASK_TI_KERNEL_SP(tp)
    addi sp, sp, -(PT_SIZE_ON_STACK)

// include/linux//kbuild.h
#define DEFINE(sym, val) \
    asm volatile("\n.ascii \"->" #sym " %0 " #val "\"" : : "i" (val))

#define OFFSET(sym, str, mem) \
    DEFINE(sym, offsetof(struct str, mem))

// arch/riscv/include/asm/asm.h
#ifdef __ASSEMBLY__
#define __ASM_STR(x)    x
#else
#define __ASM_STR(x)    #x
#endif

#if __riscv_xlen == 64
#define __REG_SEL(a, b) __ASM_STR(a)
#endif

#define REG_L       __REG_SEL(ld, lw)
#define REG_S       __REG_SEL(sd, sw)

// arch/riscv//kernel/asm-offsets.c
void asm_offsets(void)
{
    OFFSET(TASK_THREAD_RA, task_struct, thread.ra);
    OFFSET(TASK_THREAD_SP, task_struct, thread.sp);
    OFFSET(TASK_THREAD_S0, task_struct, thread.s[0]);
    OFFSET(TASK_TI_FLAGS, task_struct, thread_info.flags);
    OFFSET(TASK_TI_PREEMPT_COUNT, task_struct, thread_info.preempt_count);
    OFFSET(TASK_TI_KERNEL_SP, task_struct, thread_info.kernel_sp);
    OFFSET(TASK_TI_USER_SP, task_struct, thread_info.user_sp);
...
DEFINE(PT_SIZE, sizeof(struct pt_regs));
    OFFSET(PT_EPC, pt_regs, epc);
    OFFSET(PT_RA, pt_regs, ra);
    OFFSET(PT_SP, pt_regs, sp);
    OFFSET(PT_TP, pt_regs, tp);

// arch/riscv//include/asm/processor.h
/* CPU-specific state of a task */
struct thread_struct {
    /* Callee-saved registers */
    unsigned long ra;
    unsigned long sp;   /* Kernel mode stack */
    unsigned long s[12];    /* s[0]: frame pointer */
    struct __riscv_d_ext_state fstate;
    unsigned long bad_cause;
};

// arch/riscv//include/asm/thread_info.h
/*
 * entry.Sがすぐにアクセスする必要のある低レベルのタスクデータ
 * - この構造体はキャッシュ1行に完全に収まる必要がある
 * - この構造体のメンバが変更された場合、asm-offsets.cのアセンブリ定数を
 *   適宜更新する必要がある
 * - thread_infoはtask_structのオフセット0に含まれる。これは
 *   tpがthread_infoとtask_structの双方を指すことを意味する。
 */
struct thread_info {
    unsigned long   flags;          /* low level flags */
    int             preempt_count;  /* 0=>preemptible, <0=>BUG */
    /*
     * 以下のスタックポインタはシステムコールや例外のたびに
     * 上書きされる。SPもスタックに保存されるので上書きされても
     * 復元可能である
     */
    long            kernel_sp;  /* カーネルスタックポインタ */
    long            user_sp;    /* ユーザスタックポインタ */
    int             cpu;
};
```
