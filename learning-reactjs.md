# ReactJS 学习笔记


- [我是这样看待 React 的](http://react-china.org/t/wo-shi-zhe-yang-kan-dai-react-de/28)
  - 对比 jQuery
  - Flux 和 MVC 的区别

- [聊一聊基于Flux的前端系统](http://react-china.org/t/liao-liao-ji-yu-fluxde-qian-duan-xi-tong/615)
  - 最佳实践
  - 讲了一些具体问题的实现思路和细节
  - 他们用了 reflux 而不是 facebook flux

- 如何在团队中推动 ReactJS(其他新技术同理) ?
  - 不要一开始就搬出来说要全部替换旧技术，多做一些分享，渲染一些气氛，show 出一些 awesome 的东西
  - 对比新老技术的优缺点
  - 在一些小的项目/页面上应用起来

- [react 工程化问题](http://react-china.org/t/wen-xia-da-jia-zai-xiang-mu-zhong-shi-yong-reactde-shi-hou-xie-gong-cheng-hua-de-wen-ti/287)
  - webpack
  - requireJS, CommonJS
  - browserify + gulp

- **[flux架构中的component是否该引入store](http://react-china.org/t/fluxjia-gou-zhong-de-componentshi-fou-gai-yin-ru-store/127)**
  - 关于大型应用整个界面刷新
  - 如何拆分 store 和 components，如何传递数据
  - **过于细节，需要入门以后回过头来再看**

- [Ajax 是写在 Store 里还是 Component 里更合适](http://react-china.org/t/ajax-shi-xie-zai-store-li-huan-shi-component-li-geng-he-gua/169)
  - 官方建议是写的专门的WEB-API Utils里面，由ActionCreator调用WEB-API Utils
写在Store里是绝对不可取的，因为Store一旦异步化会导致dispatcher的waitFor失效（看waitFor的源码就知道了）

- 延展属性

```
var props = {
  foo: x,
  bar: y
};
var component = <Component {...props} />;
```

- [react training](https://github.com/ryanflorence/react-training)

- [Working with jQuery UI Dialog and ReactJS components](http://sterling.ghost.io/working-with-jqueryui-and-reactjs-components/)

- [Integrate jQuery UI autocomplete and React](http://ludovf.net/reactbook/blog/reactjs-jquery-ui-autocomplete.html)

- 快速入门

  - [React 入门教程](http://hulufei.gitbooks.io/react-tutorial/content/index.html)
  - [React 入门视频教程](http://www.tudou.com/listplay/ah20h1-t4V4/ikzB7N0ssxw.html?FR=LIAN)


---------
### 参考
