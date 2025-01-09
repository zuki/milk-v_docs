# 第2章 システム機能

- ポータブルおよび据え置き型アプリケーションを対象
- メモリ容量
    1. SDSC (Stabdard Capacity SD Memory Card): 2GBまで
    2. SDHC (High Capacity SD Memory Card): 2GB以上、32GBまで
    3. SDXC (Extended Capacity SD Memory Card): 32GB以上、2TBまで
    4. SDUC (Ultra Capacity SD Memory Card): 2TB以上、128TBまで
- 電圧範囲
    - High Voltage SD Memory Card: 動作電圧範囲: 2.7 - 3.6V
    - UHS-II SD Memory Card: 動作電圧範囲: VDD1: 2.7 - 3.6V, VDD2: 1.70 - 1.95V 
    - SD Express Memory Card: 動作電圧範囲: VDD1: 2.7 - 3.6V, VDD2: 1.70 - 1.95V, 任意でVDD3: 1.14 - 1.30V （サポートされている場合、VDD2の代わりに動作）
- 読み込み専用カードと読み書きカード用に設計
- バススピードモード: 4本のパラレルデータラインを使用
    1. デフォルトスピードモード (DS): 3.3V信号、周波数は最大25MHz, 12.5MB/sec
    2. 高速スピードモード(HS): 3.3V信号、周波数は最大50MHz, 25MB/sec
    3. SDR12: UHS-1 1.8V信号, 周波数は最大25MHz, 12.5MB/sec
    4. SDR25: UHS-I 1.8V信号、周波数は最大50MHz, 25MB/sec
    5. SDR50: UHS-I 1.8V信号、周波数は最大100MHz, 50MB/sec
    6. SDR104: UHS-I 1.8V信号、周波数は最大208MHz, 104MB/sec
    7. DDR50: UHS-I 1.8V信号、周波数は最大50MHz, クロックの両エッジで
       サンプルすることで最大50MB/sec
- バススピードモード: UHS-II差分インタフェースラインを使用
    1. UHS156: UHS-II RCLK 周波数範囲は 26MHz - 52MHz, ラインあたり最大1.56Gbps
    2. UHS626: UHS-II RCLK 周波数範囲は 26MHz - 52MHz, ラインあたり最大1.6.24Gbps
- バススピードモード: PCIe差分インタフェースラインを使用
    1. PCIe Gen 3.1 ライン, 最大985MB/s
    2. PCIe Gen 3.2 ライン, 最大1,969MB/s
    3. PCIe Gen 4.1 ライン, 最大1,969MB/s
    4. PCIe Gen 4.2 ライン, 最大3,938MB/s
- スイッチ機能コマンドはバス速度モード、コマンドシステム、ドライブ強度、
  将来の機能をサポート
- メモリフィールドエラーを補正
- 読み込み処理中のカード取り出しはコンテンツを壊さない
- コンテンツ保護機構: SDMI規格の最高級セキュリティでコンパイル
- カードのパスワード保護（CMD42: LOCK_UNLOCK）
- メカニカルスイッチを使用した書き込み保護機能
- 組み込みの書き込み保護機能 (永久書き込み保護、一時的な書き込み保護、電源サイクルまでの書き込み保護)
- カード検出 (脱着)
- アプリケーション固有のコマンド
- 快適な消去メカニズム
- SD Express カードの特徴:
    - PCI Express ® (PCle) Gen 3 または Gen 4: 1 レーンまたは 2 レーン
        - デュアルシンプレックス ポイントツーポイント接続
        - Gen3x1 バスの場合、8Gbps 伝送の 2つの差動 I/O (1 RX / 1 TX)、各方向で最大 985MB/s を
          生成（128/130 エンコーディングによる ~1.5% のオーバーヘッド）
        - Gen3x2 バスの場合、8Gbps 伝送の4つの差動 I/O (2 RX / 2 TX)、各方向で最大1,969MB/sを
          生成（128/130エンコーディングによる~1.5%のオーバーヘッド)
        - Gen4x1バスの場合、16Gbps転送の2つの差動 I/O (1 RX / 1 TX)、各方向で最大1,969MB/sを
          生成（128/130エンコーディングによる~1.5%のオーバーヘッド）
        - Gen4x2バスの場合、16Gbps転送の4つの差分 I/O (2 RX / 2 TX)、各方向で最大3,938MB/sを
          生成（128/130エンコーディングによる約1.5%のオーバーヘッド）
        - ホットプラグイン/アウトのサポート
        - 標準大容量コントローラとして識別: NVM Express ™デバイス
    - NVM-Expressリビジョン1.3以降でPCle インターフェイスをサポート
        - リビジョン 1.3 のサポートは必須、リビジョン 1.3a, 1.3b, 1.30, 1.3d, 1.4 の機能のサポートは
          オプション
        - NVMeはパフォーマンス用に構築された軽量プロトコル
        - PCleおよびNVMe標準のすべてのオプション標準機能は必要に応じてホストまたはカードで
          実装可能（実装まで）。以下はその一部
            - NVMeでサポートされているホストメモリバッファ
            - マルチキューとロック機構のないNVMe
    - レガシーSD UHS-Iインタフェースを下位互換のためにサポート（セクション8.1.4、8.1.1.5と9で
      説明されているように機能は制限される）
- 基本的なSD通信チャネルのプロトコル属性

| SDメモリカード通信チャネル |
|:---------------------------|
| 6線式通信チャネル（クロック、コマンド、4つのデータライン） |
| エラー保護データ転送 |
| シングルまたはマルチブロック指向データ転送 |
    
- SD ExpressカードはPCleインターフェイスを介したNVMeプロトコル（リビジョン1.3以降）をサポート
- SDメモリカードフォームファクター
  パート1の機械的補遺は次のとおり
    - 標準サイズSDメモリカード: 「パート1 標準サイズSDカード補遺」で規定
    - miniSDメモリカード:「パート1 miniSDカード補遺」で規定
    - microSDメモリカード: 「パート1 microSDカード補遺」で規定
- 標準サイズSDメモリカードの厚さは2.1 mm（標準）と1.4 mm（薄型SDメモリカード）として定義
- ブートパーティションとファストブートを含むブート機能
- MBRシャドウイングを含むTCG（Trusted Computing Group）セキュリティ
- RPMB (Replay Protected Memory Block)

## 注

1. NVM Express™（ワードマーク）とNVMeT™（ワードマーク）は、NVM Express, Inc.の商標です。
    NVM Express, Inc.はNVM Expressの標準と仕様を定義しています。詳細についてはNVM Express,
    Inc.にお問い合わせください。URL: [http://nvmexpress.org/](http://nvmexpress.org/).
2. PCI-SIG®、PCle®、PCI Express®は、PCI-SIGの登録商標です。PCI-SIGはPCI Expressの標準と仕様を
    定義しています。詳細についてはPCI-SIGにお問い合わせください。URL: [https://pcisig.com/](https://pcisig.com/)
    PCI-SIG®、PCle®、PCI Express®は、PCI-SIGの登録商標です。
3. SD Express 8.0ホストは初期リビジョンを使用しているSD Expressカードとの互換性の問題を
    回避するために、NVM Expressリビジョン1.3dまたは1.4をサポートすることを勧めています。
4. TCGはTCGセキュリティ標準と仕様を定義しています。詳細についてはTCGにお問い合わせください。
    URL: [https://www.trustedcomputinggroup.org/](https://www.trustedcomputinggroup.org/)。