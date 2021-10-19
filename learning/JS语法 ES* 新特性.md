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

![image](https://user-images.githubusercontent.com/31462942/137841147-f3466129-c651-4e8c-a1be-bf624a0ffd5a.png)


<br />

#### 正则表达式反向断言

```js
(?=p)、(?<=p)  p 前面(位置)、p 后面(位置)
(?!p)、(?<!p>) 除了 p 前面(位置)、除了 p 后面(位置)
```

<br />

#### 正则表达式dotAll模式

> 正则表达式中点.匹配除回车外的任何单字符，标记s改变这种行为，允许行终止符的出现

```js
/hello.world/.test('hello\nworld'); // false
```

<br />

<br />

## ES10（2019）

#### Array.flat() 和 Array.flatMap()

flat()

```js
[1, 2, [3, 4]].flat(Infinity); // [1, 2, 3, 4]
```

flatMap()

```js
[1, 2, 3, 4].flatMap(a => [a**2]); // [1, 4, 9, 16]
```

<br />

#### String.trimStart()和 String.trimEnd()

去除字符串首尾空白字符

<br />

#### String.prototype.matchAll

> matchAll（）为所有匹配的匹配对象返回一个迭代器

```js
const raw_arr = 'test1  test2  test3'.matchAll((/t(e)(st(\d?))/g));
const arr = [...raw_arr];
```

<br />

#### Symbol.prototype.description

> 只读属性，回 Symbol 对象的可选描述的字符串。

```js
Symbol('description').description; // 'description'
```

<br />

#### Object.fromEntries()

> 返回一个给定对象自身可枚举属性的键值对数组

```js
// 通过 Object.fromEntries， 可以将 Map 转化为 Object:
const map = new Map([ ['foo', 'bar'], ['baz', 42] ]);
console.log(Object.fromEntries(map)); // { foo: "bar", baz: 42 }
```

<br />

#### 可选 Catch

<br />

<br />

## ES11（2020）

#### Nullish coalescing Operator(空值处理)

表达式在 ?? 的左侧 运算符求值为undefined或null，返回其右侧

```js
let user = {
    u1: 0,
    u2: false,
    u3: null,
    u4: undefined
    u5: '',
}
let u2 = user.u2 ?? '用户2'  // false
let u3 = user.u3 ?? '用户3'  // 用户3
let u4 = user.u4 ?? '用户4'  // 用户4
let u5 = user.u5 ?? '用户5'  // ''
```

<br />

#### Optional chaining（可选链）

?.用户检测不确定的中间节点

```js
let user = {}
let u1 = user.childer.name // TypeError: Cannot read property 'name' of undefined
let u1 = user.childer?.name // undefined
```

<br />

#### Promise.allSettled

> 返回一个在所有给定的promise已被决议或被拒绝后决议的promise，并带有一个对象数组，每个对象表示对应的promise结果

```js
const promise1 = Promise.resolve(3);
const promise2 = 42;
const promise3 = new Promise((resolve, reject) => reject('我是失败的Promise_1'));
const promise4 = new Promise((resolve, reject) => reject('我是失败的Promise_2'));
const promiseList = [promise1,promise2,promise3, promise4]
Promise.allSettled(promiseList)
.then(values=>{
  console.log(values)
});
```

![image](https://user-images.githubusercontent.com/31462942/137853361-1d4dccd0-2417-475b-91e6-8b0eee509203.png)

<br />

#### import()

按需导入

<br />

#### 新基本数据类型BigInt

任意精度的整数

<br />

#### globalThis

* 浏览器：window
* worker：self
* node：global

<br />

<br />

## ES12（2021）

#### replaceAll

> 返回一个全新的字符串，所有符合匹配规则的字符都将被替换掉

```js
const str = 'hello world';
str.replaceAll('l', ''); // "heo word"
```

<br />

#### Promise.any

> Promise.any() 接收一个Promise可迭代对象，只要其中的一个 promise 成功，就返回那个已经成功的 promise 。如果可迭代对象中没有一个 promise 成功（即所有的 promises 都失败/拒绝），就返回一个失败的 promise

```js
const promise1 = new Promise((resolve, reject) => reject('我是失败的Promise_1'));
const promise2 = new Promise((resolve, reject) => reject('我是失败的Promise_2'));
const promiseList = [promise1, promise2];
Promise.any(promiseList)
.then(values=>{
  console.log(values);
})
.catch(e=>{
  console.log(e);
});
```

![image](https://user-images.githubusercontent.com/31462942/137853827-72df0a9b-816c-462a-bf92-ff85f71e3c3e.png)

<br />

#### WeakRefs

> 使用WeakRefs的Class类创建对对象的弱引用(对对象的弱引用是指当该对象应该被GC回收时不会阻止GC的回收行为)

<br />

#### 逻辑运算符和赋值表达式

> 逻辑运算符和赋值表达式，新特性结合了逻辑运算符（&&，||，??）和赋值表达式而JavaScript已存在的 复合赋值运算符有：

```js
a ||= b
//等价于
a = a || (a = b)

a &&= b
//等价于
a = a && (a = b)

a ??= b
//等价于
a = a ?? (a = b)
```

<br />

#### 数字分隔符

数字分隔符，可以在数字之间创建可视化分隔符，通过_下划线来分割数字，使数字更具可读性

```js
const money = 1_000_000_000;
//等价于
const money = 1000000000;

1_000_000_000 === 1000000000; // true
```

![image](https://user-images.githubusercontent.com/31462942/137854026-14d3d031-7407-48d3-97da-01c86bd27c8c.png)

