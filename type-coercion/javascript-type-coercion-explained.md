# JavaScript 类型强制转换

[JavaScript type coercion explained](https://medium.freecodecamp.org/js-type-coercion-explained-27ba3d9a2839) by `Alexey Samoshkin` on 2018-01-18

![weird](https://cdn-images-1.medium.com/max/2000/1*7awmfn1lq2McPz8ggapndw.png)

<center>JavaScript里的怪异行为</center>

**类型强制转换** 是将值从一种类型转换为另一种类型的过程，例如字符串转数字、对象转布尔等。任何类型，无论是原始类型还是对象，都是类型转换的有效主体。回忆下，原始类型有：`number`、`string`、`boolean`、`null`、`undefined` 以及 `Symbol`(ES6 新增)。

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

<!-- prettier-ignore -->
```javascript
Boolean(2)          // 显式
if (2) { ... }      // 隐式，由于逻辑上下文
!!2                 // 隐式，由于逻辑运算符
2 || 'hello'        // 隐式，由于逻辑运算符
```

**说明:** 逻辑运算符比如`||`和`&&`在内部做布尔转换，但实际上返回的是原始的运算对象，就算它们不是布尔值。

```javascript
// 返回数字 123 而不是 true
// 'hello' 和 123 还是在内部转换为布尔值来计算表达式
let x = 'hello' && 123; // x === 123
```

布尔转换只有 2 种可能得结果: `true` 或 `false`，所以记下这个假值列表还简单一点。

<!-- prettier-ignore -->
```javascript
Boolean('')           // false
Boolean(0)            // false     
Boolean(-0)           // false
Boolean(NaN)          // false
Boolean(null)         // false
Boolean(undefined)    // false
Boolean(false)        // false
```

任何不在上面列表中的值都转换为`true`，包括对象、函数、`Array`、`Date`、自定义类型等。Symbols 都是真值，空对象和数组也都是真值：

<!-- prettier-ignore -->
```javascript
Boolean({})             // true
Boolean([])             // true
Boolean(Symbol())       // true
!!Symbol()              // true
Boolean(function() {})  // true
```

### 数值转换

显式转换直接应用`Number()`函数，就像`Boolean()`和`String()`一样。

隐式转换比较棘手一点，因为它的触发场景很多:

- 比较运算符(`>` `<` `<=` `>=`)
- 位运算符(`|` `&` `^` `~`)
- 数学运算符(`-` `+` `*` `/` `%`)。注意，当有运算元是字符串的时候，二元运算符`+`不会触发数值转换
- 一元运算符`+`
- 宽松相等运算符`==`(包括 `!=`)

<!-- prettier-ignore -->
```javascript
Number('123')   // 显式
+'123'          // 隐式
123 != '456'    // 隐式
4 > '5'         // 隐式
5/null          // 隐式
true | 0        // 隐式
```

下面是原始值转换为数值：

<!-- prettier-ignore -->
```javascript
Number(null)                   // 0
Number(undefined)              // NaN
Number(true)                   // 1
Number(false)                  // 0
Number(" 12 ")                 // 12
Number("-12.34")               // -12.34
Number("\n")                   // 0
Number(" 12s ")                // NaN
Number(123)                    // 123
```

当把一个字符串转换为数值时，引擎首先会去掉首尾的空格、`\n`、`\t`字符，如果修剪的字符串不是一个有效的数值，那么就返回`NaN`。如果字符串是空的，就返回`0`。

`null`和`undefined`的处理是不同的：`null`变为`0`，而`undefined`变为`NaN`。

Symbol 不能被转换为数值，不管是显式还是隐式。而且`TypeError`会被抛出，不会像`undefiend`那样静默地转换为`NaN`。可以再[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol#Symbol_%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2)上查看更多关于 Symbol 的转换规则。

<!-- prettier-ignore -->
```javascript
Number(Symbol('my symbol'))    // TypeError is thrown
+Symbol('123')                 // TypeError is thrown
```

有两条**特殊的规则**要记住：

1. 当给`null`或`undefined`运用`==`时，数值转换并不会发生。`null`只与`null`或`undefined`相等，而不等于其他任意值。

<!-- prettier-ignore -->
```javascript
null == 0               // false, null is not converted to 0
null == null            // true
undefined == undefined  // true
null == undefined       // true
```

2. `NaN`不等于任意值，连它自身都不相等。

```javascript
if (value !== value) {
  console.log("we're dealing with NaN here");
}
```

## 对象的类型强制转换

目前为止，我们已经看过了原始值的类型强制转换，这些并不是那么令人兴奋。

当处理对象的时候，比如引擎遇到像`[1] + [2,3]`这样的表达式，首先需要把对象转换为原始值，再把原始值转换为最终的类型。最终还是只有三种类型转换：数值(numeric)、字符串(string)和布尔(boolean)。

最简单的情况是布尔转换：任何非原始值都强制转换为`true`，不管对象或数组是否为空。

对象转换为原始值是通过内部的`[[ToPrimitive]]`方法，该方法负责数值和字符转换。

下面是`[[ToPrimitive]]`方法的实现参考：

```javascript
function ToPrimitive(input, preferredType) {
  switch (preferredType) {
    case Number:
      return toNumber(input);
      break;
    case String:
      return toString(input);
      break;
    default:
      return toNumber(input);
  }

  function isPrimitive(value) {
    return value !== Object(value);
  }

  function toString() {
    if (isPrimitive(input.toString())) return input.toString();
    if (isPrimitive(input.valueOf())) return input.valueOf();
    throw new TypeError();
  }

  function toNumber() {
    if (isPrimitive(input.valueOf())) return input.valueOf();
    if (isPrimitive(input.toString())) return input.toString();
    throw new TypeError();
  }
}
```

`[[ToPrimitive]]`的参数是一个输入值和更倾向的转换类型：`Number`或`String`，`preferredType`是可选的。

数值和字符转换都是使用输入对象的两个方法：`valueof`和`toString`。这两个方法都是声明在`Object.prototype`上，所以对任何传递进来的类型都是可用的，比如`Date`、`Array`等。

通常情况下，算法是下面这样的：

1. 如果输入值已经是一个原始值，什么也不做，直接返回
2. 调用`input.toString()`，如果结果是原始值，返回
3. 调用`input.valueOf()`，如果结果是原始值，返回
4. 如果`input.toString()`和`input.valueOf()`的结果都不是原始值，抛出`TypeError`

数值转换先调用`valueOf`(3)，`toString`(2)作为备用。  
字符转换刚好相反：`toString`(2)先调用，`valueOf`(3)作为备用。

大多数内建的类型都没有`valueOf`方法，或有`valueOf`但返回的是`this`对象本身，所以会被忽略掉，因为它不是一个原始值。这就是为什么数值和字符转换结果可能是一样的，最后都是调用`toString()`。

不通的运算符会触发带有辅助`preferredType`参数的数值或字符转换。但有 2 个例外：宽松相等运算符`==`和二元`+`运算符的默认转换模式(`preferredType`没有指定，或默认为`default`)。这种情况下，大多数内建类型把数值转换做为默认，除了`Date`是以字符转换。

下面是`Date`转换行为的一个例子：

```javascript
et d = new Date();

// 获取字符串表示
let str = d.toString();  // 'Wed Jan 17 2018 16:15:42'

// 获取数值表示
let num = d.valueOf();   // 1516198542525

// 与字符串表示做比较
// true，由于 d 被转化为一样的字符串
console.log(d == str);   // true

// 与数值表示做标胶
// false，由于 d 并不是通过 valueOf() 转化为数值
console.log(d == num);   // false

// 结果是 'Wed Jan 17 2018 16:15:42Wed Jan 17 2018 16:15:42'
// '+' 与 '==' 一样触发默认转换模式
console.log(d + d);

// 结果是 0, 由于 '-' 运算符显式触发数值转换，而不是默认模式
console.log(d - d);
```

你可以覆盖默认的`toString()`和`valueOf()`方法，来看清对象到原始类型的转换逻辑。

<!--prettier-ignore -->
```javascript
var obj = {
  prop: 101,
  toString(){
    return 'Prop: ' + this.prop;
  },
  valueOf() {
    return this.prop;
  }
};

console.log(String(obj));  // 'Prop: 101'
console.log(obj + '')      // '101'
console.log(+obj);         //  101
console.log(obj > 100);    //  true
```

注意`obj + ''`是怎么以字符串的形式返回`'101'`。`+`运算符触发默认转换模式，前面提到过的，`Object`以数值转换作为默认模式，所以优先使用`valueOf()`方法而不是`toString()`。

## ES6 Symbol.toPrimitive 方法

在 ES5 里你可以覆盖`toString`和`valueOf`方法来获取对象转原生类型的转换逻辑。

在 ES6 里你可以走的更远，通过在一个对象上实现`[Symbol.toPrimitive]`方法来完全替换内部的`[[ToPrimitive]]`路由。

<!--prettier-ignore-->
```javascript
class Disk {
  constructor(capacity){
    this.capacity = capacity;
  }

  [Symbol.toPrimitive](hint){
    switch (hint) {
      case 'string':
        return 'Capacity: ' + this.capacity + ' bytes';

      case 'number':
        // 转换为 KiB
        return this.capacity / 1024;

      default:
        // 假设数值转换是默认
        return this.capacity / 1024;
    }
  }
}

// 1MiB disk
let disk = new Disk(1024 * 1024);

console.log(String(disk))  // Capacity: 1048576 bytes
console.log(disk + '')     // '1024'
console.log(+disk);        // 1024
console.log(disk > 1000);  // true
```

## 例子

用理论武装起来，现在我们再回过头来看我们的例子：

<!--prettier-ignore-->
```javascript
true + false             // 1
12 / "6"                 // 2
"number" + 15 + 3        // 'number153'
15 + 3 + "number"        // '18number'
[1] > null               // true
"foo" + + "bar"          // 'fooNaN'
'true' == true           // false
false == 'false'         // false
null == ''               // false
!!"false" == !!"true"    // true
['x'] == 'x'             // true 
[] + null + 1            // 'null1'
[1,2,3] == [1,2,3]       // false
{}+[]+{}+[1]             // '0[object Object]1'
!+[]+[]+![]              // 'truefalse'
new Date(0) - 0          // 0
new Date(0) + 0          // 'Thu Jan 01 1970 02:00:00(EET)0'
```

下面你可以找到每个表达式的解释

二元`+`运算符为`true`和`false`触发数值转换

```javascript
true + false
==> 1 + 0
==> 1
```

算术除法运算符`/`为字符串`6`触发数值转换

```javascript
12 / '6'
==> 12 / 6
==> 2
```

运算符`+`有从左到右的结合性，所以表达式`"number" + 15`先执行。由于有一个运算元是字符串，`+`运算符为数字`15`触发字符转换。第二步表达式`"number15" + 3`的计算就类似了。

```javascript
“number” + 15 + 3
==> "number15" + 3
==> "number153"
```

表达式 `15 + 3`先计算。由于 2 个运算元都是数值，所以不需要任何强制转换。第二步计算表达式`18 + 'number'`，由于有一个运算元是字符串，触发了字符转换。

```javascript
15 + 3 + "number"
==> 18 + "number"
==> "18number"
```

比较运算符`>`为`[1]`和`null`触发数值转换。

```javascript
[1] > null
==> '1' > 0
==> 1 > 0
==> true
```

一元`+`运算符拥有比二元`+`运算符更高的优先级，所以先计算`+'bar'`表达式。一元加号为`'bar'`触发了字符转换。由于字符串无法表示一个有效的数值，所以结果是`NaN`。第二步，计算表达式`'foo' + NaN`。

```javascript
"foo" + + "bar"
==> "foo" + (+"bar")
==> "foo" + NaN
==> "fooNaN"
```

`==`运算符触发数值转换，字符串`'true'`被转换为`NaN`，布尔值`true`被转换为`1`。

```javascript
'true' == true
==> NaN == 1
==> false

false == 'false'
==> 0 == NaN
==> false
```

`==`通常触发数值转换，但`null`不是这样的。`null`只与`null`或`undefined`相等，不等于任何其他。

```javascript
null == ''
==> false
```

`!!`运算符把`'true'`和`'false'`这两个字符串都转换为布尔值`true`，由于它们都不是空的字符串。然后，`==`只是检查两个布尔值`true`的相等性，而不需要做任何转换。

```javascript
!!"false" == !!"true"
==> true == true
==> true
```

`==`运算符为数组触发数值转换，数组的`valueOf()`方法返回数值本身，由于不是原始值，所以被忽略了。数组的`toString()`方法把`['x']`转换为`'x'`字符串。

```javascript
['x'] == 'x'
==> 'x' == 'x'
==>  true
```

`+`运算符为`[]`触发数值转换，数组的`valueOf()`方法返回数值本身，由于不是原始值，所以被忽略了。数组的`toString()`方法返回空字符串。第二步计算表达式`'' + null + 1`。

```javascript
[] + null + 1
==>  '' + null + 1
==>  'null' + 1
==> 'null1'
```

逻辑运算符`||`和`&&` 把运算元都转为布尔值，但是返回原始运算元。`0`为假值，而`'0'`为真值因为它不是一个空字符串。`{}`空对象也是真值。

```javascript
0 || "0" && {}
==>  (0 || "0") && {}
==> (false || true) && true  // internally
==> "0" && {}
==> true && true             // internally
==> {}
```

因为 2 个运算元的类型是一样的，所以不需要强制转换。由于`==`检查对象标识(不是对象相等性)，而这两个数组是 2 个不同的实例，所以结果为`false`。

```javascript
[1,2,3] == [1,2,3]
==>  false
```

所有的运算元都不是原始值，所以`+`从最左边开始触发数值转换。`Object`和`Array`的`valueOf`方法都返回自身，所以被忽略了。`toString()`作为备选被使用。这里的一个技巧是，第一个`{}`不是当做一个对象字面量，而是作为一个块声明，所以被忽略了。接着计算下一个`+[]`表达式，先是通过`toString()`方法转换为空字符串，然后再转换为`0`。

```javascript
{}+[]+{}+[1]
==> +[]+{}+[1]
==> 0 + {} + [1]
==> 0 + '[object Object]' + [1]
==> '0[object Object]' + [1]
==> '0[object Object]' + '1'
==> '0[object Object]1'
```

这个最好通过运算符的优先级一步步来解释。

```javascript
!+[]+[]+![]
==> (!+[]) + [] + (![])
==> !0 + [] + false
==> true + [] + false
==> true + '' + false
==> 'truefalse'
```

`-`运算符为`Date`触发数值转换。`Date.valueOf()`返回子 Unix 新纪元开始的毫秒数。

```javascript
new Date(0) - 0
==> 0 - 0
==> 0
```

`+`运算符触发默认转换。日期默认为字符串转换，所以使用的是`toString()`，而不是`valueOf()`。

```javascript
new Date(0) + 0
==> 'Thu Jan 01 1970 02:00:00 GMT+0200 (EET)' + 0
==> 'Thu Jan 01 1970 02:00:00 GMT+0200 (EET)0'
```

## 资源

我非常想推荐[Nicholas C.Zakas](https://medium.com/@slicknet)写的优秀的书[UnderStanding ES6](https://leanpub.com/understandinges6)。这是一个很好的学习 ES6 的资源，不是很高级，也不太深入底层。

这里还有一本只关于 ES5 的好书，[Axel Rauschmayer](https://medium.com/@rauschma)写的[SpeakingJS](http://speakingjs.com/)

JavaScript 比较表  — [ https://dorey.github.io/JavaScript-Equality-Table/](https://dorey.github.io/JavaScript-Equality-Table/)

wtfjs - 一个很小的关于我们又爱又恨的语言的代码博客 -- [https://wtfjs.com/](https://wtfjs.com/)
