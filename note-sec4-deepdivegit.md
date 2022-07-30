# Note section 4

このセクションで取り扱う話

- commitに関して深堀する
- 異なるブランチの統合と管理について
- コンフリクトの解決について

## git stashで一旦隠しておく

```bash
$ cd ..

$ echo "# Note section 4" > note-sec4-deepdivegit.md        

$ mkdir git-deep-dive/

$ cd git-deep-dive


$ touch file1.txt

    
$ git init
Initialized empty Git repository in C:/Users/yashi/Udemy/git-and-github-practical-guide/git-deep-dive/.git/


$ git add .


$ git commit -m "file1 added"
[master (root-commit) 13a5fb0] file1 added
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 file1.txt


$ git log --oneline
13a5fb0 (HEAD -> master) file1 added


$ git stach
git: 'stach' is not a git command. See 'git --help'.

The most similar command is
        stash


$ git stash
No local changes to save


$ echo "changes before stash" >> file1.txt


$ git stash
Saved working directory and index state WIP on master: 13a5fb0 file1 added


$ git status
On branch master
nothing to commit, working tree clean


$ git ls-files -s
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0       file1.txt


$ git stash apply
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   file1.txt

no changes added to commit (use "git add" and/or "git commit -a")


$ git ls-files -s
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0       file1.txt

```
