# milk-v duo純正Linux_SDKコンパイルプロセス

[オリジナル](https://forum.sophgo.com/t/milk-v-duo-linux-sdk/295)

この記事では主にmilk-v duo純正のLinux_SDKによるfip.binの
コンパイルプロセスを紹介します。

## 1. プリコンパイル準備

新バージョンのSDKは`build_milkv.sh`スクリプトによりワン
クリックでコンパイルされます。

```bash
MILKV_BOARD_CONFIG=${MILKV_BOARD_DIR}/boardconfig-${MILKV_BOARD}.sh

function prepare_env()
{
  source ${MILKV_BOARD_CONFIG}

  source build/${MV_BUILD_ENV} > /dev/null 2>&1
  defconfig ${MV_BOARD_LINK} > /dev/null 2>&1

  echo "OUTPUT_DIR: ${OUTPUT_DIR}"  # @build/milkvsetup.sh
}
```

各環境変数の実際の値は次のとおりです。

- ${MV_BOARD_CPU}=cv1800b
- ${MV_VENDOR}=milkv
- ${MILKV_BOARD_DIR}=milkv
- ${MILKV_BOARD}=milkv-duo
- ${MV_BUILD_ENV}=milkvsetup.sh
- ${V_BOARD_LINK MV_BOARD_LINK}=cv1800b_milkv_duo_sd
- OUTPUT_DIR: /home/share/samba/risc-v/duo-buildroot-sdk/install/soc_cv1800b_milkv_duo_sd

これらの変数は後で使用します。

`README.md`には段階的にコンパイルする方法についても
記載されています。

```bash
export MILKV_BOARD=milkv-duo
source milkv/boardconfig-milkv-duo.sh

source build/milkvsetup.sh
defconfig cv1800b_milkv_duo_sd
clean_all
build_all
pack_sd_image
```

生成されたファームウェアは`install/soc_cv1800b_milkv_duo_sd/milkv-duo.img`にあります。`build_milkv.sh`もこれらの関数を
段階的に呼び出しています。

## 2. コンパイルの開始

コンパイルのはじめにはclean_all関数が呼び出されます。

```bash
function milkv_duo_build()
{
  # clean old img
  old_image_count=`ls ${OUTPUT_DIR}/*.img* | wc -l`
  if [ ${old_image_count} -ge 0 ]; then
    pushd ${OUTPUT_DIR}
    rm -rf *.img*
    popd
  fi

  clean_all
  build_all
  if [ $? -eq 0 ]; then
    print_info "Build board ${MILKV_BOARD} success!"
  else
    print_err "Build board ${MILKV_BOARD} failed!"
    exit 1
  fi
}
```

`build/milkvsetup.sh`ファイルにある`build_all`関数に注目して
ください。

```bash
function build_all()
{
  # build bsp
  build_uboot || return $?
  build_kernel || return $?
  build_middleware || return $?
  pack_access_guard_turnkey_app || return $?
  pack_ipc_turnkey_app || return $?
  pack_boot || return $?
  pack_cfg || return $?
  pack_rootfs || return $?
  pack_data
  pack_system || return $?
  copy_tools
  pack_upgrade
}
```

上記の関数に従ってまずbuild_uboot関数が呼び出されます。

## 3. ubootのコンパイル

1. `build_uboot`関数は`build/cvisetup.sh`ファイルで定義
   されています。

    ```bash
    function build_uboot()
    {(
    print_notice "Run ${FUNCNAME[0]}() function"
    _build_uboot_env
    _build_opensbi_env
    _link_uboot_logo

    cd "$BUILD_PATH" || return
    [[ "$CHIP_ARCH" == CV182X ]] || [[ "$CHIP_ARCH" == CV183X ]] && \
        cp -f "$OUTPUT_DIR"/fip_pre/fip_pre_${ATF_KEY_SEL}.bin \
        "$OUTPUT_DIR"/fip_pre/fip_pre.bin

    make u-boot
    )}
    ```

2. `build/Makefile`ファイルの208行目を見ると`u-bootがu-boot-dep`
   に依存しています。

    ```Makefile
    u-boot: u-boot-dep
    ```

   `u-boot-dep`を探すと（ファイル数が多い）`build/.config`に
   `CONFIG_FIP_V2=y`が定義されており、`scripts/fip_v2.mk`ファイルの
   48行目に見つかりました。

    Makefile

    ```Makefile
    ifeq (${CONFIG_FIP_V1},y)
    include scripts/fip_v1.mk
    else ifeq (${CONFIG_FIP_V2},y)
    include scripts/fip_v2.mk
    else
    $(error no fip version)
    endif
    ```

    fip_v2.mk

    ```Makefile
    u-boot-dep: fsbl-build ${OUTPUT_DIR}/elf
    $(call print_target)
    ifeq ($(call qstrip,${CONFIG_ARCH}),riscv)
    ${Q}cp ${OPENSBI_PATH}/build/platform/generic/firmware/fw_payload.bin ${OUTPUT_DIR}/fw_payload_uboot.bin
    ${Q}cp ${OPENSBI_PATH}/build/platform/generic/firmware/fw_payload.elf ${OUTPUT_DIR}/elf/fw_payload_uboot.elf
    endif
    ```
3. `u-boot-dep`は`fsbl-build`に依存します。fsbl-buildの定義は
    `scripts/fip_v2.mk`ファイルの28行目で定義されています。

    ```Makefile
    ifeq ($(call qstrip,${CONFIG_ARCH}),riscv)
    fsbl-build: opensbi
    endif

    ifeq (${CONFIG_ENABLE_FREERTOS},y)
    fsbl-build: rtos
    fsbl%: export BLCP_2ND_PATH=${FREERTOS_PATH}/cvitek/install/bin/cvirtos.bin
    fsbl%: export RTOS_DUMP_PRINT_ENABLE=$(CONFIG_ENABLE_RTOS_DUMP_PRINT)
    fsbl%: export RTOS_DUMP_PRINT_SZ_IDX=$(CONFIG_DUMP_PRINT_SZ_IDX)
    fsbl%: export RTOS_FAST_IMAGE_TYPE=${CONFIG_FAST_IMAGE_TYPE}
    fsbl%: export RTOS_ENABLE_FREERTOS=${CONFIG_ENABLE_FREERTOS}
    endif
    fsbl%: export FSBL_SECURE_BOOT_SUPPORT=${CONFIG_FSBL_SECURE_BOOT_SUPPORT}
    fsbl%: export ARCH=$(call qstrip,${CONFIG_ARCH})
    fsbl%: export OD_CLK_SEL=${CONFIG_OD_CLK_SEL}
    fsbl%: export VC_CLK_OVERDRIVE=${CONFIG_VC_CLK_OVERDRIVE}

    fsbl-build: u-boot-build memory-map
    $(call print_target)
    ${Q}mkdir -p ${FSBL_PATH}/build
    ${Q}ln -snrf -t ${FSBL_PATH}/build ${CVI_BOARD_MEMMAP_H_PATH}
    ${Q}$(MAKE) -j${NPROC} -C ${FSBL_PATH} O=${FSBL_OUTPUT_PATH} BLCP_2ND_PATH=${BLCP_2ND_PATH} \
        LOADER_2ND_PATH=${UBOOT_PATH}/${UBOOT_OUTPUT_FOLDER}/u-boot-raw.bin
    ${Q}cp ${FSBL_OUTPUT_PATH}/fip.bin ${OUTPUT_DIR}/
    ```

    `opensbi`に依存しています。

    ```Makefile
    opensbi: export CROSS_COMPILE=$(CONFIG_CROSS_COMPILE_SDK)
    opensbi: u-boot-build
    $(call print_target)
    ${Q}$(MAKE) -j${NPROC} -C ${OPENSBI_PATH} PLATFORM=generic \
        FW_PAYLOAD_PATH=${UBOOT_PATH}/${UBOOT_OUTPUT_FOLDER}/u-boot-raw.bin \
        FW_FDT_PATH=${UBOOT_PATH}/${UBOOT_OUTPUT_FOLDER}/arch/riscv/dts/${CHIP}_${BOARD}.dtb
    ```

    `u-boot-build`に依存しています。

    ```Makefile
    u-boot-build: memory-map
    u-boot-build: u-boot-dts
    u-boot-build: ${UBOOT_PATH}/${UBOOT_OUTPUT_FOLDER} ${UBOOT_CVIPART_DEP} ${UBOOT_OUTPUT_CONFIG_PATH}
    $(call print_target)
    ${Q}ln -snrf ${CVI_BOARD_MEMMAP_H_PATH} ${UBOOT_PATH}/include/
    ${Q}rm -f ${UBOOT_CVI_BOARD_INIT_PATH}
    ${Q}ln -s ${BUILD_PATH}/boards/${CHIP_ARCH_L}/${PROJECT_FULLNAME}/u-boot/cvi_board_init.c ${UBOOT_CVI_BOARD_INIT_PATH}
    ${Q}rm -f ${UBOOT_CVITEK_PATH}
    ${Q}ln -s ${BUILD_PATH}/boards/${CHIP_ARCH_L}/${PROJECT_FULLNAME}/u-boot/cvitek.h ${UBOOT_CVITEK_PATH}
    ${Q}$(MAKE) -j${NPROC} -C ${UBOOT_PATH} olddefconfig
    ${Q}$(MAKE) -j${NPROC} -C ${UBOOT_PATH} all
    ${Q}cat ${UBOOT_PATH}/${UBOOT_OUTPUT_FOLDER}/u-boot.bin > ${UBOOT_PATH}/${UBOOT_OUTPUT_FOLDER}/u-boot-raw.bin
    ```

    `u-boot-dts`に依存しています。

    ```Makefile
    u-boot-dts:
    $(call print_target)
    ifeq ($(UBOOT_SRC), u-boot-2021.10)
    # U-boot doesn't has arch/arm64
    ifeq ($(ARCH), arm64)
    ${Q}find ${BUILD_PATH}/boards/${CHIP_ARCH_L} \
        \( -path "*linux/*.dts*" -o -path "*dts_${ARCH}/*.dts*" \) \
        -exec cp {} ${UBOOT_PATH}/arch/arm/dts/ \;
    ${Q}find ${DTS_DEFATUL_PATHS} -name *.dts* -exec cp {} ${UBOOT_PATH}/arch/arm/dts/ \;
    else
    ${Q}find ${BUILD_PATH}/boards/${CHIP_ARCH_L} \
        \( -path "*linux/*.dts*" -o -path "*dts_${ARCH}/*.dts*" \) \
        -exec cp {} ${UBOOT_PATH}/arch/${ARCH}/dts/ \;
    ${Q}find ${DTS_DEFATUL_PATHS} -name *.dts* -exec cp {} ${UBOOT_PATH}/arch/${ARCH}/dts/ \;
    endif
    endif
    ```

    最後に実行されるコマンドは次のようになります。

    ```Makefile
    find /home/share/samba/risc-v/duo-buildroot-sdk/build/boards/cv180x \
    \( -path "*linux/*.dts*" -o -path "*dts_riscv/*.dts*" \) \
    -exec cp {} /home/share/samba/risc-v/duo-buildroot-sdk/u-boot-2021.10/arch/riscv/dts/ \;
    find /home/share/samba/risc-v/duo-buildroot-sdk/build/boards/default/dts/cv180x /home/share/samba/risc-v/duo-buildroot-sdk/build/boards/default/dts/cv180x_riscv -name *.dts* -exec cp {} /home/share/samba/risc-v/duo-buildroot-sdk/u-boot-2021.10/arch/riscv/dts/ \;
    ```

    `home/share/samba/risc-v/duo-buildroot-sdk/u-boot-2021.10/arch/riscv/dts`
    ディレクトリで`tree`コマンドを実行するとディレクトリ内の`cv180x`と`cv1801c`で
    始まるファイルが新しいディレクトリからコピーされています。milk-v duoは
    cv1800bを使っているので`cv1800b_milkv_duo_sd.dts`ファイルを使用することに
    なります。

    ```bash
    $ tree
    .
    |-- Makefile
    |-- ae350-u-boot.dtsi
    |-- ae350_32.dts
    |-- ae350_64.dts
    |-- binman.dtsi
    |-- cv1800b_milkv_duo_sd.dts
    |-- cv1800b_sophpi_duo_sd.dts
    |-- cv1800b_wdmb_0008a_spinor.dts
    |-- cv1800b_wevb_0008a_spinor.dts
    |-- cv1800c_wevb_0009a_spinor.dts
    |-- cv1801b_wevb_0008a_spinor.dts
    |-- cv1801c_wdmb_0009a_spinor.dts
    |-- cv1801c_wevb_0009a_spinand.dts
    |-- cv1801c_wevb_0009a_spinor.dts
    |-- cv180x_asic_bga.dtsi
    |-- cv180x_asic_qfn.dtsi
    |-- cv180x_asic_sd.dtsi
    |-- cv180x_asic_spinand.dtsi
    |-- cv180x_asic_spinor.dtsi
    |-- cv180x_base.dtsi
    |-- cv180x_base_riscv.dtsi
    |-- cv180x_default_memmap.dtsi
    |-- cv180x_fpga.dts
    |-- cv180x_palladium.dts
    |-- cv180zb_wdmb_0008a_spinor.dts
    |-- cv180zb_wevb_0008a_spinor.dts
    |-- fu540-c000-u-boot.dtsi
    |-- fu540-c000.dtsi
    |-- fu540-hifive-unleashed-a00-ddr.dtsi
    |-- fu740-c000-u-boot.dtsi
    |-- fu740-c000.dtsi
    |-- fu740-hifive-unmatched-a00-ddr.dtsi
    |-- hifive-unleashed-a00-u-boot.dtsi
    |-- hifive-unleashed-a00.dts
    |-- hifive-unmatched-a00-u-boot.dtsi
    |-- hifive-unmatched-a00.dts
    |-- k210-maix-bit.dts
    |-- k210.dtsi
    |-- microchip-mpfs-icicle-kit-u-boot.dtsi
    |-- microchip-mpfs-icicle-kit.dts
    |-- openpiton-riscv64.dts
    `-- qemu-virt.dts
    ```

    u-bootのコンパイルはここからが本番で以下のコマンドを実行します。

    ```Makefile
    ${Q}$(MAKE) -j${NPROC} -C ${UBOOT_PATH} olddefconfig
    ${Q}$(MAKE) -j${NPROC} -C ${UBOOT_PATH} all
    ${Q}cat ${UBOOT_PATH}/${UBOOT_OUTPUT_FOLDER}/u-boot.bin > ${UBOOT_PATH}/${UBOOT_OUTPUT_FOLDER}/u-boot-raw.bin
    ```

    最終的に`/home/share/samba/risc-v/duo-buildroot-sdk/u-boot-2021.10/build/cv1800b_milkv_duo_sd/u-boot-raw.bin`が作成されます。

4. opensbiのコンパイル

    u-bootの次はopensbiのコンパイルを開始します。

    OpenSBIは様々なランタイム要件に対応するため、次の3種類のファームウェアを
    サポートしています。

    - dynamic: 前のブートステージから次のブートステージのエントリ情報を取得し、
    `struct fw_dynamic_info`として`a2`レジスタを通して渡します。
    - jump: 次のブートステージエントリが固定アドレスであると仮定して、そこに
    直接ジャンプします。
    - payload: ジャンプする次のブートステージのバイナリを直接パックします。
    バイナリは通常、ブートローダかOS（例：U-Boot、Linux）です。

    関連するビルドスクリプトは` build/scripts/fip_v2.mk`です。

    ```Makefile
    opensbi: export CROSS_COMPILE=$(CONFIG_CROSS_COMPILE_SDK)
    opensbi: u-boot-build
    $(call print_target)
    ${Q}$(MAKE) -j${NPROC} -C ${OPENSBI_PATH} PLATFORM=generic \
        FW_PAYLOAD_PATH=${UBOOT_PATH}/${UBOOT_OUTPUT_FOLDER}/u-boot-raw.bin \
        FW_FDT_PATH=${UBOOT_PATH}/${UBOOT_OUTPUT_FOLDER}/arch/riscv/dts/${CHIP}_${BOARD}.dtb
    ```

    上のスクリプトによるとmilk-v duoではペイロードモードを使用し、ペイロード
    ファイルは`u-boot-raw.bin`であり、これは上でコンパイルしたファイルです。
    `FW_FDT_PATH`はデバイスツリーへのパスで `/home/share/samba/risc-v/duo-buildroot-sdk/u-boot-2021.10/build/cv1800b_milkv_duo_sd/arch/riscv/dts/cv1800b_milkv_duo_sd.dtb` です。

    コンパイル後の出力は`/home/share/samba/risc-v/duo-buildroot-sdk/opensbi/build/platform/generic/firmware`ディレクトリにあります。

5. fsblのビルド

    u-bootとopensbiをコンパイルした後は`fsbl-build`に移ります（この間に`rtos`の
    ビルドがありますが現在はサポートされていないので省略します）。`FSBL`は
    "First Stage Boot Loader"の略です。

    ```Makefile
    fsbl-build: u-boot-build memory-map
    $(call print_target)
    ${Q}mkdir -p ${FSBL_PATH}/build
    ${Q}ln -snrf -t ${FSBL_PATH}/build ${CVI_BOARD_MEMMAP_H_PATH}
    ${Q}$(MAKE) -j${NPROC} -C ${FSBL_PATH} O=${FSBL_OUTPUT_PATH} BLCP_2ND_PATH=${BLCP_2ND_PATH} \
        LOADER_2ND_PATH=${UBOOT_PATH}/${UBOOT_OUTPUT_FOLDER}/u-boot-raw.bin
    ${Q}cp ${FSBL_OUTPUT_PATH}/fip.bin ${OUTPUT_DIR}/
    ```

    実行のためのコンパイルコマンドは以下の通りです。

    ```bash
    make -j8 -C /home/share/samba/risc-v/duo-buildroot-sdk/fsbl O=/home/share/samba/risc-v/duo-buildroot-sdk/fsbl/build/cv1800b_milkv_duo_sd BLCP_2ND_PATH=/home/share/samba/risc-v/duo-buildroot-sdk/freertos/cvitek/install/bin/cvirtos.bin \
    LOADER_2ND_PATH=/home/share/samba/risc-v/duo-buildroot-sdk/u-boot-2021.10/build/cv1800b_milkv_duo_sd/u-boot-raw.bin
    ```

    `fsbl/Makefile`ファイルから始めて`fsbl`のビルドプロセスを見てみましょう。

    ```Makefile
    all: fip bl2 blmacros

    include ${MAKE_HELPERS_DIRECTORY}fip.mk
    ```

    fsbl/make_helpers/fip.mkファイルでfipを順番に探します。

    ```Makefile
    gen-chip-conf:
    $(print_target)
    ${Q}./plat/${CHIP_ARCH}/chip_conf.py ${CHIP_CONF_PATH}

    macro_to_env = ${NM} '${BLMACROS_ELF}' | awk '/DEF_${1}/ { rc = 1; print "${1}=0x" $1 } END { exit !rc }' >> ${BUILD_PLAT}/blmacros.env

    blmacros-env: blmacros
    $(print_target)
    ${Q}> ${BUILD_PLAT}/blmacros.env  # clear .env first
    ${Q}$(call macro_to_env,MONITOR_RUNADDR)
    ${Q}$(call macro_to_env,BLCP_2ND_RUNADDR)

    fip: fip-all

    fip-dep: bl2 blmacros-env gen-chip-conf

    fip-simple: fip-dep
    $(print_target)
    ${Q}echo "  [GEN] fip.bin"
    ${Q}${FIPTOOL} -v genfip \
        '${BUILD_PLAT}/fip.bin' \
        --CHIP_CONF='${CHIP_CONF_PATH}' \
        --NOR_INFO='${NOR_INFO}' \
        --NAND_INFO='${NAND_INFO}'\
        --BL2='${BUILD_PLAT}/bl2.bin'
    ${Q}echo "  [LS] " $(ls -l '${BUILD_PLAT}/fip.bin')

    fip-all: fip-dep
    $(print_target)
    ${Q}echo "  [GEN] fip.bin"
    ${Q}. ${BUILD_PLAT}/blmacros.env && \
    ${FIPTOOL} -v genfip \
        '${BUILD_PLAT}/fip.bin' \
        --MONITOR_RUNADDR="${MONITOR_RUNADDR}" \
        --BLCP_2ND_RUNADDR="${BLCP_2ND_RUNADDR}" \
        --CHIP_CONF='${CHIP_CONF_PATH}' \
        --NOR_INFO='${NOR_INFO}' \
        --NAND_INFO='${NAND_INFO}'\
        --BL2='${BUILD_PLAT}/bl2.bin' \
        --BLCP_IMG_RUNADDR=${BLCP_IMG_RUNADDR} \
        --BLCP_PARAM_LOADADDR=${BLCP_PARAM_LOADADDR} \
        --BLCP=${BLCP_PATH} \
        --DDR_PARAM='${DDR_PARAM_TEST_PATH}' \
        --BLCP_2ND='${BLCP_2ND_PATH}' \
        --MONITOR='${MONITOR_PATH}' \
        --LOADER_2ND='${LOADER_2ND_PATH}' \
        --compress='${FIP_COMPRESS}'
    ${Q}echo "  [LS] " $(ls -l '${BUILD_PLAT}/fip.bin')
    ```

    複雑な操作の数々を経て`bl2.bin`ファイルが生成されます（通常、開発者は
    このようなものを修正することはないです）。

    需要なものは`fip-all`オペレーションで、`fip.bin`ファイルの生成に使われます。

6. fip.binファイルの合成

    限られた公式情報を確認すると`fip.bin`はブートローダとubootを含むファイルで
    あることがわかります。

    > なぜこんなことをするのか、公式ドキュメントにはこう書かれています。
    > ネイティブなu-bootはFLASHに直接書き込むことができないu-boot.binを
    > コンパイルするので、ARM Trusted Firmware DesignのFIP (Firmware Image
    > Package) 方式を採用してuboot.binをFIP.binにカプセル化します。
    > https://doc.sophgo.com/cvitek-develop-docs/master/docs_latest_release/CV180x_CV181x/zh/01.software/OSDRV/U-boot_Porting_Development_Guide/build/html/3_U-boot_Transplant.html

    以下はchatgptにfipについて問い合わせた結果です。

    FIP (Flexible Image Processor) ファイルとはARMアーキテクチャのプロセッサ
    デバイスに保存されるファームウェアイメージファイルです。FIPファイルは通常、
    以下の内容を含んでいます。

    1. Trusted Firmware-A (TF-A): TF-A はARMアーキテクチャデバイス上のオープン
       ソースのトラステッドファームウェアです。 デバイスのブートストラップ、
       セキュリティチェック、ブートローダの実行を担当します。
    2. U-Boot：U-Bootはデバイスのブートに使用されるオープンソースのブート
       ローダです。 デバイスを起動する機能を提供し、デバイスハードウェアを
       設定・管理するオプションを提供します。
    3. ARM Trusted Firmware (ATF): ATFはセキュアブート用のファームウェア
       セットであり、デバイス上の他のソフトウ ェアコンポーネント（OSなど）を
       認証してブートするために使用されます。
    4. その他のコンポーネント: FIPファイルにはデバイスツリーファイル、TEE
       ファームウェア（OP-TEE など）、暗号化キー、構成パラメータなどのその他の
       ファームウェアコンポーネントを含めることもできます。

    FIPファイルはARMアーキテクチャデバイスでは一般的であり、プロセッサを
    起動し、初期化するために必要なコンポーネントを提供します。これらのファイルは
    デバイスメーカから提供され、特定のデバイス、ハードウェア構成、要件に
    合わせてカスタマイズされます。

    前述の解析のように`fip.bin`は`fsbl`の最終段階で合成されます。

    ```Makefile
    fip-all: fip-dep
    $(print_target)
    ${Q}echo "  [GEN] fip.bin"
    ${Q}. ${BUILD_PLAT}/blmacros.env && \
    ${FIPTOOL} -v genfip \
        '${BUILD_PLAT}/fip.bin' \
        --MONITOR_RUNADDR="${MONITOR_RUNADDR}" \
        --BLCP_2ND_RUNADDR="${BLCP_2ND_RUNADDR}" \
        --CHIP_CONF='${CHIP_CONF_PATH}' \
        --NOR_INFO='${NOR_INFO}' \
        --NAND_INFO='${NAND_INFO}'\
        --BL2='${BUILD_PLAT}/bl2.bin' \
        --BLCP_IMG_RUNADDR=${BLCP_IMG_RUNADDR} \
        --BLCP_PARAM_LOADADDR=${BLCP_PARAM_LOADADDR} \
        --BLCP=${BLCP_PATH} \
        --DDR_PARAM='${DDR_PARAM_TEST_PATH}' \
        --BLCP_2ND='${BLCP_2ND_PATH}' \
        --MONITOR='${MONITOR_PATH}' \
        --LOADER_2ND='${LOADER_2ND_PATH}' \
        --compress='${FIP_COMPRESS}'
    ${Q}echo "  [LS] " $(ls -l '${BUILD_PLAT}/fip.bin')
    ```

    実際の実行コマンドは以下のように変換されます。

    ```bash
    . /home/share/samba/risc-v/duo-buildroot-sdk/fsbl/build/cv1800b_milkv_duo_sd/blmacros.env && \
    ./plat/cv180x/fiptool.py -v genfip \
    '/home/share/samba/risc-v/duo-buildroot-sdk/fsbl/build/cv1800b_milkv_duo_sd/fip.bin' \
    --MONITOR_RUNADDR="${MONITOR_RUNADDR}" \
    --BLCP_2ND_RUNADDR="${BLCP_2ND_RUNADDR}" \
    --CHIP_CONF='/home/share/samba/risc-v/duo-buildroot-sdk/fsbl/build/cv1800b_milkv_duo_sd/chip_conf.bin' \
    --NOR_INFO='FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF' \
    --NAND_INFO='00000000'\
    --BL2='/home/share/samba/risc-v/duo-buildroot-sdk/fsbl/build/cv1800b_milkv_duo_sd/bl2.bin' \
    --BLCP_IMG_RUNADDR=0x05200200 \
    --BLCP_PARAM_LOADADDR=0 \
    --BLCP=test/empty.bin \
    --DDR_PARAM='test/cv181x/ddr_param.bin' \
    --BLCP_2ND='/home/share/samba/risc-v/duo-buildroot-sdk/freertos/cvitek/install/bin/cvirtos.bin' \
    --MONITOR='../opensbi/build/platform/generic/firmware/fw_dynamic.bin' \
    --LOADER_2ND='/home/share/samba/risc-v/duo-buildroot-sdk/u-boot-2021.10/build/cv1800b_milkv_duo_sd/u-boot-raw.bin' \
    --compress='lzma'
    ```

最後に、スタートアップのプロセスは次のようになります。

![boot process](boot_process.png)
