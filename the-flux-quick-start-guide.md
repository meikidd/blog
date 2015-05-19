# [译] Flux 入门

原文链接：[The Flux Quick Start Guide](http://www.jackcallister.com/2015/02/26/the-flux-quick-start-guide.html)

本文将概括性的介绍如何使用 Flux 架构开发 JavaScript 应用，用尽可能少的篇幅带你熟悉 Flux 的核心概念。你也可以结合 [starter kit](https://github.com/jarsbe/flux-starter-kit) 一起学习。你最好先对 React 有基本的了解，并且有一些开发 React 组件的经验。如果不熟悉也没关系，可以先读一读这篇文章 [The React Quick Start Guide](http://www.jackcallister.com/2015/01/05/the-react-quick-start-guide.html) (译注：中文版本 [React 入门](http://segmentfault.com/a/1190000002759878))。

---------------

## 概念
Flux 是用来构建用户端 Web 应用的架构，它包含三个核心概念：**Views**, **Stores** 和 **Dispatcher**，还有一些次级概念：**Actions**, **Action Types**, **Action Creators** 和 **Web Utils**。

请耐心学习以下概念定义然后再看后面的教程。当你准备开始开发 Flux 应用之前，建议你再回过头来看一遍基本概念。

### 核心概念
**Views** 即 React 组件。它们负责渲染界面，捕获用户事件，从 Stores 获取数据。

**Stores** 用于管理数据。 一个 Store 管理一个区域的数据，当数据变化时它负责通知 Views。

**Dispatcher** 接收新数据然后传递给 Stores，Stores 更新数据并通知 Views。

### 次级概念
**Actions** 是传递给 Dispatcher 的对象，包含新数据和 Action Type。

**Action Types** 指定了可以创建哪些 Actions，Stores 只会更新特定 Action Type 的 Actions 触发的数据。

**Action Creators** 是 Actions 的创建者，并将其传递给 Dispatcher 或 Web Utils。

**Web Utils** 是用于与外部 API's 通信的对象。例如 Actions Creator 可能需要从服务器请求数据。

----------

是不是一次给的信息量太多啦？强烈建议你们结合 [starter kit](https://github.com/jarsbe/flux-starter-kit) 边看文章边敲代码，可以达到更好的学习效果。

提示：这里省略了 constants 和 Web Utils，是为了更快速简单地理解 Flux。更深入阅读 [官方示例](https://github.com/facebook/flux/tree/master/examples) 能很好地补充这些知识。

## Views
部署好 [starter kit](https://github.com/jarsbe/flux-starter-kit) 后，你会看到在 `src` 目录下有个 `app.js` 文件。
```javascript
var React = require('react');
var Comments = require('./views/comments');
var CommentForm = require('./views/comment-form');

var App = React.createClass({
  
  render: function() {
    return (
      <div>
        <Comments />
        <CommentForm />
      </div>  
    );
  }
});

React.render(<App />, document.getElementById('app'));
```

上面代码把 Views 渲染到 DOM 中。先忽略 `Comments`，看一下 `CommentFrom` 的实现。

```javascript
var React = require('react');

var CommentActionCreators = require('../actions/comment-action-creators');

var CommentForm = React.creatClass({
  
  onSubmit: function(e) {
    var textNode = this.refs.text.getDOMNode();
    var text = textNode.value;

    textNode.value = '';

    CommentActionCreators.createComment({
      text: text
    });
  },

  render: function() {
    return (
      <div className='comment-form'>
        <textarea ref='text' />
        <button onClick={this.onSubmit}>Submit</button>
      </div>
    );
  }
});

module.exports = CommentForm;
```

`CommentForm` 依赖的 `CommentActionCreators` 是一个 Action Creator (正如它的名字一样)。

当表单提交时 `createComment` 函数传递了 `comment` 对象，它的值是根据 textarea 的值构造出来的。让我们开发这个 Action Creator 来接收 comment。

----------

## Actions
在 `actions` 目录里有如下的 `comment-action-creators.js` 文件。

```javascript
var AppDispatcher = require('../dispatcher/app-dispatcher');

module.exports = {
  
  createComment: function(comment) {
    var action= {
      actionType: "CREATE_COMMENT",
      comment: comment
    };

    AppDispatcher.dispatch(action);
  }
};
```

`createComment` 函数构造了一个 Action，包含 Action Type 和 comment 数据，并将这个 Action 传递给 Dispatcher 的 `dispatch` 函数。

接下来编写 Dispatcher 用于接收 Actions。

提示：也可以把这些逻辑写在 View 里面 - 直接跟 Dispatcher 通信，但最佳实践是用 Action Creator。它能降低代码的耦合度并给 Dispatcher 提供一个单独的接口。

---------------

## Dispatcher
在 `dispatcher` 目录下有一个 `app-dispatcher.js` 文件。

```javascript
var Dispatcher = require('flux').Dispatcher;

module.exports = new Dispatcher();
```

Flux 库的 Dispatcher 提供了一个 `dispatch` 函数，将接收到的 Actions 传递给所有注册的回调函数，回调函数由 Stores 提供。

提示：这里没有 Dispatcher 的具体实现，源码在[这里](https://github.com/facebook/flux/blob/master/src/Dispatcher.js#L181)。

---------------

## Stores
在 `stores` 目录下有一个 `comment-store.js` 文件。

```javascript
var AppDispatcher = require('../dispatcher/app-dispatcher');

var EventEmitter = require('events').EventEmitter;
var assign = require('object-assign');


var comments = [];

var CommentStore = assign({}, EventEmitter.prototype, {
  
  emitChange: function() {
    this.emit('change');
  },

  addChangeListener: function(callback) {
    this.on('change', callback);
  },

  removeChangeListener: function(callback) {
    this.removeListener('change', callback);
  },

  getAll: function() {
    return comments;
  }
});

AppDispatcher.register(function(action) {
  switch(action.actionType) {

    case "CREAT_COMMENT":
      comments.push(action.comment);
      CommentStore.emitChange();
      break;

    default:
  }
});

module.exports = CommentStore;
```

这段代码分为两部分：创建 Store 和 注册 Store。

Store 由 `EventEmitter.prototype` 和自定义对象整合而成。`EventEmitter.prototype` 给 Store 赋予了订阅和触发事件的能力。

自定义对象定义了订阅和取消订阅事件的函数，同时定义了 `getAll` 函数返回 `comments` 数据。

然后，通过 Dispatcher 注册了一个回调函数。当 Dispatcher 调用 `dispatch` 时传递 Actions 参数给每个注册过的回调函数。

---------------

现在我们需要一个 View 来展示 Store 的数据，并订阅数据的变化。

在 `views` 目录里有个 `comments.js` 文件。把它修改成如下所示：

```javascript
var React = require('react');

var CommentStore = require('../stores/comment-store');

function getStateFromStore() {
  return {
    comments: CommentStore.getAll()
  }
}

var Comments = React.createClass({
  
  onChange: function() {
    this.setState(getStateFromStore());
  },

  getInitialState: function() {
    return getStateFromStore();
  },

  componentDidMount: function() {
    CommentStore.addChangeListener(this.onChange);
  },

  componentWillUnmount: function() {
    CommentStore.removeChangeListener(this.onChange);
  },

  render: function() {
    var comments = this.state.comments.map(function(comment, index) {
      return (
        <div className='comment' key={'comment-' + index}>
          {comment.text}
        </div>
      );
    });

    return (
      <div className='comments'>
        {comments}
      </div>
    )
  }
});

module.exports = Comments;
```

`getStateFromStores` 函数从 Store 获取 comment 数据，并在 `getInitialState` 中设置为初始值。

在 `componentDidMount` 中，`onChange` 函数作为 `addChangeListener` 的回调函数，当 Store 触发 `change` 事件时 `onChange` 函数将被调用，即当 Store 数据变化时，它用于更新组件的 state 状态。

最后 `componentWillUnmount` 将 `onChange` 事件监听从 Store 移除。

------------------

## 结语

现在这个 Flux 应用可以运行起来了，同时我们也学习了 Flux 架构的核心概念：Views, Stores 和 Dispatcher。

- 当提交 comment 时，View 调用了 Action Creator
- Action Creator 创建一个 Action 并传给 Dispatcher
- Dispatcher 将 Action 发送给 Store 中注册的回调函数
- Store 更新 comment 数据，并触发一个 change 事件
- View 更新 state 并重新渲染界面

这就是 Flux 的本质，Dispatcher 发送数据给所有 Stores，后者通知 Views 进行更新。

要想更深入理解 Flux 架构，我建议阅读 [官方文档](https://facebook.github.io/flux/)，或者看看这个 [视频教程](https://www.youtube.com/watch?v=nYkdrAPrdcw&list=PLb0IAmt7-GS188xDYE-u1ShQmFFGbrk0v)，还有 [官方示例](https://github.com/facebook/flux/tree/master/examples)。

如果本文有什么错误之处，欢迎在 [twitter](http://twitter.com/jarsbe) 上联系我，或者给我提 [pull request](https://github.com/jarsbe/jarsbe.github.io)。欢迎大家给我提建议改善这边文章。
