---
title: ES6标准入门 -- 学习笔记 (下)
date: 2019-07-22 22:41:50
tags: ES6
categories: JavaScript
---



## Class

ES6中的Class写法只是让对象原型的写法更加清晰，更像面向对象编程 ( Object Oriented Programming，OOP ), 于构造函数用法完全一致。

```javascript
// ES5

function Fun(x, y) {
    this.x = x
    this.y = y
}

Fun.prototype.plus = function() {
    return this.x + this.y
}

var num = new Fun(1, 2)
num.plus() // 3

// ES6
// 类的内部默认使用严格模式，不需要再显示声明。

class Fun {
    constructor(x, y) {
        // 类的原型方法，使用new关键字则自动调用该方法
        // 若未声明该方法，则会默认添加一个空的constructor方法
        this.x = x
        this.y = y
        
        // 默认return this;
    }
    
    plus() {
        return this.x + this.y
    }
}
var num = new Fun(1, 2)
num.plus() // 3

typeof Fun // "function"

// 所有方法不可枚举

Object.keys(Fun.prototype) // []

// 所有实例共享同一个原型对象

let num1 = new Fun(1, 2)
let num2 = new Fun(2, 3)
num1.__proto__ === num2.__proto__ // true

// class表达式

let name = new class {
    constructor() {
        this.name = 'j.'
    }
    
    console() {
        console.log(this.name)
    }
}
name.console() // 'j.'

// 类不存在变量提升

new Fun() // Cannot access 'Fun' before initialization
class Fun {}

// 私有方法
// ES6不提供私有方法，但可使用其他方法实现

// 方法一
class Widget {
    foo(options) {
        this._bar(options)
    }
    
    // 方法前加_表示为私有方法，但在类的外部依旧可以调用该方法。
    _bar(options){
        return options;
    }
}

// 方法二
class Widget {
    foo(options) {
        bar.call(this, options)
    }
}

function bar(options) {
    return options;
}

// 方法三 使用Symbol
class Widget {
    foo(options) {
        this[bar](options)
    }
    [bar](options) {
        return options;
    }
}

// 私有方法（提案）在变量或函数前加#
class Widget {
//    #a;
//    #fun() {
//    }
}
```

### this指向

```javascript
// 类的方法中的this默认指向类的实例，在外部调用类的方法时this指向可能会指向运行环境。

class Logger {
    print(name = 'J'){
        this.console(name)
    }
    
    console(name) {
        console.log(name)
    }
}
const logger = new Logger()
const { print } = logger
print() // Cannot read property 'console' of undefined

// 方法一
class Logger {
    constructor() {
        this.print = this.print.bind(this)
    }
    // ...
}

// 方法二 箭头函数
class Logger {
    this.print = (name = 'J') => {
        this.console(name)
    }
    // ...
}

// 方法三 Proxy
function selfish(target) {
    const cache = new WeakMap()
    const handler = {
        get (target, key) {
            const val = Reflect.get(target, key)
            if (typeof val !== 'function') {
                return val;
            }
            if (!cache.has(val)) {
                cache.set(val, val.bind(target))
            }
            return cache.get(val)
        }
    }
    const proxy = new Proxy(target, handler)
    return proxy
}

const a = selfish(new Logger())
const { pring } = a
pring() // J
```

### getter、setter

可使用get、set关键字对类内部的某个属性设置存值函数(setter)和取值函数(getter)

```javascript
class Myclass {
    constructor() {
        this.name = 'jjj'
    }
    get prop () {
        return this.name
    }
    set prop (val) {
        this.name = val
        console.log(this.name)
    }
}

let myclass = new Myclass()
myclass.prop = 'xxx' // 'xxx'
console.log(myclass.prop) // 'xxx'
```

### class的静态方法

类相当于实例的原型，所有在类上定义的方法都会被实例继承，类的方法前加static关键字则不能被实例继承，只能通过类调用，称为“静态方法”。

```javascript
class Foo {
    static hello() {
        console.log('hello')
        return 'hello'
    }
}
// 在类上调用
Foo.hello() // hello

// 在类的实例上调用
let foo = new Foo() // Uncaught TypeError: foo.hello is not a function

// 父类的静态方法可以被子类继承，子类也可以通过super调用该方法

class Son extends Foo {
    static method() {
        console.log(super.hello() + ' world')
    }
}

// 通过子类调用
Son.hello() // hello

// 从super对象上调用
Son.method() // hello
            // hello world
```

### class的静态属性和实例属性

静态属性是Class本身的属性，即Class.prop，而不是定义在实例对象this上的属性。

```javascript
// 实例属性
class Foo {
    state = 'state'
    method() {
        console.log(this.state)
    }
}
let foo = new Foo()
foo.hello() // state

// 静态属性
class Foo {
    static state = 'state'
    method() {
        console.log(this.state)
    }
}

let a = new Foo()
a.method() // Foo.state = state
          // this.state = undefined
console.log(a.state) // undefined
console.log(Foo.state) // state
```

### new.target属性

在class内部调用new.target属性，如果是通过new命令调用的会返回该类，如果不是则返回undefined

```javascript
class Foo {
    constructor() {
        console.log(new.target)
    }
}
new Foo() // class Foo {
         //     constructor() {
         //         console.log(new.target)
         //     }
         // }
         

class Foo {
    method() {
        console.log(new.target)
    }
}
const foo = new Foo()
foo.method() // undefined
```

### class继承

在ES5中继承实际是先创造子类的实例对象this，然后再将父类的方法添加到this上面。

在ES6中，子类在继承父类时实际上是继承父类的this对象，然后对其加工得到自己的this。必须在constructor中通过调用super方法，来调用父类的构造函数并新建父类的this对象，否则会报错。

```javascript
class Foo {
    constructor(x, y) {
        this.x = x
        this.y = y

        console.log('foo', x ,y)
    }

    method() {
        console.log('foo method', this.x, this.y)
    }
}

class Son extends Foo {
    constructor(x, y) {
        super(x, y) // super作为函数调用时代表父类的构造函数，且只能在constructor中使用。


        console.log(x, y)
        console.log(this.x, this.y)
        super.method() // super作为对象时指向父类的原型，可以调用父类的方法，super.method()相当于Foo.prototype.method()
    }
}

new Son(2, 3)
// foo 2 3
// 2 3
// 2 3
// foo method 2 3

// 判断一个类是否继承了另一个类
Object.getPrototypeOf(Son) === Foo // true
```

### 类的prototype属性和__proto__属性

子类的__proto__属性标识构造函数的继承，总是指向父类

子类的prototype属性的__proto__属性表示方法的继承，总是指向父类的prototype

```javascript
class Foo {
    FooMethod() {

    }
}
class Son extends Foo {
    SonMethod () {

    }
}

console.log(Son.__proto__ === Foo) // true
console.log(Son.prototype.__proto__ === Foo.prototype) // true
```

### 实例的__proto属性

子类实例的__proto__属性的__proto__属性指向父类实例的__proto__属性。即子类原型的原型是父类的原型。

```javascript
class Foo {
    FooMethod() {

    }
}
class Son extends Foo {
    SonMethod () {

    }
}
let foo = new Foo()
let son = new Son()

console.log(son.__proto__.__proto__ === foo.__proto__) // true
```



## Proxy

Proxy用于修改某些操作的默认行为，可以理解为在目标对象前架设了‘拦截层’，任何对该对象的操作都必须先通过Proxy。

```javascript
// target: 要拦截的目标对象。
// handler: 拦截行为。
var proxy = new Proxy(target, handler)
```

### 实例方法:

#### get()

用于拦截某个属性的读取操作。

```javascript
let o = {
    name: 'j',
}
let proxy = new Proxy(o, {
    get: function(target, propKey) {
        if (propKey in target) {
            console.log('get ' + propKey)
            return target[propKey]
        } else {
            console.log('没有' + propKey)
        }
    }
})

console.log(proxy.name) // get name
                     // j
                     
// get 方法可以继承

let obj = Object.create(proxy)
console.log(obj.age) // 没有age
                   // undefined
                   
// 实现读取数组负数索引
arr = [1,2,3,4,5]
let proxy = new Proxy(arr, {
    get: function(target, propKey) {
        let index = Number(propKey)
        if (index < 0) {
            propKey = target.length + index
        }
        return Reflect.get(target, propKey)
    }
})
console.log(proxy[-1]) // 5
```

#### set()

用于拦截某个属性的赋值操作。

```javascript
let proxy = new Proxy({}, {
    set: function(target, propKey, value) {
        if (propKey === 'age') {
            if (!Number.isInteger(value)) {
                throw new TypeError('The age is not an integer')
            }

            if (value > 200) {
                throw new RangeError('The age seems invalid')
            }
            target[propKey] = value
        }
    }
})
proxy.age = 100
console.log(proxy) // Proxy {age: 100}
```

#### apply()

用于拦截函数调用、call、apply操作。
apply接受三个参数，目标对象、目标对象上下文(this)、目标对象参数数组

```javascript
let target = function(){ return "I am the target"}
let prox = new Proxy(target, {
    apply(target, ctx, args) {
        console.log(target())
        console.log(ctx)
        console.log(args)
    }
})
prox(1, 2) // I am the target
          // undefined
          // [1, 2]
          
let plus = function(x, y) {
    return x + y;
}
let prox = new Proxy(plus, {
    apply(target, ctx, args) {
        return Reflect.apply(...arguments) * 2
    }
})
prox(2, 3) // 10
prox.call(null, 1, 2) // 6
prox.apply(null, [3, 4]) // 14
```

#### has()

```javascript

```



## Reflect

- 从Reflect对象上可以获取语言内部的方法
- 修改某些Object对象的返回结果，使其合理
- 让Object操作都变成函数行为
- 无论Proxy如何修改默认行为，都可以在Reflect上获取默认行为

### 静态方法

#### Reflect.get()



## Promise
异步编程的一种解决方案，比传统的回调函数更合理更强大。
promise简单来说就是一个容器，里面保存着某个未来才会结束的事件的结果。
ES6规定promise对象是一个构造函数，用来生成promise实例。promise对象有三种状态：pending（进行中）、fulfilled（已成功）和rejected（已失败）

```javascript
// promist构造函数接收一个函数作为参数，该函数接收两个函数resolve、reject作为参数。

// resolve函数将promise对象的状态从pinding变为resolved，在异步操作成功时调用。
// reject函数将promise对象的状态从pending变为rejected，在异步操作失败时调用。

var promise = new Promise(function(resolve, reject) {
    if (true) {
        resolve(val)
    } else {
        reject(error)
    }
})

Promise.prototype.then()
// promise实例生成后，可以使用then方法分别指定resolve和reject的回调函数。
promise.then(function(success) {
    // success回调
}, function(error) {
    // error回调
})


Promise.prototype.catch()
// 用于指定发生错误时的回调函数，


Promise.all()
// 用于将多个Promise实例包装成一个新的Promise实例，该方法接受一个数组，数组成员均为promise实例。
let p = Promise.all([p1, p2, p3])
// 只有所有实例成员的状态都变为resolved时，p的状态才会变成resolved。
// 只要有一个实例变成rejected，则p的状态变为rejected，返回变为rejected状态的实例的返回值。给每一个promise实例都添加catch回调，并return结果，这样就可以在最后的结果中都获取到。


Promise.allSettled()
// 接受多个promise实例数组，只有等到所有参数实例都返回结果(不管成功失败)，该实例才会结束。Promise.all()无法确定所有请求都结束。想要达到这个目的，Promise.allSettled()就容易多了。


Promise.race()
// 接受多个promise实例数组，哪个先resolved就返回哪个值。


Promise.resolve('foo')
// 等价于
new Promise((resolve) => { resolve('foo') })


Promise.reject('foo')
// 等价于
new Promise((resolve, reject) => { reject('foo') })


Promise.prototype.done()
// 处于回调链的最尾端，保证捕捉到可能出现的任何错误。


Promise.prototype.finally() // ES2018
// 不管Promise对象最后状态如何都会执行的操作，接受一个普通函数作为回调函数，不论如何都会执行。

```



## Iterator 遍历器

为各种不同的数据结构提供的统一简便的访问接口(symbol.iterator)，供for...of遍历使用。

```javascript
let arr = ['a', 'b'];
let iter = arr[Symbol.iterator]();

iter.next() // { value: 'a', done: false }
iter.next() // { value: 'b', done: false }
iter.next() // { value: undefined, done: true }
```

原生具备iterator接口的数据接口：Array、Map、Set、String、arguments、NodeList对象。除此之外都需要自己手动在Symbol.iterator属性上面部署，才会被for...of遍历。

除了for...of，还有一些会默认调用Symbol.iterator接口的场合，比如：解构赋值、扩展运算符、yield\*、Array.from()、Map()、Set()、WeakMap()、WeakSet()、Promise.all()、Promise.race()

iterator遍历过程

- 创建一个指针对象，指向当前数据的起始位置。
- 第一次调用指针对象的next方法，指针指向数据结构的第一个数据成员。
- 以此类推，直到指向数据结构的结束位置。

手动部署Symbol.iterator

```javascript
// class
class RangeIterator {
  constructor(start, stop) {
    this.value = start;
    this.stop = stop;
  }

  [Symbol.iterator]() { return this; }
  
  next() {
    var value = this.value;
    if (value < this.stop) {
      this.value++;
      return {done: false, value: value};
    }
    return {done: true, value: undefined};
  }
}

for (var value of new RangeIterator(0, 3)) {
  console.log(value); // 0, 1, 2
}

// Object
let obj = {
  data: [ 'hello', 'world' ],
  [Symbol.iterator]() {
    const self = this;
    let index = 0;
    return {
      next() {
        if (index < self.data.length) {
          return {
            value: self.data[index++],
            done: false
          };
        } else {
          return { value: undefined, done: true };
        }
      }
    };
  }
};

for (var value of obj) {
  console.log(value); // 'hello', 'world'
}

// 使用Generator部署Symbol.iterator
let myIterable = {
  [Symbol.iterator]: function* () {
    yield 1;
    yield 2;
    yield 3;
  }
}
[...myIterable] // [1, 2, 3]

// 或者采用下面的简洁写法
let obj = {
  * [Symbol.iterator]() {
    yield 'hello';
    yield 'world';
  }
};

for (let x of obj) {
  console.log(x);
}
// "hello"
// "world"
```



## Generator

Generator 函数是 ES6 提供的一种异步编程解决方案。语法上，首先可以把它理解成，Generator 函数是一个状态机，封装了多个内部状态。Generator 函数还是一个遍历器对象生成函数，可以使用for...of依次遍历 Generator 函数内部的每一个状态。

Generator与普通函数的区别在于需要在函数名前加*号，函数体内可以使用yield表达式，调用后不会直接执行，而是返回一个Iterator对象，调用该对象的next方法，直到遇到yield表达式或return为止。

```javascript
function* generator() {
  yield 1;
}

let generator = function* () {
  return 1;
}

// 作为对象属性
let obj = {
  myGeneratorMethod: function* () {}
};
// 可简写为
let obj = {
  * myGeneratorMethod() {}
};
```

### yield表达式

yield表达式是Generator函数暂停的标识，且具备位置记忆功能。

- 遇到`yield`表达式，就暂停执行后面的操作，并将紧跟在`yield`后面的那个表达式的值，作为返回的对象的`value`属性值。
- 下一次调用`next`方法时，再继续往下执行，直到遇到下一个`yield`表达式或`return`语句为止，并将`return`语句后面的表达式的值，作为返回的对象的`value`属性值。
- 如果该函数没有`return`语句，则返回的对象的`value`属性值为`undefined`。

如果在Generator函数内部需要调用另一个Generator函数或调用iterator接口的话可以使用`yield*`表达式完成遍历。等同于在yield表达式后面部署了一个for...of循环。

```javascript
function* foo() {
  yield 'a';
  yield 'b';
}
function* bar() {
  yield 'x';
	yield* foo();
  yield 'y';
}
for (let v of bar()){
  console.log(v);
}
// x, a, b, y
```

### Generator函数应用

- 控制流管理
- 部署Iterator接口
- 作为数据结构
- 异步操作的同步化表达
- 异步应用（使用co模块做执行器）



## async

Generator函数的语法糖，返回值为Promise，且自带执行器。

async函数返回一个Promise对象，必须等内部所有await执行完毕才会发生状态改变。如果await后面的异步操作出错，等同于async函数返回的Promise对象reject，最好使用try...catch包裹。

正常情况下await后面是一个Promise对象，返回该对象的结果，如果不是则直接返回。

注意：

```javascript
// 两个独立的异步操作（不互相依赖），被写成继发关系会比较耗时。
// example:
let foo = await getFoo();
let bar = await getBar();

// 可让其同时触发
// 写法一
let [foo, bar] = await Promise.all([getFoo(), getBar()]);

// 写法二
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
```



## 异步编程方法

- 回调函数
- 事件监听
- 发布/订阅
- Promise
- Generator
- async



## Decorator 装饰器

### 类的装饰

在代码编译时，将类作为装饰器的第一个参数传入装饰器。可在类上添加静态属性或实例属性/方法。

```javascript
@decorator
class A {}

// 等同于

class A {}
A = decorator(A) || A;
```

### 方法的装饰

方法的装饰器接收三个参数，类的原型对象、所要装饰的属性名、该属性的描述对象。

```javascript
class Person {
  @readonly
  name() { return `${this.first} ${this.last}` }
}

function readonly(target, name, descriptor) {
    // descriptor对象原来的值如下
  // {
  //   value: specifiedFunction,
  //   enumerable: false,
  //   configurable: true,
  //   writable: true
  // };
  descriptor.writable = false;
  return descriptor;
}
```

注：由于存在函数提升，使得装饰器不能用于函数。类是不会提升的，所以就没有这方面的问题。