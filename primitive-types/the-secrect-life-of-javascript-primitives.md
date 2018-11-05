# JavaScript 原始类型的秘密生命

[原文](https://javascriptweblog.wordpress.com/2010/09/27/the-secret-life-of-javascript-primitives/)

您可能不知道，但是在 JavaScript 中，每当您与字符串、数字或布尔原始类型交互时，您就会进入一个隐藏的对象阴影和强制转换的世界。所以，掸掉你的夏洛克·福尔摩斯服装，继续读下去吧……

## 基础

对象是属性的聚合。属性可以引用对象或原始类型。原始类型是值，它们没有属性。

> 原始类型也叫做基元类型

JavaScript 中有 5 种原始类型: `undefined`、`null`、`boolean`、`string`和`number`，其他的都是对象。原始类型`boolean`、`string`和`number`可以由它们的对象对应项包装。这些对象分别是`Boolean`、`String`和`Number`构造函数的实例。

```javascript
typeof true; //"boolean"
typeof Boolean(true); //"boolean"
typeof new Boolean(true); //"object"
typeof new Boolean(true).valueOf(); //"boolean"

typeof 'abc'; //"string"
typeof String('abc'); //"string"
typeof new String('abc'); //"object"
typeof new String('abc').valueOf(); //"string"

typeof 123; //"number"
typeof Number(123); //"number"
typeof new Number(123); //"object"
typeof new Number(123).valueOf(); //"number"
```

## 如果原始类型没有属性，那为什么`"abc".length`会返回一个值？

因为 JavaScript 很容易在原始类型和对象之间强制转换。在本例中，为了访问属性长度，将字符串值强制转换为字符串对象。string 对象只使用了几分之一秒，之后就被供奉给了垃圾回收之神——但本着电视发现节目的精神，我们将捕获这个难以捉摸的生物，并将其保存起来供进一步分析...

```javascript
String.prototype.returnMe = function() {
  return this;
};

var a = 'abc';
var b = a.returnMe();

a; //"abc"
typeof a; //"string" (还是一个原始类型)
b; //"abc"
typeof b; //"object"
```

...正如许多善意的科学调查一样，我们现在已经干扰了事物的自然进程，阻止了对象被垃圾回收，只要 b 在附近。海森堡活得好好的 😉

> 注意，在严格的模式下，难以捉摸的生物会逃跑 – 感谢 @DmitrySoshnikov

下面是一个更环保的示例，它在不影响垃圾收集的情况下验证对象类型:

```javascript
Number.prototype.toString = function() {
  return typeof this;
};

(123).toString(); //"object"
```

## 这些对象也能被强制转换为值吗？

是的, 大部分。这种类型的对象仅仅是包装器，它们的值是它们包装的原始类型，它们通常会根据需要强制转换到这个值。有关详细信息，请参阅[本文](https://javascriptweblog.wordpress.com/2010/05/03/the-value-of-valueof/)。

```javascript
//对象强制转换为原始类型
var Twelve = new Number(12);
var fifteen = Twelve + 3;
fifteen; //15
typeof fifteen; //"number" (原始类型)
typeof Twelve; //"object"; (还是对象)

//另外一个对象强制转换为原始类型
new String('hippo') + 'potamus'; //"hippopotamus"

//对象没有强制转换 (因为`typeof`运算符可以处理对象)
typeof new String('hippo') + 'potamus'; //"objectpotamus"
```

遗憾的是布尔对象不那么容易强制转换。更糟糕的是，布尔对象的计算值为`true`，除非其值为`null`或`undefined`。试试这个:

```javascript
if (new Boolean(false)) {
  alert('true???');
}
```

通常您必须显式地请求布尔对象的值。以下内容可能有助于确定值是否为“虚假”的“真实”...

```javascript
var a = '';
new Boolean(a).valueOf(); //false
```

...实际上这样更简单...

```javascript
var a = Boolean('');
a; //false
```

... 甚至这样 ...

```javascript
var a = '';
!!a; //false
```

## 强制转换允许我赋值给原始类型吗？

不可以。

```javascript
var primitive = 'september';
primitive.vowels = 3;

primitive.vowels; //undefined;
```

如果 JavaScript 检测到试图将一个属性分配给一个原始类型，它就会强制转换基础类型为一个对象。但是，与前面的示例一样，这个新对象没有引用，将立即成为垃圾收集的素材。

下面是同一个示例的`伪代码`表示，以说明实际发生了什么

```javascript
var primitive = 'september';
primitive.vowels = 3;
//创建新对象来设置属性
new String('september').vowels = 3;

primitive.vowels;
//另一个新对象被创建来获取属性
new String('september').vowels; //undefined
```

所以，正如你所看到的，它不仅无用，而且相当浪费。

## 总结

事实证明，赋予属性的能力几乎是对象相对于原始对象的唯一优势，但在实践中，即使是这种能力也值得怀疑。字符串、布尔值和数字都有明确的用途，把它们重新定义为国家所有者可能会让人感到困惑。此外，由于基础类型是不可变的，您不能通过调整对象包装器的属性来修改它们:

```javascript
var me = new String('Angus');
me.length = 2; //(严格模式下会报错)
me.length; //5 (not 2 - thanks @joseanpg)
me.valueOf();
('Angus');
```

然而，我确实认为，对原始类型的良好理解以及与之交互时隐藏在背后的内容，是深入了解该语言的重要一步。我希望这能有所帮助。
