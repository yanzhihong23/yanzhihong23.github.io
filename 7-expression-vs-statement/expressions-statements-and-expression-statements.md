<!--
 * @Author: Hom Yan
 * @Date: 2018-11-16 15:05:41
 * @LastEditors: Hom Yan
 * @LastEditTime: 2019-06-12 12:04:31
 -->

# 你所需要知道的关于 JavaScript 表达式、语句和表达式语句

[All you need to know about Javascript's Expressions, Statements and Expression Statements](https://dev.to/promhize/javascript-in-depth-all-you-need-to-know-about-expressions-statements-and-expression-statements-5k2) by `Promise Tochi` on 2017-10-21

到这篇文章的最后，你应该就可以很详细地描述下面的代码是怎样以及为什么这样工作。

<!--prettier-ignore-->
```javascript
{} + 1 // 1

{2} + 2 // 2

{2+2} + 3 // 3 

{2+2} - 3 // -3
```

JavaScript 有 2 种主要的句法类目：

1. 语句(Statements)
2. 表达式(Expressions)

做这样的区分很重要，因为表达式可以像语句一样工作，这就是为什么我们也有表达式语句。然而，另一方面，语句却不能像表达式一样工作。

## 表达式

### 表达式产生值

表达式是一段会产生一个单一值的 JavaScript 代码片段。你想表达式有多长就可以有多长，但是，它们最终的结果都会是一个单一值。

<!--prettier-ignore-->
```javascript
2 + 2 * 3 / 2

(Math.random() * (100 - 20)) + 20

functionCall()

window.history ? useHistory() : noHistoryFallback()

1+1, 2+2, 3+3

declaredVariable

true && functionCall()

true && declaredVariable
```

上面所有的都是表达式，它们可以出现在任何期望是一个值的地方。所以下面传给 `console.log` 的参数，被解析为一个单一值，也就是被打印在控制台的结果。

```javascript
console.log(true && 2 * 9); // 18
```

### 表达式不一定会改变状态

例如：

<!--prettier-ignore-->
```javascript
const assignedVariable = 2; //这是个语句, assignedVariable 是状态

assignedVariable + 4 // 表达式

assignedVariable * 10 // 表达式

assignedVariable - 10 // 表达式

console.log(assignedVariable) // 2
```

上面的片段里尽管有表达式，assignedVariable 的值还是 2。那么为什么这节标题里是`不一定`，这是因为函数调用是表达式，而函数可以包含可改变状态的语句。因此`foo()`本身就是一个表达式，要么返回 undefined，要么返回其他值。但如果`foo`是这样编写的：

<!--prettier-ignore-->
```javascript
const foo = foo () => {
  assignedVariable = 14
}
```

那么，尽管它的调用是一个表达式，它的调用还是会导致状态的改变。所以一个更好的重写 foo 函数和语句的方式是：

<!--prettier-ignore-->
```javascript
const foo = foo () => {
  return 14 //explicit return for readability
}
assignedVariable = foo()
```

甚至更好

<!--prettier-ignore-->
```javascript
const foo = foo (n) => {
  return n //explicit return for readability
}
assignedVariable = foo(14)
```

这样你的代码可读性更高、组合性更强，表达式与语句有明显的区分。这是函数式和声明式 JavaScript 的基础。

## 语句

语句是函数式编程里头疼的事。基本上，语句执行动作，它们做事情。

在 JavaScript 里，语句永远不能被用在期望是一个值的地方。所以它们不能被用作函数参数、赋值的右侧、运算元以及返回值等。

这些都是 JavaScript 语句：

1. if
2. if-else
3. while
4. do-while
5. for
6. switch
7. for-in
8. with (deprecated)
9. debugger
10. variable declaration

如果你在浏览器的控制台上输入这个片段并按回车

```javascript
if (true) {
  9 + 9;
}
```

你回看到返回了`18`，但尽管如此，你还是不能把它当做表达式或者期望一个值的地方。这很怪异，因为你期望语句不要返回任何东西，因为如果你无法使用它，那返回的值就没有任何用处。JavaScript 就是这么的怪异...

## 函数声明、函数表达式、命名函数表达式

函数声明是一个语句

```javascript
function foo(func) {
  return func.name;
}
```

函数表达式是一个表达式，这里调用了一个匿名函数

```javascript
console.log(foo(function() {})); // ""
```

命名函数表达式是一个表达式，就像一个匿名函数，只是它有名称

```javascript
console.log(foo(function myName() {})); // "myName"
```

函数作为一个表达式和一个声明的区别可以归结于理解下面：

**无论何时，你在一个需要值得地方声明一个函数，它就会尝试把它当作一个值，如果它不能被当作一个值，那么就会抛出错误。**
**而在脚本、模块的全局层或块语句的顶层(也就是说，它不需要值)声明一个函数，则会得到一个函数声明。**

例子：

```javascript
if () {
  function foo () {} // 块的顶层, 声明
}

function foo () {} // 全局, 声明

function foo () {
  function bar() {} // 块的顶层, 声明
}

function foo () {
  return function bar () {} // 命名函数表达式
}

foo(function () {}) // 匿名函数表达式

function foo () {
  return function bar () {
    function baz () {} // 块的顶层, 声明
  }
}

function () {} // 语法错误: 函数语句需要一个名称

if (true){
  function () {} // 语法错误: 函数语句需要一个名称
}
```

## 表达式转为语句：表达式语句

Javascript 有什么简单直接的地方吗 😃

```javascript
2 + 2; //表达式语句
foo(); //表达式语句
```

表达式转为表达式语句，你只要在行末添加一个分号或着允许[自动插入分号(automatic semi-colon insertion)](https://dev.to/promhize/what-you-need-to-know-about-javascripts-automatic-semi-colon-insertion-78a) 来完成这个工作。 `2+2`本身是一个表达式，但一整行就是一个语句。

<!--prettier-ignore-->
```javascript
2+2 // 表达式

foo(2+2) // 所以你可以在任何需要值的地方使用它

true ? 2+2 : 1 + 1

function foo () {return 2+2}


2+2; // 表达式语句
foo(2+2;) // 语法错误
```

## 分号和逗号

使用分号，你可以把多个语句放在同一行

<!--prettier-ignore-->
```javascript
const a; function foo () {}; const b = 2
```

逗号让你可以连接多个表达式，并只返回最后一个表达式

<!--prettier-ignore-->
```javascript
console.log( (1+2,3,4) ) //4

console.log( (2, 9/3, function () {}) ) // function (){}

console.log( (3, true ? 2+2 : 1+1) ) // 4
```

> 通过`圆括号()`可以告诉 JavaScript 引擎这里期望得到的是一个值，没有了圆括号，每个表达式都会被当作是 console.log 的参数。

<!--prettier-ignore-->
```javascript
function foo () {return 1, 2, 3, 4}
foo() //4
```

所有表达式将从左到右求值，最后一个表达式将返回。

## IIFEs (立即调用的函数表达式)

一个匿名函数可以是一个表达式，如果我们把它用在一个期望为值得地方，这意味着我们可以用圆括号来告诉 JavaScript 期望得到一个值，我们可以把匿名函数当作值传进去。

<!--prettier-ignore-->
```javascript
function () {}
```

上面这段代码是无效的，下面这段才是有效的

<!--prettier-ignore-->
```javascript
(function () {}) // this returns function () {}
```

如果把一个匿名函数放在圆括号内立即返回了同样的匿名函数，这意味着我们可以直接调用，像这样：

<!--prettier-ignore-->
```javascript
(function () {
  //do something
})()
```

所以，这些都是可以的

<!--prettier-ignore-->
```javascript
(function () {
  console.log("immediately invoke anonymous function call")
})() // "immediately invoke anonymous function call"

(function () {
  return 3
})() // 3

console.log((function () {
  return 3
})()) // 3

// 你可以传一个参数给它
(function (a) {
  return a
})("I'm an argument") // I'm an argument
```

## 对象字面量与块语句

> 旁注：这是有效的

<!--prettier-ignore-->
```javascript
r: 2+2 // 有效

foo()

const foo = () => {}
```

上面这一系列在全局作用域下的语句会被解析为有效的 JavaScript 并执行。`r`通常称为标签，在跳出循环的地方非常有用。例子：

<!--prettier-ignore-->
```javascript
loop: {
  for (const i = 0; i < 2; i++) {
    for (const n = 0; n <2; n++) {
      break loop // 跳出最外层的循环，停止整个循环
    }
  }
}
```

你可以给任何表达式或表达式语句添加标签，注意这并不是在创建一个标签变量：

<!--prettier-ignore-->
```javascript
lab: function a () {}
console.log(lab) //ReferenceError: lab is not defined
```

通过`花括号{}`你可以把表达式语句和语句分组，所以你可以这样写

<!--prettier-ignore-->
```javascript
{var a = "b"; func(); 2+2} // 4
```

如果你把上面的代码粘贴到浏览器的控制台，它将会返回 4 ，如果你执行`console.log(a)`，你就会得到字符串`'b'`。你可以称之为块语句，这是与你习惯的对象字面量不同的地方。

<!--prettier-ignore-->
```javascript
console.log({a: 'b'}) // {a: 'b'}

console.log({var a = "b", func(), 2+2}) // SyntaxError

const obj = {var a = "b", func(), 2+2} // SyntaxError
```

你不能把块语句作为一个值或表达式来使用，因为 console.log 是一个函数，它不能接受一个语句来作为一个参数，虽然可以接受一个对象字面量。
我希望你理解了我上面解释的所有内容，因为下面这段代码可能会让你很困惑。

<!--prettier-ignore-->
```javascript
{} + 1 //1

{2} + 2 // 2

{2+2} + 3 // 3

{2+2} -3 // -3
```

你可能预计它会抛出一个语法错误或是分别返回 1,4,7。记住语句是不应该返回任何东西的，因为它们不能被当作值来使用。相对于抛出一个错误，JavaScript 更倾向于把`+`操作符的操作对象转换为一个数值或字符串，如果无法转换才抛出错误。所以无论块语句返回什么，都会被隐式强制转换为`0`来作为操作对象。

唷！如果你都读完了，那你就是真正的 MVP。这大概就是所有你需要了解的关于表达式、语句以及表达式语句。
