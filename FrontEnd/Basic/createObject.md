# JavaScript创建对象
> 《You Don't Know JS: this & object prototypes》

> [JavaScript创建对象—从es5到es6](http://www.renfed.com/2017/08/07/js-oop-es52es6/)

## 创建对象的方式
### Object构造函数和对象字面量
通过调用Object构造函数来new一个Object对象，通过赋值的方式来赋值对象的每一个属性。

```javascript
var animal = new Object();
animal.age = 1;
animal.name = 'doge';
animal.property = function() {
  console.log(`${this.age}, ${this.name}`);
};
animal.property(); // 1, doge
```
由于对象字面量创建对象的写法简单直观，Object构造函数法被其替代。

```javascript
var animal = {
  age: 1,
  name: 'doge',
  property: function() {
    console.log(`${this.age}, ${this.name}`);
  }
}
animal.property(); // 1, doge
```
这两个创建对象方式所共有的问题是：无法复用，违背的对象的封装性。

### 工厂模式
工厂模式可以将对象的创建封装到一个方法中，解决了复用性的问题。

```javascript
function createAnimal(age, name) { 
    var a = new Object();
    a.age = age;
    a.name = name;
    a.property = function() {
         console.log(`${this.age}, ${this.name}`);
    };
    return a;
}
var animal1 = createAnimal(1, 'doge');
var animal2 = createAnimal(2, 'dog');
animal1.property(); // 1, doge
animal2.property(); // 2, dog
```
但是工厂模式也有不足，构造出来的对象全是基于Object
`console.log(animal1 instanceof Object); // true`

### 构造函数模式

OO语言中最经典的构造函数，通过this给对象的属性和方法进行赋值。

```javascript
function Animal(age, name) {
  this.age = age;
  this.name = name;
  this.property = function () {
    console.log(`${this.age}, ${this.name}`)
  };
}
var animal1 = new Animal(1, 'doge')
animal1.property(); // 1, doge
console.log(animal1 instanceof Animal); // true
```

### 原型模式
原型模式是通过将所有的属性和方法都定义在其prototype属性上，达到这些属性和方法能被所有的实例所共享的目的。

```javascript
function Animal(age, name) { 
    Animal.prototype.age = age;
    Animal.prototype.name = name;
    Animal.prototype.property = function() {
         console.log(`${age}, ${name}`);
    };
}
 
var animal1 = new Animal(1, 'a');
var animal2 = new Animal(2, 'b');
animal1.property(); // 2, b
animal2.property(); // 2, b
```

当一个对象上的属性改变时，所有对象上的属性也会随之改变，所以这个方法无实际运用价值。

### ES5的实际运用代表：构造函数+原型组合模式

组合模式是将构造函数模式和原型模式结合在一起，继承了它们优点的同时又避免了各自的缺点。它将具有各自特点的属性和方法定义在构造函数中，将实例间共享的属性和方法定义在prototype上，成为了在es6出现之前使用最普遍的一种创建对象模式。

```javascript
function Animal(food) {
  this.food = food;
  this.eat = function() {
    console.log(`eat ${food}`);
  }
}
Animal.prototype = {
  constructor: Animal,
  say: function() {
    console.log(`I love ${this.food}`);
  }
}

var Cat = new Animal('fish');
Cat.eat(); // "eat fish"
Cat.say(); // "I love fish"

var Dog = new Animal('bone');
Dog.eat(); // "eat bone"
Dog.say(); // "I love bone"
```
### Class定义类
ES6的Class出现，让熟悉传统OO语言（Java等）的程序员更能接受，然而这只是个语法糖，让代码看起来更**面向对象编程语法**

```javascript
class Animal{
  constructor(food) {
    this.food = food;
    this.eat = function() {
      console.log(`eat ${food}`);
    }
  }
  say() {
    console.log(`I love ${this.food}`);
  }
}

const Cat = new Animal('fish');
Cat.eat(); // "eat fish"
Cat.say(); // "I love fish"

const Dog = new Animal('bone');
Dog.eat(); // "eat bone"
Dog.say(); // "I love bone"
```

Class上的方法都是定义在prototype上，这就和原型模式有些相似，这个class中的say方法等价于

```javascript
Animal.prototype.say = function() {
  console.log(`I love ${this.food}`);
  }
```

## ES6的class转化为ES5

**ES6代码就是上文的例子**

**使用babel转换为ES5代码**

```javascript
"use strict"; // ES5严格模式

var _createClass = (function() {
  function defineProperties(target, props) {
    for (var i = 0; i < props.length; i++) {
      var descriptor = props[i];
      // 默认不可枚举
      descriptor.enumerable = descriptor.enumerable || false;
      descriptor.configurable = true;
      if ("value" in descriptor) descriptor.writable = true;
      Object.defineProperty(target, descriptor.key, descriptor);
    }
  }
  return function(Constructor, protoProps, staticProps) {
    if (protoProps) defineProperties(Constructor.prototype, protoProps);
    if (staticProps) defineProperties(Constructor, staticProps);
    return Constructor;
  };
})();

// 判断是否为构造函数
function _classCallCheck(instance, Constructor) {
  if (!(instance instanceof Constructor)) {
    throw new TypeError("Cannot call a class as a function");
  }
}

// class转换为ES5 function
var Animal = (function() {
  function Animal(food) {
    // 调用_classCallCheck判断是否为构造函数
    _classCallCheck(this, Animal);
    this.food = food;
    this.eat = function() {
      console.log("eat " + food);
    };
  }
  // 调用_createClass来处理class中定义的方法
  _createClass(Animal, [
    {
      key: "say",
      value: function say() {
        console.log("I love " + this.food);
      }
    }
  ]);

  return Animal;
})();

var Cat = new Animal("fish");
Cat.eat(); // "eat fish"
Cat.say(); // "I love fish"

var Dog = new Animal("bone");
Dog.eat(); // "eat bone"
Dog.say(); // "I love bone"
```

## ES5与ES6定义对象的区别
>  class的构造函数需要用`new`调用，普通构造函数不用`new`也可执行。

因为class中的`constructor`会直接转换为function构造函数，然后在function中用过`_classCallCheck`来判断是否为构造函数，所以class必须要用`new`调用。

>  class不存在变量提升，ES5中的function存在变量提升。

因为是函数表达式声明，所以不存在变量提升。

>  class内部方法无法枚举，ES5在prototype上定义的方法可以枚举。

因为class中定义的方法会传入`_createClas`s中，然后 `Object.defineProperty`将其定义在`Constructor.prototype`上，因为在babel中
`descriptor.enumerable = descriptor.enumerable || false;`默认设置为不可枚举。