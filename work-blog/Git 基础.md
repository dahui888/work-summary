### Git 基础

### 1. Git 中的基本概念

##### 1.1 仓库
仓库又称为版本库，英语名称 repository。说得通俗一点就是需要用 Git 进行管理的文件夹。这个仓库中存放了我们的代码、文件，我们可以通过 Git 对这些代码进行管理。

##### 1.2 工作区和暂存区
- 工作区：我们在工作区中进行代码的编辑，工作区就是我们正在操作的文件夹。

- 暂存区：工作区有一个隐藏目录 .git，这个不算工作区，而是 Git 的版本库。 Git的版本库里存了很多东西，其中最重要的就是称为 stage（或者叫 index）的暂存区，还有 Git 为我们自动创建的第一个分支 master，以及指向 master 的一个指针叫 HEAD。

##### 1.3 远程仓库
我们知道 Git 是分布式版本管理系统，那么同一个 Git 仓库可以存在任何机器上，但是最初肯定只有一个机器存在这个仓库，然后，别的机器克隆这个仓库，所以就需要存在一个机器扮演服务器的角色，不过现在已经存在了Github 的一个网站用于托管 Git 仓库，我们只要从这个远程仓库进行 clone 即可。

##### 1.4 分支
分支就是岔路，比如几个人一起旅行，走到一个岔路口，每个岔路口都有不同的风景，我们约定几天后在某个地点集合。在我们的开发过程中，我们都在一条版本库线上开发，突然有天老大安排了一个开发任务给你，这个时候，你就可以在主线上拉出一条分支，单独进行你自己的开发，这个分支只有你自己知道，不影响主线的开发者使用。最后等待你的开发任务完成，进行合并到主线即可。

##### 1.5 标签
发布一个版本时，我们通常先在版本库中打一个标签（tag），这样，就唯一确定了打标签时刻的版本。将来无论什么时候，取某个标签的版本，就是把那个打标签的时刻的历史版本取出来。所以，标签也是版本库的一个快照。

### 2. Git 常见指令操作

#### 2.1 仓库相关指令

##### 1 git init

用于初始化一个 git 仓库。此时会生成一个 `.git` 文件夹，如果没有看到可以通过 ls -ah 查看。

##### 2. git status

查看当前仓库的状态，通过git status指令可以随时让我们掌握仓库的状态。

##### 3. git log

查看我们提交的日志。里面包含提交提交的作者、日期和 commit id。

```bash
# 显示当前分支的版本历史
$ git log

# 显示commit历史，以及每次commit发生变更的文件
$ git log --stat

# 搜索提交历史，根据关键词
$ git log -S [keyword]

# 显示某个commit之后的所有变动，每个commit占据一行
$ git log [tag] HEAD --pretty=format:%s

# 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
$ git log [tag] HEAD --grep feature

# 显示当前分支的最近几次提交
$ git reflog
```

##### 4. git clone

用于 clone 复制远程连接的资源库至本地。要克隆一个仓库，首先必须知道仓库的地址，然后使用 git clone 命令克隆。Git 支持多种协议，包括 Https，但通过 ssh 支持的原生 git 协议速度最快。

#### 2.2 分支相关指令

```bash
# 列出所有本地分支
$ git branch

# 列出所有远程分支
$ git branch -r

# 列出所有本地分支和远程分支
$ git branch -a

# 新建一个分支，但依然停留在当前分支
$ git branch [branch-name]

# 新建一个分支，并切换到该分支
$ git checkout -b [branch]

# 新建一个分支，指向指定 commit
$ git branch [branch] [commit]

# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch] [remote-branch]

# 切换到指定分支，并更新工作区
$ git checkout [branch-name]

# 切换到上一个分支
$ git checkout -

# 建立追踪关系，在现有分支与指定的远程分支之间
$ git branch --set-upstream [branch] [remote-branch]
==git branch --set-upstream-to=origin/remote_branch your_branch

# 合并指定分支到当前分支
$ git merge [branch]

# 选择一个 commit，合并进当前分支
$ git cherry-pick [commit]

# 删除分支
$ git branch -d [branch-name]

# 删除远程分支
$ git push origin --delete [branch-name]
$ git branch -dr [remote/branch]

# 查看远程无效分支： 
$git remote prune origin --dry-run

#清理远程无效分支：
$git remote prune origin

#更新远程分支列表：
$git remote update origin --prune
```

#### 2.3 提交更新相关指令

##### 1. git add

用于添加文件到暂存区，git add fileName 指令用于将文件从工作区添加到暂存区。

```bash
# 添加指定文件到暂存区
$ git add [file1] [file2] ...

# 添加指定目录到暂存区，包括子目录
$ git add [dir]

# 添加当前目录的所有文件到暂存区
$ git add .

# 添加每个变化前，都会要求确认
# 对于同一个文件的多处变化，可以实现分次提交
$ git add -p
```

##### 2. git commit

使用比较多的指令，把暂存区的所有内容提交到当前分支。

```bash
# 提交暂存区到仓库区
$ git commit -m [message]

# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]

# 提交工作区自上次 commit 之后的变化，直接到仓库区
$ git commit -a

# 提交时显示所有 diff 信息
$ git commit -v

# 使用一次新的 commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次 commit 的提交信息
$ git commit --amend -m [message]

# 重做上一次 commit，并包括指定文件的新变化
$ git commit --amend [file1] [file2] ...
```

##### 3. git diff

查看当前分支的修改。

```bash
# 显示暂存区和工作区的差异
$ git diff

# 显示暂存区和上一个 commit 的差异
$ git diff --cached [file]

# 显示工作区与当前分支最新 commit 之间的差异
$ git diff HEAD

# 显示两次提交之间的差异
$ git diff [first-branch]...[second-branch]

# 显示今天你写了多少行代码
$ git diff --shortstat "@{0 day ago}"
```

##### 4. git show

```bash
# 显示某次提交的元数据和内容变化
$ git show [commit]

# 显示某次提交发生变化的文件
$ git show --name-only [commit]

# 显示某次提交时，某个文件的内容
$ git show [commit]:[filename]
```

#### 2.4 远程同步

```bash
# 下载远程仓库的所有变动
$ git fetch [remote]

# 显示所有远程仓库
$ git remote -v

# 显示某个远程仓库的信息
$ git remote show [remote]

# 增加一个新的远程仓库，并命名
$ git remote add [shortname] [url]

# 取回远程仓库的变化，并与本地分支合并
$ git pull [remote] [branch]

# 上传本地指定分支到远程仓库
$ git push [remote] [branch]

# 强行推送当前分支到远程仓库，即使有冲突
$ git push [remote] --force

# 推送所有分支到远程仓库
$ git push [remote] --all
```

#### 2.5 恢复相关指令

##### 1. git reset

用于版本的回退，在 git 中 HEAD 代表当前版本，我们回退必须知道我们的版本号才能进行回退。这时就需要结合我们上面的 git log 进行版本号的查看，然后进行版本的回退。完整指令 git reset --hard commit_id。

```bash
# 重置暂存区的指定文件，与上一次 commit 保持一致，但工作区不变
$ git reset [file]

# 重置暂存区与工作区，与上一次 commit 保持一致
$ git reset --hard

# 重置当前分支的指针为指定 commit，同时重置暂存区，但工作区不变
$ git reset [commit]

# 重置当前分支的 HEAD 为指定 commit，同时重置暂存区和工作区，与指定 commit 一致
$ git reset --hard [commit]

# 重置当前 HEAD 为指定 commit，但保持暂存区和工作区不变
$ git reset --keep [commit]
```

##### 2. git checkout

命令 git checkout — readme.txt 意思就是，把 readme.txt 文件在工作区的修改全部撤销，这里有两种情况：

- readme.txt 自修改后还没有被放到暂存区，现在撤销修改就回到和版本库一模一样的状态;
- readme.txt 已经添加到暂存区后，又作了修改，现在撤销修改就回到添加到暂存区后的状态。

```bash
# 恢复暂存区的指定文件到工作区
$ git checkout [file]

# 恢复某个 commit 的指定文件到暂存区和工作区
$ git checkout [commit] [file]

# 恢复暂存区的所有文件到工作区
$ git checkout .
```

总之，这个操作就是将版本库中的文件替换为原来工作区中的文件，比如误删除后的恢复。

##### 3. git revert

```bash
# 新建一个 commit，用来撤销指定 commit
# 后者的所有变化都将被前者抵消，并且应用到当前分支
$ git revert [commit]
```

#### 2.6 标签

当我们发布版本时，都要给版本打上 TAG 标签，用于作为记录。

```bash
# 列出所有 tag
$ git tag

# 新建一个 tag 在当前 commit
$ git tag [tag]

# 新建一个 tag 在指定 commit
$ git tag [tag] [commit]

# 删除本地 tag
$ git tag -d [tag]

# 删除远程 tag
$ git push origin :refs/tags/[tagName]

# 查看 tag 信息
$ git show [tag]

# 提交指定 tag
$ git push [remote] [tag]

# 提交所有 tag
$ git push [remote] --tags

# 新建一个分支，指向某个 tag
$ git checkout -b [branch] [tag]
```

#### 2.7 文件

```bash
# 显示某个文件的版本历史，包括文件改名
$ git log --follow [file]
$ git whatchanged [file]

# 显示指定文件相关的每一次diff
$ git log -p [file]

# 显示指定文件是什么人在什么时间修改过
$ git blame [file]

# 删除工作区文件，并且将这次删除放入暂存区
$ git rm [file1] [file2] ...

# 停止追踪指定文件，但该文件会保留在工作区
$ git rm --cached [file]

# 改名文件，并且将这个改名放入暂存区
$ git mv [file-original] [file-renamed]
```

#### 2.8 配置

```bash
# 显示当前的 Git 配置
$ git config --list

# 编辑 Git 配置文件
$ git config -e [--global]

# 设置提交代码时的用户信息
$ git config [--global] user.name "[name]"
$ git config [--global] user.email "[email address]"
```

### 3. 常见基本 Git 指令工作流程

 一般来说在日常的工作中只要记住常用的指令就行了。

![repository](https://img-blog.csdnimg.cn/20190508202239833.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01yX2Rzdw==,size_16,color_FFFFFF,t_70)

常见的工作流程：

1. 克隆远程仓库：git clone
2. 建立远程连接：git branch --set-upstream
3. 添加修改：git add .
4. 提交到暂存区：git commit -m ""
5. 推送到远程仓库：git push -u origin master



### 感谢

[常用 Git 命令清单-[阮一峰](http://www.ruanyifeng.com/)](https://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)