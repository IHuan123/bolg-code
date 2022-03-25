---
title: JavaScript判断类型的几种方式
toc: true
date: 2022-03-25 09:52:30
tags: [JavaSciprt]
urlname:
categories: [JavaSciprt]
recommend: 
cover: 
keywords: JavaScript判断类型的几种方式
top: 3
---

## JavaScript判断类型的几种方式

### 1、js中的数据类型

JavaScript有八种数据类型: Undefined、Null、Boolean、Number、String、Object、Symbol、BigInt。
<!-- more -->

其中Symbol和BigInt是ES6中新增的数据类型：

- Symbol 可以创建独一无二且不可变的数据类型，主要为了解决全局变量冲突的问题。
- BigInt 是一种数字类型的数据，可以表示任意精度格式的整数，使用 BigInt 可以安全地存储和操作大整数，即使这个数已经超出了 Number 能够表示的安全整数范围。

基本数据类型：Undefined、Null、Boolean、Number、String、Symbol、BigInt。
 引用数据类型 ：Object（JS中除了基本类型以外都是对象,数组,函数,正则表达式）。


### 2、typeof

```js
let bool = false,
    num = 0,
    str = "hello,world",
    u = undefined,
    n = null,
    arr = [],
    obj = {},
    func = function(){},
    s = Symbol()
console.log(typeof bool);  //boolean
console.log(typeof num);   //number
console.log(typeof str);   //string
console.log(typeof u);     //undefined
console.log(typeof n);     //object
console.log(typeof arr);   //object
console.log(typeof obj);   //object
console.log(typeof fun);   //function
console.log(typeof s);     //symbol
```

typeof除了基本类型boolean,number,undefined,string,symbol和function可以正确识别以外 ，Array、Object、null都为对象。

### 3、constructor

```js
console.log(bool.constructor === Boolean);// true
console.log(num.constructor === Number);// true
console.log(str.constructor === String);// true
console.log(arr.constructor === Array);// true
console.log(obj.constructor === Object);// true
console.log(fun.constructor === Function);// true
console.log(s.constructor === Symbol);//true
```

null、undefined没有construstor方法，因此constructor不能判断undefined和null。
 但是他是不安全的，因为contructor的指向是可以被改变。

### 4、instanceof

```jsx
console.log(bool instanceof Boolean);  // false
console.log(num instanceof Number);    // false
console.log(str instanceof String);    // false
console.log(u instanceof Object);      // false
console.log(n instanceof Object);      // false
console.log(arr instanceof Array);     // true
console.log(obj instanceof Object);    // true
console.log(fun instanceof Function);  // true
console.log(s instanceof Symbol);      // false
```

从结果中看出instanceof不能识别出基本的数据类型 number、boolean、string、undefined、unll、symbol。
 但是可以检测出引用类型，如array、object、function，同时对于是使用new声明的类型，它还可以检测出多层继承关系。
 其实也很好理解，js的继承都是采用原型链来继承的。比如objA instanceof A ，其实就是看objA的原型链上是否有A的原型，而A的原型上保留A的constructor属性。
 所以instanceof一般用来检测对象类型，以及继承关系。

### 5、Object.prototype.toString.call

```js
console.log(Object.prototype.toString.call(bool));//[object Boolean]
console.log(Object.prototype.toString.call(num));//[object Number]
console.log(Object.prototype.toString.call(str));//[object String]
console.log(Object.prototype.toString.call(u));//[object Undefined]
console.log(Object.prototype.toString.call(n));//[object Null]
console.log(Object.prototype.toString.call(arr));//[object Array]
console.log(Object.prototype.toString.call(obj));//[object Object]
console.log(Object.prototype.toString.call(fun));//[object Function]
console.log(Object.prototype.toString.call(s)); //[object Symbol]
```


在项目中常用的就是此方法，使用哪个判断，还是要看使用场景，一般基本的类型可以选择typeof，引用类型可以使用instanceof。

#### toString、obj.toString()

同样是检测对象obj调用toString方法，obj.toString()的结果和Object.prototype.toString.call(obj)的结果不一样，这是为什么？

这是因为toString是Object的原型方法，而Array、function等**类型作为Object的实例，都重写了toString方法**。不同的对象类型调用toString方法时，根据原型链的知识，调用的是对应的重写之后的toString方法（function类型返回内容为函数体的字符串，Array类型返回元素组成的字符串…），而不会去调用Object上原型toString方法（返回对象的具体类型），所以采用obj.toString()不能得到其对象类型，只能将obj转换为字符串类型；因此，在想要得到对象的具体类型时，应该调用Object原型上的toString方法。