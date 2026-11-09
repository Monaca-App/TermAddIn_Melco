# コマンドリファレンス：Melsec_Close

---

## 概要

**Melsec_Close** は、現在オープンされている MELSEC 計算機リンク通信用のシリアルポート（UART 回線）を安全に切断し、通信ハンドルを解放するコマンドです。

> 💡 **補足**: シーケンス終了時には必ず `Melsec_Close` を実行して COM ポートを解放してください。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE,Melsec_Close
```

### パラメータ（引数）一覧

| 引数位置 | パラメータ名 | 必須/任意 | 型・制約 | 説明 |
| :--- | :--- | :---: | :--- | :--- |
| （引数なし） | - | - | - | 現在接続中の MELSEC 通信用 COM ポートをクローズします。 |

---

## 🔄 特殊置換規約・分岐規約の適用

1. **引数なしコマンド**:
   - 引数を取らないため、変数展開（`$` / `#` / `@`）は使用しません。
2. **大文字・小文字の非区別（Case-Insensitive）**:
   - `Melsec_Close`、`MELSEC_CLOSE`、`melsec_close` のいずれも同一コマンドとして正規化されます。

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

```csv
,SEQUENCE,Melsec_Mode, "9600,O,7,1", "NONE", "P1"
,SEQUENCE,Melsec_Open
,SEQUENCE,Melsec_P1s_S, "00", "WW", "0D0000011234"
,SEQUENCE,Melsec_Res_P1s_R, "00", "@ACK_STATUS"
,SEQUENCE,Melsec_Close
,SEQUENCE,Exit
```

### 実行時の挙動・変数変化

1. `Melsec_Res_P1s_R` でシーケンサからの ACK 応答を確認した後、`Melsec_Close` によりシリアルポートが切断されます。

---

## 🔄 動作フロー（シーケンス図）

```mermaid
sequenceDiagram
    autonumber
    participant Script as "スクリプト実行エンジン (SequenceRunner)"
    participant Cmd as "Melsec_CloseCommand (TermAnalysis.Melco)"
    participant Port as "SerialPort / MELSEC ドライバ"

    Script->>Cmd: Execute(context)
    Cmd->>Port: IsOpen 状態確認
    alt 回線がオープン中の場合
        Cmd->>Port: DataReceived 購読解除・Close()
        Port-->>Cmd: クローズ完了
        Cmd-->>Script: 正常終了 (Next Step)
    else 既にクローズ済みの場合
        Cmd-->>Script: 何もしないで正常終了 (No-Op)
    end
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

1. **未接続時の安全な無視（No-Op 保証）**:
   - 未接続状態で `Melsec_Close` を実行しても例外は発生せず、安全に次のステップへ進みます。
2. **受信済みデータの保持**:
   - クローズ後も、直前の受信コマンドで変数や `TermRingBufferManager` に格納されたデータは消去されず参照可能です。

---

## 🧪 ユニットテスト仕様

### テスト実行前提条件
- **対象クラス**: `TermAnalysis.Melco.Melsec_CloseCommand`（`Melsec_CloseCommand.cs`）
- **モック環境**: `IMelsecTransportAdapter` モックが注入されていること。

### テストケース一覧

| テストID | テスト観点 | 入力スクリプト | 期待される動作・検証項目 |
| :--- | :--- | :--- | :--- |
| `UT-MELSEC-CLOSE-01` | 正常クローズ | `,SEQUENCE,Melsec_Close`（接続中） | トランスポートの `IsOpen` が `false` になりリソースが解放されること。 |
| `UT-MELSEC-CLOSE-02` | 未接続時の再クローズ | `,SEQUENCE,Melsec_Close`（未接続時） | 例外が発生せず正常終了すること。 |

