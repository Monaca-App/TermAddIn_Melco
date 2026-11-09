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

**TermAddIn_Melco** は、Excel 総合通信アドイン **TermAddIn** 上で動作する **三菱電機 MELSEC 計算機リンク通信専用エクステンション（機能ドメイン単一 DLL）** です。

三菱電機製シーケンサ（MELSEC-Q / L / iQ-R / A / FX シリーズ）とのシリアル通信（RS-232C / RS-422 / RS-485）およびイーサネット計算機リンク通信を Excel ワークシート上から完全自動化します。

VBA マクロを一切記述することなく、 **標準の `.xlsx`（マクロ無効ブック）形式のままで MELSEC 内部デバイス（Dレジスタ、X入力、Y出力、Mリレー等）のビット・ワード単位の一括読出・書込、サムチェック自動計算・照合** が安全に実行できます。

> [!NOTE]
> **利用前提とライセンス体系**  
> 本エクステンションは [TermAddIn 本体](https://github.com/Monaca-App/TermAddIn) 上で動作します。  
> TermAddIn ではプロトコル毎に個別ライセンスを設ける体系を採用しており、本機能のご利用には **`TermAddIn_Melco` 個別ライセンス（料金未定・準備中）** が適用されます（現在は評価期間として全機能を無償でご試用いただけます）。

---

## ✨ 主な機能・特徴

1. **MELSEC 計算機リンク「プロトコル 1」および「プロトコル 4」完全対応**
   * Checksum 付きの標準対話形式である「プロトコル 1（形式 1）」、および Checksum＋CR/LF 終端の「プロトコル 4（形式 4）」の双方をサポート。
2. **フレームヘッダ・サムチェック・ACK/NAK の完全自動化**
   * 要求電文送信時の局番・PC 番号・コマンド・サムチェック等のフレーム生成、および受信応答時のサムチェック照合・ACK 返信をエンジンが内部で自動実行。
3. **ビット・ワードデバイスの自在な読み書き**
   * D（データレジスタ）、W（ファイルレジスタ）、R（拡張レジスタ）のワード読出・書込、および X（入力）、Y（出力）、M（内部リレー）のビット単位制御に完全対応。
4. **取得データの 16進 / 10進 変換とセル自動展開**
   * シーケンサから返却されたデバイス値は、指定した結果セルへ 16 進または 10 進の数値データとしてリアルタイムに自動展開。
5. **Excel 32bit / 64bit 完全動作マトリクス**
   * .NET Framework 標準マネージド通信基盤により構築されており、外部の専用ネイティブ DLL に依存しないため、Excel 32bit 版および 64bit 版の双方で制限なく完全動作。

---

## 🔌 提供コマンド一覧（全 9 コマンド）

各コマンドの詳細な構文・パラメータ仕様・記述例は、以下の個別リファレンスをご参照ください。

| コマンド名 | 構文概要 | 主な役割・用途 | 詳細マニュアル |
| :--- | :--- | :--- | :---: |
| **`Melsec_Open`** | `Melsec_Open` | MELSEC 通信回線をオープンし送受信監視を開始 | [詳細説明](./Manual/Melco/Melsec_Open.Md) |
| **`Melsec_Close`** | `Melsec_Close` | MELSEC 通信回線を安全に切断・リソース解放 | [詳細説明](./Manual/Melco/Melsec_Close.Md) |
| **`Melsec_Mode`** | `Melsec_Mode, "[局番]", "[PC番号]", "[形式]", "[通信条件]"` | プロトコル形式（P1/P4）や局番等の動的設定 | [詳細説明](./Manual/Melco/Melsec_Mode.Md) |
| **`Melsec_P1s_S`** | `Melsec_P1s_S, "<コマンド電文>"` | プロトコル 1 形式での要求電文送信 | [詳細説明](./Manual/Melco/Melsec_P1s_S.Md) |
| **`Melsec_P1s_R`** | `Melsec_P1s_R, "[タイムアウトms]", "[ジャンプ先]"` | プロトコル 1 形式での応答電文受信待機 | [詳細説明](./Manual/Melco/Melsec_P1s_R.Md) |
| **`Melsec_Res_P1s_R`** | `Melsec_Res_P1s_R, "<格納先キー>", "[基数]"` | プロトコル 1 応答の検証およびデバイス値抽出 | [詳細説明](./Manual/Melco/Melsec_Res_P1s_R.Md) |
| **`Melsec_P4s_S`** | `Melsec_P4s_S, "<コマンド電文>"` | プロトコル 4 形式での要求電文送信 | [詳細説明](./Manual/Melco/Melsec_P4s_S.Md) |
| **`Melsec_P4s_R`** | `Melsec_P4s_R, "[タイムアウトms]", "[ジャンプ先]"` | プロトコル 4 形式での応答電文受信待機 | [詳細説明](./Manual/Melco/Melsec_P4s_R.Md) |
| **`Melsec_Res_P4s_R`** | `Melsec_Res_P4s_R, "<格納先キー>", "[基数]"` | プロトコル 4 応答の検証およびデバイス値抽出 | [詳細説明](./Manual/Melco/Melsec_Res_P4s_R.Md) |

---

## 💻 動作環境・システム要件

| 項目 | システム要件 |
| :--- | :--- |
| **OS** | Windows 10 / Windows 11 (64bit / 32bit) |
| **対応 Excel** | Microsoft Excel 2016 / 2019 / 2021 / Microsoft 365 (デスクトップ版 32bit / 64bit) |
| **必須アドイン** | [TermAddIn](https://github.com/Monaca-App/TermAddIn) (Excel VSTO アドイン) |
| **対象 PLC** | 三菱電機 MELSEC-Q / L / iQ-R / A / FX シリーズ（計算機リンク / シリアルコミュニケーションユニット / 内蔵 Ethernet） |
| **対応プロトコル** | MELSEC 計算機リンク プロトコル 1（形式 1） / プロトコル 4（形式 4） |

---

## 🔰 シーケンス記述例（Excel セル設定）

プロトコル 4（形式 4）を用いた MELSEC データレジスタ（D100）一括読出の構成例です。

```csv
; 1. MELSEC 通信回線をオープン
,SEQUENCE, 010, Melsec_Open
; 2. プロトコル 4 形式で D100 から 2 点読出コマンドを送信
,SEQUENCE, 020, Melsec_P4s_S, "00FFWR0D010002"
; 3. 応答電文の受信待機（最大 3000ms）
,SEQUENCE, 030, Melsec_P4s_R, 3000, "#TIMEOUT"
; 4. 受信応答からデバイス値を 10進数で抽出しセル展開
,SEQUENCE, 040, Melsec_Res_P4s_R, "PLC_D100", "DEC"
; 5. 通信回線をクローズして終了
,SEQUENCE, 050, Melsec_Close
,SEQUENCE, 060, Exit
; タイムアウト処理ブロック
#TIMEOUT
,SEQUENCE, 070, Msg_Box, "PLC からの応答がありません。"
,SEQUENCE, 080, Melsec_Close
```

---

## 💳 ライセンス・購入プラン（TermAddIn_Melco）

> [!NOTE]
> **料金プランおよび正式販売について**  
> 本エクステンション（`TermAddIn_Melco`）の販売料金およびライセンスプラン（買い切り・サブスクリプション）は **現在検討中（未定）** です。  
> 正式な価格およびお申込み受付開始時期が決定次第、本ドキュメントおよび [Monaca アプリ工房ポータル](https://github.com/Monaca-App/store) にてご案内いたします。  
> 現在は評価版として全機能をご試用いただけます。

| プラン | 価格 | お支払い・お申込み |
| :--- | :---: | :---: |
| **一括購入（買い切りライセンス）**<br>無期限で永続利用できるライセンスです。 | **未定** | 準備中 |
| **月額サブスクリプション**<br>月単位で利用できる定期プランです。中長期のプロジェクトにおすすめです。 | **未定** | 準備中 |
| **週額サブスクリプション**<br>週単位で手軽に利用できるプランです。短期評価や単発テストにおすすめです。 | **未定** | 準備中 |

* ※ **ライセンスファイル名**: `TermAddIn_Melco.License`
* ※ **配置先**: `%USERPROFILE%\Desktop\Monaca\TermAddIn_Melco.License`（設定ファイル `Setting.xml` と同階層）
* ※ **ライセンス形態**: 1 ユーザー / 1 PC（ノードロック方式：`PC名` および `ログインユーザー名` に紐づけ）

---

## 📄 ライセンス & サポート

* **開発元**: [Monaca アプリ工房 (Monaca App Store)](https://github.com/Monaca-App/store)
* **親プロジェクト**: [TermAddIn](https://github.com/Monaca-App/TermAddIn)
* **Copyright**: Copyright (C) Monaca App Store 2026 All rights reserved.
* **お問い合わせ**: 不具合報告や機能要望は、GitHub の Issue までお気軽にお寄せください。
