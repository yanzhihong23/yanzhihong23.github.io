# 构造函数和操作符 "new"

[Constructor, operator "new"](https://javascript.info/constructor-new) by `javascript.info` on 2019-09-17

常规的 `{...}` 语法允许创建一个对象。但是我们经常需要创建许多类似的对象，例如多个用户或菜单项等等。

这可以使用构造函数和 `"new"` 操作符。

## 构造函数

构造函数在技术上是常规函数。不过有两个约定：

1. 他们首先用大写字母命名。
2. 它们只能用 "new" 操作符来执行。

例如：

```javascript
function User(name) {
  this.name = name;
  this.isAdmin = false;
}

let user = new User('Jack');

alert(user.name); // Jack
alert(user.isAdmin); // false
```

当一个函数作为 new User(...)执行时，它执行以下步骤：

一个新的空对象被创建并分配给 this。

1. 函数体执行。通常它会修改 this，为其添加新的属性。
2. 返回 this 的值。
3. 换句话说，new User(...) 做类似的事情：

```javascript
function User(name) {
  // this = {};（隐式创建）

  // 添加属性到 this
  this.name = name;
  this.isAdmin = false;

  // return this;（隐式返回）
}
```

所以`let user = new User("Jack")`的结果与下面是一样的：

```javascript
let user = {
  name: 'Jack',
  isAdmin: false
};
```

现在，如果我们想创建其他用户，我们可以调用 `new User("Ann")`，`new User("Alice")` 等等。比每次使用字面量创建要短得多，而且易于阅读。

这是构造函数的主要目的 — 实现可重用的对象创建代码

让我们再次注意 — 从技术上讲，任何函数都可以用作构造函数。即：任何函数都可以运行 `new`，它会执行上面的算法。 “首字母大写”是一个共同的约定，以明确表示一个函数将被使用 `new` 运行。

> #### new function() { ... }
>
> 如果我们有许多关于创建单个复杂对象的代码行，我们可以将它们封装在构造函数中，如下所示：
>
> ```javascript
> let user = new (function() {
>   this.name = 'John';
>   this.isAdmin = false;
>
>   // ...用户创建的其他代码
>   // 也许是复杂的逻辑和陈述
>   // 局部变量等
> })();
> ```
>
> 构造函数不能被再次调用，因为它不保存在任何地方，只是被创建和调用。所以这个技巧的目的是封装构建单个对象的代码，而不是将来重用。

## 构造函数模式测试：new.target

> #### 进阶
>
> 这一节的语法很少使用，除非你想了解所有的，不然可以跳过。

在一个函数内部，我们可以使用 `new.target` 属性来检查它被调用时，是否使用了 `new`。

常规调用为空，如果通过 `new` 调用，则等于函数：

```javascript
function User() {
  alert(new.target);
}

// 不带 new：
User(); // undefined

// 带 new：
new User(); // function User { ... }
```

这可以用来检查是否通过`new`调用，“构造函数模式”或者“常规模式”。

我们还可以使`new`和常规调用的效果一致，像这样：

```javascript
unction User(name) {
  if (!new.target) { // 如果你没有运行 new
    return new User(name); // ...会为你添加 new
  }

  this.name = name;
}

let john = User("John"); // 重新调用 new User
alert(john.name); // John
```

## 构造函数 Return

通常，构造函数没有 `return` 语句。他们的任务是将所有必要的东西写入 this，并自动转换为结果。

但是，如果有 `return` 语句，那么规则很简单：

1. 如果 `return` 一个对象，则返回它，而不是 `this`。
2. 如果 `return` 一个原始类型，则忽略。

换句话说，带有对象的 `return` 返回该对象，其他所有情况都返回 `this`。

例如，这里 `return` 通过返回一个对象覆盖 `this`：

```javascript
function BigUser() {
  this.name = 'John';

  return { name: 'Godzilla' }; // <-- returns 一个 object
}

alert(new BigUser().name); // 哇哦，得到了对象，name 属性值为 Godzilla ^^
```

这里有一个 `return` 空的例子（或者我们可以在它之后放置一个原始类型，也没关系）：

```javascript
function SmallUser() {
  this.name = 'John';

  return; // returns this
}

alert(new SmallUser().name); // John
```

通常构造函数没有 `return` 语句。这里我们主要为了完整性而提及返回对象的特殊行为。

> #### 省略括号
>
> 顺便说一下，如果没有参数我们可以省略 `new` 之后的括号：
>
> ```javascript
> let user = new User(); // <-- 没有括号
> // 一样的效果
> let user = new User();
> ```
>
> 这里省略括号不被认为是一种“好风格”，但是规范允许使用该语法。

## 构造函数中的方法

使用构造函数来创建对象会带来很大的灵活性。构造函数可以通过参数来定义所要构造的对象，以及什么该放进来。

当然，我们不仅可以将属性添加到 `this` 中，而且还可以添加方法。

例如：下面 `new User(name)` 用给定的 `name` 和方法 `sayHi` 创建一个对象：

```javascript
function User(name) {
  this.name = name;

  this.sayHi = function() {
    alert('My name is: ' + this.name);
  };
}

let john = new User('John');

john.sayHi(); // My name is: John

/*
john = {
   name: "John",
   sayHi: function() { ... }
}
*/
```

## 总结

- 构造函数，或简而言之，构造器，都是常规函数，但他们有个共同的约定，以首字母大写来命名。
- 构造函数应该只能通过 `new` 来调用。这样的调用意味着在开始时创建空的 `this`，并在最后返回填充的对象。

我们可以使用构造函数来创建多个类似的对象。

JavaScript 为许多内置的对象提供了构造函数：比如日期 `Date`，设置集合 `Set` 以及其他我们计划学习的内容。
