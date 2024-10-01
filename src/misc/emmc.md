# arm版xv6のemmc関数

```c
static inline uint32_t be2le32(uint32_t x);

static int      emmc_card_init(struct emmc *self);	＃ カードの初期化: カード情報の取得
static int      emmc_card_reset(struct emmc *self);  # カードのソフトリセット: idle状態へ
static int      emmc_do_data_command(struct emmc *self, int is_write, uint8_t * buf, size_t buf_size, uint32_t block_no);	# データ送信コマンドを実行
static size_t   emmc_do_read(struct emmc *self, uint8_t * buf, size_t buf_size, uint32_t block_no);
static size_t   emmc_do_write(struct emmc *self, uint8_t * buf, size_t buf_size, uint32_t block_no);
static int      emmc_ensure_data_mode(struct emmc *self); # カードをデータ転送モードにする
static uint32_t emmc_get_base_clock();				# ベースクロックを取得: (MBOXを使用)
static uint32_t emmc_get_clock_divider(uint32_t base_clock, uint32_t target_rate); # 分周比計算
static void     emmc_handle_card_interrupt(struct emmc *self);	# 選択しているカードの状態を取得する
static void     emmc_handle_interrupts(struct emmc *self);		# 割り込み振り分け
static int      emmc_issue_command(struct emmc *self, uint32_t cmd, uint32_t arg, int timeout);		# CMD番号でコマンドを発行
static void     emmc_issue_command_int(struct emmc *self, uint32_t reg, uint32_t arg, int timeout);	# sd_(a)commands[]のインデックスでコマンド発行
static int      emmc_power_on();	# MBOXでSDの電源をON: emmc_card_init()でだけ使用
static void     emmc_power_off();	# SD bus powerをoff (control0[8]): emmc_card_reset()内でエラーリターンする際に使用
static int      emmc_reset_cmd();	# CMDラインをソフトリセット
static int      emmc_reset_dat();	# DATラインをソフトリセット
static int      emmc_switch_clock_rate(uint32_t base_clock, uint32_t target_rate);	# クロック周波数を切り替える: 400KHz -> 25MHz -> 50MHzの2回呼び出される
static int      emmc_timeout_wait(uint64_t reg, uint32_t mask, int value, uint64_t us);	# レジスタの指定ビットが指定値になるまで待つ（タイムアウトあり）

void     emmc_clear_interrupt();	# 割り込みフラグをクリアする: 未使用（割り込みを使わず割り込みフラグのポーリングで処理しているため）
int      emmc_init(struct emmc *self, void (*sleep_fn)(void *), void *sleep_arg);	# emmcを初期化
void     emmc_intr(struct emmc *self);	# 割り込みハンドラ: emmcではnoop
size_t   emmc_read(struct emmc *self, void *buf, size_t cnt);	# read
size_t   emmc_write(struct emmc *self, void *buf, size_t cnt);	# write
uint64_t emmc_seek(struct emmc *self, uint64_t off);			# seek
```

## 関数呼び出し図

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
# mmc
mmc_init
	sdhci_init: 規定値のセット、
		sdhci_setup_cfg: カードのcapabilityの取得
		sdhci_host_init
	mmc_start_init
		mmc_get_op_cond
	mmc_complete_init
		mmc_complete_op_cond
		mmc_startup

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

emmc_issue_command_int
	レジスタ操作による実際のコマンド処理
```

# milk-v版のmmc

## リセット処理

1. sdhci.c#sdhci_reset(): cmd/data両lineのソフトリセット
2. sdhci_cv180x.c#cvi_general_reset()
	1. UHS_MODE_SELに合わせた処理

# linuxのmmc

- `drivers/mmc/host/cvitek/sdhci-cv180x.c`

```bash
$ egrep "^static" sdhci-cv180x.c
static char *cvi_get_card_type(unsigned int sd_type)
static inline int is_card_uhs(unsigned char timing)
static inline int is_card_hs(unsigned char timing)
static void sdhci_cv180x_sd_setup_pad(struct sdhci_host *host, bool bunplug)
static void sdhci_cv180x_sd_setup_io(struct sdhci_host *host, bool reset)
static void sdhci_cvi_reset_helper(struct sdhci_host *host, u8 mask)
static void reset_after_tuning_pass(struct sdhci_host *host)
static inline uint32_t CHECK_MASK_BIT(void *_mask, uint32_t bit)
static inline void SET_MASK_BIT(void *_mask, uint32_t bit)
static int sdhci_cv180x_general_select_drive_strength(struct sdhci_host *host, struct mmc_card *card, unsigned int max_dtr, int host_drv, int card_drv, int *drv_type)
static void sdhci_cvi_general_set_uhs_signaling(struct sdhci_host *host, unsigned int uhs)
static unsigned int sdhci_cvi_general_get_max_clock(struct sdhci_host *host)
static void sdhci_cvi_cv180x_set_tap(struct sdhci_host *host, unsigned int tap)
static int sdhci_cv180x_general_execute_tuning(struct sdhci_host *host, u32 opcode)
static void sdhci_cv180x_sd_reset(struct sdhci_host *host, u8 mask)
static void sdhci_cv180x_sd_set_power(struct sdhci_host *host, unsigned char mode, unsigned short vdd)
static void sdhci_cv180x_sd_dump_vendor_regs(struct sdhci_host *host)
static void cvi_adma_write_desc(struct sdhci_host *host, void **desc, dma_addr_t addr, int len, unsigned int cmd)
static unsigned long sdhci_get_time_ms(void)
static void sdhci_cvi_cd_debounce_work(struct work_struct *work)
static irqreturn_t sdhci_cvi_cd_handler(int irq, void *dev_id)
static int sdhci_cvi_probe(struct platform_device *pdev)
static int sdhci_cvi_remove(struct platform_device *pdev)
