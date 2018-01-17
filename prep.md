## 事前準備

ハンズオンで作成するブログシステムの記事のフォーマットと、そのドメインクラスである`Entry`クラスについて説明します。

演習が2つありますので、ハンズオン前に必ず実施してください。

### ブログ記事のフォーマット

ブログ記事のフォーマットは次のようになります。


``` markdown
---
title: Hello Spring Boot
tags: ["Spring", "Spring Boot", "Java"]
categories: ["Programming", "Java", "org", "springframework", "boot"]
date: 2015-11-15T23:59:32+09:00
updated: 2015-11-15T23:59:32+09:00
---

Content(markdown)
Here

`date` and `updated` are optional.
If `date` is not specified, first commit date is used.
If `updated` is not specified, last commit date is used.
```

`---`と`---`の間に記事のメタ情報(Front Matter)を書きます。Front Matter以降は記事本文(`content`)であり、[Markdown](https://guides.github.com/features/mastering-markdown/)形式です。サポートするMarkdown方言はレンダリング側(ブログのフロントエンド)で規定します。

[Jekyll](https://jekyllrb.com/docs/frontmatter/)や[Hugo](https://gohugo.io/content-management/front-matter/)のFront Matterに似ていますが、同じではありません。

Front Matterの仕様は

* `title` ... 記事のタイトル  / 文字列 / **必須**    
* `tags`  ... 記事のタグ / 文字列リスト / 任意
* `categories` ... 記事の階層カテゴリ / 文字列リスト / 任意
* `date` ... 記事の作成時刻 / ISO8601のyyyy-MM-ddTHH:mm:ss.SSSZ / 任意
* `updated` ... 記事の更新時刻 / ISO8601のyyyy-MM-ddTHH:mm:ss.SSSZ / 任意

`date`や`update`は記事の時刻を固定したい場合にのみ使用し、基本的に記事の作成・更新時刻ははgitのコミット履歴を基に設定されることを想定しています。

Front Matter以外の記事情報は次の通りです。

* `entryId` ... エントリID / Long / **必須**
* `created.name` ... 作成者名 / Long / **必須**
* `created.date` ... 作成時刻 / ISO8601のyyyy-MM-ddTHH:mm:ss.SSSZ / **必須**
* `updated.name` ... 更新者名 / Long / **必須**
* `updated.date` ... 更新時刻 / ISO8601のyyyy-MM-ddTHH:mm:ss.SSSZ / **必須**

`entryId`はファイル名から取得されます。ファイル名が`00100.md`であれば、`100`が`entryId`になります。
`created`と`updated`はgitのコミット情報から得られる想定です。
Front Matterの`date`、`updated`と意味が重複しますが、明示的に時刻を上書きしたい場合にFront Matterの`date`、`updated`を使用できます。


サンプルプロジェクトは[https://github.com/making/demo-blog-posts](https://github.com/making/demo-blog-posts)です。

### ブログ記事のドメインオブジェクト

ブログ記事のドメインオブジェクトをJavaで表現するライブラリが`am.ik.blog:blog-domain`で使えます。

``` xml
<dependency>
    <groupId>am.ik.blog</groupId>
    <artifactId>blog-domain</artifactId>
    <version>4.3.4</version>
</dependency>
```

記事は`Entry`クラスで表現され、`Entry`オブジェクトはImmutableです。

``` java
Entry entry = new Entry(new EntryId(100L), new Content("記事本文"),
        new Author(new Name("作成者名"), new EventTime(OffsetDateTime.parse("2018-01-12T18:31:25.899+09:00"))),
        new Author(new Name("更新者名"), new EventTime(OffsetDateTime.parse("2018-01-12T18:31:25.899+09:00"))),
        new FrontMatter(new Title("タイトル"), 
                new Categories(new Category("カテゴリ1"), new Category("カテゴリ2")),
                new Tags(new Tag("タグ1"), new Tag("タグ2"))));
```


`EntryBuilder`経由で作成すると便利です。

``` java
Entry entry = Entry.builder()
        .entryId(new EntryId(100L))
        .content(new Content("記事本文"))
        .frontMatter(new FrontMatter(new Title("タイトル"),
                new Categories(new Category("カテゴリ1"), new Category("カテゴリ2")),
                new Tags(new Tag("タグ1"), new Tag("タグ2")))),
        .created(new Author(new Name("作成者名"), new EventTime(OffsetDateTime.parse("2018-01-12T18:31:25.899+09:00"))))
        .updated(new Author(new Name("更新者名"), new EventTime(OffsetDateTime.parse("2018-01-12T18:31:25.899+09:00"))))
        .build();
```

Front Matterに`date`や`updated`を設定する場合は、次のように作成できます。

``` java
Entry entry = Entry.builder()
        .entryId(new EntryId(100L))
        .content(new Content("記事本文"))
        .frontMatter(new FrontMatter(new Title("タイトル"),
                new Categories(new Category("カテゴリ1"), new Category("カテゴリ2")),
                new Tags(new Tag("タグ1"), new Tag("タグ2")),
                new EventTime(OffsetDateTime.parse("2018-01-12T18:31:25.899+09:00" /* date */)),
                new EventTime(OffsetDateTime.parse("2018-01-12T18:31:25.899+09:00" /* updated */)))),
        .created(new Author(new Name("作成者名"), new EventTime(OffsetDateTime.parse("2018-01-12T18:31:25.899+09:00"))))
        .updated(new Author(new Name("更新者名"), new EventTime(OffsetDateTime.parse("2018-01-12T18:31:25.899+09:00"))))
        .build()
        .useFrontMatterDate();
```

Markdownファイルから`Entry`オブジェクトを作成する場合は、次のように`EntryFactory`から作成可能です。返り値は`Optional<EntryBuilder>`になります。
ファイルの内容からは`entryId`やgitコミット履歴の`created`、`updated`は判断できないため、`EntryBuilder`を返し、残りのフィールドは個別に設定する必要があります。

``` java
EntryFactory factory = new EntryFactory();
Resource file = new ClassPathResource("foo/bar/00100.md"); // クラスパス上のMarkdownファイルから作成
// Resource file = nnew FileSystemResource("/tmp/foo/bar/00100.md"); // ファイルシステム上のMarkdownファイルから作成
Optional<EntryBuilder> entryBuilder = factory.createFromYamlFile(file);
Optional<Entry> entry = entryBuilder.map(
        builder -> builder.entryId(EntryId.fromFileName(file.getFilename()))
                .created(new Author(new Name("作成者名"), new EventTime(OffsetDateTime.parse("2018-01-12T18:31:25.899+09:00"))))
                .updated(new Author(new Name("更新者名"), new EventTime(OffsetDateTime.parse("2018-01-12T18:31:25.899+09:00"))))
                .build());
```

次のように、Markdownのファイル名とファイル内容(文字列)から`Entry`オブジェクトを作成することもできます。

``` java
EntryFactory factory = new EntryFactory();
Optional<EntryBuilder> entryBuilder = factory.parseBody(new EntryId(100L), "00100.mdのファイルの中身");
Optional<Entry> entry = entryBuilder.map(builder -> builder
        .created(new Author(new Name("作成者名"), new EventTime(OffsetDateTime.parse("2018-01-12T18:31:25.899+09:00"))))
        .updated(new Author(new Name("更新者名"), new EventTime(OffsetDateTime.parse("2018-01-12T18:31:25.899+09:00"))))
        .build());
```


### 演習1: `Entry`オブジェクトを作成してみる

ここまでの内容を試すために、[https://github.com/making/blog-handson-prep](https://github.com/making/blog-handson-prep)を`git clone`して、
[`EntryCreator`](https://github.com/making/blog-handson-prep/blob/master/src/main/java/com/example/blog/EntryCreator.java)を実装して、
[`EntryCreatorTest`](https://github.com/making/blog-handson-prep/blob/master/src/test/java/com/example/blog/EntryCreatorTest.java)がグリーンになるようにしてください。

### GitHub APIを使って、`Entry`オブジェクトを作成

`Entry`オブジェクトをGitHubレポジトリ上のMarkdownコンテンツから[Content API](https://developer.github.com/v3/repos/contents/)と[Commits API](https://developer.github.com/v3/repos/commits/)を使用して作成しましょう。

ここではGitHub APIをSpring 5の`WebClient`を使ってアクセスするラッパーライブラリを使用します。

``` xml
<dependency>
    <groupId>am.ik.github</groupId>
    <artifactId>reactive-github-client</artifactId>
    <version>0.0.4</version>
</dependency>
```



``` java
WebClient.Builder builder = WebClient.builder();
GitHubClient client = new GitHubClient(builder,
        new AccessToken("<Access Token>"));
```

Access Tokenは[Personal access tokens](https://github.com/settings/tokens)ページから"Generate new token"ボタンをクリックして生成してください。`Select scopes`では`public_repo`を選択してください。

#### Content APIで`EntryBuilder`オブジェクトの作成

Content APIを使ってファイルを取得するには次のように書けば良いです。

``` java
Mono<File> file = client.file("making", "demo-blog-posts", "content/00001.md").get();
file.map(f -> f.decode())
        .doOnNext(System.out::println)
        .block(); // 非同期処理中にプログラムが終わらないようにblockする
```

`client.file("owner名", "repository名", "Markdownファイルまでのパス").get()`で`Mono<File>`型のオブジェクトを取得できます。
GitHubのAPIで返るファイルの中身はBase64でエンコードされているため、ファイル内容をデコードする`decode()`メソッドを呼ぶことで記事本文を取得できます。

ちなみに、`File`は`am.ik.github.repositories.contents.ContentsResponse.File`であり、`java.io.File`ではありません。

この記事本文を`EntryFactory`に渡します。

``` java
EntryId entryId = new EntryId(1L);
Mono<File> file = client.file("making", "demo-blog-posts", String.format("content/%05d.md", entryId.getValue())).get();
Mono<EntryBuilder> builder = file
        .map(f -> new EntryFactory().parseBody(entryId, f.decode()))
        .flatMap(b -> Mono.justOrEmpty(b));
```

すでに見てきたように`EntryFactory.parseBody`の返り値は`Optional<EntryBuilder>`なので、`Optional`から`Mono`に変換するために、`Mono.justOrEmpty`を使用します。
ラムダ式の返り値が`Mono`(`Publisher`)なので、`subscribe`を伝播させるために`map`ではなく`flatMap`を使用します。その結果`Mono<EntryBuilder>`を得られます。


#### Commits APIから`Author`オブジェクトの作成

次にCommits APIから、作成者名・作成時刻および更新者名・更新時刻を取得します。

``` java
Flux<Commit> commits = client.commits("making", "demo-blog-posts")
        .get(params -> params
                .path(String.format("content/%05d.md", entryId.getValue())));
```

返り値は`Flux<Commit>`です。

この`Flux`の先頭が最終更新コミット、最後が作成コミットとみなし、`collectList()`メソッドを使い、一旦`List`にまとめ、先頭と最後の`Commit`を取得します。
これを後述の`toAuthor`メソッドに渡し、`Author`オブジェクトを作成します。    

``` java
commits.collectList()
        .doOnNext(list -> {
            Author updated = toAuthor(list.get(0));
            Author created = toAuthor(list.get(list.size() - 1));
        });
```

`toAuthor`メソッドは次の通りです。

``` java
private static Author toAuthor(Commit commit) {
    Committer committer = commit.getCommit().getAuthor();
    return new Author(new Name(committer.getName()), new EventTime(committer.getDate()));
}
```

2つの`Author`をまとめて返せるように、`Tuple`を使います。

``` java
Mono<Tuple2<Author, Author>> authors = commits.collectList()
        .map(list -> {
            Author updated = toAuthor(list.get(0));
            Author created = toAuthor(list.get(list.size() - 1));
            return Tuples.of(created, updated);
        });
```

#### `EntryBuilder`と`Author`の合成

`EntryBuilder`と`Author`をそれぞれ非同期で取得できましたが、`Entry`を作成するためには、これらを合成する必要があります。
まず、

`Mono<EntryBuilder>`と`Mono<Tuple2<Author, Author>>`の二つの`Mono`を`Mono.zip`で合成します。さらに`map`で`EntryBuilder`と`created`、`updated`を使い`Entry`オブジェクトを完成させます。


``` java
Mono<Entry> entry = Mono.zip(builder, authors)
        .map(t -> {
            EntryBuilder entryBuilder = t.getT1();
            Author created = t.getT2().getT1();
            Author updated = t.getT2().getT2();
            return entryBuilder
                    .created(created)
                    .updated(updated)
                    .build()
                    .useFrontMatterDate();
        });
```

これでノンブロッキングでGitHub APIから`Entry`オブジェクトを作ることができました。    


### 演習2: Mock GitHub APIにアクセスして`Entry`オブジェクトを作成してみる

ここまでの内容を試すために、[`EntryFetcher`](https://github.com/making/blog-handson-prep/blob/master/src/main/java/com/example/blog/webhook/EntryFetcher.java)を実装して、
[`EntryFetcherTest`](https://github.com/making/blog-handson-prep/blob/master/src/test/java/com/example/blog/webhook/EntryFetcherTest.java)がグリーンになるようにしてください。


