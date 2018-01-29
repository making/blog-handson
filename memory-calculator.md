## Java Memory Calculatorでメモリの調節

Java BuildpackではJava Memory Calculatorによって自動でJVMに割り当てるメモリサイズが計算されます。
デフォルトでは次の式で計算されます。

* Heap Size = Container's `memory` Size - Native Size
* Native Size = `-XX:MaxMetaSpaceSize` + `-XX:ReservedCodeCacheSize` + `-XX:CompressedClassSpaceSize` + `-XX:MaxDirectMemorySize` + (`-Xss` * Number of threads)

各種JVMパラメーターはデフォルトで次の値が設定されます。

* `-XX:MaxMetaSpaceSize`  ... クラスファイル数から**自動で算出**
* `-XX:CompressedClassSpaceSize` ... クラスファイル数から**自動で算出**
* `-XX:ReservervedCodeCacheSize` ... 240MB
* `-XX:DirectMemorySize` ... 10MB
* `-Xss` ... 1MB
* Number of threads ... 300 (Tomcatのデフォルト最大スレッド数200 + その他100)


ヒープサイズを意識することは多いですが、ネイティブサイズを細かく考えられる人は多くないです。
特に、コンテナ環境ではネイティブサイズは無視されがちで、予期せぬOut of Memory Errorを招きます。

Java Buildpackはこのような設定項目を意識しなくても、コンテナのメモリサイズを指定さえすれば、
アップロードされたファイルのクラス数から適切なメモリサイズを設定してくれるため、プラットフォームに設定を**お任せ**することができます。

一方で、ヒープサイズと`MaxMetaSpaceSize`、`CompressedClassSpaceSize`を抜いても

```
240M + 10M + 1M * 300 = 550M
```

のメモリが必要です。通常は768MB〜1GBを設定しないとOut of Memory Errorでアプリケーションが起動しなくなります。
コンテナのメモリが1GBの場合、ヒープサイズに割かれるのはおよそ350MB程度です。

実際に設定されている値は起動時のログから確認可能です。

クラス数とスレッド数はステージングのログに出力されます。

![image](https://user-images.githubusercontent.com/106908/35510377-2db19610-053b-11e8-8758-4263e2e0ab85.png)

JVMへのパラメータはアプリケーションログの先頭に出力されます。

![image](https://user-images.githubusercontent.com/106908/35510357-1c1d8300-053b-11e8-8123-134897d33c9e.png)


プロダクションでは通常1GB以上を設定することが推奨されますが、
Pivotal Web Servicesの無償利用期間に2GB制限があるのと、メモリ従量課金であるため、
不要であればメモリサイズ小さい方が運用費を低く抑えられるのでできるだけ小さく設定したいことがあります。
Pivotal Web Servicesではデフォルトのコンテナメモリサイズは1GBです。

大きく減らすことができるのはスレッドに必要なメモリ数と`ReservervedCodeCacheSize`です。
スレッド数の多くはリクエストを処理するために使用されます。デフォルトが200で想定されているため、
同時にアクセスされる数が少ないと判断できる場合はこの値を大きく減らせます。

`ReservervedCodeCacheSize`はJITコンパイルの結果をキャッシュするメモリサイズです。
この値を小さくするとJITコンパイルが増えCPU使用率が高騰する可能性があります。
パフォーマンスに影響しますが、コストを優先する場合は減らしても良いでしょう。

> JVMオプションで`-XX:+PrintCodeCache`をつけるとJVM終了時にCodeCacheの使用量が出力されます。
> 
> ![image](https://user-images.githubusercontent.com/106908/35508867-56ea108a-0535-11e8-835e-4751f11213fc.png)


Spring MVC (Tomcat)の場合とSpring WebFlux (Netty)の場合でそれぞれメモリの調節方法を説明します。

### Spring MVC (Tomcat)の場合

Spring MVC (Tomcat)の場合は、スレッド数を減らす場合に`server.tomcat.max-threads`を指定して、同時に受け付けられる処理数を制限する必要があります。
指定しないとキャパシティを超えて処理してしまう可能性があります。その場合はOut of Memory Errorでクラッシュするでしょう。

ここではmemoryを500MBに抑えるために、Tomcat側では


`manifest.yml`の`env`を次のように変更してください。

``` yaml
  env:
    SERVER_TOMCAT_MAX_THREADS: 4
    JAVA_OPTS: '-XX:ReservedCodeCacheSize=32M -Xss512k -XX:+PrintCodeCache'
    JBP_CONFIG_OPEN_JDK_JRE: '[memory_calculator: {stack_threads: 24}]' # 4 (tomcat) + 20 (etc)
```

`stack_threads`はJava Memory Calculatorが計算に使用するスレッド数(デフォルト:300)です。

上記の設定により、`ReservedCodeCacheSize` + スレッドメモリサイズ (デフォルト: 240M + 1M * 300 = 540M)は

```
32M + 0.5M * 24 = 44M
```

まで減らせます。これにヒープサイズを16M減らすとコンテナのメモリサイズを合計512M (540 - 44 + 16)減らすことができます。


これで次のように`manifest.yml`を設定可能です。

``` yaml
  memory: 512m
  env:
    SERVER_TOMCAT_MAX_THREADS: 4
    JAVA_OPTS: '-XX:ReservedCodeCacheSize=32M -Xss512k -XX:+PrintCodeCache'
    JBP_CONFIG_OPEN_JDK_JRE: '[memory_calculator: {stack_threads: 24}]' # 4 (tomcat) + 20 (etc)
```

この設定で`cf push`するとステージング時に出力されるスレッド数が24に変更され、

![image](https://user-images.githubusercontent.com/106908/35510303-eaf703fa-053a-11e8-8302-a9e8cf162283.png)

起動時のメモリオプションのログは算出された値のみになります。

![image](https://user-images.githubusercontent.com/106908/35510342-0ee80b60-053b-11e8-8b51-131f74ad55fd.png)


ヒープサイズに余裕がある場合は、ヒープサイズを減らしてスレッドメモリサイズを増やしても構いません。

> Pivotal Web Services上でJConsoleを使う方法は次のリンクを参照してください。
>
> https://discuss.pivotal.io/hc/en-us/articles/221330108-How-to-remotely-monitor-Java-applications-deployed-on-PCF-via-JMX

### Spring WebFlux (Netty)の場合

Spring WebFlux (Netty)の場合はノンブロッキングアーキテクチャであるため、元々少ないスレッド数(CPUコア数)で沢山のリクエストを同時に処理できます。
したがって、デフォルトのスレッド数300は大きすぎでありリソースの無駄です。スレッド数は24-32で十分でしょう。
またヒープサイズもSpring MVCに比べて減らすことができるので、次の設定を使用します。

``` yaml
  memory: 256m
  env:
    JAVA_OPTS: '-XX:ReservedCodeCacheSize=32M -Xss512k -XX:+PrintCodeCache'
    JBP_CONFIG_OPEN_JDK_JRE: '[memory_calculator: {stack_threads: 24}]' # 4 (core) + 20 (etc)
```

同じ4スレッドでもスループットはSpring WebFluxの方が高くなるはずです。

> Spring WebFluxの[Router Function](https://docs.spring.io/spring-framework/docs/5.0.x/spring-framework-reference/web-reactive.html#webflux-fn)フレームワークを使えば、更に低フットプリントなSpring WebFluxアプリを作成可能です。<br>
> 次のMaven Archetypeを使うとDIコンテナを使わずにWebアプリケーションを作成可能です。 
> 
> https://github.com/making/vanilla-spring-webflux-fn-blank
