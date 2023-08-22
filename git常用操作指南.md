## 常用操作命令
1. 获取当前仓库的所有分支：`git fetch`
2. 拉取当前分支最新代码：`git pull`
3. 查看当前分支工作区的更改 `git status`
4. 查看具体更改内容 `git diff`
5. 将当前更改的文件添加到暂存区 `git add <filename>` ；或者 `git add .` 将全部的更改一次性都添加到暂存区
6. 将暂存区的修改提交到分支并提交修改备注： `git commit  -m ' commit mesage'`
7. 将当前分支更改的内容推送到远程： `git push `
7. 如果当前分支还没有和远程分支建立联系，推送的时候会提示执行：`git push --set-upstream origin <分支名>`
## 撤销操作
8. 5步之前 `git checkout .` 撤销全部更改，或者 `git checkout --<filename>`撤销某个文件
9. 5步之后 `commit`之前 把提交到暂存区的修改回退到工作区，`git reset HEAD <file>` 
10. 如果已经`commit`了想撤回 走版本回退
## 版本回退
11. `git log` 可以查看近期的commit的版本记录
12. 回退到上一个版本 `git reset --hard HEAD^`
13. 如果需要回退到指定版本：可以通过`commit id`（11步可以拿到）`git reset --hard <commit id>`

## 删除文件
14. 直接在文件管理器删除或者用rm命令删除：`rm <filename>`
15.  `14` 之后 `git status`可以看到有文件被删除了，
16. 如果想撤回`14`的删除操作：git checkout --<filename>
17. 如果确定要删除并且要提交到版本库： `git rm <filename>`，`git commit -m 'remove file'`

## 添加远程库
假如你是先创建了一个本地库(本地目录test-git下运行`git init`)，又在GitHub创建了一个Git仓库，并且让这两个仓库进行远程同步。

1. 登录GitHub, 右上角’Create a new repo‘按钮点击创建一个新的空仓库假如名为testgit
2. 要讲本地库与远程库相关联，在本地的testgit仓库下运行：`git remote add origin git@github.com:<your github name>/test-git.git`。关联一个远程库时必须给远程库指定一个名字，`origin`是默认习惯命名；
3. 运行`git push -u origin master`将本地库的所有内容推送到远程；`origin`是远程库的名字，这是Git的默认叫法，由于远程库是空的，第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。
4. 推送成功后，可以在GitHub页面中看到远程库的内容已经和本地的一样了。

## 从远程库添加

这是比较推荐的方式，先创建远程库，然后从远程库克隆。

1. 登录GitHub，创建新的仓库test-git
2. 创建完成后克隆到本地 `git clone git@github.com:<your github name>/test-git.git`

## 删除远程库

1. `git remote -v`查看远程库信息：

```bash
$ git remote -v
origin  git@github.com:<your github name>/test-git.git (fetch)
origin  git@github.com:<your github name>/test-git.git (push)
```

2. ` git remote rm origin` 解除本地和远程的绑定关系
3. 要想真正删除远程库，登录GitHub，在页面中找到该仓库的删除操作删除

## 分支管理

1. `git branch`命令查看所有分支

2. 切换分支 `git checkout master` 或者 `git switch master `

3. 合并分支：运行 `git merge dev` 将dev分支合并到当前分支

4. `git branch -d dev`删除dev分支。如果该分支有提交未进行合并，则会删除失败。

5. 如果要丢弃一个没有被合并过的分支，通过`git branch -D <name>`强行删除。如果该分支有提交未进行合并，也会删除成功。

6. 创建并切换到新分支 `git checkout -b <name>`或者`git switch -c <name>`

7. 使用`git stash`可以把当前工作现场临时“储藏”起来去修复bug

8. 查看保存过的stash，运行`git stash list`

9. 恢复工作现场：`git stash apply`恢复 `git stash drop`删除。或者`git stash pop`恢复的同时把stash内容也删了。你可以多次stash，恢复的时候，先用`git stash list`查看，然后恢复指定的stash，用命令：`git stash apply stash@{0}`

   合并代码前要`git pull`拉取到最新代码，合并时如果有冲突无法快速合并要先解决冲突再提交。解决冲突就是把合并失败的文件手动编辑为你希望的内容再提交。

10. `cherry-pick`命令可以用来复制一个特定的提交（比如临时修复的bug）到当前分支：`git cherry-pick 4c805e2` **4c805e2**是这个特定提交的commit id.

## 标签操作

1. 查看当前所有tag：`git tag`
2. 打标签 `git tag v1.0`
3. `git tag -a v0.1 -m "version 0.1 released" 1094adb`  . `-a`指定标签名，`-m`指定说明文字
4. 默认标签是打在最新提交的commit上的。有时候，如果忘了打标签，可以找到历史提交的commit id，执行` git tag v0.9 f52c633`为指定的提交打标签
5. 查看当前标签信息 `git show <tagname>` 
6. 删除没有推送到远程的本地标签：`git tag -d v0.1`
7. 如果标签已经推送到远程，要删除远程标签需要先从本地删除：`git tag -d v0.1`；然后，从远程删除：`git push origin :refs/tags/v0.1`
8. 推送标签到远程 `git push origin <tagname>`；或者，`git push origin --tags`，一次性推送全部尚未推送到远程的本地标签

## 为常用操作配置别名

1. `git config --global alias.st status`     用 `st` 表示 `status`.同理可以配置下面一些操作
2. `git config --global alias.co checkout`
3. `git config --global alias.ci commit`
4. `git config --global alias.br branch`
5. `git config --global alias.unstage 'reset HEAD'`. 把暂存区的修改撤销掉（unstage），重新放回工作区
6. `git config --global alias.last 'log -1'` 显示最近一次提交

这些配置加上`--global`是针对当前用户起作用，如果不加，那只针对当前的仓库起作用。每个仓库的Git配置文件都放在`.git/config`文件中；`cat .git/config `查看配置，修改配置内容。而当前用户的Git配置文件放在用户主目录下的一个隐藏文件`.gitconfig`中；`cat .gitconfig`查看配置并修改。

## git 的工作区和暂存区
你本地被git所管理的文件夹就是一个git工作区
工作区有一个隐藏目录`.git`，它是git的版本库
git的版本库里存了很多东西，其中最重要的就是称为stage（或者叫index）的暂存区，还有git自动创建的第一个分支`master`以及指向`master`的一个指针`HEAD`.

## SSH key的配置

1. 生成SSH key：`ssh-keygen -t rsa -C "your mail address"`
2. 登录GitHub 添加SSH Key（公钥）
3. 进入.ssh目录：`cd ~/.ssh`
4. 查看文件目录： `ls`
5. 在终端查看私钥：`cat id_rsa`
6. 在终端查看公钥：`cat id_rsa.pub`

## git merge 和 git merge --no-ff的区别

合并代码时如果不带`--no-ff`参数，Git会用`Fast forward`模式合并。这种模式下，删除分支后，会丢掉分支信息。

如果用`--no-ff`参数强制禁用`Fast forward`模式，Git就会在merge时生成一个新的commit，合并后的历史有分支，能看出来曾经做过合并，而`fast forward`合并就看不出来曾经做过合并。