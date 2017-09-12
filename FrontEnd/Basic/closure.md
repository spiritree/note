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
  var n = 999;
    function f2() {
      console.log(n); //999
    }
    return f2;
}
var result = f1();
result(); //999
```
因为f2被包含在f1中，所以f2可以读取f1中的变量（JavaScript可以在函数内读取全局变量，f1是f2的相对全局作用域）
既然能读取，将f2作为返回值，这样就能在f1外部读取它内部变量。

## 如何将变量的值保存在内存中？
```javascript
function f1() {
  var n = 999;
  nAdd = function () {
    n += 1;
  }
  function f2() {   
    console.log(n);
  }
  return f2;
}
var result = f1();
result(); // 999
nAdd(); // 1000
result(); // 1000

/**
* 执行var result2 = f1()时，f1返回了f2函数的引用
* 它可以访问到f1()被调用时产生的环境，而局部变量n一直在这个环境中
* 既然局部变量还能被外部访问，这样就不用被销毁，完成+1s
*/
var result2 = f1();
result2(); // 999
```
由于nAdd是全局变量并且是匿名函数，所以nAdd类似于setter，可以在函数外部对函数内部的局部变量进行操作
因为f2被赋值一个全局变量，所以f2始终在内存中，又因f2的f1的父函数，f1也始终在内存中。

## 闭包与面向对象设计
面向对象的「对象」可以用过程与数据的结合来形容。对象以方法的形式包含了过程，而闭包则是在过程中以环境的形式包含了数据。
闭包也可以来实现面向对象设计
```javascipt
var closureOO = () => {
  let val = 0
  return {
    call() {
      val++;
      console.log(val);
    }
  }
};
var closureOO = closureOO();
closureOO.call(); // 1
closureOO.call(); // 2
```

## 闭包与内存管理
闭包会造成内存泄露？所以要减少闭包使用？？

局部变量本来应该在函数退出的时候解除引用，而使用闭包能让这个局部变量的生命周期得以延长。从这个意义上看，闭包会使一些数据无法被及时销毁。
使用闭包的一部分原因是我们选择主动把一些变量封闭在闭包中，因为可能在以后还需要使用这些变量，把这些变量放在闭包中和放在全局作用域，对内存方面的影响是一致的，这里不能说成是内存泄露。如果在将来需要回收这些变量，我们可以手动把这些变量设为`null`。

闭包和内存泄露有关系的地方是因为使用闭包容易形成循环引用，如果闭包的作用域链保存着DOM节点，就有可能造成内存泄露。而这本身是IE的COM对象的问题。
如果要解决循环引用带来的内存泄漏问题，只需把循环引用变量设为`null`即可。
## 概念与总结
闭包就是函数能够记住并访问它的词法作用域，即使当这个函数在它的词法作用域之外执行时。

闭包可以封装变量，延续局部变量的寿命。
