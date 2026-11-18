# シーケンサデータ収集用 マクロスクリプト自動生成 AI指示書 (Melco_P5.csv 生成用)

> **生成AI**: Antigravity (Gemini 4 Carbon (Medium))

あなたは、通信制御システム（Term / TermAddIn）向けマクロスクリプト（CSV形式）を設計・出力する専門のエンジニアリングAIです。  
以下の「参照ドキュメント」および「システム絶対制約」「電文フレーム完全機能別パラメータ設計仕様」を厳格に遵守し、三菱電機 MELSEC シーケンサから計算機リンク **形式5（バイナリコード通信方式：DLEフレーミング＋2バイトチェックサム）** で各種データを高速収集・制御するための Term マクロスクリプト（`Melco_P5.csv`）を作成してください。

---

## 📚 1. 参照ドキュメントおよび準拠規格

1. **システム文法・構文規約（最優先遵守）**
   * `Manual/TermAddIn/マクロスクリプト取扱説明書.Md`（Term マクロスクリプト取扱説明書 兼 AI自動生成指示書）
2. **通信・プロトコル個別仕様（Melco 専用コマンドセット）**
   * `Manual/TermAddIn/Melco/Melsec_Mode.Md`（MELSEC 計算機リンク通信条件・プロトコル形式設定）
   * `Manual/TermAddIn/Melco/Melsec_Open.Md` / `Manual/TermAddIn/Melco/Melsec_Close.Md`（MELSEC 通信回線接続・切断）
   * `Manual/TermAddIn/Melco/Melsec_P5s_S.Md`（MELSEC プロトコル5 バイナリコマンド送信）
   * `Manual/TermAddIn/Melco/Melsec_Res_P5s_R.Md`（MELSEC プロトコル5 バイナリ応答受信）
   * `Manual/Core/Sequence/Judge.Md`（比較判定・条件分岐制御）
   * `Manual/Core/Sequence/Jump.Md` / `Manual/Core/Param/Exit.Md`（無条件ジャンプ・終了制御）
3. **対象通信仕様**
   * 制御対象: 三菱電機 MELSEC シーケンサ（計算機リンク 形式5 バイナリコード通信方式）
   * 通信パラメータ: ボーレート 57600 bps、パリティ Odd (O)、データ長 8 bit、ストップビット 1 bit、フロー制御 NONE
   * 伝送制御: プロトコル形式 `P5`（`Melsec_Mode` にて設定：BIN バイナリ透過受信モード自動適用、DLE+STX / DLEエスケープ / DLE+ETX、2バイト加算チェックサム自動計算・照合）

---

## 🚨 2. システム絶対制約（構文エラー・実行時クラッシュ防止の鉄則）

1. **表記言語・文字コード・AI生成元の明記**
   * **表記言語**: 多言語化を意識した構成とし、スクリプト内のコメント、パラメータ表示名、メッセージ等は、特に指定がない限り **日本語をデフォルトとして記述・設定** してください。
   * **文字コード**: 生成する CSV ファイル（`Melco_P5.csv`）は、日本語 Excel での文字化け防止のため **Shift-JIS (CP932)** で保存してください。
   * **AI生成元の明記（他者再現性の保証）**: CSV ファイルの最先頭ヘッダコメント行（`;`）に、対象システム情報とともに **他者が同一条件で再現できるよう、実行環境（エージェント／IDE名）および自身のAIエンジン（モデル名・エディション／ティア）を正確に判別できる名称（例: `; 生成AI: Antigravity (Gemini 4 Carbon (Medium))`）** を必ず記述してください（エージェント名のみの記載や、推測・例示の丸写しによる不正確なモデル名の記載は厳禁）。
   * **パス表記**: ファイル内に開発環境固有のローカル絶対パスや個人名を含めることは厳禁です。必ず相対パスまたは抽象表記のみで記述してください。

2. **バイナリ電文ペイロードの完全機能別パラメータ分離と `List` 選択化（最重要仕様）**
   * 形式5のバイナリ要求ペイロード（`DLE+STX` 〜 `DLE+ETX` 間のバイト列の 16 進表現）を一括りの文字列として扱わず、MELSEC バイナリ規格（リトルエンディアン）に準拠して各機能要素へ完全に分離して `PARAM` 定義してください。また、 **コマンド・サブコマンド・デバイスコード・通信条件など、プロトコル規格により取り得る文字列が決まっている項目は必ず `List` 型で定義** し、ドロップダウンから選択できるようにしてください。
     * **局番号 (`STATION_NO`)**: 通信相手局番 1バイト（`Text`, `"00"`）
     * **PC番号 (`PC_NO`)**: 自局CPU PC番号 1バイト（`Text`, `"FF"`）
     * **コマンド (`CMD_READ`, `CMD_WRITE`)**: 読出 LE 2バイト（`List`, `"0104"`, `"0304"`）/ 書込 LE 2バイト（`List`, `"0114"`, `"0214"`）
     * **サブコマンド (`SUBCMD`)**: アクセス単位指定 LE 2バイト（`List`, `"0000"`, `"0100"`, `"8000"`, `"8100"`）
     * **デバイスコード (`DEV_CODE`)**: バイナリデバイスコード 1バイト（`List`, 初期値 `"B0"`［`ZR`］, 追加選択肢 `"A8"`［`D`］, `"B4"`［`W`］, `"AF"`［`R`］, `"A9"`［`SD`］, `"B5"`［`SW`］, `"C2"`［`TN`］, `"C5"`［`CN`］, `"90"`［`M`］, `"92"`［`L`］, `"93"`［`F`］, `"94"`［`V`］, `"A0"`［`B`］, `"9C"`［`X`］, `"9D"`［`Y`］, `"C1"`［`TS`］, `"C4"`［`CS`］）
     * **デバイスアドレス (`DEV_ADDR`, `ADDR_STATUS`, `ADDR_CTRL`)**: 先頭デバイス番号 3バイトリトルエンディアン（`Text`, `0x007FFE` → `"FE7F00"`, `0x007FFF` → `"FF7F00"`, `0x008020` → `"208000"`）
     * **アクセス点数 (`POINTS`)**: 読出・書込点数 2バイトリトルエンディアン（`Text`, 1点 `0x0001` → `"0100"`）
     * **設定値・書込値 (`VAL_INIT_CTRL`, `VAL_RECOVERY`, `VAL_FINISH_1`, `VAL_RESET`)**: 設定ワード値 2バイトリトルエンディアン（`Text`, `0x0004` → `"0400"`, `0x0002` → `"0200"`, `0x0001` → `"0100"`, `0x0000` → `"0000"`）
   * 送信電文は、`$STATION_NO$$PC_NO$$CMD_READ$$SUBCMD$$DEV_CODE$...` のようにパラメータ動的置換記法（`$`）を連結して構成してください（`DLE+STX`、データ中の `0x10` 二重化エスケープ、`DLE+ETX`、および 2バイトチェックサムは `Melsec_P5s_S` が自動付与します）。

3. **行頭判定と「第2フィールド」の鉄則**
   * **親ブロック（`TITLE`, `COMMAND`）**: 必ず行頭の1文字目（第1フィールド）から記述してください。
   * **子要素（`PARAM`, `SEQUENCE`）**: 必ず **行頭をカンマ `,`（第1フィールドを空）** から開始し、第2フィールドにキーワードを記述してください。スペースやタブによるインデントは一切禁止です。

4. **PARAM 定義における命名規則および「表示名」「初期値」の省略完全禁止**
   * 書式: `,PARAM, [パラメータキー], [データ型], [表示名], [初期値], [追加選択肢...]`
   * データ型は `Text`, `List`, `Range`, `Value`（または `Number`）, `Label`, `Work` の6種類のみ使用可能です。
   * **いかなるデータ型であっても、第5フィールド（表示名）および第6フィールド（初期値）の省略は絶対禁止** です（4フィールド以下で記述すると `IndexOutOfRangeException` が発生します）。初期値が空の場合は必ず `""` を明記してください。

5. **COMMAND ブロック配下での行末コメント禁止**
   * `COMMAND` ブロック配下の `PARAM` 行および `SEQUENCE` 行では **行末コメント（`; ...`）は完全禁止** です。コメントは必ず独立した行（行頭 `;`）として記述してください。

6. **Melco 専用コマンドセットの厳守（コマンドセット横断の完全禁止）**
   * 本スクリプトは `TermAnalysis.Melco` 用であるため、通信制御には必ず **Melco 用コマンドセット（`Melsec_Mode`, `Melsec_Open`, `Melsec_Close`, `Melsec_P5s_S`, `Melsec_Res_P5s_R` 等）** および共通基盤コマンド（`Judge`, `Jump`, `Exit` 等）のみを使用してください。
   * 他通信ドメインのコマンドセット（`Com_Mode`, `Com_Open`, `Com_Close`, `Telnet_*`, `Modbus_*`, `Mc_*` 等）を跨いで混在させることは排他制御違反となるため **絶対禁止** です。

---

## 📡 3. 対象システム仕様および TITLE ブロック定義

シーケンサデータ収集システム（形式5・バイナリ通信）向けとして、`TITLE` ブロックを以下の通り定義してください。

* **タイトル名**: `"シーケンサデータ収集 (形式5)"`
* **書式**:
  ```csv
  TITLE, "シーケンサデータ収集 (形式5)"
  ```

---

## 📐 4. 実装すべき COMMAND ブロックと完全機能別パラメータ設計仕様

以下の 6 つの独立した `COMMAND` ブロックを実装してください。すべての送受信ステップで機能別分離パラメータを使用します。

### ① 通信ポート接続：`COMMAND, "PORT_OPEN"`
* **目的**: MELSEC 計算機リンク通信条件（ボーレート・パリティ・データ長・ストップビット・フロー制御）を `List` パラメータから選択設定し、プロトコル形式 `P5`（BINバイナリ透過モード自動適用）で MELSEC 通信回線を接続オープンする。
* **パラメータ定義（`List` 選択式）**:
  * `BAUD` (`List`, `"ボーレート"`, `"57600"`, `"9600"`, `"19200"`, `"38400"`, `"115200"`)
  * `PARITY` (`List`, `"パリティ"`, `"O"`, `"E"`, `"N"`)
  * `DATA_BITS` (`List`, `"データ長"`, `"8"`, `"7"`)
  * `STOP_BITS` (`List`, `"ストップビット"`, `"1"`, `"2"`)
  * `FLOW_CTRL` (`List`, `"フロー制御"`, `"NONE"`, `"RTSCTS"`, `"XONXOFF"`)
* **シーケンス手順**:
  1. ステップ 10: `Melsec_Mode, "$BAUD$,$PARITY$,$DATA_BITS$,$STOP_BITS$", "$FLOW_CTRL$", "P5"`
  2. ステップ 20: `Melsec_Open`
  3. ステップ 30: `Exit`

### ② 読み出し開始・初期確認：`COMMAND, "READ_START", ON`
* **目的**: シーケンサとの初期ハンドシェイクを行い、指定デバイスの状態確認（`0x0000`）および制御コマンド送信に対する `DLE ACK` 応答を検証する。
* **パラメータ定義（完全機能別分離）**:
  * `STATION_NO` (`Text`, `"局番号"`, `"00"`)
  * `PC_NO` (`Text`, `"PC番号"`, `"FF"`)
  * `CMD_READ` (`List`, `"読出コマンド(LE)"`, `"0104"`, `"0304"`)
  * `CMD_WRITE` (`List`, `"書込コマンド(LE)"`, `"0114"`, `"0214"`)
  * `SUBCMD` (`List`, `"サブコマンド(LE)"`, `"0000"`, `"0100"`, `"8000"`, `"8100"`)
  * `DEV_CODE` (`List`, `"デバイスコード"`, `"B0"`, `"A8"`, `"B4"`, `"AF"`, `"A9"`, `"B5"`, `"C2"`, `"C5"`, `"90"`, `"92"`, `"93"`, `"94"`, `"A0"`, `"9C"`, `"9D"`, `"C1"`, `"C4"`)
  * `ADDR_STATUS` (`Text`, `"状態確認アドレス(LE)"`, `"FE7F00"`)
  * `ADDR_CTRL` (`Text`, `"制御コマンドアドレス(LE)"`, `"FF7F00"`)
  * `POINTS` (`Text`, `"アクセス点数(LE)"`, `"0100"`)
  * `VAL_INIT_CTRL` (`Text`, `"初期確認設定値(LE)"`, `"0400"`)
  * `STATUS` (`Work`, `"判定結果"`, `""`)
* **シーケンス手順**:
  1. ステップ 10: `Melsec_P5s_S, "$STATION_NO$$PC_NO$$CMD_READ$$SUBCMD$$DEV_CODE$$ADDR_STATUS$$POINTS$"` （状態読出要求）
  2. ステップ 20: `Melsec_Res_P5s_R, "$STATION_NO$", "STATUS", "DEC"` （状態ワード受信・リトルエンディアン数値変換）
  3. ステップ 30: `Judge, EQU, "$STATUS$", "0x0000", #ERR_EXIT#` （正常値 0 でなければエラー終了）
  4. ステップ 40: `Melsec_P5s_S, "$STATION_NO$$PC_NO$$CMD_WRITE$$SUBCMD$$DEV_CODE$$ADDR_CTRL$$POINTS$$VAL_INIT_CTRL$"` （初期制御値書込）
  5. ステップ 50: `Melsec_Res_P5s_R, "$STATION_NO$", "STATUS"` （DLE ACK 応答確認）
  6. ステップ 60: `Judge, EQU, "$STATUS$", "ACK", #ERR_EXIT#` （ACK でなければエラー終了）
  7. ステップ 70: `Exit` （正常終了）
  8. ステップ `#ERR_EXIT#`: `Exit` （異常終了）

### ③ 数値データ読み出し：`COMMAND, "NUM_READ", "RESULT_DATA"`
* **目的**: パラメータで指定されたデバイスアドレスから数値データを 10 進整数として読み出してセル（`RESULT_DATA`）へ展開し、チェックサム異常（`ERROR_SUMCHECK`）発生時には復旧コマンド送信と再試行を行う。
* **パラメータ定義（完全機能別分離）**:
  * `STATION_NO` (`Text`, `"局番号"`, `"00"`)
  * `PC_NO` (`Text`, `"PC番号"`, `"FF"`)
  * `CMD_READ` (`List`, `"読出コマンド(LE)"`, `"0104"`, `"0304"`)
  * `CMD_WRITE` (`List`, `"書込コマンド(LE)"`, `"0114"`, `"0214"`)
  * `SUBCMD` (`List`, `"サブコマンド(LE)"`, `"0000"`, `"0100"`, `"8000"`, `"8100"`)
  * `DEV_CODE` (`List`, `"デバイスコード"`, `"B0"`, `"A8"`, `"B4"`, `"AF"`, `"A9"`, `"B5"`, `"C2"`, `"C5"`, `"90"`, `"92"`, `"93"`, `"94"`, `"A0"`, `"9C"`, `"9D"`, `"C1"`, `"C4"`)
  * `DEV_ADDR` (`Text`, `"読出デバイス番号(LE)"`, `"208000"`)
  * `POINTS` (`Text`, `"読出点数(LE)"`, `"0100"`)
  * `ADDR_CTRL` (`Text`, `"復旧制御アドレス(LE)"`, `"FF7F00"`)
  * `VAL_RECOVERY` (`Text`, `"復旧制御設定値(LE)"`, `"0200"`)
  * `RESULT_DATA` (`Range`, `"読出データ結果"`, `""`)
* **シーケンス手順**:
  1. ステップ `#RETRY#`: `Melsec_P5s_S, "$STATION_NO$$PC_NO$$CMD_READ$$SUBCMD$$DEV_CODE$$DEV_ADDR$$POINTS$"` （データ読出要求）
  2. ステップ 20: `Melsec_Res_P5s_R, "$STATION_NO$", "RESULT_DATA", "DEC"` （10進ワード受信・結果展開）
  3. ステップ 30: `Judge, NEQ, "$RESULT_DATA$", "ERROR_SUMCHECK", #RETRY_FAIL#` （チェックサム異常時は復旧へ）
  4. ステップ 40: `Exit` （正常終了）
  5. ステップ `#RETRY_FAIL#`: `Melsec_P5s_S, "$STATION_NO$$PC_NO$$CMD_WRITE$$SUBCMD$$DEV_CODE$$ADDR_CTRL$$POINTS$$VAL_RECOVERY$"` （復旧制御値書込）
  6. ステップ 60: `Melsec_Res_P5s_R, "$STATION_NO$", "RESULT_DATA"` （DLE ACK 応答確認）
  7. ステップ 70: `Judge, EQU, "$RESULT_DATA$", "ACK", #RETRY#` （ACK 受信時は再送 `#RETRY#` へ）
  8. ステップ 80: `Exit` （復旧不能時終了）

### ④ レジスタ一括読み出し：`COMMAND, "REG_READ", "RESULT_DATA"`
* **目的**: 指定レジスタ群からバイナリデータを一括読み出しし、16 進ダンプ文字列（`HEX`）として結果変数へ格納する。
* **パラメータ定義（完全機能別分離）**:
  * `STATION_NO` (`Text`, `"局番号"`, `"00"`)
  * `PC_NO` (`Text`, `"PC番号"`, `"FF"`)
  * `CMD_READ` (`List`, `"読出コマンド(LE)"`, `"0104"`, `"0304"`)
  * `CMD_WRITE` (`List`, `"書込コマンド(LE)"`, `"0114"`, `"0214"`)
  * `SUBCMD` (`List`, `"サブコマンド(LE)"`, `"0000"`, `"0100"`, `"8000"`, `"8100"`)
  * `DEV_CODE` (`List`, `"デバイスコード"`, `"B0"`, `"A8"`, `"B4"`, `"AF"`, `"A9"`, `"B5"`, `"C2"`, `"C5"`, `"90"`, `"92"`, `"93"`, `"94"`, `"A0"`, `"9C"`, `"9D"`, `"C1"`, `"C4"`)
  * `DEV_ADDR` (`Text`, `"読出デバイス番号(LE)"`, `"208000"`)
  * `POINTS` (`Text`, `"読出点数(LE)"`, `"0100"`)
  * `ADDR_CTRL` (`Text`, `"復旧制御アドレス(LE)"`, `"FF7F00"`)
  * `VAL_RECOVERY` (`Text`, `"復旧制御設定値(LE)"`, `"0200"`)
  * `RESULT_DATA` (`Range`, `"レジスタデータ"`, `""`)
* **シーケンス手順**:
  1. ステップ `#READ_START#`: `Melsec_P5s_S, "$STATION_NO$$PC_NO$$CMD_READ$$SUBCMD$$DEV_CODE$$DEV_ADDR$$POINTS$"`
  2. ステップ 20: `Melsec_Res_P5s_R, "$STATION_NO$", "RESULT_DATA", "HEX"`
  3. ステップ 30: `Judge, NEQ, "$RESULT_DATA$", "ERROR_SUMCHECK", #ERR_RECOVERY#`
  4. ステップ 40: `Exit`
  5. ステップ `#ERR_RECOVERY#`: `Melsec_P5s_S, "$STATION_NO$$PC_NO$$CMD_WRITE$$SUBCMD$$DEV_CODE$$ADDR_CTRL$$POINTS$$VAL_RECOVERY$"`
  6. ステップ 60: `Melsec_Res_P5s_R, "$STATION_NO$", "RESULT_DATA"`
  7. ステップ 70: `Judge, EQU, "$RESULT_DATA$", "ACK", #READ_START#`
  8. ステップ 80: `Exit`

### ⑤ 読み出し終了・復旧：`COMMAND, "READ_END", ON`
* **目的**: データ読み出し処理を完了し、シーケンサの状態を通常動作モードへ安全に復旧する。
* **パラメータ定義（完全機能別分離）**:
  * `STATION_NO` (`Text`, `"局番号"`, `"00"`)
  * `PC_NO` (`Text`, `"PC番号"`, `"FF"`)
  * `CMD_READ` (`List`, `"読出コマンド(LE)"`, `"0104"`, `"0304"`)
  * `CMD_WRITE` (`List`, `"書込コマンド(LE)"`, `"0114"`, `"0214"`)
  * `SUBCMD` (`List`, `"サブコマンド(LE)"`, `"0000"`, `"0100"`, `"8000"`, `"8100"`)
  * `DEV_CODE` (`List`, `"デバイスコード"`, `"B0"`, `"A8"`, `"B4"`, `"AF"`, `"A9"`, `"B5"`, `"C2"`, `"C5"`, `"90"`, `"92"`, `"93"`, `"94"`, `"A0"`, `"9C"`, `"9D"`, `"C1"`, `"C4"`)
  * `ADDR_STATUS` (`Text`, `"状態確認アドレス(LE)"`, `"FE7F00"`)
  * `ADDR_CTRL` (`Text`, `"制御コマンドアドレス(LE)"`, `"FF7F00"`)
  * `POINTS` (`Text`, `"アクセス点数(LE)"`, `"0100"`)
  * `VAL_FINISH_1` (`Text`, `"終了処理設定値1(LE)"`, `"0100"`)
  * `VAL_RESET` (`Text`, `"復旧初期設定値(LE)"`, `"0000"`)
  * `STATUS` (`Work`, `"判定結果"`, `""`)
* **シーケンス手順**:
  1. ステップ 10: `Melsec_P5s_S, "$STATION_NO$$PC_NO$$CMD_WRITE$$SUBCMD$$DEV_CODE$$ADDR_CTRL$$POINTS$$VAL_FINISH_1$"`
  2. ステップ 20: `Melsec_Res_P5s_R, "$STATION_NO$", "STATUS"`
  3. ステップ 30: `Judge, EQU, "$STATUS$", "ACK", #ERR_EXIT#`
  4. ステップ `#CHECK_LOOP#`: `Melsec_P5s_S, "$STATION_NO$$PC_NO$$CMD_READ$$SUBCMD$$DEV_CODE$$ADDR_STATUS$$POINTS$"`
  5. ステップ 50: `Melsec_Res_P5s_R, "$STATION_NO$", "STATUS", "DEC"`
  6. ステップ 60: `Judge, EQU, "$STATUS$", "0x0001", #CHECK_LOOP#` （0x0001 になるまで再送ループ）
  7. ステップ 70: `Melsec_P5s_S, "$STATION_NO$$PC_NO$$CMD_WRITE$$SUBCMD$$DEV_CODE$$ADDR_CTRL$$POINTS$$VAL_RESET$"`
  8. ステップ 80: `Melsec_Res_P5s_R, "$STATION_NO$", "STATUS"`
  9. ステップ 90: `Judge, EQU, "$STATUS$", "ACK", #ERR_EXIT#`
  10. ステップ 100: `Exit`
  11. ステップ `#ERR_EXIT#`: `Exit`

### ⑥ 通信ポート切断：`COMMAND, "PORT_CLOSE"`
* **目的**: シーケンサへリセット制御値を書き込み、通信ポート回線を安全に切断・解放する。
* **パラメータ定義（完全機能別分離）**:
  * `STATION_NO` (`Text`, `"局番号"`, `"00"`)
  * `PC_NO` (`Text`, `"PC番号"`, `"FF"`)
  * `CMD_WRITE` (`List`, `"書込コマンド(LE)"`, `"0114"`, `"0214"`)
  * `SUBCMD` (`List`, `"サブコマンド(LE)"`, `"0000"`, `"0100"`, `"8000"`, `"8100"`)
  * `DEV_CODE` (`List`, `"デバイスコード"`, `"B0"`, `"A8"`, `"B4"`, `"AF"`, `"A9"`, `"B5"`, `"C2"`, `"C5"`, `"90"`, `"92"`, `"93"`, `"94"`, `"A0"`, `"9C"`, `"9D"`, `"C1"`, `"C4"`)
  * `ADDR_CTRL` (`Text`, `"制御コマンドアドレス(LE)"`, `"FF7F00"`)
  * `POINTS` (`Text`, `"アクセス点数(LE)"`, `"0100"`)
  * `VAL_RESET` (`Text`, `"復旧初期設定値(LE)"`, `"0000"`)
  * `STATUS` (`Work`, `"判定結果"`, `""`)
* **シーケンス手順**:
  1. ステップ 10: `Melsec_P5s_S, "$STATION_NO$$PC_NO$$CMD_WRITE$$SUBCMD$$DEV_CODE$$ADDR_CTRL$$POINTS$$VAL_RESET$"`
  2. ステップ 20: `Melsec_Res_P5s_R, "$STATION_NO$", "STATUS"`
  3. ステップ 30: `Melsec_Close`
  4. ステップ 40: `Exit`

---

## ✅ 5. 出力前セルフチェックリスト

- [ ] 他通信ドメインのコマンド（`Com_Mode`, `Com_Open`, `Com_Close` 等）を一切使用せず、Melco 用コマンドセット（`Melsec_Mode`, `Melsec_Open`, `Melsec_Close`, `Melsec_P5s_S`, `Melsec_Res_P5s_R`）と共通基盤コマンドのみで完結しているか？
- [ ] バイナリペイロードが `STATION_NO`, `PC_NO`, `CMD_READ`/`CMD_WRITE`, `SUBCMD`, `DEV_CODE`, アドレス, `POINTS`, 設定値の各機能要素にリトルエンディアン16進表記で完全に分離・パラメータ化され、取り得る文字列が決まっている項目（`BAUD`, `PARITY`, `DATA_BITS`, `STOP_BITS`, `FLOW_CTRL`, `CMD_READ`, `CMD_WRITE`, `SUBCMD`, `DEV_CODE`）がすべて `List` 型で選択可能になっているか？
- [ ] 送信ステップ（`Melsec_P5s_S`）および受信照合ステップ（`Melsec_Res_P5s_R`）において、分離されたパラメータが `$STATION_NO$$PC_NO$...` のように正しく変数展開で組み立てられているか？
- [ ] 形式5（バイナリコード通信方式）に合わせて `Melsec_Mode` のプロトコル形式が `"P5"` に設定され、チェックサム異常判定が `"ERROR_SUMCHECK"` となっているか？
- [ ] すべての `TITLE`, `COMMAND` 行が行頭（第1フィールド）から始まっているか？
- [ ] すべての `PARAM`, `SEQUENCE` 行が行頭カンマ（`,`）から始まっているか？
- [ ] すべての `PARAM` 行において、第5フィールド（表示名）および第6フィールド（初期値）が省略・欠落せずに明記されているか？
- [ ] `COMMAND` ブロック配下の `PARAM` 行・`SEQUENCE` 行に行末コメント（`;`）が一切含まれていないか？
- [ ] コメント行は必ず独立した行（行頭 `;`）として配置されているか？
- [ ] 生成される CSV ファイルは Shift-JIS (CP932) で保存される前提となっているか？

---

## 📋 6. 出力フォーマット

余計な自然言語による前置きや解説を省き、すぐに CSV ファイル（`Melco_P5.csv`）として保存できるように、 **マークダウンのコードブロック（```csv）に作成した CSV コードのみ** を出力してください。
