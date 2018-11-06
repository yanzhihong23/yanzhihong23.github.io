# 你需要了解的 JavaScript Number 类型

[原文](https://medium.com/dailyjs/javascripts-number-type-8d59199db1b6)

<div style="text-align:center">
 <img src="https://cdn-images-1.medium.com/max/1600/1*aGnwvStsrH3OnrwdlHvsgQ.jpeg">
</div>
<center>为什么 0.1+0.2 不等于 0.3 而 9007199254740992 等于 9007199254740993 ?</center>

很多静态语言像 Java 或 C 对于数字都有不同的数据类型。比如，如果你需要存储一个范围在[-128;127]之间的整数，Java 中你可以使用`byte`，C 中你可以使用`char`，这两个都只占用一个字节。如果你需要存储更大的整数，你可以使用`int`或`long`这两个数据类型，分别占用 4 个和 8 个字节。还有一些单独的数据类型可用于存储带小数部分的数字 - `float`占用 4 个字节，`double`占 8 个字节。 这些通常被称为浮点格式，稍后我们将看到此名称的来源。

但是在 JavaScript 里并没有那么多的数字类型。基于 ECMAScript 标准，只有`双精度64位二进制格式 IEEE 754值`这一种数字类型。这种类型用来存储整数和小数，相当于 Java 和 C 里的`double`类型。有些新接触 JavaScript 的开发者没有意识到这点，认为`1`用 64 位存储应该是这样：

```
0000000000000000000000000000000000000000000000000000000000000001
```

然而实际上它是这么存储的:

```
0011111111110000000000000000000000000000000000000000000000000000
```

这种误解可能会造成很大的困惑。举个例子，用 Java 写了这个循环:

```java
for (int i=1; 1/i > 0; i++) {
    System.out.println("Count is: " + i);
}
```

这会运行多久呢？不难看出在第一个循环之后就会停下。在第二个循环里，计数器`i`将被增加到`2`，`1/2`将会生成`0.5`，而`0.5`将被会截断为`0`，因为计数器`i`是一个整数类型，所以`1/2 > 0`的结果会是`false`。
现在再来看下，如果我们用 JavaScript 来写同样的循环，会是怎么样的呢？

```javascript
for (var i = 1; 1 / i > 0; i++) {
  console.log('Count is: ' + i);
}
```

事实证明，这个循环永远不会停止，因为 `1/i` 的结果不会被评估为整数，而是作为浮点，这就导致了非常有趣的行为。 想了解更多？ 请继续阅读。

那些对 JavaScript 工作方式不熟悉的通常会提到另外一个意想不到的行为，一旦理解了这门语言就很好解释了。`0.1`加上`0.2`会生成`0.30000000000000004`，这意味着`0.1+0.2`不等于`0.3`。有关这个怪异行为的问题经常出现，stackoverflow 人员不得不添加特殊注释：

![](https://cdn-images-1.medium.com/max/1600/1*SnYf-ofdeR3Fe-pbU341cg.png)

有趣的是，这种行为经常归因于 JavaScript，实际上任一使用浮点格式作为数字类型的语言都有同样的问题。这意味着，如果你在 Java 或 C 里使用`float`或`double`，将会看到同样的结果。另一个有趣的点是，`0.1+0.2`的结果并不是浏览器控制台上看到的`0.30000000000000004`，实际上是`0.3000000000000000444089209850062616169452667236328125`。

这篇文章我将解释浮点数的工作原理，更深入的探索`for`循环以及上面提到的`0.1+0.2`的例子。

> 值得一提的是 BigInt，它是 JavaScript 中一个新的数字基元，可以表示任意精度的整数。 使用`BigInt`，您可以安全地存储和操作大整数，甚至超出 `Number` 的安全整数限制。 它于今年(2016)在 [V8](https://v8.dev/blog/bigint) 中推出，并在 Chrome 67+和 Node v10.4.0 +中得到支持。 你可以在[这里](https://developers.google.com/web/updates/2018/05/bigint)读更多关于它的内容。

## 科学记数法

在开始讨论浮点数和 IEEE754 标准前，我们需要先来看下数字在科学记数法中的表示法。在一般形式中，科学记数法中的数字可以表示如下：

<center><span style="font-size:30px;color:#42b983">significant</span> <span style="font-size: 20px;color:#666">x</span> <span style="font-size: 30px;color:#e96900">base</span><sup style="font-size: 18px;color:#22a2c9">exponent</sup> </center>

`significand`(有效数)显示有效位数，通常也被称作尾数或精度。0 不被认为是有效的，它们只是占位。`base`指定数字系统基数，例如：`10`代表十进制，`2`代表二进制。 `exponent`(指数)定义小数点必须向左或向右移动多少个位置才能获得原始数字。

任何数字都可以用科学记数法来表示。例如，数字`2`的十进制和二进制可以这样表示：

<div style="font-size: 24px">
  <center> 2<sub>10</sub> = 2·10<sup>0</sup></center>
  <center> 10<sub>2</sub> = 10·2<sup>0</sup></center>
</div>

指数为 0 很清楚的表明，不需要额外的操作就可以得到原始数。来看下另外一个例子，数字`0.00000022`。这里的有效数是`22`，我们来移动一下 0：

<div style="text-align:center">
  <img src="https://cdn-images-1.medium.com/max/1600/1*AsTGOQl6gbLWCVBgjrSuUw.png">
</div>

上面的计算论证了为什么小数点右移的时候指数变小。因此，通过执行乘法，我们将原始数字细化为仅具有有效数字:

<div style="text-align:center">
  <img src="https://cdn-images-1.medium.com/max/1600/1*HKEcO_6mfIU2EZSlobFlpw.png">
</div>

由于我们使用了`8`次乘法，所以需要使用除法来补偿它，这就是指数`-8`的来源。同样的过程，只是这次除法是用来获取有效数，在`22300000`上执行：

<div style="font-size: 24px">
  <center> 22300000 = 223·10<sup>5</sup></center>
</div>

这次小数点向左移动了，这样指数变大了。如你所见，科学记数法是一种很容易表示非常大或小的数字的方法。 取决于指数，有效数可以表示整数或具有小数部分的数字。 转换为原始数字时，负指数需要将小数点向左移动。 正指数需要向右移动，通常表示大整数。

理解数字的标准化形式也很重要。 当数字以小数点前的一个非零十进制数字用科学记数法书写时，数字会被标准化。 因此，如果我们采用原始数字并以标准化形式表示它们，它们将具有以下表示：

<div style="font-size: 24px">
  <center> 0.00000022 = 2.2·10<sup>-7</sup></center>
  <center> 22300000 = 2.23·10<sup>7</sup></center>
</div>

你可能已经猜到，二进制数字的小数点前始终都有`1`。以标准化数字表示的数字使得按数量级来比较数字变得更容易。

科学记数法可以被认为是数字的浮点表示。 术语`浮点`指的是数字的小数点可以“浮动” - 它可以放在相对于数字有效数字的任何位置。 正如我们所了解的那样，原始位置由指数表示。

## IEEE754 标准浮点数

IEEE 浮点运算标准（IEEE 754）定义了许多与浮点运算相关的内容，但出于我们的探索目的，我们只关注数字的存储，舍入和添加方式。 我写了一篇非常详细的文章，解释[如何舍入二进制数](https://medium.com/@maximus.koretskyi/how-to-round-binary-fractions-625c8fa3a1af#.e9nykp8sj)。 舍入是一种常见的操作，当选定的格式不允许有足够的位来存储数字时，就会发生舍入。 这是一个重要的主题，所以要掌握它的机制。 现在，我们来看看如何存储数字。 所有示例都将主要用于二进制系统中的数字。

### 理解数字是怎么存储的

标准定义了 2 种用的最多的格式 -- 单精度和双精度。它们的区别在于占用的位数和可存储的数字范围。把科学记数法格式的数字转换为 IEE754 形式的方法都是一样的，只是给尾数和指数分配的位数不一样。

IEEE754 浮点分配位用来存储数字符号、尾数（有效数）和指数。下图展示了 JavaScript 里使用的双精度格式的位是如何分配的：

![](https://cdn-images-1.medium.com/max/1600/1*V0mPKZLKeAOxZMwspz3Htw.png)

符号位获得 1 位，指数 11 位，并为尾数（有效数）分配 52 位。 以下是显示为每种格式分配的位数的表：

| 名字   | 总位数 | 指数 | 有效数 |
| ------ | ------ | ---- | ------ |
| 单精度 | 32     | 8    | 23     |
| 双精度 | 64     | 11   | 52     |

指数以`偏移二进制格式`存储。我已经写过一篇[详细的文章](https://medium.com/@maximus.koretskyi/the-mechanics-behind-exponent-bias-in-floating-point-9b3185083528#.zacphtue3)解释这种格式以及它与 2 种补码的不同之处。请花点时间理解下这个主题，因为我将用它来把数字转换为浮点格式。

### 整数是怎么存储的

要了解位的分配方案，我们来看下整数`1`和`3`是怎么存储的。数字`1`在任何数字系统中都表示`1`，所以不需要做转换。用科学记数法表示为：

<center style="font-size: 24px">1.0 · 2<sup>0</sup></center>

这里我们有一个尾数为`1`和一个指数为`0`。通过这些信息，你可能以为这个数用浮点表示为：

<center style="font-size: 22px">0 00000000000 0000000000000000000000000000000000000000000000000001</center>

一起来看下真的是这样的吗。遗憾的是，并没有内置的 JavaScript 函数可以让你看到数字的位模式。但我编写了一个简单的 JavaScript 函数，可以查看数字的存储方式，而不管计算机的字节顺序如何。下面是代码：

```javascript
function to64bitFloat(number) {
  var i,
    result = '';
  var dv = new DataView(new ArrayBuffer(8));

  dv.setFloat64(0, number, false);

  for (i = 0; i < 8; i++) {
    var bits = dv.getUint8(i).toString(2);
    if (bits.length < 8) {
      bits = new Array(8 - bits.length).fill('0').join('') + bits;
    }
    result += bits;
  }

  return (
    result.substr(0, 1) + ' ' + result.substr(1, 11) + ' ' + result.substr(12)
  );
}
```

所以，使用它你可以看到数字`1`的存储看起来是这样的：

<center style="font-size:22px">0 01111111111 0000000000000000000000000000000000000000000000000000</center>

与上面猜想的完全不一样，尾数里并没有数字，指数位里有很多`1`。现在我们来看下为什么是这样的。我们需要理解的第一件事是每个数字都是从标准化的科学形式中翻译出来的。这有什么好处呢？如果小数点前的第一个数字一直都是`1`，那么我们就没有必要去存储它，这样就可以多 1 位可以用来存储尾数。当执行数学运算的时候，第一位的`1`会被硬件添加回来，由于数字`1`在标准化格式下小数点后面没有数字，而且小数点前一位不存储，这样我们没有任何东西可以放到尾数里，所以这些位都是 0。  
现在我们再来看下指数里的`1`是怎么来的。前面我提到过指数是通过偏移二进制存储的。如果我们这样来算偏移：

<center style="font-size:24px">K = 2<sup>n-1</sup> - 1 = 1023<sub>10</sub> = 01111111111<sub>2</sub></center>

可以看到这正是上面我们看到的。因此，偏移二进制下，存储的实际是`0`。如果不清楚偏移是怎么给我们`0`的话，可以看下我这篇关于[偏移二进制](https://medium.com/@maximus.koretskyi/the-mechanics-behind-exponent-bias-in-floating-point-9b3185083528#.2q5qxmdn3)的文章。

运用上面学到的信息，我们来试下数字`3`的浮点格式表示。在二进制中，它表示为`11`。如果你不记得了，可以看下我写的这篇关于[十进制-二进制转换算法](https://medium.com/@maximus.koretskyi/the-simple-math-behind-decimal-binary-conversion-algorithms-d30c967c9724#.vkz0k3jon)很详细的文章。通过格式化，`3`有这样的格式(二进制数字)：

<center style="font-size:24px">11 · 10<sup>0</sup> = 1.1 · 10<sup>1</sup></center>

小数点之后我们只有一个数字`1`将被存储到尾数里。上面解释过的，小数点前的一位不存储。而且，标准化也给我们指数`1`。我们来计算下载偏移二进制中试怎么表示的，这样我们就有了所有需要的信息：

<div style="font-size:24px;text-align:center">
  K = 2<sup>n-1</sup> -1 = 1023  <br>
  1 + 1023 = 1024  <br>
  1024<sub>10</sub> = 10000000000<sub>2</sub>
</div>

关于尾数需要记得的一件事试，数字是完全按照它们在科学格式里的顺序存储的，即小数点开始从左到右。记得这些，我们把所有的数字放入浮点表示：

<center style="font-size:22px">0 10000000000 1000000000000000000000000000000000000000000000000000</center>

## 为什么 0.1+0.2 不是 0.3

现在我们知道了数字是如何存储的，我们再来看下这个经常引用的例子。下面是一个快速的解释：

> 只有分母为 2 的幂的分数才可以以二进制形式有限的表示。由于 0.1(1/10)和(1/5)的分母都不是 2 的幂，这两个数字都不能通过二进制有限(完整)的表示。为了存储它们为 IEEE-754 浮点格式，必须对可用的尾数位做舍入，即半精度 10 位、单精度 23 位、双精度 52 位。根据可用的精度位数，0.1 和 0.2 的浮点近似值可能略小于或大于相应的十进制表示，但永远不会相等。基于这个事实，你永远无法得到 0.1+0.2==0.3

这个解释对一些开发者可能已经足够了，但是要看清幕后发生了什么的最好方式是你自己来完成计算机所做的所有计算。而这正是我现在要做的事。

### 用浮点格式表示 0.1 和 0.2

我们来看下`0.1`在浮点格式吓的位模式。我们要做的第一件事就是把`0.1`转换为二进制，这可以使用乘以 2 的算法来完成。在我的文章[十进制二进制转换算法]()里解释过这个机制。如果我们把`0.1`转换为二进制，我们将得到一个无线的分数：

<div style="text-align:center">
  <img src="https://cdn-images-1.medium.com/max/1600/1*47litx2SPldE4_bhytk-ug.png">
</div>

下一步就是用标准科学记数法来表示这个数：

<div style="text-align:center">
  <img src="https://cdn-images-1.medium.com/max/1600/1*qEBApVomEABGVK4eXh09wA.png">
</div>

由于尾数只有 52 位，我们需要把小数点后面的无线数字舍入为 52 位。

<div style="text-align:center">
  <img src="https://cdn-images-1.medium.com/max/1600/1*E1aUp_0BuTREWdtUquhulA.png">
</div>

使用 IEEE-754 标准定义的舍入规则，我的文章[二进制舍入](https://medium.com/@maximus.koretskyi/how-to-round-binary-fractions-625c8fa3a1af#.gj8rdmqul)解释过，我们需要把这个数舍入为：

<div style="text-align:center">
  <img src="https://cdn-images-1.medium.com/max/1600/1*3c2yLvBN5zeQdrVp74-GNg.png">
</div>

剩下的最后一件事就是计算指数的偏移二进制表示：

<div style="text-align:center;font-size:24px">
  K = 2<sup>11-1</sup> - 1 = 1023 <br>
  -4 + 1023 = 1019<sub>10</sub> <br>
  1019<sub>10</sub> = 01111111011<sub>2</sub>
</div>

当放入浮点格式的时候，数字`0.1`就有了下面的位模式：

<div style="text-align:center">
  <img src="https://cdn-images-1.medium.com/max/1600/1*kx17MFmx0gpX_j8LLYp1uA.png">
</div>

我鼓励你自己计算一下`0.2`的浮点表示。你应该会得到下面的科学记数和二进制：

![](https://cdn-images-1.medium.com/max/1600/1*fLRp3gdoDoUbXJ-Fkroo-Q.png)

### 计算 0.1 + 0.2 的结果

如果我们把数字的浮点格式转换为科学格式，我们将得到：

<div style="text-align:center;font-size:22px">
  0.1<sub>10</sub> ≈ 1.1001100110011001100110011001100110011001100110011010 · 2<sup>-4</sup> <br>
  0.2<sub>10</sub> ≈ 1.1001100110011001100110011001100110011001100110011010 · 2<sup>-3</sup>
</div>

要把两数相加，它们需要有一样的指数。规则说明我们需要把小的指数调整为更大的。所以，我们来把指数为`-4`的第一个数调整为像指数为`-3`的第二个数：

<div style="text-align:center;font-size:22px">
  0.1<sub>10</sub> ≈ 0.11001100110011001100110011001100110011001100110011010 · 2<sup>-3</sup>
</div>

现在把两数相加：

![](https://cdn-images-1.medium.com/max/1600/1*uS64AitPYwcx_MhF9IVcZw.png)

现在，把计算结果存为浮点格式，那么我们需要格式化这个结果，必要时做舍入和用偏移二进制计算指数。

![](https://cdn-images-1.medium.com/max/1600/1*FUwRm1QN4oMrvxdYuTQcSw.png)

归一化数正好落在舍入选项之间，所以我们采用打破平局的规则，向**上**舍入到偶数。这给出了以下归一化科学形式的结果数字:

<div style="text-align:center">
  <img src="https://cdn-images-1.medium.com/max/1600/1*I2wM8yj42guOgiurlplCcw.png">
</div>

当我们再转换为浮点格式存储的时候，就有了下面的位模式：

<center style="font-size:22px">0 01111111101 0011001100110011001100110011001100110011001100110100</center>

**当你执行表达式`0.1+0.2`时，这正是存储的位模式**。要得到它，计算机需要做 3 次舍入，即数字本身各一次，第三次是它们的和。当存储简单的`0.3`时，计算机只做了一次舍入。**舍入操作导致了`0.1+0.2`和单独`0.3`存储的位模式的不同**。当 JavaScript 执行比较语句 `0.1+0.2 === 0.3`时，比较的是它们的位模式，由于它们位模式的不同所以返回的结果是`false`。**如果存在这样的格式，即使对于舍入，位模式也是相等的，0.1 + 0.2 === 0.3 将评估为`true`，而不管 0.1 和 0.2 在二进制中不是有限可表示的事实。**

试下用我上面展示的 `to64bitFloat(0.3)`看下数字`0.3`的位模式。这个模式将会和我们上面计算`0.1+0.2`的结果是不一样的。
如果你想知道十进制数的存储位表示，把位组装为指数为 0 的科学形式，然后再转换为十进制数。那么`0.1+0.2`实际存储的十进制数是`0.3000000000000000444089209850062616169452667236328125` 而`0.3`则是`0.299999999999999988897769753748434595763683319091796875`。

## 为什么`for`循环永远不会停下

理解`for`循环永远不会停下的关键在于数字`9007199254740991`。我们来看下这个数字有啥特殊之处。

### Number.MAX_SAFE_INTEGER

如果你在控制台输入 Number.MAX_SAFE_INTEGER，就会得到我们的关键数字 `9007199254740991`。这个数字这么特别在于它有自己的常量？下面是[ECMAScript 语言规范](https://www.ecma-international.org/ecma-262/6.0/#sec-number.max_safe_integer)对它的描述：

> Number.MAX_SAFE_INTEGER 的值是最大的整数 n，使得**n**和**n + 1**都可以精确地表示为 Number 值。 Number.MAX_SAFE_INTEGER 的值是**9007199254740991**（2⁵³-1）。

还有[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_SAFE_INTEGER)上的解释：

> 安全存储的意思是指能够准确区分两个不相同的值，例如 Number.MAX_SAFE_INTEGER + 1 === Number.MAX_SAFE_INTEGER + 2 将得到 true 的结果，而这在数学上是错误的。

要明白的第一点是这并不是最大可以表示的整数。例如，数字`9007199254740994`就可以安全的表示`MAX_SAFE_INTEGER + 3`。通过常量`Number.MAX_VALUE`可以看到最大的可表示的数字，而它是`1.7976931348623157e+308`。可能会让你感到意外的是，有些在`MAX_SAFE_INTEGER`和`MAX_VALUE`的数字不能被表示出来。实际上，在`MAX_SAFE_INTEGER`和`MAX_SAFE_INTEGER + 3`之间有个整数不呢给你被表示，这个数是`9007199254740993`。如果你在控制台输入它，你将看到它被解析为`9007199254740992`。因此，JavaScript 不是使用原始数字，而是将其转换为比原始数字少 1 的数字。

要明白为什么会这样，我们先来看下`9007199254740991 (MAX_SAFE_INTEGER)`在浮点形式里的位表示：

<center style="font-size:22px">0 10000110011 1111111111111111111111111111111111111111111111111111</center>

当转换为科学形式就有了下面的表示：

<div style="text-align:center">
  <img src="https://cdn-images-1.medium.com/max/1600/1*P60vgnZwLYwU1UbaA2pHsg.png">
</div>

现在，像右移动小数点 52 位来取到指数为 0 的二进制数：

<div style="text-align:center">
  <img src="https://cdn-images-1.medium.com/max/1600/1*uDFvnqHBM1-vwao77rjkeQ.png">
</div>

所以，为了存储`MAX_SAFE_INTEGER`，我们用尽了指数为 52 的尾数里的所有空间。由于所有空间都被使用了，为了存储下一个数字，唯一的选项只能是把指数加`1`，变大到`53`。对于指数`53`，我们向右移动小数点`53`位。但是由于尾数里我们只有`52`个数字，我们再末尾添加了`0`。对于指数`54`，就添加两个`0`。`55`就`3`个，以此类推。

这暗示着什么呢？你可能自己已经猜到了。**由于所有大于`MAX_SAFE_INTEGER`的数字都将以 0 结尾，这样大于`MAX_SAFE_INTEGER`的奇数就无法用 64 位浮点表示。**为了能够存储一些，尾数就必须分配超过 52 位。来看下这个实例：

<div style="text-align:center">
  <img src="https://cdn-images-1.medium.com/max/1600/1*9WgZriHQa0yQZFQLWr-E_A.png">
</div>

你可以看到数字`9007199254740993`和`9007199254740995`无法通过`64位`浮点表示。如果你继续这个队列，你会看到有些偶数也无法表示，例如`9007199254740998`。随着指数的增加，无法被存储的数字显著增加。

### 永不停止的循环

我们再来看下`for`循环的例子：

```javascript
for (var i = 1; 1 / i > 0; i++) {
  console.log('Count is: ' + i);
}
```

永远都不会停下。开始的时候我提到过这是因为`1/i`的结果不是作为一个整数，而是一个浮点数。现在你知道了浮点数的工作原理和`Number.MAX_SAFE_INTEGER`，就不难理解为什么永远都不会停下了。

要想让循环停下，计数器`i`就需要达到`Infinity`，这是由于`1/Infinity > 0`会解析为`false`。但这不可能发生。前面我已经解释了为什么有些整数无法存储和被舍入到最接近的偶数。那么，在我们的例子中，JavaScript 给计数器`i`不断加`1`直到`9007199254740993`，也就是`MAX_SAFE_INTEGER+2`。这是第一个无法被存储的整数，这样它就会被舍入成最接近的偶数`9007199254740992`。所以循环被卡在这个数字上。循环无法跳过这个数字，这样我们就得到了一个无限循环。

## 关于 NaN 和 Infinity 的一些话

为了结束这篇文章，我决定对`NaN`和`Infinity`做个非常简短的解释。`NaN`代表`Not a Number`，它和`Infinity`是不一样的，虽然它们在浮点表示和浮点操作里都是作为真实数字来做特殊处理。它们的指数为`1024（11111111111）`而不是`Number.MAX_VALUE`，其指数为`1023（111111111101）`。

由于`NaN`可以通过浮点表示，所以当你在浏览器输入`typeof NaN`得到`number`，就不会感到很惊讶了。它的指数位全部是`1`，尾数位有一个非零的数字：

<center style="font-size:22px">0 11111111111 1000000000000000000000000000000000000000000000000000</center>

有很多数学运算的结果都是`NaN`，比如 `0/0` 或者 `Math.sqrt(-4)`。JavaScript 里有些函数也会返回`NaN`。例如，把字符串作为`parseInt("s")`的参数时，`parseInt`会返回`NaN`。有趣的是，任何与`NaN`的比较运算都返回`false`。例如，下面的这些运算都返回`false`：

```javascript
NaN === NaN;
NaN > NaN;
NaN < NaN;

NaN > 3;
NaN < 3;
NaN === 3;
```

`NaN`，也只有`NaN`，与自身相比较都不相等。JavaScript 有个`isNan()`函数可以用来判断一个值是否为`NaN`。

`Infinity` 是另一个特例，在浮点里被设计来处理溢出和一些数学运算，比如 `1/0`。Infinity 被表示为指数位全为 1，尾数位全为 0：

<center style="font-size:22px">0 11111111111 0000000000000000000000000000000000000000000000000000</center>

对于正`Infinity`，它的符号位是 0，负`Infinity`是 1。这篇 MDN [文章](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Infinity)描述了一些结果是`Infinity`的运算。不像`NaN`，`Infinity`可以安全的用来比较。
