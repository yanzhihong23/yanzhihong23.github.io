# 为什么在 JavaScript 中使用三重等号？

[原文](https://www.impressivewebs.com/why-use-triple-equals-javascipt/)

“判断两个变量是否相等是在编程里最重要的操作符之一” 这是 [Nicholas Zakas](http://www.nczonline.net/) 在他的书
[JavaScript for Web Developers](http://www.amazon.com/Professional-JavaScript-Developers-Wrox-Programmer/dp/047022780X/) 提到的。

换句话说，在你的脚本里很可能会有像这样的代码：

```javascript
if (x == y) {
  // do something here
}
```

或者，你可能更符合最佳实践，像这样：

```javascript
if (x === y) {
  // do something here
}
```

这两个例子的区别是第二个例子使用了三重等号操作符，也称作『严格相等』或者『完全相等』。

想要遵循最佳实践的 JavaScript 初学者可能会使用三重等号而不是双重，但他们可能并不能完整理解两者之间的区别或者说为什么坚持使用三重等号是那么重要。

## 区别

在一个使用双重等号操作符的比较里，如果相比较的两个东西是相等的，结果会返回`true`。但有一点很重要：如果被比较的是两个不同类型的值，那么类型强制转换就会发生。

每一个 JavaScript 值都属于一个特定的『类型』。这些类型有：number, string, boolean, function 和 object。所以如果你尝试比较一个字符串和一个数值，在做比较之前浏览器将会把字符串转换为数值。类似的，如果你比较`true`或`false`和一个数值，`true`或`false`会各自被转换为`1`或`0`。

这将会带来不可预测的结果，看下下面的例子：

```javascript
console.log(99 == '99'); // true
console.log(0 == false); // true
```

虽然这刚开始看起来是个好事，因为浏览器似乎帮了你的忙。但这会导致问题。例如：

```javascript
console.log(' \n\n\n' == 0); // true
console.log(' ' == 0); // true
```

鉴于此，绝大多数 JavaScript 专家建议一直使用三重等号，永远不要使用双重等号。

现在你可能已经猜到，三重等号操作符不会做类型强制转换。所以无论何时你使用三重等号，你在做的是对真实的值的精确比较。你要确保的是值是『严格相等』或者『完全相等』。

这意味着，使用三重等号，上面的所有例子都会有正确的结果：

```javascript
console.log(99 === '99'); // false
console.log(0 === false); // false
console.log(' \n\n\n' === 0); // false
console.log(' ' === 0); // false
```

## 不等号

同样的规则可以应用在不等表达式上。只是这次，不是三重等号与双重等号，而是使用双等于单等。

上面一样的例子，这次我们使用`!=`操作符：

```javascript
console.log(99 != '99'); // false
console.log(0 != false); // false
console.log(' \n\n\n' != 0); // false
console.log(' ' != 0); // false
```

注意到每种情况的期望结果应该是`true`，然后结果都是`false`，因为发生了类型强制转换。

如果我们改为双等，就能得到期望的正常结果：

```javascript
console.log(99 !== '99'); // true
console.log(0 !== false); // true
console.log(' \n\n\n' !== 0); // true
console.log(' ' !== 0); // true
```

## 总结

正如前面提到的，你可能早就只使用三重等号。在研究这篇文章时，对这个概念我自己还是学到了一些。

我觉得最好的总结还是来自 Zakas, 在推荐一直使用严格相等之后，他说：“这有助于维护你代码里的数据类型”。
