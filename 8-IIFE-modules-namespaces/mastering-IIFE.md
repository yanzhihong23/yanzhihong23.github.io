# 掌握立即调用函数表达式(IIFE)

[Essential JavaScript: Mastering Immediately-invoked Function Expressions](https://medium.com/@vvkchandra/essential-javascript-mastering-immediately-invoked-function-expressions-67791338ddc6) BY `Chandra Gundamaraju` ON 2017-08-04

彻底理解函数，然后学习如何利用它们编写现代的、干净的 JavaScript 代码，是成为 JavaScript 忍者的关键技能。

一种经常使用的，带有函数的编码模式，有一个好听的名称：立即调用函数表达式。或者更广为人知的是**IIFE**，发音为**iffy**。

在我们理解什么是 IIFE 以及为什么需要它之前，我们需要先快速回顾下 JavaScript 函数的一些基础概念。

## 自然函数定义

初学 JavaScript 的开发人员在处理函数时自然会熟悉以下语法。

```javascript
function sayHi() {
  alert('Hello, World!');
}

sayHi(); // 在浏览器里显示弹窗 "Hello, World!"
```

1. 1-3 行定义了一个函数，名称为`sayHi`
2. 第 5 行通过常用的`()`语法来调用这个函数

这种创建函数的方式称为“一个函数定义”或“一个函数声明”或“一个函数语句”。通常，初学者使用这种语法没有问题，因为它跟当前流行的其他语言很接近。

> 这些函数定义通常以 `function` 关键字开头，后面跟着一个名称。你不能忽略这个名称，那是无效的语法。

## 函数表达式

这正是 JavaScript 开始变得有趣的时候，我们来看下函数表达式长什么样。

```javascript
var msg = 'Hello, World!';
var sayHi = function() {
  alert(msg);
};

sayHi(); // 在浏览器里显示弹窗 "Hello, World!"
```

这个看似简单例子可以帮你把 JavaScript 技能提升到下一个阶段。

1. 第一行声明了`msg`变量，并分配了一个`string`给它。
2. 2-4 行声明了`sayHi`变量，并分配了一个`function`类型的值给它。
3. 第 5 行调用这个`sayHi`函数。

第一行很容易理解。但第一次看到 2-4 行时，如果开发者是来自其它像 Java 的语言，这通常会违背他们的预期。

> 基本上，在 2-4 行我们分配了一个`function`类型的值给一个名称为`sayHi`的变量。
>
> 在上面的例子中，赋值运算符右边的函数通常称为“函数表达式”。在 JavaScript 中到处都是。可能你写的大部分回调函数通常都是函数表达式。

你可能在不理解这些基础的情况下已经使用了这些函数表达式，但是掌握它们将给你一些 JavaScript 的超能力。

> 所以这里要记住的重要概念是，在 JavaScript 中函数就像其它类型的值一样。它们可以放在赋值运算符的右边，也可以作为参数传给其它函数。

## 匿名函数表达式

你已经知道它们是什么了。上面的例子就是一个匿名函数表达式。它们是匿名的，因为它们没有一个名称跟随在`function`关键字后面。

## 命名函数表达式

函数表达式可以有名称。这些命名函数表达式最无聊、最普遍的用法是递归。现在不用太担心这些，因为不了解命名函数表达式你也能掌握 IIFE。

```javascript
var fibo = function fibonacci() {
  // 这里你可以使用 "fibonacci()"，因为这个函数表达式有个名称。
};

// fibonacci() 会失败, 但 fibo() 可以。
```

所以这里的区别是，函数表达式有一个名称为`fibonacci`可以在函数表达式内部使用，用来递归调用自身。（它还有更多的功能，比如函数名显示在堆栈跟踪等中，但是在本教程中我们不需要担心它们。）

## 够了！给我一个 IIFE，不然我就走了！

感谢你的耐心，耐心是掌握 JavaScript 最重要的技能！

现在你已经学习了函数定义和函数表达式，让我们来深入了解下 IIFE 的秘密吧。它们有几种变体，我们先来看一个很容易理解的变体。

<!--prettier-ignore-->
```javascript
!function() {
  alert("Hello from IIFE!");
}();
// Shows the alert "Hello from IIFE!"
```

这就是 IIFE！当你在浏览器里运行这段代码，你将看到来自第二行的弹框。差不多就是这样，没人可以让这个弹框再显示一次。

> 就是这么一个函数，在它被激活后立即消失。

现在我们来理解这个不太直观的语法: 我知道你注意到了第 1 行的`!；如果你没有，也没关系，你现在就会发现它！

1. 像之前看到的，一个函数语句通常以`function`关键字开头。当 JavaScript 在一个有效的语句里看到`function`作为第一个词，它就预计一个函数定义将发生。那么为了阻止发生这样的情况，我们在第 1 行的`function`关键字前加`!`。这样基本上就强制让 JavaScript 把跟在`!`后面的任何东西当作一个表达式。

2. 但是最有趣的是在第 3 行我们立即执行了函数表达式。

> 所以我们有一个函数表达式，在创建之后被立即调用。朋友们，这就是一个 IIFE，不用管用的是什么语体来达到这个效果。

上面的语体里的`!`可以用`+`、`-`或者`~`来代替。基本上任一一元运算符都可以。

现在，在控制台里试下吧！自己来玩玩 IIFE！

> 这所有的第一个字符(`!`)，做的就是把函数变成一个表达式而不是一个函数语句或定义。然后我们立即执行那个函数。

看下下面另外一个变体：

<!--prettier-ignore-->
```javascript
void function() {
  alert("Hello from IIFE!");
}();
```

`void` 基本上也是强制把函数当作表达式来处理。

当你不用关心 IIFE 返回的是什么时，上面这些模式都是很有用的。

但是如果你想要 IIFE 返回一个值，并在其它地方使用它呢？继续玩下看！

## 经典 IIFE 样式

上面看到的 IIFE 模式都是很容易理解的，所以我以这种模式开头，而不是另外更传统、使用更广泛的样式。

> 我们上面看到的 IIFE 例子，对 IIFE 的关键是把函数变为一个表达式并立即执行它。

首先，我们来看下变为函数表达式的另一种方式！

```javascript
(function() {
  alert('I am not an IIFE yet!');
});
```

上面的代码中，一个函数表达式被包裹在一对圆括号内。这还不是一个 IIFE，因为它还没被执行。现在我们来把这段代码转变为一个 IIFE，我们有以下两种语体：

<!--prettier-ignore-->
```javascript
// Variation 1
(function() {
  alert("I am an IIFE!");
}());

// Variation 2
(function() {
  alert("I am an IIFE, too!");
})();
```

现在我们有 2 个 IIFE，可能很难看出这两个语体的区别。下面我来解释下。

1. 语体 1，第 4 行，用来调用函数表达式的圆括号`()`位于外层圆括号内。同样外层圆括号也是需要的，用来把那个函数转变为函数表达式。

2. 语体 2，第 9 行，用来调用函数表达式的圆括号`()`位于函数表达式的包裹圆括号之外。

这两种语体都使用的很广泛，我个人更倾向于语体 1。如果我们更深入的了解，这两种语体的工作原理相差甚微。但是出于所有的实际目的，为了使这本来就长的教程更简短，我要说的是，您可以根据自己的喜好使用它们中的任何一个。

让我们通过看一个可行的例子，和两个不可行的例子，再把这个问题搞清楚。从现在开始我们将给 IIFEs 命名，因为使用匿名函数绝不是一个好主意。

<!--prettier-ignore-->
```javascript
// 有效 IIFE
(function initGameIIFE() {
    // 开始你的游戏吧
}());

// 下面这个是无效 IIFE 例子
function nonWorkingIIFE() {
    // 现在你知道为什么需要包裹的圆括号
    // 没有圆括号，这只是一个函数定义，而不是一个表达式
    // 你将会得到一个语法错误
}();

function () {
    // 这里同样会得到一个语法错误
}();
```

现在你知道了为什么需要在函数表达式周围用圆括号括起来形成一个 IIFE。

## IIFEs 和 私有变量

IIFEs 非常擅长的一个能力是为 IIFE 创建一个函数作用域。

> 任何定义在 IIFE 内部的变量对外都是不可访问的。

来看个例子

<!--prettier-ignore-->
```javascript
(function IIFE_initGame() {
  // 在这个 IIFE 之外，没人可以访问到的私有变量
  var lives;
  var weapons;
  
  init();

  // 在这个 IIFE 之外，没人可以访问到的私有函数
  function init() {
      lives = 5;
      weapons = 10;
  }
}());
```

在这个例子中，我们在 IIFE 内声明了 2 个变量，它们是 IIFE 私有的。在 IIFE 之外，没人可以访问到。类似的，我们有一个`init`函数，在 IIFE 之外，没人可以访问到。但是`init`函数可以访问这些私有变量。

> 下次如果你在全局作用域中创建一堆变量和函数，除了你之外没人使用，那就把所有它们包裹在一个 IIFE 内，这样做有很大的好处。你的代码还能继续工作，但你不会污染全局作用域。也能避免别人有意无意地改变你的全局变量。

当我们看到模块模式时，我将解释如何将这些私有变量的特权和受控访问权授予 IIFE 之外的世界。所以继续读下去，即使你已经觉得自己是一个忍者了!

## 带返回值的 IIFEs

如果你不需要一个 IIFE 有返回值，那么你可以使用我们前面看到的带一元运算符的 IIFE 语体，比如`!`、`+`、`void`等。

> 但是 IIFEs 另一个很重要也很强大的功能是他们可以返回一个值，并分配给一个变量。

```javascript
var result = (function() {
  return 'From IIFE';
})();

alert(result); // alerts "From IIFE"
```

1. 在这个语体中，我们有一个 IIFE，在第 2 行有一个返回语句。
2. 当我们执行上述代码时，第 5 行显示了带有来自 IIFE 的返回值的警告。

基本上，IIFE 被执行，当然是立即执行，然后它的返回值被分配给`result`变量。

## 带参数的 IIFEs

IIFEs 不仅可以返回值，它们被调用时还能携带参数。我们看一个简单的例子。

<!--prettier-ignore-->
```javascript
(function IIFE(msg, times) {
  for (var i = 1; i <= times; i++) {
    console.log(msg);
  }
}("Hello!", 5));
```

1. 上面的例子中，在第 1 行，IIFE 有 2 个形参，分别为`msg`和`times`。
2. 当我们再第 5 行执行 IIFE，我们传入了参数，而不是像前面看到的空括号。
3. 第 2、3 行在 IIFE 内部使用这些参数。

这是一个很强大的模式，我们经常在 jQuery 的代码里看到，当然其它的库也是。

<!--prettier-ignore-->
```javascript
(function($, global, document) {
  // 使用 $ 代表 jQuery, global 代表 window
}(jQuery, window, document));
```

上面的例子中，我们在第 3 行传入了`jQuery`、`window`、`document`三个参数给 IIFE。在 IIFE 内部可以分别通过`$`、`global`、`document`来引用它们。

下面是传入这些给 IIFE 的优势。

1. JavaScript 总是从当前函数的作用域进行作用域查找，并一直在更高的作用域中搜索，直到找到标识符为止。当我们再第 3 行传入`document`时，这是唯一一次我们对`document`进行的超出本地作用域的作用域查找。在 IIFE 内对`document`的任何引用都不再需要做超出 IIFE 本地作用域的查找。`jQuery`也是一样的。基于 IIFE 的琐碎或复杂程度，通过这种方法获得的性能收益可能并不大，但它仍然是一个需要了解的有用技巧。

2. 而且，JavaScript 压缩器可以安全的压缩函数声明的参数。如果我们没有把它们作为参数传进去，压缩器就无法压缩对`document`或`jQuery`的直接引用，因为它们在函数的作用域之外。。

## 经典JavaScript模块模式
现在你已经掌握了IIFEs，现在我们来看一个把IIFEs和闭包用到极致的模块模式。

我们将要实现一个无缝的经典 `队列` 单例对象，没有人能不小心破坏当前的队列值。

```javascript
var Sequence = (function sequenceIIFE() {
    
    // 存储当前计数的私有变量
    var current = 0;
    
    // IIFE返回的对象
    return {
    };
    
}());

alert(typeof Sequence); // alerts "object"
```
1. 在上面的例子中，我们有一个返回对象的IIFE。
2. 在IIFE里，我们还有一个名为`current`的本地变量。
3. IIFE的返回值，就是这个例子里的对象，被分配给了`Sequence`变量。12行准确的弹出`object`，因为我们在IIFE里返回了一个对象。

现在我们来给返回的对象添加一些函数。

```javascript
var Sequence = (function sequenceIIFE() {
    
    // 存储当前计数的私有变量
    var current = 0;
    
    // IIFE返回的对象
    return {
        getCurrentValue: function() {
            return current;
        },
        
        getNextValue: function() {
            current = current + 1;
            return current;
        }
    };
    
}());

console.log(Sequence.getNextValue()); // 1
console.log(Sequence.getNextValue()); // 2
console.log(Sequence.getCurrentValue()); // 2
```

1. 在这个例子中，我们给IIFE返回的对象添加了2个函数。
2. `getCurrentValue`函数返回 `current`变量的值。
3. `getNextValue`函数给`current`变量加1并且返回`current`的值。

由于`current`是IIFE的私有变量，除了可以返问闭包的函数，没人可以改变或返问变量`current`。

(如果你想掌握闭包，请阅读我的关于闭包的不可思议、火爆的教程：[Learning JavaScript Closures through the Laws of Karma](https://engineering.salesforce.com/learn-javascript-closures-through-the-laws-of-karma-49d32d35b3f7))

现在你已经学到一个无比强大的JavaScript模式，它结合了IIFE和闭包的力量。

这是一个非常基本的模块模式变化。有更多的模式，但几乎所有都使用IIFE来创建一个私有闭包作用域。

## 什么时候可以省略括号
包裹函数表达式的括号强制把函数变成一个表达式而不是一个声明。

但是当对于JavaScript引擎来看，很明显是一个函数表达式的时候，我们就可以像下面这样去掉包裹的括号。

```javascript
var result = function() {
    return "From IIFE!";
}();
```

上面的例子中，`function`关键字不是声明的第一个词。所以JavaScript不会把它当做一个函数声明/定义。类似的还有其它地方你可以省略括号，当你知道那是一个表达式。

但我通常还是更倾向于使用括号。使用括号可以提高可读性，通过第一行的书写法来提醒读者这个函数将要转变为一个IIFE。他们不需要滚动到函数的最后一行，才意识到原来这是一个IIFE。


这就是当你在代码里开始使用IIFE所要知道的。他们不仅帮助组织和让你的代码表达的更优雅，而且可以避免创建不必要的全局变量来减少bug。现在，你就是一个认证过的IIFE忍者！