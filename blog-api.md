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

まずはビルドして実行可能jarを作成してください。

```
./mvn clean package
```

Pivotal Web Servicesにログインをしていない場合は、次のコマンドでログインしてください。

```
cf login -a api.run.pivotal.io
```

次の`manifest.yml`を作成してください。`route`の値は重複しないように自分のアカウント等を含めてください。

``` yaml
applications:
- name: blog-api
  path: target/demo-blog-api-0.0.1-SNAPSHOT.jar
  routes:
  - route: blog-api-<your account>.cfapps.io
  env:
    BLOG_GITHUB_ACCESS_TOKEN: <GitHubのAccess Token>
    BLOG_GITHUB_WEBHOOK_SECRET: <任意の値>
```

次のコマンドでデプロイできます。


```
cf push
```

次のようなログが出力されます。

```
$ cf  push
Using manifest file /Users/maki/git/demo-blog-api/manifest.yml

Creating app blog-api in org APJ / space development as tmaki@pivotal.io...
OK

Creating route blog-api-tmaki.cfapps.io...
OK

Binding blog-api-tmaki.cfapps.io to blog-api...
OK

Uploading blog-api...
Uploading app files from: /var/folders/9n/34xf4kbd1nl_8__3ghkl8_kc0000gn/T/unzipped-app744606369
Uploading 729.8K, 137 files
Done uploading               
OK

Starting app blog-api in org APJ / space development as tmaki@pivotal.io...
Downloading binary_buildpack...
Downloading staticfile_buildpack...
Downloading java_buildpack...
Downloading ruby_buildpack...
Downloading nodejs_buildpack...
Downloaded nodejs_buildpack
Downloading go_buildpack...
Downloaded staticfile_buildpack
Downloading python_buildpack...
Downloaded go_buildpack
Downloading php_buildpack...
Downloaded java_buildpack
Downloading dotnet_core_buildpack...
Downloaded ruby_buildpack
Downloading dotnet_core_buildpack_beta...
Downloaded binary_buildpack
Downloaded python_buildpack
Downloaded php_buildpack
Downloaded dotnet_core_buildpack
Downloaded dotnet_core_buildpack_beta
Creating container
Successfully created container
Downloading app package...
Downloaded app package (27.3M)
-----> Java Buildpack v4.5 (offline) | https://github.com/cloudfoundry/java-buildpack.git#ffeefb9
-----> Downloading Jvmkill Agent 1.10.0_RELEASE from https://java-buildpack.cloudfoundry.org/jvmkill/trusty/x86_64/jvmkill-1.10.0_RELEASE.so (found in cache)
-----> Downloading Open Jdk JRE 1.8.0_141 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_141.tar.gz (found in cache)
       Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.4s)
-----> Downloading Open JDK Like Memory Calculator 3.9.0_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-3.9.0_RELEASE.tar.gz (found in cache)
       Loaded Classes: 16816, Threads: 300
-----> Downloading Client Certificate Mapper 1.2.0_RELEASE from https://java-buildpack.cloudfoundry.org/client-certificate-mapper/client-certificate-mapper-1.2.0_RELEASE.jar (found in cache)
-----> Downloading Container Security Provider 1.8.0_RELEASE from https://java-buildpack.cloudfoundry.org/container-security-provider/container-security-provider-1.8.0_RELEASE.jar (found in cache)
-----> Downloading Spring Auto Reconfiguration 1.12.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.12.0_RELEASE.jar (found in cache)
Exit status 0
Uploading droplet, build artifacts cache...
Uploading build artifacts cache...
Uploading droplet...
Uploaded build artifacts cache (131B)
Uploaded droplet (73.6M)
Uploading complete
Stopping instance 37ce9828-a30c-479a-910f-3441b97f18e0
Destroying container
Successfully destroyed container

0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
1 of 1 instances running

App started


OK

App blog-api was started using this command `JAVA_OPTS="-agentpath:$PWD/.java-buildpack/open_jdk_jre/bin/jvmkill-1.10.0_RELEASE=printHeapHistogram=1 -Djava.io.tmpdir=$TMPDIR -Djava.ext.dirs=$PWD/.java-buildpack/container_security_provider:$PWD/.java-buildpack/open_jdk_jre/lib/ext -Djava.security.properties=$PWD/.java-buildpack/security_providers/java.security $JAVA_OPTS" && CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-3.9.0_RELEASE -totMemory=$MEMORY_LIMIT -stackThreads=300 -loadedClasses=17525 -poolType=metaspace -vmOptions="$JAVA_OPTS") && echo JVM Memory Configuration: $CALCULATED_MEMORY && JAVA_OPTS="$JAVA_OPTS $CALCULATED_MEMORY" && SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.JarLauncher`

Showing health and status for app blog-api in org APJ / space development as tmaki@pivotal.io...
OK

requested state: started
instances: 1/1
usage: 1G x 1 instances
urls: blog-api-tmaki.cfapps.io
last uploaded: Sun Jan 21 15:54:31 UTC 2018
stack: cflinuxfs2
buildpack: client-certificate-mapper=1.2.0_RELEASE container-security-provider=1.8.0_RELEASE java-buildpack=v4.5-offline-https://github.com/cloudfoundry/java-buildpack.git#ffeefb9 java-main java-opts jvmkill-agent=1.10.0_RELEASE open-jdk-like-jre=1.8.0_1...

     state     since                    cpu      memory         disk           details
#0   running   2018-01-22 12:55:47 AM   174.4%   290.5M of 1G   154.5M of 1G
```

Cloud Foundry上では`cloud`プロファイルが有効になるため、`DemoInserter`は動作しません。`entry`テーブルは空です。

[https://blog-api-your-account.cfapps.io/v1/entries](https://blog-api-your-account.cfapps.io/v1/entries)にアクセスすると空のリストが返ります。

#### MySQLのバインド

バックエンドのデータベースをMySQLに切り替えます。

MySQLのサービスブローカーである`cleardb`の`spark`プラン(free)で`blog-db`という名前のサービスインスタンスを作成します。

```
cf create-service cleardb spark blog-db
```

次のように`manifest.yml`に`services`を追加してください。

``` yaml
applications:
- name: blog-api
  path: target/demo-blog-api-0.0.1-SNAPSHOT.jar
  routes:
  - route: blog-api-<your account>.cfapps.io
  env:
    BLOG_GITHUB_ACCESS_TOKEN: <GitHubのAccess Token>
    BLOG_GITHUB_WEBHOOK_SECRET: <任意の値>
  services:
  - blog-db
```

再度`cf push`してください。Maria DBのJDBCドライバーが自動で含まれます。


```
$ cf push
Using manifest file /Users/maki/git/demo-blog-api/manifest.yml


Updating app blog-api in org APJ / space development as tmaki@pivotal.io...
OK

Using route blog-api-tmaki.cfapps.io
Uploading blog-api...
Uploading app files from: /var/folders/9n/34xf4kbd1nl_8__3ghkl8_kc0000gn/T/unzipped-app531234602
Uploading 729.8K, 137 files
Done uploading               
OK
Binding service blog-db to app blog-api in org APJ / space development as tmaki@pivotal.io...
OK

Stopping app blog-api in org APJ / space development as tmaki@pivotal.io...
OK

Starting app blog-api in org APJ / space development as tmaki@pivotal.io...
Downloading binary_buildpack...
Downloading staticfile_buildpack...
Downloading java_buildpack...
Downloading ruby_buildpack...
Downloading nodejs_buildpack...
Downloaded ruby_buildpack
Downloaded java_buildpack
Downloading python_buildpack...
Downloading go_buildpack...
Downloaded binary_buildpack
Downloading php_buildpack...
Downloaded nodejs_buildpack
Downloading dotnet_core_buildpack...
Downloaded staticfile_buildpack
Downloading dotnet_core_buildpack_beta...
Downloaded php_buildpack
Downloaded python_buildpack
Downloaded go_buildpack
Downloaded dotnet_core_buildpack
Downloaded dotnet_core_buildpack_beta
Creating container
Successfully created container
Downloading build artifacts cache...
Downloading app package...
Downloaded build artifacts cache (131B)
Downloaded app package (27.3M)
-----> Java Buildpack v4.5 (offline) | https://github.com/cloudfoundry/java-buildpack.git#ffeefb9
-----> Downloading Jvmkill Agent 1.10.0_RELEASE from https://java-buildpack.cloudfoundry.org/jvmkill/trusty/x86_64/jvmkill-1.10.0_RELEASE.so (found in cache)
-----> Downloading Open Jdk JRE 1.8.0_141 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_141.tar.gz (found in cache)
       Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.4s)
-----> Downloading Open JDK Like Memory Calculator 3.9.0_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-3.9.0_RELEASE.tar.gz (found in cache)
       Loaded Classes: 16816, Threads: 300
-----> Downloading Client Certificate Mapper 1.2.0_RELEASE from https://java-buildpack.cloudfoundry.org/client-certificate-mapper/client-certificate-mapper-1.2.0_RELEASE.jar (found in cache)
-----> Downloading Container Security Provider 1.8.0_RELEASE from https://java-buildpack.cloudfoundry.org/container-security-provider/container-security-provider-1.8.0_RELEASE.jar (found in cache)
-----> Downloading Maria Db JDBC 2.1.0 from https://java-buildpack.cloudfoundry.org/mariadb-jdbc/mariadb-jdbc-2.1.0.jar (found in cache)
-----> Downloading Spring Auto Reconfiguration 1.12.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.12.0_RELEASE.jar (found in cache)
Exit status 0
Uploading droplet, build artifacts cache...
Uploading build artifacts cache...
Uploading droplet...
Uploaded build artifacts cache (131B)
Uploaded droplet (74.1M)
Uploading complete
Stopping instance e37ef977-7989-41e2-b2be-e18b8df03d9d
Destroying container
Successfully destroyed container

0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
1 of 1 instances running

App started


OK

App blog-api was started using this command `JAVA_OPTS="-agentpath:$PWD/.java-buildpack/open_jdk_jre/bin/jvmkill-1.10.0_RELEASE=printHeapHistogram=1 -Djava.io.tmpdir=$TMPDIR -Djava.ext.dirs=$PWD/.java-buildpack/container_security_provider:$PWD/.java-buildpack/open_jdk_jre/lib/ext -Djava.security.properties=$PWD/.java-buildpack/security_providers/java.security $JAVA_OPTS" && CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-3.9.0_RELEASE -totMemory=$MEMORY_LIMIT -stackThreads=300 -loadedClasses=17672 -poolType=metaspace -vmOptions="$JAVA_OPTS") && echo JVM Memory Configuration: $CALCULATED_MEMORY && JAVA_OPTS="$JAVA_OPTS $CALCULATED_MEMORY" && SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.JarLauncher`

Showing health and status for app blog-api in org APJ / space development as tmaki@pivotal.io...
OK

requested state: started
instances: 1/1
usage: 1G x 1 instances
urls: blog-api-tmaki.cfapps.io
last uploaded: Sun Jan 21 16:15:42 UTC 2018
stack: cflinuxfs2
buildpack: client-certificate-mapper=1.2.0_RELEASE container-security-provider=1.8.0_RELEASE java-buildpack=v4.5-offline-https://github.com/cloudfoundry/java-buildpack.git#ffeefb9 java-main java-opts jvmkill-agent=1.10.0_RELEASE maria-db-jdbc=2.1.0 open-...

     state     since                    cpu      memory         disk         details
#0   running   2018-01-22 01:17:07 AM   143.0%   353.1M of 1G   155M of 1G
```

### ブログ記事の作成

いよいよブログ記事を作成します。GitHubで新規レポジトリを作成してください。

![image](https://user-images.githubusercontent.com/106908/35196207-7176d786-ff12-11e7-8934-41c03a206d2b.png)

レポジトリ名は任意の値で構いません。ここでは`my-blog`として説明します。"Initialize this repository with a README"にチェックを入れて下さい。

![image](https://user-images.githubusercontent.com/106908/35196247-d8b5581e-ff12-11e7-8852-60b9c005f23c.png)


"Settings"をクリックして、"Webhooks"をクリックし、"Add webhook"をクリックして下さい。

![image](https://user-images.githubusercontent.com/106908/35196316-a700979c-ff13-11e7-8fe6-2cdbe12b5ebd.png)

* "Payload URL"には`https://blog-api-<your account>.cfapps.io/webhook`
* "Content type"には`application/json`
* "Secret"には`BLOG_GITHUB_WEBHOOK_SECRET`で設定した値

を設定し、"Add webhook"ボタンをクリックして下さい。

![image](https://user-images.githubusercontent.com/106908/35196359-51245362-ff14-11e7-9b49-e5426b3a783b.png)


Topに戻って、"Create new file"をクリックして下さい。

![image](https://user-images.githubusercontent.com/106908/35196299-5af8748c-ff13-11e7-8f4c-00cf86c4a9b1.png)

ファイル名に`content/00001.md`を入力して下さい。

![image](https://user-images.githubusercontent.com/106908/35196454-8d3a5a26-ff15-11e7-80a5-250939416d24.png)

ファイルコンテンツに次の内容を入力して下さい。

``` markdown
---
title: First article
tags: ["Demo"]
categories: ["Demo", "Hello"]
---

This is my first blog post!
```

![image](https://user-images.githubusercontent.com/106908/35196464-b08bd144-ff15-11e7-89e7-87758b67fdba.png)

"Commit new file"ボタンをクリックして下さい。

Webhook画面に戻って、最新のWebhookが✅になっていることを確認して下さい。

![image](https://user-images.githubusercontent.com/106908/35196515-5c40daca-ff16-11e7-9d01-c438b8a10bee.png)

これでデータベースが更新されているので、
[https://blog-api-your-account.cfapps.io/v1/entries](https://blog-api-your-account.cfapps.io/v1/entries)にアクセスすると作成した記事が返ります。

![image](https://user-images.githubusercontent.com/106908/35196505-4599b4c2-ff16-11e7-8b39-6a4208cb6a3a.png)

Github上で記事を更新すると、[https://blog-api-your-account.cfapps.io/v1/entries](https://blog-api-your-account.cfapps.io/v1/entries)の結果も反映されることを確認して下さい。

### [補足] 外部のMySQLを使用する

### [補足] DB更新処理を行うスレッドを指定する
