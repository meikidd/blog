# ES8 走马观花

距离上一篇[《ES6 走马观花》][kick-off-es6]已经快两年时间了，上个月底 [ES8][ecma-262-8.0] 正式发布，再写一篇姊妹篇，介绍 ES8。

## 什么是 ES8 

[ES8][ecma-262-8.0] 是 ECMA-262 标准第 8 版的简称，从 ES6 开始每年发布一个版本，以年份作为名称，因此又称 ECMAScript 2017，简称 ES2017。

### 每年一个版本

两个版本之间间隔时间太久（从 ES5 到 ES6 经历了 6 年）会有以下两个问题：
- 有很多早已讨论完毕的特性需要等到标准的大版本发布才能进入标准
- 有一些特性本身比较复杂，需要较长的时间去讨论。但如果推迟到下一个版本，又必须等待很长的时间才能发布

从 ES6 开始新版本发布会更频繁，每年发布一个版本，把这一年内讨论完毕的特性纳入标准。

### TC39 流程

TC39（Technical Committee 39）是一个推动JavaScript发展的委员会。它的成员由各个主流浏览器厂商的代表构成。会议的每一项决议必须大部分人赞同，并且没有人强烈反对才可以通过。因为，对成员来说，同意就意味着有责任去实现它。每个 ECMAScript 特性都会经历 stage 0 到 stage 4 的每一个阶段。在 [TC39 proposals][tc39-proposals] 这个 github 仓库可以看到每个特性的进度。

#### Stage 0: strawman

一种推进ECMAScript发展的自由形式，任何TC39成员，或者注册为TC39贡献者的会员，都可以提交。

#### Stage 1: proposal

该阶段产生一个正式的提案。

1. 确定一个带头人来负责该提案，带头人或者联合带头人必须是TC39的成员。
1. 描述清楚要解决的问题，解决方案中必须包含例子，API以及关于相关的语义和算法。
1. 潜在问题也应该指出来，例如与其他特性的关系，实现它所面临的挑战。
1. polyfill和demo也是必要的。


#### Stage 2: draft

草案是规范的第一个版本，与最终标准中包含的特性不会有太大差别。草案之后，原则上只接受增量修改。

1. 草案中包含新增特性语法和语义的，尽可能的完善的形式说明，允许包含一些待办事项或者占位符。
1. 必须包含2个实验性的具体实现，其中一个可以是用转译器实现的，例如Babel。

#### Stage 3: candidate

候选阶段，获得具体实现和用户的反馈。此后，只有在实现和使用过程中出现了重大问题才会修改。
1. 规范文档必须是完整的，评审人和ECMAScript的编辑要在规范上签字。
1. 至少要有两个符合规范的具体实现。

#### Stage 4: finished

已经准备就绪，该特性会出现在年度发布的规范之中。
1. 通过Test 262的验收测试。
1. 有2个通过测试的实现，以获取使用过程中的重要实践经验。
1. ECMAScript的编辑必须规范上的签字。

## 新特性

### 1. String padding

新增了 [String.prototype.padStart][es8-padStart] 和 [String.prototype.padEnd][es8-padEnd] 两个函数，用于在字符串开头或结尾添加填充字符串。函数的声明如下：

```
String.prototype.padStart( maxLength [ , fillString ] )
String.prototype.padEnd( maxLength [ , fillString ] )
```
其中第一个参数是目标长度；第二个参数是填充字符串，默认是空格。示例：

```
'es8'.padStart(2);          // 'es8'
'es8'.padStart(5);          // '  es8'
'es8'.padStart(6, 'woof');  // 'wooes8'
'es8'.padStart(14, 'wow');  // 'wowwowwowwoes8'
'es8'.padStart(7, '0');     // '0000es8'

'es8'.padEnd(2);          // 'es8'
'es8'.padEnd(5);          // 'es8  '
'es8'.padEnd(6, 'woof');  // 'es8woo'
'es8'.padEnd(14, 'wow');  // 'es8wowwowwowwo'
'es8'.padEnd(7, '6');     // 'es86666'
```

#### 典型的应用场景

使用 `padStart` 进行时间格式化。

```
'8:00'.padStart(5, '0');  // '08:00'
'18:00'.padStart(5, '0');  // '18:00'
'12'.padStart(10, 'YYYY-MM-DD') // "YYYY-MM-12"
'09-12'.padStart(10, 'YYYY-MM-DD') // "YYYY-09-12"
```

使用 `padStart` 给命令行输出信息对齐。

```
Commands:

  run       Start a front service
  start     Start a background service
  stop      Stop current background service
  restart   Restart current background service
  help      Display help information
```

> 感谢 [left-pad 事件][left-pad]  为此特性的贡献 

### 2. Object.values & Object.entries

这两个静态方法是对原有的 `Object.keys()` 方法的补充。

```
const obj = { 
  x: 'xxx', 
  y: 1 
};
Object.keys(obj); // ['x', 'y']
```

#### 2.1 Object.values 

静态方法 `Object.values()` 获取对象的所有可遍历属性的值，返回一个数组。示例如下：

```
// 基本用法
const obj = { 
  x: 'xxx', 
  y: 1 
};
Object.values(obj); // ['xxx', 1]

// 数组可以看做键为下标的对象
// ['e', 's', '8'] -> { 0: 'e', 1: 's', 2: '8' }
const obj = ['e', 's', '8'];
Object.values(obj); // ['e', 's', '8']

// 字符串可以看做键为下标的对象
// 'es8' -> { 0: 'e', 1: 's', 2: '8' }
Object.values('es8'); // ['e', 's', '8']

// 如果是纯 number 型的键值，则返回值顺序根据键值从小到大排列
const obj = { 10: 'xxx', 1: 'yyy', 3: 'zzz' };
Object.values(obj); // ['yyy', 'zzz', 'xxx']
```

#### 2.2 Object.entries

静态方法 `Object.entries` 获取对象的虽有可遍历属性的键值对，以 [key, value] 数组的形式返回，顺序和 `Object.values()` 一致。  
![举个栗子][example]

```
// 基本用法
const obj = { 
  x: 'xxx', 
  y: 1 
};
Object.entries(obj); // [['x', 'xxx'], ['y', 1]]

// 数组可以看做键为下标的对象
// ['e', 's', '8'] -> { 0: 'e', 1: 's', 2: '8' }
const obj = ['e', 's', '8'];
Object.entries(obj); // [['0', 'e'], ['1', 's'], ['2', '8']]

// 字符串可以看做键为下标的对象
// 'es8' -> { 0: 'e', 1: 's', 2: '8' }
Object.entries('es8'); // [['0', 'e'], ['1', 's'], ['2', '8']]

// 如果是纯 number 型的键值，则返回值顺序根据键值从小到大排列
const obj = { 10: 'xxx', 1: 'yyy', 3: 'zzz' };
Object.entries(obj); // [['1', 'yyy'], ['3', 'zzz'], ['10': 'xxx']]
```

#### 知识点展开：`for...in` 和 `for...of` 循环
上述的 `Object.keys()`, `Object.values()`, `Object.entries()` 通常用来遍历一个对象，除了这三个方法外，常用的还有 `for...in` 和 `for...of` + `Object.keys()` 循环  
![举个栗子][example]

使用 `for...in` 遍历

```
const obj = { 
  x: 'xxx', 
  y: 1 
};
for (let key in obj) {
  console.log(key);
}
```

使用 `for...of` + `Object.keys()` 遍历

```
const obj = { 
  x: 'xxx', 
  y: 1 
};
for (let key of Object.keys(obj)) {
  console.log(key);
}
```

上述例子中两种遍历方式等价。但在更复杂的情况下，这两种方式的结果会不一样。`for...in` 循环会遍历对象的可枚举属性，包括原型链上继承的属性，而 `Object.keys()` 不会遍历继承的属性。下面是一个继承的例子，Human 继承自 Animal。

```
function Animal() {
  this.legs = 4;
}
function Human(name) {
  this.name = name;
}
Human.prototype = new Animal();
let human = new Human('es8');
```

使用 `for...in` 遍历

```
for (let key in human) {
  console.log(key);
}
// 'name', 'legs'
```

使用 `for...of` + `Object.keys()` 遍历

```
for (let key of Object.keys(human)) {
  console.log(key);
}
// 'name'
```

### 3. Object.getOwnPropertyDescriptors
静态方法 `Object.getOwnPropertyDescriptors` 用于获取对象的属性描述符，该属性必须是对象自己定义而不是继承自原型链。结果中包含的键可能有 configurable、enumerable、writable、get、set 以及 value。

```
const obj = { es8: 'hello es8' };
Object.getOwnPropertyDescriptor(obj, 'es8');
// {
//   configurable: true,
//   enumerable: true,
//   value: "hello es8"
//   writable: true
// }
```

### 4. Trailing commas in function
ES8 标准中允许函数参数列表与调用中的尾部逗号，该特性允许我们在定义或者调用函数时添加尾部逗号。

```
function es8(var1, var2, var3,) {
  // do something
}
es8(10, 20, 30,);
```

> 思考：在上述例子中，函数内部的 arguments.length 是 3 还是 4 ？

### 5. Async functions

为解决异步调用引入的 async 函数，由于 Babel 和 Nodejs 很早就支持 async 和 await 关键字，这个特性应该是最众望所归、最应用广泛的 ES8 特性了。

Async 函数主要是从 ES6 的 generator 和 yield 进化而来，另外还得益于 TJ 大神的 [co][npm-co] 模块的广泛应用，使得 JavaScript 的异步流程控制在 Async 函数进入标准之前就已经在社区经过了广泛的实践和讨论。这里就不对 Async 函数再做介绍了，社区里有很多优秀的文章，大家自行搜索吧。

### 6. Shared memory and atomics

SharedArrayBuffer 和 Atomics 是 JavaScript 为多线程能力增加的特性，暂时使用的场景不多，更多信息可以参考这个知乎的讨论： [hax 的回答 —— JavaScript 如果拥有多线程能力会怎样？][zhihu-hax-answer]，还有这篇文章对 Shared memory and atomics 介绍得很详细 [《ES proposal: Shared memory and atomics》][shared-array-buffer]



## 参考文献
- [[ECMAScript] TC39 process][ECMAScript-TC39-process]
- [The TC39 process for ECMAScript features][tc39-process]
- [ECMAScript 6 入门][ruanyifeng-es6]

[kick-off-es6]: https://github.com/meikidd/blog/blob/master/kick-off-es6-1.md
[ecma-262-8.0]: http://www.ecma-international.org/ecma-262/8.0
[tc39-proposals]: https://github.com/tc39/proposals
[tc39-process]: http://2ality.com/2015/11/tc39-process.html
[ECMAScript-TC39-process]: http://www.jianshu.com/p/b0877d1fc2a4
[es8-padStart]: http://www.ecma-international.org/ecma-262/8.0/#sec-string.prototype.padstart
[es8-padEnd]: http://www.ecma-international.org/ecma-262/8.0/#sec-string.prototype.padend
[left-pad]: https://zhuanlan.zhihu.com/p/20669077
[example]: https://raw.githubusercontent.com/meikidd/blog/master/images/example.png
[shared-array-buffer]: http://2ality.com/2017/01/shared-array-buffer.html
[zhihu-hax-answer]: https://www.zhihu.com/question/50911384/answer/123291232
[ruanyifeng-es6]: http://es6.ruanyifeng.com/#docs/object#Object-keys，Object-values，Object-entries
[npm-co]: https://www.npmjs.com/package/co