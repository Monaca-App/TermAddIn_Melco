# コマンドリファレンス：Melsec_P2s_R

---

## 概要

**Melsec_P2s_R** は、三菱電機 MELSEC 計算機リンク **形式 2（サムチェックなし対話手順）** の要求電文を受信検証し、データ長および局番ヘッダを確認した上で抽出データを指定のレジスタまたはセルへ格納するコマンドです。

サムチェック照合を行わない設定の PLC または外部機器との通信で使用します。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE,Melsec_P2s_R, "<StationNoOrHeader>", "<SizeNumOrCommand>", "<TargetOrDataLength>", "[Format]"
```

### パラメータ（引数）一覧

| 引数位置 | パラメータ名 | 必須/任意 | 型・制約 | 説明 |
| :---: | :--- | :---: | :--- | :--- |
| 第1引数 | `StationNoOrHeader` | **必須** | 文字列（例: `"00FF"`） | 期待する局番またはヘッダ文字列を指定します。 |
| 第2引数 | `SizeNumOrCommand` | **必須** | 数値または文字列 | データ長桁数またはコマンドを指定します。 |
| 第3引数 | `TargetOrDataLength` | **必須** | 文字列 | 抽出データの格納先レジスタ名、またはデータ長を指定します。 |
| 第4引数 | `Format` | 任意 | 文字列（`"HEX"` / `"DEC"` 等） | 抽出データの変換フォーマットを指定します。 |

---

## 💡 動作例

```csv
; 形式2で要求電文を受信し、RECV_BUFへ格納
,SEQUENCE,Melsec_P2s_R, "00FF", "2", "RECV_BUF"
```

---

## 🔄 動作フロー（シーケンス図）

```mermaid
sequenceDiagram
    autonumber
    participant Script as "スクリプト実行エンジン"
    participant Cmd as "Melsec_P2s_RCommand"
    participant Buffer as "受信バッファ / リングバッファ"

    Script->>Cmd: コマンド実行要求
    Cmd->>Buffer: フレーム文字列取得 (サムチェック照合なし)
    Cmd->>Cmd: ヘッダ照合およびデータ部抽出
    Cmd-->>Script: 格納完了通知
```

