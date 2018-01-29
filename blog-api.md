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
./mvnw clean package
```

Pivotal Web Servicesにログインをしていない場合は、次のコマンドでログインしてください。

```
cf login -a api.run.pivotal.io
```

次の`manifest.yml`を作成してください。`<your account>`の値は重複しないように自分のアカウント等を含めてください。

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

> `manifest.yml`に機密情報を含めたくない場合は`cf set-env`コマンドでアプリケーションに環境変数を埋め込めます。
>
> ```
> cf set-env blog-api BLOG_GITHUB_ACCESS_TOKEN asdfghujiko
> cf set-env blog-api BLOG_GITHUB_WEBHOOK_SECRET foofoo
> cf restart blog-api
> ```

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

### [補足] メモリを節約する

`manifest.yml`を次のように変更し、コンテナメモリサイズを512MBに減らせます。

``` yaml
applications:
- name: blog-api
  memory: 512m
  path: target/demo-blog-api-0.0.1-SNAPSHOT.jar
  routes:
  - route: blog-api-<your account>.cfapps.io
  env:
    BLOG_GITHUB_ACCESS_TOKEN: <GitHubのAccess Token>
    BLOG_GITHUB_WEBHOOK_SECRET: <任意の値>
    SERVER_TOMCAT_MAX_THREADS: 4
    JAVA_OPTS: '-XX:ReservedCodeCacheSize=32M -Xss512k -XX:+PrintCodeCache'
    JBP_CONFIG_OPEN_JDK_JRE: '[memory_calculator: {stack_threads: 24}]' # 4 (tomcat) + 20 (etc)
  services:
  - blog-db
```

詳細は[Java Memory Calculatorでメモリの調節](memory-calculator.md)を参照してください。

### [補足] 外部のMySQLを使用する

cleardbのsparkプランは貧弱(ディスクサイズ5MB,最大接続数4)なので、他のMySQLサービスを使いたいことが多いです。外部のMySQLサービスを使う場合は、次の2通りあります。

いずれにせよ、まずは`blog-api`から`blog-db`をアンバインドして、`blog-db`サービスインスタンスを削除して下さい。

```
cf unbind-service blog-api blog-db
cf delete-service blog-db
```

#### BuildpackのSpring Auto Reconfigurationを使う場合

BuildpackのSpring Auto Reconfigurationを使う場合はJDBCドライバの設定は不要で、DIコンテナ中の`Datasource`インスタンスも自動で挿し変わります。
設定が不要で便利な一方、`DataSource`の設定を自由に行うことができません。

こちらを使用する場合、User Provided Serviceを次の形式で作成して下さい。

```
cf create-user-provided-service blog-db -p '{"uri":"mysql://username:password@hostname:port/db"}'
```

このあと、再度`cf push`して下さい。

#### BuildpackのSpring Auto Reconfigurationを使わない場合

BuildpackのSpring Auto Reconfigurationを使わない場合はサービスインスタンスは作成せず、環境変数でDBの情報を設定します。
また、JDBCドライバを含めてアプリケーションをビルドし直す必要があります。

`pom.xml`に次の`dependency`を追加して下さい。

``` xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

アプリケーションを再度ビルドして下さい。

```
./mvnw clean package
```

`manifest.yml`の`env`にMySQLの接続情報を記述します。

``` yaml
applications:
- name: blog-api
  path: target/demo-blog-api-0.0.1-SNAPSHOT.jar
  routes:
  - route: blog-api-<your account>.cfapps.io
  env:
    BLOG_GITHUB_ACCESS_TOKEN: <GitHubのAccess Token>
    BLOG_GITHUB_WEBHOOK_SECRET: <任意の値>
    SPRING_DATASOURCE_DRIVER_CLASS_NAME: com.mysql.jdbc.Driver
    SPRING_DATASOURCE_URL: jdbc:mysql://hostname:port/db
    SPRING_DATASOURCE_USERNAME: user
    SPRING_DATASOURCE_PASSWORD: password
```

このあと、再度`cf push`して下さい。こちらの方法のメリットはCloud Foundry(Buildpack)に依存しないPureな12 Factor Appになる点です。

> `manifest.yml`に`SPRING_DATASOURCE_PASSWORD`を含めたくない場合は、別途`cf set-env`を実行して下さい。
>
> ```
> cf set-env blog-api SPRING_DATASOURCE_PASSWORD password
> cf restart blog-api
> ```

MySQLをサービスブローカーで作成しつつ、Spring Auto Reconfigurationを使わずに環境変数でDB接続情報を設定したい場合は、`cf create-service-key`でサービスインスタンスから接続情報だけ作成して表示してください。

```
$ cf create-service cleardb spark blog-db # 未作成の場合
$ cf create-service-key blog-db blog-api-key
$ cf service-key blog-db blog-api-key

{
 "hostname": "us-cdbr-iron-east-04.cleardb.net",
 "jdbcUrl": "jdbc:mysql://us-cdbr-iron-east-04.cleardb.net/ad_bc365753fe5cd77?user=b746d44bae5f3a\u0026password=b5e1d9d7",
 "name": "ad_bc365753fe5cd77",
 "password": "b5e1d9d7",
 "port": "3306",
 "uri": "mysql://b746d44bae5f3a:b5e1d9d7@us-cdbr-iron-east-05.cleardb.net:3306/ad_bc365753fe5cd77?reconnect=true",
 "username": "b746d44bae5f3a"
}
```

得られた`uri`、`username`、`password`をそれぞれ環境変数`SPRING_DATASOURCE_URL`、`SPRING_DATASOURCE_USERNAME`、`SPRING_DATASOURCE_PASSWORD`に設定すれば良いです。`cf bind-service`は**行わないでください**。


### [補足] DB更新処理を行うスレッドを指定する


今回のケースでは問題にはなりませんが、次のコードには改善すべき点があります。(`modified`、`deleted`も同じ)

```java
Flux<EntryId> added = this.paths(commit.get("added"))
        .flatMap(path -> this.entryFetcher.fetch(owner, repo, path)) // (A)
        .doOnNext(e -> this.publisher.publishEvent(new EntryCreateEvent(e))) // (B)
        .map(Entry::entryId);
```

`(A)`では`WebClient`を使用しているため、Reactor Nettyのイベントループスレッドプールが使用されます。
このコードではそのまま`(B)`の処理が行われるため、`(B)`も`(A)`と同じスレッド上で実行されます。<br>
`(B)`ではデータベースアクセスを伴うブロッキングIO処理が行われますが、Nettyのイベントループスレッドは
ノンブロッキングIO処理を想定しており、スレッドプール数はCPU数しかありません。<br>
もしも`(B)`の処理が同時に多数呼ばれるような状況では、この処理がスレッドプールを専有してしまい、
本来ノンブロッキングである`(A)`の`WebClient`の処理が妨げられます。

今回のケースでは`(B)`はWebHook経由でしか呼ばれないため、実質的に問題ありません。<br>
ただし、このようにブロッキング処理がNettyのイベントループスレッドプールで実行されることを避けるには、
明示的に`(B)`を実行するスレッドプールを指定する必要があります。

Blog APIの実装では`com.example.blog.DemoBlogApiApplication`にデータベースアクセス用のスレッドプール`ThreadPoolTaskExecutor`が定義されています。
このスレッドプール数はコネクションプール数と同じであるべきです。適切な値を設定してください。

この`TaskExecutor`を`WebhookController`にインジェクションします。

```java
public class WebhookController {
	private final EntryFetcher entryFetcher;
	private final TaskExecutor taskExecutor; // **追加**
	private final ApplicationEventPublisher publisher;
	private final WebhookVerifier webhookVerifier;
	private final ObjectMapper objectMapper;

	public WebhookController(BlogProperties props, EntryFetcher entryFetcher,
			TaskExecutor taskExecutor /** 追加 **/, ApplicationEventPublisher publisher,
			ObjectMapper objectMapper)
			throws NoSuchAlgorithmException, InvalidKeyException {
		this.entryFetcher = entryFetcher;
		this.taskExecutor = taskExecutor; // **追加**
		this.publisher = publisher;
		this.objectMapper = objectMapper;
		this.webhookVerifier = new WebhookVerifier(props.getGithub().getWebhookSecret());
	}
	/* ... */
}
```

そして、データベースアクセス処理の前に`publishOn`メソッドでこの`TaskExecutor`を使った`reactor.core.scheduler.Scheduler`を指定する必要があります。

```java
Flux<EntryId> added = this.paths(commit.get("added"))
        .flatMap(path -> this.entryFetcher.fetch(owner, repo, path)) // (A)
        .publishOn(Schedulers.fromExecutor(this.taskExecutor)) // (*)
        .doOnNext(e -> this.publisher.publishEvent(new EntryCreateEvent(e))) // (B)
        .map(Entry::entryId);
```

このコードの`(*)`より後のオペレーションは`publishOn`で指定した`Scheduler`で生成されるスレッド上で実行されます。

> `(*)`より前のオペレーションを実行する`Scheduler`を指定したい場合は`subscribeOn`を使用してください。

コード変更前はWebHookで次のログが出力されます。

```
2018-01-30 01:40:03.735 DEBUG 82098 --- [ctor-http-nio-6] r.ipc.netty.http.client.HttpClient       : [id: 0x5be4e902, L:/192.168.11.6:54511 - R:api.github.com/192.30.255.116:443] READ COMPLETE
2018-01-30 01:40:03.766  INFO 82098 --- [ctor-http-nio-4] c.e.b.e.event.EntryCreateEventListener   : Create 497
2018-01-30 01:40:03.766 DEBUG 82098 --- [ctor-http-nio-4] o.s.j.d.DataSourceTransactionManager     : Creating new transaction with name [com.example.blog.entry.EntryRepository.create]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT; ''
2018-01-30 01:40:03.767 DEBUG 82098 --- [ctor-http-nio-4] o.s.j.d.DataSourceTransactionManager     : Acquired Connection [HikariProxyConnection@1208355775 wrapping conn0: url=jdbc:h2:mem:testdb user=SA] for JDBC transaction
2018-01-30 01:40:03.767 DEBUG 82098 --- [ctor-http-nio-4] o.s.j.d.DataSourceTransactionManager     : Switching JDBC Connection [HikariProxyConnection@1208355775 wrapping conn0: url=jdbc:h2:mem:testdb user=SA] to manual commit
2018-01-30 01:40:03.767 DEBUG 82098 --- [ctor-http-nio-4] o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL update
2018-01-30 01:40:03.767 DEBUG 82098 --- [ctor-http-nio-4] o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL statement [INSERT INTO entry(entry_id, title, content, created_by, created_date, last_modified_by, last_modified_date) VALUES(?, ?, ?, ?, ?, ?, ?)]
```

HTTP処理もデータベースアクセス処理も`reactor-http-nio-N`という名前のスレッド上で実行されており、Reactor Nettyのイベントループスレッド上であることがわかります。

コード変更後はWebHookで次のログが出力されます

```
2018-01-30 01:42:55.583 DEBUG 82649 --- [ctor-http-nio-4] r.ipc.netty.http.client.HttpClient       : [id: 0x7c2d0c17, L:/192.168.11.6:54537 - R:api.github.com/192.30.255.117:443] READ COMPLETE
2018-01-30 01:42:55.583  INFO 82649 --- [lTaskExecutor-4] c.e.b.e.event.EntryCreateEventListener   : Create 497
2018-01-30 01:42:55.583 DEBUG 82649 --- [lTaskExecutor-4] o.s.j.d.DataSourceTransactionManager     : Creating new transaction with name [com.example.blog.entry.EntryRepository.create]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT; ''
2018-01-30 01:42:55.584 DEBUG 82649 --- [lTaskExecutor-4] o.s.j.d.DataSourceTransactionManager     : Acquired Connection [HikariProxyConnection@1785931203 wrapping conn0: url=jdbc:h2:mem:testdb user=SA] for JDBC transaction
2018-01-30 01:42:55.584 DEBUG 82649 --- [lTaskExecutor-4] o.s.j.d.DataSourceTransactionManager     : Switching JDBC Connection [HikariProxyConnection@1785931203 wrapping conn0: url=jdbc:h2:mem:testdb user=SA] to manual commit
2018-01-30 01:42:55.584 DEBUG 82649 --- [lTaskExecutor-4] o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL update
2018-01-30 01:42:55.584 DEBUG 82649 --- [lTaskExecutor-4] o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL statement [INSERT INTO entry(entry_id, title, content, created_by, created_date, last_modified_by, last_modified_date) VALUES(?, ?, ?, ?, ?, ?, ?)]
```

今度はHTTP処理は`reactor-http-nio-N`スレッド上ですが、データベースアクセス処理は`threadPoolTaskExecutor-N`スレッド上で実行されています。
これでデータベースアクセス処理にNettyのスレッドプールを使用されることを防げました。当然ですが、その分生成するスレッド数は増えるのでメモリ使用量は増えます。
