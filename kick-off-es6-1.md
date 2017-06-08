# ES6 走马观花

本文为 ES6 系列的第一篇。旨在给新同学一些指引，带大家走近 ES6。
简要介绍：

- 什么是 ES6
- 它有哪些“明星”特性
- 它可以运行在哪些环境

## What's ES6


### ES6 or ECMAScript 2015 ?
[ECMAScript 6](http://www.ecma-international.org/ecma-262/6.0/)（以下简称ES6）是JavaScript语言的下一代标准，已经在2015年6月正式发布了。

ECMA-262 标准 第6版。计划以后每年发布一次标准，使用年份作为标准的版本。因为当前版本是在2015年发布的，所以又称ECMAScript 2015。ECMA 的全称是 European Computer Manufacturers Association(欧洲计算机制造商协会)。ECMA-262 被 ISO 国际标准化组织采纳为 [ISO/IEC 16262:2011](http://www.iso.org/iso/home/store/catalogue_ics/catalogue_detail_ics.htm?csnumber=55755)

日常讨论的 JavaScript 通常还包括 DOM(文档对象模型)、BOM(浏览器对象模型)，而 ES6 不包含这些。 

### ES6 现状
- 主流框架全面转向 ES6 
	- Angular 2
	- ReactJs
	- koa
- 兼容性 [对比表格](http://kangax.github.io/compat-table/es6/)

## Well-known Features

本节介绍一些广为人知的 ES6 “明星”特性，也就是讨论 ES6 时经常提及的一些新特性。当然 ES6 并不仅限于这些，还包括很多其他有用的特性，会在本系列的其他文章中介绍。

### let and const

#### let 命令
原来的 javascript 中没有块级作用域，只有函数级作用域。ES6 中新增了 let 命令，使用 let 命令代替 var 命令声明变量，具有块级作用域。

- 函数级作用域

```
function test() {
  var hello = 'world';
  console.log(hello);
}
test(); // 'world'
console.log(hello); // Error: hello is not defined
```

- 块级作用域

`var` 命令

```
if(true) {
  var hello = 'world';
  console.log(hello); // 'world'
}
console.log(hello); // 'world'
```
  
`let` 命令

```
if(true) {
  let hello = 'world';
  console.log(hello); // 'world'
}
console.log(hello); // Error: hello is not defined
```

#### const 命令
使用 const 命令声明常量

```
const STATUS_NOT_FOUND = 404;
```
常量的值为只读，不能修改

```
STATUS_NOT_FOUND = 200;
// SyntaxError: "STATUS_NOT_FOUND" is read-only
```

### Template String
- 传统的字符串

```
var name = 'es6';

var sayhello = 'hello, \
my name is ' + name + '.';

console.log(sayhello); 
```
输出：

```
hello, my name is es6.
```

- ES6 模板字符串

```
var name = 'es6';

var sayhello = `hello,
my name is ${name}.`;

console.log(sayhello);
```
输出：

```
hello,
my name is es6.
```
> 空格和换行都会被保留

### Arrow Function
允许使用 `=>` 定义函数，箭头函数自动绑定当前上下文 this。

```
x => x+1
```
等同于匿名函数

```
function (x) {
  return x + 1;
}
```

- 多个参数：

```
(a,b) => a+b
```
等同于

```
function (a, b) {
  return a + b;
}
```

- 多行函数体：

```
(a,b) => {
	console.log(a + b);
	return a + b;
}
```
等同于

```
function (a, b) {
	console.log(a + b);
	return a + b;
}
```

### Promise
原生的 Promise 实现，不再需要 `bluebird` 或 `Q+`。

### Generator
Generator 生成器允许你通过写一个可以保存自己状态的的简单函数来定义一个迭代算法。和 Generator 一起出现的通常还有 yield。

Generator 是一种可以停止并在之后重新进入的函数。生成器的环境（绑定的变量）会在每次执行后被保存，下次进入时可继续使用。generator 字面上是“生成器”的意思，在 ES6 里是迭代器生成器，用于生成一个迭代器对象。

```
function *gen() {
    yield 'hello';
    yield 'world';
    return true;
}
```
以上代码定义了一个简单的 generator，看起来就像一个普通的函数，区别是function关键字后面有个*号，函数体内可以使用yield语句进行流程控制。

```
var iter = gen();
var a = iter.next();
console.log(a); // {value:'hello', done:false}
var b = iter.next();
console.log(b); // {value:'world', done:false}
var c = iter.next();
console.log(c); // {value:true, done:true}
```
当执行gen()的时候，并不执行 generator 函数体，而是返回一个迭代器 Iterator。迭代器具有 next() 方法，每次调用 next() 方法，函数就执行到yield语句的地方。next() 方法返回一个对象，其中value属性表示 yield 关键词后面表达式的值，done 属性表示是否遍历结束。generator 生成器通过 next 和 yield 的配合实现流程控制，上面的代码执行了三次 next() ，generator 函数体才执行完毕。

### Class
在 JavaScript 中引入 OO 面向对象。实际上是语法糖，只是看上去更面向对象而已。也有观点极力反对 Class，认为 Class 隐藏了 JavaScript 本身原型链的语言特性，使其更难理解。

一半以上库是按 OO/class 方式写的，除了jQuery之外，几乎每个“严肃”的JS基础库都有一个Class实现，工具、IDE 更容易识别，JavaScript 引擎性能优化。---- johnhax

- ES5

```
function Point(x,y){
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function () {
  return '(' + this.x + ', ' + this.y + ')';
}
```

- ES6

```
class Point {

  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '('+this.x+', '+this.y+')';
  }

}
```

### Module

许多JS框架都会实现一套自己的 module/loader 机制。反复造轮子这也就罢了，更大的问题在于，这些轮子互相都是不兼容的。结果就造成了JS社区的分化和内耗，阻碍了JS库和组件在较细粒度上的竞争和发展，JS框架和库的切换成了强迫开发者做出非此即彼的选择。缺乏语言级别的 module，其恶果就是既没有足够的标准库，也很难像其他语言一样通过丛林法则发展出事实标准库。 ---- johnhax

社区主流解决方案有 CommonJS 和 AMD，分别用于服务器端和浏览器端，浏览器端还有 seajs 遵循的 CMD。

CommonJS

```
exports.firstName = 'mei';
exports.lastName = 'qingguang';
exports.year = 1988;

// or

module.exports = {
	firstName: 'mei',
	lastName: 'qingguang',
	year: 1988
}

// or

module.exports = function() {
	// do something
}
```

AMD

```
define(['./a', './b'], function(a, b) { // 依赖必须一开始就写好
	a.doSomething()
	// 此处略去 100 行
	b.doSomething()
	// ...
	
	exports.action = function() {};
}) 
```

CMD

```
define(function(require, exports, module) {
	var a = require('./a')
	a.doSomething()
	// 此处略去 100 行
	var b = require('./b') // 依赖可以就近书写
	b.doSomething()
	// ... 
	
	exports.action = function() {};
})
```

ES6 Module

- export 命令 和 import 命令

```
export var firstName = 'mei';
export var lastName = 'qingguang';
export var year = 1988;
```

```
import {firstName, lastName, year} from './profile'

console.log(firstName, lastName, year)
```

- 模块整体输出

```
var firstName = 'mei';
var lastName = 'qingguang';
var year = 1988;

export {firstName, lastName, year};
```

```
import * as Profile from './profile'

console.log(Profile.firstName, Profile.lastName, Profile.year)
```
- export default 整体输出

```
export default function() {
	console.log('My name is mei qingguang');
};
```

```
import sayMyName from './profile' 

console.log(sayMyName())
```

## Node.js 运行环境

可以在 Node.js 和 io.js 中使用部分 ES6 特性。Node.js 和 io.js 都是使用 V8 引擎作为 JavaScript 运行环境，io.js 集成了更高版本的 V8 引擎，因此可以比 Node.js 支持更多的 ES6 特性。

在 Node.js 中，需要使用 `--harmony` 参数开启 ES6 特性，包括所有已完成、待完成和修订中的特性。为了避免用到那些废弃的特性，可以通过加类似 `--harmony_generators` 参数开启特定的特性。

而在 io.js 中，所有已完成的稳定 ES6 特性都已经默认开启，不需要加任何运行时参数。而待完成和修订中的特性也可以通过特定的参数开启。

io.js 默认开启了以下 ES6 特性：

- block scoping
  - let
  - const
  - function-in-blocks
- Classes
- Collections
	- Map
	- WeakMap
	- Set
	- WeakSet
- Generators
- Binary and Octal literals
- Object literal extensions
- Promises
- New String methods
- Symbols
- Template strings


## 编译器

有两个著名的编译器，能将 ES6 代码编译成 ES5 代码，本节只介绍 Babel。

- Babel
- Traceur

### Babel
Babel 从根本上讲是一个平台，这是它与 compile-to-JS 语言 CoffeeScript 和 TypeScript 最大的不同。Babel 的插件系统允许开发者自定义代码转换器并插入到编译过程。Babel 还能提升 JavaScript 的执行速度，在编译时进行性能优化。

#### babel 命令
编译单个文件

```
babel script.js --out-file script-compiled.js
```
监听文件变化

```
babel script.js --watch --out-file script-compiled.js
```
编译整个文件夹

```
babel src --out-dir lib
```

使用 source map，方便调试

```
babel script.js --out-file script-compiled.js --source-maps
```

#### babel-node 命令
使用 `babel-node` 命令代替 `node` 命令，实时编译并执行 ES6 代码。不要在生产环境使用 `babel-node` 命令，它非常耗内存，并且会拖慢应用的性能。

```
node app.js
```
```
babel-node app.js
```
#### require hook
使用 require 钩子，可以让你的应用 require 模块时自动编译 ES6 代码。例如：

run.js

```
require('babel/register')
require('./app.js')
```

将 `run.js` 作为整个应用的入口，就可以在 `app.js` 和其他业务代码中编写 ES6 代码，当代码被 `require` 进来时，自动编译成 ES5 代码。 



## Learn a bit of ES6 daily
[ES6 Katas](http://es6katas.org/)