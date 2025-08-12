---
title: "Dockerの基礎1: 最短理解のための重要な概念"
emoji: "🐳"
type: "tech"
topics: ["docker", "infra", "container", "vscode"]
published: true
published_at: 2025-08-12 18:00
publication_name: "paiza"
---

## はじめに
ソフトウェアの現場では例えば「実行環境の再現性の確保」あるいは「ホストよりも制限された環境でのプログラム実行」のためにDockerを用いることがよくあります。  
前者は効率的なソフトウェアデリバリーを支える根幹であり、後者はOSSなどから手に入れたツールを制限された環境で実行し、相対的に安全性を高めることができます。

本記事では、Dockerの基本概念を[公式のGet Started](https://docs.docker.com/get-started/)を参照しつつ、Dockerをうまく扱う方法についてのインサイトを与えます。  
Dockerの基礎力を養いたい人はGet Startedを一通り読むことを推奨します。そこには理解すべき概念がハンズオン形式でまとまっています。  
とはいえボリュームはあるので読むキッカケが無ければ読み切るのは難しいです。  
この記事はさらに簡略化して重要概念をピックしつつ、何を集中的に学ぶと効率がよいかを明らかにします。

## Dockerのインストール
Windows, Macの場合はDocker Desktopを入れるのが簡単です。  
https://docs.docker.com/get-started/get-docker/  

Linuxの場合はDocker Engineだけを入れるだけでも十分です。  
https://docs.docker.com/engine/install/

## Dockerの基本概念
公式のGet StartedではDockerの基本概念がどのように説明されているかを抜粋します。  
Dockerの公式ドキュメントはやや抽象的な表現が用いられていますが、安易な比喩を使わないのにはそれなりの理由があると考えるべきでしょう。  
(筆者が考えるに安易な比喩は応用が効かない。)

### コンテナとは
- ホストマシン(Mac, Windows, Linux, etc)からゆるく分離されたプログラムの実行環境
- アプリケーションコンポーネントの構成単位
  - コンポーネントの例：Reactアプリ, Python APIサーバー, データベースなど
  - 各コンポーネントをそれぞれ分離したプロセス(コンテナ)として実行できる
- 高度な移植性、ポータビリティがある
  - クラウド、ローカルコンピュータ、データセンターなど実行環境(インフラ)を問わない
  - ホストマシンに何が事前インストールされていたかに依存しない

### イメージとは
- Dockerコンテナの作成手順書
  - コンテナを実行するための設定情報が含まれている
- Dockerfileを作成し自分でbuildできる
- 既存のイメージをそのまま利用できる
- 複数の(イメージ)レイヤーから構成される

### (イメージ)レイヤーとは
- ファイルの追加、削除、変更といった**ファイルシステムの変更のセットを表す**
- Dockerfileの各命令(FROM, RUN, COPY, etc)がレイヤーを構成する
- 各命令により積み重ねられ、(ユニオン)ファイルシステムを構成する
- 一度作成されたレイヤは不変(イミュータブル)である
  - **レイヤに変更が必要な場合は新しくレイヤを作成するしかない**
  - 各レイヤはビルドキャッシュできる
    - ビルド結果が変わらなければ既に作成されたレイヤを使いまわせる
    - ビルド結果が変化する場合はそれより上にあるレイヤを使いまわせなくなるため再ビルドが必要

![](/images/articles/explain-docker-simply1/show-image-layer-by-docker-image-history.png =500x)
*docker image historyコマンドでpython:bookwormイメージのレイヤーを表示、レイヤが下から上に積まれていく。*

![](/images/articles/explain-docker-simply1/show-image-layer-by-dive.png =500x)
*diveコマンドを使えば各レイヤーが持つファイル、ディレクトリも表示できる。buildにおけるRUNというのはファイルシステムの書き換えに過ぎないということがわかる。*

### (イメージ)リポジトリとは
- コンテナイメージの集合
  - さまざまなイメージタグ(例：latest, 24.04など)を指定することで特定のイメージを取得できる
- 代表例: [Docker Hubのubuntuリポジトリ](https://hub.docker.com/_/ubuntu)

### Dockerとは
- コンテナのライフサイクルを管理するツールとプラットフォームを提供する
- インフラをアプリケーションと同様に(テキストで)管理できる(Dockerfile + Docker Buildによるイメージ定義をサポートする)

### ミニマルに考える
Get StartedにはDocker Compose、Docker Network、Docker Volume、マルチステージビルドなどその他理解するべきコンセプトが多数現れます。  
たしかにそれらは重要なのですが、Dockerの活用する上で絶対に避けて通れないであろう概念はコンテナ、次にイメージとレイヤーです。  
そして**docker build(イメージとレイヤーの操作コマンド)とdocker run(コンテナを作成、起動するコマンド)の理解がDockerを理解する最短経路**になります。  
例えばマルチステージビルドの必要性は、docker buildコマンドの理解の延長上にあり、Docker Network, Docker Volumeはrunを使いこなす過程で十分理解できます。 

Docker Composeではbuildとrunの両者を扱いやすく記述でき、利便性が高いですがこの記事シリーズでは触れません。  
build(タイミング)とrun(タイミング)を明確に区別することがDockerを効果的に学習、利用する上で重要です。   
(例えばbuildのRUN命令でデーモン起動する技術記事をたまに見かけますが、多くの場合誤りです。とくにデーモンがなくても処理が動くケースでは誤りを気づきにくいです。)  

またDockerには例えばコンテナを停止するdocker stopコマンドやイメージを削除するdocker image rmコマンドなどもあります。  
Dockerを管理するコマンドは他にもたくさんありますが、これらはDocker DesktopなどGUIまたはDocker Composeで管理した方が楽なのでこれも割愛できます。  
これらは使っていけば慣れていくし、コマンドが必要であればそのGUI操作に対応するコマンドをその都度探せばいいだけになります。  
(私個人の私的な環境ではDocker Desktopを入れていないのでVSCodeのDocker拡張を利用しています。)
<!-- textlint-disable -->
![](/images/articles/explain-docker-simply1/docker-management-by-vscode-extension.png =500x)
*VSCode拡張を利用したDocker管理、コンテナやイメージ管理は概ねこれでカバーできる*
<!-- textlint-enable -->


### まとめと次の記事の予定について
今回はDockerの重要概念は何かと、それを扱うbuildとrunコマンドの理解の重要性を強調しました。
つづく記事では、docker buildとdocker runコマンドを深掘りする予定です。  
(最初はこの記事に統合する予定でしたが、長くなるか薄味になるかの2択になりそうだったため記事を分けます。)  
次の記事はrun編かbuild編になるでしょう。  
順序で考えるとbuildが先なのですが、Dockerを利用するだけならrunを理解するだけでも十分なので、run編を先に書く想定です。
