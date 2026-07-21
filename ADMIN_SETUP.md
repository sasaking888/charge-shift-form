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

- 入口（TOP）: `https://sasaking888.github.io/charge-shift-form/top.html`
- シフト管理: `https://sasaking888.github.io/charge-shift-form/admin.html`（main反映後）
- 初回に管理キーを入力するとブラウザに保存され、次回から自動ログインします。
- 共有PCなどでは右上の「ログアウト」でキーを消せます。
- キーを変えたいとき（漏れた・店長交代など）は、スクリプトプロパティの `ADMIN_KEY` を
  変更するだけで、全端末で再ログインが必要になります。

## 補足

- 機能はシフトボード（board.html v3.3）と同一です: 月/週/日ビュー、LINE希望の反映、
  日ガントのドラッグ編集（下書き→まとめて保存）、前月コピー。
- スタッフ向けのLINE申請フォーム（index.html）・LIFF版ボードはこれまで通り動きます。

## 4. スタッフ管理タブ用のAPI追加（addStaff / updateStaff）

admin.html に「スタッフ」タブを追加しました。氏名・雇用形態・時給・希望シフト収集の
ON/OFF・希望する曜日/時間帯を、スプレッドシートを直接触らずに画面から登録・編集できます。
これを動かすには、GAS側に2つのアクションを追加してください。

### スタッフシートに列を追加

既存の「スタッフ」シート（`D.staff` を返している元シート）に、以下の列がなければ追加します。
列名・並び順は既存の実装に合わせて調整してください。

| 列 | 説明 |
|---|---|
| employmentType | 雇用形態（アルバイト/パート/社員/業務委託/その他） |
| requestCollection | 希望シフト収集の対象か（TRUE/FALSE） |
| preferredDays | 希望する曜日。カンマ区切りの数値文字列。0=月〜6=日（例: `0,2,4` = 月水金） |
| preferredStart | 希望する時間帯の開始（時刻の小数表現。17.5 = 17:30） |
| preferredEnd | 希望する時間帯の終了（同上） |

`boardData` のレスポンスに含まれる各スタッフのJSONに、この5項目をそのまま含めてください
（`preferredDays` は配列でもカンマ区切り文字列でもフロント側で吸収します）。

### doPost に action 分岐を追加

```js
// === スタッフ追加 ===
if (payload.action === 'addStaff') {
  var s = payload.staff;
  // スタッフシートに新規行を追加。id は他の項目同様に採番してください。
  // 追加後は { ok: true, data: {} } を返す（admin.html は成功後に再読込します）
  var newId = addStaffRow_(s); // 例: 実装に合わせた関数名で
  return ContentService.createTextOutput(JSON.stringify({ ok: true, data: { id: newId } }))
    .setMimeType(ContentService.MimeType.JSON);
}

// === スタッフ更新 ===
if (payload.action === 'updateStaff') {
  var sid = payload.staffId;
  var s2 = payload.staff;
  // staffId の行を見つけて、name/storeId/employmentType/wage/requestCollection/
  // preferredDays/preferredStart/preferredEnd を上書きしてください。
  updateStaffRow_(sid, s2); // 例: 実装に合わせた関数名で
  return ContentService.createTextOutput(JSON.stringify({ ok: true, data: {} }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

送られてくるペイロードの形:

```json
{
  "action": "addStaff",  // または "updateStaff"（この場合は staffId も付く）
  "userId": "ADMIN",
  "staffId": "sf1",       // updateStaff のときのみ
  "staff": {
    "name": "山田太郎",
    "storeId": "st1",
    "employmentType": "アルバイト",
    "wage": 1100,
    "requestCollection": true,
    "preferredDays": [0, 2, 4],
    "preferredStart": 17,
    "preferredEnd": 22
  },
  "adminKey": "（管理キー）"
}
```

管理キー検証（手順1）が `action` 分岐より前に実行されていれば、`addStaff`/`updateStaff`
も自動的に管理者だけに絞られます。追加のガードは不要です。

最後に、追加した行を保存したら**必ず再デプロイ**してください（手順3と同じ「新バージョン」の
やり方）。
