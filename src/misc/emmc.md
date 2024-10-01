# arm版xv6のemmc関数

```c
static inline uint32_t be2le32(uint32_t x);

static int      emmc_card_init(struct emmc *self);   // カードの初期化: カード情報の取得
static int      emmc_card_reset(struct emmc *self);  // カードのソフトリセット: idle状態へ
static int      emmc_do_data_command(struct emmc *self, int is_write, uint8_t * buf, size_t buf_size, uint32_t block_no);    // データ送信コマンドを実行
static size_t   emmc_do_read(struct emmc *self, uint8_t * buf, size_t buf_size, uint32_t block_no);
static size_t   emmc_do_write(struct emmc *self, uint8_t * buf, size_t buf_size, uint32_t block_no);
static int      emmc_ensure_data_mode(struct emmc *self); // カードをデータ転送モードにする
static uint32_t emmc_get_base_clock();                    // ベースクロックを取得: (MBOXを使用)
static uint32_t emmc_get_clock_divider(uint32_t base_clock, uint32_t target_rate); // 分周比計算
static void     emmc_handle_card_interrupt(struct emmc *self);    // 選択しているカードの状態を取得する
static void     emmc_handle_interrupts(struct emmc *self);        // 割り込み振り分け
static int      emmc_issue_command(struct emmc *self, uint32_t cmd, uint32_t arg, int timeout);        // CMD番号でコマンドを発行
static void     emmc_issue_command_int(struct emmc *self, uint32_t reg, uint32_t arg, int timeout);    // sd_(a)commands[]のインデックスでコマンド発行
static int      emmc_power_on();    // MBOXでSDの電源をON: emmc_card_init()でだけ使用
static void     emmc_power_off();   // SD bus powerをoff (control0[8]): emmc_card_reset()内でエラーリターンする際に使用
static int      emmc_reset_cmd();   // CMDラインをソフトリセット
static int      emmc_reset_dat();   // DATラインをソフトリセット
static int      emmc_switch_clock_rate(uint32_t base_clock, uint32_t target_rate);      // クロック周波数を切り替える: 400KHz -> 25MHz -> 50MHzの2回呼び出される
static int      emmc_timeout_wait(uint64_t reg, uint32_t mask, int value, uint64_t us); // レジスタの指定ビットが指定値になるまで待つ（タイムアウトあり）

void     emmc_clear_interrupt();     // 割り込みフラグをクリアする: 未使用（割り込みを使わず割り込みフラグのポーリングで処理しているため）
int      emmc_init(struct emmc *self, void (*sleep_fn)(void *), void *sleep_arg);    // emmcを初期化
void     emmc_intr(struct emmc *self);    // 割り込みハンドラ: emmcではnoop
size_t   emmc_read(struct emmc *self, void *buf, size_t cnt);    // read
size_t   emmc_write(struct emmc *self, void *buf, size_t cnt);   // write
uint64_t emmc_seek(struct emmc *self, uint64_t off);             // seek
```

## 関数呼び出し

```bash
# emmc
emmc_init
    emmc_card_init: sdversion, vendor, slot-status(0xFCレジスタ)
        emmc_power_on
        emmc_card_reset: host-circuitのreset, clockの停止, control2のクリア, inner-clock, sd-clockの有効化, 割り込みの無効化/クリア/セット, emmc変数クリア
            emmc_get_base_clock: base-clockの取得(MBOX経由,default=100MHz)
            emmc_get_clock_divider: 分周比を計算
            emmc_issue_command(CMD0): CMD0送信: Idle状態に移行
            emmc_issue_command(CMD8, 0x1aa): voltageの確認, SDv2未満のチェック
            emmc_issue_command(CMD5): SDIOかチェック
            emmc_issue_command(ACMD41): 問い合わせACMD41（OCRを取得）
            while(busy) emmc_issue_command(ACMD41, 0x00ff8000 | v2_flags): First + repeated ACMD41
            emmc_switch_clock_rate(25MHz)
            if(supports_18v) emmc_issue_command(CMD11): 1.8Vに切り替え
            emmc_issue_command(CMD2): CIDを取得
            emmc_issue_command(CMD3): RCAを取得, data tranferモードへの移行
            emmc_issue_command(CMD7): RCAを指定してカードを選択
            if (!SDHC) emmc_issue_command(CMD16, 512): SDHC出なかったらブロック長を512
            emmc_issue_command(ACMD51): SCRを取得, カードバージョンの判定
            emmc_issue_command(CMD6, 0x00fffff0): clock周波数の変更
            emmc_issue_command(ACMD6, 2): バス幅を4bit
            割り込みを有効化
```

```bash
emmc_read
    emmc_do_read
        emmc_ensure_data_mode
            if (card_rca) emmc_card_reset: データ転送モードにないのでカードをリセット
            emmc_issue_command(CMD13, rca): select(status)
              case(3): emmc_issue_command(CMD7): カードを選択 => OK return
              case(5): mmc_issue_command(CMD12): 転送をキャンセル
                         emmc_reset_dat: 転送をキャンセル後にカードをリセット
              case(!4): emmc_card_reset: データ転送モードにないのでカードをリセット
            if(!4): emmc_issue_command(CMD13, rca): もさらにう一度チェックしてもだめならエラー
        emmc_do_data_command
            emmc_issue_command(CMD18/CMD17, block_no): multi-blockかsingle-block: 最大3回トライ
                emmc_handle_interrupts: 保留割り込みを処理してクリア
                    if (WRITE_RDY || READ_RDY) emmc_reset_dat: DATラインをリセット
                    if (CARD) emmc_handle_card_interrupt
                        emmc_issue_command_int(CMD13)
                ACMD: emmc_issue_command_int(acmd)
                      emmc_issue_command_int(cmd)
                CMD: emmc_issue_command_int(cmd)

emmc_write
    emmc_do_write
        emmc_ensure_data_mode
        emmc_do_data_command
            emmc_issue_command(WRITE_BLOCK/WRITE_MULTIPLE_BLOCK)
                以下emmc_readに同じ

emmc_seek: 変数操作のみ
emmc_intr: noop
emmc_clear_interrupt: EMMC_INTERRUPTレジスタのread32/write32のみ
```

# Linuxのmmcドライバ

## 3つのホスト構造体

```c
struct mmc_host {
)
```
```c
struct sdhci_host {
    const struct sdhci_ops *ops;    /* Low level hw interface */
    struct mmc_host *mmc;            /* MMC structure */
    struct mmc_host_ops mmc_host_ops;    /* MMC host ops */
}
```

```c
struct sdhci_cvi_host {
    struct sdhci_host *host;
}
```

### これらの紐づけ

- `struct mmc_host *mmc_host;`
- `struct sdhci_host *sdhci_host->mmc = mmc_host;`
- `struct sdhci_cvi_host *cvi_host->host = sdhci_host;`

## mmcの初期化

### `mmc.c`

```bash
mmc_attach_mmc(mmc_host)
    mmc_set_bus_mode(mmc_host, MMC_BUSMODE_OPENDRAIN);
    mmc_send_op_cond(mmc_host, 0, &ocr);
    mmc_attach_bus(mmc_host, &mmc_ops);
    mmc_select_voltage(mmc_host, ocr);
    mmc_init_card(mmc_host, rocr, NULL);
    mmc_release_host(mmc_host);
    mmc_add_card(mmc_host->card);
    mmc_claim_host(mmc_host);
```

### `sdhci.c`

```bash
sdhci_add_host(sdhci_host)
    sdhci_setup_host(sdhci_host);
    sdhci_init(sdhci_host);
    mmc_add_host(mmc_host);

sdhci_reset(struct sdhci_host *host, u8 mask):
    write8(SDHCI_SOFTWARE_RESET, mask)して安定するのを待つ

static void sdhci_do_reset(struct sdhci_host *host, u8 mask): sdhci_init()から呼び出し
    sdhci_cv180x_sd_reset(host, mask)
    if (mask & SDHCI_RESET_ALL) host->preset_enabled = false;

static void sdhci_init(struct sdhci_host *host, int soft): sdhci_add_host()から呼び出し
    sdhci_do_reset: softリセットの場合はCMD/DATラインのリセット、それ以外はSDHCI_RESET_ALL
```

### `sdhci-pltfm.c`

```bash
struct sdhci_host *sdhci_pltfm_init(struct platform_device *pdev,
                    const struct sdhci_pltfm_data *pdata,
                    size_t priv_size):
    host = sdhci_alloc_host(): mmc/sdhci/sdhci_cvi_hostの割当
    mmc/sdhci_hostの紐づけ、sdhci_hostのデータ設定
```

### `sdhci-cv180x.c`

```bash
sdhci_cvi_probe()
    sdhci_pltfm_init():
    sdhci/sdhci_cvi_hostの紐づけ、sdhci_cvi_hostのデータ設定
    sdhci_cv180x_sd_voltage_restore(host, false); # 電圧を3.3V
    sdhci_add_host(sdhci_host)

sdhci_cv180x_sd_reset(struct sdhci_host *host, u8 mask)
    sdhci_cvi_reset_helper()
        割り込みを無効
        sdhci_reset()
        割り込みを有効
```

## OPS

### `sdgcu-cv180x.c`

```c
static const struct sdhci_ops sdhci_cv180x_sd_ops = {
    .reset                   = sdhci_cv180x_sd_reset,
    .set_clock               = sdhci_set_clock,
    .set_power               = sdhci_cv180x_sd_set_power,
    .set_bus_width           = sdhci_set_bus_width,
    .get_max_clock           = sdhci_cvi_general_get_max_clock,
    .voltage_switch          = sdhci_cv180x_sd_voltage_switch,
    .set_uhs_signaling       = sdhci_cvi_general_set_uhs_signaling,
    .platform_execute_tuning = sdhci_cv180x_general_execute_tuning,
    .select_drive_strength   = sdhci_cv180x_general_select_drive_strength,
    .dump_vendor_regs        = sdhci_cv180x_sd_dump_vendor_regs,
    .adma_write_desc         = cvi_adma_write_desc,
};

static const struct sdhci_pltfm_data sdhci_cv180x_sd_pdata = {
    .ops     = &sdhci_cv180x_sd_ops,
    .quirks  = SDHCI_QUIRK_INVERTED_WRITE_PROTECT | SDHCI_QUIRK_CAP_CLOCK_BASE_BROKEN,
    .quirks2 = SDHCI_QUIRK2_PRESET_VALUE_BROKEN,
};
```

### `sdhci-pltfm.c`

```c
static const struct sdhci_ops sdhci_pltfm_ops = {
    .set_clock         = sdhci_set_clock,
    .set_bus_width     = sdhci_set_bus_width,
    .reset             = sdhci_reset,
    .set_uhs_signaling = sdhci_set_uhs_signaling,
};
```

### `sdhci.c`

```c
static const struct mmc_host_ops sdhci_ops = {
    .request                     = sdhci_request,
    .post_req                    = sdhci_post_req,
    .pre_req                     = sdhci_pre_req,
    .set_ios                     = sdhci_set_ios,
    .get_cd                      = sdhci_get_cd,
    .get_ro                      = sdhci_get_ro,
    .hw_reset                    = sdhci_hw_reset,
    .enable_sdio_irq             = sdhci_enable_sdio_irq,
    .ack_sdio_irq                = sdhci_ack_sdio_irq,
    .start_signal_voltage_switch = sdhci_start_signal_voltage_switch,
    .prepare_hs400_tuning        = sdhci_prepare_hs400_tuning,
    .execute_tuning              = sdhci_execute_tuning,
    .select_drive_strength       = sdhci_select_drive_strength,
    .card_event                  = sdhci_card_event,
    .card_busy                   = sdhci_card_busy,
};
```

### `mmc.c`

```c
static const struct mmc_bus_ops mmc_ops = {
    .remove          = mmc_remove,
    .detect          = mmc_detect,
    .suspend         = mmc_suspend,
    .resume          = mmc_resume,
    .runtime_suspend = mmc_runtime_suspend,
    .runtime_resume  = mmc_runtime_resume,
    .alive           = mmc_alive,
    .shutdown        = mmc_shutdown,
    .hw_reset        = _mmc_hw_reset,
};
```
