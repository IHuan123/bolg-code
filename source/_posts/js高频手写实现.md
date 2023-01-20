---
title: 高频手写实现
toc: true
date: 2022-07-09 16:52
tags: [JavaScript]
urlname:
categories: [JavaScript]
recommend:
cover:
keywords:  高频手写实现
top: 3

---

# instanceof

> `instanceof`运算符用于检查构造函数的`prototype`是否在某个实例对象中存在。

所以只要遍历实例对象的原型链，挨个往上查找看是否有与Fn的`prototype`相等的原型，直到最顶层`Object`还找不到，那么就返回false。

<!-- more -->

## 实现

### 遍历实现

```js
    function _instanceof(custructor, obj) {
          // 必须是对象或者函数 
        if (!(obj && ['object', 'function'].includes(typeof obj))) {
            return false
        }
        let prototype = custructor.prototype
        let __proto__ = obj
        while (__proto__ = Object.getPrototypeOf(__proto__)) {
            if (__proto__ === prototype) {
                return true
            }
        }
        return false
    }
```

### 递归实现

```js
    function _instanceof(custructor,obj){
        if (!(obj && ['object', 'function'].includes(typeof obj))) {
            return false
        }
        let proto = Object.getPrototypeOf(obj);
        let prototype = custructor.prototype;
        if(proto === null) return false;
        if(proto === prototype) return true;
        return _instanceof(custructor, proto)
    }
```

# JSON.stringify

`JSON.stringify()`将值转换为相应的 JSON 格式：

- 转换值如果有 toJSON() 方法，该方法定义什么值将被序列化。
- 非数组对象的属性不能保证以特定的顺序出现在序列化后的字符串中。
- 布尔值、数字、字符串的包装对象在序列化过程中会自动转换成对应的原始值。
- `undefined`、任意的函数以及 symbol 值，在序列化过程中会被忽略（出现在非数组对象的属性值中时）或者被转换成 `null`（出现在数组中时）。函数、undefined 被单独转换时，会返回 undefined，如`JSON.stringify(function(){})` or `JSON.stringify(undefined)`.
- 对包含循环引用的对象（对象之间相互引用，形成无限循环）执行此方法，会抛出错误。
- 所有以 symbol 为属性键的属性都会被完全忽略掉，即便 `replacer` 参数中强制指定包含了它们。
- Date 日期调用了 toJSON() 将其转换为了 string 字符串（同 Date.toISOString()），因此会被当做字符串处理。
- NaN 和 Infinity 格式的数值及 null 都会被当做 null。
- 其他类型的对象，包括 Map/Set/WeakMap/WeakSet，仅会序列化可枚举的属性。

```js
if (!window.JSON) {
    JSON = {
        parse: function(jsonStr) {
            return eval('(' + jsonStr + ')');
        },
        stringify: function(jsonObj) {
            var result = '',
                curVal;
            if (jsonObj === null) {
                return String(jsonObj);
            }
            //处理引用数据
            switch (Object.prototype.toString.call(jsonObj)) {
                case '[object Array]':
                    result += '[';
                    for (var i = 0, len = jsonObj.length; i < len; i++) {
                        curVal = JSON.stringify(jsonObj[i]); // 递归处理子集
                        result += (curVal === undefined ? null : curVal) + ",";
                    }
                    if (result !== '[') {
                        result = result.slice(0, -1);
                    }
                    result += ']';
                    return result;
                case '[object Date]':
                    return '"' + (jsonObj.toJSON ? jsonObj.toJSON() : jsonObj.toString()) +'"';
                case '[object RegExp]':
                    return "{}";
                case '[object Object]':
                    result += '{';
                    for (i in jsonObj) {
                        if (jsonObj.hasOwnProperty(i)) {
                            curVal = JSON.stringify(jsonObj[i]);
                            if (curVal !== undefined) {
                                result += '"' + i + '":' + curVal + ',';
                            }
                        }
                    }
                    if (result !== '{') {
                        result = result.slice(0, -1);
                    }
                    result += '}';
                    return result;

                case '[object String]':
                    return '"' + jsonObj.toString() + '"';
                case '[object Number]':
                	if(!isNaN(jsonObj)) return jsonObj
                case '[object Boolean]':
                    return jsonObj.toString();
            }
        }
    };
}
```

# Promise

```js
// 判断变量否为function
const isFunction = (variable) => typeof variable === "function";
// 定义Promise的三种状态常量
// promise状态只能在resolve与reject中改变且只改变一次
// 添加状态（只能由PENDING->FULFILLED/REJECTED ,不可逆）
const PENDING = "PENDING"; // 执行中
const FULFILLED = "FULFILLED"; // 成功
const REJECTED = "REJECTED"; // 失败

class MyPromise {
  _status = PENDING;
  // 添加返回结果
  _value = undefined;
  // 添加成功回调函数队列
  _fulfilledQueues = [];
  // 添加失败回调函数队列
  _rejectedQueues = [];

  constructor(handle) {
    if (!isFunction(handle)) {
      throw new Error("MyPromise must accept a function as a parameter");
    }
    // 执行handle
    try {
      // class中的function的prototype为undefind，需要bind处理
      handle(this._resolve.bind(this), this._reject.bind(this));
    } catch (err) {
      this._reject(err);
    }
  }
  // 添加resovle时执行的函数
  _resolve(val) {
    const run = () => {
      if (this._status !== PENDING) return;
      this._status = FULFILLED; // 在status为pending时改变状态

      // 依次执行成功队列中的函数，并清空队列
      const runFulfilled = (value) => {
        let cb;
        while ((cb = this._fulfilledQueues.shift())) {
          cb(value);
        }
      };
      // 依次执行失败队列中的函数，并清空队列
      const runRejected = (error) => {
        let cb;
        while ((cb = this._rejectedQueues.shift())) {
          cb(error);
        }
      };
      /* 如果resolve的参数为Promise对象，则必须等待该Promise对象状态改变后,
               当前Promsie的状态才会改变，且状态取决于参数Promsie对象的状态
             */
      if (val instanceof MyPromise) {
        val.then(
          (value) => {
            this._value = value;
            runFulfilled(value);
          },
          (err) => {
            this._value = err;
            runRejected(err);
          }
        );
      } else {
        this._value = val;
        runFulfilled(val);
      }
    };
    // 为了支持同步的Promise，这里采用异步调用
    setTimeout(run, 0);
  }
  // 添加reject时执行的函数
  _reject(err) {
    if (this._status !== PENDING) return;
    // 依次执行失败队列中的函数，并清空队列
    const run = () => {
      this._status = REJECTED; // 在status为pending时改变状态
      this._value = err;
      let cb;
      while ((cb = this._rejectedQueues.shift())) {
        cb(err);
      }
    };
    // 为了支持同步的Promise，这里采用异步调用
    setTimeout(run, 0);
  }
  // 添加then方法
  then(onFulfilled, onRejected) {
    const { _value, _status } = this;
    // then返回一个新的Promise对象
    return new MyPromise((onFulfilledNext, onRejectedNext) => {
      // 封装一个成功时执行的函数
      let fulfilled = (value) => {
        try {
          if (!isFunction(onFulfilled)) {
            onFulfilledNext(value);
          } else {
            let res = onFulfilled(value); //这里返回.then中的结果 传入下一个then
            //判断then中的函数是否为Promise
            if (res instanceof MyPromise) {
              // 如果当前回调函数返回MyPromise对象，必须等待其状态改变后在执行下一个回调
              res.then(onFulfilledNext, onRejectedNext);
            } else {
              // 否则会将返回结果直接作为参数，传入下一个then的回调函数，并立即执行下一个then的回调函数
              onFulfilledNext(res);
            }
          }
        } catch (err) {
          // 如果函数执行出错，新的Promise对象的状态为失败
          onRejectedNext(err);
        }
      };

      // 封装一个失败时执行的函数
      let rejected = (error) => {
        try {
          if (!isFunction(onRejected)) {
            onRejectedNext(error);
          } else {
            let res = onRejected(error);
            if (res instanceof MyPromise) {
              // 如果当前回调函数返回MyPromise对象，必须等待其状态改变后在执行下一个回调
              res.then(onFulfilledNext, onRejectedNext);
            } else {
              // 否则会将返回结果直接作为参数，传入下一个then的回调函数，并立即执行下一个then的回调函数
              onFulfilledNext(res);
            }
          }
        } catch (err) {
          // 如果函数执行出错，新的Promise对象的状态为失败
          onRejectedNext(err);
        }
      };
      switch (_status) {
        // 当状态为pending时，将then方法回调函数加入执行队列等待执行
        case PENDING:
          this._fulfilledQueues.push(fulfilled);
          this._rejectedQueues.push(rejected);
          break;
        // 当状态已经改变时，立即执行对应的回调函数
        case FULFILLED:
          fulfilled(_value);
          break;
        case REJECTED:
          rejected(_value);
          break;
      }
    });
  }
  // 添加catch方法
  catch(onRejected) {
    return this.then(undefined, onRejected);
  }

  // 添加静态resolve方法
  static resolve(value) {
    // 如果参数是MyPromise实例，直接返回这个实例
    if (value instanceof MyPromise) return value;
    return new MyPromise((resolve) => resolve(value));
  }
  // 添加静态reject方法
  static reject(value) {
    return new MyPromise((resolve, reject) => reject(value));
  }
  // 添加静态all方法
  static all(list) {
    return new MyPromise((resolve, reject) => {
      /**
       * 返回值的集合
       */
      let values = [];
      let count = 0;
      for (let [i, p] of list.entries()) {
        // 数组参数如果不是MyPromise实例，先调用MyPromise.resolve
        this.resolve(p).then(
          (res) => {
            values[i] = res;
            count++;
            // 所有状态都变成fulfilled时返回的MyPromise状态就变成fulfilled
            if (count === list.length) resolve(values);
          },
          (err) => {
            // 有一个被rejected时返回的MyPromise状态就变成rejected
            reject(err);
          }
        );
      }
    });
  }
  // 添加静态race方法
  static race(list) {
    return new MyPromise((resolve, reject) => {
      for (let p of list) {
        // 只要有一个实例率先改变状态，新的MyPromise的状态就跟着改变
        this.resolve(p).then(
          (res) => {
            resolve(res);
          },
          (err) => {
            reject(err);
          }
        );
      }
    });
  }
  // 不管成功与失败都执行
  finally(cb) {
    return this.then(
      (value) => MyPromise.resolve(cb()).then(() => value),
      (reason) =>
        MyPromise.resolve(cb()).then(() => {
          throw reason;
        })
    );
  }
}
```

# [Promise.all](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)

Promise.all() 方法接收一个 promise 的 iterable 类型（注：Array，Map，Set 都属于 ES6 的 iterable 类型）的输入，并且只返回一个[`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)实例， 那个输入的所有 promise 的 resolve 回调的结果是一个数组。这个[`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)的 resolve 回调执行是在所有输入的 promise 的 resolve 回调都结束，或者输入的 iterable 里没有 promise 了的时候。它的 reject 回调执行是，只要任何一个输入的 promise 的 reject 回调执行或者输入不合法的 promise 就会立即抛出错误，并且 reject 的是第一个抛出的错误信息。

**Promise.resolve(value)**方法返回一个以给定值解析后的 [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 对象。如果这个值是一个 promise ，那么将返回这个 promise ；如果这个值是 thenable（即带有 [`"then" `](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/then)方法），返回的 promise 会“跟随”这个 thenable 的对象，采用它的最终状态；否则返回的 promise 将以此值完成。此函数将类 promise 对象的多层嵌套展平。

```js
    Promise.myAll = function (promises) {
        return new Promise((resolve, reject) => {
            let values = [], len = promises.length, count = 0
            for (let i = 0; i < len; i++) {
                let item = promises[i]
                
                Promise.resolve(item).then(res=>{
                    count+=1
                    values[i] = res
                    if(count===len) resolve(values)
                }).catch(reject)
            }
        })
    }
```

# Promise.race

**`Promise.race(iterable)`** 方法返回一个 promise，一旦迭代器中的某个 promise 解决或拒绝，返回的 promise 就会解决或拒绝。

```js
Promise.myRace = (promises) => {
  // 返回一个新的Promise
  return new Promise((rs, rj) => {
    promises.forEach((p) => {
      // 包装一下promises中的项，防止非Promise .then出错
      // 只要有任意一个完成了或者拒绝了，race也就结束了
      Promise.resolve(p).then(rs).catch(rj)
    })
  })
}

```

# 深拷贝

```js
//对象深克隆
function deepClone(obj){
    //处理obj为正则、原始数据、
    if(typeof obj !== 'object') return obj;
    if(obj instanceof RegExp) return new RegExp(obj);
    if(obj instanceof Date) return new Date(obj);
    if(typeof obj === 'symbol') return Symbol(obj);
    
    let newObj = new obj.constructor();
    for(let key in obj){
        if(obj.hasOwnProperty(key)){
            newObj[key] = deepClone(obj[key])
        }
    }
    return newObj;
}
```

# 实现new操作符

[原理及实现](https://bobodog.cn/2022/03/26/new%E8%BF%90%E7%AE%97%E7%AC%A6/#new%E8%BF%90%E7%AE%97%E7%AC%A6)

# call、apply

## call

### [参数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call#%E5%8F%82%E6%95%B0)

thisArg: 可选的。在 *`function`* 函数运行时使用的 `this` 值。请注意，`this`可能不是该方法看到的实际值：如果这个函数处于[非严格模式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode)下，则指定为 `null` 或 `undefined` 时会自动替换为指向全局对象，原始值会被包装。没有传递它的第一个参数。如果没有传递第一个参数，`this` 的值将会被绑定为全局对象。

**备注：**在严格模式下，`this` 的值将会是 `undefined`。见下文。

### [返回值](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call#返回值)

使用调用者提供的 `this` 值和参数调用该函数的返回值。若该方法没有返回值，则返回 `undefined`。



```javascript
    Function.prototype.myCall = function(ctx, ...args){
         // 简单处理未传ctx上下文，或者传的是null和undefined等场景
        if (!ctx) {
            ctx = typeof window !== 'undefined' ? window : global
        }
        let fnName = Symbol()
        let obj = Object(ctx)
        obj[fnName] = this;
        let res = obj[fnName](...args)
        delete obj[fnName]
        return res
    }
```

## apply

该方法的语法和作用与 [`call`](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FFunction%2Fapply) 方法类似，只有一个区别，就是 `call` 方法接受的是**一个参数列表**，而 `apply` 方法接受的是**一个包含多个参数的数组**。

```js
Function.prototype.myApply = function(ctx, args=[]){
     // 简单处理未传ctx上下文，或者传的是null和undefined等场景
    if (!ctx) {
        ctx = typeof window !== 'undefined' ? window : global
    }
    let fnName = Symbol()
    let obj = Object(ctx)
    obj[fnName] = this;
    let res = obj[fnName](...args)
    delete obj[fnName]
    return res
}
```

# trim

**`trim`** 方法会从一个字符串的两端删除空白字符。在这个上下文中的空白字符是所有的空白字符 (space, tab, no-break space 等) 以及所有行终止符字符（如 LF，CR等）

```js
  // 方法一 去除空格法(方式1)
	String.prototype.trim = function () {
    return this.replace(/^[\s\uFEFF\xA0]+|[\s\uFEFF\xA0]+$/g, '');
  };
	// 方法二 去除空格法(方式1)
	const trim = (str) => { 
    return str.replace(/^\s*|\s*$/g, '')
  }
  // 方法三 字符提取法(方式2)
  const trim = (str) => { 
    return str.replace(/^\s*(.*?)\s*$/g, '$1') 
  } 

```

# Object.create

**`Object.create()`** 方法用于创建一个新对象，使用现有的对象来作为新创建对象的原型（prototype）。

## 参数

proto

新创建对象的原型对象。

`propertiesObject` 可选

如果该参数被指定且不为 [`undefined`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)，则该传入对象的自有可枚举属性（即其自身定义的属性，而不是其原型链上的枚举属性）将为新创建的对象添加指定的属性值和对应的属性描述符。这些属性对应于 [`Object.defineProperties()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties) 的第二个参数。

### [返回值](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create#返回值)

一个新对象，带着指定的原型对象及其属性。

```js
   	function ObjectCreate(proto, propertiesObject){
        if (![ 'object', 'function' ].includes(typeof proto)) {
            throw new TypeError(`Object prototype may only be an Object or null: ${proto}`)
        }
        let Ctor = new Function()
        Ctor.prototype = proto
        const res = new Ctor()
        // 支持第二个参数
        if (propertiesObject) {
            // Object.defineProperty的批量处理方法
            Object.defineProperties(res, propertiesObject)
        }
          // 支持空原型
        if (proto === null) {
            res.__proto__ = null
        }
        return res
    }
```

# reduce

**`reduce()`** 方法对数组中的每个元素按序执行一个由您提供的 **reducer** 函数，每一次运行 **reducer** 会将先前元素的计算结果作为参数传入，最后将其结果汇总为单个返回值。

第一次执行回调函数时，不存在“上一次的计算结果”。如果需要回调函数从数组索引为 0 的元素开始执行，则需要传递初始值。否则，数组索引为 0 的元素将被作为初始值 *initialValue*，迭代器将从第二个元素开始执行（索引为 1 而不是 0）。

### [参数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce#参数)

- `callbackFn`

  一个 “reducer” 函数，包含四个参数：`previousValue`：上一次调用 `callbackFn` 时的返回值。在第一次调用时，若指定了初始值 `initialValue`，其值则为 `initialValue`，否则为数组索引为 0 的元素 `array[0]`。`currentValue`：数组中正在处理的元素。在第一次调用时，若指定了初始值 `initialValue`，其值则为数组索引为 0 的元素 `array[0]`，否则为 `array[1]`。`currentIndex`：数组中正在处理的元素的索引。若指定了初始值 `initialValue`，则起始索引号为 0，否则从索引 1 起始。`array`：用于遍历的数组。

- `initialValue` 可选

  作为第一次调用 `callback` 函数时参数 *previousValue* 的值。若指定了初始值 `initialValue`，则 `currentValue` 则将使用数组第一个元素；否则 `previousValue` 将使用数组第一个元素，而 `currentValue` 将使用数组第二个元素。

```js
    Array.prototype.myReduce = function(callbackFn, initialValue){
        console.log(this)
        if (typeof callbackFn !== 'function') {
            throw `${callbackFn} is not a function`
        }
        let previousValue = this[0]
        , len = this.length
        , i = 1 ;
        if(initialValue){
            previousValue = initialValue
            i = 0
        }
        for(; i < len; i++){
            previousValue = callbackFn(previousValue, this[i] ,i , this)
        }
        return previousValue
    }
```

# 柯里化

```js
   	// 一
		function compose(...fns){
        let len = fns.length;
        fns = [].slice.apply(fns)
        return function(...args){
            if (len===0) return args
            if (len===1) return fns[0](...args)
            return fns.reduce((previousFn,currentFn)=>{
                if(typeof previousFn === 'function'){
                    return currentFn(previousFn(...args))
                }else{
                    return currentFn(...args)
                }
            })
        }
    }
    // 二
    function compose2(...funcs) {
        if (funcs.length ===0) {
            return arg => arg
        }
        if (funcs.length ===1) {
            return funcs[0]
        }
        return funcs.reduce((a, b) => (...args) => b(a(...args)));
    }
```

# 数组去重

```js
/** 数组去重**/
let arr = [1,13,464,1311,3,4,1,1,474,6,4,317,3335,4,444,55,55];
let arr2 =[1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN','NaN', 0, 0, 'a', 'a',{name:'zhangsan'}, {name:'zhangsan'},{name:'李四'}]
/**一 */
// 通过arr.indexOf === i来判断
function unique1(arr){
    let res = [];
    for(let i=0;i<arr.length;i++){
        if(arr.indexOf(arr[i])===i){
            res.push(arr[i])
        }
    }
    return res;
}
//2 arr.indexOf(NaN)总是-1 ，不能找到数组中的NaN
//[1, "true", 15, false, undefined, NaN, NaN, "NaN", "a", {…}, {…}]     //NaN和object没有去重


/**二 */
// 通过es6 Set
function unique2(arr){
    return Array.from(new Set(arr));
}
//在Set中NaN===NaN
// [1, "true", true, 15, false, undefined, null, NaN, "NaN", 0, "a", {…}, {…}]  //object没有去重

/**三 */
// 利用for嵌套for，然后splice去重（ES5中最常用）
function unique3(arr){
    for(var i=0;i<arr.length;i++){
        for(var j = i+1;j<arr.length;j++){
            if(arr[i]===arr[j]){
                arr.splice(j,1)
                j--
            }
        }
    }
    return arr
}
// [0, 1, 15, "NaN", NaN, NaN, {…}, {…}, "a", false, null, true, "true", undefined]      //NaN、object没有去重


/**四 */
//利用对象的属性不能相同的特点进行去重（这种数组去重的方法有问题，不建议用，有待改进）
function unique4(arr){
    if(!Array.isArray(arr)) return arr;
    var res = [],obj = {};
    for(var i = 0;i<arr.length;i++){
        if(!obj[arr[i]]){
            res.push(arr[i]);
            obj[arr[i]] = true;
        }
    }
    return res;
}

/**五 */
//利用includes
function unique5(arr){
    let res = [];
    for(var i=0;i<arr.length;i++){
        if(!res.includes(arr[i])){
            res.push(arr[i])
        }
    }
    return res;
}// [1, "true", true, 15, false, undefined, null, NaN, "NaN", 0, "a", {…}, {…}]     //object没有去重

/**六 */
//利用hasOwnProperty
function unique6(arr){
    let res = [],obj = {};
    for(var i=0;i<arr.length;i++){
        if(!obj.hasOwnProperty(typeof arr[i] + arr[i])){
            obj[typeof arr[i] + arr[i]] = true;
            res.push(arr[i])
        }
    }
    return res
}// 所有的都去重了（问题：object不管相不相等都会去重）


let res = unique6(arr2)
console.log(res,arr2.length,res.length)
```

# 多维数组扁平化

### Array.flat()

flat() 方法是ES10提出的，它会按照一个可指定的深度递归遍历数组，并将所有元素与遍历到的子数组中的元素合并为一个新数组返回。（flat意为“水平的；平坦的”）

```js
arr.flat(Infinity) // 指定深度为无限
```

### 递归方式

使用reduce拿到数组的当前值和前一项值，判断当前值是否为数组，初始值设置为[]，然后使用concat进行数组合并。

```js
    const { isArray } = Array
    const flat = (arr) => {
        if (isArray(arr)) {
            return arr.reduce((item, it) => {
                return item.concat(Array.isArray(it) ? flat(it) : it)
            }, [])
        }
    }
```

### 使用函数递归

循环遍历数组，发现含有数组元素就进行递归处理，最终将数组转为一维数组。

```js
const result = []
function exec(arr) {
  arr.forEach(item => {
    if (Array.isArray(item)) {
      exec(item)
    } else {
      result.push(item)
    }
  })
}
```

### 使用扩展运算符+concat()

ES6新推出的扩展运算符能对数组进行降维处理（一次降一维），循环判断是否含有数组，进行concat合并。

```js
function flatten(arr) {
  while (arr.some(item => Array.isArray(item))) {
    arr = [].concat(...arr)
  }
  return arr
}
```

# 提取两个数组的相同元素

```ts
    function getCommon<T>(arr1:Array<T>,arr2:Array<T>):Array<T>{
        let res:Array<T> = [], len1:number = arr1.length
        for(let i=0;i<len1;i++){
            let item = arr1[i]
            if(arr2.includes(item)) res.push(item)
        }
        return res
    }
```

# 数组排序

```js
let arr = [12, 4, 31, 65, 44, 6, 132, 1, 34, 65, 4, 64, 3, 13, 21, 31];

//冒泡排序
function sort(arr) {
    if(!Array.isArray(arr)) return;
    for (let i = 1; i < arr.length; i++) {
        for (let j = 0; j < arr.length - 1; j++) {
            if (arr[j + 1] < arr[j]) {
                [arr[j + 1], arr[j]] = [arr[j], arr[j + 1]]
            }
        }
    }
    return arr;
}
// console.log('sort',sort(arr))
//快速排序
function quickSort(arr) {
    if (!Array.isArray(arr)) return;
    if (arr.length <= 1) return arr;
    let leftArr = [];
    let rightArr = [];
    let valueIndex = ~~(arr.length / 2);
    let piovt = arr.splice(valueIndex, 1)[0];
    for (let i = 0; i < arr.length; i++) {
        if (arr[i] > piovt) {
            rightArr.push(arr[i])
        } else {
            leftArr.push(arr[i])
        }
    }
    return [...quickSort(leftArr), piovt, ...quickSort(rightArr)];
}
// console.log('quickSort',quickSort(arr))

//插入排序
function insertion(arr) {
    let current = 0,
        preIndex = 0;
    for (let i = 0; i < arr.length; i++) {
        current = arr[i]; //当前
        preIndex = i - 1; //上一个
        while (preIndex >= 0 && arr[preIndex] > current) {
            arr[preIndex + 1] = arr[preIndex];
            preIndex--
            console.log(arr,preIndex+1,current)
        }
        
        arr[preIndex + 1] = current;
    }
    return arr;
}
var arr1 = [3, 5, 7, 1, 4, 56, 12, 78, 25, 0, 9, 8, 42, 37];
console.log('insertion',insertion(arr1));


//选择排序
function selectSort(arr) {
    var len = arr.length;
    var minIndex;
    for (var i = 0; i < len - 1; i++) {
        minIndex = i;
        for (var j = i + 1; j < len; j++) {
            if (arr[minIndex] > arr[j]) {
                minIndex = j;
            }
        }
        [arr[i], arr[minIndex]] = [arr[minIndex], arr[i]]
    }
    return arr;
}
// console.log('selectSort',selectSort(arr1))






//希尔排序
function shellSort(arr) {
    var len = arr.length;
    for (var gap = ~~(len / 2); gap > 0; gap = ~~(gap / 2)) {
        // 注意：这里和动图演示的不一样，动图是分组执行，实际操作是多个分组交替执行
        for (var i = gap; i < len; i++) {
            var j = i;
            var current = arr[i];
            while (j - gap >= 0 && current < arr[j - gap]) {
                arr[j] = arr[j - gap];
                j = j - gap;
            }
            arr[j] = current;
        }
    }
    return arr;
}
// console.log(shellSort(arr1))

//归并排序
function mergeSort(arr) { //采用自上而下的递归方法
    var len = arr.length;
    if (len < 2) {
        return arr;
    }
    var middle = Math.floor(len / 2),
        left = arr.slice(0, middle),
        right = arr.slice(middle);
    return merge(mergeSort(left), mergeSort(right));
}

function merge(left, right) {
    var result = [];
    while (left.length && right.length) {
        // 不断比较left和right数组的第一项，小的取出存入res
        left[0] < right[0] ? result.push(left.shift()) : result.push(right.shift());
    }
    return result.concat(left, right);
}
// console.log(mergeSort(arr1))
```

