# Note ミクシィ Git 検収

https://mixi-developers.mixi.co.jp/22-technical-training-5fc362a9dc41

参考：

https://git-scm.com/book/ja/v2/Git%E3%81%AE%E5%86%85%E5%81%B4-Git%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88

## Git によるチーム開発のいろは

世の中の version control system(VCS)には集中型と分散型の 2 つがある。

分散型：リポジトリの全履歴を含めた完全なコピーをローカルに持つ

git branch はチーム開発を実現してくれる機能だが、むやみに使うと複雑なコンフリクトを発生させる。
git branch の方法論に則ろう。

集中型：リポジトリは完全にサーバが管理している

なので自分が編集したいファイルだけをサーバからローカルへコピーして、
編集後サーバへ push する。その間他の人間は自分が編集してるファイルを編集できない。
これはチーム開発に向いていない。

#### Git Flow

登場ブランチ：

- master(main)
- developer
- feature
- release
- hotfix

開発は developer ブランチで行い、master はリリース時に初めてコミットされる。

master のコミットが常に製品の最新のリリースになる。

master には(チームの共通認識として)「出来上がったものだけ」をコミットするのである。

Feature ブランチは機能開発のために切るブランチ。

1 機能 1 ブランチの法則の元、ブランチを developer ブランチから切って開発する。

機能開発が完了したら developer ブランチにマージする。

機能が大きい場合は Feature ブランチからさらに Feature ブランチを切る場合もある

Release ブランチは、master へ merge するための準備をするブランチ。

準備とは、

バグ修正のみ、version 表記やタイムスタンプを押すために更新する、Apple や Google の審査のために使うなど

Release で新規追加するのはご法度。

Release を経て初めて master へコミットされる。developer から master へコミットすることはあり得ない。

hotfix は本番環境で発生したバグのうち、緊急性の高いものを修正するためにある。

developer 以外で唯一、master から直接切られるブランチである。

つまり、

基本的に developer ブランチを開発中は相手にして、

master ブランチにコミットすることは非常に重要なコミットを行うことを意味する。

各ブランチには扱いのルールがあってそれらを完全に区別することで安全に開発を進める。

#### Atlassian の GitFlow ワークフローの話

https://www.atlassian.com/ja/git/tutorials/comparing-workflows/gitflow-workflow

> Gitflow とは、元来は Git ブランチを管理するための破壊的で斬新な戦略のレガシー Git ワークフローです。Gitflow の需要は落ち込み、トランク ベースのワークフローが利用されるようになっています。現在ではこれが最新の継続的なソフトウェア開発のベスト プラクティスおよび DevOps プラクティスと見なされています。

git-flow は Git のツールセットである。OS へインストールしてつかう。

git-flow は git の拡張機能のようなものらしく、先の話の master,develop,feature,release,hotfix の開発方法に特化した Git コマンドを用意してくれるらしい。

なので別になくてもいい。

gitflow は git のプラグインでもあるし、Git を使ったバージョン管理開発手法のことでもある

具体的な開発手順:

```bash
# 開発はじめ
# ローカルリポジトリ
git branch develop
git push -u origin develop

# チームの他の人
# developブランチに移動してから
git switch develop
# developブランチの最新に更新してから
git pull origin develop
# Featureブランチを切る
git branch feature-toggle-button
git switch feature-toggle-button
# Featureブランチで機能開発
# 機能が開発し終わってFeatureにてコミットまでしたとして
# developへ戻る
git switch develop
# developをこれまた更新して
git pull origin develop
# 作成した新機能をマージする
#
# 多分マージする前に先輩とかチームに確認とかあると思うけど
git merge --no-ff feature-toggle-button
# リモートへプッシュ
git push origin develop
```

とにかく参考 URL の先に詳しく載っている。

今回得たものは、

ブランチを切って開発するときに、ブランチをわざわざ master へ戻さないままリモートリポジトリへプッシュして、ローカルリポジトリはそのリモートのブランチから取得できるということを知ることができたこと。

### GitHub の機能

#### Issue

チーム開発に便利な機能

Issue は気になったことをマークダウンで書いておくところ

ここに実タスクを並べたり、議論の土台を作ったり、相談事を書いたり

#### Pull Request

重要な機能。PR と略される。

特定のブランチや fork 元のリポジトリに、「私が行った作業を取り込んでください」と申し出る機能。

「取り込んでください」なのに pull request である（merge でない）

お願いされた側はよく吟味して大丈夫そうなら merge してあげる

出すときの注意：

PR はできるだけ細かく。レビューする側が大変でバグの温床になりやすい。

PR する側はコミットログをできるだけきれいにしておこう。

見る側の注意：

機械的にチェックできる部分は CI に任せる

人間が見るべきポイントにしっかり集中する

レビュー表現は相手の気持ちに立って

#### Actions

GItHub が提供している CI/CD 環境。

他のサービスと連携の必要がない。

簡単に使える。

パブリックリポジトリの場合無料で使える。

hook したいイベントとワークフローを書いた yaml を`.github/workflow/`に配置すると勝手に走ってくれる(?)

#### Projects

Trello みたいな看板タスクツール

講師は利用しているところは少ないのでは？とのこと

#### Wiki

シンプルな貧弱な Wiki

#### Security

主に 3 つの機能についていじるタブ

- Security Policy
- Security Advisories
- Dependebot

#### Settings

最もよく使うのが`Branch protection rules`

指定した正規表現にマッチした brnach に対して PR を経ない直接 push や force push を禁止するよう設定できたりする

master や develop ブランチには何らかのルールが施されているはず。

## Git の内部構造

#### git add したときに何が起こっているか

「コミットさせたいファイルを index に登録している」

index とは.git/index である

index の中身を確認するコードがある

```bash
# README.mdをaddしたとする
$ git ls-files --stage

# ファイルの種類＋パーミッション blobハッシュ コンフリクトフラグ ファイル名
# という並び
10064 e23fs43232fsr432fse232 0 README.md
```

Git は様々なデータをオブジェクトと呼ばれる概念で表現しており

オブジェクトには次の 4 つがある

- commit: コミット情報が入っている
- tree: ディレクトリの情報が入っている
- blod: ファイルの情報が入っている
- tag: annotating tag の情報が入っている

オブジェクトの実態は次の場所に zlib 圧縮形式で保存されている

`.git/objects/`

先のやつなら blob オブジェクトが

`.git/objects/e2/3fs43232fsr432fse232`に保存されている

各オブジェクトの中身を確認するためのコマンドがある

`git cat-file -p <object-hash>`

あたらしくファイルを作ってオブジェクトを確認してみると

```bash
$ echo "This is README" > README.md
$ git add README.md
$ git ls-files --stage
100644 171cd12d63717a3f594589b2313e9470fcd1003c 0       README.md
100644 d158b023ecbf9e0a69daa8536f04273587396977 0       initial-commit.txt

# 新たにファイルを保存する
$ echo hoge > hoge.txt
$ git add hoge.txt
$ git ls-files --stage
100644 171cd12d63717a3f594589b2313e9470fcd1003c 0       README.md
100644 2262de0c121f22df8e78f5a37d6e114fd322c0b0 0       hoge.txt
100644 d158b023ecbf9e0a69daa8536f04273587396977 0       initial-commit.txt
```

**git add は基本的に index の更新と blob オブジェクトの生成しかしていない**

ディレクトリを作成してみる

```bash
# huga/huga.txtを作ってaddした
$ git ls-files --stage
100644 171cd12d63717a3f594589b2313e9470fcd1003c 0       README.md
100644 2262de0c121f22df8e78f5a37d6e114fd322c0b0 0       hoge.txt
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0       huga/huga.txt
100644 d158b023ecbf9e0a69daa8536f04273587396977 0       initial-commit.txt
```

ディレクトリの追加が起こったけれど、tree オブジェクトは生成していない

#### git commit したときに何が起こっているか

##### tree object

主にこの 3 つを行っているらしい

1. index から tree オブジェクトを生成する
2. commit オブジェクトを生成する
3. HEAD を新しい commit ハッシュへ書き換える

tree オブジェクトは Git が管理するあらゆるコンテンツへの多くのポインタを含む

コミットをすると、差分だけでなくて、リポジトリのルートディレクトリを含むすべてのディレクトリの tree オブジェクトを自動で生成する。

先の add してあったものをコミットして、tree オブジェクトをのぞいてみる

```bash
# master^{tree} のシンタックスは、
# master ブランチ上での最後のコミットが指しているツリーオブジェクトを示します。
# つまり最後のコミットのtreeオブジェクトをcat-fileしてねみたいなコマンド
$ git cat-file -p master^{tree}
100644 blob 171cd12d63717a3f594589b2313e9470fcd1003c    README.md
100644 blob 2262de0c121f22df8e78f5a37d6e114fd322c0b0    hoge.txt
040000 tree b5a33ea70f1cb1d361014f0f88a2f724fb9522ca    huga
100644 blob d158b023ecbf9e0a69daa8536f04273587396977    initial-commit.txt
```

つまり、

コミットするとそのディレクトリのファイル情報は blob オブジェクト

ディレクトリの情報は tree オブジェクトに保存する

blob オブジェクトはファイルの情報、tree オブジェクトは blob オブジェクトと下層のディレクトリの tree オブジェクトを保存する

なのでコミットするとその時点のスナップショットを保存できるのである

ファイルの状態、ディレクトリの状態をすべて記録するから。

つまり tree オブジェクトはある時点のスナップショット情報を参照できるということにもなる

上記の各ファイルをそれぞれ変更してまたコミットしてみる

```bash
# 次のコミットしたあとのルートディレクトリのtreeオブジェクトの中身
$ git cat-file -p master^{tree}
100644 blob 4cf5010e7b78cfd61d33983044299260d21b9189    README.md
100644 blob 1904c092b649dc54f3c8fc931acb0ca5bb952c3b    hoge.txt
040000 tree 8406deefc043c8f56777cd1ca55610536442d1f3    huga
100644 blob 5c6af654a3b833764a26e8baf849312555134997    initial-commit.txt
```

先のコミットと最新のコミットで各オブジェクトの SHA-1 がまったく異なるのがわかる

ということは、たとえばコミットを以前の状態に戻したいというときには

その時の tree オブジェクトが必ず必要になるということで

どの tree オブジェクトなの？の情報も必要になってくる

それを収めるのが commit オブジェクトである

##### commit object

commit オブジェクトが収めている情報

- ルートディレクトリの tree オブジェクトの SHA-1
- committer と author のタイムスタンプ、名前、メアド
- 親コミット・ノードの SHA-1
- コミット・メッセージ

確認方法解らん

改善されているかどうかの確認：親コミットを commit オブジェクトに含めることで改ざんされていないことが保証される

commit オブジェクトの一部を変更（改ざん）すると、別の commit ハッシュに変わってしまうから

ブロックチェーンと同じ仕組みみたい

ということでここまでの簡単なまとめ：

- commit をすると作られるオブジェクトは４つ
  commit, tree, blob
- blob オブジェクトはファイルの情報を保存するオブジェクトである。ファイルを復元するのにこの情報が必要である。
- tree オブジェクトはその時点のスナップショットを保存するオブジェクトである。
  ルートディレクトリのファイル情報を保存した blob ファイルへの SHA-1、ルートディレクトリ以下のディレクトリを収めた tree オブジェクトへの SHA-1 を保存してあるので、その時点の状態を後から復元できる。
  tree オブジェクトはディレクトリごとに作成される。
- commit オブジェクトはスナップショットを取ったときの tree オブジェクトの SHA-1 とタイムスタンプ、コミットメッセージ、誰がコミットしたのかの情報
- commit オブジェクトが tree オブジェクトを参照しており、tree オブジェクトは blob オブジェクトを参照する

#### HEAD

コミットしたときに内部的に起こること仕上げ。

commit オブジェクトを作ったら、Git は最後に HEAD を書き換える。

HEAD とは？その前に refs を理解しないといかん。

refs は特定の commit を指すポインタのようなもの。

HEAD は現在の commit を指す refs の一つである。

checkout すると HEAD は書き変わっている。（別のコミットを指すことになるから）

.git/HEAD に保存されている

```bash
$ cat .git/HEAD
ref: refs/heads/master
```

commit すると、HEAD を書き換えるという話だけど

- HEAD が直接 commit ハッシュを参照している場合：HEAD の commit ハッシュを書き換える

  これって git checkout commit-hash しているときに commit した場合のことかしら？

HEAD が branch を参照している場合：HEAD が参照している branch の commit ハッシュを書き換える

    ほとんどこの通りなんじゃないの？

## ちょっとコマンドまとめ

確認用コマンドまとめ：

`git ls-files -s`:

staging-area（ステージングエリア）でインデックスされているファイルを確認したい場合に利用するコマンド

`-s`は blob の hash 値とファイル名の一覧を表示する

blob オブジェクトはステージングされたときに生成されるファイル情報を保存した git オブジェクトである。

`git cat-file -p master^{tree}`:

> cat-file コマンドを使うと、コンテンツを Git から取り出すことができます。 このコマンドは、Git オブジェクトを調べるための万能ナイフのようなものです。 -p オプションを付けると、cat-file コマンドはコンテンツのタイプを判別し、わかりやすく表示してくれます。

`master^{tree}`は、master ブランチの最新のコミットにおけるルートディレクトリの tree オブジェクトを指定している

#### git hooks

.git/hooks/にスクリプトを書いておくと、イベントが来たときに自動的にスクリプトが発火してくれるらしい

Git フックは、Git でコマンドを実行する直前もしくは実行後に特定のスクリプトを実行するための仕組みです。

たとえば、pre-commit という Git フックを使用すると、git commit を実行する直前に任意のスクリプトを実行することができます。これを利用することで、コミットメッセージにイシューやチケットの番号が含まれていなかったらコミットを中止する、などの処理を行うことができるようになります。

ということでミスを減らすための仕組みのようです

#### Packfile

https://git-scm.com/book/ja/v2/Git%E3%81%AE%E5%86%85%E5%81%B4-Packfile

Git は基本的に毎回 blob オブジェクトを生成するときは差分じゃなくてファイルすべてをバックアップする

なので容量の大きいファイルを毎回 blob させていたらさぁ大変

そんな時に差分だけを記録する仕組みがある

詳しくは上記のリンクを...

#### どこからも参照されていないオブジェクトを探すとき

git fsck を使うらしい
