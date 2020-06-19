# 在 JavaScript 进行数据变更还是不变更

[To mutate, or not to mutate, in JavaScript](https://slemgrim.com/mutate-or-not-to-mutate/) by `Markus Raudaschl` on 2016-04-16

有很多人在讲不可变性是所有一切的最佳解决方案。也有人说，对于企业系统来说，不可变性不算什么，只是推广函数式编程的一大部分。如果我们在 Google 上搜索，就会发现大量描述什么是不可变性的观点和文章。本文，我们将重点介绍在 Javascript 中的正确使用场景。

## 基础

和其他很多语言一样，在 JavaScript 中原始类型是不可变的。这意味着，任何时候我们改变一个原始类型，都会创建一个新的实例。

```javascript
// Strings
let str = 'Hello world!';
let res = str.slice(1, 5); // ello

// Numbers
let number = 10;
number += 15;
```

这带来一堆好处：

- ~~并发~~
- 实例共享
- 无副作用
- 无耦合
- 更好的可读性
- 更少的内存消耗
- 易缓存
- 易测试

像 ELM 和 Haskell 这样的纯函数语言，任何值都是不可变的。JavaScript 不是其中之一。JavaScript 中的对象默认是可变的。一旦我们使用它们，就丢掉了上面的这些好处。但是，还是有很多方式，可以像不可变的原始类型一样去使用。

## 手动的方式

尽管 JavaScript 不支持不可变的对象，但我们仍然可以通过避免大多数突变的方式编写代码。

### 不要在函数中改变对象

编写的函数应该返回变更的副本，而不是直接改变给定对象的属性。

```javascript
//bad
function save(object) {
  object.saved = true;
  return object;
}

//better
function save(object) {
  let newObject = object.clone();
  newObject.saved = true;
  return newObject;
}
```

### 不要在构造之后改变对象

对象是引用，如果我们避免改变它们的属性，就可以避免不清楚的状态。这样最终的代码也更好理解并且更好测试。

```javascript
let request = {
  method: 'GET',
  uri: 'http://slemgrim.com',
};

// don't do this
request.method = 'POST';
```

### 始终避免 setters

这对于上面两点来说是冗余的。如果使用 setters，我们就会改变对象。因此，最好不要使用 setters。

看起来需要很多繁琐的工作来避免变更。JavaScript 没有内置对不可变对象的支持，确实让我们有些困难。是时候获取一些帮助了。

## Immutable.js

Facebook 的[Immutable.js](https://facebook.github.io/immutable-js/) 是一个很小的库，可以帮助我们保持状态不可变。还有些其他类似的库（[Mori](https://github.com/swannodette/mori), [seamless-immutable](https://github.com/rtfeldman/seamless-immutable)），但本文我们就专注在 Immutable.js。

我不会深入细节，因为[文档](https://facebook.github.io/immutable-js/)真的很简单，但总结起来：

> Immutable.js 提供很多持久化的不可变的数据结构，包括：List, Stack, Map, OrderedMap, Set, OrderedSet 和 Record

或以代码的形式：

```javascript
import Map from 'immutable';
var request = Map({ method: 'GET', uri: 'http://slemgrim.com' });
var post = request.set('method', 'POST');
request.get('method'); // GET
post.get('method'); // POST
```

尽管丢失了对属性的直接访问，但我们可以聚焦在我们的目标上而不是变更的斗争中。

## ESLint Immutable

还有一个 [eslint-plugin-immutable](https://github.com/jhusain/eslint-plugin-immutable) 的插件可以禁用 JavaScript 中的所有数据突变。说明文档中提到了很多 react/redux，但是没有这俩你也可以放心使用。

**这个插件添加了 3 调规则：**

- no-let: 使用 const 而不是 let
- no-this: 禁用 this 和 ES6 类
- no-mutation: 禁止为成员表达式的结果赋值

感谢 [@thisisgodon](https://k94n.com/)的提醒

## 什么时候使用？

### 并发

这是不可变性有意义的第一个原因。为了线程安全，你不需要去锁定不会变更的东西。JS 中没有真实的并发，而是一个[Event Loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)。所以可以忽略了。

### 用来避免副作用

改变对象的属性就是突变。根据定义，突变是一种副作用。我们每个人都学到过，应该尽量避免副作用。如果一个函数没有副作用，我们就不需要深入了解它的具体实现。我们的代码会变的更合理，更可预测，更可测试。

### 函数式编程中

有时候，您必须停下来思考一下要处理的是值还是引用。 不可变的对象始终是值。 这是函数式编程的核心原则之一。 如果将值传递给函数，我可以保证它永远不变。

## 什么时候不要使用

### 频繁改变属性

如果你的对象改变的比较频繁，那么最好不要每次都创建一个新的实例。对于每秒发生多次的游戏和模拟来说尤其如此。 并不是说你不能在游戏中使用不可变性，但是与可变对象相比，更容易陷入性能问题。

### 庞大的数据结构

如果你有一个非常庞大的数据树作为不可变容器存储，那么建议你检查内存消耗。 当然，垃圾回收会杀死所有旧对象，但是复制大对象仍然有代价。

## 总结

一般来说，坚持不可变类型是一种非常合理的语言设计选择，即使当所有东西都是不可变的时候，也会出现建模不好的问题。在大多数情况下，不可变类型的优点大大超过了缺点，而且它们比其他选择更好，总体上可以带来更好的代码和更快的可执行程序。
