# 基本設計書


## 1. 目的と全体像

* **目的**：文章（縦書き）読解 → 選択式設問 → 主観評価（Likert）→ 休憩、を複数ブロックで実施し、回答ログを **CSV** で保存します。
* **構成**：

  1. フルスクリーン開始（`jsPsychFullscreen`）
  2. 研究趣旨提示（Page1）
  3. 顔情報入力（Page2：学籍番号・年齢）
  4. 注意事項提示（Page3）
  5. 練習試行（読解 → 設問（縦書き） → Likert → 休憩）
  6. 本番試行（出題**パターン**に応じて 4 つの文章ブロックを提示）
  7. フルスクリーン終了
  8. 終了メッセージ（bye）
* **キーポイント**：

  * 読解画面は `q` キーで進行（`choices: 'q'`）。
  * 設問（`survey-multi-choice`）では「**問題文を表示**」ボタンでモーダル表示（ESC で閉じる）。
  * **Ctrl + Q** で「次へ相当」の動作をグローバルに付与（Page1 の `on_load` で登録）。
  * 回答データは**自動で CSV 保存**（ファイル名は日時入り）。

---

## 2. 必要環境

* ブラウザ：最新の Chrome / Edge / Firefox（**Chrome 推奨**）
* ネットワーク：CDN（unpkg）からプラグインを読み込みます。**オフライン運用時はローカル配置**が必要です。
* HTTPS 環境推奨（フルスクリーン制御・音声系での制限回避のため）

---

## 3. 起動方法と URL パラメータ

### 3.1 そのまま起動

* ファイルをブラウザで開くだけで実行可能（ただし CDN 参照のためネット接続が必要）。

### 3.2 URL パラメータ

* **`?pattern={1|2|3|4}`**：本番試行の出題順を切り替え

  * `1`（既定）：1→2→7→8
  * `2`：7→8→1→2
  * `3`：3→4→5→6
  * `4`：5→6→3→4
* **`?test=1`**：**テストモード**（大幅に短時間化）

  * 読解表示：2分→**5秒**
  * 設問回答制限：1分→**5秒**
  * 休憩：3分→**5秒**

> 例）`index.html?pattern=2&test=1`

---

## 4. データ保存

* **保存タイミング**：実験終了時（`initJsPsych({ on_finish })`）
* **形式**：`CSV`
* **ファイル名**：`sugiyama2025-0812-002-YYYYMMDD-hhmmss.csv`
* **保存先**：ブラウザのダウンロード（`jsPsych.data.get().localSave('csv', filename)`）

---

## 5. タイムライン詳細

### 5.1 フルスクリーン開始

* `enter_fullscreen`：開始ボタン → 全画面へ。

### 5.2 Page1（研究趣旨）

* ボタン：**次へ**
* **Ctrl + Q** グローバルハンドラを**ここで一度だけ登録**。

  * 機能：各 trial で「次へ」ボタンがあればクリック、なければ `jsPsych.finishTrial()` を試行。

### 5.3 Page2（被験者情報）

* `jsPsychSurveyText`：学籍番号（id）、年齢（age）
* `required`：既定 **必須**（`test=1` のときのみ任意）
* ボタン：**次へ**

### 5.4 Page3（注意事項）

* ボタン：**開始する**

### 5.5 練習試行（PracticeTrials）

1. 案内（PracticePage1）
2. **読解（縦書きテキスト）**：`jsPsychHtmlKeyboardResponse`

   * **`q` キー**で次へ（時間到達でも進む設定）
3. **設問（縦書き）**：`survey-multi-choice`

   * レイアウト・モーダル処理は **`MakeVerticalQuestions()`** で自動付与
   * **回答必須**（`test=1` の場合は任意）
   * **回答制限時間**：`responseTimeLimit`（経過で自動クリック）
4. **Likert 評価**（QofFont）：全て **required**（`test=1` では任意）
5. **休憩**：`jsPsychHtmlKeyboardResponse`（`q`）＋`trial_duration` と `stimulus_duration` を `resttime_msec` に設定

### 5.6 本番試行（Trials）

* **ExamText\[n]**：各文章の縦書き読解画面（`q` キーで進行）
* **ExamQuestions\[n]**：各文章に対する 5 問の多肢選択（縦書き）
* 各ブロックの最後に **Likert → 休憩**（末尾ブロックは休憩なしのパターンあり）

### 5.7 フルスクリーン終了 → 終了画面

---

## 6. 操作方法（被験者向け）

* **読解画面**：`q` キーで次へ（時間満了でも進行）
* **設問画面**：

  * 右上の **「問題文を表示」** ボタンで文章をモーダル表示
  * **ESC** でモーダルを閉じる
  * **選択必須**（テストモードを除く）
  * **Ctrl + Q** で次へ（実験者が必要に応じて案内）
* **Likert**：各項目を 5 段階で評価（基本必須）
* **休憩画面**：`q` でスキップ可／時間経過で自動終了

---

## 7. 設定変更ガイド（頻出）

### 7.1 時間パラメータ

* 冒頭のグローバル変数で管理：

  * `stimulusDuration`（読解：ミリ秒、既定 120000 = 2分）
  * `responseTimeLimit`（設問の回答制限：既定 60000 = 1分）
  * `resttime_msec`（休憩：既定 180000 = 3分）

### 7.2 必須回答の切替

* `MakeVerticalQuestions()` 内：

  ```js
  mustflag = true; if (testmode == 1) mustflag = false;
  // ↳ questions[0].required = mustflag
  ```
* `QofFont`（Likert）も同様。

### 7.3 出題順の変更

* `switch (pattern)` を編集（1～4 既定の並びをカスタマイズ可）。

### 7.4 Ctrl + Q の挙動

* **Page1.on\_load** 内のセレクタ配列を編集：

  ```js
  const selectors = [
    '.jspsych-btn',
    '#jspsych-survey-text-next',
    '#jspsych-survey-multi-choice-next',
    '#jspsych-survey-likert-next',
    '.jspsych-html-button-response-button',
    '.jspsych-audio-button-response-button'
  ];
  ```
* **無効化したい trial** がある場合は、フラグで分岐させるか、`jsPsych.pluginAPI` のキーハンドラ利用へ移行を検討（高度）。

### 7.5 モーダルの文面差し替え

* `MakeVerticalQuestions(…, PopupText)` に渡している **ExamText** を編集。

### 7.6 縦書きスタイル

* `on_load` 内で `<style>` を毎回注入しています。プロジェクト全体で一貫させるなら、**外部 CSS** にまとめると管理が容易です。

  * `writing-mode: vertical-rl;`
  * `text-orientation: upright;`
  * レイアウト（flex）など

---

## 8. データ仕様（CSV）

* 各 trial の key/value が 1 行化されます（jsPsych 既定仕様）。
* 主なフィールド例：

  * **timestamp / rt / trial\_type / trial\_index**
  * **responses**（`survey-*` 系の JSON 文字列）
  * **選択肢のテキスト**／**名前（name）**
* **学籍番号（id）・年齢（age）** は Page2 の `name` で紐づく（`responses` 内）。

> 解析前に、`responses` JSON を列へ正規化するスクリプト（R/Python）を用意すると効率的です。

---

## 9. よくある質問（FAQ）

**Q1. Ctrl + Q が効かない**

* Page1 の `on_load` で **`document.addEventListener` が実行されているか**確認。
* ブラウザショートカットと競合していないか（**Mac の Cmd+Q は別**）。
* `e.preventDefault()` だけでなく `e.stopImmediatePropagation()` を追加すると競合に強い場合あり。

**Q2. 設問が時間切れで自動的に進まない**

* `responseTimeLimit` の値を確認（`MakeVerticalQuestions` の `setTimeout`）。
* 次へボタンのセレクタ `#jspsych-survey-multi-choice-next` の ID 名がプラグイン側と一致しているか確認。

**Q3. 回答必須にしたのに未回答で進めてしまう**

* `questions: [{ required: true }]` がセットされているか。
* テストモード（`?test=1`）だと `required: false` に落としている箇所がないか確認。

**Q4. 休憩が長すぎ／短すぎる**

* `resttime_msec` を変更。`stimulus_duration` と `trial_duration` の両方に適用されています。

**Q5. CSV が保存されない**

* ブラウザのダウンロード権限（ポップアップや自動ダウンロードのブロック）を確認。
* ローカルファイルで権限問題が出る場合は **簡易サーバ**（例：`python -m http.server`）経由で配信。

---

## 10. 安定運用の小ワザ

* **グローバルキーリスナーの後始末**：
  実験終了時（`on_finish`）に

  ```js
  if (window.__globalCtrlQHandler) {
    document.removeEventListener('keydown', window.__globalCtrlQHandler);
    window.__globalCtrlQHandler = null;
  }
  ```

  を入れておくと次回ロード時の不意動作を防げます。

* **CSS の一元管理**：
  何度も `<style>` を注入するより、固定 CSS を `<head>` の `<style>` もしくは外部 CSS にまとめると見通しが良くなります。

* **テキストフォント**：
  日本語縦書きはフォントで視認性が変わります。明朝系（例：`"MS Mincho", "ＭＳ 明朝", serif`）を指定済みですが、事前にターゲット環境で可読性を確認してください。

---

## 11. コードの主な編集ポイント早見表

| 目的           | 変更箇所                                | 例                      |
| ------------ | ----------------------------------- | ---------------------- |
| 読解時間を変える     | 冒頭グローバル `stimulusDuration`          | `120000 → 90000`       |
| 設問制限時間を変える   | `responseTimeLimit`                 | `60000 → 45000`        |
| 休憩時間を変える     | `resttime_msec`                     | `180000 → 60000`       |
| 出題順の変更       | `switch(pattern)`                   | ブロック順の入替               |
| 回答必須/任意      | `MakeVerticalQuestions` / `QofFont` | `required: true/false` |
| Ctrl+Q 対象の拡張 | Page1 の `selectors`                 | セレクタの追加                |
| モーダル文面       | `ExamText[n]` / `PracticeExam1Text` | テキスト編集                 |

---

## 12. 既知の制限・注意

* **CDN 依存**：学内ネットワーク制限で `unpkg.com` が遮断されると動作しません。必要に応じてライブラリをローカルに配置してください。
* **ブラウザ差**：縦書きの描画・スクロール挙動はブラウザ差があります。**実施ブラウザを固定**するのが安全です。
* **キーボード奪取**：他のリスナーや IME 状態によってキー検出が遅延/無視される場合があります。可能なら IME オフ推奨の運用指示を添えると安定します。

