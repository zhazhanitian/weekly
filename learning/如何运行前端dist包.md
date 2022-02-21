## 前言

很多时候我们build前端项目后，部署到线上后发现有bug，和测试环境对不上，或由于缓存等什么原因，导致达不到预期效果，就想要验证一下dist包是否有问题，基此记录一下如果本地运行dist包

<br />

<br />

## 步骤

#### 全局安装express-generator生成器

```nginx
npm install express-generator -g 
```

<br />

#### 创建一个express项目

```nginx
express 项目名
```

<br />

#### 进入项目目录，安装相关项目依赖

```nginx
cd 项目名

cnpm i
```

<br />

#### 运行

```nginx
将 build 后生成的 dist 文件夹下的所有文件复制到 express 项目的 publick 文件夹下面，然后运行 npm start 来启动 express 项目
```

<br />

#### 查看

```nginx
打开浏览器，输入localhost:3000就可以看到效果了

备注：这个3000是自己设置的端口号，如果设置888，那么就要输入localhost:888
打开配置文件./bin/www设置可设置 ip 和端口
```

