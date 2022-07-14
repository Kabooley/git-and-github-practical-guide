# Git Undoing chnages

まとめノート。

講義ノートと区別。

## 参考

https://www.atlassian.com/ja/git/tutorials/undoing-changes

https://www.atlassian.com/ja/git/tutorials/undoing-changes/git-reset



## 3つのツリー(領域)

## git checkout を使用してコミットを元に戻す方法

git checkoutをブランチに対して使うのではなくてコミット（コミットid）に使う場合

`git checkout <過去のコミットid>`を実行すると

HEADはブランチポインタが指すコミットから外れて過去のコミットを指す。

そんなブランチから離れたHEADはdetached HEADと呼ばれる。

detached HEADはブランチ上で作業しておらず、このコミットはどう扱われるのか

検証：

```bash
$ git log --oneline
# この最新のコミットをなかったことにしたい
815499b (HEAD -> master) Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit
```

```bash
$ git checkout 3bc319b
Note: switching to '3bc319b'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at 3bc319b Make some important changes to initial-commit.txt
```

> あなたは「切り離された頭」の状態です。この状態では、いろいろなものを見たり、実験的な変更を加えたり、コミットしたりすることができます。
> また、この状態で行ったコミットを破棄することもできます。
> ブランチに戻ることで、どのブランチにも影響を与えずにこの状態でのコミットを破棄することができます。

> 新しいブランチを作成して自分のコミットを保持したい場合は、次のようにします。
> switch コマンドで -c を指定することで、(今からでも) ブランチを作成することができます。例

>  git switch -c <新しいブランチ名> とします。

> あるいは、この操作を元に戻すには

>  git switch - を実行すると、この操作を取り消すことができます。

つまり、

- detached HEAD状態ではコミットができる
- ただしブランチへ戻ることでそのコミットが破棄できる

これらのことはAtlassianにも記述がある。

つまり、

**detached HEAD状態でのコミットはどこにも反映されず孤立する**

```bash
$ cat initial-commit.txt
Write some text at first time
Make some important changes to initial-commit.txt
# Try something crazyで行った変更がないので指定したコミットへ戻ったことが確認できる

# detached HEADでコミットした
# ブランチの指すコミットがない。これはブランチ上で作業していないから。
$ git log --oneline
e9ee83c (HEAD) Commited on detahced HEAD
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# これでブランチに戻ってみる
# 
# すると、
# 「どのブランチにも参照されていない1つのコミットを残しています」という警告が出る
$ git checkout -
Warning: you are leaving 1 commit behind, not connected to
any of your branches:

  e9ee83c Commited on detahced HEAD

If you want to keep it by creating a new branch, this may be a good time
to do so with:

 git branch <new-branch-name> e9ee83c

Switched to branch 'master'

# detached HEADでのコミットが含まれていないことがわかる
# なので過去のコミットへchekcoutで戻って変更してもブランチに戻ればなかったことになる
# というのは確認できた
$ git log --oneline
815499b (HEAD -> master) Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# 先の孤立したコミットへ移動はできる
$ git checkout e9ee83c
$ git switch -c new-branch-from-detached-head
Switched to a new branch 'new-branch-from-detached-head'
# それまでのコミットは参照できる
$ git log --oneline
e9ee83c (HEAD -> new-branch-from-detached-head) Commited on detahced HEAD
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# 戻ってみると...
# detached HEADへ戻り、masterではない
git checkout -
Note: switching to 'e9ee83cb92bdefb984657d53ec6146dcc9934af0'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at e9ee83c Commited on detahced HEAD

# masterにも戻れる
$ git checkout master
Switched to branch 'master'

# detached HEADから確立したブランチをmasterへmergeしようとすると、conflictが起こる
$ git merge new-branch-from-detached-head
Auto-merging initial-commit.txt
CONFLICT (content): Merge conflict in initial-commit.txt
Automatic merge failed; fix conflicts and then commit the result.

# コンフリクトを解消する前だと、解消前の状態のまま
$ git log --oneline
815499b (HEAD -> master) Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# ためしにマージしたブランチの内容を受け付けてみる
# 
# conflictが起こる
$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   initial-commit.txt

no changes added to commit (use "git add" and/or "git commit -a")

# 結果
$ git log --oneline
909a60e (HEAD -> master) Merged branch that from detached HEAD
e9ee83c (new-branch-from-detached-head) Commited on detahced HEAD
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit
```

結果：

- git checkout <過去コミット>では過去のコミットを作業ツリーにロードできる
- detached HEADで行ったコミットはもとのブランチから孤立する
    なのでブランチに戻ればdetached HEADで行ったあらゆる変更（コミット含む）はなかったことになる
    なので過去のコミットをいろいろ確認するにはgit checkoutは便利。

- detached HEADで新たにブランチを確立してもそのブランチは確立前のブランチに戻れない
    conflictなしにmergeできないという意味で！
    detached HEADで確立したブランチをmasterに無理やりマージすることになるのでコンフリクトが起こる
    つまりそんなことはするな。

この結果からわかるgit checkoutの使い道:

- 過去のコミットを作業ツリーにロードしていろいろ確認するのに使う


「元に戻す」ためにgit checkoutは使うべきでない

## git revertを使用してパブリック・コミットを元に戻す

結論：revertは直前のコミット一つだけを打ち消すことができるコマンドである

```bash
# 今、またTry something crazyという取り消したいコミットがあるとする
$ git log --oneline
815499b (HEAD -> master) Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# HEADを「打ち消し」た
# 
$ git revert HEAD
hint: Waiting for your editor to close the file... 
[master 71e8b44] Revert "Try something crazy"
 1 file changed, 1 deletion(-)

# このコミットは"Try something crazxy"をコミット履歴から消すのではなく
# のこしたまま、
# revertによるコミットがそのコミットを打ち消すような変更でコミットされるのである
$ git log --oneline
71e8b44 (HEAD -> master) Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# 次でそれが確認できる
# 
# "Try something crazxy"の書き込みがなくなっている
$ cat initial-commit.txt
Write some text at first time
Make some important changes to initial-commit.txt
```

- 同じブランチを使い続けることができる
- 重要なコミットを消す必要がなくなる
- 取り消したいコミットもコミット履歴に残すことができる

検証：

- 2つ以上前のコミットをrevertすると1つ前のコミットはどうなるのか？
- revertしたコミットをrevertしたらどうなるのか？

```bash
$ git log --oneline
71e8b44 (HEAD -> master) Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# さらにrevert
$ git revert  HEAD
hint: Waiting for your editor to close the file...
[master 00203e3] Revert "Revert "Try something crazy""
 1 file changed, 1 insertion(+)

# revertしたコミットをrevertすると元に戻るのが確認できる
$ cat initial-commit.txt
Write some text at first time
Make some important changes to initial-commit.txt
Add something crazy

```

```bash
$ git log --oneline
47e40b8 (HEAD -> master) Try another something more crazy
00203e3 Revert "Revert "Try something crazy""
71e8b44 Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# 2つ前のコミットをrevertしてみる
# 
# revertはできなくてconflictが起こる
$ git revert 815499b
Auto-merging initial-commit.txt
CONFLICT (content): Merge conflict in initial-commit.txt
error: could not revert 815499b... Try something crazy
hint: after resolving the conflicts, mark the corrected paths
hint: with 'git add <paths>' or 'git rm <paths>'
hint: and commit the result with 'git commit'
```

結果：

- git revertは１つのコミットだけ打ち消すことができる

> git revert は、1 つのコミットのみを元に戻すコマンドであることをしっかりと理解してください。

つまり結局直前のコミットだけになるね

先の2つまえのコミットを指定してrevertしたらrevertはできなくてコンフリクトが起こった

#### git reset

軽く復習:

git checkoutはHEAD refポインタだけを指定のコミットへ移動するので、その操作は必ずdetacged HAEDになる

detached HEAD上での操作は、元のブランチに戻れば無かったことになる
detached HEADでコミットすると、そのコミットは孤立する
detached HEADで新たなブランチを切っても、もとのブランチにmergeできない（conflictが起こる）

git revertは直前のコミット内容を打ち消すように新たなコミットを生成する。

git revertは2つ前のコミットに対してrevertしようとすると、revertはできなくてコンフリクトが起こる

git resetはHEADrefもブランチrefも指定のブランチに移動する

たとえば2つ前のコミットを指定すると、1つ前以降のコミットは孤立す。

```bash
# 一つのファイルを変更して、一つのファイルを新規追加してgit addした
$ git add new-file.txt 
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   new-file.txt

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   initial-commit.txt
# インデックスの状態を確認
$ git ls-files -s
100644 36db3e74125eff086804e547bcb87afb096e50e2 0       initial-commit.txt
100644 2e35eb989a37ee7f7df45aa9f472b71e9b492b16 0       new-file.txt
# 
# reset --mixed HEADしてみる
#   
$ git reset --mixed HEAD
Unstaged changes after reset:
M       initial-commit.txt

$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   initial-commit.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        new-file.txt

no changes added to commit (use "git add" and/or "git commit -a")

$ git ls-files -s
100644 36db3e74125eff086804e547bcb87afb096e50e2 0       initial-commit.txt
```

つまり、

- indexされたことはなかったことになる

  `git ls-files -s`で新しく追加したファイルのことがなくなっているし、
  `git status`でuntrackedファイル扱いになっている
  更新したファイル(initial-commit.txt)もステージングされていない扱いになっている

- 作業ディレクトリへの変更はそのままだけど、
- ステージングはキャンセルされて、
- コミットは指定のコミットの状態にリセットされる。


```bash
# ファイルの更新と新規ファイルの追加をしてgit addした
$ git ls-files -s
100644 ca6137f59ff6211ec6473d9efc725e5570bc702a 0       initial-commit.txt
100644 2e35eb989a37ee7f7df45aa9f472b71e9b492b16 0       new-file.txt

$ git reset --mixed HEAD
Unstaged changes after reset:
M       initial-commit.txt
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   initial-commit.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        new-file.txt

no changes added to commit (use "git add" and/or "git commit -a")
# 更新した方のファイルはインデックスされる前のSHA-1になっている
# つまり以前のコミット時のSHA-1値である
$ git ls-files -s
100644 36db3e74125eff086804e547bcb87afb096e50e2 0       initial-commit.txt
```