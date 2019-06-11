<!--
 * @Author: Hom Yan
 * @Date: 2019-06-10 17:44:12
 * @LastEditors: Hom Yan
 * @LastEditTime: 2019-06-11 21:25:53
 -->

# 调度：setTimeout 和 setInterval

[Scheduling: setTimeout and setInterval](https://javascript.info/settimeout-setinterval) by `JAVASCRIPT.INFO` on 2019-05-19

我们可能想要执行一个函数，但不是现在，而是过了一个特定的时间之后。这就称作“计划一个调用”。

有两种方法来实现：

- `setTimeout` 允许在特定的时间间隔之后执行一次函数
- `setInterval` 允许在特定的时间间隔定期的执行函数

这些方法不是 JavaScript 规范的一部分。但大多数环境都有内部调度器，并提供这些方法。特别是，浏览器和 Node.js 都支持。

## setTimeout

语法：

```javascript
let timerId = setTimeout(func|code, [delay], [arg1], [arg2], ...)
```

参数：

`func|code`  
函数或着代码字符串。通常，都是一个函数。由于历史原因，可以传入代码字符串，但并不推荐。

`delay`  
执行之前的延时，以毫秒为单位，默认是 0。

`arg1`, `arg2`...  
函数的参数(IE9 以下不支持)

例如，这段代码会在 1 秒之后调用`sayHi()`:

```javascript
function sayHi() {
  alert('Hello');
}

setTimeout(sayHi, 1000);
```

携带参数：

```javascript
function sayHi(phrase, who) {
  alert(phrase + ', ' + who);
}

setTimeout(sayHi, 1000, 'Hello', 'John'); // Hello, John
```

如果第一个参数是个字符串，那么 JavaScript 会创建一个函数。

所以，下面同样是有效的：

```javascript
setTimeout("alert('Hello')", 1000);
```

但是不推荐使用字符串，可以用函数来代替，就像这样：

```javascript
setTimeout(() => alert('Hello'), 1000);
```

> **传入一个函数，但不要执行它**  
> 新手有时会犯一个错误，给函数添加括号`()`
>
> ```javascript
> // wrong!
> setTimeout(sayHi(), 1000);
> ```
>
> 这是无效的，因为`setTimeout`期望的是一个函数的引用。这里`sayHi()`执行了函数，并把执行结果传给了`setTimeout`。在我们这个例子中，`sayHi()`的结果是`undefined`(函数没有返回任何东西)，所以没有任何东西被调度。

### 使用 clearTimeout 取消

对`setTimeout`的一次调用会返回一个“定时器标识符” `timeId`，我们可以用来取消执行。

取消的语法：

```javascript
let timerId = setTimeout(...);
clearTimeout(timerId);
```

下面的这段代码中，我们先调度了一个函数，然后取消它。结果就是，什么都没发生：

```javascript
let timerId = setTimeout(() => alert('never happens'), 1000);
alert(timerId); // 定时器标识符

clearTimeout(timerId);
alert(timerId); // 同样的标识符（不会因为取消而变成null）
```

从`alert`的输出结果我们可以看出，在浏览器中定时器标识符就是一个数字。在其他环境中，有可能是其他的。例如，Node.js 返回的是一个带有常用方法的定时器对象。

再一次，这些方法没有统一的规范，所以没关系。

对于浏览器，定时器在 HTML5 标准的[定时器章节](https://www.w3.org/TR/html5/webappapis.html#timers)有描述。

## setInterval

`setInterval`方法的语法和`setTimeout`一样：

```javascript
let timerId = setInterval(func|code, [delay], [arg1], [arg2], ...)
```

所有参数也都是一样的。但与`setTimeout`不同，它不仅运行一次，而且在给定的时间间隔后定期运行。

我们可以调用`clearInterval(timerId)`来停止后续的调用。

下面的例子将每隔 2 秒显示一条信息，5 秒之后会停止。

```javascript
// 每隔2秒重复一次
let timerId = setInterval(() => alert('tick'), 2000);

// 5秒之后停止
setTimeout(() => {
  clearInterval(timerId);
  alert('stop');
}, 5000);
```

> **`alert`显示的时候定时不会停下来**
>
> 大多数浏览器，包括 Chrome 和 Firefox，在显示`alert/confirm/prompt`时，内部的定时器都还在继续“嘀嗒”。
>
> 所以如果你运行上面的代码，`alert`弹出时多等一会儿再关闭，那么下一个`alert`立马会显示出来。弹框之间的真实间隔时长会小于 5 秒。

## 递归 setTimeout

有两种方法可以定期运行。

一种是`setInterval`，另一种是递归`setTimeout`，就像这样：

```javascript
/** instead of:
let timerId = setInterval(() => alert('tick'), 2000);
*/

let timerId = setTimeout(function tick() {
  alert('tick');
  timerId = setTimeout(tick, 2000); // (*)
}, 2000);
```

上面的`setTimeout`在当前的一个（\*）结束时调度下一个调用。

递归`setTimeout`是一种比`setInterval`更灵活的方法。这样可以依据当前调用的结果，来调度不同的下一个调用。

例如，我们需要编写这样的一个服务，每 5 秒向服务端发一个请求获取数据，但是如果服务端超载，它需要不断增加时间间隔到 10, 20, 40 秒。。。

下面是伪代码：

```javascript
let delay = 5000;

let timerId = setTimeout(function request() {
  ...send request...

  if (request failed due to server overload) {
    // increase the interval to the next run
    delay *= 2;
  }

  timerId = setTimeout(request, delay);

}, delay);
```

如果我们调度的函数是很消耗 CPU 的，那么我们可以计算执行所耗费的时间，然后计划更早或更晚执行下一次调用。

**递归`setTimeout`可以保证两次执行之间的延迟，`setInterval`则做不到。**

我们来比较下这两个代码片段。第一个使用`setInterval`:

```javascript
let i = 1;
setInterval(function() {
  func(i);
}, 100);
```

第二个使用递归`setTimeout`:

```javascript
let i = 1;
setTimeout(function run() {
  func(i);
  setTimeout(run, 100);
}, 100);
```

对于`setInterval`内部调度器将每 100 毫秒运行一次`func(i)`:

<img src="https://javascript.info/article/settimeout-setinterval/setinterval-interval@2x.png" width="550px" alt="setInterval">

你意识到了吗？

**`setInterval`两次调用`func`之间的真实延迟其实比代码里的更短！**

这是正常的，因为`func`的执行时间消耗了一部分时间间隔。

有可能`func`的执行时间比我们预期的要长，甚至超过 100 毫秒。这种情况下，引擎会等待`func`执行完成，然后检查调度器，如果时间到了，便会立即执行。在极端情况下，如果函数的执行时间总是超过延时的时间，那么调用根本就不会暂停。

下图是递归`setTimeout`:

<img src="https://javascript.info/article/settimeout-setinterval/settimeout-interval@2x.png" width="550px" alt="setTimeout">

**递归`setTimeout`保证了固定的延时(这里是 100 毫秒)**

> **垃圾回收**
>
> 当一个函数被传递给`setInterval/setTimeout`，一个内部的引用就被创建并保存在调度器中。这样可以避免函数被垃圾回收，就算已经没有其他的引用。
>
> ```javascript
> // 函数一直保留在内存中，直到调度器调用它
> setTimeout(function() {...}, 100);
> ```
>
> 对于`setInterval`，函数会一直保留在内存中，直到`clearInterval`被调用。
>
> 还有一个副作用。函数引用外部词法环境，因此，只要它存在，外部变量也会一直存在。他们会耗费的内存比函数本身要多。所以，当我们不再需要调度函数，最好取消它，就算它很小。

## setTimeout(...,0)

有一个特殊的用例：`setTimeout(func, 0)`，或`setTimeout(func)`。

计划尽快的执行`func`，但调度器只会等到当前代码执行完成才调用它。

因此函数被安排在“当前代码之后”运行。换句话说，<i>异步</i>。

举个例子，下面会先输出“Hello”，然后立即“World”。

```javascript
setTimeout(() => alert('World'));

alert('Hello');
```

第一行“0 毫秒之后把调用放进计划表”。但调度器只会在当前代码完成之后才会“检查计划表”，所以`"Hello"`在前，`"World`在后。

### 分割消耗 CPU 的任务

有一个技巧是用`setTimeout`来分割高 CPU 消耗的任务。

举个例子，一个语法高亮的脚本(当前页面用来给代码实例着色)是很吃 CPU 的。为了高亮代码，它需要分析，创建很多着色的元素，并添加到文档中 - 长文本会消耗很大。有可能会导致浏览器“挂起”，这是不可接受的。

那么我们可以把长文本分片。先 100 行，然后通过`setTimeout(..., 0)`再 100 行，以此类推。

为了更清晰点，我们来考虑一个更简单的例子。假设我们有一个函数，需要从`1`计数到`1000000000`。

如果你运行它，CPU 将挂起。对于服务端 JS 这是很明显的，如果你在浏览器运行它，然后尝试在页面上点击其他的按钮 -- 你会看到整个 JavaScript 实际上暂停了，在它完成之前其他操作都不会生效。

```javascript
let i = 0;

let start = Date.now();

function count() {
  // do a heavy job
  for (let j = 0; j < 1e9; j++) {
    i++;
  }

  alert('Done in ' + (Date.now() - start) + 'ms');
}

count();
```

浏览器甚至会显示"the script takes too long"的警告（希望不会，因为这个数并不是特别大）。

我们来把这个任务通过嵌套的`setTimeout`做拆分：

```javascript
let i = 0;

let start = Date.now();

function count() {
  // do a piece of the heavy job (*)
  do {
    i++;
  } while (i % 1e6 != 0);

  if (i == 1e9) {
    alert('Done in ' + (Date.now() - start) + 'ms');
  } else {
    setTimeout(count); // schedule the new call (**)
  }
}

count();
```

现在浏览器界面在“计数”期间是完全可用的。

我们做了一部分任务`(*)`：

1. 先运行：`1...1000000`
2. 然后再运行：`1000001...2000000`
3. 以此类推，`while`会检查`i`是否能被`1000000`整除

然后，如果还没完成，则在`(**)`中安排下一个调用。

`count`执行之间的暂停刚好提供了足够的“喘息”时间，让 JavaScript 引擎可以处理其他事，响应用户的操作。

值得注意的是，两种变体 - 无论是否通过 `setTimeout` 分割作业 - 在速度上都具有可比性。 总计数时间没有太大差异。

为了让他们更接近，我们来做个优化提升。

我们将把调度移动到`count()`的开头位置：

```javascript
let i = 0;

let start = Date.now();

function count() {
  // move the scheduling at the beginning
  if (i < 1e9 - 1e6) {
    setTimeout(count); // schedule the new call
  }

  do {
    i++;
  } while (i % 1e6 != 0);

  if (i == 1e9) {
    alert('Done in ' + (Date.now() - start) + 'ms');
  }
}

count();
```

现在，当我们开始运行`count()`，并看到需要更多的`count()`时，我们会在完成任务之前立即安排。

如果你运行它，很容易注意到它花费的时间要少得多。

> **浏览器内嵌定时器的最小延迟**
>
> 在浏览器中，有一个内嵌定时器运行频率的限制。HTML5 标准提到：“在 5 个嵌套定时器之后，间隔被强制为至少 4 毫秒。”
>
> 我们用下面的例子来示范下这到底意味着什么。`setTimeout`的调用被安排在自身的`0毫秒`之后。每个调用都在`times`数组里记录了真实的时间。真实的延迟会是怎么样的？我们来看下：
>
> ```javascript
> let start = Date.now();
> let times = [];
> setTimeout(function run() {
>   times.push(Date.now() - start); // remember delay from the previous call
>   if (start + 100 < Date.now()) alert(times);
>   // show the delays after 100ms
>   else setTimeout(run); // else re-schedule
> });
> // 输出示例:
> // 1,1,1,1,9,15,20,24,30,35,40,45,50,55,59,64,70,75,80,85,90,95,100
> ```

第一次定时器立即运行（正如规范中所述），然后延迟发挥作用，我们看到 9,15,20,24 ....

这种限制比较久远，许多脚本依赖它，因此它存在是出于历史原因。

对于服务器端 JavaScript，该限制不存在，并且存在其他方法来调度立即异步作业，例如针对 Node.js 的`process.nextTick`和`setImmediate`。 所以这个概念只是浏览器特定的。

### 让浏览器可渲染

另一个给浏览器脚本拆分繁重任务的好处是，我们可以显示一个进度条或其他东西给用户。

通常浏览器会在当前代码执行之后进行所有的“repainting(重绘)”。所以如果我们在一个单一的大函数里改变很多元素，这些改变不会被渲染，直到它完成。

下面是演示：

```javascript
<div id="progress"></div>

<script>
  let i = 0;

  function count() {
    for (let j = 0; j < 1e6; j++) {
      i++;
      // put the current i into the <div>
      // (we'll talk about innerHTML in the specific chapter, it just writes into element here)
      progress.innerHTML = i;
    }
  }

  count();
</script>
```

如果你运行它，对`i`的改变直到整个计数完成之后才会显示出来。

如果我们用`setTimeout`来分割，那么这些改变就会在运行的间隙被应用，看起来会好很多：

```javascript
<div id="progress"></div>

<script>
  let i = 0;

  function count() {

    // do a piece of the heavy job (*)
    do {
      i++;
      progress.innerHTML = i;
    } while (i % 1e3 != 0);

    if (i < 1e9) {
      setTimeout(count);
    }

  }

  count();
</script>
```

现在就可以在`<div>`中看到不断变大的`i`。

## 总结

- 方法`setInterval(func, delay, ...args)`和`setTimeout(func, delay, ...args)` 可以在延迟`delay`毫秒之后定期的/一次运行`func`
- 要取消执行，可以调用`clearInterval/clearTimeout`并传入`setInterval/setTimeout`的返回值
- 嵌套的`setTimeout`比`setInterval`更灵活，并且可以保证两次执行之间的最小间隔
- 零延时调度`setTimeout(func, 0)`(与`setTimeout(func)`一样)是用来调度调用“尽可能快，但是在当前代码完成之后”

`setTimeout(func)`的一些用例：

- 拆分高 CPU 消耗的任务，这样脚本不会“挂起”
- 让浏览器在进程执行的时候还能处理其他事（比如画进度条）

要注意所有的调度方法都不能保证确切的延时。我们不能依赖于它。

例如，浏览器的定时器可能会因为很多原因慢下来：

- CPU 超载
- 浏览器标签处在后台模式
- 笔记本在使用电池电量

所有这些都可能增加最小定时器方案（最小延迟）到 300 毫秒甚至 1000 毫秒，具体取决于浏览器和设置。
