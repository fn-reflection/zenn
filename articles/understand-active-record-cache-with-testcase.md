---
title: "テストケースで体感するN+1問題とRails ActiveRecordキャッシュメソッドの使いわけ"
emoji: "🎶"
type: "tech"
topics: ["performance", "rails", "activerecord", "cache", "model"]
published: false
publication_name: "paiza"
---

## モチベーション
データベースの実行パフォーマンス改善を進めていく上で、SQLを制御すること(とインデックス設計)はとても重要な要素技術になります。  
この記事ではテストケース(サンプルコード)を用いて、Railsの3つのキャッシュメソッドの振る舞いを体感しつつ、その使い分けについて理解を深めます。  
そしてActiveRecord(ORM)が発行するSQLをうまく制御できるようになることを目指します。

## 想定読者
- サーバサイドのパフォーマンスに興味がある人
  - ActiveRecordを利用している初級者、中級者
    - **よくわかってないけど(bulletの)言われるがままにincludesしている人**
  - ActiveRecordに影響を受けたORMを利用している人
- プログラムの抽象化に興味がある人
  - パフォーマンスを無視した抽象化(API設計)はシステムパフォーマンスを毀損することがあります

## ActiveRecordのキャッシュと3つのメソッド
今回の記事はActiveRecordの[(SQL)キャッシュ](https://railsguides.jp/caching_with_rails.html#sql%E3%82%AD%E3%83%A3%E3%83%83%E3%82%B7%E3%83%A5)に焦点を当てます。  
これはSQLのクエリ結果をメモリに保持する機能で、うまく活用できないとN+1問題やスロークエリを招きます。

preload, eager_load, includesという3つのメソッドを使うことでActiveRecordレベルでのキャッシュができます。  
**ではこの3つのメソッドをどう使い分ければよいのでしょうか**  
テストを書いてみていろいろ動作を調べてみると、理解が深まります。

## テストケースについて
この記事を書くにあたりキャッシュメソッドの理解を促進する[テストケースファイル](https://github.com/fn-reflection/testcases/tree/main/rails/active_record_cache_methods)を用意しました。  
バージョン3.1以降のRubyでBundlerをインストール済の環境であれば、コピーして`ruby test_for_article.rb`とするだけでサンプルコードが動きます。  
(うまく動かない人のためにDocker Composeファイルも用意しています。)
この記事のメインはこのファイルの解説なのでぜひ動かしてみてください。

## テストケースのデータ構造
データ定義はテストケースファイルに記載されていますが、以下のようなイメージです。  
(Userは複数のPostをもち、Postは複数のPostCommentを持ち、PostCommentは複数のPostCommentReviewを持つ)
```
User─┰─ Post1─┰─ PostComment1-1─┰─PostCommentReview1-1-1
     ┃        ┃                 ├─PostCommentReview1-1-2
     ┃        ┃                 └─PostCommentReview1-1-3
     ┃        ├─ PostComment1-2─┰─PostCommentReview1-2-1
     ┃        ┃                 ├─PostCommentReview1-2-2
     ┃        ┃                 └─PostCommentReview1-2-3
     ┃        └─ PostComment1-3─┰─PostCommentReview1-3-1
     ┃                          ├─PostCommentReview1-3-2
     ┃                          └─PostCommentReview1-3-3
     ├─ Post2─┰─ PostComment2-1─┰─PostCommentReview2-1-1
     ┃        ┃                 ├─PostCommentReview2-1-2
     ┃        ┃                 └─PostCommentReview2-1-3
     ┃        ├─ PostComment2-2─┰─PostCommentReview2-2-1
     ┃        ┃                 ├─PostCommentReview2-2-2
     ┃        ┃                 └─PostCommentReview2-2-3
     ┃        └─ PostComment2-3─┰─PostCommentReview2-3-1
     ┃                          ├─PostCommentReview2-3-2
     ┃                          └─PostCommentReview2-3-3
     └─ Post3─┰─ PostComment3-1─┰─PostCommentReview3-1-1
              ┃                 ├─PostCommentReview3-1-2
              ┃                 └─PostCommentReview3-1-3
              ├─ PostComment3-2─┰─PostCommentReview3-2-1
              ┃                 ├─PostCommentReview3-2-2
              ┃                 └─PostCommentReview3-2-3
              └─ PostComment3-3─┰─PostCommentReview3-3-1
                                ├─PostCommentReview3-3-2
                                └─PostCommentReview3-3-3
```
今回では、テストケースのログ出力がわかりやすくなるように各層が3つのアイテムを持つ(例：ユーザは3個のポストデータを持つ)としています。  
しかし現実のデータはもっと大量のデータを持つ(例：ユーザが50個のポストデータを持つ)のが普通ですので、それをイメージしたり、コードを改変しながら実行すると理解が捗ります。

## モデルメソッドについて
PostとUserメソッドには情報を出力するメソッドをいくつか定義しました。  
Postのdescribeメソッドが関連(子モデル、孫モデル)の情報を含めて出力するメソッドであり、Userはそれを用いて出力メソッドを定義しています。

```ruby
class User < ActiveRecord::Base
  # ...

  # user.postsに関する情報を関連を含めて出力する
  def full_describe
    posts.each do |post|
      post.describe(user: self)
    end
  end

  # ある特定の名前を持つuser.postsに関する情報を関連を含めて出力する
  def partial_describe(title_to_find:)
    posts.filter{ |post| post.title == title_to_find }.each do |post|
      post.describe(user: self)
    end
  end

  # remarkable_postsという特化した関連を用いて絞り込んだ情報を関連を含めて出力する
  def remarkable_describe()
    remarkable_posts.each do |post|
      post.describe(user: self)
    end
  end
end

class Post < ActiveRecord::Base
  # ...

  # postに関する情報を関連を含めて出力する
  def describe(user:)
    post_comments.each do |comment|
      comment.post_comment_reviews.each do |review|
        puts "#{user.name}, #{title}, #{comment.comment}, #{review.grade}"
      end
    end
  end
end
```

## テストケースの解説
最初のテストケースは、何の細工(キャッシュ)もせず関連を含めた全データを出力します。
```ruby
テストケース1
user = User.find_by(name: 'Alice')
user.full_describe
```
コード実行するとわかりますが、SQLの発行回数はUserが1回、Postが1回、PostCommentが3回、PostCommentReviewが9回、計14回であり、必要以上のSQLが発行されます。  
これが**N+1問題**と呼ばれているものです。  
これはあくまでテストデータでの発行回数であり、実際のサービスだともっと多くのクエリが発行されてしまい、あまりよくないです。

次のテストケースではpreloadメソッドを用いて、全てのひ孫モデル(PostCommentReview)までをキャッシュします。
```ruby
# テストケース2
user = User.preload(posts: { post_comments: :post_comment_reviews }).find_by(name: 'Alice')
user.full_describe
```
こう書くことでuser変数に必要な関連モデルのデータが事前ロードされます。  
SQLの発行回数はUserが1回、Postが1回、PostCommentが1回、PostCommentReviewが1回、計4回でSQL発行数を削減できます。 

ではpreloadで全て書けばよいのかというとそうではなく、preloadには限界があります。  
ここまでは全部のデータを一括で引いてくる想定でしたが、次のテストケースでは特定のPostに紐づいたデータをpartial_describeメソッドで引いてきます。
```ruby
# テストケース3
user = User.preload(posts: { post_comments: :post_comment_reviews }).find_by(name: 'Alice')
user.partial_describe(title_to_find: 'Post 2')
```
コードの出力を見ればわかりますが、テストケース2と全く同じpreloadクエリが発行されています。  
特定のPostデータだけ引っ張ろうとしても、**preloadでは細やかなSQLの制御ができず**無駄なデータを取得してしまうことがわかります。
これではPostデータが増えたときにも、全部のデータを引っ張ってくる(そして捨てる)わけなのでシステムがスローダウンしやすいです。  
ではpreloadを用いつつ、'Post 2'に関連づいたデータだけをSQLで抽出する方法はあるかというと以下の方法が一応考えられます。

```ruby
class User < ActiveRecord::Base
  # ...

  has_many :remarkable_posts, -> { where(posts: { title: 'Post 2' }) }, class_name: 'Post'
  
  # ...
end
```
以上のような**スコープ付きのhas_many関連**をpostsとは別に定義し、
この関連を用いることでpreloadが発行するSQLを制御できます。

```ruby
# テストケース4
user = User.preload(remarkable_posts: { post_comments: :post_comment_reviews }).find_by(name: 'Alice')
user.remarkable_describe
```
このようにするとPost 2に紐づいたデータだけをpreloadで抽出できます。  
ただ見ての通りこの関連にはまったく柔軟性がなく**負債化しやすいため極力採用したくはない**です。  
(スコープ付き関連の適切な応用例として、カバリングインデックスを効かせるためにselectでカラムを絞った関連を作る方法が考えられますが、上級者向けです。)

さて今までfull_describeやpartial_describeというAPIを真面目に使ってきましたが、これを無視したらどうでしょうか。

```ruby
# テストケース5
user.posts.preload(post_comments: :post_comment_reviews).where(posts: { title: 'Post 2' }).each { |post| post.describe(user:) }
```

こう書けば余計な関連を追加する必要もなくpreloadを用いて必要最小限のデータ取得を実現できます。  
とくにpartial_describeのような抽象化はデータのフィルタリングをruby側でやると指定してしまっている時点であまりよい抽象化ではありません。  
こういったメソッドを作るぐらいなら子モデルの参照を公開した方が(システムスローダウンを誤った方法で解決せずに済むため)結果として負債が少なくなります。

ここまではpreloadメソッドの挙動について深掘りしてきましたが、eager_loadメソッドも併用した場合は以下のような書き方もできます。
```ruby
# テストケース6
user = User.eager_load(:posts).preload(posts: { post_comments: :post_comment_reviews }).where(posts: { title: 'Post 2' }).find_by(name: 'Alice')
user.full_describe
```
この書き方をするとUserとPostモデルはJOIN(とDISTINCT)を用いてまとめてデータを抽出し、UserとPostに対するWHEREクエリ(SQLでのフィルタリング)なども実行できます。  
一方PostComment, PostCommentReviewモデルについては今まで通りpreloadとして別のクエリが発行され処理されます。  
必要なデータをSQLで抽出しつつ、パフォーマンス劣化の原因になりうる多段のJOINを避けることができるため**要件にそったバランスが良い書き方**です。

ここまで、includesという人気メソッドを無視してきましたが、筆者は**積極的に使うべきではない**と考えています。
```ruby
# テストケース7
user = User.includes(posts: { post_comments: :post_comment_reviews }).where(posts: { title: 'Post 2' }).find_by(name: 'Alice')
user.full_describe
```
例えば以上のように書けばテストケース6と同じようなSQLになってくれるのではないかと期待しますが、実行してみればわかるとおり**そうなりません**。  

1つでもJOINの必要なテーブル(posts)があればincludesに含まれたテーブルも全てeager_load(多段JOIN)に切り替わります。  
例えば**機能改善を通じてwhere句をほんの少し追加しただけでも**、元々preloadを想定していたテーブルが全てeager_loadに切り替わり予期せぬスローダウンへと繋がります。
とはいえincludesは人気メソッドであるため、includesだらけのコードのリファクタリングを進めなければならないというのもよくあることでしょう。
既存コードのincludesがeager_loadとして動いているのか、preloadとして動いているのかは、`eager_loading?`メソッドで識別でき調査に使えます。

以下はincludesの基本挙動を示したテストコードです。
```ruby
# テストケース8
# SQLが発行される直前でeager_loading?を呼び出すと、includesがeager_loadになるかpreloadになるかを判定できる
# サブテーブルに対するwhere句がある場合は、この絞り込みを処理するために最低でもuserとpostをJOINする必要があるためeager_loadになる
log(User.includes(posts: { post_comments: :post_comment_reviews }).where(posts: { title: 'Post 2' }).eager_loading?) # true
# eager_loadが一つでも含まれる場合はひ孫テーブルまで含めてeager_load(JOIN)する
log(User.eager_load(:posts).includes(posts: { post_comments: :post_comment_reviews }).eager_loading?) # true
# eager_loadまたはwhere句など絞り込みがない場合はpreloadとして同じになる
log(User.includes(posts: { post_comments: :post_comment_reviews }).eager_loading?) # false
```

## まとめ
- 数多くのテストケースを用いてpreloadとeager_loadの挙動について説明しました
  - N+1問題は時折語られますが、この2つのメソッドをよく理解していればおそれることはありません
- スコープ付き関連は柔軟性が低いのでよほどの理由がない限り避けます
- パフォーマンスを無視したメソッド(API)は時として無視することも大事です
  - partial_describeを使う(正当化する)ためにスコープ付き関連を導入するのはバランス感覚にかけています
- 新規実装においてincludesはあまり推奨できません
  - includesはカジュアルに扱われますがピーキーな挙動を示し、保守性が低いです
  - preloadを基本としてwhere句で絞りたいテーブルを必要に応じてeager_loadに置き換えていくのがよいです
- こうしたテストコードを持っていると挙動の詳細を忘れたときに役に立ちます
  - 自由に改変して使っていただいてOKです

## 参考文献
https://tech.stmn.co.jp/entry/2020/11/30/145159  
素晴らしい記事です。  
本記事はこの記事の内容を追試しつつテストコードとして実行したり、より直観的に課題意識が伝わるように書き換えたものになります。

https://moneyforward-dev.jp/entry/2019/04/02/activerecord-includes-preload-eagerload/  
こちらもまた素晴らしい記事です。
各メソッドの振る舞いをソースコードレベルで検証していて、非常に参考になります。
