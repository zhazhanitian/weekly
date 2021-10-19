## 前言

本文集合了 ES6 至 ES11 常用到的特性，包括还在规划的 ES12，只列举大概使用。使用部分新特性需要使用最新版的 bable 就行转义

<br />

<br />

## ES6（2015）

#### 类（class）

```js
class Man {
  constructor(name) {
    this.name = '小豪';
  }
  console() {
    console.log(this.name);
  }
}
const man = new Man('小豪');
man.console(); // 小豪
```

<br />

#### 模块化(ES Module)

```js
// 模块 A 导出一个方法
export const sub = (a, b) => a + b;
// 模块 B 导入使用
import { sub } from './A';
console.log(sub(1, 2)); // 3
```

<br />

#### 箭头（Arrow）函数

```js
const func = (a, b) => a + b;
func(1, 2); // 3
```

<br />

#### 函数参数默认值

```js
function foo(age = 25,){ // ...}
```

<br />

#### 模板字符串

```js
const name = '小豪';
const str = `Your name is ${name}`;
```

<br />

#### 解构赋值

```js
let a = 1, b= 2;
[a, b] = [b, a]; // a 2  b 1
```

<br />

#### 延展操作符

```js
let a = [...'hello world']; // ["h", "e", "l", "l", "o", " ", "w", "o", "r", "l", "d"]
```

<br />

#### 对象属性简写

```js
const name='小豪',
const obj = { name };
```

<br />

### Promise 

```js
Promise.resolve().then(() => { console.log(2); });
console.log(1);
// 先打印 1 ，再打印 2
```

<br />

#### let 和 const

```js
let name = '小豪'；
const arr = [];
```

<br />

<br />

## ES7（2016）

#### Array.prototype.includes()

```js
[1].includes(1); // true
```

<br />

#### 指数操作符

```js
2**10; // 1024
```

<br />

<br />

## ES8（2017）

#### async / await

异步终极解决方案

```js
async getData(){
    const res = await api.getTableData(); // await 异步任务
    // do something    
}
```

<br />

#### Object.values()

```js
Object.values({a: 1, b: 2, c: 3}); // [1, 2, 3]
```

<br />

#### Object.entries()

```js
Object.entries({a: 1, b: 2, c: 3}); // [["a", 1], ["b", 2], ["c", 3]]
```

<br />

#### String padding

```js
// padStart
'hello'.padStart(10); // "     hello"
// padEnd
'hello'.padEnd(10) "hello     "
```

<br />

#### 函数参数列表结尾允许逗号

<br />

#### Object.getOwnPropertyDescriptors()

> 获取一个对象的所有自身属性的描述符,如果没有任何自身属性，则返回空对象

<br />

### SharedArrayBuffer 对象 

> SharedArrayBuffer 对象用来表示一个通用的，固定长度的原始二进制数据缓冲区

```js
/**
 * 
 * @param {*} length 所创建的数组缓冲区的大小，以字节(byte)为单位。
 * @returns {SharedArrayBuffer} 一个大小指定的新 SharedArrayBuffer 对象。其内容被初始化为 0。
 */
new SharedArrayBuffer(10)
```

<br />

#### Atomics 对象

> Atomics 对象提供了一组静态方法用来对 SharedArrayBuffer 对象进行原子操作

<br />

<br />

## ES9（2018）

#### 异步迭代

await 可以和 for...of 循环一起使用，以串行的方式运行异步操作

```js
async function process(array) {
  for await (let i of array) {
    // doSomething(i);
  }
}
```

<br />

#### Promise.finally()

```js
Promise.resolve().then().catch(e => e).finally()
```

<br />

#### Rest / Spread 属性

对象或数组的展开运算符（...）

```js
const values = [1, 2, 3, 5, 6];
console.log( Math.max(...values) ); // 6
```

<br />

#### 正则表达式命名捕获组

```js
const reg = /(?<year>[0-9]{4})-(?<month>[0-9]{2})-(?<day>[0-9]{2})/;
const match = reg.exec('2021-02-23');
```

<br />

#### 

```js

```

<br />

#### 

```js

```

<br />

#### 

```js

```

<br />

#### 

```js

```























































