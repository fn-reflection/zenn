---
title: "リファクタリングを避けるコードデザイン(Railsを題材として)"
emoji: "🏙️"
type: "tech"
topics: ["rails", "ruby", "design", "architecture", "lessismore"]
published: true
published_at: 2023-08-30 17:00
publication_name: "paiza"
---

## 2つの不確実性とリファクタリング
プロダクションコードを書いていると、リファクタリングをしなければならないコードにぶち当たります。  
正直なところリファクタリングは時間がかかるので避けたいものですが、必要になるようです。  
必要な理由は大きく分けて2つあります。  
1つ目は市場など外部の不確実性に対抗し、既存実装では不要だった抽象化を機能のために追加するためです。  
これは**因果的に回避できません**が、プロダクトの改善に直結するという意味でポジティブなものです。  
2つ目は内部不確実性に対抗し、既存コードの意図の明瞭化や、必要以上の抽象化で身動きが取れない状況を改善するためです。  
これは注意深くコードを構成することで回避可能なものです。  
今回の記事では後者のリファクタリングを回避するためにどのようにコードを構成すべきかについて、筆者の判断基準を明確化することと、Railsでの適用例を示します。  
(事例は紹介程度にとどめますが、必要性を感じたら続編を書きます)

## 想定読者
- Web(サーバサイド)エンジニアの中級者
- プロダクションコードの実践的な問題に対峙しているエンジニア
  - 世間に流通している設計論に腹落ちしていないエンジニア
  - 新規開発コードを書くエンジニア
    - ファーストコミットは長期的な開発生産性に大きな影響を与える

## 判断基準
新規コードやリファクタリングでいつも気になるのは、「**どのような客観的基準でコードが構成されるべきか**」ということです。  
この観点が明瞭でないといくらリファクタリングをしたと主張しても、リファクタリング返しされる可能性は否定できないでしょう。  
ある仕様を実現する実装は星の数ほどあり、その中で一番よいと思えるものを選ぶにはその判断基準(=設計思想)が必要です。  
個人的には、**継続的開発をする上で、最小限のリスクで最新仕様に追従可能なコード構成がよい**と考えています。  
それはパフォーマンスチューニング、抽象化、機能の高度化が必要な時に最小限の編集で実行可能な状態を維持することが継続開発において重要だからです。  
そのようにコードを構成する上で筆者が重要だと考える判断基準は以下のとおりです。
- ある要件を達成する最もシンプル(≠easy)な実装を選択する
  - 「Pattern Language」よりも「**Less is More**」で考える
    - 筆者は建築製品設計のバックグラウンドを持ち、プロダクトデザインの基本原理として今なお機能していると確信している
      - 極めて有効に機能するミニマルなルールを見出すことが知的作業の本質と考える
  - 仕様を過剰拘束する言語機能を乱用した実装にしない
    - 拘束を振り解くためのリファクタリングが必要になり、仕様改善の採算性が悪化する
      - そのような例の1つとメカニズムについては[quora](https://jp.quora.com/%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E6%8C%87%E5%90%91%E3%81%AB%E3%81%8A%E3%81%84%E3%81%A6-%E7%B6%99%E6%89%BF-%E3%81%8C%E5%95%8F%E9%A1%8C%E8%A6%96%E3%81%95%E3%82%8C%E3%82%84%E3%81%99%E3%81%84/answers/1477743628162710)に書いてある
      - 多数の著名人からも高評価をいただいた
      - 他にも問題を指摘すべき言語機能は沢山あるが、これ以上の例証は無益なのでやっていない
      - 論証主義者に対する回答
    - RustやGoなど現代を象徴する言語が示している結論をとりいれる
       - 優れた言語設計者により不要な言語機能が取り除かれているので、上記の論証は読む必要がない
         - Less Is More
         - 権威主義者に対する回答
- 極力静的かつ明示的な参照解決
  - 静的(レキシカル)に処理を読みくだせない＝動的な実行環境(コンソールetc)による検証が必要
  - 動的な環境の状態によって意味が変化する＝コードレビューによる検証精度も低下する
    - 継承元に含まれたとんでもない副作用をレビュアーは見落とす可能性が高い
  - これもRustやGoが示しているその結論をとりいれる
  - 動的言語ではコンパイル時検証はできないが、動的機能を減らすようにコード構成できる
    - 言語文化的にそうではなくても、そのように取りあつかうことはできる
- 不要な(とくに1vs1)抽象化を排除する
  - 抽象化が施されたコードは不要化した時に削除しにくくなる
    - 関連記事：[zenn: 不要コードを継続的に削除し、技術的負債に対抗する](https://zenn.dev/paiza/articles/continuous-deleting-unused-codes)
    - 抽象化により**スコープが必要以上に拡大しどこから参照されているか不明瞭になる**から削除しにくい
  - 抽象化は[近接の法則](https://blog.adobe.com/jp/publish/2021/12/13/cc-web-proximity-in-design-principles)に反し、自明性(=可読性)を損ねる
    - 自明だが、誰も指摘しない
    - 直接参照を可能な限り維持し、効果的に間接参照を使うこと
      - 関連記事：[zenn: 言語哲学のアイデアからより良い名付けのスタンスを考える](https://zenn.dev/paiza/articles/derive-naming-principle-from-philosophers-idea)
  - **可読性低下、スコープ拡大という損失を補うだけの利益があるかを考える**
    - たとえば引数が関数内部で再利用される(つまり仮引数と実引数参照が1vsNになる)ので扱いやすくなる
  - 「転ばぬ先の杖」的な抽象化レイヤは可読性の低下か、不必要な仕様制約を引き起こす
    - 事前のインタフェース定義は、**ウォーターフォールプロセスと仕様の固定が事業的に約束できる場合のみ**使用する
      - 筆者は建築分野でそれを実行したが、ソフトウェアでそうする必然性はないと考える

![](/images/articles/code-design-to-avoid-refactoring/timeless-facade.jpg =500x)
*シンプルさをつきつめることで見出されたデザインは65年の時の試練に耐え、いろあせることがなく現代的でありつづける*

自分がリファクタリングを考えるとき、メタレイヤーとしてこのような判断基準を使っています。  
極めて自明なルールのように思えますが、その適用は時として慣習や風潮と対峙することになります。

## プラクティス1: インスタンス変数の封印
Railsスタイルとして、コントローラ層においてインスタンス変数とコールバックを使うような実装があります。たとえば以下の通りです。
これはRailsの標準的な記述として非常によく使われるものです。
```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  before_action :set_user

  def index
  end

  def set_user
    @user = User.find(user_params)
  end

  def user_params
    params.require(:user).permit(:id)
  end
end
```

```haml
-# app/views/my/index.html.haml
%p=@user.name
%p=@user.address
```

コントローラ内のアクション数を**安易に増やさなければ**ある程度うまくいくでしょう。  
ただこうしたコールバックベースの抽象化は、その**コントローラ内での処理の共通化を暗黙的に仮定しています**。  
(そうでなければ、そのようにコードを構成する必然性はないのですから。)  
慣れた(easyな)取り扱いを継続するといつの間にか以下のようなコントローラに変貌します。
```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  before_action :set_user, only: [:index, :create, :destroy, :other_action2]
  before_action :set_posts, only: [:index, :create, :destroy, :other_action2]
  before_action :set_post_memos, only: [:index, :create, :destroy, :other_action2]
  # ... 以下無数のbefore_action

  def index
  end
  # ... 以下無数のアクションメソッド
  def other_actions2
  end

  private

  def set_user
    @user = User.find(user_params)
  end

  def set_post
    @posts = @user.posts(post_params)
  end
  ...
end
```
慣習的な記述法をそのまま継続するとコントローラがモンスター化します。  
(慣習の力は恐ろしくそれに対抗する理論がなければ、それに流される他ないのです。)  
このコードをベースにパフォーマンスチューニングするとして、どのような修正をするでしょうか。  
関連データをキャッシュするためset_userメソッドを編集するにしても、全てのアクションに対し共通化されている以上、あるアクションにだけeager_loadを足すことはできないはずです。  
そしてset_user内で呼び出し元を区別する条件分岐を書く結論に行き着きます。要求はそれでも満足しますが、その選択の結果コードがカオスになっていきます。  
これは**スケールしない抽象化**の一例です。抽象化を施す場合それを拡張した時の様相を想像する必要があります。

たとえば筆者がこのコントローラを実装する場合、初期実装(あるいはリファクタリング結果)はおおよそ以下のようになるでしょう。
```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  def index
    user_params = params.require(:user).permit(:id)
    user = User.find(user_params)
    render locals: { user: }
  end
end
```

```haml
-# app/views/my/index.html.haml
%p=user.name
%p=user.address
```
こう記述することのメリットは以下の通りです。
- アクションに対して処理が集約されている
  - 疎結合なアクション
  - 各アクションが必要とするデータに即したローディング戦略が選択できる
    - setterごと結合していると、必要以上のデータロードをしたり、N+1がなかなか改善できない状態に陥る
      - **処理量が拡大したときに役に立たなくなる抽象化はいかなる時点においても役に立たない**ので、まずリファクタリング対象になる
- 処理の順序関係が明白である
  - スクリプトが読めるなら誰でも理解できる
    - たとえばJavaScript専門のフロントエンドエンジニアにコードだけで振る舞いを納得させられる
  - コードジャンプによる介入余地が限りなく少ない
    - コードが読んだ通りに動く可能性が高い＝不確実性の低下
    - コードのスコープ(ライフタイム)が最小化される＝不確実性の低下
- viewのインターフェースが明確になる
  - インスタンス変数によるデータパッシングはpartialの再利用性に問題が出てくる
    - partialをインスタンス変数を用いて再利用する＝同一のインスタンス変数を異なるアクションで設定する
      - setter+コールバックアクションの再利用も助長する
        - 結果コントローラがスパゲッティ・モンスター化する

モンスターコントローラになっていた実装もたとえば以下のように書けます。
```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  def index
    user = User.eager_load(:posts).find(user_params)
    render locals: { user: } # postsなどはuserの参照から取得できる
  end
  ...
  def other_actions2
    user = User.eager_load(posts: :memos).find(user_params)
    render locals: { user: }
  end

  private

  def user_params
    params.require(:user).permit(:id)
  end
end
```

これならば「改修対象のアクションに必要な分だけの関連参照追加と、キャッシュローディングを記述してビューの機能拡張します(してください)」で報告（作業指示）が終わります。  
「利用者が必要なことに集中できる状態を作り出す(=関心の分離)」が設計の価値提供の本質というわけです。

## プラクティス2: 1vs1パーシャルをやめる
これも実際のリファクタリングを実行する上でよくある事例です。
初期実装者は善意で1vs1パーシャルを作成することが多いですが、長期的にみるとネガティブに作用することが多いです。
たとえば以下のような1vs1パーシャル分割の例を考えてみましょう。

```haml
-# app/views/foo/bar.html.haml
-# ...

- if foo
  - if bar
    render 'page_specific_ui' # 過剰なネストを抑えるためにパーシャル化する
-# ...
```

```haml
-# app/views/foo/_page_specific_ui.html.haml
%p=@user.level
-# ...以下それなりに長いUI実装
```

「ネストが深いからパーシャル化する」という分割基準は自然にみえますが、その恣意性は複数の課題を創出します。
- 分割基準が人によって揃わないこと
  - 筋が悪い設計は「ネストN段の場合はパーシャル分割する」というルールを作成するだろう
    - パーシャル分割の理由が「ネスト段数」と「参照共有の必要性」の2つになることにより一貫性がなくなる
  - そもそも要求実装がネストを要求する場合、要求の複雑度によってパーシャルが分割されたりされなかったりすることになる
- インスタンス変数への依存がパーシャルに隠蔽されること
  - foo/bar.html.hamlだけ見て、@userへの依存がないと判断してインスタンス変数を削除する
    - 意図せず本番で実行時エラーを踏む
    - 「しっかりrender内のパーシャル実装を調査すればそんなミスはしない」
      - その言い回しは余計な調査工数がかかることを認めている
      - と同時にカプセル化としても失敗していることを認めている
- page_specific_uiが何回参照されているのか(単一参照or共有参照)が不明瞭になる
  - どのページビューからでも参照可能(パーシャル化＝グローバルスコープ化)
  - `page_specific_ui`という名前でソースコード検索して、参照数を数える
    - `form`というパーシャル名だとしたら、このビューからしか参照されていないと断言できるだろうか
      - 不必要な不確実性が出てくる

結局のところ以下のような極めてシンプルな実装の方が厄介な問題を抱えずに済むのです。
(もし後からそのパーシャルが必要になったとしても、より多くの情報を持った後からきた人が判断します。)

```haml
-# app/views/foo/bar.html.haml
-# ...
- if foo
  - if bar
    %p=user.level # ネストがやや深いが、パーシャル化するともっと多くの問題を抱えることになる
    -# ...以下それなりに長いUI実装
-# ...
```


## 結論
他にも山のように事例(たとえば継承、メタプログラミング、mixin、ポリモーフィズムなどの利用判断)を構成できますが、別の機会とします。  
以下が筆者の結論です。

- プロダクションシステムこそシンプルに構成すべきである
  - 可能な限り仕様に対して端的に実装し、風呂敷を広げない努力をする
    - 抽象化手法の選択権を機能拡張時の開発者に委ねる
  - 必要以上に複雑かつ動的な言語機能を用いると、解釈コストは増加し仕様を過剰拘束する結果になる
  - 上記例のように素人が読んでもわかるようなコードからスタートするべきである
    - 要求を達成できるミニマルなコード
    - 要求が複雑化しても段階的にそれを織り込める
- 必然性のない抽象化は回避する
  - 必然性の有無はトポロジー(1vs1 or 1vsN)でおおむね決まる
    - とくに参照・被参照の関係性が1vs1ならば、分離をしない方が可読性は高まる
      - 近接の法則
      - 1vs1のpartialファイル分割した場合、それが不要化した時に残留し続ける
  - ソフトウェアの強みはあとから抽象化できること
    - 情報不足の状況で行なった複雑な抽象化は時の試練を耐えらえず、多くが技術的負債に転換する
    - ハードウェアにはそれができない
      - リリースしたが最後、インタフェースは変更できない(ポイントオブノーリターン)
      - コストダウン及び品質保証要求が抽象化プロセスを必要とする
      - **ハードウェアの真似(事前設計)をする必然性がソフトウェアには存在しない**
        - 事前設計するならば、調停すべきリソースはなんであるかを言語化しなければならない

この記事の要点を一言で要約するならば、「抽象化や設計は、到達目標が明晰に言語化できないならば高確率で負債化する」ということですが、これは一定の非自明性を帯びている点において興味深いものであります。

## 参考文献
[Railsアプリケーションの実装で気をつけている8つのこと](https://blog.recruit.co.jp/rmp/server-side/rails-development-policy/)
この記事にだいぶ助けられた部分が多いです。  
before_actionの乱用をやめたり、コントローラーを分割を推奨すると言う点において同じだし、他も概ねこの記事と同じ意見です。  

[Rustで始めるWebアプリケーション](https://www.itmedia.co.jp/author/235506/)
これは私が書いた記事ですが、現代の静的型付け言語でWebアプリをどのように構成、表現しうるのかの概要を理解しておくとRailsでもそれを応用できます。  
複数言語で比較できると強みと弱みを明晰に言語化できるようになります。  

[Don't use instance variables in partials](https://andycroll.com/ruby/dont-use-instance-variables-in-partials/)
パーシャルではインスタンス変数を使用すべきではないという記事です。  
筆者はパーシャルでないビューについてもインスタンス変数を使わない方が良いと考えています。  
(何がパーシャル化されるべきかは事前予想できないため)  

[Less is Moreとは](https://ideasforgood.jp/glossary/less-is-more/)
ややミニマリスト的な含意を持つ言葉ですが、少なくとも設計文脈においては概ね「徹底した恣意性の排除」を意味します。
恣意性を織り込むとそれを正当化するための大量のルールを必要になりますが、そのようなルールはステークホルダー(ユーザ、チーム、etc)の重荷になります。
設計には「引き算の美学」という概念がありますが、この機微はその職責を負わない限り理解できないかもしれません。  

[ものつくりのセンス ---Taste for Makers---](https://practical-scheme.net/trans/taste-j.html)
よいデザインについてよく書かれたよいエッセイです。  
よいデザインは必然性があるので、シンプルで時として退屈に思えるものです。(Less Is Bore)   
一方でよいデザインから外れるということは、ある(あらゆる)ステークホルダーにとって不利益があるので、それが一時的に支持されることはあっても継続することがないのです。
