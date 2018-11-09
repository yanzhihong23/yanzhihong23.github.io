# JavaScript 类型强制转换

[原文](https://medium.freecodecamp.org/js-type-coercion-explained-27ba3d9a2839)

![weird](https://cdn-images-1.medium.com/max/2000/1*7awmfn1lq2McPz8ggapndw.png)

<center>JavaScript里的怪异行为</center>

**类型强制转换** 是将值从一种类型转换为另一种类型的过程，例如字符串赚数字、对象转布尔等。任何类型，无论是原始类型还是对象，都是类型转换的有效主体。回忆下，原始类型有：`number`、`string`、`boolean`、`null`、`undefined` 以及 `Symbol`(ES6 新增)。

作为类型强制转换的实践例子，可以看下 [JavaScript 比较表](https://dorey.github.io/JavaScript-Equality-Table/unified/)，表里显示了对于 2 个不同类型`a`和`b`做宽松等于时 `==` 运算符的行为。由于`==`运算符做了隐式类型强制转换，导致了这张矩阵图看起来很可怕，而且几乎不可能记住这所有的结合。当然你也不需要这么做，只要学习类型强制转换的底层原则。

这篇文章将会深入解读 JavaScript 类型强制转换的工作原理，给你提供必要的知识，这样你就能自信地解释下面的表达式是怎么计算的。文章的最后我将给出答案以及解释。

```javascript
true + false
12 / "6"
"number" + 15 + 3
15 + 3 + "number"
[1] > null
"foo" + + "bar"
'true' == true
false == 'false'
null == ''
!!"false" == !!"true"
[‘x’] == ‘x’
[] + null + 1
[1,2,3] == [1,2,3]
{}+[]+{}+[1]
!+[]+[]+![]
new Date(0) - 0
new Date(0) + 0
```

是的，这个列表是你作为一个开发者可能会做的一堆非常愚蠢的事。90%的用例最好去避免隐式类型强制转换。把这个列表当做你了解类型强制转换工作原理的练习。如果觉得无聊，你可以在 [wtfjs.com](https://wtfjs.com/) 上找到更多的例子。

顺便说下，有时你可能会在 JavaScript 开发这个职位的面试中碰到这类问题。所以，继续读下去吧 😄

## 隐式 && 显式强制转换

类型强制转换可以是显式和隐式的。

当一个开发者通过编写合适的代码来表达类型转换的意图的时候，比如`Number(value)`，这就称为 **显示类型强制转换**。

由于 JavaScript 是一个弱类型语言，值可以在不同的类型之间自动转换，这就称为 **隐式类型强制转换**。它通常发生在你在不同类型的值上执行操作的时候，比如`1 == null`、`null + new Date()`，或者由周围的上下文触发，比如`if(value) {...}`，这里`value`被强制转换为布尔值。

有个不会触发隐式类型强制转换的运算符是`===`，也叫做严格相等运算符。宽松相等运算符 `==`执行比较，必要时做类型强制转换。

隐式类型强制转换是一把双刃剑：它是失望和缺陷的一个很大来源，同时也是一个有很用的机制，它让我们可以编写更少的代码却不损失可读性。

## 3 种类型转换

需要知道的第一条规则就是，JavaScript 里只有 3 种类型转换：

- 转为 `string`
- 转为 `boolean`
- 转为 `number`

第二，原始类型与对象的转换逻辑是不一样的。但原始类型与对象都只能通过这 3 种方式转换。

我们先从原始类型开始吧。

### 字符串转换(String)

通过应用`String()`函数，可以显式的把值转换为字符串。只要任何一个运算元是字符串，隐式强制转换就会由`+`运算符触发：

```javascript
String(123); // 显式
123 + ''; // 隐式
```

所有原始值都很自然地转换为字符串，就像你预期的那样：

<!-- prettier-ignore -->
```javascript
String(123)                   // '123'
String(-12.3)                 // '-12.3'
String(null)                  // 'null'
String(undefined)             // 'undefined'
String(true)                  // 'true'
String(false)                 // 'false'
```

Symbol 的转换要稍微复杂一点，因为它只能被显式的转换而不能隐式。关于 `Symbol` 强制转换的规则可以看[这里](https://leanpub.com/understandinges6/read/#leanpub-auto-symbol-coercion)。

```javascript
String(Symbol('my symbol')); // 'Symbol(my symbol)'
'' + Symbol('my symbol'); // TypeError is thrown
```

### 布尔值转换(Boolean)

通过应用`Boolean()`函数，可以显式的把值转换为布尔值。隐式转换发生在逻辑上下文中，或由逻辑运算符（`||` `&&` `!`）触发。
