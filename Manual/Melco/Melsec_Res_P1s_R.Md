# コマンドリファレンス：Melsec_Res_P1s_R

## 概要

**Melsec_Res_P1s_R** は、`Melsec_P1s_S` で送信した要求電文に対する MELSEC シーケンサからの **専用プロトコル形式 1 応答電文** （データ応答フレーム `STX [0x02] ... ETX [0x03] + Sum2桁`、肯定応答 `ACK [0x06] ...`、または否定応答 `NAK [0x15 / 0x0F] ...`）を受信・解析し、サムチェック照合後に読み出しデータ（または `ACK` / `NAK` ステータス）を指定ターゲット変数へ格納するコマンドです。

> 💡 **補足**: 読み出し要求（`BR` / `WR`）に対する `STX...ETX` データ応答と、書き込み要求（`BW` / `WW`）に対する `ACK` 応答の双方を自動判別して処理します。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], Melsec_Res_P1s_R, [StationNoOrHead], "[TargetOrPoints]", "[LegacyDest]", "[LegacyFormat]"
```

※ `ParameterSpec` は `EnableVariableLength(true)` で定義されていますが、実行時（`Func`）は `ValidateParamRange(1, 4)` により **1〜4 個** の引数を受け付けます（5 個以上指定した場合はエラー停止します）。

### パラメータ（引数）一覧

| 引数位置 | 引数名 | 必須/任意 | 説明・制約 |
| :--- | :--- | :---: | :--- |
| 第 1 引数 | `StationNoOrHead` | **必須** | 2 引数以上の構文時は期待するシーケンサ局番または応答ヘッダ文字列（例: `"00"` または `"00FF"`。2 引数標準構文で 2 文字局番 `"00"` を指定した場合は自動的に `"00FF"` として照合されます）。1 引数構文時は格納先パラメータキー名として扱われます（ヘッダ照合省略）。 |
| 第 2 引数 | `TargetOrPoints` | 任意 | 2 引数標準構文時は抽出データ格納先のパラメータキー名（`"@"` や `"$"` を付けない素のキー名、例: `"D100_HEX"`。データ部は 4 文字ごとの `"0xXXXX,0xYYYY,..."` 形式で格納）、旧 3〜4 引数構文時は 16 進読出点数（非 16 進文字時は `0`＝全ワード自動算出、省略時既定値: `""`）。 |
| 第 3 引数 | `LegacyDest` | 任意 | 旧 3〜4 引数互換構文における格納先レジスタ／パラメータキー名（省略時既定値: `""`）。 |
| 第 4 引数 | `LegacyFormat` | 任意 | 旧 4 引数互換構文（格納先が数値キーで連番格納する場合）における `TermStringUtility.Sscanf` 変換フォーマット（省略時既定値: `""`）。 |

---

## 🔄 特殊置換規約・分岐規約の適用

本コマンドの引数における特殊記法および解釈ルールです。
* **動的置換（`$`）**: 引数内で `$PARAMキー$` のように記述すると、実行時に変数プール（PARAM）の値へ動的置換されます。（適用可能な引数: 全引数対応）
* **ジャンプ先指定（`#`）**: 本コマンドはジャンプ制御を行わないため適用されません。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）**: 本コマンドは外部マクロを呼び出さないため、共通コマンド呼び出し記法は適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ**: 引数自体にカンマを含める場合は、必ず `"..."` と二重引用符で囲んで記述します。
* **エイリアス（別名）**: `Res_P1s_R`, `Analysis_Res_P1s_R`, `Sqn_Res_P1s_R`, `Analysis_SQN_RES_P1S_R`

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※COMMAND ブロック配下の子要素行（PARAM, SEQUENCE）は必ず行頭カンマから開始し、スペースインデントや行末コメント（`;`）は記述しないでください。

```csv
COMMAND, "SAMPLE_MELSEC_RES_P1S_R", "D100_HEX"
,PARAM, "D100_HEX", Work, "D100読出データ", ""
,SEQUENCE, 010, Melsec_Mode, "9600,O,7,1", "NONE", "P1"
,SEQUENCE, 020, Melsec_Open
; 1. 局番00・PC番号FF、D0100から2ワード読出要求(WR)を形式1で送信
,SEQUENCE, 030, Melsec_P1s_S, "00FF", "WR", "00D010002"
; 2. 形式1のSTX...ETX応答電文を受信し、2ワード分のデータを D100_HEX へ格納
,SEQUENCE, 040, Melsec_Res_P1s_R, "00", "D100_HEX"
; 3. 読出データが期待値 "0x03E8,0x07D0" と一致するか判定（不一致時は #NG_PROC# へ分岐）
,SEQUENCE, 050, Judge, "EQU", "$D100_HEX$", "0x03E8,0x07D0", "#NG_PROC#"
,SEQUENCE, 060, Melsec_Close
,SEQUENCE, 070, Exit, "$D100_HEX$", 0
; 照合不一致時のクローズ＆異常終了経路
,SEQUENCE, #NG_PROC#, Melsec_Close
,SEQUENCE, 090, Exit, "NG", -1
```

### 実行時の挙動・変数変化

* **実行前の状態**: Term 通信エンジンが初期化され、変数プール（`PARAM`）に `"D100_HEX"` が定義されている状態。
* **実行後の状態**: シーケンサから `[STX(0x02)] + "00FF" + "03E807D0" + [ETX(0x03)] + [Sum2桁]` が返却されます。`Melsec_Res_P1s_R` は `STX` 直後〜`ETX` までのサムチェック 2 桁および局番ヘッダ `"00FF"` の一致を検証し、4 文字単位で整形した `"0x03E8,0x07D0"` を `"D100_HEX"` に格納します。後続の `Judge` で一致が確認されると回線を閉じて正常終了し、不一致時は `#NG_PROC#` へ分岐してクローズ後に異常終了します。

---

## 🔄 動作フロー（シーケンス図）

```mermaid
sequenceDiagram
    autonumber
    participant Driver as "TermSequenceDriver (実行エンジン)"
    participant Cmd as "Melsec_Res_P1s_RCommand"
    participant Buf as "TermRingBufferManager / PollingHost"
    participant Var as "変数プール (PARAM / SBuf)"

    Driver->>Driver: ライセンス検証 (ValidateStep)
    Driver->>Cmd: Func(parent, param)
    activate Cmd
    Cmd->>Cmd: ValidateParamRange(1, 4) ＆ ResolveResponseArgs
    Cmd->>Buf: ReadResponseFromHostOrBuffer (requireSumCheck: true)
    Buf-->>Cmd: 受信応答文字列
    alt STX (0x02) データ応答 かつ サムチェック・期待ヘッダ一致
        Cmd->>Var: 4文字単位 "0xXXXX,0xYYYY,..." 形式で格納 (IsCommandGo = true)
        Cmd-->>Driver: 正常完了 (次ステップへ進行)
    else ACK (0x06) 肯定応答の場合
        Cmd->>Var: 格納先へ "ACK" を保存 (IsCommandGo = true)
        Cmd-->>Driver: 正常完了 (次ステップへ進行)
    else 引数範囲外・NAK (0x15 / 0x0F)・サム不一致・フレーム異常・タイムアウト
        Cmd->>Var: "NAK" / "ERROR_SUMCHECK" / "ERROR_HEADER" / "ERROR_FRAME" / "ERROR_TIMEOUT" 格納
        Cmd-->>Driver: 異常停止 (IsCommandGo = false / SetErrorAndHalt)
    end
    deactivate Cmd
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

1. **引数個数境界および NAK 否定応答の検出**:
   - 引数が 0 個または 5 個以上指定された場合（`ValidateParamRange(1, 4)` 違反）は `_log.LogError` に記録し `SetErrorAndHalt` で停止します。シーケンサから `NAK`（`0x15` または `0x0F`）が返された場合は、格納先キーへ `"NAK"` を記録し `param.IsCommandGo = false` として異常状態で後続ステップへ進行することを防止します。
2. **サムチェック・フレーム構造・タイムアウト検証**:
   - `STX`〜`ETX` 応答フレームのサムチェック不一致時は `"ERROR_SUMCHECK"`、期待ヘッダ不一致時は `"ERROR_HEADER"`、制御文字位置や長さ異常時は `"ERROR_FRAME"`、無応答時は `"ERROR_TIMEOUT"` を格納先に記録して `param.IsCommandGo = false` で停止します（例外発生時は `_log.LogCritical` と `SetErrorAndHalt(-1, ex.Message)`）。

---

## 🧪 ユニットテスト仕様

本コマンドクラス（`Melsec_Res_P1s_RCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー**: `RunUnitTest()` メソッドの呼び出し
* **有効化条件**: システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること（それ以外の場合は無効化ガードにより早期リターン）
* **検証結果出力**: `_log.LogTest` によるログ出力。検証失敗時は `_log.LogError`、予期せぬ例外発生時は `_log.LogCritical` で記録し、`finally` ブロックで `testSbuf?.Clear()`、`TermRingBufferManager.Instance.ClearBuffer()`、および `UartTransportCore.Shared.IsLoopbackMode` の復元を実行します。

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | `IsLoopbackMode = true` 環境下で引数 `"00"`, `"RES_DATA"` および擬似応答 `'\u0002' + "00FF01000200" + '\u0003' + SQNSUM(...)` を与えて実行 | `!param.HasError` かつ `param.IsCommandGo == true`、格納先 `SBuf["RES_DATA"]` が `"0x0100,0x0200"` と一致する | 形式 1 `STX`〜`ETX` 応答のサムチェック照合およびデータ抽出格納の検証 |
| **2** | 異常系 | `NAK` 否定応答フレーム `"\u001500FF55"` を指定した `Func(null, nakParam)` および `Func(null, null)` を実行 | `NAK` 検出により `"NAK"` を格納して `nakParam.HasError \|\| !nakParam.IsCommandGo` となり、`null` 入力でも例外クラッシュしない | `NAK` エラー応答検出および `null` パラメータに対する安全防御検証 |
