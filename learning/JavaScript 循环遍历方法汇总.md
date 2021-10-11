## 前言

<img src="https://qiniu-image.qtshe.com/1FA02E03F13A.png" style="float:left" width="700" />

<br />

## 数组遍历方法

### forEach()

 forEach  方法用于调用数组的每个元素，并将元素传递给回调函数。数组中的每个值都会调用回调函数。其语法如下

```js
array.forEach(function(currentValue, index, arr), thisValue)
```

该方法的第一个参数为回调函数，是必传的，它有三个参数

- currentValue：必需。当前元素
- index：可选。当前元素的索引值
- arr：可选。当前元素所属的数组对象

```js
let arr = [1,2,3,4,5]
arr.forEach((item, index, arr) => {
  console.log(index+":"+item)
})
```

该方法还可以有第二个参数，用来绑定回调函数内部this变量（前提是回调函数不能是箭头函数，因为箭头函数没有this）

```js
let arr = [1,2,3,4,5]
let arr1 = [9,8,7,6,5]
arr.forEach(function(item, index, arr){
  console.log(this[index])  //  9 8 7 6 5
}, arr1)
```

注意

- forEach 方法不会改变原数组，也没有返回值
- forEach无法使用 break，continue 跳出循环，使用 return 时，效果和在 for 循环中使用 continue 一致
- forEach 方法无法遍历对象，仅适用于数组的遍历

<br />

### map()

 map()  方法会返回一个新数组，数组中的元素为原始数组元素调用函数处理后的值。该方法按照原始数组元素顺序依次处理元素。其语法如下

```js
array.map(function(currentValue,index,arr), thisValue)
```

该方法的第一个参数为回调函数，是必传的，它有三个参数

- currentValue：必须。当前元素的值
- index：可选。当前元素的索引值
- arr：可选。当前元素属于的数组对象

```js
let arr = [1, 2, 3];
 
arr.map(item => {
    return item + 1;
})

// 输出结果： [2, 3, 4]
```

该方法的第二个参数用来绑定参数函数内部的this变量，是可选的

```js
let arr = ['a', 'b', 'c'];
 
[1, 2].map(function (e) {
    return this[e];
}, arr)

// 输出结果： ['b', 'c']
```

该方法还可以进行链式调用

```js
let arr = [1, 2, 3];
 
arr.map(item => item + 1).map(item => item + 1)

// 输出结果： [3, 4, 5]
```

注意

- map 方法不会对空数组进行检测
- map 方法遍历数组时会返回一个新数组，**不会改变原始数组**
- map 方法有返回值，可以return出来，map的回调函数中支持return返回值
- map 方法无法遍历对象，仅适用于数组的遍历

<br />

### for of

 for...of  语句创建一个循环来迭代可迭代的对象。在 ES6 中引入的  for...of  循环，以替代  for...in  和  forEach()  ，并支持新的迭代协议。其语法如下

```js
for (variable of iterable) {
    statement
}
```

该方法有两个参数

- variable：每个迭代的属性值被分配给该变量
- iterable：一个具有可枚举属性并且可以迭代的对象

该方法可以获取数组的每一项

```js
let arr = [
    {id:1, value:'hello'},
    {id:2, value:'world'},
    {id:3, value:'JavaScript'}
]
for (let item of arr) {
  console.log(item); 
}
// 输出结果：{id:1, value:'hello'}  {id:2, value:'world'} {id:3, value:'JavaScript'}
```

注意：

- for of 方法只会遍历当前对象的属性，不会遍历其原型链上的属性
- for of 方法适用遍历 **数组/ 类数组/字符串/map/set** 等拥有迭代器对象的集合
- for of 方法不支持遍历普通对象，因为其没有迭代器对象。如果想要遍历一个对象的属性，可以用 for in 方法
- 可以使用 break、continue、return 来中断循环遍历

<br />

### filter()

 filter() 方法用于过滤数组，满足条件的元素会被返回。它的参数是一个回调函数，所有数组元素依次执行该函数，返回结果为true的元素会被返回，如果没有符合条件的元素，则返回空数组。其语法如下

```js
array.filter(function(currentValue,index,arr), thisValue)
```

该方法的第一个参数为回调函数，是必传的，它有三个参数

- currentValue：必须。当前元素的值
- index：可选。当前元素的索引值
- arr：可选。当前元素属于的数组对象

```js
const arr = [1, 2, 3, 4, 5]
arr.filter(item => item > 2) 

// 输出结果：[3, 4, 5]
```

同样，它也有第二个参数，用来绑定参数函数内部的this变量

可以使用 filter() 方法来移除数组中的undefined、null、NAN等值

```js
let arr = [1, undefined, 2, null, 3, false, '', 4, 0]
arr.filter(Boolean)

// 输出结果：[1, 2, 3, 4]
```

注意

- filter 方法会返回一个新的数组，不会改变原数组
- filter 方法不会对空数组进行检测
- filter 方法仅适用于检测数组

<br />

### some()、every()

some() 方法会对数组中的每一项进行遍历，只要有一个元素符合条件，就返回true，且剩余的元素不会再进行检测，否则就返回false

every() 方法会对数组中的每一项进行遍历，只有所有元素都符合条件时，才返回true，如果数组中检测到有一个元素不满足，则整个表达式返回 false ，且剩余的元素不会再进行检测。其语法如下

两者的语法如下

```js
array.some(function(currentValue,index,arr),thisValue)
array.every(function(currentValue,index,arr), thisValue)
```

两个方法的第一个参数为回调函数，是必传的，它有三个参数

- currentValue：必须。当前元素的值
- index：可选。当前元素的索引值
- arr：可选。当前元素属于的数组对象

```js
let arr = [1, 2, 3, 4, 5]
arr.some(item => item > 4) 

// 输出结果：true

let arr = [1, 2, 3, 4, 5]
arr.every(item => item > 0) 

// 输出结果：true
```

注意

- 两个方法都不会改变原数组，会返回一个布尔值
- 两个方法 **都不会对空数组进行检测**
- 两个方法都仅适用于检测数组

<br />

### reduce()、reduceRight()

reduce() 方法接收一个函数作为累加器，数组中的每个值（从左到右）开始缩减，最终计算为一个值。reduce() 可以作为一个高阶函数，用于函数的 compose。其语法如下

```js
array.reduce(function(total, currentValue, currentIndex, arr), initialValue)
```

reduce 方法会为数组中的每一个元素依次执行回调函数，不包括数组中被删除或从未被赋值的元素，回调函数接受四个参数

- total：上一次调用回调返回的值，或者是提供的初始值（initialValue）
- currentValue：当前被处理的元素
- currentIndex：当前元素的索引
- arr：当前元素所属的数组对象

该方法的第二个参数是  initialValue ，表示传递给函数的初始值 （作为第一次调用 callback 的第一个参数）

```js
let arr = [1, 2, 3, 4]
let sum = arr.reduce((prev, cur, index, arr) => {
    console.log(prev, cur, index);
    return prev + cur;
})
console.log(arr, sum);
```

输出结果

```js
1 2 1
3 3 2
6 4 3
[1, 2, 3, 4] 10
```

再来加一个初始值试试

```js
let arr = [1, 2, 3, 4]
let sum = arr.reduce((prev, cur, index, arr) => {
    console.log(prev, cur, index);
    return prev + cur;
}, 5)
console.log(arr, sum);
```

输出结果

```js
5 1 0
6 2 1
8 3 2
11 4 3
[1, 2, 3, 4] 15
```

由此可以得出结论：**如果没有提供初始值initialValue，reduce 会从索引1的地方开始执行 callback 方法，跳过第一个索引。如果提供了初始值initialValue，从索引0开始执行**

reduceRight() 方法和的 reduce() 用法几乎一致，只是该方法是对数组进行倒序遍历的，而 reduce() 方法是正序遍历的

```js
let arr = [1, 2, 3, 4]
let sum = arr.reduceRight((prev, cur, index, arr) => {
    console.log(prev, cur, index);
    return prev + cur;
}, 5)
console.log(arr, sum);
```

```js
5 4 3
9 3 2
12 2 1
14 1 0
[1, 2, 3, 4] 15
```

注意

- 两个方法都不会改变原数组
- **两个方法如果添加初始值，就会改变原数组，会将这个初始值放在数组的最后一位**
- **两个方法对于空数组是不会执行回调函数**

<br />

### find()、findIndex()

 find()  方法返回通过函数内判断的数组的第一个元素的值。当数组中的元素在测试条件时返回 true 时，  find()  返回符合条件的元素，之后的值不会再调用执行函数。如果没有符合条件的元素返回 undefined

 findIndex()  方法返回传入一个测试函数符合条件的数组**第一个元素位置**（索引）。当数组中的元素在函数条件时返回 true 时，  findIndex()  返回符合条件的元素的索引位置，之后的值不会再调用执行函数。如果没有符合条件的元素返回 -1

两个方法的语法如下

```js
array.find(function(currentValue, index, arr),thisValue)
array.findIndex(function(currentValue, index, arr), thisValue)
```

两个方法的第一个参数为回调函数，是必传的，它有三个参数

- currentValue：必需。当前元素
- index：可选。当前元素的索引
- arr：可选。当前元素所属的数组对象

```js
let arr = [1, 2, 3, 4, 5]
arr.find(item => item > 2) 

// 输出结果：3

let arr = [1, 2, 3, 4, 5]
arr.findIndex(item => item > 2) 

// 输出结果：2
```

 find() 和 findIndex() 两个方法几乎一样，只是返回结果不同

-  find() ：返回的是第一个符合条件的值
-  findIndex ：返回的是第一个返回条件的值的索引值

注意

- 两个方法对于空数组，函数是不会执行的
- 两个方法不会改变原数组

<br />

### keys()、values()、entries()

三个方法都返回一个数组的迭代对象，对象的内容不太相同

- keys() 返回数组的索引值
- values() 返回数组的元素
- entries() 返回数组的键值对

三个方法的语法如下

```js
array.keys()
array.values()
array.entries()
```

这三个方法都没有参数

```js
let arr = ["Banana", "Orange", "Apple", "Mango"];
const iterator1 = arr.keys();
const iterator2 = arr.values() 
const iterator3 = arr.entries() 

for (let item of iterator1) {
  console.log(item);
}
// 输出结果： 0 1 2 3

for (let item of iterator2) {
  console.log(item);
}
// 输出结果：Banana Orange Apple Mango

for (let item of iterator3) {
  console.log(item);
}
// 输出结果：[0, 'Banana'] [1, 'Orange'] [2, 'Apple'] [3, 'Mango']
```

<br />

### 小结

|          **方法**           | **是否改变原数组** | **特点**                                                     |
| :-------------------------: | :----------------: | :----------------------------------------------------------- |
|          forEach()          |         否         | 没有返回值                                                   |
|            map()            |         否         | 有返回值，可链式调用                                         |
|           for of            |         否         | for...of 遍历具有 Iterator 迭代器的对象的属性，返回的是数组的元素、对象的属性值，不能遍历普通的 obj 对象，将异步循环变成同步循环 |
|          filter()           |         否         | 过滤数组，返回包含符合条件的元素的数组，可链式调用           |
|       every()、some()       |         否         | some() 只要有一个是 true，便返回 true；而 every() 只要有一个是 false，便返回 false |
|     find()、findIndex()     |         否         | find() 返回的是第一个符合条件的值；findIndex() 返回的是第一个返回条件的值的索引值 |
|   reduce()、reduceRight()   |         否         | reduce() 对数组正序操作；reduceRight() 对数组逆序操作        |
| keys()、values()、entries() |         否         | keys() 返回数组的索引值；values() 返回数组元素；entries() 返回数组的键值对 |

<br />

<br />

## 对象遍历方法

### for in

`for…in` 主要用于循环对象属性。循环中的代码每执行一次，就会对对象的属性进行一次操作。其语法如下

```js
for (var in object) {
 执行的代码块
}
```

其中两个参数

- var：必须。指定的变量可以是数组元素，也可以是对象的属性
- object：必须。指定迭代的的对象

```js
var obj = {a: 1, b: 2, c: 3}; 
 
for (var i in obj) { 
    console.log('键名：', i); 
    console.log('键值：', obj[i]); 
}
```

输出结果

```js
键名：a
键值： 1
键名：b
键值： 2
键名：c
键值： 3
```

注意

- for in 方法不仅会遍历当前的对象所有的可枚举属性，**还会遍历其原型链上的属性**

<br />

### Object.keys()、Object.values()、Object.entries()

这三个方法都用来遍历对象，它会返回一个由给定对象的自身可枚举属性（不含继承的和Symbol属性）组成的数组，数组元素的排列顺序和正常循环遍历该对象时返回的顺序一致，这个三个元素返回的值分别如下

- Object.keys()：返回包含对象键名的数组
- Object.values()：返回包含对象键值的数组
- Object.entries()：返回包含对象键名和键值的数组

```js
let obj = { 
  id: 1, 
  name: 'hello', 
  age: 18 
};
console.log(Object.keys(obj));   // 输出结果: ['id', 'name', 'age']
console.log(Object.values(obj)); // 输出结果: [1, 'hello', 18]
console.log(Object.entries(obj));   // 输出结果: [['id', 1], ['name', 'hello'], ['age', 18]
```

注意

- Object.keys()方法返回的数组中的值都是字符串，也就是说不是字符串的key值会转化为字符串
- 结果数组中的属性值都是对象本身**可枚举的属性**，不包括继承来的属性

<br />

### Object.getOwnPropertyNames()

`Object.getOwnPropertyNames()`方法与`Object.keys()`类似，也是接受一个对象作为参数，返回一个数组，包含了该对象自身的所有属性名。但它能返回**不可枚举的属性**，例如示例代码中的 length

```js
let a = ['Hello', 'World'];
 
Object.keys(a) // ["0", "1"]
Object.getOwnPropertyNames(a) // ["0", "1", "length"]
```

这两个方法都可以用来计算对象中属性的个数

```js
var obj = { 0: "a", 1: "b", 2: "c"};
Object.getOwnPropertyNames(obj) // ["0", "1", "2"]
Object.keys(obj).length // 3
Object.getOwnPropertyNames(obj).length // 3
```

<br />

### Object.getOwnPropertySymbols()

`Object.getOwnPropertySymbols()` 方法返回对象自身的 Symbol 属性组成的数组，不包括字符串属性

```js
let obj = {a: 1}

// 给对象添加一个不可枚举的 Symbol 属性
Object.defineProperties(obj, {
 [Symbol('baz')]: {
  value: 'Symbol baz',
  enumerable: false
 }
})
 
// 给对象添加一个可枚举的 Symbol 属性
obj[Symbol('foo')] = 'Symbol foo'
 
Object.getOwnPropertySymbols(obj).forEach((key) => {
 console.log(obj[key]) 
})

// 输出结果：Symbol baz Symbol foo
```

<br />

### Reflect.ownKeys()

Reflect.ownKeys() 返回一个数组，包含对象自身的所有属性。它和Object.keys() 类似，Object.keys() 返回属性 key，但不包括不可枚举的属性，而 Reflect.ownKeys() 会返回所有属性 key

```js
var obj = {
 a: 1,
 b: 2
}
Object.defineProperty(obj, 'method', {
 value: function () {
     alert("Non enumerable property")
 },
 enumerable: false
})

console.log(Object.keys(obj))
// ["a", "b"]
console.log(Reflect.ownKeys(obj))
// ["a", "b", "method"]
```

注意

- Object.keys() ：相当于返回对象属性数组

- Reflect.ownKeys() : 相当于

  ```js
  Object.getOwnPropertyNames(obj).concat(Object.getOwnPropertySymbols(obj)
  ```

<br />

### 小结

|          **对象方法**          | **遍历基本属性** | **遍历原型链** | **遍历不可枚举属性** | **遍历Symbol** |
| :----------------------------: | :--------------: | :------------: | :------------------: | :------------: |
|             for in             |        是        |       是       |          否          |       否       |
|         Object.keys()          |        是        |       否       |          否          |       否       |
|  Object.getOwnPropertyNames()  |        是        |       否       |          是          |       否       |
| Object.getOwnPropertySymbols() |        否        |       否       |          是          |       是       |
|       Reflect.ownKeys()        |        是        |       否       |          是          |       是       |

<br />

<br />

## 其他遍历方法

### for

for循环是应该是最常见的循环方式了，它由三个表达式组成，分别是声明循环变量、判断循环条件、更新循环变量。这三个表达式用分号分隔。可以使用临时变量将数组的长度缓存起来，避免重复获取数组长度，当数组较大时优化效果会比较明显

```js
const arr = [1,2,3,4,5]
for(let i = 0, len = arr.length; i < len; i++ ){
  console.log(arr[i])
}
```

在执行的时候，会先判断执行条件，再执行。for循环可以用来遍历数组，字符串，类数组，DOM节点等。可以改变原数组

<br />

### while

`while` 循环中的结束条件可以是各种类型，但是最终都会转为布尔值，转换规则如下

- Boolean：true为真，false为假
- String：空字符串为假，所有非空字符串为真
- Number：0为假，非0数字为真
- null/Undefined/NaN：全为假
- Object：全为真

```js
let num = 1;
            
while (num < 10){
    console.log(num);
    num ++;
}
```

`while` 和 `for` 一样，都是先判断，再执行。只要指定条件为 true，循环就可以一直执行代码

<br />

### do / while

该方法会先执行再判断，即使初始条件不成立，`do/while` 循环也至少会执行一次

```js
let num = 10;
            
do
 {
    console.log(num);
    num--;
  }
while(num >= 0);
            
console.log(num); // -1
```

不建议使用 do / while 来遍历数组

<br />

### for await of

`for await...of` 方法被称为**异步迭代器**，该方法是主要用来遍历异步对象。它是ES2018中引入的方法

`for await...of` 语句会在异步或者同步可迭代对象上创建一个迭代循环，包括 String，Array，类数组，Map， Set和自定义的异步或者同步可迭代对象。这个语句只能在  `async function` 内使用

```js
function Gen (time) {
  return new Promise((resolve,reject) => {
    setTimeout(function () {
       resolve(time)
    },time)
  })
}

async function test () {
   let arr = [Gen(2000),Gen(100),Gen(3000)]
   for await (let item of arr) {
      console.log(Date.now(),item)
   }
}

test()
```

输出结果

<img src="https://qiniu-image.qtshe.com/0BE2D6A4E8F.png" style="zoom:80%;float:left;" />