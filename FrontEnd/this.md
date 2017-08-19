> [this - JavaScript | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this)

> 《You Don't Know JS: this & object prototypes》

## 定义&调用点&调用栈
`this`定义：this引用的是函数赖以执行的环境对象

调用点：函数在代码中被调用的位置**（不是被声明的位置）**

调用栈：**使我们到达当前执行位置而被调用的所有方法的堆栈**


```javascript
function baz() {
    // 调用栈是: `baz`
    // 我们的调用点是 global scope（全局作用域）

    console.log( "baz" );
    bar(); // <-- `bar` 的调用点
}

function bar() {
    // 调用栈是: `baz` -> `bar`
    // 我们的调用点位于 `baz`

    console.log( "bar" );
    foo(); // <-- `foo` 的 调用点
}

function foo() {
    // 调用栈是: `baz` -> `bar` -> `foo`
    // 我们的调用点位于 `bar`

    console.log( "foo" );
}

baz(); // <-- `baz` 的调用点
```

## 判定 `this`

在函数执行时，this 总是指向调用该函数的对象。要判断 this 的指向，其实就是判断 this 所在的函数属于谁。

我们可以按照优先顺序来总结一下从函数调用的调用点来判定 this

1. 函数是通过 `new` 被调用的吗？如果是，`this` 就是新构建的对象。

    `var bar = new foo()`

2. 函数是通过 `call` 或 `apply` 被调用，甚至是隐藏在 `bind`  之中吗？如果是，`this` 就是那个被明确指定的对象。

    `var bar = foo.call( obj2 )`

3. 函数是通过环境对象（也称为拥有者或容器对象）被调用的吗？如果是，`this` 就是那个环境对象。

    `var bar = obj1.foo()`

4. 否则，使用默认的 `this`。如果在 `strict mode` 下，就是 `undefined`，否则是 `global` 对象。

    `var bar = foo()`
    
    
在实际运用中找this最简单的方法

1. 看文档找到call&apply&bind指定

2. `console.log(this)`

## 「绑定this」call&apply&bind

call和apply是为了动态改变this而出现的，当一个object没有某个方法，但是其他的object有，我们就可以借助call或apply用其它对象的方法来操作。

apply() 方法接受两个参数第一个是函数运行的作用域，另外一个是一个参数数组(arguments)。

call() 方法第一个参数的意义与 apply() 方法相同，只是其他的参数需要一个个列举出来。

简单来说，call 的方式更接近我们平时调用函数，而 apply 需要我们传递 Array 形式的数组给它。它们是可以互相转换的。

```javascript
function cat() {}
cat.prototype = {
  food: "fish",
  say: function() {
    console.log(`I love ${this.food}`);
  }
}
var blackCat = new cat;
blackCat.say(); // "I love fish"

var doge = { food: "bone" };
blackCat.say.call(doge); // "I love bone"

var blackDoge = blackCat.say.bind(doge);
blackDoge(); // "I love bone"
```