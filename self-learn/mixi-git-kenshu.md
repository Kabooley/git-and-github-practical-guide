# Note ミクシィ Git 検収

https://mixi-developers.mixi.co.jp/22-technical-training-5fc362a9dc41

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

-   master(main)
-   developer
-   feature
-   release
-   hotfix

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

-   Security Policy
-   Security Advisories
-   Dependebot

#### Settings

最もよく使うのが`Branch protection rules`

指定した正規表現にマッチした brnach に対して PR を経ない直接 push や force push を禁止するよう設定できたりする

master や develop ブランチには何らかのルールが施されているはず。

## Git の内部構造

git add したときに何が起こっているか:

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

-   commit
-   tree
-   blod
-   tag
