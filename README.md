# tuexp2025 — 渡辺ゼミ 2025年用  縦書き読解実験プログラム

## 概要
このプログラムは **jsPsych v7** を用いた縦書き読解課題です。  
「文章読解 → 設問（多肢選択）→ 主観評価（Likert）→ 休憩」を複数ブロック実施し、回答データを **CSV** で保存します。


## 起動方法

### 1) 本番用
https://temochiz-lab.github.io/tuexp2025/sugiyama/?pattern=1  
https://temochiz-lab.github.io/tuexp2025/sugiyama/?pattern=2  
https://temochiz-lab.github.io/tuexp2025/sugiyama/?pattern=3  
https://temochiz-lab.github.io/tuexp2025/sugiyama/?pattern=4  
  
### 2) テスト用
https://temochiz-lab.github.io/tuexp2025/sugiyama/?pattern=1&test=1  
https://temochiz-lab.github.io/tuexp2025/sugiyama/?pattern=2&test=1  
https://temochiz-lab.github.io/tuexp2025/sugiyama/?pattern=3&test=1  
https://temochiz-lab.github.io/tuexp2025/sugiyama/?pattern=4&test=1  
https://temochiz-lab.github.io/tuexp2025/sugiyama/?pattern=0&test=1  


## Usage（URL 引数）

### 書式

```
index.html?pattern=<番号>&test=<0|1>
```

### 引数一覧

* **pattern**（出題順の指定。省略時は `1`）

  * `1` … 1 → 2 → 7 → 8（既定）
  * `2` … 7 → 8 → 1 → 2
  * `3` … 3 → 4 → 5 → 6
  * `4` … 5 → 6 → 3 → 4
  * `0` … 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8(確認用)


* **test**（テストモード切替）

  * `0` … 本番モード（読解120秒 / 設問60秒 / 休憩180秒）
  * `1` … テストモード（読解10秒 / 設問10秒 / 休憩10秒、必須回答オフ）

---

## 使用例

**標準実行（pattern=1, 本番設定）**

```
index.html
```

または

```
index.html?pattern=1&test=0
```

**出題順を pattern=3 にして実行**

```
index.html?pattern=3
```

**テストモードで起動**

```
index.html?test=1
```

**出題順を 2 にし、テストモードで起動**

```
index.html?pattern=2&test=1
```

---

## 実行中の操作（被験者向け）

* **読解画面**：`q` キーで次へ（時間経過でも自動進行）
* **設問画面**：

  * 「問題文を表示」ボタンで本文をモーダル表示（`ESC` で閉じる）
  * 選択肢は必須（テストモードでは任意）
* **Likert 評価**：5 段階必須（テストモードでは任意）
* **休憩画面**：`q` でスキップ可、時間経過で自動終了
* **共通操作**：`Ctrl + Q` で「次へ」相当の操作

---

## データ保存

* 実験終了時に **CSV ファイル**が自動ダウンロードされます。
* CSVファイルをExcelで読み込む場合は一旦メモ帳で開いてから「ファイル」→「名前を付けて保存」→「エンコード」 を UTF-8(BOM付き) に変更しておきます。
* ファイル名形式：

```
sugiyama2025-0812-002-YYYYMMDD-hhmmss.csv
```

---

## 注意事項

* 実施環境は **Chrome** を推奨します（縦書き表示やキー入力の安定性のため）。
* Mac では `Cmd + Q` はアプリ終了のショートカットです。**本プログラムは `Ctrl + Q`** を使用します。
* ネット環境が不安定な場合は、依存ライブラリ（jsPsych 本体・各プラグイン）をローカルに配置してください。

```
```
