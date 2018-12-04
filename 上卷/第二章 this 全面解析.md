# 第二章 this 全面解析

<!-- TOC -->

- [第二章 this 全面解析](#第二章-this-全面解析)
  - [基本概念](#基本概念)
  - [绑定的四大规则](#绑定的四大规则)
    - [默认绑定](#默认绑定)
    - [隐式绑定](#隐式绑定)
      - [隐式丢失](#隐式丢失)
    - [显式绑定](#显式绑定)
      - [硬绑定](#硬绑定)
    - [new 绑定](#new-绑定)
      - [普通函数](#普通函数)
      - [构造函数](#构造函数)
    - [优先级](#优先级)
  - [ES6 箭头函数](#es6-箭头函数)
    - [代码片段1:](#代码片段1)
    - [代码片段2:](#代码片段2)

<!-- /TOC -->

## 基本概念
`this` 是个很特别的关键字, 被自动定义在所有函数的作用域中;

`this` 关键字并不会像它的字面意思那样, 指向函数自身; 相对**词法作用域**(只关注函数在何处声明), 它的行为更像**动态作用域**(只关注函数从何处调用);

> 下面一个简单例子, 解释**词法作用域**和**动态作用域**的区别:

``` javascript
function foo() {
    console.log(a)
}

function bar() {
    var a = 2
    foo()
}

var a = 1

bar()
```

运用**词法作用域**:

- `foo` 函数在全局作用域下声明, 所以其**作用域内部**可以访问到全局作用域的 `a`, 但它访问不到 `bar` 函数内部的 `a`;
- 输出结果 `1`

运用**动态作用域**:

- `foo` 函数在 `bar` 函数内部调用, 第一个查找到的 `a` 变量是 `bar` 函数作用域内部的 `a`;
- 输出结果 `2`

---

## 绑定的四大规则

> `this` 的指向, 是根据函数`调用的位置` 决定的, 我们必须先找到调用位置, 然后判断其适用以下四条规则的哪一条

### 默认绑定

我们可以把这条规则看做无法使用其他规则时的默认规则;

**直接使用不带任何修饰的函数引用**, 在调用时, 只能使用 `默认绑定`;

`默认绑定` 会将 `this` 绑定到 `window` 对象上;(`this === window`)

``` javascript
function foo() {
  console.log(this === window)

  bar() // true
}

function bar() {
  console.log(this === window)
}

foo() // true
```

> 但在严格模式下, **默认绑定**会绑定一个 `undefined`:

``` javascript
"use strict"

function foo() {
  console.log(this)
}

foo() // undefined
```

### 隐式绑定

通过考虑 **调用位置**是否具有**上下文对象**, 或者说是否被某个对象拥有包含, 来判断是否运用这条规则:

> 如下示例, 我们声明一个 `foo` 函数, 并将其赋值给 `obj` 对象的 `foo` 属性:

``` javascript
function foo() {
    console.log(this.a)
}

var obj = {
    a: 1,
    foo: foo
}

obj.foo() // 1
foo()     // undefined
```

> 对象属性引用链中只有最后一层在调用位置中起作用, 如下:

``` javascript
function fn() {
    console.log(this.msg)
}

var child = {
    msg: 'child',
    fn: fn
}

var parent = {
    msg: 'parent',
    child: child
}

parent.child.fn() // child
```

#### 隐式丢失

如下示例, `o.foo` 的 `this` 隐式绑定在了 `o` 对象上, 而 `bar` 引用了 `o.foo` 函数本身, 所以此时的 `bar()` 其实是一个不带任何修饰的函数调用, 因此使用了 **默认绑定** 规则:

``` javascript
var o = {
    a: 1,
    foo() {
        console.log(this.a)
    }
}

var bar = o.foo

o.foo() // 1
bar()   // undefined
```

另一个很出乎意料的例子, 示例中, `bar(o.foo)` 实际上采用了**隐式赋值**: `callback = o.foo`, 事实上跟上面的例子一样, 都是直接引用了 `o.foo` 函数本身, 所以造成了 **隐式丢失**:

``` javascript
function bar(callback) {
    callback()
}

var o = {
    a: 1,
    foo() {
        console.log(this.a)
    }
}

bar(o.foo) // undefined
```

### 显式绑定

现在, 我们并不想通过 **隐式** 或者 **默认** 的方式来间接绑定 `this` 的指向, 我们需要强制的为函数指定一个绑定对象!

我们可以通过使用 `call()` 和 `apply()` 方法来实现;

`JavaScript` 提供的大多数函数以及**我们自己创建的所有函数**都可以使用这两个方法;

``` javascript
fn.call(obj Object [, ...arguments])
```

第一个参数接收一个对象, 作为 `this` 关键字绑定的对象, 第二个参数是该函数传递的参数;

``` javascript
function foo(a, b) {
    console.log((a + b) * this.c)
}

var obj = {
    c: 2
}

foo.call(obj, 1, 2) // 6

// foo.apply(obj, [1, 2])
// call() 与 apply() 的效果完全一致, 唯一不同的只是传递参数的格式不同
```

#### 硬绑定

使用**显示绑定**时, 我们重复进行绑定, 仍然会让之前的绑定值丢失:

``` javascript
var obj = { name: 'muzi' }

function foo() {
    console.log(this.name)
}

foo.call(obj)  // muzi

setTimeout(() => (
    foo.call({ name: 'yaya' } // yaya
)), 100)
```

如果我们想避免这种情况, 就需要使用 **硬绑定**;

硬绑定的典型应用场景就是创建一个**包裹函数**, 传入所有的参数并返回接收到的所有值:

``` javascript
var obj = {
    name: 'muzi'
}v

function foo() {
    console.log(this.name)
}

function wrapper() {
    foo.call(obj)
}

wrapper()  // muzi
wrapper.call({ name: 'yaya' })  // muzi
```

虽然 `wrapper` 绑定了一个新的对象, 但当 `wrapper` 每次被调用时,
`foo` 都会显示绑定 `obj` 对象;

所以无论 `wrapper` 如何调用, `foo` 函数的绑定值都不会被改变.

> 我们可以根据这个特性, 创建一个**辅助绑定函数**:

``` javascript
function bind(fn, obj) {
  return function() {
    return fn.apply(obj, arguments)
  }
}
```

``` javascript
function foo(b, c) {
    return this.a + b + c
}

var bar = bind(foo, { a: 1 })

bar(2, 3) // 6
bar.call({ a: 10 }, 2, 3) // 6
```

> 事实上, 早在 `ES5` 就提供了原生的**硬绑定**方法: `Function.prototype.bind`

``` javascript
var bar = foo.bind({ a: 1 })

bar(2, 3)  // 6
bar.call({ a: 10 }, 2, 3)  // 6
```

> `Function.prototype.bind` 的实现:

``` javascript
Function.prototype._bind = function(obj) {
  var self = this
  return function() {
    return self.apply(obj, arguments)
  }
}
```

### new 绑定

> 构造函数只是一些使用 `new` 操作符时被调用的函数, 它们并不会属于某个类, 也不会实例化一个类, 它们只是被 `new` 操作符调用的普通函数而已

#### 普通函数

以下示例, `Person` 是一个所谓的构造函数, 根据**词法作用域**的规则, `Person` 被调用时, 内部的 `this` 应当指向**全局作用域**;

因此, 当我们访问 `person.name` 时, 会得到 `TypeError` 的结果, 因为 `Person()` 没有返回任何东西, 是个 `undefined`

``` javascript
function Person() {
  this.name = 'muzi'
}

const person = Person()

console.log(window.name) // muzi
console.log(person.name) // TypeError
```

可以看到, 没有被 `new` 操作符调用的所谓构造函数, 仅仅是普通函数而已

#### 构造函数

``` javascript
const person = new Person()

console.log(window.name)  // undefined
console.log(person.name)  // muzi
```

我们会得到截然不同的结果, 这是因为 `new` 操作符做了以下四件事情:

- 创建或构造了一个全新的对象
- 这个新对象会被执行[[原型]]连接
- 函数中的 `this` 会指向这个新对象
- 如果被调用的函数没有返回(return), 则 `new` 表达式中的函数调用会自动返回这个新对象

### 优先级

`new` > `显式绑定` > `隐式绑定` > `默认绑定`

## ES6 箭头函数

> 箭头函数不使用 `this` 的四种标准规则, 而是根据此法作用域来决定 `this`

``` javascript
var obj = { a: 1 }

// 1
function fn() {
    setTimeout(function() {
        console.log(this.a)
    }, 1000)
}

fn.call(obj)  // undefined

// 2
function arrow() {
    setTimeout(() => {
        console.log(this.a)
    }, 1000)
}

arrow.call(obj) // 1
```

### 代码片段1:
`fn` 的 `this` 指向 `obj`, `setTimeout` 的回调函数是个普通函数, 并且没有被特殊调用, 执行 `默认绑定` 规则, 其 `this` 指向全局作用域

> 这种情况下, 通常有两种方法, 可以让回调函数绑定到父级的 `this`

``` javascript
var obj = { a: 1 }

function fn() {
    var self = this
    setTimeout(function() {
        console.log(self.a)
    }, 1000)
}

fn.call(obj) // 1
```

``` javascript
var obj = { a: 1 }

function fn() {
    setTimeout(function() {
        console.log(this.a)
    }.bind(this), 1000)
}

fn.call(obj) // 1
```

### 代码片段2:
`arrow` 的 `this` 指向 `obj`, `setTimeout` 的回调函数是个箭头函数, 根据 **词法作用域**, 该箭头函数的 `this` 也指向 `obj`
