# ブログシステム作成ハンズオン

Spring Boot 2を使って下図のような自分専用ブログシステムを構築するハンズオンです。
<br>
**ハッシュタグ**: [#bloghol](https://twitter.com/search?f=tweets&q=%23bloghol&src=typd)

Blog APIとBlog UIという二つのWebアプリケーションを実装(穴埋め)して、[Pivotal Web Services](https://run.pivotal.io/)にデプロイします。
ブログの記事はGitHubで管理し、Webhookを使ってデータベースの更新を行います。

![image](https://user-images.githubusercontent.com/106908/35030944-363f5740-fba4-11e7-88a5-b2c387eedc16.png)

## 目次

1. [必要なソフトウェアのインストール](install.md)
1. [(＊＊必須**) 事前準備](prep.md)
1. [Blog APIの実装およびデプロイ](blog-api.md)
1. [Blog UIの実装およびデプロイ](blog-ui.md)
1. [Java Memory Calculatorでメモリの調節](memory-calculator.md)
1. [HTML5のServer-Sent EventsとNotifications APIを使ってブログ記事の更新通知](notification.md)
1. [独自ドメインの利用](create-domain.md)


スケールアウト、ロギング、Blue/GreenデプロイなどCloud Foundryの基本的な使い方は[Cloud Foundryワークショップ資料](https://github.com/pivotal-japan/cf-workshop)を参照してください。<br>
ConcourseによるCI/CDを実践したい場合は、[Concourseワークショップ資料](https://github.com/pivotal-japan/concourse-workshop)を参照してください。

## 利用規約

無断で本ドキュメントの一部または全部を改変したり、本ドキュメントを用いた二次的著作物を作成することを禁止します。ただし、ドキュメント修正のためのPull Requestは大歓迎です。

---

© 2018 Toshiaki Maki
