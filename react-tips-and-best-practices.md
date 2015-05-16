# ReactJS 实用技巧和最佳实践

我去年用 [React](http://facebook.github.io/react/) 进行大量了开发工作，写了很多 React 组件，也重构了很多，并且从中总结出一些最佳实践经验。

这里不会介绍什么是 React 以及为什么我们要用 React，网上充斥着大量这类文档（译注：参考这篇 [React 入门指南](http://segmentfault.com/a/1190000002759878)）。我假设你已经具备 React 的基础知识，并且已经用它写过一两个组件。

## 使用 PureRenderMixin

[PureRenderMixin](http://facebook.github.io/react/docs/pure-render-mixin.html) 是一种 [mixin](http://reactjs.cn/react/docs/reusable-components.html#mixins)，它覆盖了 `shouldComponentUpdate` 方法，只有当 `props` 和 `state` 发生改变时才会 重新渲染组件。React 本身的性能已经非常好了，但 PureRenderMixin 在此之上还能提升更多。有了它你也可以更频繁的使用 `setState` 而不必担心被重复渲染。更不必在你的代码中添加这样的检查：
```javascript
if(this.state.someVal !== computedVal) {
  this.setState({someVal: computedVal})
}
```

用 PureRenderMixin 的前提是你的 `render()` 方法必须是“纯粹”的。当传给它同样的 `props` 和 `state` ，它应该渲染出同样的效果。这意味着你不能在 `render()` 方法中使用类属性，或者当属性值变化时渲染出不一样的内容。
```javascript
render: function () {
  //...
  if(this._previousFoo !== this.props.foo) {
    return renderSomethingDifferent();
  }
}
```

在实践中使用 `PureRenderMixin` 不会有什么问题，我在好几个组件中使用它，效果很好。如果你发现加上它之后出了问题，可以考虑重构一下你的组件，因为这通常表示你在组件里写了一些不规范的奇怪的东西。

记住重要的一点，它只是对 `nextProps` 和 `nextState` 做浅比较。如果这些值是对象或数组，就算修改它的属性值或对数组添加一项新值，都不会触发组件的更新，因为引用没有变。所以如果你要修改对象或数组，最好返回一个新的实例以更新引用。

有很多方法实现这一点。[Immutable.js](http://facebook.github.io/immutable-js/) 和 [mori](http://swannodette.github.io/mori/) 提出了新的不可变数据类型。如果你还是想用原生的 JavaScript 对象，可以选择 React 提供的 [immutability helpers](http://facebook.github.io/react/docs/update.html) 或我写的 [icepick](https://npmjs.org/package/icepick) 来把固定不变的 JavaScript 对象伪装成持续更新的集合。

用这些库操作 `props` 确实增加了一些工作量，但是能有效减少无意义的重复渲染还是很值得的。还有一种观点认为应该尽量减少对 `props` 和 `state` 使用对象，属性越简单就越能减少重新渲染的可能性。

如果你还没有使用 PureRenderMixin，你就错过了 React 最优秀的特性。它能极大的帮助简化代码、提升性能。

## 使用 PROP TYPES
当你开发的应用越来越大，开始把各种组件组合起来形成一个复杂的架构，这时候调试 `props` 将痛苦无比。幸好 React 提供了一种方法检测组件 `props` 的类型和是否必须: `PropTypes`。每个 React 类都可以定义一个 `propTypes map` 为每个 `props` 指定校验行数。如果校验失败，会在控制台输出一条 warning 信息。比如当一个 prop 是 `isRequired` 型而你却忘了传值，或者它是 `object` 型而你传了一个 `string` 型的值时，都能在控制台得到有用的提示信息。

`propTypes` 同时也具有说明文档的作用，指出组件需要哪些 `props` 以及它们是用来做什么的。

另外，`propType` 类型检查只在非生产环境使用，请将环境变量设置为 `NODE_ENV="production"` 以确保它不会拖慢生产环境的性能。用 [envify](https://github.com/hughsk/envify) 配合 `uglify --compress` 也是手动设置的方式。

另外，每个非 `isRequired` 类型的 `propType` 都要在 `getDefaultProps()` 里声明默认值（此法并不总是可行，因为 `getDefaultProps()` 只在 `createClass` 时调用一遍，而不是每次实例化时调用，也就是说所有实例会共享 `getDefaultProps()` 的执行结果）。反过来想想，如果一个 `prop` 不是必须的，又没有默认值，那我们真的需要它吗？

## 避免使用 STATE
"Avoid State" 几乎是所有编程开发的金玉良言，在 React 中也不例外。即便 React 非常赞的采取了很多措施让我们可以用上 state，但还是应该避免使用它。

State 不是个好的选择。它会增加组件复杂度，造成组件理解困难，触发各种反复重新渲染，就算你用了 `PureRenderMixin` 也没用。（译注：这部分观点译者认为有待讨论，可能作者的本意是告诉大家谨慎使用 state，但也不是绝对不能用。）最糟的情况是将 `setState` 写在 `componentDidMount()` 或 `componentDidUpdate()` 后面。当尝试从 DOM 中读取参数时，会导致 `render()` 执行两次。如果在子组件中也这么干了，将会调用四次 `render ()`：父组件两次，它内部的 `setState()` 两次。在复杂组件中，这将导致界面卡顿，在更深层级的子组件嵌套的情况下，甚至渲染次数成指数倍增长。

然而，有些场景我们确实需要 state，即使要用，也请确保将它封装在组件内部。如果父组件还需要知道子组件的 state 逻辑，那一定是组件构造有问题，组件抽象得不够好，最好对它进行重构。

