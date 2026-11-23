# シーケンサデータ収集用 マクロスクリプト自動生成 AI指示書 (Melco_P3.csv 生成用)

> **生成AI**: Antigravity (Gemini 4 Carbon (Medium))

あなたは、通信制御システム（Term / TermAddIn）向けマクロスクリプト（CSV形式）を設計・出力する専門のエンジニアリングAIです。  
以下の「参照ドキュメント」および「システム絶対制約」「電文フレーム完全機能別パラメータ設計仕様」を厳格に遵守し、三菱電機 MELSEC シーケンサから計算機リンク **形式3（サムチェックなし・CR+LF終端）** で各種データを収集・制御するための Term マクロスクリプト（`Melco_P3.csv`）を作成してください。

---

## 📚 1. 参照ドキュメントおよび準拠規格

1. **システム文法・構文規約（最優先遵守）**
   * `Manual/TermAddIn/マクロスクリプト取扱説明書.Md`（Term マクロスクリプト取扱説明書 兼 AI自動生成指示書）
2. **通信・プロトコル個別仕様（Melco 専用コマンドセット）**
   * `Manual/TermAddIn/Melco/Melsec_Mode.Md`（MELSEC 計算機リンク通信条件・プロトコル形式設定）
   * `Manual/TermAddIn/Melco/Melsec_Open.Md` / `Manual/TermAddIn/Melco/Melsec_Close.Md`（MELSEC 通信回線接続・切断）
   * `Manual/TermAddIn/Melco/Melsec_P3s_S.Md`（MELSEC プロトコル3 コマンド送信）
   * `Manual/TermAddIn/Melco/Melsec_Res_P3s_R.Md`（MELSEC プロトコル3 応答受信）
   * `Manual/Core/Sequence/Judge.Md`（比較判定・条件分岐制御）
   * `Manual/Core/Sequence/Jump.Md` / `Manual/Core/Param/Exit.Md`（無条件ジャンプ・終了制御）
3. **対象通信仕様**
   * 制御対象: 三菱電機 MELSEC シーケンサ（計算機リンク 形式3 通信・QnA互換3Cフレーム）
   * 通信パラメータ: ボーレート 57600 bps、パリティ Odd (O)、データ長 8 bit、ストップビット 1 bit、フロー制御 NONE
   * 伝送制御: プロトコル形式 `P3`（`Melsec_Mode` にて設定：ENQ/STX/ETX、サムチェックなし、CR+LF 終端）

---

## 🚨 2. システム絶対制約（構文エラー・実行時クラッシュ防止の鉄則）

1. **表記言語・文字コード・AI生成元の明記**
   * **表記言語**: 多言語化を意識した構成とし、スクリプト内のコメント、パラメータ表示名、メッセージ等は、特に指定がない限り **日本語をデフォルトとして記述・設定** してください。
   * **文字コード**: 生成する CSV ファイル（`Melco_P3.csv`）は、日本語 Excel での文字化け防止のため **Shift-JIS (CP932)** で保存してください。
   * **AI生成元の明記（他者再現性の保証）**: CSV ファイルの最先頭ヘッダコメント行（`;`）に、対象システム情報とともに **他者が同一条件で再現できるよう、実行環境（エージェント／IDE名）および自身のAIエンジン（モデル名・エディション／ティア）を正確に判別できる名称（例: `; 生成AI: Antigravity (Gemini 4 Carbon (Medium))`）** を必ず記述してください（エージェント名のみの記載や、推測・例示の丸写しによる不正確なモデル名の記載は厳禁）。
   * **パス表記**: ファイル内に開発環境固有のローカル絶対パスや個人名を含めることは厳禁です。必ず相対パスまたは抽象表記のみで記述してください。

2. **HEADER を含む電文フレームの完全機能別パラメータ分離と `List` 選択化（最重要仕様）**
   * ヘッダ情報（`"F90000FF00"`）を一括りの文字列として扱わず、プロトコル規格に準拠して各 2 桁の機能要素へ完全に分離して `PARAM` 定義してください。また、 **サブヘッダ・コマンド・サブコマンド・デバイス種別・通信条件など、プロトコル規格により取り得る文字列が決まっている項目は必ず `List` 型で定義** し、ドロップダウンから選択できるようにしてください。
     * **サブヘッダ (`SUB_HEADER`)**: QnA互換フレーム識別子（`List`, 初期値 `"F9"`, 追加選択肢 `"F8"`）
     * **局番号 (`STATION_NO`)**: 通信相手局番（`Text`, `"00"`）
     * **ネットワーク番号 (`NET_NO`)**: 自局ネットワーク番号（`Text`, `"00"`）
     * **PC番号 (`PC_NO`)**: 自局CPU PC番号（`Text`, `"FF"`）
     * **要求先ユニットI/O (`UNIT_IO`)**: 要求先ユニットI/O番号（`Text`, `"00"`）
   * さらにコマンド部・データ部もすべて機能ごとに分離してください。
     * **コマンド (`CMD_READ`, `CMD_WRITE`)**: 読出（`List`, `"0401"`, `"0403"`）/ 書込（`List`, `"1401"`, `"1402"`）
     * **サブコマンド (`SUBCMD`)**: アクセス単位指定（`List`, `"0000"`, `"0001"`, `"0080"`, `"0081"`）
     * **デバイス種別 (`DEV_TYPE`)**: QnA互換3C ASCII 2文字デバイスコード（`List`, 初期値 `"ZR"`, 追加選択肢 `"D*"`, `"W*"`, `"R*"`, `"SD"`, `"SW"`, `"TN"`, `"CN"`, `"M*"`, `"L*"`, `"F*"`, `"V*"`, `"B*"`, `"X*"`, `"Y*"`, `"TS"`, `"CS"`）
     * **デバイスアドレス (`DEV_ADDR`, `ADDR_STATUS`, `ADDR_CTRL`)**: 先頭デバイス番号（`Text`, 例: `"007FFE"`, `"007FFF"`, `"008020"`）
     * **アクセス点数 (`POINTS`)**: 読出・書込点数（`Text`, 例: `"0001"`）
     * **設定値・書込値 (`VAL_INIT_CTRL`, `VAL_RECOVERY`, `VAL_FINISH_1`, `VAL_RESET`)**: 設定データ（`Text`, 例: `"0004"`, `"0002"`, `"0001"`, `"0000"`）
   * 送信電文および受信照合ヘッダは、`$SUB_HEADER$$STATION_NO$$NET_NO$$PC_NO$$UNIT_IO$...` のようにパラメータ動的置換記法（`$`）を連結して構成してください。

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
   * 本スクリプトは `TermAnalysis.Melco` 用であるため、通信制御には必ず **Melco 用コマンドセット（`Melsec_Mode`, `Melsec_Open`, `Melsec_Close`, `Melsec_P3s_S`, `Melsec_Res_P3s_R` 等）** および共通基盤コマンド（`Judge`, `Jump`, `Exit` 等）のみを使用してください。
   * 他通信ドメインのコマンドセット（`Com_Mode`, `Com_Open`, `Com_Close`, `Telnet_*`, `Modbus_*`, `Mc_*` 等）を跨いで混在させることは排他制御違反となるため **絶対禁止** です。

---

## 📡 3. 対象システム仕様および TITLE ブロック定義

シーケンサデータ収集システム（形式3）向けとして、`TITLE` ブロックを以下の通り定義してください。

* **タイトル名**: `"シーケンサデータ収集 (形式3)"`
* **書式**:
  ```csv
  TITLE, "シーケンサデータ収集 (形式3)"
  ```

---

## 📐 4. 実装すべき COMMAND ブロックと完全機能別パラメータ設計仕様

以下の 6 つの独立した `COMMAND` ブロックを実装してください。すべての送受信ステップで機能別分離パラメータを使用します。

### ① 通信ポート接続：`COMMAND, "PORT_OPEN"`
* **目的**: MELSEC 計算機リンク通信条件（ボーレート・パリティ・データ長・ストップビット・フロー制御）を `List` パラメータから選択設定し、プロトコル形式 `P3` で MELSEC 通信回線を接続オープンする。
* **パラメータ定義（`List` 選択式）**:
  * `BAUD` (`List`, `"ボーレート"`, `"57600"`, `"9600"`, `"19200"`, `"38400"`, `"115200"`)
  * `PARITY` (`List`, `"パリティ"`, `"O"`, `"E"`, `"N"`)
  * `DATA_BITS` (`List`, `"データ長"`, `"8"`, `"7"`)
  * `STOP_BITS` (`List`, `"ストップビット"`, `"1"`, `"2"`)
  * `FLOW_CTRL` (`List`, `"フロー制御"`, `"NONE"`, `"RTSCTS"`, `"XONXOFF"`)
* **シーケンス手順**:
  1. ステップ 10: `Melsec_Mode, "$BAUD$,$PARITY$,$DATA_BITS$,$STOP_BITS$", "$FLOW_CTRL$", "P3"`
  2. ステップ 20: `Melsec_Open`
  3. ステップ 30: `Exit`

### ② 読み出し開始・初期確認：`COMMAND, "READ_START", ON`
* **目的**: シーケンサとの初期ハンドシェイクを行い、指定デバイスの状態確認（`0x0000`）および制御コマンド送信に対する ACK 応答を検証する。
* **パラメータ定義（完全機能別分離）**:
  * `SUB_HEADER` (`List`, `"サブヘッダ"`, `"F9"`, `"F8"`)
  * `STATION_NO` (`Text`, `"局番号"`, `"00"`)
  * `NET_NO` (`Text`, `"ネットワーク番号"`, `"00"`)
  * `PC_NO` (`Text`, `"PC番号"`, `"FF"`)
  * `UNIT_IO` (`Text`, `"要求先ユニットI/O"`, `"00"`)
  * `CMD_READ` (`List`, `"読出コマンド"`, `"0401"`, `"0403"`)
  * `CMD_WRITE` (`List`, `"書込コマンド"`, `"1401"`, `"1402"`)
  * `SUBCMD` (`List`, `"サブコマンド"`, `"0000"`, `"0001"`, `"0080"`, `"0081"`)
  * `DEV_TYPE` (`List`, `"デバイス種別"`, `"ZR"`, `"D*"`, `"W*"`, `"R*"`, `"SD"`, `"SW"`, `"TN"`, `"CN"`, `"M*"`, `"L*"`, `"F*"`, `"V*"`, `"B*"`, `"X*"`, `"Y*"`, `"TS"`, `"CS"`)
  * `ADDR_STATUS` (`Text`, `"状態確認アドレス"`, `"007FFE"`)
  * `ADDR_CTRL` (`Text`, `"制御コマンドアドレス"`, `"007FFF"`)
  * `POINTS` (`Text`, `"アクセス点数"`, `"0001"`)
  * `VAL_INIT_CTRL` (`Text`, `"初期確認設定値"`, `"0004"`)
  * `STATUS` (`Work`, `"判定結果"`, `""`)
* **シーケンス手順**:
  1. ステップ 10: `Melsec_P3s_S, "$SUB_HEADER$$STATION_NO$$NET_NO$$PC_NO$$UNIT_IO$$CMD_READ$$SUBCMD$$DEV_TYPE$$ADDR_STATUS$$POINTS$"` （状態読出要求）
  2. ステップ 20: `Melsec_Res_P3s_R, "$SUB_HEADER$$STATION_NO$$NET_NO$$PC_NO$$UNIT_IO$", "$POINTS$", "STATUS"` （状態データ受信）
  3. ステップ 30: `Judge, EQU, "$STATUS$", "0x0000", #ERR_EXIT#` （正常値 0x0000 でなければエラー終了）
  4. ステップ 40: `Melsec_P3s_S, "$SUB_HEADER$$STATION_NO$$NET_NO$$PC_NO$$UNIT_IO$$CMD_WRITE$$SUBCMD$$DEV_TYPE$$ADDR_CTRL$$POINTS$$VAL_INIT_CTRL$"` （初期制御値書込）
  5. ステップ 50: `Melsec_Res_P3s_R, "ACK", "DUMMY", "STATUS"` （ACK 応答確認）
  6. ステップ 60: `Judge, EQU, "$STATUS$", "ACK", #ERR_EXIT#` （ACK でなければエラー終了）
  7. ステップ 70: `Exit` （正常終了）
  8. ステップ `#ERR_EXIT#`: `Exit` （異常終了）

### ③ 数値データ読み出し：`COMMAND, "NUM_READ", "RESULT_DATA"`
* **目的**: パラメータで指定されたデバイスアドレスから数値データを読み出してセル（`RESULT_DATA`）へ展開し、フレーム異常（`ERROR_FRAME`）発生時には復旧コマンド送信と再試行を行う（※形式3はサムチェックなしのためフレーム欠損エラーを監視）。
* **パラメータ定義（完全機能別分離）**:
  * `SUB_HEADER` (`List`, `"サブヘッダ"`, `"F9"`, `"F8"`)
  * `STATION_NO` (`Text`, `"局番号"`, `"00"`)
  * `NET_NO` (`Text`, `"ネットワーク番号"`, `"00"`)
  * `PC_NO` (`Text`, `"PC番号"`, `"FF"`)
  * `UNIT_IO` (`Text`, `"要求先ユニットI/O"`, `"00"`)
  * `CMD_READ` (`List`, `"読出コマンド"`, `"0401"`, `"0403"`)
  * `CMD_WRITE` (`List`, `"書込コマンド"`, `"1401"`, `"1402"`)
  * `SUBCMD` (`List`, `"サブコマンド"`, `"0000"`, `"0001"`, `"0080"`, `"0081"`)
  * `DEV_TYPE` (`List`, `"デバイス種別"`, `"ZR"`, `"D*"`, `"W*"`, `"R*"`, `"SD"`, `"SW"`, `"TN"`, `"CN"`, `"M*"`, `"L*"`, `"F*"`, `"V*"`, `"B*"`, `"X*"`, `"Y*"`, `"TS"`, `"CS"`)
  * `DEV_ADDR` (`Text`, `"読出デバイス番号"`, `"008020"`)
  * `POINTS` (`Text`, `"読出点数"`, `"0001"`)
  * `ADDR_CTRL` (`Text`, `"復旧制御アドレス"`, `"007FFF"`)
  * `VAL_RECOVERY` (`Text`, `"復旧制御設定値"`, `"0002"`)
  * `RESULT_DATA` (`Range`, `"読出データ結果"`, `""`)
* **シーケンス手順**:
  1. ステップ `#RETRY#`: `Melsec_P3s_S, "$SUB_HEADER$$STATION_NO$$NET_NO$$PC_NO$$UNIT_IO$$CMD_READ$$SUBCMD$$DEV_TYPE$$DEV_ADDR$$POINTS$"` （データ読出要求）
  2. ステップ 20: `Melsec_Res_P3s_R, "$SUB_HEADER$$STATION_NO$$NET_NO$$PC_NO$$UNIT_IO$", "$POINTS$", "RESULT_DATA", "%X"` （16進データ受信・結果展開）
  3. ステップ 30: `Judge, NEQ, "$RESULT_DATA$", "ERROR_FRAME", #RETRY_FAIL#` （フレーム異常時は復旧へ）
  4. ステップ 40: `Exit` （正常終了）
  5. ステップ `#RETRY_FAIL#`: `Melsec_P3s_S, "$SUB_HEADER$$STATION_NO$$NET_NO$$PC_NO$$UNIT_IO$$CMD_WRITE$$SUBCMD$$DEV_TYPE$$ADDR_CTRL$$POINTS$$VAL_RECOVERY$"` （復旧制御値書込）
  6. ステップ 60: `Melsec_Res_P3s_R, "ACK", "DUMMY", "RESULT_DATA"` （ACK 応答確認）
  7. ステップ 70: `Judge, EQU, "$RESULT_DATA$", "ACK", #RETRY#` （ACK 受信時は再送 `#RETRY#` へ）
  8. ステップ 80: `Exit` （復旧不能時終了）

### ④ レジスタ一括読み出し：`COMMAND, "REG_READ", "RESULT_DATA"`
* **目的**: 指定レジスタ群からデータを一括読み出しし、連続データとして結果変数へ格納する。
* **パラメータ定義（完全機能別分離）**:
  * `SUB_HEADER` (`List`, `"サブヘッダ"`, `"F9"`, `"F8"`)
  * `STATION_NO` (`Text`, `"局番号"`, `"00"`)
  * `NET_NO` (`Text`, `"ネットワーク番号"`, `"00"`)
  * `PC_NO` (`Text`, `"PC番号"`, `"FF"`)
  * `UNIT_IO` (`Text`, `"要求先ユニットI/O"`, `"00"`)
  * `CMD_READ` (`List`, `"読出コマンド"`, `"0401"`, `"0403"`)
  * `CMD_WRITE` (`List`, `"書込コマンド"`, `"1401"`, `"1402"`)
  * `SUBCMD` (`List`, `"サブコマンド"`, `"0000"`, `"0001"`, `"0080"`, `"0081"`)
  * `DEV_TYPE` (`List`, `"デバイス種別"`, `"ZR"`, `"D*"`, `"W*"`, `"R*"`, `"SD"`, `"SW"`, `"TN"`, `"CN"`, `"M*"`, `"L*"`, `"F*"`, `"V*"`, `"B*"`, `"X*"`, `"Y*"`, `"TS"`, `"CS"`)
  * `DEV_ADDR` (`Text`, `"読出デバイス番号"`, `"008020"`)
  * `POINTS` (`Text`, `"読出点数"`, `"0001"`)
  * `ADDR_CTRL` (`Text`, `"復旧制御アドレス"`, `"007FFF"`)
  * `VAL_RECOVERY` (`Text`, `"復旧制御設定値"`, `"0002"`)
  * `RESULT_DATA` (`Range`, `"レジスタデータ"`, `""`)
* **シーケンス手順**:
  1. ステップ `#READ_START#`: `Melsec_P3s_S, "$SUB_HEADER$$STATION_NO$$NET_NO$$PC_NO$$UNIT_IO$$CMD_READ$$SUBCMD$$DEV_TYPE$$DEV_ADDR$$POINTS$"`
  2. ステップ 20: `Melsec_Res_P3s_R, "$SUB_HEADER$$STATION_NO$$NET_NO$$PC_NO$$UNIT_IO$", "$POINTS$", "RESULT_DATA", "%X"`
  3. ステップ 30: `Judge, NEQ, "$RESULT_DATA$", "ERROR_FRAME", #ERR_RECOVERY#`
  4. ステップ 40: `Exit`
  5. ステップ `#ERR_RECOVERY#`: `Melsec_P3s_S, "$SUB_HEADER$$STATION_NO$$NET_NO$$PC_NO$$UNIT_IO$$CMD_WRITE$$SUBCMD$$DEV_TYPE$$ADDR_CTRL$$POINTS$$VAL_RECOVERY$"`
  6. ステップ 60: `Melsec_Res_P3s_R, "ACK", "DUMMY", "RESULT_DATA"`
  7. ステップ 70: `Judge, EQU, "$RESULT_DATA$", "ACK", #READ_START#`
  8. ステップ 80: `Exit`

### ⑤ 読み出し終了・復旧：`COMMAND, "READ_END", ON`
* **目的**: データ読み出し処理を完了し、シーケンサの状態を通常動作モードへ安全に復旧する。
* **パラメータ定義（完全機能別分離）**:
  * `SUB_HEADER` (`List`, `"サブヘッダ"`, `"F9"`, `"F8"`)
  * `STATION_NO` (`Text`, `"局番号"`, `"00"`)
  * `NET_NO` (`Text`, `"ネットワーク番号"`, `"00"`)
  * `PC_NO` (`Text`, `"PC番号"`, `"FF"`)
  * `UNIT_IO` (`Text`, `"要求先ユニットI/O"`, `"00"`)
  * `CMD_READ` (`List`, `"読出コマンド"`, `"0401"`, `"0403"`)
  * `CMD_WRITE` (`List`, `"書込コマンド"`, `"1401"`, `"1402"`)
  * `SUBCMD` (`List`, `"サブコマンド"`, `"0000"`, `"0001"`, `"0080"`, `"0081"`)
  * `DEV_TYPE` (`List`, `"デバイス種別"`, `"ZR"`, `"D*"`, `"W*"`, `"R*"`, `"SD"`, `"SW"`, `"TN"`, `"CN"`, `"M*"`, `"L*"`, `"F*"`, `"V*"`, `"B*"`, `"X*"`, `"Y*"`, `"TS"`, `"CS"`)
  * `ADDR_STATUS` (`Text`, `"状態確認アドレス"`, `"007FFE"`)
  * `ADDR_CTRL` (`Text`, `"制御コマンドアドレス"`, `"007FFF"`)
  * `POINTS` (`Text`, `"アクセス点数"`, `"0001"`)
  * `VAL_FINISH_1` (`Text`, `"終了処理設定値1"`, `"0001"`)
  * `VAL_RESET` (`Text`, `"復旧初期設定値"`, `"0000"`)
  * `STATUS` (`Work`, `"判定結果"`, `""`)
* **シーケンス手順**:
  1. ステップ 10: `Melsec_P3s_S, "$SUB_HEADER$$STATION_NO$$NET_NO$$PC_NO$$UNIT_IO$$CMD_WRITE$$SUBCMD$$DEV_TYPE$$ADDR_CTRL$$POINTS$$VAL_FINISH_1$"`
  2. ステップ 20: `Melsec_Res_P3s_R, "ACK", "DUMMY", "STATUS"`
  3. ステップ 30: `Judge, EQU, "$STATUS$", "ACK", #ERR_EXIT#`
  4. ステップ `#CHECK_LOOP#`: `Melsec_P3s_S, "$SUB_HEADER$$STATION_NO$$NET_NO$$PC_NO$$UNIT_IO$$CMD_READ$$SUBCMD$$DEV_TYPE$$ADDR_STATUS$$POINTS$"`
  5. ステップ 50: `Melsec_Res_P3s_R, "$SUB_HEADER$$STATION_NO$$NET_NO$$PC_NO$$UNIT_IO$", "$POINTS$", "STATUS"`
  6. ステップ 60: `Judge, EQU, "$STATUS$", "0x0001", #CHECK_LOOP#` （0x0001 になるまで再送ループ）
  7. ステップ 70: `Melsec_P3s_S, "$SUB_HEADER$$STATION_NO$$NET_NO$$PC_NO$$UNIT_IO$$CMD_WRITE$$SUBCMD$$DEV_TYPE$$ADDR_CTRL$$POINTS$$VAL_RESET$"`
  8. ステップ 80: `Melsec_Res_P3s_R, "ACK", "DUMMY", "STATUS"`
  9. ステップ 90: `Judge, EQU, "$STATUS$", "ACK", #ERR_EXIT#`
  10. ステップ 100: `Exit`
  11. ステップ `#ERR_EXIT#`: `Exit`

### ⑥ 通信ポート切断：`COMMAND, "PORT_CLOSE"`
* **目的**: シーケンサへリセット制御値を書き込み、通信ポート回線を安全に切断・解放する。
* **パラメータ定義（完全機能別分離）**:
  * `SUB_HEADER` (`List`, `"サブヘッダ"`, `"F9"`, `"F8"`)
  * `STATION_NO` (`Text`, `"局番号"`, `"00"`)
  * `NET_NO` (`Text`, `"ネットワーク番号"`, `"00"`)
  * `PC_NO` (`Text`, `"PC番号"`, `"FF"`)
  * `UNIT_IO` (`Text`, `"要求先ユニットI/O"`, `"00"`)
  * `CMD_WRITE` (`List`, `"書込コマンド"`, `"1401"`, `"1402"`)
  * `SUBCMD` (`List`, `"サブコマンド"`, `"0000"`, `"0001"`, `"0080"`, `"0081"`)
  * `DEV_TYPE` (`List`, `"デバイス種別"`, `"ZR"`, `"D*"`, `"W*"`, `"R*"`, `"SD"`, `"SW"`, `"TN"`, `"CN"`, `"M*"`, `"L*"`, `"F*"`, `"V*"`, `"B*"`, `"X*"`, `"Y*"`, `"TS"`, `"CS"`)
  * `ADDR_CTRL` (`Text`, `"制御コマンドアドレス"`, `"007FFF"`)
  * `POINTS` (`Text`, `"アクセス点数"`, `"0001"`)
  * `VAL_RESET` (`Text`, `"復旧初期設定値"`, `"0000"`)
  * `STATUS` (`Work`, `"判定結果"`, `""`)
* **シーケンス手順**:
  1. ステップ 10: `Melsec_P3s_S, "$SUB_HEADER$$STATION_NO$$NET_NO$$PC_NO$$UNIT_IO$$CMD_WRITE$$SUBCMD$$DEV_TYPE$$ADDR_CTRL$$POINTS$$VAL_RESET$"`
  2. ステップ 20: `Melsec_Res_P3s_R, "ACK", "DUMMY", "STATUS"`
  3. ステップ 30: `Melsec_Close`
  4. ステップ 40: `Exit`

---

## ✅ 5. 出力前セルフチェックリスト

- [ ] 他通信ドメインのコマンド（`Com_Mode`, `Com_Open`, `Com_Close` 等）を一切使用せず、Melco 用コマンドセット（`Melsec_Mode`, `Melsec_Open`, `Melsec_Close`, `Melsec_P3s_S`, `Melsec_Res_P3s_R`）と共通基盤コマンドのみで完結しているか？
- [ ] ヘッダ（`"F90000FF00"`）が `SUB_HEADER`, `STATION_NO`, `NET_NO`, `PC_NO`, `UNIT_IO` の 5 つの機能要素（各2桁）に完全に分離・パラメータ化されているか？
- [ ] コマンド・サブコマンド・デバイス種別・アドレス・点数・設定値もすべて独立したパラメータとして定義され、取り得る文字列が決まっている項目（`BAUD`, `PARITY`, `DATA_BITS`, `STOP_BITS`, `FLOW_CTRL`, `SUB_HEADER`, `CMD_READ`, `CMD_WRITE`, `SUBCMD`, `DEV_TYPE`）がすべて `List` 型で選択可能になっているか？
- [ ] 送信ステップ（`Melsec_P3s_S`）および受信照合ステップ（`Melsec_Res_P3s_R`）において、分離されたパラメータが `$SUB_HEADER$$STATION_NO$$NET_NO$$PC_NO$$UNIT_IO$...` のように正しく変数展開で組み立てられているか？
- [ ] 形式3（サムチェックなし・CR+LF終端）に合わせて `Melsec_Mode` のプロトコル形式が `"P3"` に設定され、フレーム異常判定が `"ERROR_FRAME"` となっているか？
- [ ] すべての `TITLE`, `COMMAND` 行が行頭（第1フィールド）から始まっているか？
- [ ] すべての `PARAM`, `SEQUENCE` 行が行頭カンマ（`,`）から始まっているか？
- [ ] すべての `PARAM` 行において、第5フィールド（表示名）および第6フィールド（初期値）が省略・欠落せずに明記されているか？
- [ ] `COMMAND` ブロック配下の `PARAM` 行・`SEQUENCE` 行に行末コメント（`;`）が一切含まれていないか？
- [ ] コメント行は必ず独立した行（行頭 `;`）として配置されているか？
- [ ] 生成される CSV ファイルは Shift-JIS (CP932) で保存される前提となっているか？

---

## 📋 6. 出力フォーマット

余計な自然言語による前置きや解説を省き、すぐに CSV ファイル（`Melco_P3.csv`）として保存できるように、 **マークダウンのコードブロック（`csv` 形式）に作成した CSV コードのみ** を出力してください。

---

## ⚙️ プロパティ設定優先およびパラメータ定義規約（必須）

* **プロパティ設定優先（既定動作）**: COM ポート・IP アドレス・ポート番号・通信条件・局番号・PC 番号・スレーブアドレス・ルーティング番号など、プロパティ設定画面で指定できる項目は **プロパティ設定を優先して使用すること** を基本とします。
* **空欄 "" デフォルトの List 形式**: シーケンス用コマンドのパラメータとして上記項目を定義する場合は、必ず **""（空欄＝プロパティ設定を使用）を先頭デフォルトとした代表値の List 形式** で記述してください（例: `,PARAM, STATION_NO, List, "局番号(空欄=プロパティ)", "", "00", "01", "02", "03"`）。
* **省略パターンの両対応**: コマンド実行時は、引数自体を記述しない場合（パターン A）と、空文字 ""・空白・Excel 空セルを渡した場合（パターン B）の双方で自動的にプロパティ設定値が適用されます。一時的に別局へアクセスしたい場合のみリストから代表値を選択または直接入力してください。
* **先頭 "0" 付加による必要桁数の自動補正（通信エラー防止）**: Excel セルへ "00", "01", "0401", "0001", "007FFE" 等を入力すると先頭ゼロが省略される場合があるため、Melsec_P*s_S / Melsec_Res_P*s_R は 1 文字列へ連結せず **各機能要素（サブヘッダ[2]・局番号[2]・NW番号[2]・PC番号[2]・ユニットI/O[2]・コマンド[4]・サブコマンド[4]・デバイス・アドレス[6]・点数[4]・設定値[4]）を個別引数として渡す形式** としてください。コマンド側で空欄時のプロパティ補完と必要桁数への先頭 "0" 付加補正を必ず実行します。
