# コマンドリファレンス：Melsec_P1s_R

## 概要

**Melsec_P1s_R** は、三菱電機 MELSEC 計算機リンク **専用プロトコル形式 1（サムチェック付き、CR/LF なし）** において、外部マスターまたはシーケンサから送信される要求電文（`ENQ [0x05]` 開始フレーム）を受信・解析し、局番・コマンド・データ長の検証および 2 桁 ASCII サムチェックの照合を行った上で、データ部を指定ターゲット変数へ格納するコマンドです。

> 💡 **補足**: シーケンサシミュレータ（スレーブ応答側）として要求電文を待ち受ける場合や、形式 1 フレームを直接受信解析する場合に使用します。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], Melsec_P1s_R, "<StationNo>", "<Command>", "<DataLength>", "<Target>"
```

### パラメータ（引数）一覧

| 引数位置 | パラメータ名 | 必須/任意 | 型・制約 | 説明 |
| :---: | :--- | :---: | :--- | :--- |
| 第1引数 | `StationNo` | **必須** | 2桁16進文字列（例: `"00"`, `"01"`, `"FF"`） | 受信を期待する局番（Station No.）＋PC番号（通常 `"00FF"` または局番 `"00"`）を指定します。 |
| 第2引数 | `Command` | **必須** | 2文字コマンド文字列（例: `"BR"`, `"WR"`, `"BW"`, `"WW"`） | 受信を期待する計算機リンクコマンド名を指定します。 |
| 第3引数 | `DataLength` | **必須** | 整数（文字数） | コマンドおよび伝送待ち時間フィールドに続くデータ部のバイト（文字）長を指定します。 |
| 第4引数 | `Target` | **必須** | 変数名（`$` / `@`） | 受信フレームから抽出したデータ部文字列を格納するターゲット変数を指定します。 |

---

## 🔄 特殊置換規約・分岐規約の適用

本コマンドの引数における特殊記法および解釈ルールです。
* **動的置換（`$`）**: 引数内で `$PARAMキー$` のように記述すると、実行時に変数プール（PARAM）の値へ動的置換されます。（適用可能な引数: 全引数対応）
* **ジャンプ先指定（`#`）**: 本コマンドはジャンプ制御を行わないため適用されません。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）**: 本コマンドは外部マクロを呼び出さないため、共通コマンド呼び出し記法は適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ**: 通信条件文字列（例: `"9600,E,7,1"`）や複数データのカンマ区切り値を単一引数として渡す場合は、必ず `"..."` と二重引用符で囲んで記述します。
* **エイリアス（別名）**: `P1s_R`, `Analysis_P1s_R`, `Sqn_Com_P1s_R`, `Analysis_SQN_COM_P1S_R`

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※COMMAND ブロック配下の子要素行（PARAM, SEQUENCE）は必ず行頭カンマから開始し、スペースインデントや行末コメント（`;`）は記述しないでください。

```csv
COMMAND, "SAMPLE_MELSEC_P1S_R", ""
,PARAM, "RECV_PAYLOAD", Text, "RECV_PAYLOAD", ""
,SEQUENCE, 010, Melsec_Mode, "9600,O,7,1", "NONE", "P1"
,SEQUENCE, 020, Melsec_Open
; 局番00、ワード一括書込コマンド(WW)、データ部12文字の形式1フレームを受信し $RECV_PAYLOAD へ格納
,SEQUENCE, 030, Melsec_P1s_R, "00", "WW", "12", "$RECV_PAYLOAD$"
,SEQUENCE, 040, Melsec_Close
,SEQUENCE, 050, Exit
```

### 実行時の挙動・変数変化

* **実行前の状態**: Term 通信エンジンが初期化され、必要な通信パラメータおよび変数プール（`PARAM`）が準備されている状態。
* **実行後の状態**: 受信リングバッファから `ENQ (0x05)` + `"00FFWW0D0100011234"` + `Sum(2桁)` のフレームが到着するのを待機します。 サムチェック 2 桁が一致し、局番 `"00"`・コマンド `"WW"` が一致すると、データ部 `"D0100011234"`（または伝送待ち時間を含む指定長文字列）が `$RECV_PAYLOAD` に格納されます。

---

## 🔄 動作フロー（シーケンス図）

```mermaid
sequenceDiagram
    autonumber
    participant Script as "スクリプト実行エンジン (SequenceRunner)"
    participant Cmd as "Melsec_P1s_RCommand (TermAnalysis.Melco)"
    participant Buf as "TermRingBufferManager (受信バッファ)"
    participant Var as "VariableManager (変数管理)"

    Script->>Cmd: Execute(context, stationNo, command, dataLength, target)
    Cmd->>Buf: 形式1フレーム受信待機 (ENQ [0x05] + 局番 + PC番号 + Cmd + Wait + Data + Sum2桁)
    Buf-->>Cmd: 受信バイト列返却
    Cmd->>Cmd: 局番・コマンド一致確認 ＆ ASCIIサムチェック(下位1バイト2桁HEX)照合
    alt サムチェック一致・フォーマット正常
        Cmd->>Var: Target ($ / @) へデータ部文字列を格納
        Cmd-->>Script: 正常終了 (Next Step)
    else サムチェック不一致 または タイムアウト
        Cmd-->>Script: MelsecProtocolException / TimeoutException 送出・停止
    end
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

1. **サムチェック（Sum Check）の厳格検証**:
   - `ENQ` の次バイトからデータ部末尾までの ASCII コード総和の下位 1 バイト（16 進 2 文字）と受信サム値を照合し、不一致の場合は変数を更新せずプロトコルエラーを送出します。
2. **受信タイムアウト保護**:
   - 規定時間内に指定長の形式 1 フレームが揃わない場合は `TimeoutException` を送出して無限待機を防止します。

---

## 🧪 ユニットテスト仕様

本コマンドクラス（`Melsec_P1s_RCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー**: `RunUnitTest()` メソッドの呼び出し
* **有効化条件**: システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること（それ以外の場合は無効化ガードにより早期リターン）
* **検証結果出力**: `_log.LogTest` によるログ出力。検証失敗時は `_log.LogError`、予期せぬ例外発生時は `_log.LogCritical` で記録し、`finally` ブロックで `testSbuf?.Clear()`、`TermRingBufferManager.Instance.ClearBuffer()`、および `UartTransportCore.Shared.IsLoopbackMode` の復元を実行します。

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | `IsLoopbackMode = true` 環境下で引数 `"00"`, `"FFWW0D0000"`, `"2"`, `"MEL_RCV"` および擬似要求電文 `'\u0005' + "00FFWW0D0000021234ABCD" + SQNSUM(...)` を与えて実行 | `!param.HasError` かつ `param.IsCommandGo == true`、格納先 `SBuf["MEL_RCV"]` が `"0x1234,0xABCD"` と一致する | 形式 1 要求電文のサムチェック照合およびワードデータ抽出格納の検証 |
| **2** | 異常系 | サムチェック不一致フレーム（末尾 `"00"`）および `Func(null, null)` を指定して実行 | サムチェック異常により `"ERROR_SUMCHECK"` が格納されて `!badParam.IsCommandGo` となり、`null` 入力でもクラッシュしない | サムチェック不一致検出および `null` パラメータに対する安全防御検証 |
