# コマンドリファレンス：Melsec_Res_P5s_R

## 概要

**Melsec_Res_P5s_R** は、三菱電機 MELSEC 計算機リンク **形式 5（バイナリコード通信方式）** の応答電文を受信検証し、DLE フレーミング・チェックサムおよび終了コード（正常 `0x0000`）を確認した上で、応答データを 16 進文字列（`HEX`）または 10 進整数（`DEC`）に変換して指定セルへ展開するコマンドです。

書込コマンドに対する正常応答（`DLE ACK [0x10 0x06]`）やエラー応答（`DLE NAK [0x10 0x15]`）の自動判定にも完全対応しています。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], Melsec_Res_P5s_R, "<StationNoOrTarget>", "[TargetOrRadix]", "[RadixFormat]"
```

### パラメータ（引数）一覧

| 引数位置 | パラメータ名 | 必須/任意 | 型・制約 | 説明 |
| :---: | :--- | :---: | :--- | :--- |
| 第1引数 | `StationNoOrTarget` | **必須** | 文字列（例: `"00"`, `"PLC_D100"`） | 期待する局番（16進）、または格納先セル/レジスタ名を指定します。 |
| 第2引数 | `TargetOrRadix` | 任意 | 文字列（例: `"PLC_D100"`, `"DEC"`, `"HEX"`） | 格納先キー、または基数変換フォーマットを指定します。 |
| 第3引数 | `RadixFormat` | 任意 | 文字列（既定: `"HEX"`, `"DEC"` 可） | 取得データの基数変換フォーマットを指定します。`"DEC"` 指定時は 2 バイト単位で 10 進数値に変換します。 |

---

## 🔄 特殊置換規約・分岐規約の適用

本コマンドの引数における特殊記法および解釈ルールです。
* **動的置換（`$`）**: 引数内で `$PARAMキー$` のように記述すると、実行時に変数プール（PARAM）の値へ動的置換されます。（適用可能な引数: 全引数対応）
* **ジャンプ先指定（`#`）**: 本コマンドはジャンプ制御を行わないため適用されません。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）**: 本コマンドは外部マクロを呼び出さないため、共通コマンド呼び出し記法は適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ**: 通信条件文字列（例: `"9600,E,7,1"`）や複数データのカンマ区切り値を単一引数として渡す場合は、必ず `"..."` と二重引用符で囲んで記述します。
* **エイリアス（別名）**: `Res_P5s_R`, `Analysis_Res_P5s_R`, `Sqn_Res_P5s_R`, `Analysis_SQN_RES_P5S_R`

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※COMMAND ブロック配下の子要素行（PARAM, SEQUENCE）は必ず行頭カンマから開始し、スペースインデントや行末コメント（`;`）は記述しないでください。

```csv
COMMAND, "SAMPLE_MELSEC_RES_P5S_R", ""
; 形式5でD100からの2ワード読出応答を受信し、10進数値（カンマ区切り）でセル格納
,SEQUENCE, 010, Melsec_P5s_S, "00FF04010000A80000640002"
,SEQUENCE, 020, Melsec_Res_P5s_R, "00", "PLC_D100", "DEC"
```

### 実行時の挙動・変数変化

* **実行前の状態**: Term 通信エンジンが初期化され、必要な通信パラメータおよび変数プール（`PARAM`）が準備されている状態。
* **実行後の状態**: ステップ実行により `Melsec_Res_P5s_R` の通信制御・プロトコル処理が正常に遂行され、結果または回線状態が更新されます。

---

## 🔄 動作フロー（シーケンス図）

```mermaid
sequenceDiagram
    autonumber
    participant Script as "スクリプト実行エンジン"
    participant Cmd as "Melsec_Res_P5s_RCommand"
    participant Buffer as "受信リングバッファ (GetReceivedBytes)"
    participant Helper as "MelsecBinaryProtocolHelper"

    Script->>Cmd: コマンド実行要求
    Cmd->>Buffer: 生受信バイト列取得
    Cmd->>Helper: TryParseResponseFrame (ACK/NAK/STX..ETX)
    Helper-->>Cmd: ペイロード ＆ 終了コード検証結果
    Cmd->>Cmd: 基数変換処理 (HEX / DEC ワード変換)
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

本コマンドクラス（`Melsec_Res_P5s_RCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー**: `RunUnitTest()` メソッドの呼び出し
* **有効化条件**: システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること（それ以外の場合は無効化ガードにより早期リターン）
* **検証結果出力**: `_log.LogTest` によるログ出力。検証失敗時は `_log.LogError`、予期せぬ例外発生時は `_log.LogCritical` で記録し、`finally` ブロックで `testSbuf?.Clear()`、`TermRingBufferManager.Instance.ClearBuffer()`、および `UartTransportCore.Shared.IsLoopbackMode` の復元を実行します。

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | `IsLoopbackMode = true` 環境下で引数 `"VAL_DEC"`, `"DEC"` および擬似バイナリ応答（本文 `00 FF 00 00 64 00 C8 00` を `BuildRequestFrame` で構築）を与えて実行 | `!param.HasError` かつ `param.IsCommandGo == true`、格納先 `SBuf["VAL_DEC"]` が `"100,200"` と一致する | 形式 5 バイナリ応答の終了コード検証およびリトルエンディアン 10 進（`DEC`）変換格納の検証 |
| **2** | 異常系 | 不正フレームおよび引数不足（`RecData = "00"`、引数 0 個の `TermSequence`）、ならびに `Func(null, null)` を指定して実行 | `ValidateParamRange(1, 4)` により `!invalidParam.IsCommandGo` で安全に拒否され、`null` 入力でもクラッシュしない | 不正バイナリフレーム・必須引数欠落および `null` パラメータに対する安全防御検証 |
