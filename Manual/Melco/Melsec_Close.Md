# コマンドリファレンス：Melsec_Close

## 概要

**Melsec_Close** は、現在オープンされている MELSEC 計算機リンク通信用のシリアルポート（UART 回線）を安全に切断し、通信ハンドルを解放するコマンドです。

> 💡 **補足**: シーケンス終了時には必ず `Melsec_Close` を実行して COM ポートを解放してください。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], Melsec_Close
```

### パラメータ（引数）一覧

| 引数位置 | パラメータ名 | 必須/任意 | 型・制約 | 説明 |
| :--- | :--- | :---: | :--- | :--- |
| （引数なし） | - | - | - | 現在接続中の MELSEC 通信用 COM ポートをクローズします。 |

---

## 🔄 特殊置換規約・分岐規約の適用

本コマンドの引数における特殊記法および解釈ルールです。
* **動的置換（`$`）**: 本コマンドは引数を取らないため、引数に対する `$PARAMキー$` 動的置換は適用されません。（適用可能な引数: 非適用）
* **ジャンプ先指定（`#`）**: 本コマンドはジャンプ制御を行わないため適用されません。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）**: 本コマンドは外部マクロを呼び出さないため、共通コマンド呼び出し記法は適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ**: 通信条件文字列（例: `"9600,E,7,1"`）や複数データのカンマ区切り値を単一引数として渡す場合は、必ず `"..."` と二重引用符で囲んで記述します。
* **エイリアス（別名）**: なし（標準コマンド名 `Melsec_Close` のみ。大文字・小文字は区別されません）

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※COMMAND ブロック配下の子要素行（PARAM, SEQUENCE）は必ず行頭カンマから開始し、スペースインデントや行末コメント（`;`）は記述しないでください。

```csv
COMMAND, "SAMPLE_MELSEC_CLOSE", ""
,SEQUENCE, 010, Melsec_Mode, "9600,O,7,1", "NONE", "P1"
,SEQUENCE, 020, Melsec_Open
,SEQUENCE, 030, Melsec_P1s_S, "00", "WW", "0D0000011234"
,SEQUENCE, 040, Melsec_Res_P1s_R, "00", "@ACK_STATUS"
,SEQUENCE, 050, Melsec_Close
,SEQUENCE, 060, Exit
```

### 実行時の挙動・変数変化

* **実行前の状態**: Term 通信エンジンが初期化され、必要な通信パラメータおよび変数プール（`PARAM`）が準備されている状態。
* **実行後の状態**: `Melsec_Res_P1s_R` でシーケンサからの ACK 応答を確認した後、`Melsec_Close` によりシリアルポートが切断されます。

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

本コマンドクラス（`Melsec_CloseCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー**: `RunUnitTest()` メソッドの呼び出し
* **有効化条件**: システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること（それ以外の場合は無効化ガードにより早期リターン）
* **検証結果出力**: `_log.LogTest` によるログ出力。検証失敗時は `_log.LogError`、予期せぬ例外発生時は `_log.LogCritical` で記録し、`finally` ブロックで `UartTransportCore.Shared.IsSimulationMode` を開始前の状態へ復元します。

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | `IsSimulationMode = true` 環境下で `OpenExclusiveSession(TermTransportDomain.Melsec, null, useSocket: false)` 確立後、引数なしで `Melsec_Close` を実行 | `!param.HasError` かつ `ActiveDomain == TermTransportDomain.None` となる | MELSEC 計算機リンク排他セッションの正常切断およびポート解放の検証 |
| **2** | 異常系 | 余剰引数の指定（`ValidateParamRange(0, 0)` 違反）または切断処理中の例外発生 | 引数範囲外エラー出力による早期リターン、または `_log.LogCritical` 記録と `SetErrorAndHalt(-1, ex.Message)` による安全停止 | 引数なし制約の境界検証および回線切断例外時のクラッシュ防御検証 |
