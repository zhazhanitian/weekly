## 前言

有时候我们可能需要在同一台电脑上针对不同平台、或同一平台同时使用多个Git账户的情况，这时候我们就需要针对多个平台和账户进行不同的设置， 同时管理多个SSH key

> 这里使用Mac环境举例

<br />

## 操作步骤

注意：我操作的背景是我已有一个GitHub账号，现在需要再登录另外一个GitHub账号操作另外的仓库代码

<br />

#### 生成新key

注意： 在生成多个SSH key的时候一定要在`~/.ssh`目录下进行，否则生成的SSH key不会在`~/.ssh`目录下，所以以下有操作都是在`~/.ssh`目录下进行的

> 在生成之前尽量删除此目录下的所有文件再进行，以免出现不必要的问题
> 如果自信可以没必要进行此操作，比如我是在原有的账号下新增一个账号，我就没对原账号进行改动

```nginx
// git注册邮箱
ssh-keygen -t rsa -C "yshij***@gmail.com"
```

注意，这里执行生成key的时候，要对生成的key文件重新命名

复制代码再输入命令行的时候在第一次提示`Enter file in which to save the key`的时候对ssh文件进行重命名

比如：我这里就对文件进行了重新命名为：id_ras_zook

<img width="673" style="float: left;" alt="image" src="https://user-images.githubusercontent.com/31462942/233816240-53463a36-e32c-4274-a722-bbd2f575cd97.png">

生成文件：

<img width="363" style="float: left;" alt="image" src="https://user-images.githubusercontent.com/31462942/233816290-45a040a8-feb6-4f64-a1a3-94b34d055c47.png">

<br />

#### 配置config文件

看我上面的生成文件图，有一个config文件，这个文件如果没有的话，需要自己生成，注意文件没有文件后缀

```js
// 生成文件命令
touch config
```

这样就会在`~/.ssh`目录下生成一个空的`config`文件，然后我们在文件中添加以下内容：

```js
// 原有github账号配置
Host github.com
  HostName github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa
  User gu*****g@126.com

// 新账号配置
Host zook.github.com
  HostName github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa_zook
  User ys****1@gmail.com
```

这里说明一下配置各字段的含义

```js
Host myhost（这里是自定义的host简称，以后连接远程服务器就可以用命令ssh myhost）
HostName 主机名可用ip也可以是域名(如:github.com或者bitbucket.org)
Port 服务器open-ssh端口（默认：22,默认时一般不写此行）
PreferredAuthentications   配置登录时用什么权限认证--可设为publickey,password publickey,keyboard-interactive等
IdentityFile 证书文件路径（如~/.ssh/id_rsa_*)
User 登录用户名(如：注册邮箱)
```

每个账号单独配置一个Host，每个Host要取一个别名，一般为每个Host主要配置HostName和IdentityFile两个属性，配置完保存即可

**Host 的名字可以自定义名字，不过这个会影响 git 相关命令，例如：Host mygithub 这样定义的话，使用命令 git clone git@mygithub:PopFisher/AndroidRotateAnim.git，git@后面紧跟的名字改为mygithub**

<br />

#### 部署 SSH key

查看key文件

```js
cat ~/.ssh/zook_id_rsa.pub
```

登陆 github 账号，进入Settings –> SSH and GPG keys，点击"new SSH key"， 把上面查到的公钥内容添加到 Github 账号中，其中 Title 为自定义的名字，Key 为 .pub 文件的内容，最后点击“ Add SSH key ”即可

<br >

#### 远程测试

```js
ssh -T git@zook.github.com
```

执行结果如下表示成功

<img width="754" style="float: left;" alt="image" src="https://user-images.githubusercontent.com/31462942/233816821-3d28b29f-1bde-4a15-ae5a-1bc422a845e6.png">

<br >

#### clone项目

**需要使用ssh方式clone**

```js
// 原来的写法
git clone git@github.com:sevendsss888/sd_h5.git

// 现在的写法
git clone git@zook.github.com:sevendsss888/sd_h5.git
```

<br >

#### 配置账号

一般我们会给全局设置Git账号和邮箱，所以在新的仓库下，给仓库设置局部用户名和邮箱就OK

```js
git config user.name "ysh****1"
git config user.email "yshi****1@gmail.com"
```

然后随便修改一点代码测试提交，提交成功那就至此结束

<img width="500" style="float: left;" alt="image" src="https://user-images.githubusercontent.com/31462942/233817178-0cdaddaa-9f8c-4003-be5c-af6c717b4913.png">

