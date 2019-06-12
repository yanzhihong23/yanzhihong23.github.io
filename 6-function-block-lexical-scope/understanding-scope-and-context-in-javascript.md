<!--
 * @Author: Hom Yan
 * @Date: 2018-11-28 15:05:41
 * @LastEditors: Hom Yan
 * @LastEditTime: 2019-06-12 12:03:18
 -->

# 理解 JavaScript 里的作用域和上下文

[Understanding Scope and Context in JavaScript](http://ryanmorr.com/understanding-scope-and-context-in-javascript/) by `Ryan Morr` on 2013-08-16

JavaScript 对作用域(`Scope`)和上下文(`Context`)的实现是该语言的一个独特特性，部分原因是它非常灵活。函数可用于各种上下文，作用域可封装和保存。这些概念有助于 JavaScript 提供的一些最强大的设计模式。然而，在开发人员中，这也是一个巨大的困惑之源，这是有充分理由的。下面是对作用域和上下文、它们之间的区别以及各种设计模式如何使用它们的全面解释。

## 上下文和作用域

首先要澄清的是，上下文和作用域是不相同的。多年来，我注意到许多开发人员常常混淆这两个术语(包括我自己)，错误地将一个术语描述为另一个术语。公平地说，这些年来术语变得相当混乱。

每个函数调用都有与之相关的一个作用域和一个上下文。基本上，作用域是基于函数而上下文是基于对象。换句话说，作用域属于函数调用时的变量访问，并且对每次调用都是独立的。上下文总是`this`关键字的值，该关键字是对“拥有”当前执行代码的对象的引用。

## 变量作用域

一个变量可以被定义在局部或全局作用域，这就确立了变量在运行时在不同作用域内的可访问性。任何定义的全局变量，即变量定义在函数体之外，会在运行时一直存在，并且可以在任何作用域内被改变。局部变量只存在于定义它的函数体内，而且每次调用的作用域都是不一样的。仅可在该调用中完成值分配、检索和操作，在该作用域外不可访问。

ECMAScript 6(ES6/ES2015) 引进了`let`和`const`关键词用来支持块作用域局部变量的声明。这意味着变量将被限制在定义它的块范围内，比如一个`if`语句或者`for`循环，在块的花括号之外都不可访问。这与`var`声明是相反的，var 声明的是可以在定义的块之外访问到。`let`与`const`的区别在于，`const`声明值的引用是只读的，就像它名称所预示的--常量。这并不意味着该值是不可变的，只是不能重新分配变量标识符。

## 什么是 this 上下文

上下文通常是由函数的调用方式决定的。当一个函数是作为一个对象的方法被调用时，`this`就被设置为调用该方法的对象：

```javascript
var obj = {
  foo: function() {
    return this;
  }
};

obj.foo() === obj; // true
```

同样的准则可用于通过`new`操作符调用函数来创建一个对象的实例。通过这种方式调用时，函数作用域内`this`的值就被置为新创建的实例：

```javascript
function foo() {
  alert(this);
}

foo(); // window
new foo(); // foo
```

当作为未绑定函数调用时，`this`将默认为全局上下文或浏览器里的 window 对象。然而，如果函数式在严格模式下执行的，那么上下文默认为 undefined。

## 执行上下文

JavaScript 是一个单线程语言，意味着同时只能有一个任务被执行。当 JavaScript 解释器开始执行代码时，首先默认进入一个全局执行上下文。这时候每个函数的调用，都会创建一个新的执行上下文。

这就是经常出现混淆的地方，术语“执行上下文”实际上所有的意图和目的更多的偏向于作用域，而不是前面讨论的上下文。这是个不幸的命名约定，然而这个术语是由 ECMAScript 规范定义的，所以呢我们也只能忍受它。

每次一个新的执行上下文被创建，它就会被加到`执行栈(execution stack)`的最上面。浏览器就会执行执行栈最上层的当前执行上下文。一旦完成，它就会从栈里被移除，控制权就交回给下面的执行上下文。

一个执行上下文可以分为创建和执行两个阶段。在创建阶段，解释器首先会创建一个变量对象（也称为激活对象），由定义在执行上下文里的所有变量、函数声明以及参数组成。接下来初始化作用域链，最后确定`this`的值。然后在执行阶段，代码被解释执行。

## 作用域链

对于每个执行上下文，都有一个与之耦合的作用域链。作用域链包含用于执行栈里每个执行上下文的变量对象。它用于确定变量访问和标识符解析。例如：

```javascript
function first() {
  second();
  function second() {
    third();
    function third() {
      fourth();
      function fourth() {
        // do something
      }
    }
  }
}
first();
```

运行上面的代码的结果是嵌套的函数一路被执行，直到`fourth`函数。这时作用域链从上到下就是：fourth, third, second, first, global。`fourth`函数可以访问全局变量和任意定义在`first`、`second`、`third`函数里的变量，当然也包括这些函数。

不同执行上下文之间的变量之间的名称冲突可以通过向上攀登作用域链来解决，从局部转移到全局。这意味着具有与作用域链上游变量相同名称的本地变量优先。

简单来说，每次你在一个函数的执行上下文里试图访问一个变量，查找过程最先从自身的变量对象开始。如果标识符在变量对象里无法找到，搜索就会进入作用域链。它将向上攀登作用域链，检测每个执行上下文的变量对象来查找匹配的变量名称。

## 闭包

访问直接词法作用域之外的变量将创建一个闭包。换句话说，闭包的形成是，一个嵌套的函数被定义在另外一个函数体内，并允许访问外层函数的变量。返回嵌套函数允许你维护对其外部函数的局部变量、参数和内部函数声明的访问。这种封装使我们能够在公开公共接口时，在外部范围内隐藏和保存执行上下文，从而受到进一步的操作。下面是一个简单的例子：

```javascript
function foo() {
  var localVariable = 'private variable';
  return function() {
    return localVariable;
  };
}

var getLocalVariable = foo();
getLocalVariable(); // "private variable"
```

最流行的闭包类型之一是众所周知的模块模式；它允许您模仿公共、私人和特权成员:

```javascript
var Module = (function() {
  var privateProperty = 'foo';

  function privateMethod(args) {
    // do something
  }

  return {
    publicProperty: '',

    publicMethod: function(args) {
      // do something
    },

    privilegedMethod: function(args) {
      return privateMethod(args);
    }
  };
})();
```

模块的行为就好像它是一个单例对象，一旦编译器解释它，它就会被执行，因此在函数的末尾有一个开括号和一个闭括号。闭包的执行上下文之外唯一可用的成员是位于 return 对象(模块)中的公共方法和属性(例如 `Module.publicMethod`)。然而，随着执行上下文的保存，所有私有属性和方法都将贯穿应用程序的整个生命周期，这意味着变量将受到通过公共方法进行的进一步交互的影响。

另一种类型的闭包称为即时调用函数表达式(IIFE)，它只不过是在 window 上下文中执行的自调用匿名函数:

```javascript
(function(window) {
  var foo, bar;

  function private() {
    // do something
  }

  window.Module = {
    public: function() {
      // do something
    }
  };
})(this);
```

当试图保存全局名称空间时，这个表达式非常有用，因为函数体中声明的任何变量都将是闭包的本地变量，但在整个运行时仍然存在。这是一种流行的封装应用程序和框架源代码的方法，通常公开一个用于交互的全局接口。

## Call 和 Apply

这两个方法是所有函数固有的，它让你可以在任意指定的上下文中执行任意函数。这造就了难以置信的强大功能。`call`函数要求参数必须显式的列出来，而`apply`函数则允许你以数组的形式来提供参数：

```javascript
function user(firstName, lastName, age) {
  // do something
}

user.call(window, 'John', 'Doe', 30);
user.apply(window, ['John', 'Doe', 30]);
```

这两个调用的结果是完全一样的，`user`函数在 window 的上下文中被调用并提供了 3 个一样的参数。

ECMAScript5(ES5)引进了`Function.prototype.bind`方法用来操作上下文。它返回了一个新的函数，永远的绑定到`bind`的第一个参数，而不管函数是怎么被使用的。它使用了一个闭包来负责重定向到合适的上下文。对于不支持的浏览器，请参见下面的 polyfill (腻子脚本):

```javascript
if (!('bind' in Function.prototype)) {
  Function.prototype.bind = function() {
    var fn = this;
    var context = arguments[0];
    var args = Array.prototype.slice.call(arguments, 1);
    return function() {
      return fn.apply(context, args.concat([].slice.call(arguments)));
    };
  };
}
```

它通常用于上下文经常丢失的地方、面向对象和事件处理。这是必要的，因为一个节点的`addEventListener`方法总是在节点事件处理器绑定的节点的上下文中执行回调，这是应该的方式。然而如果你正在使用先进的面向对象技术，并要求你的回调是一个实例的方法，你就需要手动去改变上下文，这时`bind`就很方便了：

```javascript
function Widget() {
  this.element = document.createElement('div');
  this.element.addEventListener('click', this.onClick.bind(this), false);
}

Widget.prototype.onClick = function(e) {
  // do something
};
```

在查看`Function.prototype.bind`函数的 polyfill 源码时，你可能会注意到有 2 个调用涉及`Array`的`slice`方法：

```javascript
Array.prototype.slice.call(arguments, 1);
[].slice.call(arguments);
```

注意这里有趣的是`arguments`对象根本不是一个数组，然而它通常会被描述为一个像 nodelist(`element.childNodes`返回的任何东西)的类数组对象。它们包含一个 length 属性和下标值，但它们依然不是数组，也不支持数组的原生方法，比如`slice`和`push`。但是，由于它们的类似行为，可以采用或劫持数组的方法(如果愿意)，并在类数组对象的上下文中执行，如上所述。

这种采用另一个对象方法的技术也适用于在 JavaScript 中模拟基于经典继承时的面向对象:

```javascript
SubClass.prototype.init = function() {
  // 在"SubClass"实例的上下文中调用超类init方法
  SuperClass.prototype.init.apply(this, arguments);
};
```

通过在子类(`SubClass`)实例的上下文中调用超类(`SuerClass`)的方法，我们可以模拟调用方法的 super 类充分利用这种强大的设计模式的能力。

## 结尾

在你开始接触高级设计模式前，理解这些概念是很重要的。因为作用域和上下文在现代 JavaScript 中扮演了一个基石的角色。不管我们是在谈论闭包、面向对象和继承还是大量的原生实现，上下文和作用域都在它们中扮演了一个重要的角色。如果你的目标是掌握 JavaScript 并更好的理解它所包含的所有内容，那么作用域和上下文应该是你开始的一个点。
