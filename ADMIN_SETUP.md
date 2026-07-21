# 管理コンソール（admin.html）セットアップ手順

admin.html はオーナー・店長用のPC向け管理画面です。LINE（LIFF）不要で、
共有の「管理キー」でログインします。キーの検証はGAS側で行います。

有効化するには、以下の3ステップが必要です。

## 1. GASに管理キーの検証コードを追加

GASエディタで `doPost` 関数を開き、リクエストボディを `JSON.parse` した**直後**に
以下を追加します（`payload` の部分は、パース結果を入れている変数名に合わせてください）。

```js
// === 管理コンソール用: 管理キー検証 ===
var ADMIN_KEY = PropertiesService.getScriptProperties().getProperty('ADMIN_KEY');
if (payload.adminKey !== undefined || payload.userId === 'ADMIN') {
  if (!ADMIN_KEY || payload.adminKey !== ADMIN_KEY) {
    return ContentService.createTextOutput(JSON.stringify({ ok: false, error: '管理キーが違います' }))
      .setMimeType(ContentService.MimeType.JSON);
  }
  payload.userId = 'ADMIN';
}
```

ポイント:
- 正しいキーを送ってきたリクエストだけを、特別ユーザー `ADMIN` として扱います。
- キーなしで `userId: 'ADMIN'` を直接送る「なりすまし」も拒否します。

## 2. 管理キーを設定

GASエディタの「プロジェクトの設定」（歯車アイコン）→「スクリプト プロパティ」で追加:

| プロパティ | 値 |
|---|---|
| `ADMIN_KEY` | 好きなパスコード（長めのランダム文字列を推奨。例: `charge-2026-Xy7kPq`） |

このキーをオーナーと店長だけで共有してください。

## 3. 権限マスタに ADMIN を登録 → 再デプロイ

スプレッドシートの「⑥権限マスタ」シートに、**店舗ごとに1行**追加します:

| LINEユーザーID | 氏名 | 店舗ID | 権限レベル |
|---|---|---|---|
| ADMIN | 管理者 | （各店舗のID） | 編集可 |

最後にGASを再デプロイします。**「デプロイ」→「デプロイを管理」→ 既存デプロイの編集（鉛筆）→
バージョン: 新バージョン → デプロイ**の手順で更新すると、URLが変わらずそのまま使えます。

## 使い方

- URL: `https://sasaking888.github.io/charge-shift-form/admin.html`（main反映後）
- 初回に管理キーを入力するとブラウザに保存され、次回から自動ログインします。
- 共有PCなどでは右上の「ログアウト」でキーを消せます。
- キーを変えたいとき（漏れた・店長交代など）は、スクリプトプロパティの `ADMIN_KEY` を
  変更するだけで、全端末で再ログインが必要になります。

## 補足

- 機能はシフトボード（board.html v3.3）と同一です: 月/週/日ビュー、LINE希望の反映、
  日ガントのドラッグ編集（下書き→まとめて保存）、前月コピー。
- スタッフ向けのLINE申請フォーム（index.html）・LIFF版ボードはこれまで通り動きます。
