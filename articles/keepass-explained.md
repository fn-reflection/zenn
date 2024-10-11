---
title: "KeePassXCでパスワードもMFA(TOTP)もssh秘密鍵も管理する方法"
emoji: "🗝"
type: "tech"
topics: ["security", "password", "keepass", "keepassxc", "secret"]
published: true
published_at: 2023-03-19 09:30
publication_name: "paiza"
---

[前回の記事](https://zenn.dev/naokifujita/articles/password-explained)では、認証方法に関する整理とパスワード管理のツールについてシンプルかつ網羅的に記述しました。  
今回の記事では、[KeePassXC](https://keepassxc.org/)というツールがどのように活用できるのかをシンプルかつ要点を外さず紹介するとともに、パスワード管理に対する議論の基礎としても使えるように構成しています。


## KeePassXCとは？
KeePassXCは、[KeePass Password Safe](https://keepass.info/)というプロジェクトから派生したパスワード管理ツールです。   
KeePass本家も非常に良いツールですが、XCの方が様々なOS(Win, Mac, Linux)でシームレスに動きます。  
またKDBXという(本家からも開ける)互換性の高いファイルフォーマットでパスワードを保存でき、アプリケーションからパスワードを参照するという使い方([例](https://github.com/fn-reflection/testcases/blob/main/rust/src/unittest/keepass.rs#L60-L71))もできます。   
KeePassXC自体はAndroidやiPhoneに対応してないですが、KeePass2Androidなどの兄弟アプリからKDBXを開くことができます。  
:::message 
KDBXは4.0が最新バージョンです。バージョンアップは簡単にできますが、ファイル生成時には最新バージョンで作成した方が良いでしょう。
:::

## KeePassXCで使える資格情報
KDBXファイルは生成時に暗号と復号に用いる情報(資格情報)を以下の3つのうちから選べます。

![](/images/articles/keepass-explained/three-credential-types.png =500x)*選べる3つの資格情報*  
暗号強度は一般に、(マスター)パスワード＜キーファイル＜YubiKey(FIDO)ですが、資格情報をなくすと**自分も復号できなくなる**ので、ここの運用だけは良く考える必要があります。
- パスワード再発行するから問題ない
- バックアップ手段を計画する

## KeePassXCで使える便利機能
以下に私が便利だと思う機能を並べます。(いくつかはあとで詳述します。)
- パスワード管理機能
    - これがないと始まらない
    - 検索もスムーズ
- パスワード生成機能
    - 指定文字数、文字種のランダムパスワードを生成
    - とりあえず64とか128文字のパスでアカウントを保護するという**習慣が定着する**
- ワンタイムトークン(OTP)管理機能
    - Google Authenticatorとかで管理するアレ
    - **これでスマホを水没させても安心**  
    - ビジネスユーザにとっても魅力
- ショートカットコマンドによる自動入力機能
    - ブラウザ統合されたツールよりは不便だが十分便利
- ssh-agent機能(エンジニア向け)
    - ssh秘密鍵をKeePassXCに内包させつつ電子署名できる
    - **ssh秘密鍵を素置きしなくて良くなる**

その他特筆すべき機能として以下のようなものもあります。
- 変更履歴記録
    - あまり使わないが**操作ミスしても良い**という安心感
- CSVインポート機能
    - 別の管理ツールからの移行がスムーズ
- ブラウザ統合機能
    - クラウドベースのツールと似たようなUXになる

:::message alert 
少し気になるニュースとしてKeePassXCには[Microsoft Storeに偽物が登録される](https://forest.watch.impress.co.jp/docs/news/1445178.html)という騒動もあったようですが、現在は対策されているようです。

(セキュリティツール周辺は治安が悪くてかなわないので、[公式サイト](https://keepassxc.org/)からインストールした方が良いです。) 
:::

パスワードの生成と管理については、特筆すべき事項はない(使えばわかる)ので割愛して、ワンタイムトークンの管理から詳述します。

## タイムベースドワンタイムトークン(TOTP)とその管理
パスワード単体だけでは脆弱であるということで、近年普及してきたワンタイムトークン。  
MFA(多要素認証)の構成には必須ではないはずですが、MFAの代名詞と呼べるほどよく見かけます。  
6桁の数字が30秒ごとに切り替わるので、巧妙なトリックがあるかのように思えますが、隠されたテキストデータ(≒パスワード)と時間から6桁の数字を生成しているだけです。(エンコーダを書くのも難しくなかった記憶があります。)  
その性質を踏まえると、**パスワード用のデータベースと同じデータベース**で管理すべきではないでしょう。  
KeePassXCでのTOTP管理は以下の手順を踏めば理解できます。

![](/images/articles/keepass-explained/totp-setting1.png =300x)
![](/images/articles/keepass-explained/totp-setting2.png =300x)
*TOTP秘密鍵をKeePassXCで設定すると*  

![](/images/articles/keepass-explained/totp-display.png =300x)
*いつもの奴が見れるようになる*  

![](/images/articles/keepass-explained/totp-explained.png =300x)
*詳細設定からも秘密鍵(secret)が確認できる(が極力隠す)*  

:::details TOTPの秘密鍵はどこに？
登録に必要な秘密鍵は、多くの場合バックアップや移行手順を踏めば確認することができると思います。  
ポピュラーなGoogle Authenticatorについては特殊で、`otpauth://`から始まるsecretを含む情報を直接取得できないです。  
[デコーダ](https://github.com/dim13/otpauth#example)を利用して`otpauth-migration://`から始まるテキストを変換することで秘密鍵をスマホから取り出すことができます。
:::

## 自動入力機能について
KeePassXCでは、大きく分けて
- ブラウザ拡張(Chrome Extension)による自動入力
- ショートカットによる自動入力
- エントリー選択による自動入力(かなり煩雑)

の3つの方法が選べます。  
利便性や総合的な観点からすればブラウザ拡張の方が便利で、ブラウザ拡張自体もKeePassXCの開発組織が開発を進めています。  
気になる点があるとすれば、拡張自体の脆弱性に晒されうる([Bitwardenの例](https://gigazine.net/news/20230309-bitwarden-flaw-steal-passwords-using-iframes/))などが気になるところでしょうか。  
ショートカットによる自動入力は、拡張をインストール(検証)しなくても良いですが、**ブラウザウィンドウの\<title\>に一致するエントリーを見つけてくる**という関係上フィッシングに弱いと感じます。

![](/images/articles/keepass-explained/phishing-risk.png =500x)
*ドメインが異なってもウィンドウタイトルが一致していれば反応する(ブラウザ拡張だとURLベースで反応する)*  
まあ**絶対的にいい方法はない**ということなので、個々人のリスク管理の観点から方法を選択することになります。  
またショートカット入力については、
1. ユーザID入力
2. エンターキーを押す 
3. パスワード入力
4. エンターキーを押す  

という入力シーケンスがデフォルトで適用されています。  
![](/images/articles/keepass-explained/custom-key-sequence.png =500x)

これはカスタマイズ可能であり、例えばMicrosoftのアカウントサインインには、
```
{USERNAME}{TAB}{TAB}{TAB}{TAB}{ENTER}{DELAY 2000}{PASSWORD}{ENTER}
```
といったカスタムシーケンス([docs](https://keepassxc.org/docs/KeePassXC_UserGuide.html#_auto_type_actions))を定義できます。

## SSH Agent連携機能(エンジニア向け)
KeepassXCはSSH Agentと連携しssh秘密鍵を管理できます。  
ssh秘密鍵を暗号化データベースに格納しつつ、データベースの開閉状態に応じて暗号鍵の登録・解除(ssh-add)させるような設定も行えます。
これにより平文で秘密鍵を置くよりかは、幾分かセキュアな運用が構成できます。

![](/images/articles/keepass-explained/register-ssh-secret1.png =500x)
*ssh秘密鍵を添付ファイル欄で添付して*

![](/images/articles/keepass-explained/register-ssh-secret2.png =500x)
*データベース展開時にssh agent keyを登録するよう設定した。公開鍵(.pub)はKeePassXCが導出している*

![](/images/articles/keepass-explained/register-ssh-secret3.png =500x)
*データベースを開き直すとキーが登録された*

:::details SSH Agentを使う場合のIdentityFileの指定
ややマニアックな話になりますが、
```
# in ~/.ssh/config
IdentityFile ~/.ssh/id_rsa
```
みたいな指定方法を知っている人がいるかもしれません。(これにより指定鍵でのみ電子署名を試みることになります。)  
このような指定を外せば、持っている鍵で片っ端から電子署名を試すことになりますが、あまりスマートではないと感じる方もいると思います。  
秘密鍵ファイルを削除する場合、代わりに公開鍵を指定することでエージェントが保持しているキーを指定して使わせることができます。
```
# in ~/.ssh/config
IdentityFile ~/.ssh/id_rsa.pub
```
↓manにもそう書いてあります。  
https://github.com/openssh/openssh-portable/blob/610ac1cb077cd5a1ebfc21612154bfa13d2ec825/ssh.1#L294-L300

:::

## バックアップ
公式では、自動同期するクラウドドライブに置くというユースケースが想定されています。  
https://keepassxc.org/docs/#faq-cloudsync  
そもそもバックアップを取らないとか、もう少しひねりを加えて同期するなどのアイデアは検討の余地がありますが、運用のベースアイデアとして悪くはないように感じます。

## まとめ
今回の記事ではKeePassXCの有益な機能にしぼって紹介をしました。  
使わないにしても、こうしたものが世の中に既にあると理解しておくと、よりよいデータ管理に結びつけることができると思います。   
今回の記事について追記すべき内容や修正案等がございましたら、ぜひコメントを残していただけると幸いです。よろしくお願いいたします。
