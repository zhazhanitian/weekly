### 前言

本文主要对以下高级类型做举例介绍

* 联合类型
* keyof
* Record
* Partial (部分的; 不完全的)
* Required（必须的）
* Pick（选择）
* Readonly (意思是只读的)
* Exclude(排除)
* Omit (省略的)

<br />

<br />

### 联合类型

```js
/* 首先是联合类型的介绍 */
let a: string | number = '123' // 变量a的类型既可以是string，也可以是number
a = 123
```

<br />

### keyof

将一个类型的属性名全部提取出来当做联合类型

```js
// 1. 定义一个接口
interface Person {
  name: string
  age: number
}

type PersonKeys = keyof Person // 等同于 type PersonKeys = 'name' | 'age'

const p1: PersonKeys = 'name' // 可以
const p2: PersonKeys = 'age' // 可以
const p3: PersonKeys = 'height' // 不能将类型“"height"”分配给“name | age”
```

<br />

### Record

Record 用于属性映射，直接看案例就了然了

1. 定义一个普通的对象类型

























































