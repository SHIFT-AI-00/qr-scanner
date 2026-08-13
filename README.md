# qr-scanner — 当日QR受付スキャナー画面

GitHub Pages で配信する受付スタッフ用のQRスキャナー（単一ページ）。
出席登録の実処理は GAS Web App 側が行い、この画面は読み取りとAPI呼び出し・結果表示だけを担う。

## 使い方（受付スタッフ）

1. スマホのブラウザで下記URLを開く（`{デプロイメントID}` は当日配布されるGASのデプロイメントID）

   ```
   https://shift-ai-00.github.io/qr-scanner/?id={デプロイメントID}
   ```

2. カメラの使用を許可する
3. 画面を一度タップする（スキャン音が有効になる。タップしなくても動作するが音は鳴らない）
4. 参加者のQRコードをかざす → バイブ＋音＋緑の「スキャン成功！」で読み取り確定
5. 登録結果が表示されたら「次をスキャン」で次の人へ

`?id=` を付けずに開くと「URLにIDが指定されていません」と表示される。

## ファイル構成

| ファイル | 役割 |
|---|---|
| `index.html` | 画面・スタイル・スキャン処理・GAS API呼び出しのすべて（単一ファイル構成） |
| `vendor/html5-qrcode.min.js` | QR読み取りライブラリ v2.3.7（同梱。**差し替え・バージョン変更禁止**） |

### vendor/ を同梱している理由

以前は `https://unpkg.com/html5-qrcode@2.3.7/...` から読み込んでいたが、unpkg は国内エッジが弱く、
会場Wi-Fi混雑時に初回読み込みが数百ms〜数秒かかっていた。GitHub Pages と同一オリジンに置くことで
この待ちをほぼゼロにしている。CDNから読み直すとタイムラグが再発するので戻さないこと。

ライブラリを更新する場合（原則しない）は npm の同一ファイルを使う:

```bash
npm pack html5-qrcode@2.3.7
tar xzf html5-qrcode-2.3.7.tgz package/html5-qrcode.min.js
cp package/html5-qrcode.min.js vendor/html5-qrcode.min.js
```

## GAS 側とのインターフェース（変更禁止）

GAS 側は別管理。以下の契約で固定しているので、どちらか片方だけを勝手に変えないこと。

| リクエスト | 用途 |
|---|---|
| `{GAS_URL}?action=getEventInfo` | イベント名・日付・予約者数・出席者数の取得 |
| `{GAS_URL}?action=processQRCode&code={QR文字列}` | 出席登録 |

レスポンスJSONのキー: `success` / `message` / `eventName` / `eventDate` / `totalCount` / `attendCount` / `name` / `alreadyAttended` / `group`

## ローカルでの動作確認

```bash
cd /Users/shiftai-pc-074/work/realevent/attendance-system/qr-scanner-github
python3 -m http.server 8123
```

ブラウザで `http://127.0.0.1:8123/index.html?id={デプロイメントID}` を開く。

- `file://` で直接開くとカメラAPI（getUserMedia）が使えないため、必ずHTTPサーバー経由で開く
- スマホ実機で確認する場合はGitHub Pages にデプロイして https で開く（カメラはhttpsか localhost のみ許可）
- 画面右下のデバッグログを見たいときは `index.html` の `debugMode = false` を `true` にする

## 変更履歴

- 2026-08-12: 読み込みタイムラグ対策（ライブラリ同梱・起動時の固定待ち500ms撤去・GASへのpreconnect・
  読み取り時のバイブ／スキャン音追加・「次をスキャン」時の不要なカメラ再起動を廃止）
