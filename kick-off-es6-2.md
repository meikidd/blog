# ES6 旧瓶新酒

本文是 ES6 系列的第二篇，前一篇[《ES6 走马观花》](http://segmentfault.com/a/1190000003764489)概要介绍了ES6，这一篇标题之所以叫“旧瓶新酒”，是想介绍那些原来就被广泛使用的JS对象，例如String、Array，ES6对这些JS对象扩展了一些很有用的新方法。本文会介绍一些个人觉得很有用的方法，并不会覆盖所有ES6新增方法。

本文将介绍以下JS对象：
- String
- Number
- Array
- Object

这里有ES6特性的兼容[对比表格](http://kangax.github.io/compat-table/es6/#babel)，方面大家查看本文提及的ES6特性在各个运行环境的兼容情况。

## String
### Unicode 表示法
1) Unicode 表示法  

JavaScript允许采用`\uxxxx`形式表示一个字符，支持`\u0000—\uFFFF`之间的字符，对于超过`\uFFFF`的字符，使用两个字节的形式:
```
'\uD842\uDFB7'
// '𠮷'
```
ES6 在此基础上做了改进，引入大括号来表示一个码点（code point）
```
'\u{20BB7}'
// '𠮷"

'\u0041'
// 'A'

'\u{41}'
// 'A'
```
除此之外，还扩展了`String.prototype.codePointAt()`和`String.fromCodePoint()`方法，可以识别超出`\uFFFF`的码点。  

2) codePointAt()

以前通过`charCodeAt()`获取字符码点，但是对超出`\uFFFF`的双字节字符则会出错，只能获取第一个字节的码点，ES6新增了`codePointAt()`方法来获取字符的unicode码点，可以正确识别双字节的字符。
```
"𠮷".charCodeAt(0)
// 55362
"𠮷".charCodeAt(0).toString(16)
// 'd842'

"𠮷".codePointAt(0)
// 134071
"𠮷".codePointAt(0).toString(16)
// '20bb7'
```
3) String.fromCodePoint()

此方法用于从码点返回对应字符，支持识别双字节字符。
```
String.fromCodePoint(0x20bb7)
// '𠮷'
```


### 遍历器接口
ES6 的字符串实现了 Iterator 遍历器接口，因此可以用 `for...of` 循环遍历字符串。  

```
for (let codePoint of 'foo') {
  console.log(codePoint)
}
// 'f'
// 'o'
// 'o'
```
与普通 `for` 循环最大的不同是，`for...of` 循环遍历的单位是字符码点（code point），而 `for` 循环遍历的单位是字符单元（code unit）
```
var text = String.fromCodePoint(0x20BB7);

for (let i = 0; i < text.length; i++) {
  console.log(text[i]);
}
// ' '
// ' '

for (let i of text) {
  console.log(i);
}
// '𠮷'
```

### 实用方法
1) includes()

以前无法直接判断一个字符串是否另一个字符串的子字符串，只能通过`indexOf()`方法来判断。
```
'hello world'.indexOf('hello')
// 0
'hello world'.indexOf('es6')
// -1
```
在 ES6 中可以直接使用`includes()`直接判断。
```
'hello world'.includes('hello')
// true
'hello world'.includes('es6')
// false
```
includes() 还可以接收第二个参数，表示从第几位开始检索
```
'hello world'.includes('hello', 1)
// false
```

2) startsWith(), endsWith()

在ES6中还可以直接判断一个字符串是否以某个特定子串开头，或是否以某个特定子串结尾。  
以前只能用`String.prototype.indexOf()`和`RegExp.prototype.test()`判断：
```
'hello world'.indexOf('hello')
// 0
/world$/.test('hello world')
// true
```
在ES6中可以这么写
```
'hello world'.startsWith('hello')
// true
'hello world'.endsWith('world')
// true
```
startsWith() 和 endsWith() 还可以接收第二个参数，表示从第几位开始检索
```
'hello world'.startsWith('world', 6)
// true
'hello world'.endsWith('hello', 5)
// true
```

3) repeat()

此方法返回一个新字符串，表示将原字符串重复n次。
```
'hello'.repeat(2)
// 'hellohello'
```


## Number
### 二进制和八进制
ES6 里可以用`0b`前缀表示二进制，用`0o`表示八进制
```
console.log(0b1100)
// 12

console.log(0o1100)
// 576
```
### 指数运算
ES6 新增了指数运算符 `**`
```
console.log(2 ** 4)
// 16

console.log(Math.pow(2, 4))
// 16
```
### Number.isFinite()
`Number.isFinite()` 与全局函数 `isFinite()` 的区别是，只判断数值，所有非数值均返回false，而`isFinite()` 会先将非数值的值转为数值
```
Number.isFinite(16)
// true
Number.isFinite('16')
// false

isFinite(16)
// true
isFinite('16')
// true

Number.isFinite(Infinity)
// false
```
### Number.isNaN()
与 `Number.isFinite()` 类似，`Number.isNaN()` 值判断数值，所有非数值均返回false
```
Number.isNaN(NaN)
// true
Number.isNaN('a')
// false
isNaN('a')
// true
```

### Number.parseInt(), Number.parseFloat()
`Number.parseInt(), Number.parseFloat()`与全局函数`parseInt(), parseFloat()`作用相同，放到 Number 对象的静态方法，目的是逐步减少全局性方法，使得语言逐步模块化。

### Number.isInteger()
`Number.isInteger()` 用来判断一个值是否为整数。
```
Number.isInteger(16)
// true
Number.isInteger(16.2)
// false
Number.isInteger('16')
// false
```


### Number.EPSILON
由于 Javascript 的浮点运算不是精确的，存在误差。`Number.EPSILON` 表示一个极小常量，当浮点运算的误差小于这个常量时，就认为浮点运算结果是正确的。
```
0.1 + 0.2
// 0.30000000000000004

0.1 + 0.2 - 0.3 < Number.EPSILON
// true
```

### Number.isSafeInteger()
用 `Number.isSafeInteger()` 函数判断整数是否在-2^53到2^53之间，因为JavaScript能够准确表示的整数范围在-2^53到2^53之间。
```
Number.isSafeInteger(16)
// true

Number.isSafeInteger(2 ** 54)
// false
```
## Array

### Array.from()
`Array.from()` 用于将对象转换为数组。  
例如将字符串转为数组：
```
Array.from('hello')
// ["h", "e", "l", "l", "o"]
```
例如将DOM的NodeList转为数组：
```
var elm = document.querySelectorAll('div');
Array.from(elm).forEach(function (div) {
    // do something
});
```
`Array.from()` 函数还可以接受第二个参数，用于处理数组的每个元素。
```
Array.from('123', (x) => x * x)
// [1, 4, 9]
```
### Array.of()
`Array.of` 用于构造一个新数组，可以取代 `new Array()`。因为当 `new Array()` 一个参数和多个参数时，行为不一致。
```
new Array(3)
// [ , , ]
new Array(3, 2, 1)
// [3, 2, 1]

Array.of(3)
// [3]
Array.of(3, 2, 1)
// [3, 2, 1]
```

### Array.observe
用于监听（取消监听）数组的变化，指定回调函数。在ES7中已被建议撤销。

### find(), findIndex()
`find()` 遍历数组直到找到符合条件的元素，返回该元素，若找不到则返回undefined。`findIndex()` 用法跟 `find()` 类似，但返回值是元素的位置，若找不到则返回-1
```
['hello', 'world', 'es6'].find(function(item) {
  if(item === 'world') {
    return true;
  }
});
// 'world'

[1, 0, 2].findIndex(function(item) {
  if(item === 0) {
    return true;
  }
});
// 1
```

`find()`和`findIndex()` 还可以接收第二个参数，用于指定回调函数的 `this` 上下文



### entries(), keys(), values()
这三个方法都返回一个遍历器对象，用于遍历数组。`entries()`用于遍历数组键值对，`keys()`用于遍历数组下标，`values()`用于遍历数组的值。
```
for (let index of ['a', 'b'].keys()) {
  console.log(index);
}
// 0
// 1

for (let elem of ['a', 'b'].values()) {
  console.log(elem);
}
// 'a'
// 'b'

for (let [index, elem] of ['a', 'b'].entries()) {
  console.log(index, elem);
}
// 0 "a"
// 1 "b"
```
## Object
### Object 的简洁表示法
直接写变量
```
{x, y}
// 等同于
{
    x: x,
    y: y
}
```
直接写函数
```
{
    say() {
        console.log('I am ES6');
    }
}
// 等同于
{
    say: function() {
        console.log('I am ES6')；
    }
}
```
### 属性名表达式
ES6 支持在字面量定义对象时使用表达式作为属性名。
```
var world = 'es6';
var obj = {
    hello: 1,
    [world]: 2
};
// {hello: 1, es6: 2}
```
### Object.assign()
`Object.assign()` 方法拷贝一个对象属性到目标对象上。
```
Object.assign({}, {a: 1})
// {a: 1}

Object.assign({a: 1}, {a: 2})
// {a: 2}
```
对于嵌套的对象，Object.assign的处理方法是替换，而不是添加。
```
var target = {
    a: {
        b: 'hello', 
        c: 'world'
    }
};
var source = {
    a: {
        b: 'es6' 
    } 
};
Object.assign(target, source);
// { a: { b: 'es6' } }
```
## 参考
- [ECMAScript 6 入门](http://es6.ruanyifeng.com/)
- [ES6 Strings (and Unicode, ❤) in Depth](https://ponyfoo.com/articles/es6-strings-and-unicode-in-depth)