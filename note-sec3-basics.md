# Note Section 3

Version Management with Git - The Basics

#### 28: Understanding Branch

マスターブランチ：これまでのすべての履歴が含まれているブランチ

通常マスターブランチで作業しない。

開発は通常チームで行われ、自分の担当すべき機能とメンバーが担当する機能がある

自分の担当機能開発のためにマスターブランチは使わずに

完全なコピーでマスターブランチから独立した新たなブランチを作成して

そのブランチ上で開発する

機能を開発し終えたらブランチの内容をマスターブランチへ上書きすることができる

これがマージ

#### 29~32: skipped

#### 33: git init & git commit

はじめて git を使うときはクレデンシャルを登録します

git config --global で user.email, user.name などを登録する

#### 34~: git log とブランチの理解と作成

```bash
# git commit 履歴が確認できる
# q で復帰
git log

# 現在のブランチ一覧が表示される
git branch
# ブランチの作成
git branch <BRANCH-NAME>
# ブランチの移動
git checkout <BRANCH-NAME>
# この時点でgit log　すると、マスターブランチと全く同じ履歴が確認できる
git log

# git branchをたたかなくても、git checkoutでブランチ作成＆移動可能
git checkout -b <ANOTHER-BRANCH-NAME>

# マージする
# mergeはブランチを別のブランチと統合することである
# たとえば
git checkout -b development
# (development)ブランチに移動して作業をしたとして
# 作業内容をdevelopmentへコミットして
(development) git add .
(development) git commit .
# ブランチをmasterへ切り替えて
(development) git checkout master
# developmentブランチをmasterへ統合する
(master) git merge development
# ログを確認すると
(master) git log
# 最新のコミットのところに
(HEAD -> master, de)
```

##### 38: HEAD を理解する

今 master ブランチには 3 つのコミットの履歴があるとする

```bash
# 今３つのブランチがあるとする
(master) git branch
* master
    development
    feature1

(master) git log
# こんな感じのコミット内容があって、HEADはmasterの最新ブランチを指している
commit bcf048778b15a7962cc9faf53eed6af29cf8be30 (HEAD -> master)
Author:
Date:   Wed Jun 29 21:43:58 2022 +0900

    Rewrite initial-commit.txt

commit 1c2a5c1ba2a54a21a3076eeddf408bdbc58f181a
Author:
Date:   Wed Jun 29 00:52:37 2022 +0900

    first commit

# ブランチを切り替えて
(master) git checkout development
# いろいろ変更してからコミットした...
# するとlogは
(development) git log
commit b6acf47ca07fd097613375f3c18348bc14c8ee4f (HEAD -> development)
Author:
Date:   Wed Jun 29 21:46:22 2022 +0900

    development generated

commit bcf048778b15a7962cc9faf53eed6af29cf8be30 (master)
Author:
Date:   Wed Jun 29 21:43:58 2022 +0900

    Rewrite initial-commit.txt

commit 1c2a5c1ba2a54a21a3076eeddf408bdbc58f181a
Author:
Date:   Wed Jun 29 00:52:37 2022 +0900

    first commit
# developmentの最新コミットを指していることがわかる
# ここでmasterに戻ると...
(development) git checkout -
(master) git log
# 変更は全く反映されていないので、git logの結果はdevelopmentに移る前のまま
commit bcf048778b15a7962cc9faf53eed6af29cf8be30 (HEAD -> master)
Author:
Date:   Wed Jun 29 21:43:58 2022 +0900

    Rewrite initial-commit.txt

commit 1c2a5c1ba2a54a21a3076eeddf408bdbc58f181a
Author:
Date:   Wed Jun 29 00:52:37 2022 +0900

    first commit
(master) git merge development
(master) git log
# HEADがmaster, development両ブランチにおいて同じ最新のコミットを指している
commit b6acf47ca07fd097613375f3c18348bc14c8ee4f (HEAD -> master, development)
Author:
Date:   Wed Jun 29 21:46:22 2022 +0900

    development generated

commit bcf048778b15a7962cc9faf53eed6af29cf8be30
Author:
Date:   Wed Jun 29 21:43:58 2022 +0900

    Rewrite initial-commit.txt

commit 1c2a5c1ba2a54a21a3076eeddf408bdbc58f181a
Author:
Date:   Wed Jun 29 00:52:37 2022 +0900

    first commit

```

ここからわかること: HEAD は各ブランチの最新ブランチを参照する...のかも

参考：

https://qiita.com/ymzkjpx/items/00ff664da60c37458aaa

- ブランチとはコミットを指すポインタである

これの意味するところは、ブランチで分岐させたこれまでのそのブランチ上でのコミットすべてがブランチではなくて、ブランチは分岐した枝上の最新のコミットひとつだけを指しているに過ぎないということである

新しいブランチを生成することは、新しいポインタを作られるだけである

- コミットとは

#### 44: Undoing staged changing

ステージングした段階で、ステージングした内容をキャンセルする方法

```bash
git checkout <staged-filename>

git reset <staged-filename>

# initial-commit.txtを変更した

git add .
git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   initial-commit.txt

git checkout initial-commit.txt
Updated 0 paths from the index

git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   initial-commit.txt

# そのままコミットできちゃう
git commit
cat initial-commit.txt
# 内容ばっちり上書きされているからキャンセルされていないんだけど...
```

## git checkout

file:///C:/Program%20Files/Git/mingw64/share/doc/git-doc/git-checkout.html

branch を指定しなかった場合...

- `git checkout [<tree-ish>] [--] <pathspec>`

> インデックスまたは <tree-ish> (ほとんどの場合コミット) の内容で置き換えることにより、作業ツリーのパスを上書きします。<tree-ish> が指定された場合、<pathspec> にマッチするパスはインデックスと作業木の両方で更新されます。

> インデックスには、以前のマージの失敗により、未マージの項目が含まれている可能性があります。デフォルトでは、このようなエントリをインデックスからチェックアウトしようとすると、チェックアウト操作に失敗し、何もチェックアウトされません。f を使用すると、これらの未マージエントリを無視することができます。マージの特定の側からのコンテンツは、--ours または --theirs を使用してインデックスからチェックアウトすることができます。m を使用すると、作業ツリーファイルへの変更を破棄して、元の競合するマージ結果を再作成することができます。

- `<tree-ish>`

(path が渡されたなら)チェックアウトするツリーのこと。
特定できない path だったら、index が使用される

## Atlasian Tutorial: 元に戻す

https://www.atlassian.com/ja/git/tutorials/undoing-changes

Git に「元に戻す」という言葉は存在しない。

reset, 打消し、checkout, clean up

#### 過去のコミットに戻って操作する

たとえば過去のコミットに checkout で戻ったら、作業ディレクトリ上の操作は
現在のリポジトリに一切保存されない。

なので過去の何かを調べてからもとのコミットに戻れば

過去のコミットで行った操作による影響はない。

```bash
git log --oneline

b7119f2 Continue doing crazy things
872fa7e Try something crazy
a1e8fb5 Make some important changes to hello.txt
435b61d Create hello.txt
9773e52 Initial import

git checkout a1e8fb5

# 過去のコミットで変更を施した
(detached HEAD) echo "Modified on detached HEAD" >> initial-commit.txt

# 変更した場合、コミットしないとcheckoutできない...
(detached HEAD) git add .
(detached HEAD) git commit

git checkout main

# mainブランチの現在のコミットには影響しない
cat initial-commit.txt

some changes
development branch generated
About to git reset
# 変更は反映されていない
```

## Atlassian Tutorial: リセット、チェックアウト、元に戻す

git reset を理解するには 3 つのツリーの概念を理解しよう。

https://www.atlassian.com/ja/git/tutorials/resetting-checking-out-and-reverting

3 つの領域(ツリー)：

https://www.atlassian.com/ja/git/tutorials/undoing-changes/git-reset

- 作業ディレクトリ working directory
- ステージングインデックス Staged Snapshot
- コミット履歴 Commit History

作業ディレクトリ：

> このツリーはローカルファイルシステムと同期しており、ファイルとディレクトリへの変更はこのツリーに即座に反映されます。

つまり今作業しているフォルダ内での変更を監視しているツリーである

ステージングインデックス：

> `git add`でプロモートされて後続のコミットに保存される作業ディレクトリの変更を追跡します。

`git ls-files -s`でステージングインデックスツリーの状態を検査するコマンド。

コミットする前ならば add するたびにその add に対してハッシュ値が与えられる。

同じファイルであっても別の add ならば異なるハッシュ値が与えられるので

ステージングインデックスの中では区別されている

コミット・ヒストリー：

> コミット履歴内にある永続的なスナップショットに変更を追加します。このスナップショットには、コミット時のステージング インデックスの状態も含まれます。

## 47: Commit changing by detached HEAD

detached HEAD おさらい：

- detached HEAD はブランチと同じコミットを指していないときの HEAD ref である
- detached HEAD でのあらゆる変更はもとのコミットに戻ると、どのブランチにも影響を与えずになかったことになる
- detached HEAD でのコミットは孤立し、もとのブランチに戻るとそのコミットは破棄される
- detached HEAD でのコミットは、どのブランチにも参照されない

検証:

では以前のコミットをチェックアウトして作業ツリーにロードし、

それに変更を加えてからコミットして、マスターブランチに追加したらどうなるか

```bash
# dummy.txtを追加してコミットした
$ git log --oneline
bafb391 (HEAD -> master) Added dummy file to make sure how detached HEAD works
44d5191 Made sure how reset-mixed works
3b6f08b Make sure what tree object is 2
029342a make sure tree object
00203e3 Revert "Revert "Try something crazy""
71e8b44 Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# 一つ前のコミットをチェックアウトする
#
# detached HEADになった
$ git checkout 44d5191
Note: switching to '44d5191'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at 44d5191 Made sure how reset-mixed works

# 一つ前のコミットなのでdummy.txtはない
$ ls
hoge.txt  huga  initial-commit.txt  README.md  reset.txt

# 一つ前のコミットなのでブランチrefのコミットはない
$ git log --oneline
44d5191 (HEAD) Made sure how reset-mixed works
3b6f08b Make sure what tree object is 2
029342a make sure tree object
00203e3 Revert "Revert "Try something crazy""
71e8b44 Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# detached HEAD状態でファイルに変更を加えて
$ echo "Detached HEAD to fix forgotten things" >> initial-commit.txt
$ cat initial-commit.txt
Write some text at first time
Make some important changes to initial-commit.txt
Add something crazy
make sure what tree object is
make sure how reset works
make sure 2 commit reset
Detached HEAD to fix forgotten things

$ touch detached.txt

$ git status
HEAD detached at 44d5191
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   initial-commit.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        detached.txt

no changes added to commit (use "git add" and/or "git commit -a")

# detached HEADでの変更をコミットする
$ git add .
$ git status
HEAD detached at 44d5191
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   detached.txt
        modified:   initial-commit.txt
$ git commit -m "Changes made in detached HEAD"
[detached HEAD 1e16572] Changes made in detached HEAD
 2 files changed, 1 insertion(+)
 create mode 100644 detached.txt

# 新しいコミットが生成された
$ git log --oneline
1e16572 (HEAD) Changes made in detached HEAD
44d5191 Made sure how reset-mixed works
3b6f08b Make sure what tree object is 2
029342a make sure tree object
00203e3 Revert "Revert "Try something crazy""
71e8b44 Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# masterに戻ると...
#
# どこからも参照されていないコミットを置いてきているよとメッセージ
#
# (detached HEADから派生した)新しいブランチを作成することを提案されている
$ git switch master
Warning: you are leaving 1 commit behind, not connected to
any of your branches:

  1e16572 Changes made in detached HEAD

If you want to keep it by creating a new branch, this may be a good time
to do so with:

 git branch <new-branch-name> 1e16572

Switched to branch 'master'

# その通りにブランチを作成した
$ git branch detached-branch 1e16572


# 新しく作成したdetached HEAD派生のブランチ
$ git switch detached-branch
Switched to branch 'detached-branch'
$ ls
detached.txt  hoge.txt  huga  initial-commit.txt  README.md  reset.txt
$ git switch master
Switched to branch 'master'

# mergeしたら...
$ git merge detached-branch
Merge made by the 'recursive' strategy.
 detached.txt       | 0
 initial-commit.txt | 1 +
 2 files changed, 1 insertion(+)
 create mode 100644 detached.txt


# detached HEADする前のコミットが残っているのが確認できる。
$ git log --oneline
ea096b5 (HEAD -> master) Merge branch 'detached-branch' into master
1e16572 (detached-branch) Changes made in detached HEAD
bafb391 Added dummy file to make sure how detached HEAD works
44d5191 Made sure how reset-mixed works
3b6f08b Make sure what tree object is 2
029342a make sure tree object
00203e3 Revert "Revert "Try something crazy""
71e8b44 Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# detached HEADする前のコミットで追加されたファイルdummy.txtが残っているのがわかる
$ ls
detached.txt  dummy.txt  hoge.txt  huga  initial-commit.txt  README.md  reset.txt

```

以前は detached HEAD で git switch -c <new-branch-name>で派生させたブランチを merge させたら

コンフリクトが起こった。

今回と前回の違いは何だろう。

検証する必要はある。

## .gitignore

Gitが無視するデータ、ファイル、ディレクトリをこのファイルに記述すると

Gitは以降それらを追跡しない。

こんな使い方

```bash
# 特定のファイルだけ追跡する
# 
# text.txtだけ追跡する
!text.txt
```

## Section Wrap up

コマンド気になったのまとめ

```bash
# WD File*
git rm <filename>
git add <filename>

# undoing unstaged changes
git checkout (--) .
git restore <filename> or .

# undoing staged changes
git reset filename
# and
git checkout -- filename
# or
git restore --staged filename or .


```

#### 課題１

