```javascript
/*
* 作者：蔡东-uestc-2017届-cs
* 转载请注明出处并保留原文链接
*/
```
## JavaScript的设计模式

JavaScript具有面向对象的特性，所以面向对象的程序设计还是蛮重要的。
《JavaScript高级程序设计》P144把设计模式写的很详细，是一本特别好的书。
本文主要讲解以下模式，及其优缺点。

- 工厂模式
- 构造函数模式
- 原型模式
- 构造函数+原型模式
- 动态原型模式
    
## 工厂模式
最开始，我们要创建很多相同属性的对象，如果所有对象都要一个一个的赋值，会导致的问题有

- 生成很多新对象会花费大量时间做重复性赋值工作，效率低下
- 需要添加/删除/修改所有对象某一个属性，工作量仍然巨大，效率低下

于是我们就开始考虑把公共部分提取成函数来创建对象，这样只需要修改函数部分，就可以操作所有函数产生的对象的属性。
```javascript
//  非构造函数的函数名第一个字母小写
function person(name, age){
    var obj = {};
    obj.name = name;
    obj.age = age;
    obj.sayName = function(){
        console.log(this.name);
    }
    return obj;
}
var Jack = person('Jack', 23);
var Rose = person('Rose', 21);
console.log(Jack.sayName === Rose.sayName);  //  false
```
我们可以看出，工厂模式具备的优点

- 减少重复性代码量
- 利于扩展和修改

但是缺点还是比较明显的

- 每个实例的方法(sayName)都会创建一遍，每个对象的sayName都不同，因为函数也是对象，函数名是指针

## 构造函数模式
构造函数模式初版本，构造函数会经历的四个步骤

- 创建新对象
- 讲构造函数的作用域赋给新对象（this指向了这个对新象）
- 执行构造函数中的代码（为新对象添加属性）
- 返回新对象

```javascript
//  构造函数的函数名第一个字母大写
function Person(name, age){
    this.name = name;
    this.age = age;
    this.sayName = function(){
        console.log(this.name);
    }
}
var Jack = new Person('Jack', 23);
var Rose = new Person('Rose', 21);
console.log(Jack.sayName === Rose.sayName);  //  false
```
与工厂模式比较

- 没有显示创建对象
- 属性和方法赋值给this
- 没有return语句
- 实例需要使用new操作符

新对象的实例的constructor属性指向了构造函数Person，同时引用类型也指向了Person
```javascript
console.log(Jack.constructor == Person)  //  true
console.log(Rose.constructor == Person)  //  true
console.log(Jack instanceof  Person)  //  true
console.log(Rose instanceof == Person)  //  true
```
但是仍然具备了工厂模式该有的缺点，于是产生了升级版构造函数模式
```javascript
function Person(name, age){
    this.name = name;
    this.age = age;
    this.sayName = sayName;
}
function sayName(){
    console.log(this.name);
}
var Jack = new Person('Jack', 23);
var Rose = new Person('Rose', 21);
console.log(Jack.sayName === Rose.sayName);  //  true
```
这样的优点

- 解决了工厂模式的问题，每个实例的方法不会重复创建了

不过缺点还是出现了

- 实例方法是全局函数了，这样可能会不注意影响到其他的全局函数
- 本来只能被对象调用的方法，现在可以直接调用
- 如果给对象添加很多不同的方法，那么需要注册很多全局函数

不过好在共享方法且不是全局函数这个问题，原型模式可以解决。
## 原型模式
原型模式初版本
```javascript
function Person(){}
Person.prototype.name = 'Jack';
Person.prototype.age = 23;
Person.prototype.sayName = function(){
    console.log(this.name);
}
var per1 = new Person();
var per2 = new Person();
console.log(per1.sayName === per2.sayName);  //  true
per2.name = 'Rose'
console.log(per2.name);  //  Rose
console.log(per1.name);  //  Jack
```
通过对象的prototype指针，可以实现共享方法且不是全局函数，并且扩展自己的属性方便，但是仍然存在一个问题
```javascript
function Person(){}
Person.prototype.arr = [1, 2, 3]
var Jack = new Person();
var Rose = new Person();
Rose.arr.push(4);
console.log(Jack.arr);  //  [1, 2, 3, 4]
console.log(Rose.arr);  //  [1, 2, 3, 4]
console.log(Jack.arr === Rose.arr);  //  true
```
我们发现了，如果共享的数据是对象（包括数组、函数等等引用类型），改变其中一个都会改变所有新对象的属性值，因为他们储存的是相同的指针，指针地址的数据变化之后，仍然跟着变化。

根据以上我们可以知道，原型模式的优点

- 能够共享方法且不是全局函数，保护了对象方法的特性

缺点显而易见

- 如果对象的属性值是对象（指针），改变一个对象会改变全部对象的属性值

不过我们可以发现，构造函数模式和原型模式似乎可以互补。
## 构造函数+原型模式
根据构造函数模式和原型模式的取长补短产生了新模式
```javascript
function Person(name, age, arr){
    this.name = name;
    this.age = age;
    this.arr = [1, 2, 3];
}
Person.prototype = {
    constructor: Person,
    sayName: function(){
        console.log(this.name);
    }
}
var Jack = new Person('Jack', 23);
var Rose = new Person('Rose', 21);
console.log(Jack.sayName === Rose.sayName);  //  true
Rose.arr.push(4);
console.log(Jack.arr);  //  [1, 2, 3]
console.log(Rose.arr);  //  [1, 2, 3, 4]
console.log(Jack.arr === Rose.arr);  //  false
```
优点

- 需要共享的方法可以实现共享，并不是全局函数，最大化的节省内存
- 新对象自己也可以拥有单独的对象数据

缺点

- 不能把所有的信息都封装在构造函数中，不能通过构造函数初始化原型

## 动态原型模式
其他面向对象语言程序员喜欢的模式
```javascript
function Person(name, age, arr){
    this.name = name;
    this.age = age;
    this.arr = [1, 2, 3];
    if(this.sayName != "function"){
        Person.prototype.sayName = function(){
            console.log(this.name);
        }
    }
}
var Jack = new Person('Jack', 23);
var Rose = new Person('Rose', 21);
console.log(Jack.sayName === Rose.sayName);  //  true
Rose.arr.push(4);
console.log(Jack.arr);  //  [1, 2, 3]
console.log(Rose.arr);  //  [1, 2, 3, 4]
console.log(Jack.arr === Rose.arr);  //  false
```
优点

- 所有信息都封装在构造函数里面，原型可以在构造函数中初始化

缺点

- 不能使用对象字面量重写原型，如果创建了实例的情况下重写原型，会切断实例与原型的联系
    
作者：微博 [@itagn][1] - Github [@itagn][2]

[1]: https://weibo.com/p/1005053782707172
[2]: https://github.com/itagn




