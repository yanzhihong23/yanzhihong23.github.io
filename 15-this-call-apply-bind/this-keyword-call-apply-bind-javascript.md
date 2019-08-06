<!--
 * @Author: Hom Yan
 * @Date: 2019-08-05 10:49:52
 * @LastEditors: Hom Yan
 * @LastEditTime: 2019-08-06 14:09:32
 -->

# 理解 JavaScript 中的"this"关键字、call、apply 以及 bind

[Understanding the "this" keyword, call, apply, and bind in JavaScript](https://tylermcginnis.com/this-keyword-call-apply-bind-javascript/) by `Tyler McGinnis` on 2018-03-26

JavaScript 中最容易被误解的一个方面是`this`关键字。在这篇文章中，你将学习五个规则来确定`this`关键字引用的内容。隐式绑定，显式绑定，new 绑定，window 绑定和词法绑定。在涉及这些技术，你还可以学习一些 JavaScript 的其他混淆部件以及类似`.call`，`.apply`，`.bind`和`new`关键字。

在深入研究 JavaScript 中的`this`关键字的细节之前，我们先退一步，首先来看下`this`关键字存在的原因。`this`关键字允许你在不同的上下文环境中重用函数。换句话说，**`this`关键字允许你在调用函数或方法时决定哪个对象应该是焦点。** 之后我们讨论的一切都将建立在这个想法上。我们希望在不同的上下文或不同的对象中重用函数或方法。

我们要看的第一件事是如何判断`this`关键字引用的内容。当你试图回答这个问题时，你需要问自己的第一个也是最重要的问题是”这个函数在哪里被调用？”。判断`this`关键字引用的内容的**唯一**方法就是，通过查看调用使用了`this`关键字函数的位置。

用一个你已经很熟悉的例子来证明这一点，比如我们有一个`greet`函数，它接受一个`name`然后显示一条欢迎的`alert`信息。

```javascript
function greet(name) {
  alert(`Hello, my name is ${name}`);
}
```

如果我问你`greet`到底会发出什么警报，你的回答是什么呢？只给出函数定义，是不可能知道的。要知道`name`是什么，你必须看看`greet`的函数调用。

```javascript
greet('Tyler');
```

这与弄清楚`this`关键字引用的内容是完全一样的思路。你甚至可以把`this`关键字当做一个正常的参数一样 - 它的改变将基于函数的调用方式。

既然你知道确定`this`关键字引用的内容的第一步是查看函数的调用位置，接下来呢？为了帮助我们完成下一步，我们将制定 5 条规则或指南。

1. 隐式绑定
2. 显示绑定
3. new 绑定
4. 词法绑定
5. window 绑定

## 隐式绑定

请记住，这里的目标是能够查看使用`this`关键字定义的函数并辨别`this`所引用的内容。第一个也是最常见的这么做的规则称为`隐式绑定`。这大概可以辨别 80%`this`关键字引用的内容。

假设我们有一个看起来像这样的对象

```javascript
const user = {
  name: 'Tyler',
  age: 27,
  greet() {
    alert(`Hello, my name is ${this.name}`);
  }
};
```

现在，如果要`greet`在`user`对象上调用方法，则可以使用点表示法。

```javascript
user.greet();
```

这将我们带到隐式绑定规则的主要关键点。为了弄清楚`this`关键字引用的内容，首先，**在调用函数时查看点的左侧**。如果存在“点”，请查看该点的左侧以查找 `this` 关键字引用的对象。

在上面的例子中，`user`在“点的左边”，这意味着`this`关键字引用了`user`对象。因此，这就**好像**在`greet`方法内部，JavaScript 解释器把`this`更改为`user`。

```javascript
greet() {
  // alert(`Hello, my name is ${this.name}`)
  alert(`Hello, my name is ${user.name}`) // Tyler
}
```

让我们来看一个类似的，但稍微更高级的例子。现在，不是仅仅有一个`name`，`age`和`greet`属性，让我们也给我们的`user`对象一个`mother`属性，该属性也同样有一个`name`和一个`greet`属性。

```javascript
const user = {
  name: 'Tyler',
  age: 27,
  greet() {
    alert(`Hello, my name is ${this.name}`);
  },
  mother: {
    name: 'Stacey',
    greet() {
      alert(`Hello, my name is ${this.name}`);
    }
  }
};
```

那么问题来了，下面的每个调用会发出什么警报？

```javascript
user.greet();
user.mother.greet();
```

每当我们试图找出`this`关键字引用的内容时，我们需要查看调用并查看“点的左边”是什么。在第一次调用中，`user`位于点的左侧，表示`this`将引用`user`。在第二次调用中，`mother`位于点的左侧，表示`this`将引用`mother`。

```javascript
user.greet(); // Tyler
user.mother.greet(); // Stacey
```

如前所诉，大约 80%的情况下“点的左边”都会有一个对象。这就是为什么在确定`this`关键字引用的内容时应该采取的第一步是“看点的左边”。但是，如果没有点怎么办呢？这就将我们带到下一个规则 -

## 显示绑定

现在，如果我们的`greet`函数不是`user`对象的一个方法，它就是它自己的独立函数。

```javascript
function greet() {
  alert(`Hello, my name is ${this.name}`);
}

const user = {
  name: 'Tyler',
  age: 27
};
```

我们知道，为了辨别`this`关键字引用的内容，我们首先要查看函数的调用位置。那么，问题来了，我们该如何调用`greet`使`this`关键字引用的对象是`user`对象。我们不能像之前`user.greet()`那样做，因为`user`没有`greet`方法。在 JavaScript 中，每个函数都包含一个方法，允许你完成此操作，该方法为`call`。

> “call”是每个函数的都有的一个方法，它允许你调用函数，指定调用函数的上下文。

记住这一点，我们可以通过以下代码在`user`上下文中调用`greet` -

```javascript
greet.call(user);
```

同样，`call`是每个函数的属性，并且传递给它的第一个参数将是调用函数的上下文（或焦点对象）。换句话说，传递给调用的第一个参数将是该函数内部`this`关键字引用的内容。

这是规则 #2（显示绑定）的基础，因为我们明确地（使用`.call`）指定`this`关键字引用的内容。

现在我们稍微修改下`greet`函数。如果我们还想传入一些别的参数怎么办？与`name`一样，我们也想提醒他们所知道的语言。就像这样

```javascript
function greet(l1, l2, l3) {
  alert(`Hello, my name is ${this.name} and I know ${l1}, ${l2}, and ${l3}`);
}
```

现在，为了把参数传递给通过`.call`调用的函数，你可以在指定了第一个参数作为上下文之后，把参数一个个传递。

```javascript
function greet(l1, l2, l3) {
  alert(`Hello, my name is ${this.name} and I know ${l1}, ${l2}, and ${l3}`);
}

const user = {
  name: 'Tyler',
  age: 27
};

const languages = ['JavaScript', 'Ruby', 'Python'];

greet.call(user, languages[0], languages[1], languages[2]);
```

这是有效的，它显示了如何将参数传递给通过`.call`调用的函数。然后，你可能也意识到了，必须从`language`数组中逐个传递参数，这有点烦人。如果我们可以将整个数组作为第二个参数传入，然后 JavaScript 再传播给我们，那该多好。好消息是，这正是`.apply`干的事。`.apply`与`.call`是完全一样的，只不过参数不是逐个传入，你可以传入一个数组，它会将数组中的每个元素作为参数传播给你。

所以现在使用`.apply`，我们的代码可以改为下面这样，其他一切保持不变。

```javascript
const languages = ['JavaScript', 'Ruby', 'Python'];

// greet.call(user, languages[0], languages[1], languages[2])
greet.apply(user, languages);
```

目前为止，在“显示绑定”规则下，我们已经了解`.call`以及`.apply`都允许你在调用函数时，指定函数内部`this`关键字引用的内容。这条规则的最后一部分是`.bind`。`.bind`与`.call`是完全一样的，只是它不会立即调用该函数，而是返回一个你可以稍后调用的新函数。如果使用`.bind`，我们之前的代码看起来会是下面这样

```javascript
function greet(l1, l2, l3) {
  alert(`Hello, my name is ${this.name} and I know ${l1}, ${l2}, and ${l3}`);
}

const user = {
  name: 'Tyler',
  age: 27
};

const languages = ['JavaScript', 'Ruby', 'Python'];

const newFn = greet.bind(user, languages[0], languages[1], languages[2]);
newFn(); // alerts "Hello, my name is Tyler and I know JavaScript, Ruby, and Python"
```

## new 绑定

确定`this`关键字引用内容的第三条规则称为`new`绑定。如果你不熟悉 JavaScript 中的`new`关键字，每当你使用`new`关键字调用函数时，JavaScript 解释器都会为你创建一个全新的对象并称之为 `this`。因此，自然地，如果通过 `new` 调用函数，则 `this` 关键字引用的就是解释器创建的新对象。

```javascript
function User(name, age) {
  /*
   * 在引擎内部，JavaScript创建一个新对象称为`this`，它代表了查找失败下的User的原型。
   * 如果一个函数是通过new关键字调用，那么解释器所创建的新对象就是this关键字引用的内容。
   */
  this.name = name;
  this.age = age;
}

const me = new User('Tyler', 27);
```

## 词法绑定

此刻，我们处在第 4 条规则，你可能会感到有些不知所措。这是正常的，JavaScript 中的`this`关键字可能比它本该的样子要复杂。好消息是，下一个规则是最直观的。

你应该已经听过并使用过箭头函数。它们是 ES6 里新增的。它们允许你以更简洁的格式来编写函数。

```javascript
friends.map(friend => friend.name);
```

除了简洁之外，箭头函数在`this`关键字问题上更直观。与普通函数不同，箭头函数没有自己的`this`。相反，`this`是词法决定的。这是一个神奇的说法，`this`正是你所预期的，按照正常的变量查找规则。我们来继续前面使用的例子。现在，我们把`languages`和`greet`绑定在一起，而不是作为单独的对象。

```javascript
const user = {
  name: 'Tyler',
  age: 27,
  languages: ['JavaScript', 'Ruby', 'Python'],
  greet() {}
};
```

先前我们假设`languages`数组的长度总是 3。这样我们就可以使用硬编码的变量，如`l1`、`l2`和`l3`。现在我们来把`greet`变得更智能一些，并假设`languages`可以是任意长度的。为此，我们将使用`.reduce`来创建字符串。

```javascript
const user = {
  name: 'Tyler',
  age: 27,
  languages: ['JavaScript', 'Ruby', 'Python'],
  greet() {
    const hello = `Hello, my name is ${this.name} and I know`;

    const langs = this.languages.reduce(function(str, lang, i) {
      if (i === this.languages.length - 1) {
        return `${str} and ${lang}.`;
      }

      return `${str} ${lang},`;
    }, '');

    alert(hello + langs);
  }
};
```

多了很多代码，但是结果是一样的。当我们调用`user.greet()`，期望看到的是`Hello, my name is Tyler and I know JavaScript, Ruby, and Python.`。不幸的是，有一个错误。你能发现它吗？抓取上面的代码并在控制台中运行它。你会发现抛出了一个错误`Uncaught TypeError: Cannot read property 'length' of undefined`。很明显，只有第 9 行使用了`.length`，那么问题就在那里了。

```javascript
if (i === this.languages.length - 1) {
}
```

根据我们的错误，`this.languages`是 undefined。我们一步步来看下`this`关键字的引用，为什么不是它本该指向的`user`对象。首先，我们来看下函数是在哪里被调用的。等一下？函数在哪里被调用的？函数是被传递给`.reduce`，所以我们不知道。我们从来没有真正看到匿名函数的调用，因为 JavaScript 在`.reduce`的内部实现中完成的。那就是问题所在。我们需要指定传递给`.reduce`的匿名函数是在`user`的上下文中调用的。那样`this.languages`将引用`user.languages`。从上面学到的，可以使用`.bind`。

```javascript
const user = {
  name: 'Tyler',
  age: 27,
  languages: ['JavaScript', 'Ruby', 'Python'],
  greet() {
    const hello = `Hello, my name is ${this.name} and I know`;

    const langs = this.languages.reduce(
      function(str, lang, i) {
        if (i === this.languages.length - 1) {
          return `${str} and ${lang}.`;
        }

        return `${str} ${lang},`;
      }.bind(this),
      ''
    );

    alert(hello + langs);
  }
};
```

所以我们已经看到`.bind`是怎么解决这个问题，但是这与箭头函数有什么关系呢？先前我提到箭头函数“`this`是词法决定的。这是一个神奇的说法，`this`正是你所预期的，按照正常的变量查找规则。”。

在上面的代码中，凭你的自然直觉，匿名函数内的`this`应该引用的是什么？对我来说，应该是`user`。没有理由创建一个新的上下文就因为我需要传递一个函数给`.reduce`。凭借这种直觉，箭头函数的价值经常被忽视。如果我们重新编写上面的代码，除了使用匿名箭头函数而不是匿名函数声明之外什么都不变，一切就都“正常”了。

```javascript
const user = {
  name: 'Tyler',
  age: 27,
  languages: ['JavaScript', 'Ruby', 'Python'],
  greet() {
    const hello = `Hello, my name is ${this.name} and I know`;

    const langs = this.languages.reduce((str, lang, i) => {
      if (i === this.languages.length - 1) {
        return `${str} and ${lang}.`;
      }

      return `${str} ${lang},`;
    }, '');

    alert(hello + langs);
  }
};
```

再次强调下原因，因为在箭头函数中，`this`是词法决定的。箭头函数没有自身的`this`。相反，就像变量查找，JavaScript 解释器将查找最近的（父级）作用域来确定`this`的引用。

## window 绑定

最后是“万能”的情况 - window 绑定。假设我们有一下代码

```javascript
function sayAge() {
  console.log(`My age is ${this.age}`);
}

const user = {
  name: 'Tyler',
  age: 27
};
```

正如我们前面提到的，如果你想要在`user`的上下文中调用`sayAge`，你可以使用`.call`、`.apply`或者`.bind`。如果我们没有使用这些，而是正常的直接调用会发生什么呢？

```javascript
sayAge(); // My age is undefined
```

不出所料，你得到的是`My age is undefined`因为`this.age`未定义。这里事情变得有些奇怪。点的左右没有任何东西，我们也没使用`.call`、`.apply`、`.bind`或者`new`关键字，这里真正发生的是，JavaScript 默认`this`引用`window`对象。这意味着如果我们给`window`对象添加一个`age`属性
，然后再调用`sayAge`函数，`this.age`就不再是未定义的，而是`window`对象上的`age`属性。不相信我？运行这段代码

```javascript
window.age = 27;

function sayAge() {
  console.log(`My age is ${this.age}`);
}
```

非常粗糙，对吗？这就是为什么第五条规则是`window 绑定`。如果没有满足其他规则，则 JavaScript 将默认 `this` 关键字引用 `window` 对象。

> 从 ES5 开始，如果启用了“严格模式”，JavaScript 将做正确的事，`this`将保持为未定义，而不是默认为`window`对象。

```javascript
'use strict';

window.age = 27;

function sayAge() {
  console.log(`My age is ${this.age}`);
}

sayAge(); // TypeError: Cannot read property 'age' of undefined
```

## 总结

因此，将所有规则付诸实践，每当我在函数内部看到`this`关键字时，这些就是我为了弄清楚它所引用的内容而采取的步骤。

1. 查看调用函数的位置。
2. 点左边有对象吗？如果有，那就是“this”关键字引用的内容。如果没有，继续＃3。
3. 该函数是否使用“call”、“apply”或者“bind”调用的？如果是，它将明确说明“this”关键字引用的内容。如果没有，继续＃4。
4. 是否使用“new”关键字调用了该函数？如果是，“this”关键字引用的是由 JavaScript 解释器创建的新对象。如果没有，继续＃5。
5. “this”在箭头函数中？如果是，则可以在最近（父）作用域内以词法方式找到其引用。如果没有，继续＃6。
6. 处在“严格模式”中吗？如果是，则“this”关键字未定义。如果没有，继续＃7。
7. JavaScript 很奇怪。“this”引用了“window”对象。
