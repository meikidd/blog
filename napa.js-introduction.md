# Napa.js 简介

本文介绍 Napa.js 的核心概念，带领大家探索 Napa.js 是如何运转起来的。关于它的由来和开发初衷，可以阅读 [这篇文章](https://github.com/Microsoft/napajs/wiki/why-napa.js)


## 简介

### Zone
Zone 是 Napa.js 中的核心概念，它是执行 JavaScript 代码的基本单元，所有涉及多线程相关的内容都离不开 Zone 这个概念。一个进程可以包含多个 zone，而每个 zone 又由多个 JavaScript Worker 组成。

![](https://github.com/Microsoft/napajs/raw/master/docs/architecture.png)

在 zone 内部的所有 worker 都是相似的：他们加载相同的代码，几乎以相同的方式处理 `broadcast` 和 `execute` 请求，你无法指定执行某一个特定 worker 中的代码。在不同 zone 之间的 worker 是完全不同的：他们加载不同的代码，或者虽然加载相同代码但以不同的策略执行，例如堆栈大小不同、安全策略不同等。应用会利用多个 zone 来加载不同的策略。

有两种类型的 zone：

- Napa zone - 由多个 Napa.js 管理的 JavaScript worker 组成。Napa zone 内的 worker 支持部分 Node.js API
- Node zone - 暴露了 Node.js event loop 的虚拟 zone，具有完备的 Node.js 能力

这样划分让你既可以用 Napa zone 处理繁重的计算事务，也可以用 Node zone 处理 IO 事务。同时 Node zone 也是对 Napa zone 无法完整支持 Node API 的一种补充。

以下代码创建了一个包含 8 个 worker 的 Napa zone：

```
var napa = require('napajs');
var zone = napa.zone.create('sample-zone', { workers: 8 });
```

以下代码演示如何访问 Node zone：

```
var zone = napa.zone.node;
```

在 zone 上可以做两种类型的操作：

1. Broadcast - 所有 worker 执行同样的代码，改变 worker 状态，返回 promise 对象。不过我们只能通过 promise 的返回结果判断执行成功还是失败。通常用 `broadcast` 来启动应用、预加载一些数据或者修改应用设置。
2. Execute - 在一个随机 worker 上执行，不改变 worker 状态，返回一个包含结果数据的 promise。 `execute` 通常是用来做实际业务的。

Zone 的操作采用“先进先出”的策略，但 `broadcast` 比 `execute` 优先级更高。

以下代码演示了使用 `broadcast` 和 `execute` 完成一个简单的任务：

```
function foo() {
   console.log('hi');
}

// This setups function definition of foo in all workers in the zone.
zone.broadcast(foo.toString());

// This execute function foo on an arbitrary worker.
zone.execute(() => { global.foo() });
```

### 数据传输
由于 V8 不适合在多个 isolate 间执行 JavaScript 代码，每个 isolate 管理自己内部的堆栈。在 isolate 之间传递值需要封送/拆收（marshalled/unmarshalled），载荷的大小和对象复杂度决定着通信效率。所有 JavaScript isolate 都属于同一个进程，且原生对象可以被包装成 JavaScript 对象，我们尝试在此基础上为 Napa 设计一种高效传输数据的模式。

为了实现上述模式，引入了以下概念：

#### 可传输类型
可传输类型是指可以在 worker 中自由传输的 JavaScript 类型。包括

- JavaScript 基础类型：null, boolean, number, string
- 实现了 `Transportable` 接口的对象（TypeScript class）
- 由以上类型构成的数组或对象
- 还有 undefined

### 跨 worker 存储
Store API 用于在 JavaScript worker 中共享数据。当执行 `store.set` 时，数据被封送到 JSON 并存储在进程的堆栈中，所有线程都可以访问；当执行 `store.get` 时，数据被拆收出来。

以下代码演示如何利用 store 共享数据：

```
var napa = require('napajs');

var zone = napa.zone.create('zone1');
var store = napa.store.create('store1');

// Set 'key1' in node.
store.set('key1', { 
    a: 1, 
    b: "2", 
    c: napa.memory.crtAllocator      // transportable complex type.
};

// Get 'key1' in another thread.
zone.execute(() => {
    var store = global.napa.store.get('store1');
    console.log(store.get('key1'));
});
```

尽管很方便，但不建议在同一个事务里用 store 传值，因为这样做不仅仅只传输了数据（还附带了别的事情，比如加锁）。另外，虽然有垃圾回收机制，但开发者还是应当在使用完数据后手动删除相应的 key。

## 安装
执行 `npm install napajs` 安装。

<del>在 OSX 系统安装后执行会报错，在 [github issue](https://github.com/Microsoft/napajs/issues/80) 中里也有同样的提问，解决方法是按照官方的[构建文档](https://github.com/Microsoft/napajs/wiki/build-napa.js)，自己手动构建。</del> 最新版 v0.1.4 版本已经修复上述问题。

步骤如下:
1. 安装先决依赖
    - Install C++ compilers that support C++14:
        - `xcode-select --install`
    - Install CMake:
        - `brew install cmake`
    - Install cmake-js:
        - `npm install -g cmake-js`

2. 通过 npm 构建
    - `npm install --no-fetch`

## 快速上手示例

### 计算圆周率 π 值
下面是一个计算 π 值的例子，演示了如何利用多线程执行子任务。
```
var napa = require("napajs");

// Change this value to control number of napa workers initialized.
const NUMBER_OF_WORKERS = 4;

// Create a napa zone with number_of_workers napa workers.
var zone = napa.zone.create('zone', { workers: NUMBER_OF_WORKERS });

// Estimate the value of π by using a Monte Carlo method
function estimatePI(points) {
    var i = points;
    var inside = 0;

    while (i-- > 0) {
        var x = Math.random();
        var y = Math.random();
        if ((x * x) + (y * y) <= 1) {
            inside++;
        }
    }

    return inside / points * 4;
}

function run(points, batches) {
    var start = Date.now();

    var promises = [];
    for (var i = 0; i < batches; i++) {
        promises[i] = zone.execute(estimatePI, [points / batches]);
    }
    
    return Promise.all(promises).then(values => {
        var aggregate = 0;
        values.forEach(result => aggregate += result.value);
        printResult(points, batches, aggregate / batches, Date.now() - start);
    });
}

function printResult(points, batches, pi, ms) {
    console.log('\t' + points
          + '\t\t' + batches
          + '\t\t' + NUMBER_OF_WORKERS
          + '\t\t' + ms
          + '\t\t' + pi.toPrecision(7)
          + '\t' + Math.abs(pi - Math.PI).toPrecision(7));
}

console.log();
console.log('\t# of points\t# of batches\t# of workers\tlatency in MS\testimated π\tdeviation');
console.log('\t---------------------------------------------------------------------------------------');

// Run with different # of points and batches in sequence.
run(4000000, 1)
.then(result => run(4000000, 2))
.then(result => run(4000000, 4))
.then(result => run(4000000, 8))
```

运行结果如下，当设置为 1 组、2 组、4 组子任务并行计算时，可以看出执行时间有明显提升，当设置为 8 组子任务并行计算时，由于没有更多的空闲 worker 资源，也就没有明显的执行时间的提升。

```
# of points	# of batches	# of workers	latency in MS	estimated π	deviation
---------------------------------------------------------------------------------------
40000000	1		4		1015		3.141619	0.00002664641
40000000	2		4		532		3.141348	0.0002450536
40000000	4		4		331		3.141185	0.0004080536
40000000	8		4		326		3.141620	0.00002724641
```

### 计算斐波那契数列

```
var napa = require("napajs");

// Change this value to control number of napa workers initialized.
const NUMBER_OF_WORKERS = 4;

// Create a napa zone with number_of_workers napa workers.
var zone = napa.zone.create('zone', { workers: NUMBER_OF_WORKERS });

/*
Fibonacci sequence 
n:              |   0   1   2   3   4   5   6   7   8   9   10  11  ...
-------------------------------------------------------------------------
NTH Fibonacci:  |   0   1   1   2   3   5   8   13  21  34  55  89  ...
*/
function fibonacci(n) {
    if (n <= 1) {
        return n;
    }

    var p1 = zone.execute("", "fibonacci", [n - 1]);
    var p2 = zone.execute("", "fibonacci", [n - 2]);

    // Returning promise to avoid blocking each worker.
    return Promise.all([p1, p2]).then(([result1, result2]) => {
        return result1.value + result2.value;
    });
}

function run(n) {
    var start = Date.now();

    return zone.execute('', "fibonacci", [n])
        .then(result => {
            printResult(n, result.value, Date.now() - start);
            return result.value;
        });
}

function printResult(nth, fibonacci, ms) {
    console.log('\t' + nth
          + '\t' + fibonacci
          + '\t\t' + NUMBER_OF_WORKERS
          + '\t\t' + ms);
}

console.log();
console.log('\tNth\tFibonacci\t# of workers\tlatency in MS');
console.log('\t-----------------------------------------------------------');

// Broadcast declaration of 'napa' and 'zone' to napa workers.
zone.broadcast(' \
    var napa = require("napajs"); \
    var zone = napa.zone.get("zone"); \
');
// Broadcast function declaration of 'fibonacci' to napa workers.
zone.broadcast(fibonacci.toString());

// Run fibonacci evaluation in sequence.
run(10)
.then(result => { run(11)
.then(result => { run(12)
.then(result => { run(13)
.then(result => { run(14)
.then(result => { run(15)
.then(result => { run(16)
}) }) }) }) }) })
```

运算结果

```
Nth	Fibonacci	# of workers	latency in MS
-----------------------------------------------------------
10	55		4		10
11	89		4		13
12	144		4		15
13	233		4		22
14	377		4		31
15	610		4		50
16	987		4		81
```
