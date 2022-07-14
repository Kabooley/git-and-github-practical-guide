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

#### git reset --mixed HEAD

先にまとめ：
- HEAD, indexが書き換えられて、作業ツリーがそのままになる

    つまりファイルの編集やファイルの追加などはそのままだけど、
    .git/indexに登録されたファイルがなくなり、
    HEADとブランチは一つ前のコミットを指す

- ステージングがなかったことになるのでblobファイルはすべて一つ前のコミットの時のものになる

- なのでaddをしたことだけを取り消して、編集内容などはそのまま残したいときは--mixed HEADを使うといい

軽く復習 ---

git checkoutはHEAD refポインタだけを指定のコミットへ移動するので、その操作は必ずdetacged HAEDになる
detached HEAD上での操作は、元のブランチに戻れば無かったことになる
detached HEADでコミットすると、そのコミットは孤立する
detached HEADで新たなブランチを切っても、もとのブランチにmergeできない（conflictが起こる）
git revertは直前のコミット内容を打ち消すように新たなコミットを生成する。
git revertは2つ前のコミットに対してrevertしようとすると、revertはできなくてコンフリクトが起こる

---

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

- 作業ディレクトリへの変更はそのままだけど、ステージングはキャンセルされて、コミットは指定のコミットの状態にリセットされる。


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

2つ以上まえのコミットを指定するとどうなるか

```bash
# 一度コミットをして、
$ git log --oneline
2b3cef7 (HEAD -> master) Make sure reset--mixed if specified 2 commit ago
3b6f08b Make sure what tree object is 2
029342a make sure tree object
00203e3 Revert "Revert "Try something crazy""
71e8b44 Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit
# そのコミットの一つ前のコミットでgit reset --mixedを行った
$ git reset --mixed 3b6f08b
Unstaged changes after reset:
M       initial-commit.txt
# HEADもブランチも指定のコミットへ戻った
$ git log --oneline
3b6f08b (HEAD -> master) Make sure what tree object is 2
029342a make sure tree object
00203e3 Revert "Revert "Try something crazy""
71e8b44 Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit
# 検証１：2b3cef7のコミット内容は作業ツリーに残っているのか？
# 
# 結果：各ファイルの最後の行がコミットしたときに追加した文章で、これが残っている
$ cat reset.txt
make sure how reset works
make sure 2 commit reset

$ cat initial-commit.txt
Write some text at first time
Make some important changes to initial-commit.txt
Add something crazy
make sure what tree object is
make sure how reset works
make sure 2 commit reset
# なのでこのままadd, commitすればresetする前のコミットの通りになる
```

2つ以上前のコミットを指定して`git reset --mixed`したら、その間のコミット内容はすべて作業ツリーへ戻されるので、

そのままコミットすればresetするまえのコミットの通りになる

#### git reset --hard HEAD

結論：

- 作業ツリーは書き変わる、indexは書き変わる、HEAD、ブランチは指定のコミットを指す

つまり、--hardにするとステージングだけじゃなくて、編集したファイルや追加、削除したファイルもなかったことになるので

完全に指定したコミットの直後の状態に戻される

検証１：

既存ファイル更新: initial-commit.txt、
既存ファイル１つ削除: reset.txt、
新規ファイル追加：reset-hard.txt
git add .し、reset --hard HEADした

```bash
# addしたあと
$ git ls-files -s
100644 4cf5010e7b78cfd61d33983044299260d21b9189 0       README.md
100644 1904c092b649dc54f3c8fc931acb0ca5bb952c3b 0       hoge.txt
100644 1618b9c3afe455248859eea088e09b571acab392 0       huga/huga.txt
100644 a2fbecb07e63156bcf56ece45564f6d1c55a65e2 0       initial-commit.txt
100644 920678635e2bf1e679087a9b14bf342fd02cd15b 0       reset-hard.txt

$ git reset --hard HEAD
HEAD is now at 44d5191 Made sure how reset-mixed works

# 作業ツリーもクリーンである
# 新規に追加したファイルがどこにも存在しない
$ git status
On branch master
nothing to commit, working tree clean


$ git ls-files -s
100644 4cf5010e7b78cfd61d33983044299260d21b9189 0       README.md
100644 1904c092b649dc54f3c8fc931acb0ca5bb952c3b 0       hoge.txt
100644 1618b9c3afe455248859eea088e09b571acab392 0       huga/huga.txt
# 元に戻っているのがわかる
100644 da4528a61a2f56cf348ec8567cc03a56434c110d 0       initial-commit.txt
# 削除したファイルも戻っている
100644 13a430dcab6be55ccae190467062dd451a057338 0       reset.txt
```

間違いなく指定のコミットの直後の状態に戻っている

検証２：

2つ以上前のコミットを指定してみる

--mixedではすべて作業ツリーに戻されたが、--hardではどうなるか

って書くまでもないけど（確認はしたけど）、

指定したコミットの直後の状態に戻るだけだから作業ツリーもインデックスも空である。

--mixedはリセット前の状態を作業ツリーに残しておけるけど
--hardは完全に指定のコミットの直後のクリーンな状態に作業ツリーもインデックスも戻される

#### git reset --soft

ちょっと割愛
