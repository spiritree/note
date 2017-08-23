# 再谈Javascript作用域与闭包

> 《JavaScript高级程序设计》

> 《You Don't Know JS: Scope & Closures》

> [学习Javascript闭包（Closure）](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)

## 作用域

谈闭包，首先需要理解JavaScript变量的作用域概念！

- 作用域链

子对象一级一级地向上寻找父对象的变量，一旦找到第一个匹配，作用域查询就停止了。

- 变量作用域

变量作用域只有两种：全局和局部变量
在JavaScript中，函数内可以读取全局变量，但函数外无法读取函数内的局部变量

<!--more-->

- 词法作用域

![zuoyongyu.png](https://ooo.0o0.ooo/2017/08/01/598056edb3897.png)

1. 全局作用域，标识符foo
2. function foo(a)作用域，标识符a，bar，b
3. function bar(c)作用域，标识符c

## 如何从外部读取内部变量？
```javascript
function f1() {
  var n = 999
    function f2() {
      console.log(n) //999
    }
    return f2
}
var result = f1()
result() //999
```
因为f2被包含在f1中，所以f2可以读取f1中的变量（JavaScript可以在函数内读取全局变量，f1是f2的相对全局作用域）
既然能读取，将f2作为返回值，这样就能在f1外部读取它内部变量。

## 如何将变量的值保存在内存中？
```javascript
function f1() {
  var n = 999
  nAdd = function () {
    n += 1
  }
  function f2() {   
    console.log(n)
  }
  return f2
}
var result = f1()
result() // 999
nAdd() // 1000
result() // 1000

// 定义另一个变量，这样会多一个变量存在内存中
var result2 = f1()
result2() // 999
```
由于nAdd是全局变量并且是匿名函数，所以nAdd类似于setter，可以在函数外部对函数内部的局部变量进行操作
因为f2被赋值一个全局变量，所以f2始终在内存中，又因f2的f1的父函数，f1也始终在内存中。

## 概念与总结
闭包就是函数能够记住并访问它的词法作用域，即使当这个函数在它的词法作用域之外执行时。

闭包可以读取函数内部的变量，还可以让变量的值始终保持在内存中。
