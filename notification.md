# HTML5のServer-Sent EventsとNotifications APIを使ってブログ記事の更新通知

Blog APIの`@EventListener`に処理を追加して、記事の更新イベントをServer Pushで通知できるようにします。
ここではBlog API側からはServer-Sent EventでEventを通知します。
Blog UI側では`WebClient`でEventを購読して、再度HTMLにServer-Sent Eventでイベントを通知します。

```
[Blog API] --(SSE)--> <--(WebClient)-- [Blog UI] --(SSE)--> <--(EventSource)--[HTML]
```

### Blog APIの修正

https://github.com/making/demo-blog-api/commit/1479b33f91f1c1bb8ad4374861cba5f9440d7ad9

### Blog UIの修正

https://github.com/making/demo-blog-ui/commit/ac45538116eab7361ecf90e589b2a226867a8fe9

### 動作確認

Blog API, Blog UIを再起動して、`sample-create-request.sh`、`sample-update-request.json`を実行してください。

![image](https://user-images.githubusercontent.com/106908/35485244-1fdca908-04a0-11e8-8842-847101649c52.png)

### Cloud Foundryへデプロイ


```
cd ../demo-blog-api
./mvn clean package
cf push

cd ../demo-blog-ui
./mvn clean package
cf push
```

記事の作成、更新でイベントが通知されることを確認してください。

### スケールアウト対策

今の実装ではBlog APIをスケールアウトした際に、WebHookがロードバランスしてしまうため、
Server Pushをするインスタンスがラウンドロビンになってしまい、通知を受けられる場合と受けられない場合が出てしまいます。

これを防ぐには、`EventNotifyer#notify`でRabbitMQやKafkaのようなMessage Brokerにメッセージを送信し、
そのメッセージリスナーで`this.processor.onNext(event);`を実行する必要があります。


Spring Cloud Streamを使う場合は次のような実装になるでしょう。

``` java
@Component
public class EventNotifyer {
	final UnicastProcessor<EntryEvent> processor = UnicastProcessor.create();
	final Flux<EntryEvent> flux;
	final Source source;

	public EventNotifyer(Source source) {
		this.flux = this.processor.publish().autoConnect().log("event").share();
		this.source = source;
	}

	public void notify(EntryEvent event) {
	    source.output().send(MessageBuilder.withPayload(event).build());
	}

    @StreamListener(Sink.INPUT)
	public void onMessage(EntryEvent event) {
		this.processor.onNext(event);
	}

	public Publisher<EntryEvent> publisher() {
		return this.flux;
	}

}
```


[Spring Cloud Stream Tutorial](https://github.com/Pivotal-Japan/spring-cloud-stream-tutorial)を実践し、Message Broker版を実装してみてください。
