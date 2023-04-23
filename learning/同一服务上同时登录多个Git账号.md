## 前言

有时候我们可能需要在同一台电脑上针对不同平台、或同一平台同时使用多个Git账户的情况，这时候我们就需要针对多个平台和账户进行不同的设置， 同时管理多个SSH key

> 这里使用Mac环境举例

<br />

## 操作步骤

#### 生成key

生成多个SSH key，这里使用one、two两个账户进行举例

注意： 在生成多个SSH key的时候一定要在`~/.ssh`目录下进行，否则生成的SSH key不会在`~/.ssh`目录下，所以以下有操作都是在`~/.ssh`目录下进行的

> 在生成之前尽量删除此目录下的所有文件再进行，以免出现不必要的问题
> 如果自信可以没必要进行此操作，比如我是在原有的账号下新增一个账号，我就没对原账号进行改动

```nginx
// git注册邮箱 one
ssh-keygen -t rsa -C "one@email.com"

// git注册邮箱 two
ssh-keygen -t rsa -C "two@email.com"
```

注意，这里执行生成key的时候，要对生成的key文件重新命名

复制代码再输入命令行的时候在第一次提示`Enter file in which to save the key`的时候对ssh文件进行重命名



















