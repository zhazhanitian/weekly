## 前言

学习前端开发时，几乎最先学习的就是console.log()

毕竟多数人的第一行代码都是：console.log('Hello World')

console对象提供了对于浏览器调试控制台的访问，可以从任何全局对象中访问到console对象

灵活运用console对象所提供的方法，可以让开发变得更简单

<br />

<br />

## 常规运用

```nginx
console.log()– 打印内容的通用方法
console.info()– 打印资讯类说明信息
console.debug()– 在控制台打印一条 "debug" 级别的消息
console.warn()– 打印一个警告信息
console.error()– 打印一条错误信息
```

<img src="https://user-images.githubusercontent.com/31462942/156098274-7aa600d8-4fa8-4540-923d-681f3b27122c.png"  style="float:left;width: 600px" />

<br />

#### console.log()写css

<img src="https://user-images.githubusercontent.com/31462942/156098477-726259bc-b13b-419f-b2e0-9210581caa37.png"  style="float:left;width: 600px" />

<br />

#### console.log() 使用参数

<img src="https://user-images.githubusercontent.com/31462942/156098635-3034887e-88d7-4f4b-b1d9-632964cf3e75.png"  style="float:left;width: 600px" />

<br />

#### console.clear()

用于清除控制台信息

<img src="https://user-images.githubusercontent.com/31462942/156098746-c1108965-255f-4940-882b-ee22792abec6.png"  style="float:left;width: 600px" />

<br />

#### console.count(label)

输出count()被调用的次数，可以使用一个参数label。演示如下

```js
var user = "";

function greet() {
  console.count(user);
  return "hi " + user;
}

user = "bob";
greet();
user = "alice";
greet();
greet();
console.count("alice");
```

输出

<img src="https://user-images.githubusercontent.com/31462942/156098861-9bc651d3-8ade-4101-8348-647e34556895.png"  style="float:left;width: 600px" />

<br />

#### console.dir()

使用console.dir()可以打印对象的属性，在控制台中逐级查看对象的详细信息

<img src="https://user-images.githubusercontent.com/31462942/156099013-91b95241-4902-4052-9a95-2611e454e0d1.png"  style="float:left;width: 600px" />

<br />

#### console.memory

console.memory是一个属性，而不是方法，使用memory属性用来检查内存信息

<img src="https://user-images.githubusercontent.com/31462942/156099175-42693eac-90f3-4080-a420-453e053e6874.png"  style="float:left;width: 600px" />

<br />

#### console.time() 和 console.timeEnd()

* console.time()– 使用输入参数的名称启动计时器。在给定页面上最多可以同时运行 10,000 个计时器
* console.timeEnd()– 停止指定的计时器并记录自启动以来经过的时间（以毫秒为单位）

<img src="https://user-images.githubusercontent.com/31462942/156099294-bca1381b-bb40-4775-a17f-5ea77b589391.png"  style="float:left;width: 600px" />

<br />

#### console.assert()

如果断言为假，将错误信息写入控制台，如果为真，无显示

<img src="https://user-images.githubusercontent.com/31462942/156099397-8cbb41d0-7418-4ca7-b072-efc816266bd7.png"  style="float:left;width: 600px" />

<br />

#### console.trace()

console.trace()方法将堆栈跟踪输出到控制台

<img src="https://user-images.githubusercontent.com/31462942/156099502-1ef8a75c-952b-43a2-a441-779c95759d2d.png"  style="float:left;width: 600px" />

<br />

#### console.group() 和 console.groupEnd()

在控制台上创建一个新的分组，随后输出到控制台上的内容都会被添加到一个锁进，表示该内容属于当前分组，知道调用console.groupEnd()之后，当前分组结束

<img src="https://user-images.githubusercontent.com/31462942/156099621-13b74e8f-a765-41e9-b495-ea717522db86.png"  style="float:left;width: 600px" />

