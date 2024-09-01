# Genericプラットフォーム

[オリジナル](https://github.com/riscv-software-src/opensbi/blob/master/docs/platform/generic.md)

**Generic**プラットフォームはFDT (Flattened device tree) ベースのプラットフォームであり、
プラットフォーム固有のすべての機能は直前のブートステージから渡されるFDTに基づいて
提供されます。Genericプラットフォームは様々なエミュレータ、シミュレータ、FPGA、ボード上で
同じOpenSBIファームウェアバイナリを使用することを可能にします。

デフォルトではGeneric FDTプラットフォームは次を仮定しています。

1. プラットフォームの機能はデフォルト
2. プラットフォームのスタックサイズはデフォルト
3. プラットフォームには癖や回避策はない

上記の仮定（1を除く）はFDTのrootノード互換文字列に基づいて呼び出される特別な
プラットフォームコールバックを追加することでオーバーライドすることができます。

Generic FDTプラットフォームのユーザは以下を確認する必要があります。

1. `lib/utils`ディレクトリにあるさまざまなFDTベースのドライバはそのプラットフォーム要件に
   基づいて最新であること
2. 直前のブートステージから渡されるFDTは`lib/utils`ディレクトリ配下のFDTベースのドライバと
   同期したDT互換文字列とDTプロパティを持つこと
3. プラットフォームに複数のシリアルポートまたはコンソールがある場合、FDTは"/chosen" DT
   ノードに"stdout-path" DTプロパティを持つこと
4. マルチHARTプラットフォームでは、FDTはIPIデバイス用のDTノードを持ち、`lib/utils/ipi`
   ディレクトリに対応するFDTベースのIPIドライバを持つこと
5. FDTはタイマーデバイス用のDTノードを持ち、`lib/utils/timer`ディレクトリに対応する
   FDTベースのタイマードライバを持つこと

プラットフォーム固有のライブラリとファームウェアイメージをビルドするには、トップレベルの
`make`コマンドで _PLATFORM=generic_ パラメータを指定してください。

## プラットフォームオプション

_Generic_ プラットフォームにはプラットフォーム固有のオプションはありません。

## Genericプラットフォームを使用しているRISC-Vプラットフォーム

- Andes AE350 Platform (andes-ae350.md)
- QEMU RISC-V Virt Machine (qemu_virt.md)
- Renesas RZ/Five SoC (renesas-rzfive.md)
- Shakti C-class SoC Platform (shakti_cclass.md)
- SiFive HiFive Unleashed (sifive_fu540.md)
- Spike (spike.md)
- T-HEAD C9xx series Processors (thead-c9xx.md)

# T-HEAD C9xxシリーズプロセッサ

C9xxシリーズプロセッサはAIベクトルアクセラレーションエンジンを搭載した高性能の
RISC-Vアーキテクチャマルチコアプロセッサです。

詳細については[T-HEAD.CN](https://www.t-head.cn/)を参照してください。

プラットフォーム固有のライブラリとファームウェアイメージをビルドするには、トップレベルの
`make`コマンドで _PLATFORM=generic_ パラメータを指定してください。

## プラットフォームオプション

T-HEAD C9xxはgenericプラットフォームを使用するためプラットフォーム固有のコンパイル
オプションはありません。

```bash
CROSS_COMPILE=riscv64-linux-gnu- PLATFORM=generic make
```

以下はfpgaプロトタイプの最もシンプルなブートフローです。

```
(Jtag gdbinit) -> (zsb) -> (opensbi) -> (linux)
```

詳細については[ゼロステージブート](https://github.com/c-sky/zero_stage_boot)を参照して
ください。

# `c-sky/zero_stage_boot`の`README`

## はじめに

ゼロステージブートはopensbiの前に来るXuanTieプロセッサのinitコードです。zero_stage_bootの
前にSoCベンダはddr_initとCPUのリセット手続きを準備する必要があります。すべてのハートは
同時にzero_stage_bootに入りますが、最初のハートがGOTとオフセット変数の再配置を行い、
他のハートは待機します。各ハートはCPUIDバージョンにより各自のCSRを初期化することにより、
複数のハートが同時に動作することを可能にしています（たとえば、4*c908 + 2*c910）。
オープンソースのリポジトリから標準的なopensbiとLinuxカーネルのバイナリをコンパイルする
ことができます。すべてXuanTieプロセッサと互換性があります。以下は簡単なブートフローです。

```bash
[Jtag gdbinit] -> [zero_stage_boot] -> [opensbi] -> [Linux]

opensbi: https://github.com/riscv-software-src/opensbi
Linux:   https://kernel.org/
```

zero_stage_bootのコンパイルは非常に簡単であり、必要なものは標準的なRISC-V GCCコンパイラ
だけです。

```bash
CROSS_COMPILE=riscv64-unknown-linux-gnu- make
```

ただし、FPGAのブートアップにはリリース版のバイナリを使用することを強く勧めます。
この画面の右側にあるReleasesボタンをクリックすると、zsb、OpenSBI、Image (Linux) を
含むコンパイル済みバイナリが入手できます。これらのバイナリファイルはリリース前に
包括的なテストを受けており、詳細で正確なバージョン情報が含まれています。

- **64lp64**は、64ビットハードウェア上でlp64 ABIを実行することを意味します
- **32ilp32**は、32ビットハードウェア上でlp32 ABIを実行することを意味します
- **64ilp32**は、64ビットハードウェア上でilp32 ABIを実行することを意味します
- **zsb** は、zero_stage_bootを意味します。
- **Linux-5.10 + opensbi-0.9**は、early customers用です。
- **Linux-6.6 + opensbi-1.3**は、カレントです。
- **zsb-64lp64-xt**はカスタムコンパイラでリコンパイルしたものです。機能的には
  zsb-64lp64と同じです。


rv64プロセッサの場合、zsb-64lp64.tar.gz + opensbi-1.3-64lp64.tar.gz + linux-6.6-64lp64.tar.gz をダウンロードして、DTS + gdbinitを各自で用意してください。

これでJtagを使ってFPGA Platformを動かすことができます。

## linux-6.6 opensbi-1.3 DTSの例

OpenSBIのgenericファームウェアに提供されるT-HEAD C9xx DTBは、通常、"thead,c900-clint",
"thead,c900-plic"という互換性文字列を持ちます。

```dts
/dts-v1/;
/ {
	model = "Test Sample";
	compatible = "test,sample";
	#address-cells = <2>;
	#size-cells = <2>;

	memory@60000000 {
		device_type = "memory";
		reg = <0x0 0x60000000 0x0 0x40000000>;
	};

	cpus {
		#address-cells = <1>;
		#size-cells = <0>;
		timebase-frequency = <25000000>;
		cpu@0 {
			device_type = "cpu";
			reg = <0>;
			status = "okay";
			compatible = "riscv";
			riscv,isa = "rv64imafdc_zicbom_svpbmt_sstc_sscofpmf";
			riscv,cbom-block-size = <64>;
			mmu-type = "riscv,sv57";
			cpu0_intc: interrupt-controller {
				#address-cells = <0>;
				#interrupt-cells = <1>;
				compatible = "riscv,cpu-intc";
				interrupt-controller;
			};
		};
		cpu@1 {
			device_type = "cpu";
			reg = <1>;
			status = "okay";
			compatible = "riscv";
			riscv,isa = "rv64imafdc_zicbom_svpbmt_sstc_sscofpmf";
			riscv,cbom-block-size = <64>;
			mmu-type = "riscv,sv57";
			cpu1_intc: interrupt-controller {
				#address-cells = <0>;
				#interrupt-cells = <1>;
				compatible = "riscv,cpu-intc";
				interrupt-controller;
			};
		};
		cpu@2 {
			device_type = "cpu";
			reg = <2>;
			status = "okay";
			compatible = "riscv";
			riscv,isa = "rv64imafdc_zicbom_svpbmt_sstc_sscofpmf";
			riscv,cbom-block-size = <64>;
			mmu-type = "riscv,sv57";
			cpu2_intc: interrupt-controller {
				#address-cells = <0>;
				#interrupt-cells = <1>;
				compatible = "riscv,cpu-intc";
				interrupt-controller;
			};
		};
		cpu@3 {
			device_type = "cpu";
			reg = <3>;
			status = "okay";
			compatible = "riscv";
			riscv,isa = "rv64imafdc_zicbom_svpbmt_sstc_sscofpmf";
			riscv,cbom-block-size = <64>;
			mmu-type = "riscv,sv57";
			cpu3_intc: interrupt-controller {
				#address-cells = <0>;
				#interrupt-cells = <1>;
				compatible = "riscv,cpu-intc";
				interrupt-controller;
			};
		};
	};


	soc {
		#address-cells = <2>;
		#size-cells = <2>;
		compatible = "simple-bus";
		dma-noncoherent;
		ranges;

		clint0: clint@c000000 {
			compatible = "thead,c900-clint";
			interrupts-extended = <
				&cpu0_intc  3 &cpu0_intc  7
				&cpu1_intc  3 &cpu1_intc  7
				&cpu2_intc  3 &cpu2_intc  7
				&cpu3_intc  3 &cpu3_intc  7
				>;
			reg = <0x0 0x0c000000 0x0 0x04000000>;
			clint,has-no-64bit-mmio;
		};

		intc: interrupt-controller@8000000 {
			#address-cells = <0>;
			#interrupt-cells = <2>;
			compatible = "thead,c900-plic";
			reg = <0x0 0x08000000 0x0 0x04000000>;
			riscv,ndev = <64>;
			interrupt-controller;
			interrupts-extended = <
				&cpu0_intc  0xffffffff &cpu0_intc  9
				&cpu1_intc  0xffffffff &cpu1_intc  9
				&cpu2_intc  0xffffffff &cpu2_intc  9
				&cpu3_intc  0xffffffff &cpu3_intc  9
				>;
		};
	};
};
```

## linux-5.10 opensbi-0.9 DTSの例

OpenSBIのgenericファームウェアに提供されるT-HEAD C9xx DTBは、通常、"riscv,clint0",
"riscv,plic0"という互換性文字列を持ちます。

```dts
/dts-v1/;
/ {
	model = "Test Sample";
	compatible = "test,sample";
	#address-cells = <2>;
	#size-cells = <2>;

	memory@60000000 {
		device_type = "memory";
		reg = <0x0 0x60000000 0x0 0x40000000>;
	};

	cpus {
		#address-cells = <1>;
		#size-cells = <0>;
		timebase-frequency = <25000000>;
		cpu@0 {
			device_type = "cpu";
			reg = <0>;
			status = "okay";
			compatible = "riscv";
			riscv,isa = "rv64ima";
			mmu-type = "riscv,sv39";
			cpu0_intc: interrupt-controller {
				#address-cells = <0>;
				#interrupt-cells = <1>;
				compatible = "riscv,cpu-intc";
				interrupt-controller;
			};
		};
		cpu@1 {
			device_type = "cpu";
			reg = <1>;
			status = "okay";
			compatible = "riscv";
			riscv,isa = "rv64ima";
			mmu-type = "riscv,sv39";
			cpu1_intc: interrupt-controller {
				#address-cells = <0>;
				#interrupt-cells = <1>;
				compatible = "riscv,cpu-intc";
				interrupt-controller;
			};
		};
		cpu@2 {
			device_type = "cpu";
			reg = <2>;
			status = "okay";
			compatible = "riscv";
			riscv,isa = "rv64ima";
			mmu-type = "riscv,sv39";
			cpu2_intc: interrupt-controller {
				#address-cells = <0>;
				#interrupt-cells = <1>;
				compatible = "riscv,cpu-intc";
				interrupt-controller;
			};
		};
		cpu@3 {
			device_type = "cpu";
			reg = <3>;
			status = "okay";
			compatible = "riscv";
			riscv,isa = "rv64ima";
			mmu-type = "riscv,sv39";
			cpu3_intc: interrupt-controller {
				#address-cells = <0>;
				#interrupt-cells = <1>;
				compatible = "riscv,cpu-intc";
				interrupt-controller;
			};
		};
	};


	soc {
		#address-cells = <2>;
		#size-cells = <2>;
		compatible = "simple-bus";
		ranges;

		clint0: clint@c000000 {
			compatible = "riscv,clint0";
			interrupts-extended = <
				&cpu0_intc  3 &cpu0_intc  7
				&cpu1_intc  3 &cpu1_intc  7
				&cpu2_intc  3 &cpu2_intc  7
				&cpu3_intc  3 &cpu3_intc  7
				>;
			reg = <0x0 0x0c000000 0x0 0x04000000>;
			clint,has-no-64bit-mmio;
		};

		intc: interrupt-controller@8000000 {
			#address-cells = <0>;
			#interrupt-cells = <1>;
			compatible = "riscv,plic0";
			reg = <0x0 0x08000000 0x0 0x04000000>;
			riscv,ndev = <64>;
			interrupt-controller;
			interrupts-extended = <
				&cpu0_intc  0xffffffff &cpu0_intc  9
				&cpu1_intc  0xffffffff &cpu1_intc  9
				&cpu2_intc  0xffffffff &cpu2_intc  9
				&cpu3_intc  0xffffffff &cpu3_intc  9
				>;
		};
	};
};
```

## CPU gdbinitスクリプト

```bash
# Set gdb environment
set confirm off
set height  0
monitor set resume-bkpt-exception on

# memory layout
set $opensbi_addr = 0x60000000
set $vmlinux_addr = $opensbi_addr + 0x00400000
set $rootfs_addr  = $opensbi_addr + 0x04000000
set $dtb_addr     = $rootfs_addr  - 0x00100000
set $zsb_addr     = $rootfs_addr  - 0x00008000
set $dyninfo_addr = $rootfs_addr  - 0x40
set $flag_addr    = $rootfs_addr  - 0x100

# Load kernel
restore zero_stage_boot.bin binary          $zsb_addr
restore <preceding dts example>.dtb binary  $dtb_addr
restore fw_dynamic.bin binary               $opensbi_addr
restore Image binary                        $vmlinux_addr

# Set opensbi dynamic info param
set *(unsigned long *)($dyninfo_addr)      = 0x4942534f
set *(unsigned long *)($dyninfo_addr + 8)  = 2
set *(unsigned long *)($dyninfo_addr + 16) = $vmlinux_addr
set *(unsigned long *)($dyninfo_addr + 24) = 1
set *(unsigned long *)($dyninfo_addr + 32) = 0
set *(unsigned long *)($dyninfo_addr + 40) = -1

# Set boot flag for CPU functional setting
# This flag.BIT[0] makes zsb enable RV64XT32 by setting mxstatus.[63]=1
# set *(unsigned int *)$flag_addr = 0x1
set *(unsigned int *)$flag_addr = 0x0

# PLIC delegate (Only opensbi-0.9 & Linux-5.10 need it)
set *0x081ffffc=1

# Set all harts reset address
set *0x18030010 = $zsb_addr
set *0x18030018 = $zsb_addr
set *0x18030020 = $zsb_addr
set *0x18030028 = $zsb_addr
set *0x18030030 = $zsb_addr
set $pc         = $zsb_addr

# Release all harts from reset
set *0x18030000 = 0x7f
```

## 実行

Jtagサーバをスタートします。

```bash
DebugServerConsole -prereset
```

次に、gdbを使ってJtagサーバに接続します。

```bash
riscv64-elf-gdb -ex "tar remote <Jtag Server ip:port>" -x <your soc gdbinit> -x <preceding cpu gdbinit> -ex "c"
```

gdb shellに入るには`ctrl+c`を使います。

```bash
file vmlinux
source gdbmarcos.txt
dmesg
```

gdbmacros.txt:

https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/Documentation/admin-guide/kdump/gdbmacros.txt

vmlinux: LinuxカーネルELFファイル

## 付録A: DTSにおけるPMU

PMUの構成は[OpenSBI SBI PMU extension](https://github.com/riscv-software-src/opensbi/blob/master/docs/pmu_support.md)を
参照してください。

以下はXuantie CシリーズCPU用のPMUの構成例です。実際に使用する際にはデータシートに従って
変更する必要があるかもしれません。

```dts
pmu {
	compatible = "riscv,pmu";
	riscv,event-to-mhpmevent =
		/* PMU_HW_CACHE_REFERENCES -> ll_cache_read_access */
		<0x00003 0x00000000 0x00000010>,
		/* PMU_HW_CACHE_MISSES -> ll_cache_read_miss */
		<0x00004 0x00000000 0x00000011>,
		/* PMU_HW_BRANCH_INSTRUCTIONS -> inst_branch */
		<0x00005 0x00000000 0x00000007>,
		/* PMU_HW_BRANCH_MISSES -> inst_branch_mispredict */
		<0x00006 0x00000000 0x00000006>,
		/* PMU_HW_STALLED_CYCLES_FRONTEND -> ifu_stalled_cycle */
		<0x00008 0x00000000 0x00000027>,
		/* PMU_HW_STALLED_CYCLES_BACKEND -> idu_stalled_cycle */
		<0x00009 0x00000000 0x00000028>,
		/* L1D_READ_ACCESS -> l1_dcache_read_access */
		<0x10000 0x00000000 0x0000000c>,
		/* L1D_READ_MISS -> l1_dcache_read_miss */
		<0x10001 0x00000000 0x0000000d>,
		/* L1D_WRITE_ACCESS -> l1_dcache_write_access */
		<0x10002 0x00000000 0x0000000e>,
		/* L1D_WRITE_MISS -> l1_dcache_write_miss */
		<0x10003 0x00000000 0x0000000f>,
		/* L1I_READ_ACCESS -> l1_icache_access */
		<0x10008 0x00000000 0x00000001>,
		/* L1I_READ_MISS -> l1_icache_miss */
		<0x10009 0x00000000 0x00000002>,
		/* LL_READ_ACCESS -> ll_cache_read_access */
		<0x10010 0x00000000 0x00000010>,
		/* LL_READ_MISS -> ll_cache_read_miss */
		<0x10011 0x00000000 0x00000011>,
		/* LL_WRITE_ACCESS -> ll_cache_write_access */
		<0x10012 0x00000000 0x00000012>,
		/* LL_WRITE_MISS -> ll_cache_write_miss */
		<0x10013 0x00000000 0x00000013>,
		/* DTLB_READ_MISS -> dtlb_miss */
		<0x10019 0x00000000 0x00000004>,
		/* ITLB_READ_MISS -> itlb_miss */
		<0x10021 0x00000000 0x00000003>,
		/* BPU_READ_ACCESS -> branch_direction_prediction */
		<0x10030 0x00000000 0x0000001c>,
		/* BPU_READ_MISS -> branch_direction_misprediction */
		<0x10031 0x00000000 0x0000001b>;
	riscv,event-to-mhpmcounters =
		<0x00003 0x00003 0xfffffff8>,
		<0x00004 0x00004 0xfffffff8>,
		<0x00005 0x00005 0xfffffff8>,
		<0x00006 0x00006 0xfffffff8>,
		<0x00007 0x00007 0xfffffff8>,
		<0x00008 0x00008 0xfffffff8>,
		<0x00009 0x00009 0xfffffff8>,
		<0x0000a 0x0000a 0xfffffff8>,
		<0x10000 0x10000 0xfffffff8>,
		<0x10001 0x10001 0xfffffff8>,
		<0x10002 0x10002 0xfffffff8>,
		<0x10003 0x10003 0xfffffff8>,
		<0x10008 0x10008 0xfffffff8>,
		<0x10009 0x10009 0xfffffff8>,
		<0x10010 0x10010 0xfffffff8>,
		<0x10011 0x10011 0xfffffff8>,
		<0x10012 0x10012 0xfffffff8>,
		<0x10013 0x10013 0xfffffff8>,
		<0x10019 0x10019 0xfffffff8>,
		<0x10021 0x10021 0xfffffff8>,
		<0x10030 0x10030 0xfffffff8>,
		<0x10031 0x10031 0xfffffff8>;
	riscv,raw-event-to-mhpmcounters =
		<0x00000000 0x00000001 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000002 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000003 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000004 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000005 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000006 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000007 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000008 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000009 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x0000000a 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x0000000b 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x0000000c 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x0000000d 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x0000000e 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x0000000f 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000010 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000011 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000012 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000013 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000014 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000015 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000016 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000017 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000018 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000019 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x0000001a 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x0000001b 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x0000001c 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x0000001d 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x0000001e 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x0000001f 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000020 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000021 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000022 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000023 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000024 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000025 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000026 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000027 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000028 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x00000029 0xffffffff 0xffffffff 0xfffffff8>,
		<0x00000000 0x0000002a 0xffffffff 0xffffffff 0xfffffff8>;
};
```

たとえば、`perf stat`と`perf record`を使うと

```bash
# perf stat ls

 Performance counter stats for 'ls':

             74.05 msec task-clock                       #    0.747 CPUs utilized
                 0      context-switches                 #    0.000 /sec
                 0      cpu-migrations                   #    0.000 /sec
                58      page-faults                      #  783.256 /sec
           3689065      cycles                           #    0.050 GHz
           1336494      instructions                     #    0.36  insn per cycle
            162119      branches                         #    2.189 M/sec
             28716      branch-misses                    #   17.71% of all branches

       0.099143960 seconds time elapsed

       0.016153000 seconds user
       0.092880000 seconds sys
```

```bash
# echo 1000 > /proc/sys/kernel/perf_event_max_sample_rate
# perf record -g ls
perf.data
[ perf record: Woken up 1 times to write data ]
[ perf record: Captured and wrote 0.006 MB perf.data (9 samples) ]
```

## 付録B: perfのコンパイル法

buildrootを使ってperfツールを持つrootfsをコンパイルすることができます。

```bash
# git clone https://github.com/buildroot/buildroot.git
# cd buildroot/
# make qemu_riscv64_virt_defconfig
# make menuconfig
```

次のPACKAGE configをmenuconfigで有効にしてください。

```bash
BR2_PACKAGE_LINUX_TOOLS=y
BR2_PACKAGE_LINUX_TOOLS_PERF=y
BR2_PACKAGE_ELFUTILS=y
```

## 付録C: 追加のDTS

追加のDTSの例（シリアルとinitrdを持つbootargs）

```dts
serial@1900d000 {
	compatible = "snps,dw-apb-uart";
	reg = <0x0 0x1900d000 0x0 0x400>;
	interrupt-parent = <&intc>;
	interrupts = <20 4>;
	clock-frequency = <36000000>;
	clock-names = "baudclk";
	reg-shift = <2>;
	reg-io-width = <4>;
};

chosen {
	bootargs = "console=ttyS0,115200 norandmaps loglevel=7";
	linux,initrd-start = <0x0 0x64000000>;
	linux,initrd-end = <0x0 0x66000000>;
	stdout-path = "/soc/serial@1900d000:115200";
};
```

'serial'は'reg', 'interrupts', 'clock-frequency'の実際の構成に基づいて構成する
必要があり、'chosen'は'linux,initrd-start', 'linux,initrd-end'の構成に基づいて構成する
必要があります。
