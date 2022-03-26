---
title: new运算符
toc: true
date: 2022-03-26 21:08:08
tags: [JavaScript]
urlname:
categories: [JavaScript]
recommend:
cover:
keywords: new运算符
top: 3
---

## new运算符

### new的过程

new 运算符创建一个用户定义的对象类型的实例或具有构造函数的内置对象的实例。

[官方文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new)

<!-- more -->

```js
function Person(name){
  this.name = name
}
let p = new Person("张三")
console.log(typeof p) //object
console.log(typeof Person) //function
console.log(p) //{ name:"张三" }
```

- 创建一个对象；

  ```js
  var obj = new Object()
  ```

- 将这个空对象的__proto__成员指向构造函数对象的prototype成员；

  ```js
  obj._proto_ = Person.prototype
  ```

- 将构造函数的this指向obj，即在obj的作用域中调用Person函数；

  ```js
  Person.call(obj);
  ```

- 返回一个新的对象。

如何new一个箭头函数的话，由于箭头函数没有自己的this，通过call()、apply()方法调用时，第一个参数会被忽略，箭头函数也没有prototype，所以new的第二和第三步就无法执行。

### 手写new

```js
function myNew(func,...args){
  //用Object.create以func.prototype问原型创建一个对象;
  const obj = Object.create(func.prototype)
  //将func的this指向obj
  const rel = fn.apply(obj,args)
  //返回对象（ fn函数一般不会有返回值（null或者undefined） ）
  return rel instanceof Object ? rel : obj
}
```

