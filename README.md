# ブログシステム作成ハンズオン

Spring Boot 2を使って下図のような自分専用ブログシステムを構築するハンズオンです。

Blog APIとBlog UIという二つのWebアプリケーションを実装(穴埋め)して、[Pivotal Web Services](https://run.pivotal.io/)にデプロイします。
ブログの記事はGitHubで管理し、Webhookを使ってデータベースの更新を行います。

![image](https://user-images.githubusercontent.com/106908/35030944-363f5740-fba4-11e7-88a5-b2c387eedc16.png)

## 目次

1. [必要なソフトウェアのインストール](install.md)
1. [(必須!!) 事前準備](prep.md)
1. [Blog APIの実装およびデプロイ](blog-api.md)
1. [Blog UIの実装およびデプロイ](blog-ui.md)
1. PCF MetricsのTrace Exporterで分散トレーシング
1. (時間があれば) HTML5のServer-Sent EventsとNotifications APIを使ってブログ記事の更新通知

## 利用規約

無断で本ドキュメントの一部または全部を改変したり、本ドキュメントを用いた二次的著作物を作成することを禁止します。ただし、ドキュメント修正のためのPull Requestは大歓迎です。
