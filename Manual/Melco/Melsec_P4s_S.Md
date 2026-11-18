# コマンドリファレンス：Melsec_P4s_S

## 概要

**Melsec_P4s_S** は、三菱電機 MELSEC 計算機リンク **専用プロトコル形式 4（サムチェック付き、末尾 `CR+LF` あり）** の要求電文フレームを自動構築し、先頭制御コード（`ENQ [0x05]`）、2 桁 ASCII サムチェックコード、および末尾の `CR+LF`（`0x0D 0x0A`）を自動付与してシリアルポートへ送信するコマンドです。

> 💡 **補足**: 形式 4 設定の MELSEC 計算機リンクユニットに対してデバイス読み出し（`BR` / `WR`）やデバイス書き込み（`BW` / `WW`）を要求する際に使用します。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], Melsec_P4s_S, "<StationNo>", "<Command>", "<Data>"
```

### パラメータ（引数）一覧

| 引数位置 | パラメータ名 | 必須/任意 | 型・制約 | 説明 |
| :---: | :--- | :---: | :--- | :--- |
| 第1引数 | `StationNo` | **必須** | 2桁または4桁16進文字列（例: `"00"`, `"00FF"`） | 送信先の局番（2桁指定時は自局 PC 番号 `"FF"` が自動補完されます）を指定します。 |
| 第2引数 | `Command` | **必須** | 2文字コマンド文字列（例: `"BR"`, `"WR"`, `"BW"`, `"WW"`） | 計算機リンクコマンド名を指定します。 |
| 第3引数 | `Data` | **必須** | 文字列（伝送待ち時間＋先頭デバイス＋点数＋書込値） | コマンドに続くデータ部（例: `"0D010001"`）を指定します。 |

---

## 🔄 特殊置換規約・分岐規約の適用

本コマンドの引数における特殊記法および解釈ルールです。
* **動的置換（`$`）**: 引数内で `$PARAMキー$` のように記述すると、実行時に変数プール（PARAM）の値へ動的置換されます。（適用可能な引数: 全引数対応）
* **ジャンプ先指定（`#`）**: 本コマンドはジャンプ制御を行わないため適用されません。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）**: 本コマンドは外部マクロを呼び出さないため、共通コマンド呼び出し記法は適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ**: 通信条件文字列（例: `"9600,E,7,1"`）や複数データのカンマ区切り値を単一引数として渡す場合は、必ず `"..."` と二重引用符で囲んで記述します。
* **エイリアス（別名）**: `P4s_S`, `Analysis_P4s_S`, `Sqn_Com_P4s_S`, `Analysis_SQN_COM_P4S_S`

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※COMMAND ブロック配下の子要素行（PARAM, SEQUENCE）は必ず行頭カンマから開始し、スペースインデントや行末コメント（`;`）は記述しないでください。

```csv
COMMAND, "SAMPLE_MELSEC_P4S_S", ""
,PARAM, "D_ADDR", Text, "D_ADDR", "D0100"
,PARAM, "W_VAL", Text, "W_VAL", "0064"
,SEQUENCE, 010, Melsec_Mode, "9600,O,7,1", "NONE", "P4"
,SEQUENCE, 020, Melsec_Open
; 1. 局番00、D0100へ1ワード(0064h = 100)書込要求(WW)を形式4で送信
,SEQUENCE, 030, Melsec_P4s_S, "00", "FFWW", "0$D_ADDR$01$W_VAL$"
,SEQUENCE, 040, Melsec_Res_P4s_R, "00", "@ACK_RES"
; 2. 局番00、D0100から1ワード読出要求(WR)を形式4で送信し、読出値を確認
,SEQUENCE, 050, Melsec_P4s_S, "00", "FFWR", "0$D_ADDR$01"
,SEQUENCE, 060, Melsec_Res_P4s_R, "00", "@READ_VAL"
,SEQUENCE, 070, Melsec_Close
,SEQUENCE, 080, Exit
```

### 実行時の挙動・変数変化

* **実行前の状態**: Term 通信エンジンが初期化され、必要な通信パラメータおよび変数プール（`PARAM`）が準備されている状態。
* **実行後の状態**: 1 回目の `Melsec_P4s_S` では、`[ENQ(0x05)] + "00FFWW0D0100010064" + [Sum2桁] + [CR][LF]` が構築・送信されます。 2 回目の `Melsec_P4s_S` では、`[ENQ(0x05)] + "00FFWR0D010001" + [Sum2桁] + [CR][LF]` が送信されます。

---

## 🔄 動作フロー（シーケンス図）

```mermaid
sequenceDiagram
    autonumber
    participant Script as "スクリプト実行エンジン (SequenceRunner)"
    participant Cmd as "Melsec_P4s_SCommand (TermAnalysis.Melco)"
    participant Framing as "MelsecFrameEngine (形式4)"
    participant Port as "SerialPort / MELSEC ドライバ"

    Script->>Cmd: Execute(context, stationNo, command, data)
    Cmd->>Cmd: 引数の変数展開 ($ / @ / #)
    Cmd->>Framing: BuildP4Frame(stationNo, command, data)
    Framing->>Framing: PC番号("FF")補完 ＆ ASCIIサムチェック2桁算出
    Framing-->>Cmd: [0x05(ENQ)] + Body + SumHex(2byte) + [0x0D(CR), 0x0A(LF)]
    Cmd->>Port: Write(frameBytes)
    Cmd-->>Script: 正常終了 (Next Step)
    alt パラメータおよびプロトコル検証が正常な場合
        Cmd-->>Script: 正常完了 (IsCommandGo = true)
    else 引数範囲外または通信状態異常の場合
        Cmd->>Cmd: エラーログ出力 (_log.LogError)
        Cmd-->>Script: 安全停止 (SetErrorAndHalt)
    end
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

1. **回線未オープン時の送信防止**:
   - `Melsec_Open` が実行されていない状態で呼び出された場合、`InvalidOperationException` を送出して停止します。
2. **不正な局番・コマンド長の検証**:
   - 局番が 16 進表記でない場合やコマンド名が 2 文字でない場合は、`ArgumentException` を送出して不正電文の送出を防止します。

---

## 🧪 ユニットテスト仕様

本コマンドクラス（`Melsec_P4s_SCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー**: `RunUnitTest()` メソッドの呼び出し
* **有効化条件**: システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること（それ以外の場合は無効化ガードにより早期リターン）
* **検証結果出力**: `_log.LogTest` によるログ出力。検証失敗時は `_log.LogError`、予期せぬ例外発生時は `_log.LogCritical` で記録し、`finally` ブロックで `UartTransportCore.Shared.IsSimulationMode` を開始前の状態へ復元します。

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | `IsSimulationMode = true` 環境下で 3 引数 `"00"`, `"FFWR0"`, `"D001002"` を指定して実行 | `!param.HasError` かつ `param.SndData` が `'\u0005' + "00FFWR0D001002" + SQNSUM("00FFWR0D001002") + "\r\n"` と一致する | 形式 4（`ENQ`＋本体＋2 桁サムチェック＋`CR+LF`）電文構築および送信の検証 |
| **2** | 異常系 | 必須引数欠落・引数 9 個以上の過剰指定（`ValidateParamRange(1, 8)` 違反）、または送信例外発生 | 引数検証エラーによる早期リターン、または `_log.LogCritical` 記録と `SetErrorAndHalt(-1, ex.Message)` による安全停止 | 引数境界チェックおよび送信異常時のクラッシュ防御検証 |
