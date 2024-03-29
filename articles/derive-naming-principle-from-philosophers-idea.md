---
title: "言語哲学のアイデアからより良い名付けのスタンスを考える"
emoji: "📛"
type: "idea"
topics: ["naming", "philosophy", "design", "math", "logic"]
published: true
published_at: 2023-08-07 06:00
publication_name: "paiza"
---

## この記事を書こうと思ったきっかけ
プログラムを書いていると何かに名前をつけなければならないという状況によく直面します。  
例えば「リーダブルコード」においてもよりよい名付けの重要性が取り上げられています。  
ところが名付けは計算機科学における[Two Hard Things](https://www.karlton.org/2017/12/naming-things-hard/)の1つとされており、2023年現在においても命名者の感性によらないプラクティスは提示されていないように思えます。  
(そのような文書があれば、コメントでぜひ教えてください。)

どちらかといえばエンジニアの仕事は実装(有益な構造やシステム構成)を見出すことが主であり、それを説明したりプログラムを構成する上で、名付けに迫られるという関係にあります。  
名付けについて専門的に考察してきたのは伝統的には(言語)哲学者であり哲学者のアイデアを調べてみることは、名付けの問題を考える材料になるでしょう。  
この記事では偉大なる政治哲学者のJ.S.ミルが提示した直接参照理論と、現代論理学を確立し人工言語(≒プログラミング言語)の意味論確立にも影響を与えたフレーゲのアイデアを紹介しつつ、名付けの問題に関する考察をします。  
またそれに関連する具体と抽象、ソフトウェアデザインに関する洞察を間接的に示します。  

## 直接参照理論の考え方
[直接参照理論](https://en.wikipedia.org/wiki/Direct_reference_theory)は、名前が持つ含意は指示対象(referent)を指すことだけにあるという考え方です。  
この考え方を信じるならば、名前はその指示対象を指す以上の含意を持たないということになります。  
例として「渋谷」という固有名には谷を想起させる単語が含まれていますが、歴史的事実がどうあれ必ずしも谷と関係がある必要はないということです。  
もう1つの例として愛知をギリシャ語に直すとフィロソフィア(=哲学)になるということですが、そのような発想をする人もなかなかいないでしょう。  
(愛知県の命名者にはそのような意図(含意)があった可能性は高いですが、その含意によらず愛知県という言葉はその実体をただ言及(refer)するために使うというスタンスです。)  
この考え方は経験的にも理解しやすく強力な考え方です。

## 直接参照理論をプログラミングの文脈で読み替える
直接参照理論をプログラミングの文脈で解釈するとどのような考えが得られるでしょうか。  
それは「変数名(左辺値)はその実体(右辺値)を指しているもの(ポインタ)にすぎず、あるオブジェクトの識別子以上の意義を持たない」と割り切ることです。  
直接参照理論をそのまま受け入れるのであれば、左辺値は何でもよい(言い換えれば**命名など重要ではない**)ということも受け入れなければなりません。  
これは「命名は重要である」という通説に真っ向から対立する考え方です。

さて、このように考えることでうまくいく事例はあるのでしょうか。  
例えば、RDBMSなどでよく使用されるid(UUID, シリアル値)やGitで使われるようなHash(SHA-1, etc)は人為的な命名を必要としない識別子として非常によく機能します。  
Gitの事例を考えるとコミットハッシュに人為的な何らかの含意を含む名付けをしているようでは、分散VCSとして有益なものにはならなかったでしょう。  

このような人為的な命名を回避して単に識別子を識別子として扱うというアイデアは、プログラマブルに扱える対象を増やすという意味で都合がよいです。  

つまり人為的に名付けていくことは名付け能力がボトルネックになり、さらに抽象化の上限がその恣意性によって決まるということです。

また命名以前に変数のライフタイムを短くすることや、媒介変数を削除するようなリファクタリングによって、コードの明快さを改善できます。  

一方で上記のような直接参照理論の(乱雑な)援用をして、「命名は重要でない」と切り捨てられるのでしょうか。

## フレーゲのパズル
フレーゲは「意義と意味について」という論文において、金星に関する以下のような2つの文を提示し、直接参照理論に対する疑問を投げかけました。
- 明けの明星と明けの明星は同じである
- 明けの明星と宵の明星は同じである

もしも直接参照理論を信じるならば、この2つの文は全く同じ価値(含意)しかないと言えます。  
なぜなら明けの明星と宵の明星のいずれもその指示対象(referent)は金星の実体であり、2つの文は同じ文に還元されると言えるからです。  
(プログラマ的に言えば、ポインタ(参照)解決すると上の文も下の文も全く同じであるといった方がやや親しみやすいです。)  
一方でフレーゲは上の文は先験的に自明な文なのに対して、下の文は明け方に観測される星と、日没に観測される星が実は同じであるという非自明性があると考えました。  
フレーゲは宵の明星、明けの明星という名前には単に指示対象を指す以上の意義(Sinn)があると考えたのです。
事実、このような提示をされると我々は「後者の文の方が前者の文よりも情報量が多いと言えそうだ」という直観が働きます。  
直接参照理論かフレーゲの理論どちらかが正しいのかの相手をしていたら、記事が未完で終わるのでこの探究の仕事は哲学者に委ねます。  
この記事では名前にどのような意義(情報量)を織り込めるのか、あるいは織り込むべきなのかを考察します。

## 名付けの意義を考える
金星には「明けの明星」「宵の明星(一番星)」「金星」「Venus」「太陽系第二惑星」などたくさんの呼び名があります。  
この名前を眺めていくと、その星の**観察事実に基づく対象のある側面(アスペクト)についての記述**がその名前に織り込まれていることに気が付きます。  
例えば、太陽系の惑星の順序について言及する場合は太陽系第二惑星という呼び名がうまく機能し、宵の時刻に煌々と輝く一番星のことについて言及する場合は宵の明星という呼び名がうまく機能するといった具合です。  
逆に一番星のことを語りたい文脈で「太陽系第二惑星」という呼び名を使うと、文としては正しいですが文脈がずれていて我々がいう「可読性が低い」状態になります。  

意義という考えに基づき名前をつけるならば、実体の観察事実やアスペクトを実体から抽出(捨象)し、それに基づき名前をつけることが有益であるという立場をとることになります。 

例えばGitの中にもコミットやブランチという名前が与えられている概念があります。コミットは編集データをGitに記憶させるという意義があり、ブランチはコミット列が分岐しうるという含意(アナロジー)があります。

一方で現実の枝がマージされることは想起しにくいので、川やレールのようなアナロジーや、コミットラインのようにアナロジーを使わない命名もありえたと考えることもできます。こう考えるとブランチを作るときは我々はその意義を意識するけれど、マージするときにはブランチを「渋谷」と同様のものとして取り扱っているという複雑な思考を組み立てているということが言えます。

意義に基づく名付けは、その名付け者の表現したい対象や文脈に強く依存することで、これをメリットと考えることもデメリットと考えることもできます。  
https://www.youtube.com/watch?v=V40sMUAE5ek   
上記の動画では、意義に基づく名前の合理性は文脈(開発指揮者orユーザ)によって逆転してしまうということを示唆しています。  
メリットとしては、ユーザに対して直観的で一貫性のある操作(命名)体系を提示できるということです。これは命名体系がユーザビリティという価値に転換される好例です。  
デメリットとしては、開発文脈と操作文脈のそれぞれに対して最適化された複数の命名体系を維持管理する必要が出てくるということです。  
複数の命名体系があるということは秩序だって管理されなければ、**メンバー間での意思疎通を損ねる**原因になります。  
また一貫性のある命名体系を構成するには、その全体像を理解することがとても重要です。  
アジャイル開発を採用する場合は、システム仕様を初期段階で固定できず、命名体系を逐次的に修正しながら発見していくことになることは意識されなければなりません。(上記の動画の例は仕様が固定化されているからこそ可能なことです。)  

## 名づけにおいて何が言えるか
2つの相反するどちらの立場を信じるにしても確実に言えることがあります。  
それはトートロジーな命名をつけても何の情報も読み手に与えることはないということです。  
例えば教義に従いマジックナンバーを避けるがために以下のような変数(定数)定義をしてもあまり意味がありません。  
```javascript
const THREE = 3;
const ENUM_VALUE_A = 'enum_value_a';
```
このように定式化すれば、値(実体)を変更する際に一括で変更できるではないかということですが、そうした変更結果は例えば以下のようなものになります。
```javascript
const THREE = 5;
const ENUM_VALUE_A = 'enum_value_b';
```
これでは変数名自体が認識形成を阻害するようになります。  
マジックナンバーを形式的に避けるがために、トートロジーな名前をつけるぐらいならばその値を直接参照したほうが良いということは言えそうです。 

![](/images/articles/derive-naming-principle-from-philosophers-idea/tautology.png =500x)*こういった提案を無批判に受け入れるとあとが大変*  


もっと言えば変数名に対する良いスタンスは、変数名に意義を一切持たせないか、変数名に明確な意義を持たせるかのいずれかであると筆者は考えています。例えば以下の通りです。  
```javascript
const 77de68daecd823babbb58edb1c8e14d7106e83bb = 3; // 一切の意義を読み手に感じさせない、しかしこれなら直接3が必要なところに3を放り込んだほうが良さそうだ
const GRADUATE_MONTH_DEFAULT_IN_JAPAN = 3; // 3という値の文脈上の意義を明確に(不変かつ一意となるように)記述する
```
後者の例では、数ある3という数字のユースケースの中から特定のユースケース(アスペクト)を切り出しそのことに関心があることを表明します。  
その特定のユースケースについて賛同が得られれば再利用されますし、そうでなければ別の名前が割り当てられるでしょう。  
(実際には命名変更のコストもどこまで命名を精緻化するかの指標になります。ライフタイムが短いローカル変数の命名など誰も気にしません。)

## 実体の可変性を考慮する
私は怠惰を美徳とするプログラマであるので、直接参照理論の立場から名前を取り扱うことをまず考えます。  
そうすれば名前を不変として取り扱うことができます。  
そうするとリネームに起因するリファクタリングをしなくて済むと考えられます。  
この考えに基づき以下のようなcssについての思考実験を考えてみます。  
```css
.f48d8cd9902f39c4f06da62bdbbfdb88ff3eeb698 {
    padding-top: 1rem;
    padding-bottom: 1rem;
}
```
このcssをhtmlを埋め込むことにより、CSS ModulesやCSS in JSなんて使わなくてもある実体を固定的に(不変)参照するクラススタイルが提供できそうです。    
実際tailwindなどはこのスタンスに立脚していると考えられます。  
しかしこの考え方がうまく機能するのは、実体(右辺値)を不変な値として扱い続けるという前提がある時です。  
```css
.f48d8cd9902f39c4f06da62bdbbfdb88ff3eeb698 {
    padding-top: 1rem;
    padding-bottom: 1rem;
    font-size: 1.5rem;
}
```
のような実体の変更(更新)を行った時、スタイルが想像できない形で崩壊します。  
メカニカルな識別子による直接参照は、それがどのような実体を指しているのかが極めて明快になる点において有益です。  
それは裏を返せば実体(How)をカプセル化することがなく、HowとWhatが密に結合します。  
例えば新しいCSSプロパティを用いてCSS実装(実体)を改善したいと考えた時に、利用箇所(ユースケース)がその実体のどのようなアスペクトが必要であったのかを区別しえません。それゆえある利用箇所では実装を差し替えてもうまく動作し、別の利用箇所はうまく動作しないということが起こります。これが直接参照の明快さの代償と呼べるものです。  
一方でそれに対抗する手順を戦略的に用意していた場合は、抽象化を人手で構成するよりもはるかにスケールアウトしうることはやはり指摘されなければなりません。

## より良い名付けのスタンスを考察する
改めて2つの立場のメリットデメリットを整理します。
- 直接参照理論
  - 名付けに求められるのは識別子としての役割のみ
    - 名付けを機械的に割り振ることができる(UUID, SERIAL, SHA1, etc)
    - 実体が不変ならば合理的に機能する
      - 好例: Git commit, PRIMARY ID
      - ハマればスケールアウトする
  - 参照と実体が密(1vs1)結合となる
    - 名前を用いたカプセル化(間接参照による抽象)を利用できない
      - 利用箇所がその実体にどのようなアスペクトを期待しているのかを表明できない
- フレーゲの理論
  - 名前を用いたカプセル化(間接参照による抽象)を利用できる
    - 利用箇所がその実体にどのようなアスペクトを期待しているのかを表明できる
      - 名付け手の認識バイアスが名前にのってしまう
        - 名付け(参照)が安定しない
          - それを効果的に生かした作品は多い、「君の名は」など
          - コードがミステリー小説化する原因にもなり得る
  - 名付けに解釈可能な意義を織り込む必要がある
    - 実体の変更に対して適切な再命名が必要になる
    - 名付けは人的作業となり
      - ボトルネックになる

考えなければいけないことは何のために抽象化をするのかの目的意識をはっきりさせることだと言えます。  
ユーザに対して明快な操作体系を提供するために名付けをするのであれば、その管理コストについて考えても仕方ありません。(言い方を変えればそれが**デザイン**です。)  
一方でそのような管理コストや共通認識の確立といったものを考慮する必要があるのであれば、以下のようなことを考えてみるとよいでしょう。
- 直接参照のままで良いものは直接参照として取り扱う
  - 機械的に名付けられるのであれば、人為的な名付けを回避できる
  - 直接参照可能なものは命名を回避し、代入してしまうでよい
- 間接参照を用いる場合は、一意に要素を特定でき変更可能性が低い呼称を選択する
  - GRADUATE_MONTH_DEFAULT = 3 よりはGRADUATE_MONTH_DEFAULT_IN_JAPAN = 3の方が長期的に見て安定する
  - **3をどのように(HOW, 実装)作るかではなく、3を何(WHAT, ユースケース)として扱いたいかを名前にすることを考えてみる**
    - 常にそうすべきとは限らないが、いい名前を思い浮かぶことがある
  - 「太陽系第二惑星」よりは「金星」か
    - 公転軌道が変わる可能性を考慮するとそうなる、金星が金色でなくなる可能性はあるが金星と呼ばれ続ける方に賭ける  
    - Venusは多義的なので、一意性の観点から金星の方が良い呼称、ただやはり`const kinsei = ...;`とはしないだろう。
  - 静的型付けを使用するなら、名称変更コストは低いので気にしなくてよいことが多い
- 間接参照を用いる場合は、消失しにくい意義(実体の不変なアスペクト)を探し出す
  - 命名の意義が縮退、消失すると直接参照と差はなくなる
    - 愛知で想起されるのはフィロソフィではなく自動車やひつまぶしである
    - マージをするときブランチという命名の意義を忘却している(ブランチが「渋谷」化している)
- 間接参照を用いる場合は、他に命名したものを用いて、追加の命名を避けられるか考える
  - ブランチよりはコミットラインのほうがバイアス(による認識のズレ)を避けられる可能性がある
    - 集団でプロダクトを構成する場合は特に意識する価値がある(プロジェクトマネジメントに有益)

もっと精緻化された内容や、もっと具体的な事例も盛り込みたかったですが、単一記事としてのサイズ限界に到達している(紙面の余白がない)ので、またの機会とさせていただきます。

実際のところ上記のような考えはプログラミングよりも、要件定義や対外的な説明を行う上で役に立つかもしれません。

## 参考文献
[Element of clojure](https://www.amazon.co.jp/dp/0359360580)

フレーゲの理論が命名作業においてなんらかの示唆を与えうるというコンセプトは6年前に読んだこの書籍からインスパイアされました。

[言語哲学大全I](https://www.amazon.co.jp/dp/B0C84PNZ3X/)

この話を取り上げる上で、「意義と意味について」の「意義」を調査するために読みました。この本を読むとクリプキの考え方についても、学ぶ価値がありそうですが、それを名付けに転用可能なレベルで整理するにはもう少し時間がかかりそうです(筆者のモチベーションも尽きているようです)。  
また現代のLLMのような確率的論理モデルの産物が自然言語をなめらかに生成しうるのは、フレーゲが提示した文脈原理や合成原理といった仮説が関係しているのかもしれません。

[テセウスの船](https://ja.wikipedia.org/wiki/%E3%83%86%E3%82%BB%E3%82%A6%E3%82%B9%E3%81%AE%E8%88%B9)

我々が可変な実体を間接参照(名付け)を使って捉える場合、何を何と同一と捉えるか(同一性)という問題にさらされます。
我々は実は名付けに悩んでいるようで、実際には同一性に関する仕様の矛盾や策定の難しさに晒されていると言い換えられるでしょう。

[リーダブルコード](https://www.amazon.co.jp/dp/4873115655)

https://overreacted.io/goodbye-clean-code/

https://en.wikipedia.org/wiki/Leaky_abstraction
