# コマンドリファレンス：Melsec_Res_P2s_R

## 概要

**Melsec_Res_P2s_R** は、三菱電機 MELSEC 計算機リンク **形式 2（サムチェックなし対話手順）** の応答電文を受信検証し、STX〜ETX で囲まれたデータ部（または ACK / NAK）を抽出して指定のセルまたはレジスタへ格納するコマンドです。

形式 1 と異なり、ETX 直後のサムチェック（2 桁 ASCII）の照合は省略されます。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], Melsec_Res_P2s_R, [StationNoOrHead], "[TargetOrPoints]", "[LegacyDest]", "[LegacyFormat]"
```

### パラメータ（引数）一覧

| 引数位置 | 引数名 | 必須/任意 | 説明・制約 |
| :--- | :--- | :---: | :--- |
| 第 1 引数 | `StationNoOrHead` | **必須** | 期待するシーケンサ局番または応答ヘッダ文字列（例: `"00"` または `"00FF"`）。 |
| 第 2 引数 | `TargetOrPoints` | 任意 | 2 引数標準構文時は抽出データの格納先変数キー名、旧 3〜4 引数構文時は 16 進読出点数（省略時既定値: `""`）。 |
| 第 3 引数 | `LegacyDest` | 任意 | 旧 3〜4 引数互換構文における格納先レジスタ／変数キー名（省略時既定値: `""`）。 |
| 第 4 引数 | `LegacyFormat` | 任意 | 旧 4 引数互換構文における取得データの基数・書式変換指定（省略時既定値: `""`）。 |

---

## 🔄 特殊置換規約・分岐規約の適用

本コマンドの引数における特殊記法および解釈ルールです。
* **動的置換（`$`）**: 引数内で `$PARAMキー$` のように記述すると、実行時に変数プール（PARAM）の値へ動的置換されます。（適用可能な引数: 全引数対応）
* **ジャンプ先指定（`#`）**: 本コマンドはジャンプ制御を行わないため適用されません。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）**: 本コマンドは外部マクロを呼び出さないため、共通コマンド呼び出し記法は適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ**: 通信条件文字列（例: `"9600,E,7,1"`）や複数データのカンマ区切り値を単一引数として渡す場合は、必ず `"..."` と二重引用符で囲んで記述します。
* **エイリアス（別名）**: `Res_P2s_R`, `Analysis_Res_P2s_R`, `Sqn_Res_P2s_R`, `Analysis_SQN_RES_P2S_R`

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※COMMAND ブロック配下の子要素行（PARAM, SEQUENCE）は必ず行頭カンマから開始し、スペースインデントや行末コメント（`;`）は記述しないでください。

```csv
COMMAND, "SAMPLE_MELSEC_RES_P2S_R", ""
; 形式2でD100の読出要求を送信し、STX..ETX応答を PLC_D100 へ格納
,SEQUENCE, 010, Melsec_P2s_S, "00", "FFWR0", "D010001"
,SEQUENCE, 020, Melsec_Res_P2s_R, "00", "PLC_D100"
```

### 実行時の挙動・変数変化

* **実行前の状態**: Term 通信エンジンが初期化され、必要な通信パラメータおよび変数プール（`PARAM`）が準備されている状態。
* **実行後の状態**: ステップ実行により `Melsec_Res_P2s_R` の通信制御・プロトコル処理が正常に遂行され、結果または回線状態が更新されます。

---

## 🔄 動作フロー（シーケンス図）

```mermaid
sequenceDiagram
    autonumber
    participant Script as "スクリプト実行エンジン"
    participant Cmd as "Melsec_Res_P2s_RCommand"
    participant Transport as "UartTransportCore / 受信リングバッファ"

    Script->>Cmd: コマンド実行要求
    Cmd->>Transport: STX..ETX フレーム取得 (サムチェック照合なし)
    Cmd->>Cmd: データ部抽出および基数変換 (HEX / DEC)
    Cmd-->>Script: 正常終了
    alt パラメータおよびプロトコル検証が正常な場合
        Cmd-->>Script: 正常完了 (IsCommandGo = true)
    else 引数範囲外または通信状態異常の場合
        Cmd->>Cmd: エラーログ出力 (_log.LogError)
        Cmd-->>Script: 安全停止 (SetErrorAndHalt)
    end
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

入力データの異常や実行時エラーに対し、解析・実行エンジンが実行するセーフティ規約です。

1. **引数範囲・プロトコル書式バリデーション保護**:
   - 必須引数の欠落、デバイス表記・数値範囲の不正、または余剰引数が指定された場合は `_log.LogError` に詳細を記録し、`SetErrorAndHalt` により異常電文の送出を未然に遮断します。
2. **通信タイムアウト・異常応答・排他ドメイン保護**:
   - 相手機器の無応答タイムアウト、NAK／異常終了コード応答、または他通信ドメインとの排他競合が発生した場合は、例外クラッシュを起こさず安全にエラーハンドリングを行います。

---

## 🧪 ユニットテスト仕様

本コマンドクラス（`Melsec_Res_P2s_RCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー**: `RunUnitTest()` メソッドの呼び出し
* **有効化条件**: システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること（それ以外の場合は無効化ガードにより早期リターン）
* **検証結果出力**: `_log.LogTest` によるログ出力。検証失敗時は `_log.LogError`、予期せぬ例外発生時は `_log.LogCritical` で記録し、`finally` ブロックで `testSbuf?.Clear()`、`TermRingBufferManager.Instance.ClearBuffer()`、および `UartTransportCore.Shared.IsLoopbackMode` の復元を実行します。

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | `IsLoopbackMode = true` 環境下で引数 `"00"`, `"RESULT_KEY"` およびサムチェックなし擬似応答 `"\u000200FF1234ABCD\u0003"` を与えて実行 | `!param.HasError` かつ `param.IsCommandGo == true`、サムチェック照合なしでデータ部が抽出され `SBuf["RESULT_KEY"]` へ格納される | 形式 2 `STX`〜`ETX` 応答電文（サムチェックなし）の解析および格納検証 |
| **2** | 異常系 | 引数不足（引数 0 個の `TermSequence`）および `Func(null, null)` を指定して実行 | `ValidateParamRange(1, 4)` により `!invalidParam.IsCommandGo` で安全に拒否され、`null` 入力でもクラッシュしない | 必須引数欠落および `null` パラメータに対する安全防御検証 |
