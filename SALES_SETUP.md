# 売上表: 既存スプシ連携

## 対象スプシ（オーナー指定）

- URL: https://docs.google.com/spreadsheets/d/1tkaQzpRwSehJwl4tfZVqARGxORHjxY3g1XM7A1bkdwo/edit
- ID: `1tkaQzpRwSehJwl4tfZVqARGxORHjxY3g1XM7A1bkdwo`
- 形式: **月タブの日次表**（例: シート名 `7月`）
- 1行目付近ヘッダ例:

| 7月 | 曜日 | 天気 | 備考 | 施術人数 | 施+回 | リピート客数 | リ+回 | 現金 | キャッシュレス | 総売上 | 税抜 | 単価 | … |
|-----|------|------|------|----------|-------|--------------|-------|------|----------------|--------|------|------|---|
| 1 | 水 | ☁ | 無 | 18 | 19 | … | | 11,400 | 33,000 | 44,400 | … |

- 日付列: ヘッダが「7月」などで、セルは **日（1〜31）**
- 店舗列: **なし** → スクリプトプロパティで店舗を固定

## GAS スクリプトプロパティ（この設定を入れる）

| プロパティ | 値 |
|---|---|
| `SALES_SPREADSHEET_ID` | `1tkaQzpRwSehJwl4tfZVqARGxORHjxY3g1XM7A1bkdwo` |
| `SALES_SHEET_NAME` | `{m}月` |
| `SALES_DEFAULT_STORE_ID` | `K1` または `K2`（この表がどちらの店か） |
| `ADMIN_KEY` | （既存）`charge08` |

`{m}月` にすると、画面で7月を開いたときタブ「7月」、8月なら「8月」を自動選択します。

店舗がシートごとに分かれている場合は、店舗ごとに DEFAULT を変えるか、タブ名規則を相談してください。

## コード

1. `gas-sales-snippet.gs` または `gas-code-current.gs` の売上部分を本番へ  
2. `doPost` に `salesData`  
3. 既存デプロイを **新バージョン** で更新  

売上スプシを、GAS を動かしている Google アカウントに **閲覧共有** すること。

## 画面

https://sasaking888.github.io/charge-shift-form/sales.html?v=1.1
