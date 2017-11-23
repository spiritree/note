# JavaScript中的this与function.prototype.call&apply&bind

## Types
> [ECMAScript 5.1规范的第八章](http://yanhaijing.com/es5/#71)

 
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
  food: 'fish',
  say: function() {
    console.log(`I love ${this.food}`);
  }
}
var blackCat = new cat;
blackCat.say(); // "I love fish"

var doge = { food: 'bone' };
blackCat.say.call(doge); // "I love bone"

var blackDoge = blackCat.say.bind(doge);
blackDoge(); // "I love bone"
```

## call&apply模拟实现

### 初步猜想
把对象改造成这样岂不是很ez

```javascript
var doge = {
  food: 'bone',
  say: function() {
    console.log(`I love ${this.food}`);
  }
}
doge.say(); // 'I love bone'
```
思路为：
1. 给对象添加新属性
2. 执行函数
3. 删除属性

```javascript
Function.prototype.call1 = function(context) {
  // 首先要获取调用call的函数，用this可以获取
  context.fn = this;
  context.fn();
  delete context.fn;
}
```

### 第二步
call 函数还能给定参数执行函数

```javascript
var dog = {
  food: 'bone'
};
function say(name, age) {
  console.log(name);
  console.log(age);
  console.log(this.food);
}
say.call(dog, 'doge', '2');
// 'doge'
// '2'
// 'bone'
```
传入参数的处理（从arguments对象取值）

```javascript
/** 
* arguments = {
* 0: dog,
* 1: 'doge',
* 2: '2'
* }
* arguments是类数组的对象，有length属性
*/ 
var args = [];
for(var i = 1 ; i < arguments.length; i++) {
    args.push('arguments[' + i + ']');
}
```

把这个参数数组放到要执行的函数的参数里面去

```javascript
// 第二版
Function.prototype.call2 = function(context) {
    context.fn = this;
    var args = [];
    for(var i = 1, ; i < arguments.length; i++) {
        args.push('arguments[' + i + ']');
    }
    eval('context.fn(' + args +')');
    delete context.fn;
}
var dog = {
  food: 'bone'
};
function say(name, age) {
  console.log(name);
  console.log(age);
  console.log(this.food);
}
say.call2(dog, 'doge', '2');
// 'doge'
// '2'
// 'bone'
```

### 第三步
到此为止还有两个小缺陷
1. this 参数可以传 null，当为 null 的时候，视为指向 window
2. 函数是可以有返回值的

```javascript
Function.prototype.call3 = function (context) {
    var context = context || window;
    context.fn = this;
    var args = [];
    for(var i = 1, i < arguments.length; i++) {
        args.push('arguments[' + i + ']');
    }
    var result = eval('context.fn(' + args +')');
    delete context.fn
    return result;
}
```

```javascript
// ES6
Function.prototype.call3 = function(context){
    var context = context || window;
	context.fn = this;
    var args = [];
	for(var i = 1, i < arguments.length; i++) {
	args.push(arguments[i])
    }
    context.fn(...args)
    delete context.fn
}
```

apply与call类似

```javascript
Function.prototype.apply = function (context, arr) {
    var context = Object(context) || window;
    context.fn = this;

    var result;
    if (!arr) {
        result = context.fn();
    }
    else {
        var args = [];
        for (var i = 0, len = arr.length; i < len; i++) {
            args.push('arr[' + i + ']');
        }
        result = eval('context.fn(' + args + ')')
    }

    delete context.fn
    return result;
}
```

## bind模拟实现

> [Function.prototype.bind()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

特点：
1.返回一个函数

```javascript
var dog = {
  food: 'bone'
};
function say() {
  console.log(this.food);
}
var bindDog = say.bind(dog);
bindDog(); // 'bone'
```

2.可以传入参数

```javascript
var dog = {
  food: 'bone'
};
function say(name, age) {
  console.log(name);
  console.log(age);
  console.log(this.food);
}
var bindDog = say.bind(dog, 'doge')
bindDog('2');
// doge
// 2
// 'bone'
```

3.构造函数效果

当 bind 返回的函数作为构造函数的时候，bind 时指定的 this 值会失效，但传入的参数依然生效。

```javascript
var food = 'fish';
var dog = {
  food: 'bone'
};
function say(name, age) {
  this.behavior = 'wang';
  console.log(name);
  console.log(age);
  console.log(this.food);
}
say.prototype.friend = 'cat';
var bindDog = say.bind(dog, 'doge');
var obj = new bindDog('2');
// 'doge'
// '2'
// undefined 因为new，此时this指向obj
console.log(obj.behavior) // 'wang'
console.log(obj.friend) // 'doge'
```

### 当我们实现过call/apply，bind就方便了

1.返回一个函数

```javascript
Function.prototype.bind1 = function (context) {
  var _this = this
  return function () {
    _this.apply(context)
  }
}
```

2.模拟传参

```javascript
Function.prototype.bind2 = function (context) {

    var _this = this;
    // 获取bind2函数从第二个参数到最后一个参数
    var args = Array.prototype.slice.call(arguments, 1);

    return function () {
        // 这个时候的arguments是指bind返回的函数传入的参数
        var bindArgs = Array.prototype.slice.call(arguments);
        _this.apply(context, args.concat(bindArgs));
    }

}
```

3.构造函数效果

```javascript
Function.prototype.bind3 = function (context) {
    var _this = this;
    var args = Array.prototype.slice.call(arguments, 1);

    var fBound = function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        // 当作为构造函数时，this 指向实例，此时结果为 true，将绑定函数的 this 指向该实例，可以让实例获得来自绑定函数的值
        // 以上面的是 demo 为例，如果改成 `this instanceof fBound ? null : context`，实例只是一个空对象，将 null 改成 this ，实例会具有 habit 属性
        // 当作为普通函数时，this 指向 window，此时结果为 false，将绑定函数的 this 指向 context
        _this.apply(this instanceof fBound ? this : context, args.concat(bindArgs));
    }
    // 修改返回函数的 prototype 为绑定函数的 prototype，实例就可以继承绑定函数的原型中的值
    fBound.prototype = this.prototype;
    return fBound;
}
```

我们直接将 fBound.prototype = this.prototype，我们直接修改 fBound.prototype 的时候，也会直接修改绑定函数的 prototype。这个时候，我们可以通过一个空函数来进行中转：

```javascript
Function.prototype.bind2 = function (context) {

    var _this = this;
    var args = Array.prototype.slice.call(arguments, 1);

    var fNOP = function () {};

    var fBound = function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        _this.apply(this instanceof fNOP ? this : context, args.concat(bindArgs));
    }

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();
    return fBound;
}
```

## 参考资料

> [this - JavaScript | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this)

> [Function.prototype.call()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)

> 《You Don't Know JS: this & object prototypes》

> [JavaScript深入之call和apply的模拟实现](https://github.com/mqyqingfeng/Blog/issues/11)

> [不用call和apply方法模拟实现ES5的bind方法](https://github.com/jawil/blog/issues/16)


