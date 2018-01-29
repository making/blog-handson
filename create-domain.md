## 独自ドメインの利用

せっかく自分専用のブログができたので、ブログに独自ドメインを設定しましょう。

Cloud FoundryはLoad Balancerで全てのHTTP(s)リクエストを受け付けてGoRouterというコンポーネントに転送します。
GoRouterはHTTPリクエストヘッダの`Host`をみて、該当の`route`をもつアプリケーションのコンテナにリクエストをルーティングします。

![image](https://user-images.githubusercontent.com/106908/35513402-3f97e4a0-0546-11e8-86be-6003fef2ebce.png)

アプリケーションには複数の`route`を設定可能であり、独自ドメインも利用可能です。アプリケーションに独自ドメインの`route`を設定し、
そのドメインがDNSのCNAMEなどでLoad Balancerに向くようになっていれば、`Host`ヘッダはそのまま独自ドメインのものが使用されるため、
そのドメインで対象のアプリケーションにアクセス可能になります。

![image](https://user-images.githubusercontent.com/106908/35514687-5c584338-054a-11e8-8a4b-51e28da03f41.png)


### Pivotal Web Servicesにドメインを追加

独自ドメインでPivotal Web Servicesにアクセスした場合に、対象のアプリケーションにルーティングさせるには、まずは自分のOrganizationにドメインを追加する必要があります。


```
cf create-domain <your org> <your domain>
```

`cf domains`を実行して、独自ドメインが`owned`(`所有`)状態で追加されていることを確認してください。


```
$ cf domains
name            status   type
cfapps.io       shared
cf-tcpapps.io   shared   tcp
mydomain.test   owned
```


Blog UIの`manifest.yml`に、独自ドメインの`route`を追加します。

``` yaml
applications:
- name: blog-ui
  path: target/demo-blog-ui-0.0.1-SNAPSHOT.jar
  routes:
  - route: blog-ui-<your account>.cfapps.io
  - route: blog.<your domain>
  env:
    BLOG_API_URI: https://blog-api-<your account>.cfapps.io
```

`cf push`してください。

次のコマンドで`Host`ヘッダを変更しても、Blog UIにアクセスできることを確認してください。

```
curl -k https://blog-ui-<your account>.cfapps.io -H "Host: blog.<your domain>"
```

これで、Pivotal Web Servicesに独自ドメインでアクセス(`Host`ヘッダの値が独自ドメイン)した場合に、Blog UIにアクセスできるようになりました。

次に実際に独自ドメインがPivotal Web Servicesを向くようにDNSの設定を変更する必要があります。

### CloudFlareで独自ドメインを管理し、Pivotal Web Servicesに転送

[CloudFlare](https://www.cloudflare.com/)を使うとドメインを管理するだけでなく、HTTPSプロキシサーバーように使うことができます。
ブログの独自ドメインのCNAMEにPivotal Web ServicesのLBに向くホスト名(`api.run.pivotal.io`など)を設定すると、
`Host`ヘッダを独自ドメインのままリクエストをPivotal Web Servicesに向けることができます。また、TLS TerminationもCloudFlareで行えます。

![image](https://user-images.githubusercontent.com/106908/35514699-68683c3c-054a-11e8-875d-eda3a12d6f2b.png)

[https://www.cloudflare.com/](https://www.cloudflare.com/)でアカウントを作成し、自分のドメインを登録してください。
ドメインを購入したサイト上で対象の独自ドメインのName ServerにCloudFlareのName Serverを登録してください。DNSのページの下部に
(アカウント毎に異なるName Serverが用意されています)

![image](https://user-images.githubusercontent.com/106908/35515442-86b94a62-054c-11e8-900c-fc9f5520627e.png)

ブログのホスト名を`blog.<独自ドメイン>`としたい場合は、
DNSでCNAMEのNameに`blog`、Domain Nameに`api.run.pivotal.io`を設定してください。

![image](https://user-images.githubusercontent.com/106908/35514342-3b924726-0549-11e8-8b83-f90c72ead33d.png)


CloudFlare <-> Pivotal Web Services間もTLS通信にしたい場合は、Cryptoは"Full"または"Full(Strict)"を選択してください(推奨)。

![image](https://user-images.githubusercontent.com/106908/35513899-d7881d7e-0547-11e8-9847-c56b7cc3455a.png)


これで[https://blog.mydomain.test](https://blog.mydomain.test)でBlog UIにアクセスできます。

> Cloud FlareのFreeプランではTLS証明書は多くのドメインでSAN(Subject Alternative Names)を共有する形になっています。
> $10/month払うことで、専有のTLS証明書を使用することができます。

### Amazon CloudFrontからPivotal Web Servicesに転送

Amazon Route53でドメインを管理している場合は、Amazon CloudFrontでリクエストをPivotal Web Servicesに転送可能です。
この場合は、必ず`Host`ヘッダーを転送するようにしてください。
