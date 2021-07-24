## 序言

在git使用过程中发现指令实在太多，就算记忆后不长用的话很快也会忘记掉，所以编写本文的初衷是为了记录在使用git指令的过程中所遇到的需求与解决方法，毕竟使用git的需求也就那么一些，范围不大，所以可以将需求与解决方法记录下来，下次使用时遇到相同需求如果忘记了也可以得到快速解决

这里主要分三块进行记录

* [需求区](#需求区)
* [问题区](#问题区)
* [日常工作操作流程](#日常工作操作流程)
* [扩展](#扩展)

<br >

<br >

## <a id="需求区">需求区</a>

### 需求1

#### 描述：

Windows系统，在git中生产SSH key

#### 扩展：

由于本地Git仓库和GitHub仓库之间的传输是通过SSH加密的，所以必须要让github仓库认证你SSH key，在此之前，必须要生成SSH key。

#### 步骤：

第1步：创建SSH Key。在windows下查看`[c盘->用户->自己的用户名->.ssh]`下是否有`id_rsa`、`id_rsa.pub`文件，如果没有需要手动生成。
打开git bash，在控制台中输入以下命令。

```python
ssh-keygen -t rsa -C "guimashusheng@126.com"
```

密钥类型可以用 -t 选项指定。如果没有指定则默认生成用于SSH-2的RSA密钥。这里使用的是rsa。同时在密钥中有一个注释字段，用-C来指定所指定的注释，可以方便用户标识这个密钥，指出密钥的用途或其他有用的信息。所以在这里输入自己的邮箱或者其他都行。输入完毕后程序同时要求输入一个密语字符串(passphrase)，空表示没有密语。接着会让输入2次口令(password)，空表示没有口令。3次回车即可完成当前步骤，此时`[c盘>用户>自己的用户名>.ssh]`目录下已经生成好了。

第2步：登录github。打开setting->SSH keys，点击右上角 New SSH key，把生成好的公钥`id_rsa.pub`放进 key输入框中，再为当前的key起一个title来区分每个key。

<br >

### 需求2

#### 描述

将远程仓库上的文件同步到本地

#### 步骤

```python
git clone git@txxx.git
```

<br >

### 需求3

#### 描述

将本地文件push到远程仓库上

#### 步骤

进入到需要提交的文件所在目录，输入如下指令：

```python
# add后跟空格加一点会将该目录下的所有文件添加至上传列表，若需上传指定文件只需将点改为指定文件的名称带后缀
git add .      
 
git commit -m "提交描述"
 
# 如果文件所在的本地仓库和远程仓库从未连接过需要输入下面语句将本地与远程联系上
git remote add origin git@xxx.git
 
# 将文件提交到主枝上
git push -u origin master
```

<br >

### 需求4

#### 描述

为版本打tag

#### 步骤

进入到项目根目录，输入如下指令

```python
# 查看历史tag
git tag

# 查看git的相关tag操作
git tag -h

# 添加版本tag  其中v1.0.0为tag名称
git tag -a v1.0.0  -m "版本tag描述"

# 将本地tag推送到远程上  其中的v1.0.0就是上面的tag名称
git push origin v1.0.0
```

<br >

### 需求5

#### 描述

　　在网页上面新建项目后，完成初始化操作

#### 步骤

　　进入到需要提交的文件所在目录，输入如下指令：

```python
# 下载文件到本地
git clone git@xxx.git

# 进入文件目录
cd fileStorage

# 新建README文件
touch README.md

# 添加README文件到提交列表
git add README.md

# 提交说明
git commit -m "add README"

# 提交文件到主枝
git push -u origin master
```

<br >

### 需求6

#### 描述

　　本地已有相关文件夹，在网页上面新建项目后，将本地文件推送到服务器

#### 步骤

　　进入到需要提交的文件所在目录，输入如下指令：

```python
# 进入已有的本地文件目录
cd existing_folder

# 初始化git到本目录
git init

# 本地与远程取得联系
git remote add origin git@xxx.git

# 添加文件的推送列表
git add .

# 提交说明
git commit -m "Initial commit"

# 推送到主枝
git push -u origin master
```

<br >

### 需求7

#### 描述

　　预览打包项目（gh-pages）

#### 步骤

　　进入到需要提交的文件所在目录，输入如下指令：

```python
# 进入项目目录 Emoji-ChatRoom，注意这里的项目名称得和远程相同，不同的话反正我是没有上传成功过
cd Emoji-ChatRoom

# 初始化git到本目录
git init

# 取消dist文件过滤，.gitignore文件中删除dist(重要步骤)

# 本地与远程取得联系
git remote add origin https://github.com/zhazhanitian/Emoji-ChatRoom.git

# 添加文件的推送列表
git add -A

# 提交说明
git commit -m "init file"

# 推送到主枝
git push -u origin master

# 把 dist 目录推送到 gh-pages 分支
git subtree push --prefix dist origin gh-pages

# 然后通过 https://zhazhanitian.github.io/Emoji-ChatRoom 就可以访问了
```

<br >

### 需求8

#### 描述

​    查看本地分支、查看远程分支、新建本地分支、推送本地分支到远程、切换分支、删除本地分支、删除远程分支

#### 步骤

​    进入到项目根目录，输入如下指令

```python
# 查看本地分支
git branch

# 查看所有分支，包括远程，remotes为远程分支
git branch -a

# 新建本地分支
git branch newbranch

# 新建本地分支并切换
git checkout -b newbranch

# 推送本地分支到远程
git push origin newbranch

# 切换分支
git checkout master

# 删除本地分支 (要切换出要删除的分支才可以删除)
git branch -d newbranch

# 删除远程分支 方式1
git push origin --delete newbranch

# 删除远程分支 方式2
git push origin :newbranch
```

<br >

### 需求9

#### 描述

　　fetch更新本地仓库（更新远程代码到本地）

#### 步骤

　　进入到需要提交的文件所在目录，输入如下指令：

```python
# 方法一

# 从远程的 origin 仓库的 master 分支下载代码到本地的 origin master
git fetch origin master 

# 比较本地的仓库和远程参考的区别
git log -p master.. origin/master

# 把远程下载下来的代码合并到本地仓库，远程的和本地的合并
git merge origin/master


# 方法二

# 从远程的 origin 仓库的 master 分支下载到本地并新建一个分支 temp
git fetch origin master:temp 

# 比较 master 分支和 temp 分支的不同
git diff temp 

# 合并 temp 分支到 master 分支
git merge temp

# 删除 temp
git branch -d temp
```

<br >

### 需求10

#### 描述

​    开发中经常在错误的分支上改了大量代码，需要转移到正确到分支上

#### 步骤

​    进入到项目根目录，输入如下指令：

```python
# 将当前所以修改提交的本地一个储存库，使用此命令后当前分支将会是修改之前的状态
git stash

# 切换分支，通过如下命令将储存库的代码拉取下来
git checkout dev
git stash apply

# 这个命令也可以实现，和上面不同的是会自动删除本地储存，不过出现冲突则不会删除
# git stash pop 

# 查看本地储存
git stash list

# 删除本地储存
git stash drop
```

<br >

### 需求11

#### 描述

​    回滚代码到某次commit，开发过程中可能合并到不需要到分支，或个别原因需要回退

#### 步骤

​    进入到项目根目录，输入如下指令：

```python
# 回退到上个版本
git reset --hard HEAD^

# 回退到前3次提交之前，以此类推，回退到n次提交之前
git reset --hard HEAD~3

# 退到/进到 指定commit的sha码
git reset --hard commit_id

# 强推到远程 HEAD可以换为具体分支名
git push origin HEAD --force
```

<br >

### 需求12

#### 描述

​    在我们工作中，经常会需要建立很多开发分支，但是过一段时间自己都不记得那个分支是干什么的了，这里提供给git分支添加备注功能，方便管理

#### 步骤

​    进入到项目根目录，输入如下指令：

```python
# branch_name 是需要加备注的分支名
git config branch.branch_name.description "这里添加注释"

# 例如给 dev 分支加备注
git config branch.dev.description "家花不如野花香~"

# 查看 dev 分支备注
git config branch.dev.description

# 借助工具 git-br 查看分支备注
npm i -g git-br

# 查看所有分支备注
git br
```

<br >

### 需求13

#### 描述

​    Mac SSH证书配置

#### 步骤

​    打开命令行，输入如下指令：

```python
# 步骤一：检查是否已经存在SSH Key
ls -al ~/.ssh 

# 步骤二：生成/设置SSH Key

# 情况一：终端出现文件id_rsa.pub 或 id_dsa.pub，则表示该电脑已经存在SSH Key，此时可继续输入命令：
pbcopy < ~/.ssh/id_rsa.pub 

# 这样你需要的SSH Key 就已经复制到粘贴板上了，然后进行步骤3 

# 情况二：终端未出现id_rsa.pub 或 id_dsa.pub文件，表示该电脑还没有配置SSH Key，此时需要输入命令：
ssh-keygen -t rsa -C 'your_email@example.com'

# 然后一路回车(-C 参数是你的邮箱地址)
# 此时再输入命令：ls -al ~/.ssh 就会出现id_rsa.pub 和 id_dsa.pub两个文件，然后重复情况一的步骤即输入以下命令
pbcopy < ~/.ssh/id_rsa.pub

# 步骤三：将SSH Key添加到GitLab中
# 打开GitLab, 登录，找到左边栏有一个🔑的按钮，点击“ADD SSH KEY”按钮添加，将已经获得的SSH Key粘贴到“Key”
# 下边的标题可以随便取，点击加入项目，这样就保持了本地与服务器端的联系
"🔑"已替换为 Profile Setting里的"SSH Key"
```

<br >

### 需求14

#### 描述

​	git 本地分支关联远程分支

#### 步骤

​    进入到项目根目录，输入如下指令：

```python
# 远程有对应分支 remote_branch：远程分支名  local_branch：本地分支名
git branch --set-upstream-to=origin/remote_branch local_branch

# 远程无对应的分支
# 新建一个本地的分支 newbranch
git checkout -b newbranch

# 新建一个远程分支
git push origin newbranch:newbranch

# 把本地的新分支，和远程的新分支关联
git push --set-upstream origin newbranch
```

<br >

### 需求15

#### 描述

​	强制切换到本地某个分支

#### 步骤

​    进入到项目根目录，输入如下指令：

```python
# 如果要保留本地的修改
git status
git add .
git commit -m "备注信息"
git checkout newBranch

# 强制切换，即放弃本地修改
git checkout -f newBranch
```

<br >

### 需求16

#### 描述

​	将一分支的某次提交合并到另一分支，比如在develop分支需要合并test分支的某次提交

#### 步骤

​    进入到项目根目录，输入如下指令：

```python
# 切到test分支
git checkout test

# 查看commitId并复制出来
git log

#切换到develop分支
git checkout develop

# 合并需要的提交
git cherry-pick '需要合并的commitId'
```

<br >

### 需求17

#### 描述

​	查看当前分支是从什么分支切出来的 

#### 步骤

​    进入到项目根目录，输入如下指令：

```python
# 先切换到需要查看的分支synchronizeData
git checkout synchronizeData

# 查看当前分支合并情况
git reflog --date=local | grep {分支名称}

# 例
git reflog --date=local | grep synchronizeData

# 输出结果
262782e3 HEAD@{Thu Sep 24 10:01:22 2020}: checkout: moving from auto-master to synchronizeData
d236c6a7 HEAD@{Mon Sep 21 15:27:16 2020}: checkout: moving from synchronizeData to develop
5bced015 HEAD@{Tue Sep 15 10:48:52 2020}: checkout: moving from auto-master to synchronizeData
```

<br >

### 需求18

#### 描述

​	去除 rsa 私钥密码保护

#### 背景

​	之前在配置Github时候，给本地生成的ssh-key加上了访问密码，因为所有ssh连接均采用了同一个key，给之后访问带来了很多的不便

#### 步骤

​    进入到项目根目录，输入如下指令：

```js
// 终端输入，然后接连两次回车
ssh-keygen -p

// 输入旧密码，再接着两次回车，出现如下提示则修改成功
Your identification has been saved with the new passphrase
```

<br >

### 需求19

#### 描述

​	比较两次修改的差异（git diff）

#### 步骤

​    进入到项目根目录，输入如下指令：

```python
# 比较工作区与暂存区
git diff 不加参数即默认比较工作区与暂存区

# 比较暂存区与最新本地版本库（本地库中最近一次commit的内容）
git diff --cached  [<path>...] 

# 比较工作区与最新本地版本库，如果HEAD指向的是master分支，那么HEAD还可以换成master
git diff HEAD [<path>...]

# 比较工作区与指定commit-id的差异
git diff commit-id  [<path>...] 

# 比较暂存区与指定commit-id的差异
git diff --cached [<commit-id>] [<path>...] 

# 比较两个commit-id之间的差异
git diff [<commit-id>] [<commit-id>]
```

<br >

### 需求20

#### 描述

​	从源项目更新代码到自己fork过来的项目

#### 背景

​	在GitHub上我们会去fork别人的一个项目，这就在自己的Github上生成了一个与原作者项目互不影响的副本，自己可以将自己Github上的这个项目再clone到本地进行修改，修改后再push，只有自己Github上的项目会发生改变，而原作者项目并不会受影响，避免了原作者项目被污染。但经过一段时间， 有可能作者原来的代码变化很大， 你想接着在他最新的代码上修改， 这时你需要合并原作者的最新代码过来， 让你的项目变成最新的

#### 步骤

​    进入到项目根目录，输入如下指令：

```python
# 先克隆项目到本地： 
git clone https://github.com/iakuf/mojo 
cd mojo

#添加原作者项目的 remote 地址， 然后将代码 fetch 过来 
git remote add sri https://github.com/kraih/mojo

#‘sri’相当于一个别名
git fetch sri

# 查看本地项目目录： git remote -v

# 合并 
git checkout master 
git merge sri/master 

# 如果有冲突的话，需要丢掉本地分支： 
git reset –hard sri/master 

#这时你的当前本地的项目变成和原作者的主项目一样了，可以把它提交到你的GitHub库 
git commit -am ‘更新到原作者的主分支’ 
git push origin 
git push -u origin master -f –强制提交
```

<br >

### 需求21

#### 描述

​	切换远程仓库地址

#### 步骤

​    进入到项目根目录，输入如下指令：

```python
# 方式一：直接修改仓库地址
git remote set-url origin url

# 方式二：删除本地远程仓库地址，然后添加新的仓库地址
git remote rm origin
git remote add origin url

# 方式三：修改配置文件
# 每个仓库在初始化时，都会有一个 .git 的隐藏目录，其中有个 config 文件，内容如下
# 直接将 [remote "origin"] 下的 url 地址替换即可
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
	ignorecase = true
	precomposeunicode = true
[remote "origin"]
	url = https://github.com/zhazhanitian/weekly.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "main"]
	remote = origin
	merge = refs/heads/main

# 修改完成查看远程仓库地址
git remote -v
```

<br />

### <a id="需求22">需求22</a>

#### 描述

在我们开发完成一块业务时，需要将自己的开发分支合并到 master 分支进行发布上线，这时最好就走 PR 模式，便于代码回退

> PR 会将需要合并到 master 的分支上的所有提交合并为一个提交记录合并到 master 分支上，如果后续因为某种原因需要将本次合并的分支代码都回退掉，就会很方便

#### 步骤

* **GitLab**

  <img src="https://qiniu-image.qtshe.com/0DEA837B85434.png" width="800px" style="float:left;" />

  <br />

  <img src="https://qiniu-image.qtshe.com/1DEA837B85434.png" width="800px" style="float:left;" />

  <br />

  <img src="https://qiniu-image.qtshe.com/2DEA837B85434.png" width="800px" style="float:left;" />

  <br />

* **GitHub**

  <img src="https://qiniu-image.qtshe.com/2278B8A318EB4.png" width="800px" style="float:left;" />

  <img src="https://qiniu-image.qtshe.com/18EC87083DB2E.png" width="800px" style="float:left;" />

  <br />

当然发起的合并申请可能会存在冲突代码，这时需要先关闭本次合并申请，将 master 分支代码合并到自己的开发分支后再次发起 PR

<br />

<br />

## <a id="问题区">问题区</a>

### 问题1

#### 描述

　　The file will have its original line endings in your working directory.

#### 原因

　　原因不一，大体是因为使用拷贝的方式迁移文件或者从别人的仓库下载文件后想要上传到自己的仓库而出现的问题

#### 步骤

　　出现上述bug后，接着输入如下指令：

```python
# 删除缓存
git rm -r --cached .

# 解决系统不同的冲 
git config core.autocrlf false

# 接着重新执行上传操作
git add .

...
```

<br />

### 问题2

#### 描述

　　git 在pull或者合并分支的时候有时会遇到提示：Please enter a commit message to explain why this merge is necessary

#### 原因

　　原因不详

#### 步骤

1. 按键盘字母 i 进入insert模式
2. 按键盘左上角"Esc"
3. 输入":wq",进行修改后保存退出,然后按回车键即可

<br />

### 问题3

#### 描述

在执行删除远程分支的时候报错如下

```nginx
git push origin --delete digitalEconomy

# error: unable to delete 'digitalEconomy': remote ref does not exist
# error: failed to push some refs to '**/digital_reform/digital_mobile.git'
```

#### 原因

git pull 的时候会执行 git fetch 更新本地 origin 镜像，同时通过 git merge 合并代码
但是并不会同步镜像的分支信息。所以导致分支信息的同步滞后

#### 解决方案

```nginx
# 查看remote地址、远程分支和本地分支与之相对应关系等信息。
git remote show origin

# 按照提示更新镜像，尾后跟的是仓库远程地址
git remote prune 'xx/digital_reform/digital_mobile.git'
```

结论：没啥效果，再使用以下方式尝试解决

```nginx
# 更新远程镜像到本地
git fetch --prune origin

# 删除成功
From http://10.27.167.84:8666/digital_reform/digital_mobile
- [deleted]         (none)     -> origin/digitalEconomy
```

结果：成功

<br />

### 问题4

#### 描述

在进行开源项目开发过程中，需要更新源项目到自己fock过来的项目中，git 报错如下

```nginx
# fetch 报错
➜  element-plus git:(dev) git fetch element 
ssh: connect to host github.com port 22: Operation timed out
fatal: Could not read from remote repository.
```

#### 原因

端口22不能使用

#### 步骤

1. 端口22为ssh默认端口，怀疑和github服务器有关

   ```nginx
   # 测试 443 端口是否可以使用
   ssh -T -p 443 git@ssh.github.com
   
   # 过程中输入一次 yes，得到最后一行信息则表示 443 端口可用
   ➜  element-plus git:(dev) ssh -T -p 443 git@ssh.github.com
   The authenticity of host '[ssh.github.com]:443 ([18.138.202.180]:443)' can't be established.
   RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
   Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
   Warning: Permanently added '[ssh.github.com]:443,[18.138.202.180]:443' (RSA) to the list of known hosts.
   Hi zhazhanitian! You've successfully authenticated, but GitHub does not provide shell access.
   ```

2. 将修改端口为 443

   ```nginx
   # 切换到 ssh 目录
   cd ~/.ssh
   
   # 修改ssh配置，编辑 config 文件(没有则新增)
   vim config
   
   # 修改配置信息，将以下信息添加到 config 内容后面
   Host github.com /* 服务器地址为github地址 */
   User guimashusheng@126.com /* github上的注册邮箱为用户账号 只需要改这里 */
   Hostname ssh.github.com /* 服务器地址为github地址 */
   PreferredAuthentications publickey /* 采用公匙 */
   IdentityFile ~/.ssh/id_rsa /* 公匙文件路径 */
   Port 443 /* 修改端口为443 */
   ```

3. 再次尝试

   ```nginx
   # 测试 443 端口
   ➜  element-plus git:(dev) ssh -T -p 443 git@ssh.github.com
   Hi zhazhanitian! You've successfully authenticated, but GitHub does not provide shell access.
   
   # fetch 内容
   ➜  element-plus git:(dev) git fetch element               
   remote: Enumerating objects: 5699, done.
   remote: Counting objects: 100% (4062/4062), done.
   remote: Compressing objects: 100% (830/830), done.
   remote: Total 5699 (delta 3508), reused 3623 (delta 3208), pack-reused 1637
   Receiving objects: 100% (5699/5699), 9.08 MiB | 2.61 MiB/s, done.
   Resolving deltas: 100% (4568/4568), completed with 432 local objects.
   ```

4. 就此解决

<br />

<br />

## <a id="日常工作操作流程">日常工作操作流程</a>

### 新增项目

#### 描述

日常工作过程中难免需要我们创建新项目，并建立相应的 git 远程仓库，这里将流程一一记录一下

#### 演示工具

GitHub、Vue

#### 步骤

1. 创建 Vue 新项目

   ```nginx
   vue create verification-process
   ```

2. 进入项目，完成 git 初始化

   ```nginx
   cd verification-process
   
   git init
   ```

3. 创建远程仓库项目

   <img src="https://qiniu-image.qtshe.com/AD56D6F0FFB81E.png" width="800px" style="float:left;" />

   <br />

4. 将本地项目于远程仓库建立关联

   <img src="https://qiniu-image.qtshe.com/7DB08D0E2AEA8.png" width="800px" style="float:left;" />

   ```nginx
   # 用远程仓库获取的仓库地址进行建联
   git remote add origin git@github.com:zhazhanitian/verification-process.git
   
   # 查看是否关联成功
   git remote -v
   
   # 成功效果
   origin	git@github.com:zhazhanitian/verification-process.git (fetch)
   origin	git@github.com:zhazhanitian/verification-process.git (push)
   ```

<br />

### 开发新业务

#### 描述

在已有项目中进行新业务开发时，需要新建分支，开发完成后进行合并操作

#### 演示工具

GitHub、Vue

#### 步骤

1. 从master分支新切分支（newBusiness）

   ```nginx
   git checkout -b newBusiness
   ```

   > 需要注意的是，如果你要开发的业务依赖于别人正在开发的业务，那么开发前需沟通好，划清业务范围，避免出现重复劳动以及大量代码冲突，可以选择在同一个分支进行开发或协商责任范围等方式

2. 新增远程分支并与本地建立关联

   ```nginx
   git push --set-upstream origin newBusiness
   ```

3. 接着进行开发，但是过程中需要根据实际情况进行区分操作

   如果属于多业务协作，比如你在自己的 newBusiness 分支开发业务，过程中 B 同学开发的业务合并到了 master 分支，这里就要进行判断

   * 无牵扯

     如果你开发的业务不依赖 B 同学开发的业务或 B 同学的代码和自己的完全不相关、无牵扯，那就切记不要把master 分支代码合并到自己的分支（如果合并到了自己的分支，后面对分支做 code review 的时候无法区分开那些是你写的业务代码）

   * 有重叠

     如果你开发的业务代码和 B 同学开发的业务代码有交际，那就选择恰当的实际将 master 分支合并到自己的分支上

   ```nginx
   # 开发过程中要保持良好的提交习惯，开发到某个节点时就进行一次提交，并简单明了的写一下提交日志，不要一天或几天才提交一次，尽可能避免多人合作时出现大面积冲突或重复劳作，操作命令如下
   
   # 先将自己的代码添加到暂存区
   git add .
   git commit -m "提交日志"
   
   # 拉取最新代码
   git pull
   
   # 如果存在冲突，需要处理完冲突，再次进行提交
   git commit -am "解决冲突"
   git push
   
   # 如果不存在冲突，直接提交到远程
   git push
   ```

4. 开发完毕，要合并到master分支时

   * 如果开发周期较长或提交记录较多，最好选择走 PR 形式

     > PR 会将需要合并到 master 的分支上的所有提交合并为一个提交记录合并到 master 分支上，这样如果合并分支代码存在问题的话方便回退

     参考：[发起 PR （合并请求）](#需求22)

   * 如果改动不多或提交记录就一个，并且有 master 分支更改权限，可直接使用命令合并

     ```nginx
     # 从开发分支切换到 master 分支
     git checkout master
     
     # 拉取 master 分支最新代码
     git pull
     
     # 合并开发分支代码到 master 分支
     git merge newBusiness
     
     # 如果不存在冲突，可直接提交到远程，则合并完毕
     git push
     
     # 如果存在冲突，需要处理完冲突，再次进行提交，合并完毕
     git commit -am "解决冲突"
     git push
     ```

<br />

<br />

## <a id="扩展">扩展</a>

#### 知识1

```python
# git add上传本地项目所有变化的命令三种有
    git add -A
    git add -u
    git add .

git add -A 提交所有变化
 
git add -u 提交被修改(modified)和被删除(deleted)文件，不包括新文件(new)
 
git add . 提交新文件(new)和被修改(modified)文件，不包括被删除(deleted)文件
```
