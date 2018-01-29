## Blog UIの実装およびデプロイ

次はブログシステムのUI部分を実装します。

大方の部分は既に実装済みです。足りない部分をテストドリブンで埋めていきます。

### プロジェクトの作成


https://github.com/making/demo-blog-ui

をフォークして、クローンしてください。

![image](https://user-images.githubusercontent.com/106908/35192499-35e84726-fed7-11e7-851c-77e46f940dec.png)

```
GIT_USER=<your git account>
git clone git@github.com:${GIT_USER}/demo-blog-ui.git
```

クローンしたプロジェクトをIDEにインポートしてください。

### テストの実行

インポートしたプロジェクトのテストディレクトリを右クリックしてテストを実行してください。（下図はIntelliJ IDEの例）

`UiControllerTest`がエラーになるでしょう。

![image](https://user-images.githubusercontent.com/106908/35318337-30c69600-011e-11e8-8764-4b31ba12793c.png)

### 演習3: `UiController`の実装

[demo-blog-ui/src/main/java/com/example/blog/UiController.java](https://github.com/making/demo-blog-ui/blob/master/src/main/java/com/example/blog/UiController.java)


を開き、TODOの部分を確認し、実装してください。

> ヒント:
> 
> `WebClient`を使ってblog-apiの"記事全件取得API"と"記事1件取得API"をそれぞれ呼び出して下さい。
> コンストラクタを見ればわかるように、`WebClient`には`baseUrl`としてblog-apiのurlが設定されています。このプロパティは`application.properties`に設定されています。
> `WebClient`の使い方がわからない場合は、[公式ドキュメント](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-client)を参照して下さい。


実装できたら、`UiControllerTest`をテストしてください。全てグリーンになれば正解です。

![image](https://user-images.githubusercontent.com/106908/35318670-af8e56f2-011f-11e8-8aa5-3cbda470de7e.png)

### アプリケーションの実行


blog-apiが起動した状態で、`com.example.blog.DemoBlogUiApplication`を実行して下さい。

#### 一覧画面の動作確認

[http://localhost:8082](http://localhost:8082)にアクセスしてください。

![image](https://user-images.githubusercontent.com/106908/35318772-32e434c2-0120-11e8-9760-18663b5fefe8.png)


#### 記事画面の動作確認

[http://localhost:8082/entries/100](http://localhost:8082/entries/100)にアクセスしてください。


![image](https://user-images.githubusercontent.com/106908/35318797-49006dc0-0120-11e8-8aad-6e08023be77c.png)


### 演習4: HTMLのカスタマイズ

* 一覧画面: [demo-blog-ui/src/main/resources/templates/index.html](https://github.com/making/demo-blog-ui/blob/master/src/main/resources/templates/index.html)
* 記事画面: [demo-blog-ui/src/main/resources/templates/entry.html](https://github.com/making/demo-blog-ui/blob/master/src/main/resources/templates/entry.html)

を開き、HTMLを編集して好みのデザインにしてください。

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
- name: blog-ui
  path: target/demo-blog-ui-0.0.1-SNAPSHOT.jar
  routes:
  - route: blog-ui-<your account>.cfapps.io
  env:
    BLOG_API_URI: https://blog-api-<your account>.cfapps.io
```

次のコマンドでデプロイできます。

```
cf push
```

次のようなログが出力されます。

```
$ cf push
Using manifest file /Users/maki/git/demo-blog-ui/manifest.yml

Creating app blog-ui in org APJ / space development as tmaki@pivotal.io...
OK

Creating route blog-ui-tmaki.cfapps.io...
OK

Binding blog-ui-tmaki.cfapps.io to blog-ui...
OK

Uploading blog-ui...
Uploading app files from: /var/folders/9n/34xf4kbd1nl_8__3ghkl8_kc0000gn/T/unzipped-app156799962
Uploading 1.1M, 122 files
Done uploading               
OK

Starting app blog-ui in org APJ / space development as tmaki@pivotal.io...
Downloading binary_buildpack...
Downloading nodejs_buildpack...
Downloading staticfile_buildpack...
Downloading java_buildpack...
Downloading ruby_buildpack...
Downloaded binary_buildpack
Downloading php_buildpack...
Downloaded staticfile_buildpack
Downloading go_buildpack...
Downloaded ruby_buildpack
Downloading python_buildpack...
Downloaded go_buildpack
Downloading dotnet_core_buildpack...
Downloaded dotnet_core_buildpack
Downloading dotnet_core_buildpack_beta...
Downloaded dotnet_core_buildpack_beta
Downloaded python_buildpack
Downloaded php_buildpack
Downloaded java_buildpack
Downloaded nodejs_buildpack
Creating container
Successfully created container
Downloading app package...
Downloaded app package (22M)
-----> Java Buildpack v4.5 (offline) | https://github.com/cloudfoundry/java-buildpack.git#ffeefb9
-----> Downloading Jvmkill Agent 1.10.0_RELEASE from https://java-buildpack.cloudfoundry.org/jvmkill/trusty/x86_64/jvmkill-1.10.0_RELEASE.so (found in cache)
-----> Downloading Open Jdk JRE 1.8.0_141 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_141.tar.gz (found in cache)
       Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.5s)
-----> Downloading Open JDK Like Memory Calculator 3.9.0_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-3.9.0_RELEASE.tar.gz (found in cache)
       Loaded Classes: 15787, Threads: 300
-----> Downloading Client Certificate Mapper 1.2.0_RELEASE from https://java-buildpack.cloudfoundry.org/client-certificate-mapper/client-certificate-mapper-1.2.0_RELEASE.jar (found in cache)
-----> Downloading Container Security Provider 1.8.0_RELEASE from https://java-buildpack.cloudfoundry.org/container-security-provider/container-security-provider-1.8.0_RELEASE.jar (found in cache)
-----> Downloading Spring Auto Reconfiguration 1.12.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.12.0_RELEASE.jar (found in cache)
Exit status 0
Uploading droplet, build artifacts cache...
Uploading droplet...
Uploading build artifacts cache...
Uploaded build artifacts cache (131B)
Uploaded droplet (68.4M)
Uploading complete
Stopping instance 3231cd37-33a7-498b-85ff-be2b65eb97cd
Destroying container
Successfully destroyed container

0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
1 of 1 instances running

App started


OK

App blog-ui was started using this command `JAVA_OPTS="-agentpath:$PWD/.java-buildpack/open_jdk_jre/bin/jvmkill-1.10.0_RELEASE=printHeapHistogram=1 -Djava.io.tmpdir=$TMPDIR -Djava.ext.dirs=$PWD/.java-buildpack/container_security_provider:$PWD/.java-buildpack/open_jdk_jre/lib/ext -Djava.security.properties=$PWD/.java-buildpack/security_providers/java.security $JAVA_OPTS" && CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-3.9.0_RELEASE -totMemory=$MEMORY_LIMIT -stackThreads=300 -loadedClasses=16497 -poolType=metaspace -vmOptions="$JAVA_OPTS") && echo JVM Memory Configuration: $CALCULATED_MEMORY && JAVA_OPTS="$JAVA_OPTS $CALCULATED_MEMORY" && SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.JarLauncher`

Showing health and status for app blog-ui in org APJ / space development as tmaki@pivotal.io...
OK

requested state: started
instances: 1/1
usage: 1G x 1 instances
urls: blog-ui-tmaki.cfapps.io
last uploaded: Wed Jan 24 07:16:09 UTC 2018
stack: cflinuxfs2
buildpack: client-certificate-mapper=1.2.0_RELEASE container-security-provider=1.8.0_RELEASE java-buildpack=v4.5-offline-https://github.com/cloudfoundry/java-buildpack.git#ffeefb9 java-main java-opts jvmkill-agent=1.10.0_RELEASE open-jdk-like-jre=1.8.0_1...

     state     since                    cpu      memory         disk           details
#0   running   2018-01-24 04:17:33 PM   180.5%   300.5M of 1G   148.8M of 1G
```


[https://blog-ui-your-account.cfapps.io](https://blog-ui-your-account.cfapps.io)にアクセスすると一覧画面が表示されます。

![image](https://user-images.githubusercontent.com/106908/35319227-3d26a062-0122-11e8-95bf-b8ac2d71d165.png)

記事画面にもアクセスしてください。

![image](https://user-images.githubusercontent.com/106908/35319235-43c4395c-0122-11e8-95ac-c8544b49e8ff.png)

### [補足] メモリを節約する

`manifest.yml`を次のように変更し、コンテナメモリサイズを256MBに減らせます。

``` yaml
applications:
- name: blog-ui
  memory: 256m
  path: target/demo-blog-ui-0.0.1-SNAPSHOT.jar
  routes:
  - route: blog-ui-<your account>.cfapps.io
  env:
    BLOG_API_URI: https://blog-api-<your account>.cfapps.io
    JAVA_OPTS: '-XX:ReservedCodeCacheSize=32M -Xss512k -XX:+PrintCodeCache'
    JBP_CONFIG_OPEN_JDK_JRE: '[memory_calculator: {stack_threads: 24}]' # 4 (core) + 20 (etc)
```
