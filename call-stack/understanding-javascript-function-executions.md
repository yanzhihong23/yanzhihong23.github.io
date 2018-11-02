# 理解 JavaScript 函数执行 -- 调用栈、事件循环、任务以及其他

[原文](https://medium.com/@gaurav.pandvia/understanding-javascript-function-executions-tasks-event-loop-call-stack-more-part-1-5683dea1f5ec)

Web 开发人员或前端工程师(我们更喜欢的称呼)，现在可以做任何事情，从充当浏览器内部的交互源，到制作计算机游戏，桌面小部件，跨平台移动应用程序或在服务器端编写它（ 最常用于 node.js）将它与任何数据库连接 - 实现几乎无处不在的脚本语言。 因此，重要的是要了解 Javascript 的内部，更好和有效地使用它，这就是本文的内容。

Javascript 生态系统变得比以往任何时候都复杂，并将继续变得更加复杂。构建现代 Web 应用程序所需的工具 Webpack，Babel，ESLint，Mocha，Karma，Grunt 等等都是压倒性的 - 我应该使用什么以及哪些工具正在做什么？ 我找到了这个网络漫画，它完美地展示了当今 Web 开发者的挣扎。

![Javascript Fatigue — What it feels like to learn Javascript](https://cdn-images-1.medium.com/max/1600/1*1akEKXC95jhmIudAayITPA.png)

<center>Javascript的疲劳  — 学习Javascript的感受</center>

所有这些，每个 Javascript 开发者在深入使用市面上的任何框架或库之前，所需要做的是先了解所有这些的底层基本原理。大多数 JS 开发者可能都听说过 V8 这个术语，即 Chrome 的运行时环境，但有些人甚至可能不知道这意味着什么，这是做什么的。我最初作为开发人员的职业生涯的第一年，对所有这些花哨的术语都知之甚少，因为那时更多的是关于怎么让工作先完成。但这并不能满足我对 Javascript 如何能够完成所有这些工作的好奇心。我决定深入挖掘，围绕谷歌搜索，并发现了一些好的博客文章，包括 [Philip Roberts](https://twitter.com/philip_roberts) 在 JSConf 上关于[事件循环](https://www.youtube.com/watch?v=8aGhZQkoFbQ)的精彩演讲，所以我决定总结我的学习并分享它。由于有很多事情需要了解，我将文章分为两部分。本部分将介绍常用术语，第二部分介绍所有术语之间的联系。

Javascript 是单线程单一并发语言，这意味着它一次只能处理一个任务或一段代码。它有一个单独的调用栈，它与堆之类的其他部分一起构成了 Javascript 并发模型（在 V8 内部实现）。 我们首先来看看这些术语：

<div style="text-align:center">
  <img src="https://cdn-images-1.medium.com/max/1600/1*ZSFHnq9iMHIApVLcgwczPQ.png">
</div>
<center>JS模型可视化<a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop">(来源)</a></center>

**1. 调用栈:** 它是一个记录函数调用的数据结构，基本上是程序中的位置。 如果我们调用一个函数来执行，我们将一些东西推送到栈，当我们从函数返回时，我们弹出栈的顶部。

<div style="text-align:center">
  <img src="https://cdn-images-1.medium.com/max/1600/1*E3zTWtEOiDWw7d0n7Vp-mA.gif">
</div>
<center>JS调用栈可视化</center>

当我们运行上面的文件时，我们首先查找将启动所有执行的 main 函数。 在上面，从`console.log(bar(6))`开始，它被推送到栈，它上面的下一帧包含了`bar`的参数和局部变量，`bar`再调用函数`foo`，`foo`被推送到栈的顶部，`foo`立即返回，因此弹出栈，然后弹出`bar`，最后弹出`控制台语句`打印输出。 所有这些都是瞬间（毫秒）完成的。

你一定在浏览器的控制台看到过长长的红色错误栈追踪，它基本上表示调用栈的当前状态和就像栈一样以从上到下的方式展示函数在哪里失败。看下图：

<div style="text-align:center">
  <img src="https://cdn-images-1.medium.com/max/1600/1*JctnBGRAYmQQPeMsgXUi0A.png">
</div>
<center>错误栈追踪<a href="https://www.youtube.com/watch?v=8aGhZQkoFbQ">(来源)</a></center>

有时，我们进入一个无限循环，因为我们递归地多次调用一个函数，而对于 Chrome 浏览器，栈的大小有 16,000 帧的限制，超过它只会为你杀掉东西并抛出`达到最大栈数错误（下图）`。

<div style="text-align:center">
  <img src="https://cdn-images-1.medium.com/max/1600/1*tqkykdU69DFrxi82JOWLbQ.png">
  <div><a href="https://www.youtube.com/watch?v=8aGhZQkoFbQ">(来源)</a></div>
</div>

**2. 堆:** 对象分配在堆中，即主要非结构化内存区域。所有对变量和对象的内存分配发生在这里。

**3. 队列:** 一个 JavaScript 运行时包含了一个待处理的消息队列。每一个消息都有一个为了处理这个消息相关联的函数。当栈具有足够的容量时，从队列中取出消息并进行处理，该消息包括调用相关联的函数（从而创建初始栈帧）。当消息执行完时，栈又变回空的。基本来说，在给定回调函数的情况下，这些消息会响应外部异步事件（例如单击鼠标或接收对 HTTP 请求的响应）进行排队。 例如，如果用户单击按钮并且未提供回调函数 - 则不会将任何消息排入队列。

## 事件循环

基本上，当我们评估 JS 代码的性能时，栈中的一个函数会使其变慢或变快，`console.log()` 将会很快，但是使用`for`或`while`对数千或数百万行代码执行迭代将会变慢，并将保持堆栈被占用或阻塞。这被称为`阻塞脚本`，您已经在网页速度洞察中读过或听说过。

网络请求可能很慢，图片请求可能很慢，但幸运的是服务器请求可以通过 AJAX（一个异步函数）完成。如果假设，这些网络请求都通过同步来完成，那会怎么样呢？请求发送到某个服务器，也就是某个地方的一台计算机或机器。现在，计算机发送回响应可能很慢，同时，如果我点击了一个按钮，或是有些其他的需要渲染，什么都不会发生，因为栈被阻塞了。在像 Ruby 这样的多线程语言里，这是可以被处理的。但是在像 Javascript 这样的单线程语言里，这是不可能的，除非栈里的函数返回了一个值。因此网页会崩溃，因为浏览器什么也做不了。这是很不理想的，如果我们希望给用户流畅的 UI。那么我们该怎么处理呢？

> “Concurrency in JS— One Thing at a Time, except not Really, Async Callbacks”

<div align=center>
  <img src="https://cdn-images-1.medium.com/max/1600/1*nbXbMf8R6-iM7vzx9ezdgg.png">
</div>

最简单的解决方式就是使用异步回调，这意味着我们执行一部分代码，再提供一个稍后执行的回调。想必我们都遇到过像使用`$.get(),setTimeout(), setInterval(), Promise等`的 AJAX 的异步回调。所有这些异步回调都不是立即执行，而是某个时间之后再执行，所以不能被立即推进栈内，不像`console.log()`或数学运算那样的同步函数。那么它们到底去了哪？又是怎么被处理的？

<div align=center>
  <img src="https://cdn-images-1.medium.com/max/1600/1*QZkRG3HtuqrS3FDucnryKw.png">
</div>

如果我们在 JS 里看到一个像上面代码里的网络请求：

1. 执行请求函数，将`onreadystatechange`事件中的匿名函数作为回调执行，以便在将来某个时间响应可用时执行。

2. "Script call done"被立即打印在控制台上

3. 在将来的某个时间，响应回来了，我们的回调被执行，响应体被打印在控制台上。

调用者与响应的分离允许 JavaScript 运行时在等待异步操作完成和触发回调时执行其他操作。 `2` 这是浏览器 API 启动并调用其 API 的地方，这些 API 基本上是由在 C ++中实现的浏览器创建的线程，用于处理 DOM 事件，http 请求，setTimeout 等异步事件。（知道这一点后，在 `Angular 2` 中，使用了`Zones`，候补丁这些 API 以引起运行时更改检测，我现在可以得到一个画面，他们是如何实现它的。）

现在这些 WebAPIs 本身不能把执行代码放到栈上，如果可以，那么它们就会随机的出现在你的代码里。上面讨论的消息回调队列显示了方法。`3` 任一 WebAPI 执行完就把回调函数推进队列。现在`事件循环`负责执行队列里的这些回调，并在栈为空的时候把它推进栈。`4` 事件循环的基本工作就是观察栈和任务队列，当它看到栈空了的时候就把队列里的第一项推进栈。在处理任何其他消息之前，将完全处理每个消息或回调。

```javascript
while (queue.waitForMessage()) {
  queue.processNextMessage();
}
```

<div align=center>
  <img src="https://cdn-images-1.medium.com/max/1600/1*-MMBHKy_ZxCrouecRqvsBg.png">
</div>
<center>JS事件循环可视化描述</center>

在 Web 浏览器中，只要事件发生并且附加了事件监听器，就会添加消息。 如果没有监听器，则该事件将丢失。 因此，点击带有点击事件处理程序的元素将添加消息 - 与任何其他事件一样。 调用此回调函数充当调用栈中的初始帧，并且由于 JavaScript 是单线程的，因此在调用栈上的所有调用返回之前，将停止进一步的消息轮询和处理。 后续（同步）函数调用将新的调用帧添加到堆栈。
