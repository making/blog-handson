## Blog APIの実装およびデプロイ

まずはブログシステムのAPI部分から実装します。

大方の部分は既に実装済みです。足りない部分をテストドリブンで埋めていきます。

### プロジェクトの作成


https://github.com/making/demo-blog-api

をフォークして、クローンしてください。

![image](https://user-images.githubusercontent.com/106908/35192495-24d1e186-fed7-11e7-8ff6-5e7b4305562f.png)

```
GIT_USER=<your git account>
git clone git@github.com:${GIT_USER}/demo-blog-api.git
```

クローンしたプロジェクトをIDEにインポートしてください。


### テストの実行

インポートしたプロジェクトのテストディレクトリを右クリックしてテストを実行してください。（下図はIntelliJ IDEの例）

![image](https://user-images.githubusercontent.com/106908/35192523-b092613c-fed7-11e7-8a6a-4701561d2bcd.png)


`EntryControllerTest`と`WebhookControllerTest`がエラーになるでしょう。

![image](https://user-images.githubusercontent.com/106908/35192552-12233a84-fed8-11e7-8bdf-e946c6692d1f.png)


### 演習1: `EntryController`の実装


[`demo-blog-api/src/main/java/com/example/blog/entry/EntryController.java`](https://github.com/making/demo-blog-api/blob/master/src/main/java/com/example/blog/entry/EntryController.java)

を開き、`TODO`の部分を確認し、実装してください。


> **ヒント**: 
> 
> `EntryRepository`のメソッドを呼び出して下さい。404エラーを返したい場合は`new ResponseStatusException(HttpStatus.NOT_FOUND, "error message")"`をスローすれば良いです。


実装できたら、`EntryControllerTest`をテストしてください。全てグリーンになれば正解です。

![image](https://user-images.githubusercontent.com/106908/35192604-3f7a794c-fed9-11e7-9e1c-084abbc27b27.png)


### 演習2: `WebHookController`の実装

[`demo-blog-api/src/main/java/com/example/blog/webhook/WebhookController.java`](https://github.com/making/demo-blog-api/blob/master/src/main/java/com/example/blog/webhook/WebhookController.java)

を開き、`TODO`の部分を確認し、実装してください。


> **ヒント**: 
> 
> `Flux<EntryId> added`, `Flux<EntryId> modified`, `Flux<EntryId> removed`それぞれに`doOnNext`メソッドを追加して、それぞれ`EntryCreateEvent`、`EntryUpdateEvent`、`EntryDeleteEvent`を作成して、`ApplicationEventPublisher`から`publish`してください。

実装できたら、`WebHookControllerTest`をテストしてください。全てグリーンになれば正解です。

![image](https://user-images.githubusercontent.com/106908/35192678-b4526e5e-feda-11e7-97d3-4af4397c92f4.png)


### アプリケーションの実行


`com.example.blog.DemoBlogApiApplication`の起動時オプションとして、プログラム引数に

`blog.github.webhook-secret`(Webhook用の秘密キー。任意の値で良いですが、次のテストデータを試すには`foofoo`を入力してください。)と`blog.github.access-token`(GitHubのアクセストークン)を設定して実行してください。

![image](https://user-images.githubusercontent.com/106908/35192759-52315504-fedb-11e7-8565-7de4b1136135.png)


#### 記事全件取得APIの動作確認

[http://localhost:8080/v1/entries](http://localhost:8080/v1/entries)にアクセスしてください。

![image](https://user-images.githubusercontent.com/106908/35192818-7984029a-fedc-11e7-8004-3b6c47c8641e.png)


#### 記事1件取得APIの動作確認

[http://localhost:8080/v1/entries/100](http://localhost:8080/v1/entries/100)にアクセスしてください。

![image](https://user-images.githubusercontent.com/106908/35192820-85212d08-fedc-11e7-8030-ad8251e822db.png)


#### Webhookの動作確認

`src/test`以下にWebhookのテスト用ペイロードとリクエストを送るシェルスクリプトが用意されています。


記事作成のWebhookは次のコマンドで試せます。

```
cd demo-blog-api
./src/test/resources/sample-create-request.sh 
```

`[{"added":497}]`というレスポンスが返れば、記事1件取得API([http://localhost:8080/v1/entries/497](http://localhost:8080/v1/entries/497))で内容を確認してください。


`{"timestamp":"2018-01-21T09:55:49.085+0000","status":403,"error":"Forbidden","message":"Could not verify signature: 'sha1=6ff50ec0e2f69d5831d8a5a88570be819b18515a'","path":"/webhook"}`というレスポンスが返る場合は、
`blog.github.webhook-secret`が正しくないです。`foofoo`を設定してください。

記事更新のWebhookは次のコマンドで試せます。

```
./src/test/resources/sample-update-request.sh
```

`[{"modified":497}]`というレスポンスが返れば正しいです。

### Cloud Foundryへのデプロイ
