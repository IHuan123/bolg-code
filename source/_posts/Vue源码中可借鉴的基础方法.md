---
title: Vue源码中可借鉴的基础方法
toc: true
date: 2022-03-28 20:20:52
tags: [Vue,JavaScript]
urlname:
categories: [Vue,JavaScript]
recommend:
cover:
keywords: Vue源码中可借鉴的基础方法
top: 3
---



### 判断值是否为undefined或者null 

```js
function isUndef (v) {
   return v === undefined || v === null
}
```

<!--more-->

### 判断值是否不为undefined或者null

```js
function isDef (v) {
   return v !== undefined && v !== null
}
```

### 判断值是否为true

```js
function isTrue (v) {
   return v === true
}
```

### 判断值是否为false

```js
function isFalse (v) {
   return v === false
}
```

### 判断值是否为原始值

```js
  function isPrimitive (value) {
    return (
      typeof value === 'string' ||
      typeof value === 'number' ||
      // $flow-disable-line
      typeof value === 'symbol' ||
      typeof value === 'boolean'
    )
  }
```

### 判断一个对象类型

```js
  function isObject (obj) {
    return obj !== null && typeof obj === 'object'
  }
```

### 通过Object.prototype.toString 判断类型

```js
  var _toString = Object.prototype.toString;

  function toRawType (value) {
    return _toString.call(value).slice(8, -1)
  }
```

### 判断一个Promise对象

```js
  function isPromise (val) {
    return (
      isDef(val) &&
      typeof val.then === 'function' &&
      typeof val.catch === 'function'
    )
  }
```

### 将类数组转换为数组

```js
  function toArray (list, start) {
    start = start || 0;
    var i = list.length - start;
    var ret = new Array(i);
    while (i--) {
      ret[i] = list[i + start];
    }
    return ret
  }
```

### 判断两个对象或者数组是否相同

```js
  function looseEqual (a, b) {
    if (a === b) { return true }
    var isObjectA = isObject(a);
    var isObjectB = isObject(b);
    if (isObjectA && isObjectB) {
      try {
        var isArrayA = Array.isArray(a);
        var isArrayB = Array.isArray(b);
        if (isArrayA && isArrayB) {
          return a.length === b.length && a.every(function (e, i) {
            return looseEqual(e, b[i])
          })
        } else if (a instanceof Date && b instanceof Date) {
          return a.getTime() === b.getTime()
        } else if (!isArrayA && !isArrayB) {
          var keysA = Object.keys(a);
          var keysB = Object.keys(b);
          return keysA.length === keysB.length && keysA.every(function (key) {
            return looseEqual(a[key], b[key])
          })
        } else {
          /* istanbul ignore next */
          return false
        }
      } catch (e) {
        /* istanbul ignore next */
        return false
      }
    } else if (!isObjectA && !isObjectB) {
      return String(a) === String(b)
    } else {
      return false
    }
  }
```

### 简单的绑定会比原生更快

```js
function bind (fn, ctx) {
  function boundFn (a) {
    var l = arguments.length;//获取实参的数量
    return l
      ? l > 1//如果实参数量大于1
        ? fn.apply(ctx, arguments)
        : fn.call(ctx, a)//实参数量等于1
      : fn.call(ctx)//没有参数
  }
  // record original fn length//记录一下原始的形参数量
  boundFn._length = fn.length;
  return boundFn
}
```

### 将一个通过 `-`连接的字符串转为驼峰

```js
var hyphenateRE = /([^-])([A-Z])/g;
var hyphenate = cached(function (str) {
  return str
    .replace(hyphenateRE, '$1-$2')//$1为正则表达式匹配的第一个元素$2为第二个元素
    .replace(hyphenateRE, '$1-$2')
    .toLowerCase()//使之最小化
});
```

