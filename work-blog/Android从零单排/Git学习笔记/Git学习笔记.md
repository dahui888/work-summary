##Git学习笔记

>我们都知道，现在Git已经在很多企业都很普遍使用，作为一名Android开发人员，在许多开源项目中，都使用了Git作为版本的管理软件，能完美与Github进行对接，所以学习Git很有必要。

首先说下我的背景，在系统学习git的使用之前，对git有一丁点的了解，但是都不是很系统很熟悉，对常见的add、commit指令有一定了解，但是对git的分区以及分支方面是没有什么认识，所以感觉git的入手比较困难，因为平时工作采用的是SVN，只有在Gihub上用到一点Git，借用徐医生的一句话，现在Android Studio和Git的使用已经成为Androider必备的技能。特此，痛下狠心，花几天时间系统进行学习下。在我学习之前，我觉得以下几个问题是困扰我的最大问题：
- 分支是什么？分支有什么用？
- 工作区和暂存区有什么区别？

###一、Git的前生今世
作为Linux的版本管理工具而诞生，在Git之前存在SVN、CVS这些集中式的版本管理工具，但是由于集中式版本控制系统的速度慢，同时必须联网才能使用，所以遭到了Linux之父的反对，市面上同时也存在一些付费的版本管理工具，由于Linux的不断发展，BitKeeper的东家BitMover公司出于人道主义精神，授权Linux社区免费使用这个版本控制系统。但是中途出现了一些小插曲，BitMover收回Linux社区的免费使用权。然后，Linux之父Linus花了两周时间用C写了一个分布式版本控制系统，这就是Git！一个月之内，Linux系统的源码已经由Git管理了！牛是怎么定义的呢？大家可以体会一下。

在我的工作中，公司使用SVN进行代码的版本管理比较多，平时只有在Github上使用Git进行版本管理。基于我这种背景，对Git的认识就比较肤浅，没有在商业项目上进行实战操作过。说先说说我使用SVN的感受，在Windows系统上使用起来很方便，但是每次使用前都要先去服务器Update下，获取到最新的代码，这个还能忍受，但是一旦离开公司，就无法进行操作了。想更新下仓库、查看下提交日志什么都无法使用，这个就比较蛋疼了。Git作为分布式版本管理系统有哪些优点呢？一是分布式管理系统没有“中央服务器”，每个机器上都有一份完整的仓库版本，所以这就不需要我们每次联网进行操作。其次，Git的强大分支功能设计和工作区的设计。

###二、Git的安装
工欲善其事必先利其器，在学习Git的使用之前，我们先安装Git进行。最初Git只在Linux系统中使用，毕竟初衷就是为了管理Linux的代码而创建，现在已经在Windows和Mac的系统上都可以使用。

**Linux上安装Git**

通过一条**sudo apt-get install git**就可以直接完成Git的安装，非常简单

**Windows上安装Git**

现在windows系统中已经有了Git的版本，直接下载安装即可。[windows下git](https://github.com/git-for-windows/git/releases/download/v2.11.0.windows.3/Git-2.11.0.3-64-bit.exe)

在安装完Git之后，我们需要通过以下指令配置以下你的机器标志。

    $ git config --global user.name "Your Name"
    $ git config --global user.email "email@example.com"

因为Git是分布式版本控制系统，所以，每个机器都必须自报家门：你的名字和Email地址。你也许会担心，如果有人故意冒充别人怎么办？这个不必担心，首先我们相信大家都是善良无知的群众，其次，真的有冒充的也是有办法可查的。

注意git config命令的--global参数，用了这个参数，表示你这台机器上所有的Git仓库都会使用这个配置，当然也可以对某个仓库指定不同的用户名和Email地址。

###三、Git中涉及的基本概念
####**1、仓库**

仓库又称为版本库，英语名称repository。说得通俗一点就是需要用Git进行管理的文件夹。这个仓库中存放了我们的代码、文件，我们可以通过Git对这些代码进行管理。

####**2、工作区和暂存区**

**（1）、工作区**

我们在工作区中进行代码的编辑，工作区就是我们正在操作的文件夹。

**（2）、暂存区**

工作区有一个隐藏目录.git，这个不算工作区，而是Git的版本库。
Git的版本库里存了很多东西，其中最重要的就是称为stage（或者叫index）的暂存区，还有Git为我们自动创建的第一个分支master，以及指向master的一个指针叫HEAD。

####**3、远程仓库**
我们知道Git是分布式版本管理系统，那么同一个Git仓库可以存在任何机器上，但是最初肯定只有一个机器存在这个仓库，然后，别的机器克隆这个仓库，所以就需要存在一个机器扮演服务器的角色，不过现在已经存在了Github的一个网站用于托管Git仓库，我们只要从这个远程仓库进行clone即可。

####**4、分支**
分支就是岔路，比如几个人一起旅行，走到一个岔路口，每个岔路口都有不同的风景，我们约定几天后在某个地点集合。在我们的开发过程中，我们都在一条版本库线上开发，突然有天老大安排了一个开发任务给你，这个时候，你就可以在主线上拉出一条分支，单独进行你自己的开发，这个分支只有你自己知道，不影响主线的开发者使用。最后等待你的开发任务完成，进行合并到主线即可。

####**5、标签**
发布一个版本时，我们通常先在版本库中打一个标签（tag），这样，就唯一确定了打标签时刻的版本。将来无论什么时候，取某个标签的版本，就是把那个打标签的时刻的历史版本取出来。所以，标签也是版本库的一个快照。

###四、Git常见指令

<h4>git init</h4>

这是我们使用的第一个指令，用于初始化一个git仓库。此时会生成一个.git文件夹，如果没有看到可以通过ls -ah查看。

![git](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E4%BB%8E%E9%9B%B6%E5%8D%95%E6%8E%92/Git%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/git%20init.png)

<h4>git add</h4>
用于添加文件到暂存区，git add fileName指令用于将文件从工作区添加到暂存区。例如我们在工作区创建一个readme.txt文件。

![add](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E4%BB%8E%E9%9B%B6%E5%8D%95%E6%8E%92/Git%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/git%20add.png)

<h4>git status</h4>
查看当前仓库的状态，通过git status指令可以随时让我们掌握仓库的状态。

![status](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E4%BB%8E%E9%9B%B6%E5%8D%95%E6%8E%92/Git%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/status.png)

根据上面的提示，在分支master上，出现了新文件readme.txt。


<h4>git commit</h4>
实际上就是把暂存区的所有内容提交到当前分支。

![commit](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E4%BB%8E%E9%9B%B6%E5%8D%95%E6%8E%92/Git%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/commit.png)

<h4>git diff</h4>
git diff顾名思义就是查看difference，显示的格式正是Unix通用的diff格式，通过该命令可以查看仓库文件修改的内容。比如我们在新增的readme.txt文件中新增一行文字。

![diff](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E4%BB%8E%E9%9B%B6%E5%8D%95%E6%8E%92/Git%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/diff.png)

<h4>git mv </h4>
该命令用于文件的重命名，同时可参照[ linux下的文件重命名](http://blog.csdn.net/mr_dsw/article/details/54236542)进行理解。注意，重命名后需要commit提交。

![mv](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E4%BB%8E%E9%9B%B6%E5%8D%95%E6%8E%92/Git%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/mv.png)

<h4>git log</h4>
查看我们提交的日志。里面包含提交提交的作者和日期。

![log](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E4%BB%8E%E9%9B%B6%E5%8D%95%E6%8E%92/Git%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/log.png)

这样的格式，我们看着也不是很舒服，此时可以通过指令
**git log --pretty=oneline**进行格式下看看。

![oneline](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E4%BB%8E%E9%9B%B6%E5%8D%95%E6%8E%92/Git%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/oneline.png)

需要友情提示的是，你看到的一大串类似48a5c3a4...55f6986a的是commit id（版本号），和SVN不一样，Git的commit id不是1，2，3……递增的数字，而是一个SHA1计算出来的一个非常大的数字，用十六进制表示，而且你看到的commit id和我的肯定不一样，以你自己的为准。为什么commit id需要用这么一大串数字表示呢？因为Git是分布式的版本控制系统，后面我们还要研究多人在同一个版本库里工作，如果大家都用1，2，3……作为版本号，那肯定就冲突了。

<h4>git reset</h4>
用于版本的回退，在git中HEAD代表当前版本，我们回退必须知道我们的版本号才能进行回退。这时就需要结合我们上面的git log进行版本号的查看，然后进行版本的回退。完整指令**git reset --hard commit_id**

![reset](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E4%BB%8E%E9%9B%B6%E5%8D%95%E6%8E%92/Git%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/reset.png)

为什么git回退版本很快呢？这时因为git每次版本的更新只是让HEAD头指针进行版本状态的指定，然后把工作区进行更新。如果我们回退之后后悔了，就需要借助git reflog进行操作命令历史查看，然后进行恢复。

<h4>git checkout -- fileName</h4>
命令git checkout -- readme.txt意思就是，把readme.txt文件在工作区的修改全部撤销，这里有两种情况：

1. 一种是readme.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态;
1. 一种是readme.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

总之，这个操作就是将版本库中的文件替换为原来工作区中的文件，比如误删除后的恢复。

<h4>git remote add origin git@server-name:path/repo-name.git</h4>
添加远程仓库，一般我们在Github上进行托管我们的代码工程，所以当我们本地仓库的代码进行推送到远程仓库的时候，就需要我们建立一个远程仓库，以便进行推送。

<h4>git clone url</h4>
用于clone复制远程连接的资源库至本地。要克隆一个仓库，首先必须知道仓库的地址，然后使用git clone命令克隆。Git支持多种协议，包括https，但通过ssh支持的原生git协议速度最快。
![clone](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E4%BB%8E%E9%9B%B6%E5%8D%95%E6%8E%92/Git%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/clone.png)

<h4>git branch</h4>
该指令用于创建一个分支。完整命令：git branch <name>。如果后面没有branchName则表示查看当前存在的分支。每次提交，Git都把它们串成一条时间线，这条时间线就是一个分支。截止到目前，只有一条时间线，在Git里，这个分支叫主分支，即master分支。
![branch](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E4%BB%8E%E9%9B%B6%E5%8D%95%E6%8E%92/Git%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/branch.png)

<h4>git checkout</h4>
该指令用于切换分支。完整命令：git checkout <name>。比如上面我们创建的develop分支，我们从master分支切换到develop分支进行开发工作。

![checkout](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E4%BB%8E%E9%9B%B6%E5%8D%95%E6%8E%92/Git%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/checkout.png)

当然，我们也可以通过**git checkout -b <name>**指令，直接完成创建分支并切换到新建的分支。

![release](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E4%BB%8E%E9%9B%B6%E5%8D%95%E6%8E%92/Git%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/release.png)

<h4>git merge</h4>
该指令用于合并分支。git merge命令用于合并指定分支到当前分支。我们在指定分支上修改的内容，必须通过分支合并才能同步到主要分支上。

![merge](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E4%BB%8E%E9%9B%B6%E5%8D%95%E6%8E%92/Git%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/merge.png)

<h4>git branch -d <name></h4>
删除指定分支。
![delete](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E4%BB%8E%E9%9B%B6%E5%8D%95%E6%8E%92/Git%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/delete.png)

<h4>git log --graph</h4>
当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。用git log --graph命令可以看到分支合并图。

![graph](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E4%BB%8E%E9%9B%B6%E5%8D%95%E6%8E%92/Git%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/graph.png)

这些指令，总得概括起来就是。

- 初始化一个Git仓库，使用git init命令。

- 添加文件到Git仓库，分两步：

	1. 	第一步，使用命令git add <file>，注意，可反复多次使用，添加多个文件；
	1. 	第二步，使用命令git commit，完成。

![gitall](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E4%BB%8E%E9%9B%B6%E5%8D%95%E6%8E%92/Git%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/0.jpg)

- Git鼓励大量使用分支：
	1. 	查看分支：git branch
	1. 	创建分支：git branch <name>
	1. 	切换分支：git checkout <name>
	1. 	创建+切换分支：git checkout -b <name>
	1. 	合并某分支到当前分支：git merge <name>
	1. 	删除分支：git branch -d <name>

###、五Git分支
####1、分支的用处
分支作为git的一个特色，很方便的提供了开发的使用。我们可以通过创建不同的分支来完成开发任务，然后合并到主分支上。

####2、常见的分支操作
通常我们默认都会有一个主分支叫 master ，在版本回退里，你已经知道，每次提交，Git都把它们串成一条时间线，这条时间线就是一个分支。截止到目前，只有一条时间线，在Git里，这个分支叫主分支，即master分支。HEAD严格来说不是指向提交，而是指向master，master才是指向提交的，所以，HEAD指向的就是当前分支。

一开始的时候，master分支是一条线，Git用master指向最新的提交，再用HEAD指向master，就能确定当前分支，以及当前分支的提交点：下面我们先来看下关于分支的一些基本操作：

（1）、新建一个叫 develop 的分支

     git branch develop

 这里稍微提醒下大家，新建分支的命令是基于当前所在分支的基础上进行的，即以上是基于
mater 分支新建了一个叫做 develop 的分支，此时 develop 分支跟 master 分支的内容完全一
样。如果你有 A、B、C三个分支，三个分支是三位同学的，各分支内容不一样，如果你当前
是在 B 分支，如果执行新建分支命令，则新建的分支内容跟 B 分支是一样的，同理如果当前
所在是 C 分支，那就是基于 C 分支基础上新建的分支。

（2）、切换到 develop 分支

     git checkout develop

如果把以上两步合并，即新建并且自动切换到 develop 分支：

	git checkout -b develop

（3）、把 develop 分支推送到远程仓库

	git push origin develop

（4）、如果你远程的分支想取名叫 develop2 ，那执行以下代码：

 	git push origin develop:develop2

 但是强烈不建议这样，这会导致很混乱，很难管理，所以建议本地分支跟远程分支名要保持
 一致。

（5）、 查看本地分支列表

	git branch

（6）、查看远程分支列表

 	git branch -r

（7）、删除本地分支

	git branch -d develop
	git branch -D develop (强制删除)

（8）、删除远程分支

 	git push origin :develop

（9）、如果远程分支有个 develop ，而本地没有，你想把远程的 develop 分支迁到本地：

	git checkout develop origin/develop

（10）、 同样的把远程分支迁到本地顺便切换到该分支：

	git checkout -b develop origin/develop

####3、团队协作
>注本段内容从stormzhang的讲解，非常棒，特此记录下。分支的使用也是git很大的特色，一定要把这段内容理解。

![flow](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E4%BB%8E%E9%9B%B6%E5%8D%95%E6%8E%92/Git%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/team.png)

第一次看上面那个图是不是一脸懵逼？跟我当时一样，不急，我来用简单的话给你们解释
下。
一般开发来说，大部分情况下都会拥有两个分支 master 和 develop，他们的职责分别是：

1. master：永远处在即将发布(production-ready)状态
1. develop：最新的开发状态

确切的说 master、develop 分支大部分情况下都会保持一致，只有在上线前的测试阶段
develop 比 master 的代码要多，一旦测试没问题，准备发布了，这时候会将 develop 合并到
master 上。

但是我们发布之后又会进行下一版本的功能开发，开发中间可能又会遇到需要紧急修复 bug
，一个功能开发完成之后突然需求变动了等情况，所以 Git Flow 除了以上 master 和 develop
两个主要分支以外，还提出了以下三个辅助分支：

1. feature: 开发新功能的分支, 基于 develop, 完成后 merge 分支切回 develop；
1. release: 准备要发布版本的分支, 用来修复 bug，基于 develop，完成后 merge 分支切回
 develop 和 master；
1. hotfix: 修复 master 上的问题, 等不及 release 版本就必须马上上线. 基于 master, 完成后merge 回 master 和 develop。

什么意思呢？

举个例子，假设我们已经有 master 和 develop 两个分支了，这个时候我们准备做一个功能
A，第一步我们要做的，就是基于 develop 分支新建个分支：

	git branch feature/A

看到了吧，其实就是一个规范，规定了所有开发的功能分支都以 feature 为前缀。
但是这个时候做着做着发现线上有一个紧急的 bug 需要修复，那赶紧停下手头的工作，立刻
切换到 master 分支，然后再此基础上新建一个分支：

	git branch hotfix/B

代表新建了一个紧急修复分支，修复完成之后直接合并到 develop 和 master ，然后发布。
然后再切回我们的 feature/A 分支继续着我们的开发，如果开发完了，那么合并回 develop 分
支，然后在 develop 分支属于测试环境，跟后端对接并且测试的差不多了，感觉可以发布到
正式环境了，这个时候再新建一个 release 分支：

	git branch release/1.0

这个时候所有的 api、数据等都是正式环境，然后在这个分支上进行最后的测试，发现 bug 直
接进行修改，直到测试 ok 达到了发布的标准，最后把该分支合并到 develop 和 master 然后
进行发布。

以上就是 Git Flow 的概念与大概流程，看起来很复杂，但是对于人数比较多的团队协作现实
开发中确实会遇到这么复杂的情况，是目前很流行的一套分支管理流程，但是有人会问每次
都要各种操作，合并来合并去，有点麻烦，哈哈，这点 Git Flow 早就想到了，为此还专门推
出了一个 Git Flow 的工具，并且是开源的：

GitHub 开源地址：https://github.com/nvie/gitflow

简单点来说，就是这个工具帮我们省下了很多步骤，比如我们当前处于 master 分支，如果想
要开发一个新的功能，第一步切换到 develop 分支，第二步新建一个以 feature 开头的分支
名，有了 Git Flow 直接如下操作完成了：

	git flow feature start A

这个分支完成之后，需要合并到 develop 分支，然而直接进行如下操作就行：

	git flow feature finish A

如果是 hotfix 或者 release 分支甚至会自动帮你合并到 develop、master 两个分支。

###六、Github的使用
Github作为免费的远程仓库，我们在上面注册账号并新建一个仓库，然后通过git remote add指令进行本地仓库和远程仓库的关联。

####1、SSH
当我们拥有一个Github账号后，就可以进行随意clone优秀的开源项目，但是此时我们无法随意提交代码，不然没有一个认证方式那就乱套了。而Github就是采用SSH授权进行认证的。为了完成认证方式，我们需要生成SSH秘钥。

（1）、输入 ssh-keygen -t rsa ，指定 rsa 算法生成密钥，接着连续三个回
车键（不需要输入密码），然后就会生成两个文件 id_rsa 和 id_rsa.pub ，而 id_rsa 是密钥，
id_rsa.pub 就是公钥。这两文件默认分别在如下目录里生成：
Linux/Mac 系统 在 ~/.ssh 下，win系统在 /c/Documents and Settings/username/.ssh 下，
都是隐藏文件，相信你们有办法查看的。
接下来要做的是把 id_rsa.pub 的内容添加到 GitHub 上，这样你本地的 id_rsa 密钥跟 GitHub
上的 id_rsa.pub 公钥进行配对，授权成功才可以提交代码。

（2）、在Github上添加SSH key。
第一步先在 GitHub 上的设置页面，点击最左侧 SSH and GPG keys，然后点击右上角的 New SSH key 按钮，需要做的只是在 Key 那栏把 id_rsa.pub 公钥文件里的内容复制粘贴进去就可以了，Title 那栏不需要填写，点击 Add SSH key 按钮就ok了。

####2、Push 和 Pull
我们在本地仓库完成的开发内容，需要推送到远程仓库中。同时我们也需要从远程仓库下载更新本地的版本库。这里就需要使用**git push origin master**指令，将本地仓库master主线推送到远程仓库origin。**git pull origin master**意思就是把远程最新的代码更新到本地。一般我们在 push 之前都会先 pull ，这样不容易冲
突。

##总结
算是对这几天学习Git的一个总结吧！Git还有很多强大的命令没有接触到，总结的这些算是入门级别，再接再厉，同时感谢

1. [廖雪峰的Github教程](http://www.liaoxuefeng.com/)、
1. [stormzhang撰写的Github教程](http://stormzhang.com)