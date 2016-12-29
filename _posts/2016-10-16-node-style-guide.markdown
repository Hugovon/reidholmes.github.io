---
published: true
layout:     post
title:      "Node.js 风格指南"
date:       2016-10-16 20:20:35
categories:
---



# Node.js 风格指南

<p>本文译自：<a class="reference external" href="https://github.com/felixge/node-style-guide">https://github.com/felixge/node-style-guid</a></p>


### 格式化
* [使用两格缩进](#2-spaces-for-indentation)
* [换行符](#newlines)
* [没有多余的空格](#no-trailing-whitespace)
* [使用分号](#use-semicolons)
* [每行80个字符](#80-characters-per-line)
* [使用单引号](#use-single-quotes)
* [在同一行上打开括号](#opening-braces-go-on-the-same-line)
* [每一个变量用var语句声明](#declare-one-variable-per-var-statement)

### 名称约定
* [使用小写字母命名变量、属性和函数名](#use-lowercamelcase-for-variables-properties-and-function-names)
* [使用大写字母命名类名称](#use-uppercamelcase-for-class-names)
* [使用大写字母命名常数](#use-uppercase-for-constants)

### 变量
* [对象/ 数组对象](#object--array-creation)

### 条件句
* [使用= = =操作符](#use-the--operator)
* [使用多行三元运算符](#use-multi-line-ternary-operator)
* [使用描述性的条件](#use-descriptive-conditions)

### 函数
* [写小函数](#write-small-functions)
* [过早的从函数返回](#return-early-from-functions)
* [命名好最后要关闭](#name-your-closures)
* [没有嵌套闭包](#no-nested-closures)
* [方法链/作用域链](#method-chaining)

### 评论/注释
* [使用斜线注释](#use-slashes-for-comments)

### 其他
* [Object.freeze, Object.preventExtensions, Object.seal, with, eval](#objectfreeze-objectpreventextensions-objectseal-with-eval)
* [Requires在上面](#requires-at-top)
* [不要扩展内置的原型](#do-not-extend-built-in-prototypes)

## 格式化

你可能想使用 [editorconfig.org](http://editorconfig.org/)在你的编辑器执行“格式设置。使用 [Node.js Style Guide .editorconfig file](.editorconfig) 去缩进和设置空格， 换行符等行为规则设置会自动设置日期。


### 使用两格缩进


使用2个空格缩进代码并发誓永不混合标签和空间——否则正在等待你的是一个大麻烦。

### 换行符

你最后的字符使用Unix风格的换行符 (`\n`)，Windows风格的换行符是(`\r\n`)。

### 没有多余的空格

就像你刷牙后吃饭一样，在你的JS文件之前你必须清理任何尾随的空白空格。否则因为你粗心的忽视，腐烂的气味最终会赶走贡献者或同事。

### 使用分号

根据 [scientific research][hnsemicolons],这种分号的用法，作为我们社区的核心价值。考虑到 [the opposition][]很多人反对, 但说到滥用纠错机制是一个传统主义者
廉价的句法乐趣。

[the opposition]: http://blog.izs.me/post/2353458699/an-open-letter-to-javascript-leaders-regarding
[hnsemicolons]: http://news.ycombinator.com/item?id=1547647

### 每行80个字符

限制你每行80个字符。是的，你的屏幕上的屏幕已经变得够大了，但你的大脑没有。使用额外的空间分割屏幕，你的编辑支持这个，对吗？

### 使用单引号

Use single quotes, unless you are writing JSON.

*Right:*

```js
var foo = 'bar';
```

*Wrong:*

```js
var foo = "bar";
```

### 在同一行上打开括号

你的打开括号和语句在同一行上。

*Right:*

```js
if (true) {
  console.log('winning');
}
```

*Wrong:*

```js
if (true)
{
  console.log('losing');
}
```

同时，让之前和之后的空白的使用条件语句。

### 每一个变量用var语句声明

声明一个变量的每一个变量的语句，它使它更容易重新排序
线.然而，当你忽略 [Crockford][crockfordconvention] 的时候只要把声明放在哪里都无所谓。


*Right:*

```js
var keys   = ['foo', 'bar'];
var values = [23, 42];

var object = {};
while (keys.length) {
  var key = keys.pop();
  object[key] = values.pop();
}
```

*Wrong:*

```js
var keys = ['foo', 'bar'],
    values = [23, 42],
    object = {},
    key;

while (keys.length) {
  key = keys.pop();
  object[key] = values.pop();
}
```

[crockfordconvention]: http://javascript.crockford.com/code.html

### 名称约定

### 使用小写字母命名变量、属性和函数名


变量，属性和函数的名称应该使用 `lowerCamelCase`.  他们
也应该被这样描述。单字符变量和罕见的变量，一般应避免缩写。

*Right:*

```js
var adminUser = db.query('SELECT * FROM users ...');
```

*Wrong:*

```js
var admin_user = db.query('SELECT * FROM users ...');
```

### 使用大写字母命名类名称

Class名称应大写 `UpperCamelCase`.

*Right:*

```js
function BankAccount() {
}
```

*Wrong:*

```js
function bank_Account() {
}
```

## 使用大写字母命名类名称

常量应该被声明为变量或静态类属性，全部使用大写字母。

*Right:*

```js
var SECOND = 1 * 1000;

function File() {
}
File.FULL_PERMISSIONS = 0777;
```

*Wrong:*

```js
const SECOND = 1 * 1000;

function File() {
}
File.fullPermissions = 0777;
```

[const]: https://developer.mozilla.org/en/JavaScript/Reference/Statements/const

## 变量

### 对象/ 数组对象

使用引号引用包裹起来数据。


*Right:*

```js
var a = ['hello', 'world'];
var b = {
  good: 'code',
  'is generally': 'pretty',
};
```

*Wrong:*

```js
var a = [
  'hello', 'world'
];
var b = {"good": 'code'
        , is generally: 'pretty'
        };
```

## 条件句

### 使用= = =操作符

编程不是记忆 [stupid rules][comparisonoperators]. 使用三重相等，因为它会像预期一样工作。

*Right:*

```js
var a = 0;
if (a !== '') {
  console.log('winning');
}

```

*Wrong:*

```js
var a = 0;
if (a == '') {
  console.log('losing');
}
```

[comparisonoperators]: https://developer.mozilla.org/en/JavaScript/Reference/Operators/Comparison_Operators

### 使用多行三元运算符

三元运算符不应该在一行上使用。它应该分裂成多行。

*Right:*

```js
var foo = (a === b)
  ? 1
  : 2;
```

*Wrong:*

```js
var foo = (a === b) ? 1 : 2;
```

### 使用描述性的条件

任何条件应该分配给一个描述性的命名变量或函数：

*Right:*

```js
var isValidPassword = password.length >= 4 && /^(?=.*\d).{4,}$/.test(password);

if (isValidPassword) {
  console.log('winning');
}
```

*Wrong:*

```js
if (password.length >= 4 && /^(?=.*\d).{4,}$/.test(password)) {
  console.log('losing');
}
```

## 函数

### 写小函数

保持你的函数是短的. 一个好的函数应该适合每一个人阅读。

### 过早返回函数

为了避免深嵌套的语句，尽可能总是返回一个函数的值。

*Right:*

```js
function isPercentage(val) {
  if (val < 0) {
    return false;
  }

  if (val > 100) {
    return false;
  }

  return true;
}
```

*Wrong:*

```js
function isPercentage(val) {
  if (val >= 0) {
    if (val < 100) {
      return true;
    } else {
      return false;
    }
  } else {
    return false;
  }
}
```



```js
function isPercentage(val) {
  var isInRange = (val >= 0 && val <= 100);
  return isInRange;
}
```

### 命名好最后要关闭

命名好之后随时关闭，否则将产生内存泄漏、堆和处理器配置文件。

*Right:*

```js
req.on('end', function onEnd() {
  console.log('winning');
});
```

*Wrong:*

```js
req.on('end', function() {
  console.log('losing');
});
```

### 没有嵌套闭包

使用闭包，但不要嵌套它们。否则你的代码将成为一个混乱。

*Right:*

```js
setTimeout(function() {
  client.connect(afterConnect);
}, 1000);

function afterConnect() {
  console.log('winning');
}
```

*Wrong:*

```js
setTimeout(function() {
  client.connect(function() {
    console.log('losing');
  });
}, 1000);
```


### 方法链

如果要使用链方法，应使用每行的一个方法。你也应该缩进这些方法。

*Right:*

```js
User
  .findOne({ name: 'foo' })
  .populate('bar')
  .exec(function(err, user) {
    return true;
  });
````

*Wrong:*

```js
User
.findOne({ name: 'foo' })
.populate('bar')
.exec(function(err, user) {
  return true;
});

User.findOne({ name: 'foo' })
  .populate('bar')
  .exec(function(err, user) {
    return true;
  });

User.findOne({ name: 'foo' }).populate('bar')
.exec(function(err, user) {
  return true;
});

User.findOne({ name: 'foo' }).populate('bar')
  .exec(function(err, user) {
    return true;
  });
````

## 评论/注释

### 使用斜线注释

使用双斜线定义评论或者注释. 试着给你的代码写注释和评论，因为这样可以更清楚这段代码。

*Right:*

```js
// 'ID_SOMETHING=VALUE' -> ['ID_SOMETHING=VALUE', 'SOMETHING', 'VALUE']
var matches = item.match(/ID_([^\n]+)=([^\n]+)/));

// This function has a nasty side effect where a failure to increment a
// redis counter used for statistics will cause an exception. This needs
// to be fixed in a later iteration.
function loadUser(id, cb) {
  // ...
}

var isSessionValid = (session.expires < Date.now());
if (isSessionValid) {
  // ...
}
```

*Wrong:*

```js
// Execute a regex
var matches = item.match(/ID_([^\n]+)=([^\n]+)/);

// Usage: loadUser(5, function() { ... })
function loadUser(id, cb) {
  // ...
}

// Check if the session is valid
var isSessionValid = (session.expires < Date.now());
// If the session is valid
if (isSessionValid) {
  // ...
}
```

## 乱七八糟其他的

### Object.freeze, Object.防止扩展, Object.封装, with, eval
生命苦短，少折腾。（好像这个意思吧）

### Requires在顶部

总是把需要在文件的顶部，以清楚地说明一个文件的依赖关系。


### 不要扩展内置的原型

Do not extend the prototype of native JavaScript objects. Your future self will
be forever grateful.
不随意扩展JavaScript对象的原型，这样的话，你会非常感激自己的。（可能是因为把javascript内置原型扩展之后，会影响后续使用吧？再或者把自己搞迷糊？）

*Right:*

```js
var a = [];
if (!a.length) {
  console.log('winning');
}
```

*Wrong:*

```js
Array.prototype.empty = function() {
  return !this.length;
}

var a = [];
if (a.empty()) {
  console.log('losing');
}
```


### 后记


- **好饿，吃饭去**
