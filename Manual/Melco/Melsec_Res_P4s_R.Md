# コマンドリファレンス：Melsec_Res_P4s_R

## 概要

**Melsec_Res_P4s_R** は、`Melsec_P4s_S` で送信した要求電文に対する MELSEC シーケンサからの **専用プロトコル形式 4 応答電文** （`STX [0x02] ... ETX [0x03] + Sum2桁 + CR + LF`、`ACK [0x06] ... + CR + LF`、または `NAK [0x15] ... + CR + LF`）を受信・解析し、サムチェックおよび末尾 `CR+LF` デリミタ照合後に読み出しデータ（または ACK/NAK ステータス）を指定ターゲット変数へ格納するコマンドです。

> 💡 **補足**: プロトコル形式 4 の応答電文は、すべての応答フレーム（`STX` / `ACK` / `NAK`）の末尾に `CR+LF`（`0x0D 0x0A`）が付与されています。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], Melsec_Res_P4s_R, [StationNoOrHead], "[TargetOrPoints]", "[LegacyDest]", "[LegacyFormat]"
```

### パラメータ（引数）一覧

| 引数位置 | 引数名 | 必須/任意 | 説明・制約 |
| :--- | :--- | :---: | :--- |
| 第 1 引数 | `StationNoOrHead` | **必須** | 期待するシーケンサ局番または応答ヘッダ文字列（例: `"00"` または `"00FF"`）。 |
| 第 2 引数 | `TargetOrPoints` | 任意 | 2 引数標準構文時は抽出データ格納先の変数キー名、旧構文時は読出点数（省略時既定値: `""`）。 |
| 第 3 引数 | `LegacyDest` | 任意 | 旧 3〜4 引数互換構文における格納先レジスタ／変数キー名（省略時既定値: `""`）。 |
| 第 4 引数 | `LegacyFormat` | 任意 | 取得データの基数変換フォーマット（`"HEX"` / `"DEC"` 等、省略時既定値: `""`）。 |

---

## 🔄 特殊置換規約・分岐規約の適用

本コマンドの引数における特殊記法および解釈ルールです。
* **動的置換（`$`）**: 引数内で `$PARAMキー$` のように記述すると、実行時に変数プール（PARAM）の値へ動的置換されます。（適用可能な引数: 全引数対応）
* **ジャンプ先指定（`#`）**: 本コマンドはジャンプ制御を行わないため適用されません。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）**: 本コマンドは外部マクロを呼び出さないため、共通コマンド呼び出し記法は適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ**: 通信条件文字列（例: `"9600,E,7,1"`）や複数データのカンマ区切り値を単一引数として渡す場合は、必ず `"..."` と二重引用符で囲んで記述します。
* **エイリアス（別名）**: `Res_P4s_R`, `Analysis_Res_P4s_R`, `Sqn_Res_P4s_R`, `Analysis_SQN_RES_P4S_R`

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※COMMAND ブロック配下の子要素行（PARAM, SEQUENCE）は必ず行頭カンマから開始し、スペースインデントや行末コメント（`;`）は記述しないでください。

```csv
COMMAND, "SAMPLE_MELSEC_RES_P4S_R", ""
,PARAM, "D200_VAL", Text, "D200_VAL", ""
,SEQUENCE, 010, Melsec_Mode, "9600,O,7,1", "NONE", "P4"
,SEQUENCE, 020, Melsec_Open
; 1. D0200から1ワード読出要求(WR)を形式4で送信
,SEQUENCE, 030, Melsec_P4s_S, "00", "WR", "0D020001"
; 2. 形式4の応答電文(STX...ETX+Sum+CRLF)を受信し、データ部を $D200_VAL へ格納
,SEQUENCE, 040, Melsec_Res_P4s_R, "00", "$D200_VAL$"
,SEQUENCE, 050, Judge, "$D200_VAL$", "==", "0064", "MATCH_OK"
,SEQUENCE, 060, Melsec_Close
,SEQUENCE, 070, Exit
,SEQUENCE, #MATCH_OK#, Msg_Box, "D0200の値が期待値(100)と一致しました"
,SEQUENCE, 080, Melsec_Close
,SEQUENCE, 090, Exit
```

### 実行時の挙動・変数変化

* **実行前の状態**: Term 通信エンジンが初期化され、必要な通信パラメータおよび変数プール（`PARAM`）が準備されている状態。
* **実行後の状態**: シーケンサから `[STX(0x02)] + "00FF0064" + [ETX(0x03)] + [Sum2桁] + [CR][LF]` が返却されます。 `Melsec_Res_P4s_R` は末尾 `CR+LF`・サムチェック 2 桁・局番 `"00FF"` を検証し、データ部 `"0064"` を `$D200_VAL` に格納します。

---

## 🔄 動作フロー（シーケンス図）

```mermaid
sequenceDiagram
    autonumber
    participant Script as "スクリプト実行エンジン (SequenceRunner)"
    participant Cmd as "Melsec_Res_P4s_RCommand (TermAnalysis.Melco)"
    participant Buf as "TermRingBufferManager (受信バッファ)"
    participant Var as "VariableManager (変数管理)"

    Script->>Cmd: Execute(context, stationNo, target)
    Cmd->>Buf: 形式4応答フレーム待機 (STX/ACK/NAK 〜 CR[0x0D]+LF[0x0A] 終端)
    Buf-->>Cmd: 受信行バイト列
    alt STX (0x02) データ応答フレームの場合
        Cmd->>Cmd: CR+LF確認 ＆ ETX(0x03)直後の2桁サムチェック照合 ＆ 局番確認
        Cmd->>Var: Target へデータ部文字列を格納
        Cmd-->>Script: 正常終了 (Next Step)
    else ACK (0x06) 肯定応答フレームの場合
        Cmd->>Cmd: 局番確認 ("00FF") ＆ CR+LF確認
        Cmd->>Var: Target へ "ACK" を格納
        Cmd-->>Script: 正常終了 (Next Step)
    else NAK (0x15) 否定応答 または サムチェック不一致
        Cmd-->>Script: エラーコード解読・MelsecProtocolException 送出
    end
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

1. **NAK（`0x15`）エラーコードの解読と通知**:
   - シーケンサから `NAK (0x15) + 局番 + PC番号 + エラーコード(2桁) + CR + LF` が返された場合、エラーコードを解読して例外を送出します。
2. **サムチェックおよび `CR+LF` 終端の検証**:
   - サムチェック不一致または末尾 `CR+LF` 欠落時は通信エラーとして処理し、不正データの変数格納を阻止します。

---

## 🧪 ユニットテスト仕様

本コマンドクラス（`Melsec_Res_P4s_RCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー**: `RunUnitTest()` メソッドの呼び出し
* **有効化条件**: システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること（それ以外の場合は無効化ガードにより早期リターン）
* **検証結果出力**: `_log.LogTest` によるログ出力。検証失敗時は `_log.LogError`、予期せぬ例外発生時は `_log.LogCritical` で記録し、`finally` ブロックで `testSbuf?.Clear()`、`TermRingBufferManager.Instance.ClearBuffer()`、および `UartTransportCore.Shared.IsLoopbackMode` の復元を実行します。

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | `IsLoopbackMode = true` 環境下で引数 `"00"`, `"RES_P4_DATA"` および擬似応答 `'\u0002' + "00FF00AA00BB" + '\u0003' + SQNSUM(...) + "\r\n"` を与えて実行 | `!param.HasError` かつ `param.IsCommandGo == true`、格納先 `SBuf["RES_P4_DATA"]` が `"0x00AA,0x00BB"` と一致する | 形式 4 `STX`〜`ETX`＋サムチェック＋`CR+LF` 応答電文の解析および格納検証 |
| **2** | 異常系 | サムチェック不一致応答フレーム（`'\u0002' + sumTarget + "00\r\n"`）および `Func(null, null)` を指定して実行 | サムチェック異常により `"ERROR_SUMCHECK"` が格納されて `!badParam.IsCommandGo` となり、`null` 入力でもクラッシュしない | サムチェック不一致検出および `null` パラメータに対する安全防御検証 |
