# 掌握立即调用函数表达式(IIFE)

[Essential JavaScript: Mastering Immediately-invoked Function Expressions](https://medium.com/@vvkchandra/essential-javascript-mastering-immediately-invoked-function-expressions-67791338ddc6) BY `Chandra Gundamaraju` ON 2017-08-04

彻底理解函数，然后学习如何利用它们编写现代的、干净的 JavaScript 代码，是成为 JavaScript 忍者的关键技能。

一种经常使用的带有函数的编码模式有一个好听的名称:立即调用函数表达式。或者更广为人知的是**IIFE**，发音为**iffy**。

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
