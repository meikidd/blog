# ES6
https://github.com/ES-CN/es6features/blob/master/README.md

### 箭头函数
箭头函数和其上下文中的代码共享同一个具有词法作用域的this

### 增强的Object字面量
```
var obj = {
    // __proto__
    __proto__: theProtoObj,
    // Shorthand for ‘handler: handler’
    // ‘handler: handler’ 的简写形式
    handler,
    // Methods
    toString() {
      // Super calls
      return "d " + super.toString();
    },
    // Computed (dynamic) property names
    // 计算所得的（动态的）属性名称
    [ 'prop_' + (() => 42)() ]: 42
};
```

### 模板字符串
```
// 基础字符串字面量
`In JavaScript '\n' is a line-feed.`
```
```
// 多行字符串
`In JavaScript this is
 not legal.`
```

### 默认参数+不定参数+参数展开
#### 默认参数
```
function f(x, y=12) {
  // y is 12 if not passed (or passed as undefined)
  return x + y;
}
f(3) == 15
```
#### 不定参数
在函数定义时使用...运算符，则可以将函数尾部的多个参数绑定到一个数组中。
```
function f(x, ...y) {
  // y is an Array
  return x * y.length;
}
f(3, "hello", true) == 6
```
#### 参数展开
在函数调用时使用...运算符，可以将作为参数的数组拆解为连续的多个参数。 
```
function f(x, y, z) {
  return x + y + z;
}
// Pass each elem of array as argument
f(...[1,2,3]) == 6
```

### 迭代器

### 生成器

### modules 模块

### 模块加载器

### Promise 对象

### Map + Set + WeakMap + WeakSet 数据结构

### 