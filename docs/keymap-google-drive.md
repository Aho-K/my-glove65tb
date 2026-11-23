# キーマップ画像を Google Drive に保存する

`Draw Keymap` の描画が成功すると、その実行で生成した `glove65tb.svg` を指定フォルダに保存します。以降は同じファイルを更新するので、Google Drive のファイルURLを引き続き使えます。対象ブランチを設定するまではアップロードジョブをスキップします。

## 初回設定

1. [rclone の手順](https://rclone.org/drive/#making-your-own-client-id)に沿って、Google Cloud で Google Drive API を有効にし、デスクトップアプリ用の OAuth クライアントを作ります。個人アカウントで継続運用する場合は、同意画面を **Testing** のままにすると更新トークンが7日で失効するため、**In production** にしてから認証します。
2. 手元のPCで `rclone config` を実行し、Google Drive 用のリモートを `keymap` という名前で作成します。作成した OAuth クライアントの `client_id` と `client_secret` を入力し、スコープは `drive` を選びます。ブラウザで保存先フォルダに書き込める Google アカウントを選びます。`drive` は認証したアカウントの Drive 全体への権限です。連携専用のアカウントを使えば、保存先フォルダだけをそのアカウントに共有できます。
3. GitHub の **Settings → Secrets and variables → Actions** の **Secrets** に次を登録します。

   | Secret | 値 |
   | --- | --- |
   | `KEYMAP_DRIVE_RCLONE_CONFIG` | `rclone config file` が示す設定ファイルのうち、`[keymap]` から始まるセクション全体 |
   | `KEYMAP_DRIVE_FOLDER_ID` | 保存先 URL の `/folders/` に続く ID |

   設定には認証情報が含まれます。GitHub の Secret に直接入力してください。
4. 同じ設定画面の **Variables** に `KEYMAP_DRIVE_BRANCH` を追加し、画像を保存するブランチ名を設定します。`main` の配列を使うなら `main` を指定します。作業ブランチを指定する場合は、そのブランチにもアップロードジョブの変更を取り込んでください。

## 動作確認

1. GitHub Actions の **Draw Keymap → Run workflow** で設定したブランチを選び、実行します。描画と `upload-to-drive` が成功し、指定フォルダに `glove65tb.svg` が表示されることを確認します。
2. 同じブランチで次にキーマップを変更したとき、Drive 上の同じファイルの内容が更新され、URL とファイル数が変わらないことを確認します。

アップロードを止める場合は `KEYMAP_DRIVE_BRANCH` を削除します。
