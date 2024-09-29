# U-Boot MMC関連の関数と構造体

## mmc.h

```c
struct dm_mmc_ops {
    // デバイスのエヌメレーション直前まで延期する必要がある構成もある
    int (*deferred_probe)(struct udevice *dev);

    // mmcを再スキャンするために古い構成をクリアするための再初期化
    int (*reinit)(struct udevice *dev);

    // MMCデバイスにコマンドを送信する
    int (*send_cmd)(struct udevice *dev, struct mmc_cmd *cmd,
            struct mmc_data *data);

    // MMCデバイスのI/O速度/幅をセットする
    int (*set_ios)(struct udevice *dev);

    // カードの存在をチェックする
    int (*get_cd)(struct udevice *dev);

    // 書き込み保護が有効になっているかチェックする
    int (*get_wp)(struct udevice *dev);

    // チューニング処理を開始する
    int (*execute_tuning)(struct udevice *dev, uint opcode);

    // dat0が指定の状態になるまで待機する
    int (*wait_dat0)(struct udevice *dev, int state, int timeout_us);

    // デバイスの電圧をセットする
    void (*set_voltage)(struct udevice *dev);

    // mmc_power_off()とmmc_power_on()の間に呼び出されるホスト固有の
    // パワーサイクルシーケンスタスク
    int (*host_power_cycle)(struct udevice *dev);

    // 1回の転送の最大長を取得する
    int (*get_b_max)(struct udevice *dev, void *dst, lbaint_t blkcnt);

    // DDRモードへ切り替える準備を行う
    int (*hs400_prepare_ddr)(struct udevice *dev);
};

// 互換性のための移行関数
int mmc_set_ios(struct mmc *mmc);
int mmc_getcd(struct mmc *mmc);
int mmc_getwp(struct mmc *mmc);
int mmc_execute_tuning(struct mmc *mmc, uint opcode);
int mmc_wait_dat0(struct mmc *mmc, int state, int timeout_us);
int mmc_set_enhanced_strobe(struct mmc *mmc);
int mmc_host_power_cycle(struct mmc *mmc);
int mmc_deferred_probe(struct mmc *mmc);
int mmc_reinit(struct mmc *mmc);
int mmc_get_b_max(struct mmc *mmc, void *dst, lbaint_t blkcnt);
int mmc_hs400_prepare_ddr(struct mmc *mmc);
// 移行関数はここまで

// DDRモードか?
static inline bool mmc_is_mode_ddr(enum bus_mode mode);
// UHSをサポートしているか
static inline bool supports_uhs(uint caps);
// 非ブロックデバイスとしてmmcを作成（使用しない）
struct mmc *mmc_create(const struct mmc_config *cfg, void *priv);

// Probe可能な新規MMCデバイスをセットアップする
int mmc_bind(struct udevice *dev, struct mmc *mmc, const struct mmc_config *cfg);
// mmcを破棄する
void mmc_destroy(struct mmc *mmc);
// mmcのバインドを解く
int mmc_unbind(struct udevice *dev);
// mmcを初期化する（使用しない）
int mmc_initialize(struct bd_info *bis);
// num番目のデバイスを初期化する（使用しない）
int mmc_init_device(int num);
// mmcを初期化する
int mmc_init(struct mmc *mmc);
// チューニングする
int mmc_send_tuning(struct mmc *mmc, u32 opcode, int *cmd_error);
// コマンドを送信する
int mmc_send_cmd(struct mmc *mmc, struct mmc_cmd *cmd, struct mmc_data *data);
// デバイスツリーを解析してホストのcapabilityを取得する（必要な箇所に直書き）
int mmc_of_parse(struct udevice *dev, struct mmc_config *cfg);
// mmcを読み込む
int mmc_read(struct mmc *mmc, u64 src, uchar *dst, int size);
// mmc_voltage_to_mv() - Convert a mmc_voltageをmvに変換する
int mmc_voltage_to_mv(enum mmc_voltage voltage);
// バスクロックを変更する
int mmc_set_clock(struct mmc *mmc, uint clock, bool disable);
// MMCデバイスの数を取得する
int get_mmc_num(void);
//
int mmc_switch(struct mmc *mmc, u8 set, u8 index, u8 value);
//
int mmc_switch_part(struct mmc *mmc, unsigned int part_num);
//
int mmc_hwpart_config(struct mmc *mmc, const struct mmc_hwpart_conf *conf, enum mmc_hwpart_conf_mode mode);
// DSRにセットする
int mmc_set_dsr(struct mmc *mmc, u16 val);
// ブートパーティションとrpmbパーティションのサイズを変更する（使用しない）
int mmc_boot_partition_size_change(struct mmc *mmc, unsigned long bootsize, unsigned long rpmbsize);
// boot時の処理でしか使用しないので今回は不要
int mmc_set_part_conf(struct mmc *mmc, u8 ack, u8 part_num, u8 access);
int mmc_set_boot_bus_width(struct mmc *mmc, u8 width, u8 reset, u8 mode);
int mmc_set_rst_n_function(struct mmc *mmc, u8 enable);
int mmc_rpmb_set_key(struct mmc *mmc, void *key);
int mmc_rpmb_get_counter(struct mmc *mmc, unsigned long *counter);
int mmc_rpmb_read(struct mmc *mmc, void *addr, unsigned short blk, unsigned short cnt, unsigned char *key);
int mmc_rpmb_write(struct mmc *mmc, void *addr, unsigned short blk, unsigned short cnt, unsigned char *key);
int mmc_rpmb_route_frames(struct mmc *mmc, void *req, unsigned long reqlen, void *rsp, unsigned long rsplen);
// ここまで

// デバイスの初期化を開始する。OCRを待たずにすぐに復帰する
int mmc_get_op_cond(struct mmc *mmc, bool quiet);
// デバイスの初期化を開始する。OCRを待たずにすぐに復帰する
int mmc_start_init(struct mmc *mmc);
// mmcドライブのpreinitフラグをセットする（未使用?）
void mmc_set_preinit(struct mmc *mmc, int preinit);
// ボードの電源初期化
void board_mmc_power_init(void);
// ボードのmmc初期化
int board_mmc_init(struct bd_info *bis);
// cpuのMMC初期化
int cpu_mmc_init(struct bd_info *bis);
// 使用しない
int mmc_get_env_addr(struct mmc *mmc, int copy, u32 *env_addr);
// 使用しない
int mmc_get_env_dev(void);
// MMCデバイスのブロックディスクリプタを取得する（sd.cで使用?）
struct blk_desc *mmc_get_blk_desc(struct mmc *mmc);
// 拡張CSDレジスタを読み込む
int mmc_send_ext_csd(struct mmc *mmc, u8 *ext_csd);
// ブートパーティションの書き込み保護
int mmc_boot_wp(struct mmc *mmc);
```

## sdhci.h

```c
struct sdhci_ops {
    int  (*get_cd)(struct sdhci_host *host);
    void (*set_control_reg)(struct sdhci_host *host);
    int  (*set_ios_post)(struct sdhci_host *host);
    void (*set_clock)(struct sdhci_host *host, u32 div);
    int  (*platform_execute_tuning)(struct mmc *host, u8 opcode);
    int  (*set_delay)(struct sdhci_host *host);
    int  (*deferred_probe)(struct sdhci_host *host);
    void (*reset)(struct sdhci_host *host, u8 mask);
    void (*voltage_switch)(struct mmc *mmc);
};

// レジスタアクセス関数
static inline void sdhci_writel(struct sdhci_host *host, u32 val, int reg);
static inline void sdhci_writew(struct sdhci_host *host, u16 val, int reg);
static inline void sdhci_writeb(struct sdhci_host *host, u8 val, int reg);
static inline u32 sdhci_readl(struct sdhci_host *host, int reg);
static inline u16 sdhci_readw(struct sdhci_host *host, int reg);
static inline u8 sdhci_readb(struct sdhci_host *host, int reg);

// Set up the configuration for DWMMC
int sdhci_setup_cfg(struct mmc_config *cfg, struct sdhci_host *host, u32 f_max, u32 f_min);
// Set up a new MMC block device
int sdhci_bind(struct udevice *dev, struct mmc *mmc, struct mmc_config *cfg);
// Add a new SDHCI interface
int add_sdhci(struct sdhci_host *host, u32 f_max, u32 f_min);
void sdhci_set_uhs_timing(struct sdhci_host *host);
// Export the operations to drivers
int sdhci_probe(struct udevice *dev);
int sdhci_set_clock(struct mmc *mmc, unsigned int clock);
struct sdhci_adma_desc *sdhci_adma_init(void);
void sdhci_prepare_adma_table(struct sdhci_adma_desc *table, struct mmc_data *data, dma_addr_t addr);
```

## sdhci-cv180x.c

```c
static void cvi_emmc_pad_setting(void);
static void cvi_sdio1_pad_setting(void);
static void cvi_sdio0_pad_setting(bool reset);
static void cvi_sdio0_pad_function(bool bunplug);
static int cvi_ofdata_to_platdata(struct udevice *dev);
static int cvi_sdhci_bind(struct udevice *dev);
static void cvi_mmc_set_tap(struct sdhci_host *host, u16 tap);
static inline uint32_t CHECK_MASK_BIT(void *_mask, uint32_t bit);
static inline void SET_MASK_BIT(void *_mask, uint32_t bit);
static void reset_after_tuning_pass(struct sdhci_host *host);

int cvi_general_execute_tuning(struct mmc *mmc, u8 opcode);

static void cvi_general_reset(struct sdhci_host *host, u8 mask);
static void cvi_sd_voltage_switch(struct mmc *mmc);

int cvi_get_cd(struct sdhci_host *host);

static int cvi_sdhci_probe(struct udevice *dev);

// ここからは登録定義

struct mmc {
    const struct mmc_config *cfg;    /* provided configuration */
    uint version;
    void *priv;
    uint has_init;
    int high_capacity;
    bool clk_disable; /* true if the clock can be turned off */
    uint bus_width;
    uint clock;
    uint saved_clock;
    enum mmc_voltage signal_voltage;
    uint card_caps;
    uint host_caps;
    uint ocr;
    uint dsr;
    uint dsr_imp;
    uint scr[2];
    uint csd[4];
    uint cid[4];
    ushort rca;
    u8 part_support;
    u8 part_attr;
    u8 wr_rel_set;
    u8 part_config;
    u8 gen_cmd6_time;    /* units: 10 ms */
    u8 part_switch_time;    /* units: 10 ms */
    uint tran_speed;
    uint legacy_speed; /* speed for the legacy mode provided by the card */
    uint read_bl_len;
    uint write_bl_len;
    uint erase_grp_size;    /* in 512-byte sectors */
    struct sd_ssr    ssr;    /* SD status register */
    u64 capacity;
    u64 capacity_user;
    u64 capacity_boot;
    u64 capacity_rpmb;
    u64 capacity_gp[4];
    char op_cond_pending;    /* 1 if we are waiting on an op_cond command */
    char init_in_progress;    /* 1 if we have done mmc_start_init() */
    char preinit;        /* start init as early as possible */
    int ddr_mode;
    struct udevice *dev;    /* Device for this MMC controller */
    u8 *ext_csd;
    u32 cardtype;        /* cardtype read from the MMC */
    enum mmc_voltage current_voltage;
    enum bus_mode selected_mode; /* mode currently used */
    enum bus_mode best_mode; /* best mode is the supported mode with the
                  * highest bandwidth. It may not always be the
                  * operating mode due to limitations when
                  * accessing the boot partitions
                  */
    u32 quirks;
    u8 hs400_tuning;

    enum bus_mode user_speed_mode; /* input speed mode from user */
};

struct mmc_config {
	const char *name;
	const struct mmc_ops *ops;
	uint host_caps;             // in sdhci_host
	uint voltages;              // in sdhci_host
	uint f_min;                 // in cvi_sdhci_host
	uint f_max;                 // in cvi_sdhci_host
	uint b_max;
	unsigned char part_type;
};

// ドライブクラス
struct uclass {
	void *priv_;                    // プライベートデータ
	struct uclass_driver *uc_drv;   // このuclassのドライバ
	struct list_head dev_head;
	struct list_head sibling_node;
};

// ドライバのインスタンス
struct udevice {
	const struct driver *driver;    // このデバイスが使うドライバ
	const char *name;
	void *plat_;            // configデータ
	void *parent_plat_;     // 親バスのconfigデータ
	void *uclass_plat_;     // uclassのconfigデータ
	ulong driver_data;
	struct udevice *parent; // 親
	void *priv_;            // プライベートデータ
	struct uclass *uclass;  // uclassへのポインタ
	void *uclass_priv_;     // uclassのプライベートデータ
	void *parent_priv_;     // 親のプライベートデータ
	struct list_head uclass_node;
	struct list_head child_head;
	struct list_head sibling_node;
	u32 flags_;
	int seq_;
	ulong dma_offset;
};

struct sdhci_host {
    const char *name;
    void *ioaddr;           // MMIOベースアドレス
    unsigned int quirks;
    unsigned int host_caps;
    unsigned int version;
    unsigned int max_clk;   /* Maximum Base Clock frequency */
    unsigned int clk_mul;   /* Clock Multiplier value */
    unsigned int clock;
    struct mmc *mmc;
    const struct sdhci_ops *ops;
    int index;

    int bus_width;                  // バス幅
    struct gpio_desc pwr_gpio;      // 電源GPIO
    struct gpio_desc cd_gpio;       // カード検出GPIO

    uint    voltages;               // 電源電圧

    struct mmc_config cfg;
    // DMA用
    void *align_buffer;
    bool force_align_buffer;
    dma_addr_t start_addr;
    int flags;
};

struct cvi_sdhci_plat {
    struct mmc_config cfg;
    struct mmc mmc;
};

struct cvi_sdhci_host {
    struct sdhci_host host;
    int pll_index;
    int pll_reg;
    int no_1_8_v;
    int is_64_addressing;
    int reset_tx_rx_phy;
    uint32_t mmc_fmax_freq;
    uint32_t mmc_fmin_freq;
    struct reset_ctl reset_ctl;
};

struct cvi_sdhci_driver_data {
    const struct sdhci_ops *ops;
    int index;
};

const struct sdhci_ops cvi_sdhci_sd_ops = {
    .get_cd = cvi_get_cd,
    .platform_execute_tuning = cvi_general_execute_tuning,
    .voltage_switch = cvi_sd_voltage_switch,
    .reset = cvi_general_reset,
};

static const struct cvi_sdhci_driver_data sdhci_cvi_sd_drvdata = {
    .ops = &cvi_sdhci_sd_ops,
    .index = MMC_TYPE_SD,
};

static const struct udevice_id cvi_sdhci_match[] = {
    {
        .compatible = "cvitek,cv180x-sd",
        .data = (ulong)&sdhci_cvi_sd_drvdata,
    },
    { }
};

U_BOOT_DRIVER(cvi_sdhci) = {
    .name = "cvi_sdhci",
    .id = UCLASS_MMC,
    .of_match = cvi_sdhci_match,
    .of_to_plat = cvi_ofdata_to_platdata,
    .bind = cvi_sdhci_bind,
    .probe = cvi_sdhci_probe,
    .priv_auto = sizeof(struct cvi_sdhci_host),
    .plat_auto = sizeof(struct cvi_sdhci_plat),
    .ops = &sdhci_ops,
};
```

## milk-v duoのデバイスツリーのSD要素

```
cv-sd@4310000 {
    compatible = "cvitek,cv180x-sd";
    reg = <0x0 0x4310000 0x0 0x1000>;
    reg-names = "core_mem";
    bus-width = <0x4>;
    cap-sd-highspeed;
    cap-mmc-highspeed;
    sd-uhs-sdr12;
    sd-uhs-sdr25;
    sd-uhs-sdr50;
    sd-uhs-sdr104;
    no-sdio;
    no-mmc;
    src-frequency = <0x165a0bc0>;
    min-frequency = <0x61a80>;
    max-frequency = <0xbebc200>;
    64_addressing;
    reset_tx_rx_phy;
    reset-names = "sdhci";
    pll_index = <0x6>;
    pll_reg = <0x3002070>;
    cvi-cd-gpios = <0xc 0xd 0x1>;
    interrupts = <0x24 0x4>;
    interrupt-parent = <0x4>;
    no-1-8-v;
};
```
