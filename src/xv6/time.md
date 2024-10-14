# 計時機能を付ける

- `sbi_set_timer()`の引数`stime_value`の単位は1/25MHz(40ns)
- この25MHzはcpuの`clock-frequency`値
- trap.c#devintr()でINTERVAL毎にタイマー割り込みが発生するよう設定している
- `INTERVAL = 250000UL = 10 ms`
- タイマー割り込みハンドラは`clockintr()`
- `1 tick = 10 ms`

## `clockintr()`

```c
void clockintr()
{
  acquire(&tickslock);
  ticks++;
  wakeup(&ticks);           // sys_sleepp()がsleep(&ticks);
  release(&tickslock);
}
```

- このハンドラ内で時計を更新する

# RTCを実装する

- データシートに基づき、RTCの較正を行った。
- `RTC_SEC_CNTR_VALUE`が秒カウンタでunix時間を示している。
- unix時間の設定手順は次の通り
    ```c
    write32(RTC_SET_SEC_CNTR_VALUE, 現在のunix時間);       // unix時間を設定
    write32(RTC_SET_SEC_CNTR_TRIG, 1);                     // RTC_SEC_CNTR_VALUEにロード
    while(read32(RTC_SEC_CNTR_VALUE) < 現在のunix時間) {}; // ロードが正しく行われるのを確認
    ```

```bash
// 32KHzクロックの粗調整
[0]rtc_calibration: [7] cvalue: 1138
[0]rtc_calibration: ftune: 0x00000180, offset: 64, ana_calib: 0x00030180
[0]rtc_calibration: [6] cvalue: 800
[0]rtc_calibration: ftune: 0x000001c0, offset: 32, ana_calib: 0x000301c0
[0]rtc_calibration: [5] cvalue: 631
[0]rtc_calibration: ftune: 0x000001a0, offset: 16, ana_calib: 0x000301a0
[0]rtc_calibration: [4] cvalue: 715
[0]rtc_calibration: ftune: 0x00000190, offset: 8, ana_calib: 0x00030190
[0]rtc_calibration: [3] cvalue: 758
[0]rtc_calibration: coarse calib ok! cvalue=758
// 秒カウンタの微調整
[0]rtc_calibration: fine calib: freq int=32989, frac=89
[0]rtc_init: RTC_SEC_CNTR_VALUE: 1728884419
[0]rtc_init: current sec: 1728884428
[0]rtc_init: rtc_init ok!
```
