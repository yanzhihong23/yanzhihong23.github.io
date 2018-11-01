# JavaScript 里的值和引用

[原文](https://codeburst.io/explaining-value-vs-reference-in-javascript-647a975e12a0)

JavaScript 有 5 种数据类型是通过`值`传递：`Boolean`, `null`, `undefined`, `String` 以及 `Number`。我们称这些为原始类型。

JavaScript 有 3 种数据类型是通过`引用`传递：`Array`, `Function` 以及 `Object`。这些都是技术上的对象，因此我们将它们统称为`对象`。

## 原始类型

如果原始类型被赋给一个变量，我们可以将该变量视为包含原始值。

```javascript
var x = 10;
var y = 'abc';
var z = null;
```

`x`包含`10`, `y`包含`'abc'`。为了巩固这个想法，我们用一张图表来模拟变量在内存中的存储。   
<br>

| 变量 | 值    |
| ---- | ----- |
| x    | 10    |
| y    | 'abc' |
| z    | null  |

当我们通过`=`把这些变量分配给其他变量时，我们将值`复制`到新的变量。它们按值复制。

```javascript
var x = 10;
var y = 'abc';
var a = x;
var b = y;
console.log(x, y, a, b); // -> 10, 'abc', 10, 'abc'
```
`a`和`x`现在都包含`10`。`b`和`y`现在都包含`'abc'`。它们是分开的，因为值本身被复制了。  
<br>

| 变量 | 值    |
| ---- | ----- |
| x    | 10    |
| y    | 'abc' |
| a    | 10    |
| b    | 'abc' |

改变一个不会改变其他的，可以认为变量之间没有任何关系。

```javascript
var x = 10;
var y = 'abc';
var a = x;
var b = y;
a = 5;
b = 'def';
console.log(x, y, a, b); // -> 10, 'abc', 5, 'def'
```

## 对象
这将会感到困惑，但容忍我，读完它。一旦你通过它，它会看起来很简单。

分配了非原始值的变量将被赋予对该值的引用。 该引用指向对象在内存中的位置。 变量实际上不包含该值。

对象是在你计算机内存中的某个位置被创建。当我们写下`arr = []`时，我们就在内存中创建了一个数组。变量`arr`所接收到的是那个数组的地址和位置。

让我们假装地址是一个按值传递的新数据类型，就像数字或字符串一样。 地址指向内存中通过引用传递的值的位置。 就像字符串用引号（''或“”）表示一样，地址将用箭头括号<>表示。

当我们分配和使用引用类型变量时，我们编写和看到的是：

```javascript
var arr = [];
arr.push(1);
```
内存中上面第1行和第2行的表示是：

第1行  

| 变量 | 值     |     | 地址 | 对象 |
| ---- | ------ | --- | ---- | ---- |
| arr  | <#001> |     | #001 | []   |

第2行  

| 变量 | 值  |  | 地址 | 对象 |
| ---- | --- |-- |---- | ---- |
| arr  | <#001> | |#001 | [1]  |

注意变量`arr`所包含的值、地址都是不变的。改变的是内存里的数组。当我们对`arr`进行操作时，比如推送一个值，JavaScript引擎会转到`arr`在内存中的位置并使用存储在那的信息。

## 按引用分配
当使用`=`将引用类型值（一个对象）复制到另一个变量时，实际上复制的是该值的地址，就像它是一个原始类型一样。 对象是通过引用而不是按值复制。

```javascript
var reference = [1];
var refCopy = reference;
```
上面的代码在内存中看起来就像这样  

| 变量      | 值     |     | 地址 | 对象 |
| --------- | ------ | --- | ---- | ---- |
| reference | <#001> |     | #001 | [1]  |
| refCopy   | <#001> |

每个变量现在都包含对同一数组的引用。这意味着如果我们改变`reference`，`refCopy`将看到这些变更：  

```javascript
reference.push(2);
console.log(reference, refCopy); // -> [1, 2], [1, 2]
```

| 变量      | 值     |     | 地址 | 对象   |
| --------- | ------ | --- | ---- | ------ |
| reference | <#001> |     | #001 | [1, 2] |
| refCopy   | <#001> |

我们已经将`2`推入内存中的数组。 当我们使用`reference`和`refCopy`时，我们指向同一个数组。

## 重新分配一个引用

重新分配一个引用将替换旧引用。

```javascript
var obj = { first: 'reference' };
```
在内存中  

| 变量 | 值     |     | 地址 | 对象                   |
| ---- | ------ | --- | ---- | ---------------------- |
| obj  | <#234> |     | #234 | { first: 'reference' } |

当我们再加一行  

```javascript
var obj = { first: 'reference' };
obj = { second: 'ref2' }
```
`obj`存储的地址改变了。第一个对象仍然存在于内存中，下一个对象也是如此:    
 <br> 

| 变量 | 值     |     | 地址 | 对象                   |
| ---- | ------ | --- | ---- | ---------------------- |
| obj  | <#678> |     | #234 | { first: 'reference' } |
|      |        |     | #678 | { second: 'ref2' }     |

当没有对剩余对象的引用时，就像上面的地址 `#234`，JavaScript引擎就会执行垃圾回收。这意味着开发者失去了所有对该对象的引用并且再也无法使用该对象，这样引擎就可以安全地从内存中删除它。在这种情况下，对象`{first：'reference'}`不再可访问，并且可供引擎用于垃圾收集。

## == 和 ===

当在引用类型变量上使用相等运算符==和===时，它们会检查引用。 如果变量包含对同一项的引用，则比较结果为true。

```javascript
var arrRef = [’Hi!’];
var arrRef2 = arrRef;
console.log(arrRef === arrRef2); // -> true
```

如果它们是不同的对象，即使它们包含相同的属性，比较结果也会是`false`。

```javascript
var arr1 = ['Hi!'];
var arr2 = ['Hi!'];
console.log(arr1 === arr2); // -> false
```

如果我们有两个不同的对象并且想要查看它们的属性是否相同，那么最简单的方法是将它们都转换为字符串然后比较字符串。 当相等运算符比较基础类型时，它们只是检查值是否相同。

```javascript
var arr1str = JSON.stringify(arr1);
var arr2str = JSON.stringify(arr2);
console.log(arr1str === arr2str); // true
```
另一种选择是递归循环遍历对象并确保每个属性都相同。

## 通过函数传递参数

当我们将原始值传递给函数时，该函数将值复制到其参数中。 它实际上与使用`=`相同。

```javascript
var hundred = 100;
var two = 2;
function multiply(x, y) {
    // PAUSE
    return x * y;
}
var twoHundred = multiply(hundred, two);
```

在上面的例子中，我们给`hundred`赋值`100`。当我们把它传递给`multiply`，变量`x`获得该值`100`。值的复制就像使用`=`操作符。并且`hundred`的值没有受到影响。下面是PAUSE注释行中内存的快照。  

| 变量     | 值     |     | 地址 | 对象                 |
| -------- | ------ | --- | ---- | -------------------- |
| hundred  | 100    |     | #333 | function(x, y) {...} |
| two      | 2      |     |
| multiply | <#333> |     |
| x        | 100    |     |
| y        | 2      |     |

## 纯函数

我们将不影响外部作用域任何内容的函数称为纯函数。只要函数只将原始值作为参数并且不在其周围作用域使用任何变量，它就会自动变纯，因为它不会影响外部作用域的任何内容。函数返回后，内部创建的所有变量都会被垃圾收集。

但是，接受对象的函数可以改变其周围作用域的状态。 如果函数接受数组引用并改变它指向的数组，可能通过推送它，引用该数组的周围作用域中的变量会看到该更改。 函数返回后，它所做的更改将在外部作用域中保留。 这可能导致难以追踪的不期望的副作用。

因此，许多本机数组函数（包括`Array.map`和`Array.filter`）都被编写为纯函数。 它们接受数组引用并在内部，它们复制数组并使用副本而不是原始参数。 这使得原始版本不受影响，外部范围不受影响，返回对全新数组的引用。

让我们来看一个纯函数与非纯函数的例子。

```javascript
function changeAgeImpure(person) {
    person.age = 25;
    return person;
}
var alex = {
    name: 'Alex',
    age: 30
};
var changedAlex = changeAgeImpure(alex);
console.log(alex); // -> { name: 'Alex', age: 25 }
console.log(changedAlex); // -> { name: 'Alex', age: 25 }
```
这个非纯函数接受一个对象，并把对象的属性`age`改为25。因为直接作用于给定的引用，它直接改变了对象`alex`。当它返回`person`对象时，它返回的正是传进来的同一个对象。`alex`和`alexChanged`包含同一个引用。返回`person`变量并将引用存储在新变量中是多余的。

我们再来看一下纯函数。

```javascript
function changeAgePure(person) {
    var newPersonObj = JSON.parse(JSON.stringify(person));
    newPersonObj.age = 25;
    return newPersonObj;
}
var alex = {
    name: 'Alex',
    age: 30
};
var alexChanged = changeAgePure(alex);
console.log(alex); // -> { name: 'Alex', age: 30 }
console.log(alexChanged); // -> { name: 'Alex', age: 25 }
```

在这个函数里，我们通过`JSON.stringify`把传进来的对象转换为字符串，然后再用`JSON.parse`转换回对象。通过这样的转换与存储结果到新的变量，我们创建了一个新的对象。还有其它的方法来完成同样的结果，比如循环遍历对象再赋值给新对象，但这种方式是最简单的。新对象具有与原始对象相同的属性，但它是内存中明显独立的对象。

当我们改变新对象的`age`属性，原始对象不受影响。这个函数是纯的，它不会影响外部作用域的任何对象，甚至传进来的对象。新的对象需要返回并且存储在新的变量上，不然一旦函数返回了，新的对象就会被垃圾回收，因为对象不在作用域内了。

## 自我测试
值与引用是一个经常在编码面试中测试的概念。自己试下下面会打印什么。

```javascript
function changeAgeAndReference(person) {
    person.age = 25;
    person = {
        name: 'John',
        age: 50
    };
    
    return person;
}
var personObj1 = {
    name: 'Alex',
    age: 30
};
var personObj2 = changeAgeAndReference(personObj1);
console.log(personObj1); // -> ?
console.log(personObj2); // -> ?
```

这个函数先改变了传进来原始对象的`age`属性，然后重新赋值给一个全新的对象，随即返回改对象。下面是这两个对象打印出来的结果。

```javascript
console.log(personObj1); // -> { name: 'Alex', age: 25 }
console.log(personObj2); // -> { name: 'John', age: 50 }
```
请记住，通过函数参数赋值与使用`=`赋值基本相同。 函数中的变量`person`包含对`personObj1`对象的引用，因此最初它直接作用于该对象。 一旦我们将`person`重新分配给新对象，它就会停止影响原始对象。

上述代码块的等效代码：  
```javascript
var personObj1 = {
    name: 'Alex',
    age: 30
};
var person = personObj1;
person.age = 25;
person = {
  name: 'john',
  age: 50
};
var personObj2 = person;
console.log(personObj1); // -> { name: 'Alex', age: 25 }
console.log(personObj2); // -> { name: 'John', age: '50' }
```
唯一的区别是我们使用了函数，函数一旦返回，`person`对象就再也不在作用域内了。