# 在 JavaScript 种怎么更好的检查数据类型？

<i>[How to better check data types in javascript](https://webbjocke.com/javascript-check-data-types/) by `Webbjocke` on 2016-08-20</i>

在 JavaScript 中检测数据类型并不总是那么容易。语言本身提供了一个很直接的方式叫做 **tyoeof**。Typeof 返回一个字符串表示一个数据的类型，对于一个对象 `"object"`会被返回，一个字符串`"string"`会被返回。

然后 JavaScript 的数据类型，typeof 操作符并不是完美的。比如对于数组和 null 都是返回`"object"`，NaN 和 Infinity 都是返回`"number"`。要检查任何不仅仅是原始数据类型的数据，要知道它到底是一个数值、字符串、null、一个数组或是一个真实的对象，就需要一点更多的逻辑。

## String

一个 `string` 一直都是一个字符串，所以这个简单一点。除非是通过调用 `new`(new String)创建的，typeof 会返回`"object"`。所以要包括这些字符串在内，可以使用 instanceof。

```javascript
// Returns if a value is a string
function isString(value) {
  return typeof value === 'string' || value instanceof String;
}
```

## Number

通过 typeof，不只是普通的`number`会返回`"number"`，还有更多的东西，比如 NaN 和 Infinity。要想知道一个值是否真的是数值，就需要用到 isFinite 函数。

```javascript
// Returns if a value is really a number
function isNumber(value) {
  return typeof value === 'number' && isFinite(value);
}
```

## Array

在 JavaScript 中，`array`并不像 Java 或其他语言里的真实数组。它们实际上是对象，所以 typeof 会返回`"object"`。要想知道某个东西是否真的是一个数组，可以将它的构造函数与 Array 做比较。

```javascript
// Returns if a value is an array
function isArray(value) {
  return value && typeof value === 'object' && value.constructor === Array;
}

// ES5 actually has a method for this (ie9+)
Array.isArray(value);
```

## Function

函数就只是函数，所以 typeof 就足够了。

```javascript
// Returns if a value is a function
function isFunction(value) {
  return typeof value === 'function';
}
```

## Object

在 JavaScript 中，很多东西都是对象。要想知道一个值是否真的是一个可以有属性、可以被遍历的对象，可以将它的构造函数与 Object 做比较。

```javascript
// Returns if a value is an object
function isObject(value) {
  return value && typeof value === 'object' && value.constructor === Object;
}
```

## Null & undefined

大多数时候你不需要显式的去检测 `null` 和 `undefined`，因为它们都是假值。下面的函数可以做到。

```javascript
// Returns if a value is null
function isNull(value) {
  return value === null;
}

// Returns if a value is undefined
function isUndefined(value) {
  return typeof value === 'undefined';
}
```

## Boolean

对于布尔值，typeof 就足够了，因为 true 和 false 都是返回`"boolean"`。

```javascript
// Returns if a value is a boolean
function isBoolean(value) {
  return typeof value === 'boolean';
}
```

## RegExp

正则表达式是对象，所以只需要检查它的构造函数是否为 RegExp。

```javascript
// Returns if a value is a regexp
function isRegExp(value) {
  return value && typeof value === 'object' && value.constructor === RegExp;
}
```

## Error

JavaScript 里的 Error 与其他语言里的 exceptions(异常)一样。它们有很多形式，比如 instance Error、TypeError 以及 RangeError 等。对应 Error，一个 instanceof 声明就足够了，但是为了更加确保，我们可以再检查下是否有 message 属性。

```javascript
// Returns if value is an error object
function isError(value) {
  return value instanceof Error && typeof value.message !== 'undefined';
}
```

## Date

JavaScript 里`Date`并不是一个真的 date 类型。但要检查一个东西是否真的是 Date 对象，可以用 instanceof。

```javascript
// Returns if value is a date object
function isDate(value) {
  return value instanceof Date;
}
```

## Symbol

ES6 里新增加了 Symbol 这个新的数据类型。typeof 返回`"symbol"`已经足够清楚，所以不需要更多的逻辑。

```javascript
// Returns if a Symbol
function isSymbol(value) {
  return typeof value === 'symbol';
}
```

目前就这些数据类型了，想要更深入的了解 JavaScript 的数据类型和数据结构，可以参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Data_structures)。
