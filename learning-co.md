首先请原谅我的标题党(●—●)，tj 大神的 co 模块源码200多行，显然不是我等屌丝能随便几行代码就能重写的。只是当今大家都喜欢《7天学会xx语言》之类的速效仙丹，于是我也弄个类似的名字《7行代码学会co模块》来博眼球。

为了避免被拖出去弹小JJ，还是先放出所谓的 7 行代码给大家压压惊：
```
function co(gen) {
	var it = gen();
	var ret = it.next();
	ret.value.then(function(res) {
		it.next(res);
	});
}
```
## 万恶的回调
对前端工程师来说，异步回调是再熟悉不过了，浏览器中的各种交互逻辑都是通过事件回调实现的，前端逻辑越来越复杂，导致回调函数越来越多，同时 nodejs 的流行也让 javascript 在后端的复杂场景中得到应用，在 nodejs 代码中更是经常看到层层嵌套。

以下是一个典型的异步场景：先通过异步请求获取页面数据，然后根据页面数据请求用户信息，最后根据用户信息请求用户的产品列表。过多的回调函数嵌套，使得程序难以维护，发展成[万恶的回调](http://callbackhell.com/)。
```javascript
$.get('/api/data', function(data) {
	console.log(data);
	$.get('/api/user', function(user) {
		console.log(user);
		$.get('/api/products', function(products) {
			console.log(products)
		});
	});
});
```

## 异步流程控制
* 最原始异步流程的写法，就是类似上面例子里的回调函数嵌套法，用过的人都知道，那叫一个酸爽。

* 后来出现了 Promise ，它极大提高了代码的可维护性，消除了万恶的回调嵌套问题，并且现在已经成为 ES6 标准的一部分。
```
$.get('/api/data')
.then(function(data) {
	console.log(data);
	return $.get('/api/user');
})
.then(function(user) {
	console.log(user);
	return $.get('/api/products');
})
.then(function(products) {
	console.log(products);
});
```

* 之后在 nodejs 圈出现了 co 模块，它基于 ES6 的 generator 和 yield ，让我们能用同步的形式编写异步代码。
```
co(function *() {
	var data = yield $.get('/api/data');
	console.log(data);
	var user = yield $.get('/api/user');
	console.log(user);
	var products = yield $.get('/api/products');
	console.log(products);
});
```

* 以上的 Promise 和 generator 最初创造它的本意都不是为了解决异步流程控制。其中 Promise 是一种编程思想，用于“当xx数据准备完毕，then执行xx动作”这样的场景，不只是异步，同步代码也可以用 Promise。而 generator 在 ES6 中是迭代器生成器，被 TJ 创造性的拿来做异步流程控制了。真正的异步解决方案请大家期待 ES7 的 async 吧！本文以下主要介绍 co 模块。

## co 模块
上文已经简单介绍了co 模块是能让我们以同步的形式编写异步代码的 nodejs 模块，主要得益于 ES6 的 generator。nodejs >= 0.11 版本可以加 `--harmony` 参数来体验 ES6 的 generator 特性，iojs 则已经默认开启了 generator 的支持。

要了解 co ，就不得不先简单了解下 ES6 的 generator 和 iterator。

### Iterator
Iterator 迭代器是一个对象，知道如何从一个集合一次取出一项，而跟踪它的当前序列所在的位置，它提供了一个next()方法返回序列中的下一个项目。
```javascript
var lang = { name: 'JavaScript', birthYear: 1995 };
var it = Iterator(lang);
var pair = it.next(); 
console.log(pair); // ["name", "JavaScript"]
pair = it.next(); 
console.log(pair); // ["birthYear", 1995]
pair = it.next(); // A StopIteration exception is thrown
```
乍一看好像没什么奇特的，不就是一步步的取对象中的 key 和 value 吗，`for ... in`也能做到，但是把它跟 generator 结合起来就大有用途了。

### Generator
Generator 生成器允许你通过写一个可以保存自己状态的的简单函数来定义一个迭代算法。
Generator 是一种可以停止并在之后重新进入的函数。生成器的环境（绑定的变量）会在每次执行后被保存，下次进入时可继续使用。generator 字面上是“生成器”的意思，在 ES6 里是迭代器生成器，用于生成一个迭代器对象。
```
function *gen() {
	yield 'hello';
	yield 'world';
	return true;
}
```
以上代码定义了一个简单的 generator，看起来就像一个普通的函数，区别是`function`关键字后面有个`*`号，函数体内可以使用`yield`语句进行流程控制。
```javascript
var iter = gen();
var a = iter.next();
console.log(a); // {value:'hello', done:false}
var b = iter.next();
console.log(b); // {value:'world', done:false}
var c = iter.next();
console.log(c); // {value:true, done:true}
```
当执行`gen()`的时候，并不执行 generator 函数体，而是返回一个迭代器。迭代器具有`next()`方法，每次调用 next() 方法，函数就执行到`yield`语句的地方。next() 方法返回一个对象，其中value属性表示 yield 关键词后面表达式的值，done 属性表示是否遍历结束。generator 生成器通过`next`和`yield`的配合实现流程控制，上面的代码执行了三次 next() ，generator 函数体才执行完毕。

### co 模块思路
从上面的例子可以看出，generator 函数体可以停在 yield 语句处，直到下一次执行 next()。co 模块的思路就是利用 generator 的这个特性，将异步操作跟在 yield 后面，当异步操作完成并返回结果后，再触发下一次 next() 。当然，跟在 yield 后面的异步操作需要遵循一定的规范 thunks 和 promises。

>#### yieldables
>The yieldable objects currently supported are:
>- promises
>- thunks (functions)
>- array (parallel execution)
>- objects (parallel execution)
>- generators (delegation)
>- generator functions (delegation)

## 7行代码
再看看文章开头的7行代码：

```javascript
function co(gen) {
	var it = gen();
	var ret = it.next();
	ret.value.then(function(res) {
		it.next(res);
	});
}
```
首先生成一个迭代器，然后执行一遍 next()，得到的 value 是一个 Promise 对象，Promise.then() 里面再执行 next()。当然这只是一个原理性的演示，很多错误处理和循环调用 next() 的逻辑都没有写出来。


下面做个简单对比：
传统方式，`sayhello`是一个异步函数，执行`helloworld`会先输出`"world"`再输出`"hello"`。
```
function sayhello() {
	return Promise.resolve('hello').then(function(hello) {
		console.log(hello);
	});
}
function helloworld() {
	sayhello();
	console.log('world');
}
helloworld();
```
输出
```
> "world"
> "hello"
```
co 的方式，会先输出`"hello"`再输出`"world"`。
```javascript
function co(gen) {
	var it = gen();
	var ret = it.next();
	ret.value.then(function(res) {
		it.next(res);
	});
}
function sayhello() {
	return Promise.resolve('hello').then(function(hello) {
		console.log(hello);
	});
}
co(function *helloworld() {
	yield sayhello();
	console.log('world');
});
```
输出
```javascript
> "hello"
> "world"
```

## 消除回调金字塔
假设`sayhello`/`sayworld`/`saybye`是三个异步函数，用真正的 co 模块就可以这么写：
```
var co = require('co');
co(function *() {
	yield sayhello();
	yield sayworld();
	yield saybye();
});
```
输出
```javascript
> "hello"
> "world"
> "bye"
```

## 参考
《es7-async》 https://github.com/jaydson/es7-async  
《Generator 函数的含义与用法》 http://www.ruanyifeng.com/blog/2015/04/generator.html  
《Iterator》 https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Iterator
