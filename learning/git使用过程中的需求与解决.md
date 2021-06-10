### 序言

　　在git使用过程中发现指令实在太多，就算记忆后不长用的话很快也会忘记掉，所以编写本文的初衷是为了记录在使用git指令的过程中所遇到的需求与解决方法，毕竟使用git的需求也就那么一些，范围不大，所以可以将需求与解决方法记录下来，下次使用时遇到相同需求如果忘记了也可以得到快速解决

<br >

<br >

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

<br >

<br >

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

<br >

### 问题2

#### 描述

　　git 在pull或者合并分支的时候有时会遇到提示：Please enter a commit message to explain why this merge is necessary

#### 原因

　　原因不详

#### 步骤

1. 按键盘字母 i 进入insert模式
2. 按键盘左上角"Esc"
3. 输入":wq",进行修改后保存退出,然后按回车键即可

<br >

### 补充

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
