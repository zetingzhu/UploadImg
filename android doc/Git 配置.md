# **Git 使用命令**

# **Git 配置**

## **生成SSH KEY**

$ ssh-keygen -t rsa -C zhu_zeting@163.com

如果我们 **Mac** 上面已经有了 **ssh-key** 再创建 **ssh-key** 的话，默认会在 **~/.ssh/** 目录下生成 **id_rsa** 和 **id_rsa.pub** 两个文件，如果不自定义，就会把原有的给覆盖掉。为了加以区分，我们需要自定义一下生成的 **key** 的名字,后面的**id_rsa_zzt**为你自定义的名字

allendeiMac:.ssh allen$ **ssh-keygen -t rsa -C zhu_zeting@163.com**

Generating public/private rsa key pair.

Enter file in which to save the key (/Users/allen/.ssh/id_rsa): **id_rsa_zzt**

Enter passphrase (empty for no passphrase): 第一次密码

Enter same passphrase again: 第二次密码

Your identification has been saved in id_rsa_zzt.

Your public key has been saved in id_rsa_zzt.pub.

The key fingerprint is:

SHA256:FRD925YVR2Ct8IWOMXJ9gzPulu9MqfbjbaqV4l2OuKA zhu_zeting@163.com

The key's randomart image is:

+---[RSA 2048]----+

|        o+.  .+=.|

|          o.==oo=|

|          .+.Bo++|

|         .  o.+ .|

|        S   .o.o |

|            .++..|

|          . o.+o.|

|         . o *+*o|

|        E   *+BB+|

+----[SHA256]-----+

allendeiMac:.ssh allen$

## **配置ssh-key**

接下来将我们配置好的ssh-key的公钥提交到github上并进行测试连接

在 ~/.ssh/ 目录下会生成 id_rsa_zzt 和 id_rsa_zzt.pub 私钥和公钥。 我们将 id_rsa_zzt.pub 中的内容粘帖到 github 的 SSH-key 的配置中，这里获取 id_rsa_zzt.pub 的内容可以使用终端也可以使用 sublime 或 atom 等一些编辑器。

查看.pub文件

$ cd ~/.ssh 切换目录到这个路径

$ vim id_rsa.pub 将这个文件的内容显示到终端上

本地电脑之后读取id_rsa id_rsa.pub 文件，最好的办法是把原有的备份，把自己生成的 id_rsa_zzt id_rsa_zzt.pub 文件改名为 id_rsa id_rsa.pub

## **在GitHub的设置中粘贴公钥**

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/97fa7b8c9aeca362e14dc0d4610548a6/0)        

配置成功后

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/b70557a9079b89610f85b706376d37f4/0)        

## **查看本地git配置**

显示当前的Git配置

$ git config --list

$ git config -l

```
allendeiMac:.ssh allen$ git config -l
credential.helper=osxkeychain
user.name=zeting
user.email=zeting@vipcare.com
core.autocrlf=false
```

## **增加/删除文件**

添加指定文件到暂存区

$ git add [file1] [file2] ...

添加指定目录到暂存区，包括子目录

$ git add [dir]

```
$ git add mykotiln/src/
提交完成后红色文件会变成绿色
```

添加当前目录的所有文件到暂存区$ git add .

添加每个变化前，都会要求确认# 对于同一个文件的多处变化，可以实现分次提交

$ git add -p

删除工作区文件，并且将这次删除放入暂存区

$ git rm [file1] [file2] ...

停止追踪指定文件，但该文件会保留在工作区

$ git rm --cached [file]

改名文件，并且将这个改名放入暂存区

$ git mv [file-original] [file-renamed]

## **代码提交**

提交暂存区到仓库区

$ git commit -m [message]

提交指定目录文件到仓库区

$ git commit  [dir] -m [message]

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/f96236f8c344ed3bf69c1639c7e5448c/0)        

提交完成后，绿色提交文件和蓝色修改文件会变成灰色已经提交颜色

提交暂存区的指定文件到仓库区

$ git commit [file1] [file2] ... -m [message]

 提交工作区自上次commit之后的变化，直接到仓库区

$ git commit -a

提交时显示所有diff信息

$ git commit -v

使用一次新的commit，替代上一次提交# 如果代码没有任何新变化，则用来改写上一次commit的提交信息

$ git commit --amend -m [message]

重做上一次commit，并包括指定文件的新变化

$ git commit --amend [file1] [file2] ...

## **查看信息**

显示有变更的文件

$ git status

$ git status  [dir]

​![img](https://qqadapt.qpic.cn/txdocpic/0/a3b7425401aedea6f05712a4c84a63ae/0)        

显示当前分支的版本历史

$ git log

$ git log -5 查看最近5条

$ git log --pretty=oneline // 只会留下commit  id (版本号 (用SHA1字串表示))和 提交版本时的描述信息

按q 可以退出查看

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/0fd8c8c172575c446797cecfd6184ef6/0)        

 显示commit历史，以及每次commit发生变更的文件

$ git log --stat

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/ae3689d8e2e04c8e10790ae54f71a580/0)        

搜索提交历史，根据关键词

$ git log -S [keyword]

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/d5b0b13e5075d07fc889dd78ceb49a6a/0)        

显示某个commit之后的所有变动，每个commit占据一行

$ git log [tag] HEAD --pretty=format:%s

显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件

$ git log [tag] HEAD --grep feature

显示某个文件的版本历史，包括文件改名

$ git log --follow [file]

$ git whatchanged [file]

显示指定文件相关的每一次diff

$ git log -p [file]

显示过去5次提交

$ git log -5 --pretty --oneline

显示所有提交过的用户，按提交次数排序

$ git shortlog -sn

显示指定文件是什么人在什么时间修改过

$ git blame [file]

显示暂存区和工作区的代码差异

$ git diff

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/ff0707f061d5bb2c29db0a2f1a02cf8d/0)        

 显示暂存区和上一个commit的差异

$ git diff --cached [file]

显示工作区与当前分支最新commit之间的差异

$ git diff HEAD

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/889b762b514751c96eef9660e00a35e5/0)        

显示两次提交之间的差异

$ git diff [first-branch]...[second-branch]

显示今天你写了多少行代码

$ git diff --shortstat "@{0 day ago}"

显示某次提交的元数据和内容变化$ git show [commit]# 显示某次提交发生变化的文件

$ git show --name-only [commit]

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/ba8240fe7f856301038b93a5f12773d6/0)        

显示某次提交时，某个文件的内容

$ git show [commit]:[filename]

显示当前分支的最近几次提交

$ git reflog

从本地master拉取代码更新当前分支：branch 一般为master

$ git rebase [branch]

## **Git查看远程分支地址**

git remote -v

```
D:\ZZTAndroid\Android_Work\RxjavaSamples>git remote -v
origin  https://gitee.com/ZeTing/RxjavaSamples.git (fetch)
origin  https://gitee.com/ZeTing/RxjavaSamples.git (push)
D:\ZZTAndroid\Android_Work\RxjavaSamples>
```

## **分支**

列出所有本地分支

$ git branch

列出所有远程分支

$ git branch -r

列出所有本地分支和远程分支

$ git branch -a

新建一个分支，但依然停留在当前分支

$ git branch [branch-name]

新建一个分支，并切换到该分支

$ git checkout -b [branch]

新建一个分支，指向指定commit

$ git branch [branch] [commit]

新建一个分支，与指定的远程分支建立追踪关系

$ git branch --track [branch] [

te-branch]

切换到指定分支，并更新工作区

$ git checkout [branch-name]

切换到上一个分支

$ git checkout -

建立追踪关系，在现有分支与指定的远程分支之间

$ git branch --set-upstream [branch] [remote-branch]

合并指定分支到当前分支

$ git merge [branch]

选择一个commit，合并进当前分支

$ git cherry-pick [commit]

删除分支

$ git branch -d [branch-name]

删除远程分支

$ git push origin --delete [branch-name]

$ git branch -dr [remote/branch]

## **标签**

列出所有tag

$ git tag

新建一个tag在当前commit

$ git tag [tag]

新建一个tag在指定commit

$ git tag [tag] [commit]

删除本地tag

$ git tag -d [tag]

删除远程tag

$ git push origin :refs/tags/[tagName]

查看tag信息

$ git show [tag]

提交指定tag

$ git push [remote] [tag]

提交所有tag

$ git push [remote] --tags

新建一个分支，指向某个tag

$ git checkout -b [branch] [tag]

## **常见使用**

git clone  

git branch [分支名] 创建分支

git branch 查看本地所有分支

git checkout [分支名称] 切换分支

---写代码---

git status （查看文件改变记录）

git diff (查看代码级改变)

git add (1：确认改变)

git commit -m 提交注释 (2：提交到当前分支的本地工作区)

git push [远程分支：origin] [本地分支的名称]

去git 管理网站创建Merge Request

等待合并

----管理员合并所有人的Merge Request----

checkout master (切换至Master)

git pull (从远程master 更新至 本地master)

checkout [branch] (切换至本地分支)

git rebase master [从本地 master 更新当前分支]

----是否有冲突----

----有----

----如何解决冲突----

1、在VS中操作代码文件并解决冲突

2、git add . 加入待提交

3、git rebase --continue

----如果仍然有冲突，重复1/2/3步骤

4、git rebase --skip

5、git push -f origin [branch] 强推

-----去网站重新创建Merge Request-------

结束，等待合并，重复上述对应步骤.......

## [**git 取消commit**](USER_CANCEL)

git如何撤销上一次commit操作

1.第一种情况：还没有push，只是在本地commit

git reset --soft|--mixed|--hard <commit_id>

git push develop develop --force  (本地分支和远程分支都是 develop)

这里的<commit_id>就是每次commit的SHA-1，可以在log里查看到

--mixed    会保留源码,只是将git commit和index 信息回退到了某个版本.

--soft   保留源码,只回退到commit信息到某个版本.不涉及index的回退,如果还需要提交,直接commit即可.

--hard    源码也会回退到某个版本,commit和index 都会回退到某个版本.(注意,这种方式是改变本地代码仓库源码)

当然有人在push代码以后,也使用 reset --hard <commit...> 回退代码到某个版本之前,但是这样会有一个问题,你线上的代码没有变,线上commit,index都没有变,当你把本地代码修改完提交的时候你会发现全是冲突.....这时换下一种

2.commit push 代码已经更新到远程仓库

对于已经把代码push到线上仓库,你回退本地代码其实也想同时回退线上代码,回滚到某个指定的版本,线上,线下代码保持一致.你要用到下面的命令

git revert <commit_id>

revert 之后你的本地代码会回滚到指定的历史版本,这时你再 git push 既可以把线上的代码更新。

注意：git revert是用一次新的commit来回滚之前的commit，git reset是直接删除指定的commit，看似达到的效果是一样的,其实完全不同。

第一:上面我们说的如果你已经push到线上代码库, reset 删除指定commit以后,你git push可能导致一大堆冲突.但是revert 并不会.

第二:如果在日后现有分支和历史分支需要合并的时候,reset 恢复部分的代码依然会出现在历史分支里.但是revert 方向提交的commit 并不会出现在历史分支里.

第三:reset 是在正常的commit历史中,删除了指定的commit,这时 HEAD 是向后移动了,而 revert 是在正常的commit历史中再commit一次,只不过是反向提交,他的 HEAD 是一直向前的.

## **如何替换git上的master分支**

希望用seotweaks替换到master

方法1：使用-s ours选项合并

git checkout seotweaks

git merge -s ours master  冲突以seotweaks版本为主

git checkout master

git merge seotweaks

方法2：改名master为old-master，然后将seotweaks更改为master，然后push。

git branch -m master old-master

git branch -m seotweaks master

git push -f origin master

方法3：将本地的旧分支 master 重置成新分支

git checkout master // 切换到旧的分支

git reset --hard develop // 将本地的旧分支 master 重置成 develop

git push origin master --force // 再推送到远程仓库

## **stGit 冲突：Your local changes would be overwritten by merge. Commit, stash or revert them to proceed.**

解决方案有三种：

1，无视，直接commit自己的代码。

git commit -m "your msg"

2，stash

​    stash翻译为“隐藏”，如下操作：

git stash

git pull

git stash pop

然后diff一下文件，看看自动合并的情况，并作出需要的修改。

git stash: 备份当前的工作区的内容，从最近的一次提交中读取相关内容，让工作区保证和上次提交的内容一致。同时，将当前的工作区内容保存到Git栈中。

git stash pop: 从Git栈中读取最近一次保存的内容，恢复工作区的相关内容。由于可能存在多个Stash的内容，所以用栈来管理，pop会从最近的一个stash中读取内容并恢复。

git stash list: 显示Git栈内的所有备份，可以利用这个列表来决定从那个地方恢复。

git stash clear: 清空Git栈。此时使用gitg等图形化工具会发现，原来stash的哪些节点都消失了。

3，硬覆盖：放弃本地修改，直接用git上的代码覆盖本地代码：

git reset --hard

git pull

补充在 android studio 中的上述操作对应的图：

方法 2 stash：

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/5073a066948b0486450b76ca0fb30842/0)        

方法 3 硬覆盖：

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/1af8c9d4e06568bdf6fc117ade606847/0)        

然后选择

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/1098e343bf76c6a64925006f98e3ec0b/0)        

git stash #可用来暂存当前正在进行的工作

git stash pop #从Git栈中读取最近一次保存的内容

git stash list #显示Git栈内的所有备份

git stash clear #清空Git栈

git stash apply stash@{1} #可以将你指定版本号为stash@{1}的工作取出来

## **实际操作使用步骤**

- 比较一下本地文件的修改状态

$ git status app

- 比较工作区和仓库去的文件差别

$ git diff  app

$ git diff --stat app

- 添加文件到本地仓库

$ git add app

- 提交代码到本地仓库

$ git commit app -m 提交3.7功能，遥控器，轨迹点，

- 提交本地分支仓库到远程仓库

$ git push origin zeting:zeting

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/ba477c7a4ca4a369e6b6ea810ab6b552/0)        

$ git push origin blueOffline

错误信息

Warning: Your console font probably doesn't support Unicode. If you experience strange characters in the output, consider switching to a TrueType font such as Consolas!

git config [--global] core.quotepath off

git config [--global] --unset i18n.logoutputencoding

git config [--global] --unset i18n.commitencoding

git config  core.quotepath off

git config  --unset i18n.logoutputencoding

git config  --unset i18n.commitencoding

- 将当前分支推送到origin主机的对应分支

$ git push origin

- 比较本地仓库和远程仓库代码区别

a.更新本地远程分支

git fetch origin

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/f70211e97646e91faddaac431b8b18c2/0)        

b.查看本地分支和远程分支提交的记录

$ git log zeting..origin/zeting

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/457c0a26c61faaae0edd7ce1f363d337/0)        

查看本地分支和远程分支提交的记录已经相信的提交信息

$ git log -p zeting..origin/zeting

查看本地分支的提交记录

$ git log zeting

查看远程分支的提交记录

$ git log origin/zeting

c.统计本地分支和远程分支的文件差异

$ git diff --stat zeting origin/zeting

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/94de80b387e49bd50050aa90e5fc503c/0)            比较本地分支和远程分支的代码区别

  $ git diff zeting origin/zeting

比较本地两个分支显示出所有有差异的文件列表

git diff  --stat  branch1 branch2

$ git diff --stat master v401

d.合并远程分支到吗到本地库代码

$ git merge origin/zeting

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/2a19f9c0316666f46cb682f19477d9ca/0)        

错误信息

Please move or remove them before you can merge.

解决方法

\1. 提交添加没有提交的文件

$ git add .

\2. 查看存储的列表

$ git stash list

\3. 把没有提交的文件存储起来

$ git stash

\4. 合并分支代码

$ git merge origin/blueOffline

合并某个分支到当前分支（合并zeting分支到当前分支）

$ git merge zeting

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/9dc21367f2f8aa894126e526d06bdd86/0)        

### 建立本地分支并切换分支

$ git checkout -b dev3.9

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/478cc6efc0deabc43abd7f4359a8d7fb/0)        

### 把新建的本地分支推送到远程仓库

$ git push origin dev3.9:dev3.9

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/3d14f19f365d464b0901d151c62af704/0)        

推送错误error: failed to push some refs to 'git@dev.qqfind.me:niu_android.git'

​                 ![img](https://qqadapt.qpic.cn/txdocpic/0/aa2c6fb490c92df5e2639df350d60dd3/0)        

可以通过如下命令进行代码合并【注：pull=fetch+merge]

$git pull --rebase origin dev3.9

执行上面代码后可以看到本地代码库中多了README.md文件

此时再执行语句

$git push -u origin dev3.9

即可完成代码上传到github

### 把远程分支的代码更新到本地分支来

$ git fetch origin dev3.9:dev3.9

### **git 放弃本地修改，强制拉取更新**

git fetch 指令是下载远程仓库最新内容，不做合并

git fetch --all

git reset 指令把HEAD指向master最新版本

git reset --hard origin/master

git fetch

### Git中从远程的分支获取最新的版本到本地有这样2个命令：

\1. git fetch：相当于是从远程获取最新版本到本地，不会自动merge

Git fetch origin master

git log -p master..origin/master

git merge origin/master

​    以上命令的含义：

   首先从远程的origin的master主分支下载最新的版本到origin/master分支上

   然后比较本地的master分支和origin/master分支的差别

   最后进行合并

   上述过程其实可以用以下更清晰的方式来进行：

git fetch origin master:tmp

git diff tmp

git merge tmp

从远程获取最新的版本到本地的test分支上

之后再进行比较合并

\2. git pull：相当于是从远程获取最新版本并merge到本地

git pull origin master

上述命令其实相当于git fetch 和 git merge

在实际使用中，git fetch更安全一些

因为在merge前，我们可以查看更新情况，然后再决定是否合并

### 查看两个版本某个文件差异

git diff ffd98b291e0caa6c33575c1ef465eae661ce40c9:filename b8e7b00c02b95b320f14b625663fdecf2d63e74c:filename

## **删除git从新绑定项目提交地址**

正确步骤：

\1. git init //初始化仓库

\2. git add .(文件name) //添加文件到本地仓库

\3. git commit -m "first commit" //添加文件描述信息

\4. git remote add origin + 远程仓库地址 //链接远程仓库，创建主分支

\5. git pull origin master // 把本地仓库的变化连接到远程仓库主分支

\6. git push -u origin master //把本地仓库的文件推送到远程仓库

## 推送多个仓库

```
git remote set-url --add origin https://gitee.com/ZeTing/ztyyxx.git
```

# **Git 问题解决**

### You have not concluded your merge. (MERGE_HEAD exists)

当本地有提交，分支也有提交，这个时候在pull 提示以上错误

解决方案：

① 保存本地代码

② 执行git fetch --all

③ 执行git reset --hard origin/master ----> git reset 把HEAD指向刚刚下载的最新的版本

④pull主分支下的代码

⑤解决冲突，然后提交代码到自己的分支那里

1 执行git fetch --all

​                 ![img](https://docimg7.docs.qq.com/image/yp5kCzLnbqCjc7xpYzZLNw?w=801&h=823)        

2 git reset --hard origin/master

​                 ![img](https://docimg8.docs.qq.com/image/PcdnbO11jinh1XCjoK1wgw?w=810&h=833)        

​                 ![img](https://docimg2.docs.qq.com/image/xSyBT93QqJ9w3czHs2szFw?w=1920&h=1039)        

3 pull主分支下的代码

​                 ![img](https://docimg10.docs.qq.com/image/QnB0EjoEmBaQxvzYFb6O0A?w=810&h=831)        

4 解决冲突

### Support for password authentication was removed on August 13, 2021. Please use a personal access token instead

D:\ZZTAndroid\Android_Work\ZT_KLineChart>git push

Logon failed, use ctrl+c to cancel basic credential prompt.

Username for 'https://github.com': zetingzhu

Password for 'https://zetingzhu@github.com':

remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.

remote: Please see https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/ for more information.

fatal: Authentication failed for 'https://github.com/zetingzhu/ZT_KLineChart.git/'

解决方法

```
https://github.com/zetingzhu/ZT_KLineChart
```

### Your local changes to the following files would be overwritten by merge:

```
git stash
git pull origin master
git stash pop
```
