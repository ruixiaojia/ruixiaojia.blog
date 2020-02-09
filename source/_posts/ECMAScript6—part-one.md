---
title: ES6标准入门 -- 学习笔记 (上)
date: 2019-07-22 22:41:49
tags: ES6
categories: JavaScript
---

> [ECMAScript 6 入门](http://es6.ruanyifeng.com/)

## 块级作用域

为什么需要块级作用域呢？
- 内层变量可能覆盖外层变量。
- 用来计数的循环变量泄露为全局变量。

块级作用域中内层变量可以读取外层作用域的变量，且内层作用域中的变量优先级高于外层作用域同名变量。

块级作用域的出现也使立即执行匿名函数不再必要。
```javascript
// ES6块级作用域
{
    let a = 1
    {
        let a = 2
        console.log(a)  // 2
    }
}

// 立即执行匿名函数 (IIFE)
(function(){
    ...
}())
```

避免在块级作用域内声明函数，如果需要，应写成函数表达式的形式，而不是函数声明语句
```javascript
// 函数声明语句
{
    function b() {}
}

// 函数表达式
{
    let a = function() {}
}
```

## let

使用let定义的变量，只在let命令所在的代码块内有效。

相同作用域内不能重复声明同一个变量。

在for循环中，这是循环变量的部分为父级作用域，循环体内为单独的子作用域。
```javascript
for (let i = 0; i < 10; i++) { // 父作用域
    // 子作用域
    let i = 10;
    console.log(i) // 10
}
```

var命令会发生“变量提升”，可以在声明之前使用，值为undefined。let命令在它声明变量之前，该变量都是不可用的，称为“暂时性死区”。

## const

声明一个只读的常量，需立即赋值，一旦赋值就不能改变。

和let一样，也不会”变量提升“、存在”暂时性死区“、不可重复声明。

对于基本数据类型Number、Boolean、String而言，变量指向的是内存地址中，
对于复合数据类型Array、Object而言，变量指向的内存地址中保存的只是一个指针，const只能保证指针不变，不能保证其数据结构是否变化。
```javascript
const obj = {}
// 为obj添加一个属性
obj.a = 1 // { a: 1 }

// 为obj重新赋值
obg = 1 // Uncaught TypeError: Assignment to constant variable.

// 冻结obj
const obj =  Object.freeze({})
obj.a = 1 // {}
```

## 解构赋值
按照一定模式从对象或数组中提取值，然后对变量赋值，称为”解构赋值“。

### 数组的解构赋值

```javascript
// 两边模式相等，左边变量就会被赋予对应的值
let [a, b, c] = [1, 2, 3]
a // 1
b // 2
c // 3

let [ , , d] = [4, 5, 6]
d //6

let [e, ...f] = [7, 8, 9]
e // 7
f // [8, 9]

// 如果解构不成功，值为 undefined
let [g, h] = [10]
g // 10
h // undefined

// 允许指定默认值
// 如果数组成员不严格等于undefined，默认值不会生效
let [i, j = 2] = [1]
i // 1
j // 2
```

### 对象的解构赋值

```javascript
let { a: c, b: b } = { a: 1, b: 2 }
// 其中a为模式，c是真正被赋值的变量
c // 1
b // 2

// 简写为
// 变量必须与属性同名
let { a, b } = { a: 1, b: 2 }

// 嵌套
let { c, c: { d } } = { c: { d: 3 }}
c // { d: 3 }
d // 3

// 默认值，对象的属性值必须严格等于 undefined
let { e = 4 } = {}
e // 4

let f;
{ f } = { f: 5 } // SyntaxError：syntax error
// JavaScript引擎会将首行的{}理解为代码块，因此语法错误
// 应写为 ({ f } = { f: 5 })
```

### 字符串、数字、布尔值的解构赋值

```javascript
// 字符串会被解构为一个类数组的对象
const [a, b, c, d, e] = 'hello'
a // h
c // l

const { length } = 'hello'
length // 5

// Number
let { toString: s } = 123
s === Number.prototype.toString // true

// Boolean
let { toString: s } = true
s === Boolean.prototype.toString // true

// Number、Boolean都有toString属性，所以变量可以取到值
```
解构赋值的规则：若等号右边不为数组或对象，就将其转换为对象。
undefined、null不能转换对对象，所以不能对其解构赋值

### 函数参数的解构赋值

```javascript
let fun = ([x, y]) => x + y
fun([1, 2]) // 3

[[1, 2], [3, 4]].map(([x ,y]) => x + y) // [3, 7]
```

### 用途

```javascript
// 交换变量的值
let x = 1
let y = 2
[x, y] = [y, x]
x // 2
y // 1

// 函数返回多个值
function fun() {
    return {a: 1, b: 2}
}
let { a, b } = fun()
a // 1
b // 2

// 引入模块
import { a, b } from 'mou'
```

## 字符串扩展

### includes(str, num)

查找是否包含参数字符串，与indexOf()类似，但是返回布尔值。

### startsWith(str, num)
查找参数字符串是否在源字符串的头部。

### endsWith(str, num)
查找参数字符串是否在源字符串的末尾。

```javascript
// includes()
'abcdefg'.includes('c') // true
'abcdefg'.includes('c', 2) // true, 第二个参数表示开始搜索的位置

'abcdefg'.indexOf('c') // 2

// startsWith()
'abcdefg'.startsWith('a') // true
'abcdefg'.startsWith('d', 3) // true

// endsWith()
'abcdefg'.endsWith('g') // true
'abcdefg'.endsWith('g', 7) // true
```

### repeat(num)
将源字符串重复N次，返回一个新的字符串。参数需为正整数。

### padStart(num, str)
字符串头部补全，第一个参数指定字符串的最小长度，第二个参数是用来补全的字符串。

### padEnd(num, str)
字符串尾部补全，指定最小长度小于源字符串长度则返回源字符串。

```javascript
// repeat()
'aa'.repeat(3) // ’aaaaaa'

// padStart()
'X'.padStart(4, 'ak') // 'akakX'

// padEnd()
'X'.padEnd(4, 'ak') // 'Xaka'
```

### 模板字符串

```javascript
let pig = 'pig'
`Hello ${ pig }` // "Hello pig"

// 括号内可放入表达式
`${ 1 + 2 } + ${ 3 + 4 } = 10` // "3 + 7 = 10"

// 可调用函数
let fun = () => 'pig'
`Hello ${ fun() }` // "Hello pig"

// 可以嵌套
`xx${ `Y` }xx` // "xxYxx"

// 标签模板
alert`123` // 123
```

## Math对象扩展

```javascript
// Math.trunc() 去除一个数的小数部分
Math.trunc(1.5) // 1

// Math.sign() 判断一个数为正数、负数、0️⃣，返回1、-1、0
Math.sign(2) // 1
Math.sign(-2) // -1

// ** 指数运算符
2 ** 3 // 8 等同于 2 * 2 * 2 
3 ** 2 // 9 等同于 3 * 3
```

## 函数扩展

```javascript
// 函数参数默认值
let fun = (x, y = 1) => {
    console.log(x, y)
}
fun(2) // 2, 1
fun() // undefined, 1

// 函数的length属性, 返回传入函数参数的个数
(function(a, b){}).length // 2
// 指定默认值会导致length属性失真
(function(a, b, c = 3){}).length // 2

// rest 参数, 返回传入函数参数的数组, rest参数后不得有其他参数。
let fun = (...value) => {
    return value.sort()
}
fun(3, 1, 2) // [1, 2, 3]
```

### 箭头函数

- 箭头函数没有自己的this，函数内部的this就是外层代码块的this。
- 不能做为构造函数来使用，即不可使用new。
- 不存在arguments对象，可用rest参数来代替。
- 不可以使用yield命令。
- call()、apply()、bind()方法无效

### 尾调用

即在函数的最后一步是调用另一个函数，且后续再无任何操作。

函数调用后会在内存中形成一个调用记录（调用帧），保存函数调用的位置及内部变量等信息， 如果在函数A的内部调用函数B，就会在A的调用帧上方形成一个B的调用帧，等到B运行结束并将结果返回到A，A、B的调用帧才会结束，如果函数B内部还调用了函数C，以此类推，会形成一个调用栈。

尾调用优化，即只保留内层函数的调用帧，内层函数不使用外层函数的变量。

### 尾递归

递归，即函数调用自身，尾调用自身称为尾递归。

递归是非常消耗内存的，因为他们在同时生成很多很多的调用帧，很容易发生“栈溢出”的错误，但是尾递归中只存在一个调用栈，所以不会消耗过多内存，也不会发生栈溢出的错误。

```javascript
// 举个例子，阶乘
let fun = (n) => {
    if (n === 1) return 1;
    return n * fun(n - 1)
}
fun(5) // 120

// 尾递归 （部分JS解释器不支持）
let fun = (n, total) => {
    if (n === 1) return total;
    return fun(n - 1, n * total)
}
fun(5, 1) // 120
fun(10000, 1) // Infinity
fun(100000, 1) // Maximum call stack size exceeded

// 正常情况下函数尾递归优化的实现，使用蹦床函数并改写原来的递归函数
// 蹦床函数将递归执行转换为循环执行
let trampoline = (f) => {
    while(f && f instanceof Function){
        f = f() 
    }
    return f;
}
// 每执行一次返回另一个函数。
let fun = (n, total) => {
    if (n === 1) return total;
    return fun.bind(null, n - 1, n * total);
}
trampoline(fun(10000000, 1)) // Infinity

// 蹦床函数并非真正的尾递归优化。
let tco = (f) => {
    let value;
    let active = false;
    let accumulated = [];
    
    return function accumulator() {
        accumulated.push(arguments)
        if (!active) {
            active = true
            while (accumulated.length) {
                value = f.apply(this, accumulated.shift())
            }
            active = false
            return value
        }
    }
}

let fun = tco(function(n, total) {
    if (n === 1) return total;
    return fun(n - 1, n * total)
})

fun(10000000, 1) // Infinity
```

## 数组扩展

### 扩展运算符
将数组转换为逗号分隔的参数序列。任何Iterator接口的对象都可使用扩展运算符。
```javascript
// 主要用于函数调用
let arr = [1, 2, 3]
let push = (arr, ...items) => {
    arr.push(...items)
}
push(arr, 4, 5, 6) // [1, 2, 3, 4, 5, 6]

// 替代apply方法
let add = (x, y) => {
    return x + y
}
add.apply(null, arr) // 3
add(...arr) // 3

// 合并数组
let arr = [3, 4]
[1, 2].concat(arr) // ES5
[1, 2, ...arr]  //ES6

// 结合解构赋值,只能放在参数的最有一位。
[first, ...rest] = [1, 2, 3, 4]
fist // 1
rest // [2, 3, 4]

// 将字符串转为数组
[...'hello'] // ["h", "e", "l", "l", "o"]
```

### Array方法

```javascript
// Array.from(obj，fun) 将类数组或可遍历对象转换为数组
let arr = {
    '0': 'a',
    '1': 'b',
    '2': 'c',
    length: 3
}
// ES6
Array.from(arr) // ["a", "b", "c"]
// ES5
[].slice.call(arr) // ["a", "b", "c"]

// 第二个参数类似map方法。
Array.from(arr, (i) => i + 'x') // ["ax", "bx", "cx"]

// Array.of() 将一组值转换为数组
Array.of(1, 2) // [1, 2]
```

### 数组实例方法

Array.protorype

```javascript
let arr = [1, 2, 3, 4, 5, 6]

// copyWithin(target, start, end) 将指定的数组成员复制到其它位置（覆盖原有成员），返回该数组
// target: 从该位置开始替换数据
// start: 从该位置开始读取数据，默认0
// end: 到该位置停止读取数据，默认数组长度
arr.copyWithin(3) // [1, 2, 3, 1, 2, 3]
arr.copyWithin(1, 2) // [1, 3, 4, 5, 6, 6]
arr.copyWithin(3, 0, 3) // [1, 2, 3, 1, 2, 3]

// find() 查找符合条件的成员，无则返回undefined
arr.find(i => i < 0) // undefined
arr.find(i => i > 3) // 4

// findIndex(val, index, arr) 查找符合条件的成员，返回其位置,无则返回-1
arr.findIndex((val) => {
    return val > 3 // 3
})

// fill(val, start, end) 使用给定的值填充数组
arr.fill('x') // ["x", "x", "x", "x", "x", "x"]
arr.fill('x', 4) // [1, 2, 3, 4, "x", "x"]
arr.fill('x', 2, 4) // [1, 2, "x", "x", 5, 6]

// includes() 类似indexOf()，返回boolean
arr.includes(1) // true

// in运算符 判断指定的属性是否在原型链中，返回boolean
2 in arr // true
length in arr // true

// 数组空位, 表示该位置没有任何值。
// ES5会忽略空位，ES6会将空位转换为undefined
let empty = [ , , , ]
0 in empty // false
```

## 对象扩展

```javascript
// ES6中，对象的属性名和属性值相同时可以不写属性值。
let a = 'b'
let obj = { a }
a // { a: 'b' }

// ES5写法
let foo = {
    'class': function() { } // class为关键字，加引号防止解析错误 
}
// ES6写法
let foo = {
    class() { }
}

// 属性名表达式
obj['j' + 'x' + 'r'] = 315
obj.jxr // 315

// Object.is() 判断两个值是否完全相等，行为与’===‘一致
Object.is({}, {}) // false
```

### Object.assign(target, source)
将一个或多个源对象所有可枚举属性复制到目标对象, 实现合并对象（浅拷贝），若有同名属性，后者则会覆盖前者。

Object.assign()只复制一个属性的值，而不会复制它的赋值方法个取值方法。

浅拷贝：将原对象的引用直接赋给新对象，新对象只是原对象的一个引用。改变
新对象的属性值时，原对象的值也会被改变。
深拷贝：创建一个全新的对象，其值不是引用，改变原对象属性值时不会影响到新对象。
```javascript
let a = {
        a: 0,
        b: {
            c: 0,
            d: 0
        }
    }
let b = Object.assign({}, a)
JSON.stringify(b) // "{"a":0,"b":{"c":0,"d":0}}" 

// 浅拷贝
a.b.c = 1
JSON.stringify(a) // "{ "a": 0, "b": { "c": 1, "d": 0 } }"
JSON.stringify(b) // "{ "a": 0, "b": { "c": 1, "d": 0 } }"

b.b.d = 2
JSON.stringify(a) // "{ "a": 0, "b": { "c": 1, "d": 2 } }"
JSON.stringify(b) // "{ "a": 0, "b": { "c": 1, "d": 2 } }"

// 扩展运算符实现浅拷贝
let d = { e: 1, ...a }
d // "{ "e": 1, "a": 0, "b": { "c": 0, "d": 0 } }"

// 深拷贝
let c = JSON.parse(JSON.stringify(a))
a.b.c = 1
JSON.stringify(a) // "{ "a": 0, "b": { "c": 1, "d": 0 } }"
JSON.stringify(c) // "{ "a": 0, "b": { "c": 0, "d": 0 } }"

// 为对象添加方法
Object.assign(example.prototype, {
    method() {}
})

example.prototype.method = function() {}
// 以上两中写法等同。
```

### Object.setPrototypeOf()、Object.getPrototypeOf()

Object.setPrototypeOf(obj, prototype): 将指定的对象的原型设置为另一个对象或null

Object.getPrototypeOf(obj): 返回指定对象的原型

```javascript
let proto = {}
let obj = {
        a: 0
    }

Object.setPrototypeOf(obj, proto)
proto.b = 1
obj.b // 1

Object.getPrototypeOf(obj) // { b: 1 }
```

### Null传导运算符 (提案)

在开发过程中我们往往会遇到需要判断某个对象的内部属性是否存在。比如要读取data.results.product, 安全的写法如下：

```javascript
const product = (data && data.results && data.results.product) || 'default'
```

这样的写法虽然解决了data 或 data.results 不存在而导致的报错，但是这样的层层判断是非常麻烦的。Null传导运算符可以简化上面的写法。

```javascript
const product = data?.results?.product) || 'default'

fun?.(...arg) // 函数调用方法

new C?.(...arg) // 构造函数调用
```

### 遍历操作

```javascript
let obj = {
        a: 1,
        b: 2
    }
    
// for...in
for(let i in obj) {
    console.log(i)
}
// a
// b

// Object.keys() 返回数组，包含对象自身的所有属性，不含Symbol
Object.keys(obj) // ["a", "b"]

// Object.values() 返回数组，包含对象自身所有的属性值。
Object.values(obj) // [1, 2]

// Object.entries()  返回二维数组，成员是键值对数组。
Object.entries(obj) // [["a", 1], ["b", 2]]

// Object.getOwnPropertyNames(obj) 返回数组, 包含对象自身所有属性，不包含Symbol和不可枚举属性
Object.getOwnPropertyNames(obj) // ["a", "b"]

// Object.getOwnPropertySymbols(symbols) 返回自身所有Symbol属性

// Reflect.ownKeys(obj) 返回所有属性。everything~
```

## Set、Map数据结构

### Set

类似数组，但是成员唯一，没有重复。本身是构造函数，用来生成set数据解构。

Set实例内部，使用类似于“===“来判断两个值是否相同。

两个NaN是不相等的，但在Set实例内部，两个NaN相等。

#### 实例属性及方法

```javascript
// 数组去重，将Set结构转换为数组。
let s = new Set([1, 2, 3, 3, 3, 4])
[...s] // [1, 2, 3, 4]
Array.from(s) // [1, 2, 3, 4]

// size 返回Set实例的成员总数
s.size // 4

// add(val) 添加成员，返回添加后的结构
s.add(5) // Set(5) {1, 2, 3, 4, 5}

// delete(val) 删除成员，返回boolean
s.delete(1) // true

// has(val) 返回boolean，是否包含成员
s.has(2) // true

// 清除成员，无返回值
s.clear() // 
```

#### 遍历操作

Set遍历的顺序就是插入的顺序。

##### keys()、values()
方法返回遍历器对象，由于Set数据结构键值键名相同，所以这两个方法的行为是完全一致的。

##### entries()
方法返回遍历器对象，同时包括键名和键值。

##### forEach()
无返回值，参数为处理函数，对每个成员执行操作。该处理函数的参数为键值、键名、集合本身。

```javascript
// keys()
let s = new Set([1, 2, 3])
for(let i of s.keys()){
    console.log(i)
}
// 1
// 2
// 3

// values()
for(let i of s)){ // 可省略values方法
    console.log(i)
}
// 1
// 2
// 3

// entries()
for(let i of s.entries()){
    console.log(i)
}
// [1, 1]
// [2, 2]
// [3, 3]

// forEach()
s.forEach((values, keys, others) => {
    console.log(values, keys, others)
})
// 1 1 Set(3) {1, 2, 3}
// 2 2 Set(3) {1, 2, 3}
// 3 3 Set(3) {1, 2, 3}
```

#### Set实现交集、并集、差集

```javascript
let a = new Set([1, 2, 3])
let b = new Set([4, 3, 2])

let union = new Set([...a, ...b])
union // 交集 Set(4) {1, 2, 3, 4}

let intersect = new Set([...a].filter(x => b.has(x)))
intersect // 并集 Set(2) {2, 3}

let difference = new Set([...a].filter(x => !b.has(x)))
difference // 差集 Set(1) {1}
```

### WeakSet

与Set结构类似，也是不重复的值的集合，但与Set有两个区别
- WeakSet成员只能是对象。
- WeakSet中的对象都是弱引用，垃圾回收机制不考虑WeakSet对该对象的应用，不会因此而应发内存泄露。

WeakSet适合临时存放一组对象，以及和对象绑定的信息，只要对象在外部小时，它在WeakSet中的应用就会自动消失。

任何具有iterable(可迭代的)接口的对象都可以作为WeakSet的参数。

```javascript
const ws = new WeakSet([[1, 2], [3, 4]])
ws // WeakSet { [1, 2], [3, 4] }

// 向实例添加成员
const obj = {}
ws.add(obj) // WeakSet { {} }

// 某个值是否在实例中
ws.has(obj) // true

// 清除实例指定成员
ws.delete(obj) // true
```

可用于储存DOM节点，不用担心节点从文档中移除时引发的内存泄露。

### Map
JavaScript的对象（Object）本质上是键值对的集合，但是只接受字符串作为键。
Map数据结构类似于Object也是键值对的集合，但键可以是各种类型的值。

```javascript
// Map构造函数接受一个二维数组作为参数.
let m = new Map([[1, 2], [3, 4], [5, 6]])
// Map(3) {1 => 2, 3 => 4, 5 => 6}

// 也可接受双元素数组的Set、Map对象用来生成新的Map数据结构。
let s = new Set([[1, 2], [3, 4], [5, 6]])
new Map(s) // Map(3) {1 => 2, 3 => 4, 5 => 6}
new Map(m) // Map(3) {1 => 2, 3 => 4, 5 => 6}

// Map的键实际上是与内存地址绑定，内存地址不同则视为两个键。
let a = ['a']
let b = ['a']
let m = new Map()

m.set(a, 1)
m.set(b, 2)

m.get(a) // 1
m.get(b) // 2
m.get(['a']) // undefined
```

#### 实例属性及方法

```javascript
let m = new Map([[1, 2], [3, 4]])

// size 返回Map结构的成员个数
m.size // 2

// set(key, val) 设置键值，若存在则覆盖，返回整个Map结构
m.set(5, 6) // Map(3) {1 => 2, 3 => 4, 5 => 6}

// get(key) 返回对应的值，无则返回undefined
m.get(3) // 4
m.get(7) // undefined

// has(key) 查找键名，返回boolean
m.has(1) // true

// delete() 删除键值，返回boolean
m.delete(1) // true

// clear() 清空数据结构，无返回值。
```

#### 遍历方法

keys(): 返回键名
values(): 返回键值
entries(): 返回所有成员
forEach(): 遍历所有成员

```javascript
let m = new Map([['a', 1], ['b', 2]])

// keys()
[...m.keys()] // ["a", "b"]

// values()
for (let value of m.values()) {
    console.log(value)
}
// 1
// 2

// entries()
for (let entries of m.entries()) {
    console.log(entries)
}
// ["a", 1]
// ["b", 2]

// forEach(value, key)
m.forEach((value, key) => {
    console.log(key, value)
})
// a 1
// b 2
```

### WeakMap

- 只接受对象作为键名(除了null)。
- 键名所指向的对象不计入垃圾回收机制。

```javascript
// methods
let wm = new WeakMap()

wm.set()
wm.get()
wm.has()
wm.delete()

// example
let e1 = document.getElementById('a')
let e2 = document.getElementById('b')

let arr = [[e1, 'a'], [e2, 'b']]
// 一旦不需要使用e1、e2这两个对象的时候，需要将其手动删除，否则会造成内存泄露。垃圾回收机制不会释放其所占用的内存。
arr[0] = null
arr[1] = null

// 当需要向对象中添加数据又不想干扰垃圾回收机制时，即可使用WeakMap
let wm = new WeakMap()
let ele = document.getElementById('c')

wm.set(ele, 'c')
wm.get(ele) // 'c'

// WeakMap弱引用的只是键名，键值正常引用，及时WeakMap外部消除了对象，WeakMap内部依然存在

let key = {}
let obj = { obj: 1 }
wm.set(key, obj)
obj = null

ws.get(key) // {obj: 1}
```

### 垃圾回收机制

> 基本类型的值保存在栈内存中。
> 引用类型的值保存在堆内存中，会在栈中保存一个指向该对象的指针。

计算机分配给浏览器的可用内存是有限制的，为了防止运行了JavaScript的页面耗尽系统内存而导致系统崩溃，执行环境会负责管理代码执行过程中使用的内存。垃圾收集器是周期性运行的，找到那些不再使用的变量打上标记然后释放其占用的内存。
标识无用变量的方式有两种：
- 标记清除
    垃圾收集器在运行的时候会给储存在内存中的所有变量都加上标记，然后会去掉环境中的变量以及被环境中的变量引用的变量的标记。在此之后，如果变量再次被加上标记，则被视为准备删除的变量，原因是环境中的变量已经无法访问到这些变量了。最后垃圾收集器将销毁那些带标记的值并收回其占用的内存空间。

- 引用计数
    跟踪记录每个值的引用次数。
    循环引用会导致大量内存泄露。

**一旦变量不再使用，最好将其值设为null来解除引用。**



## Symbol

ES5中对象属性名都是字符串，容易造成属性名冲突，Symbol表示独一无二的值，是一种新的原始数据类型。Symbol函数接收一个字符串作为参数，仅用于对当前Symbol值的描述。

```javascript
const s = Symbol()
s // Symbol()

const s1 = Symbol('sym')
const s2 = Symbol('sym')

s1 == s2 // false

let obj = {}
obj[s1] = 'hello'
obj // { Symbol(sym): "hello" }

// 在对象内部使用Symbol值定义属性名时必须放在方括号内
const fun = Symbol('fun')
let obj2 = {
    [s2]: 'hello',
    [fun]: function(){}
}
obj2 // { Symbol(sym): "hello", Symbol(fun): ƒ }
// 可简写为 [fun]() {}

// 使用Symbol作为属性名，使用常规的遍历方法无法获取Symbol属性名，需要使用Object.getOwnPropertySymbols(),该方法返回一个数组，成员是Symbol属性名的Symbol值。
Object.keys(obj2) // []
Object.getOwnPropertySymbols(obj2) // [Symbol(sym), Symbol(fun)]
```

### Symbol方法

#### Symbol.for()、Symbol.keyFor()
Symbol.for()接收一个字符串作为参数，然后搜索有没有以该字符串为名称的Symbol值，如果有则返回，无则新建一个并返回。

Symbol.for()会登记在**全局环境中**供搜索,可以再不同的iframe中取到，循环30次Symbol.for()都会返回同一个值，Symbol()则不会登记，会返回30个不同的值。

Symbol.keyFor()会返回一个已登记的Symbol类型值的key。

```javascript
let s1 = Symbol.for('sym')
let s2 = Symbol.for('sym')
let s3 = Symbol('sym')
s1 === s2 // true

Symbol.keyFor(s2) // 'sym'
Symbol.keyFor(s3) // undefined
```

#### Symbol.hasInstance()

指向一个内部方法,对象在使用instanceof时会调用该方法，判断该对象是否为某个构造函数的实例。

```javascript
class Myclass {
    [Symbol.hasInstance](foo) {
        reutrn foo instanceof Array;    
    }
}

[1, 2] instanceof new Myclass() // true
```

#### Symbol.match()

在使用String.prototype.startWith()、String.prototype.endWith()、String.prototype.includes()这类方法时，会检查其第一个参数是否为正则表达式，如果是正则表达式则会抛出错误。

将Symbol.match 置为 false 时则不会检查是否为正则表达式。

```javascript
let reg = /foo/

'/foo/'.includes(reg) // First argument to String.prototype.includes must not be a regular expression

reg[Symbol.match] = false
'/foo/'.includes(reg)
```