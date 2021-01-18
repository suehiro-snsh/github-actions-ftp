# Github Actionsを用いたpush時の自動デプロイ

Githubのmasterブランチにpushした際に自動デプロイが発火するようにする。

## 1.必要なファイル
```
.github/
  workfiles
    master.yml
.git-ftp-include
.gitignore
```

※.gitignoreがなければ、.git-ftp-includeは不要

## 2.手順
### 1. master.ymlの設定
```
name: master

on:
  push:
    branches:
      - master

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2.1.0

      - name: Deploy via FTP
        uses: SamKirkland/FTP-Deploy-Action@master
        with:
          server: ${{ secrets.FTP_SERVER }}
          username: ${{ secrets.FTP_USERNAME }}
          password: ${{ secrets.FTP_PASSWORD }}
          local-dir: ./dist/
          server-dir: /web/clients/z_suehiro/
```
| オプション名 | 説明 |
| ---- | ---- |
| local-dir  | 指定したフォルダ以下がアップロードされる |
| server-dir | 指定したフォルダ以下にアップロードされる |
* local-dir, server-dirは`/`で終わる
* local-dirはプロジェクトルートからのパス
* server, username, passwordはgithub管理画面から設定

### 2. server, username, passwordの設定
ftpの接続情報である、server, username, passwordはセキュリティを考慮し、githubの管理画面から設定します。
  1. Settingsを開く
  2. 左側のSecretsを開く
  3. New repository secretにserver, username, passwordをそれぞれ設定していく  
    ※master.ymlの`${{ secrets.FTP_SERVER }}`の`FTP_SERVER`をName、情報をValueとする
    
## 3. git-ftp-includeの設定
.gitignoreに設定しているものを、自動デプロイに含めたいときに設定する

srcを開発フォルダ、distをビルド結果の出力先とし、distをgitignoreすることが多いと思うのでその例。
```
!dist
```

## 注意点、その他
クライアントの本番サーバーで初めて行う際は、ダミーフォルダを作って一度テストをしましょう。

その他の設定などは以下を参照してください。
https://github.com/SamKirkland/FTP-Deploy-Action
