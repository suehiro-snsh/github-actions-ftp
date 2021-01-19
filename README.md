# Github Actionsを用いたpush時の自動デプロイ

## 0. 概要
Githubのmasterブランチにpushした際に自動デプロイが発火するようにする  

### 自動デプロイのメリット
* 手動アップロードによる人為的ミスを減らせる
* 本番環境・テスト環境でソースコードの一貫性を保つことができる

## 1.必要なファイル
```
.github/
  workfiles
    master.yml
.gitignore
```

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
          exclude: .git*
            - .git*/**
            -  **/.git*/**
            - node_modules/**
            - node_modules/**/*
            - README.md
            - src
            - src/**
            - src/**/*
```
| オプション名 | 説明 |
| ---- | ---- |
| local-dir  | 指定したフォルダ以下がアップロードされる |
| server-dir | 指定したフォルダ以下にアップロードされる |
* local-dir, server-dirは`/`で終わる
* local-dirはプロジェクトルートからのパス
* server, username, passwordはgithub管理画面から設定

### 2. server, username, passwordの設定
ftpの接続情報である、server, username, passwordはセキュリティを考慮し、githubの管理画面から設定する。
  1. Settingsを開く
  2. 左側のSecretsを開く
  3. New repository secretにserver, username, passwordをそれぞれ設定していく  
    ※master.ymlの`${{ secrets.FTP_SERVER }}`の`FTP_SERVER`をName、情報をValueとする

## 注意点、その他
* `.gitignore`に設定したものは当然デプロイされない。
* `.git-ftp-include`,`.git-ftp-ignore`は以前は使えたようだが、2021年1月現在は使えない。  
そのため、distを`.gitignore`に含めないこと。
* `ftp-deploy-sync-state.json`がデプロイ先のサーバーに作られるがそのままにしておく。
* パスの`/`有無でデプロイ時にエラーがでたりするので気をつける。
* クライアントの本番サーバーで初めて行う際は、ダミーフォルダを作って一度テストをすること。
* SamKirkland/FTP-Deploy-Actionは今も更新されているようなので、たまに`master.yml`に書いてるバージョンも上げる。

その他の設定などは以下を参照。
https://github.com/SamKirkland/FTP-Deploy-Action
