# 你所需要知道的关于 JavaScript 表达式、语句和表达式语句

[All you need to know about Javascript's Expressions, Statements and Expression Statements](https://dev.to/promhize/javascript-in-depth-all-you-need-to-know-about-expressions-statements-and-expression-statements-5k2) by `Promise Tochi` on 2017-10-21

到这篇文章的最后，你应该就可以很详细地描述下面的代码是怎样以及为什么这样工作。

<!--prettier-ignore-->
```javascript
{} + 1 // 1

{2} + 2 // 2

{2+2} + 3 // 3

{2+2} -3 // -3
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
