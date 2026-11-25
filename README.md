<img src="images/Monaca_store.png" align="right" width="130" alt="Monaca Mascot" />

# TermAddIn_Melco - 三菱電機 MELSEC 計算機リンク通信エクステンション

👉 [← TermAddIn 統合本体に戻る](https://github.com/Monaca-App/TermAddIn)  
👉 [← Monaca アプリ工房（トップ）に戻る](https://github.com/Monaca-App/store)

---

[![Platform](https://img.shields.io/badge/Platform-Windows%2010%20%2F%2011-blue.svg)](https://www.microsoft.com/)
[![Excel](https://img.shields.io/badge/Excel-2016%20%2F%202019%20%2F%202021%20%2F%20M365-green.svg)](https://www.microsoft.com/excel)
[![Framework](https://img.shields.io/badge/.NET%20Framework-4.8-purple.svg)](https://dotnet.microsoft.com/)
[![Architecture](https://img.shields.io/badge/Architecture-TermAddIn%20Extension-orange.svg)]()
[![License](https://img.shields.io/badge/License-TermAddIn__Melco%20License-blue.svg)]()

## 📖 概要

**TermAddIn_Melco** は、Excel 総合通信アドイン **TermAddIn** 上で動作する ** 三菱電機 MELSEC 計算機リンク通信専用エクステンション（機能ドメイン単一 DLL）** です。

三菱電機製シーケンサ（MELSEC-Q / L / iQ-R / A / FX シリーズ）とのシリアル通信（RS-232C / RS-422 / RS-485）およびイーサネット計算機リンク通信を Excel ワークシート上から完全自動化します。

VBA マクロを一切記述することなく、 **標準の `.xlsx`（マクロ無効ブック）形式のままで MELSEC 内部デバイス（Dレジスタ、X入力、Y出力、Mリレー等）のビット・ワード単位の一括読出・書込、サムチェック自動計算・照合** が安全に実行できます。

> [!NOTE]
> **利用前提とライセンス体系**  
> 本エクステンションは [TermAddIn 本体](https://github.com/Monaca-App/TermAddIn) 上で動作します。  
> TermAddIn ではプロトコル毎に個別ライセンスを設ける体系を採用しており、本機能のご利用には **`TermAddIn_Melco` 個別ライセンス（一括 ¥32,000 / 月額 ¥3,000 / 週額 ¥1,000・税別）** が適用されます（ライセンス未配置・無効時は、Excel アドイン（`TermAddIn`）・`Term.exe` 単体起動ともに 10 分間の期間限定版として動作します）。

---

## ✨ 主な機能・特徴

1. **MELSEC 計算機リンク全プロトコル（形式 1 〜 形式 5）完全網羅**
   * Checksum 付き標準対話形式の「プロトコル 1」、Checksum なし対話形式の「プロトコル 2」、CR/LF 終端・Checksum なしの「プロトコル 3」、CR/LF 終端＋Checksum 付きの「プロトコル 4」、および高速生バイナリ通信の「プロトコル 5」の全 5 形式をサポート。
2. **フレームヘッダ・サムチェック・DLEフレーミング・ACK/NAK の完全自動化**
   * 要求電文送信時の局番・PC 番号・コマンド・サムチェック・DLE エスケープ等のフレーム生成、および受信応答時のサムチェック照合・ACK 返信をエンジンが内部で自動実行。
3. **ビット・ワードデバイスの自在な読み書き**
   * D（データレジスタ）、W（リンクレジスタ）、R（ファイルレジスタ）等のワード読出・書込、および X（入力）、Y（出力）、M（内部リレー）、F、V 等のビット単位制御に完全対応。
4. **取得データの 16進 / 10進 変換とセル自動展開**
   * シーケンサから返却されたデバイス値は、指定した結果セルへ 16 進または 10 進の数値データとしてリアルタイムに自動展開。
5. **Excel 32bit / 64bit 完全動作マトリクス**
   * .NET Framework 標準マネージド通信基盤により構築されており、外部の専用ネイティブ DLL に依存しないため、Excel 32bit 版および 64bit 版の双方で制限なく完全動作。
6. **実機 PLC・ドライバ不要の仮想スレーブ検証 ＆ 電文モニター連携（`TermEmulator` / `c2c` 対応）**
   * **[TermEmulator](https://github.com/Monaca-App/TermEmulator)** の MELSEC 計算機リンク仮想 PLC スレーブ（形式 1〜5・共有メモリ対応、本ライセンス共用可）および公式無償ツール **[c2c](https://github.com/Monaca-App/c2c)** の仮想 COM ペアと連携し、実機なしで全形式フレームの事前デバッグと HEX / ASCII 電文監視を行えます。

---

## 🔌 提供コマンド一覧（全 23 コマンド）

各コマンドの詳細な構文・パラメータ仕様・記述例は、以下の個別リファレンスをご参照ください。

### 高水準 PLC デバイスアクセス・診断コマンド（5 コマンド） ⭐ 推奨

電文プロトコルフレーム（ヘッダ・フッタ・サムチェック）を意識することなく、デバイス名（`D100` や `M0` など）を指定して直接アクセスできます。通信プロトコル形式は [Melsec_Mode](./Manual/Melco/Melsec_Mode.Md) で指定した ASCII 形式（`P1` 〜 `P4`）に自動追従します（形式 5 バイナリ通信は下表の `Melsec_P5s_S` / `Melsec_P5s_R` を使用します）。

| コマンド名 | 構文概要 | 主な役割・用途 | 詳細マニュアル |
| :--- | :--- | :--- | :---: |
| **`Melsec_ReadWord`** | `Melsec_ReadWord, "<デバイス>", "<点数>", "<格納先>", "[DEC/HEX]", "[局番]"` | ワードデバイス（D, W, R 等）の一括読み出し | [詳細説明](./Manual/Melco/Melsec_ReadWord.Md) |
| **`Melsec_WriteWord`** | `Melsec_WriteWord, "<デバイス>", "<書込値>", "[DEC/HEX]", "[局番]"` | ワードデバイス（D, W, R 等）の一括書き込み | [詳細説明](./Manual/Melco/Melsec_WriteWord.Md) |
| **`Melsec_ReadBit`** | `Melsec_ReadBit, "<デバイス>", "<点数>", "<格納先>", "[局番]"` | ビットデバイス（X, Y, M, L, F, V 等）の一括読み出し | [詳細説明](./Manual/Melco/Melsec_ReadBit.Md) |
| **`Melsec_WriteBit`** | `Melsec_WriteBit, "<デバイス>", "<ビット列>", "[局番]"` | ビットデバイス（Y, M, L, B 等）の一括書き込み | [詳細説明](./Manual/Melco/Melsec_WriteBit.Md) |
| **`Melsec_Loopback`** | `Melsec_Loopback, "<テスト文字列>", "[格納先]", "[局番]"` | シーケンサ通信回線の折り返し通信診断（ループバックテスト `BT`） | [詳細説明](./Manual/Melco/Melsec_Loopback.Md) |

### 回線制御・通信条件設定コマンド（3 コマンド）

| コマンド名 | 構文概要 | 主な役割・用途 | 詳細マニュアル |
| :--- | :--- | :--- | :---: |
| **`Melsec_Open`** | `Melsec_Open` | MELSEC 通信回線をオープンし送受信監視を開始 | [詳細説明](./Manual/Melco/Melsec_Open.Md) |
| **`Melsec_Close`** | `Melsec_Close` | MELSEC 通信回線を安全に切断・リソース解放 | [詳細説明](./Manual/Melco/Melsec_Close.Md) |
| **`Melsec_Mode`** | `Melsec_Mode, "<設定1>", "[設定2]", "[形式: P1〜P5]"` | プロトコル形式（P1〜P5）や通信条件の動的設定 | [詳細説明](./Manual/Melco/Melsec_Mode.Md) |

### 計算機リンク形式別 送受信コマンド（15 コマンド）

| プロトコル形式 | 送信コマンド | 要求受信コマンド | 応答受信・抽出コマンド |
| :--- | :--- | :--- | :--- |
| **形式 1** （対話＋サムチェック） | [**`Melsec_P1s_S`**](./Manual/Melco/Melsec_P1s_S.Md) | [**`Melsec_P1s_R`**](./Manual/Melco/Melsec_P1s_R.Md) | [**`Melsec_Res_P1s_R`**](./Manual/Melco/Melsec_Res_P1s_R.Md) |
| **形式 2** （対話・サムチェックなし） | [**`Melsec_P2s_S`**](./Manual/Melco/Melsec_P2s_S.Md) | [**`Melsec_P2s_R`**](./Manual/Melco/Melsec_P2s_R.Md) | [**`Melsec_Res_P2s_R`**](./Manual/Melco/Melsec_Res_P2s_R.Md) |
| **形式 3** （CR/LF終端・サムチェックなし） | [**`Melsec_P3s_S`**](./Manual/Melco/Melsec_P3s_S.Md) | [**`Melsec_P3s_R`**](./Manual/Melco/Melsec_P3s_R.Md) | [**`Melsec_Res_P3s_R`**](./Manual/Melco/Melsec_Res_P3s_R.Md) |
| **形式 4** （CR/LF終端＋サムチェック） | [**`Melsec_P4s_S`**](./Manual/Melco/Melsec_P4s_S.Md) | [**`Melsec_P4s_R`**](./Manual/Melco/Melsec_P4s_R.Md) | [**`Melsec_Res_P4s_R`**](./Manual/Melco/Melsec_Res_P4s_R.Md) |
| **形式 5** （高速バイナリ通信） | [**`Melsec_P5s_S`**](./Manual/Melco/Melsec_P5s_S.Md) | [**`Melsec_P5s_R`**](./Manual/Melco/Melsec_P5s_R.Md) | [**`Melsec_Res_P5s_R`**](./Manual/Melco/Melsec_Res_P5s_R.Md) |

---

## 💻 動作環境・システム要件

| 項目 | システム要件 |
| :--- | :--- |
| **OS** | Windows 10 / Windows 11 (64bit / 32bit) |
| **対応 Excel** | Microsoft Excel 2016 / 2019 / 2021 / Microsoft 365 (デスクトップ版 32bit / 64bit) |
| **必須アドイン** | [TermAddIn](https://github.com/Monaca-App/TermAddIn) (Excel VSTO アドイン) |
| **対象 PLC** | 三菱電機 MELSEC-Q / L / iQ-R / A / FX シリーズ（計算機リンク / シリアルコミュニケーションユニット / 内蔵 Ethernet） |
| **インターフェース** | RS-232C, RS-422, RS-485, USB 仮想 COM ポート, [c2c](https://github.com/Monaca-App/c2c) 仮想ヌルモデムポート |
| **対応プロトコル** | MELSEC 計算機リンク プロトコル 1 〜 5（形式 1 / 形式 2 / 形式 3 / 形式 4 / 形式 5） |

---

## 🚀 インストール手順

<a href="https://github.com/Monaca-App/TermAddIn_Melco/raw/main/Setup/TermAddIn_Melco_Setup.zip">
  <img src="images/Monaca_app_get.jpg" width="140" alt="プログラム（入手）" />
</a>

> 上記の **「プログラム（入手）」アイコン** をクリックすると、セットアッププログラム一式（[`TermAddIn_Melco_Setup.zip`](https://github.com/Monaca-App/TermAddIn_Melco/raw/main/Setup/TermAddIn_Melco_Setup.zip)）が直接ダウンロードされます。展開後、`TermAnalysis.Melco\setup.exe`（または `TermAnalysis.Melco.Setup.msi`）を実行してください。

> [!IMPORTANT]
> **導入前提**  
> 本エクステンションを利用するには、事前に **[TermAddIn 本体](https://github.com/Monaca-App/TermAddIn)** （`Term.Setup` および `TermAddIn.Setup`）をインストールしておく必要があります。

### 配布アーカイブ（本リポジトリ `Setup/` 配下）

* **アーカイブ**: [`Setup/TermAddIn_Melco_Setup.zip`](https://github.com/Monaca-App/TermAddIn_Melco/raw/main/Setup/TermAddIn_Melco_Setup.zip)（収録内容: `TermAnalysis.Melco/setup.exe`, `TermAnalysis.Melco/TermAnalysis.Melco.Setup.msi`）

### 導入ステップ

1. **Microsoft Excel** および **Term** をすべて終了します。
2. ダウンロードした **`TermAddIn_Melco_Setup.zip`** を展開し、 **`TermAnalysis.Melco/`** フォルダ内の **`setup.exe`** （または **`TermAnalysis.Melco.Setup.msi`** ）を実行します。
   * ※ 管理者権限（UAC 承認）が必要です。
   * ※ 拡張 DLL は共通インストール先（`C:\Program Files (x86)\Monaca_App_Store\Monaca Common\`）へ配置され、サンプル資材が `C:\Program Files (x86)\Monaca_App_Store\Monaca Sample\Melco\` へ導入されます。
3. インストール完了後、Excel（または `Term.exe`）を起動し、メイン画面の **`[通信モード]`** メニューから **`MELSEC`** が選択できることを確認します。

### アンインストール手順
* Windows の「設定」>「アプリ」>「インストールされているアプリ」（または「コントロールパネル」の「プログラムと機能」）から **`TermAnalysis.Melco.Setup`** を選択してアンインストールしてください。

---

## 🔰 シーケンス記述例・同梱サンプル資材（`Sample/Melco/`）

プロトコル 4（形式 4）を用いた MELSEC データレジスタ（D100）一括読出の構成例です。

```csv
; 1. MELSEC 通信回線をオープン（プロトコル 4 形式）
,SEQUENCE, 010, Melsec_Mode, "9600,E,7,1", "NONE", "P4"
,SEQUENCE, 020, Melsec_Open
; 2. プロトコル 4 形式で D100 から 2 点読出コマンドを送信
,SEQUENCE, 030, Melsec_P4s_S, "00FFWR0D010002"
; 3. 応答電文を受信検証し、デバイス値を 10進数で抽出しセル展開
,SEQUENCE, 040, Melsec_Res_P4s_R, "00FF", "0002", "PLC_D100", "DEC"
; 4. タイムアウト判定（無応答時は #TIMEOUT# へ分岐）
,SEQUENCE, 050, Judge, NEQ, "$PLC_D100$", "ERROR_TIMEOUT", #TIMEOUT#
; 5. 通信回線をクローズして終了
,SEQUENCE, 060, Melsec_Close
,SEQUENCE, 070, Exit
; タイムアウト処理ブロック
,SEQUENCE, #TIMEOUT#, Msg_Box, "PLC からの応答がありません。"
,SEQUENCE, 080, Melsec_Close
,SEQUENCE, 090, Exit
```

### 同梱サンプルマクロ・AI自動生成プロンプト集（形式 1 〜 形式 5）
計算機リンクの全 5 形式それぞれに対応した動作確認用マクロスクリプト（CSV）および AI 自動生成プロンプトが **`Sample/Melco/`** に収録されています（インストール時には `C:\Program Files (x86)\Monaca_App_Store\Monaca Sample\Melco\` へ配置され、デスクトップの `Monaca\Sample` ショートカットから参照できます）。

| プロトコル形式 | サンプルマクロ（Shift-JIS CSV） | AI 自動生成プロンプト |
| :--- | :--- | :--- |
| **形式 1** （対話＋サムチェック） | [`Sample/Melco/Melco_P1.csv`](./Sample/Melco/Melco_P1.csv) | [`Sample/Melco/Melco_P1_prompt.md`](./Sample/Melco/Melco_P1_prompt.md) |
| **形式 2** （対話・サムチェックなし） | [`Sample/Melco/Melco_P2.csv`](./Sample/Melco/Melco_P2.csv) | [`Sample/Melco/Melco_P2_prompt.md`](./Sample/Melco/Melco_P2_prompt.md) |
| **形式 3** （CR/LF終端・サムチェックなし） | [`Sample/Melco/Melco_P3.csv`](./Sample/Melco/Melco_P3.csv) | [`Sample/Melco/Melco_P3_prompt.md`](./Sample/Melco/Melco_P3_prompt.md) |
| **形式 4** （CR/LF終端＋サムチェック） | [`Sample/Melco/Melco_P4.csv`](./Sample/Melco/Melco_P4.csv) | [`Sample/Melco/Melco_P4_prompt.md`](./Sample/Melco/Melco_P4_prompt.md) |
| **形式 5** （高速バイナリ通信） | [`Sample/Melco/Melco_P5.csv`](./Sample/Melco/Melco_P5.csv) | [`Sample/Melco/Melco_P5_prompt.md`](./Sample/Melco/Melco_P5_prompt.md) |

---

## 💳 ライセンス・購入プラン（TermAddIn_Melco）

> [!NOTE]
> **試用動作およびオンライン決済について**  
> ライセンス未配置・無効時は、Excel アドイン（`TermAddIn`）・`Term.exe` 単体起動ともに 10 分間の期間限定版として動作します（10 分超過後は未保有機能のみスルー／ロック）。  
> 用途や利用期間に合わせて、一括購入（買い切り）またはサブスクリプション（月額・週額）からお選びいただけます。メールアドレスの誤入力によるライセンス不着を防ぐため、 **事前のメールアドレス確認** を行った上で、 **Stripe** による安全なオンライン決済へご案内いたします（表記価格はすべて税別です）。

| プラン | 価格（税別） | お支払い・お申込み |
| :--- | :---: | :---: |
| **一括購入（買い切りライセンス）**<br>無期限で永続利用できるライセンスです。 | **¥32,000** | [👉 一括購入手続きへ](https://stripe-license-generator.y-nakai-5a3.workers.dev/?target=melco) |
| **月額サブスクリプション**<br>月単位で利用できる定期プランです。中長期のプロジェクトにおすすめです。 | **¥3,000** / 月 | [👉 月額プランに申し込む](https://stripe-license-generator.y-nakai-5a3.workers.dev/?target=melco) |
| **週額サブスクリプション**<br>週単位で手軽に利用できるプランです。短期評価や単発テストにおすすめです。 | **¥1,000** / 週 | [👉 週額プランに申し込む](https://stripe-license-generator.y-nakai-5a3.workers.dev/?target=melco) |

* ※ **ライセンスファイル名**: `TermAddIn_Melco.License`
* ※ **配置先**: `%USERPROFILE%\Desktop\Monaca\TermAddIn_Melco.License`（設定ファイル `Setting.xml` と同階層）
* ※ **ライセンス形態**: 1 ユーザー / 1 PC（ノードロック方式：`PC名` および `ログインユーザー名` に紐づけ）

---

## 📄 ライセンス & サポート

* **開発元**: [Monaca アプリ工房 (Monaca App Store)](https://github.com/Monaca-App/store)
* **親プロジェクト**: [TermAddIn](https://github.com/Monaca-App/TermAddIn)
* **関連コンパニオンツール**: [c2c - ドライバ不要の仮想ヌルモデム＆通信モニター](https://github.com/Monaca-App/c2c)
* **Copyright**: Copyright (C) Monaca App Store 2026 All rights reserved.
* **お問い合わせ**: 不具合報告や機能要望は、GitHub の Issue までお気軽にお寄せください。
