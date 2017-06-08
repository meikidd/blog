# Kick Off Web Components

跟 Web Components 打个啵

## What are Web Components

- [Web Components](http://www.w3.org/TR/components-intro/) 是 W3C 定义的新标准，目前还在草案阶段。

## Why are they important
- 前端组件化 
  - bootstrap

  ```
  	// 初始化
	$('#myModal').modal({
	  keyboard: false
	});

	// 显示
	$('#myModal').modal('show');

	// 关闭事件
	$('#myModal').on('hidden.bs.modal', function (e) {
	  // do something...
	});
  ```
  
  - atom

  ```
  // 初始化组件
  var dialog = new Dialog(
  	 trigger: '#trigger-btn',
	 title: '我是自定义的标题',
	 content: 'hello world',
	 buttons: ['submit', 'cancel']
  });
  
  // 显示
  dialog.show();
  
  // 关闭事件
  dialog.after('hide', function() {
  	 // do something...
  });
        
  ```
- 统一标准、减少轮子
- 简化代码，提高可维护性
	![gmail](http://www.html5rocks.com/zh/tutorials/webcomponents/customelements/gmail.png)

	```
	<hangout-module>
	  <hangout-chat from="Paul, Addy">
	    <hangout-discussion>
	      <hangout-message from="Paul" profile="profile.png"
	           datetime="2013-07-17T12:02">
	        <p>Feelin' this Web Components thing.</p>
	        <p>Heard of it?</p>
	      </hangout-message>
	    </hangout-discussion>
	  </hangout-chat>
	  <hangout-chat>...</hangout-chat>
	</hangout-module>
	```
	
## 关键技术
- HTML Imports
- HTML Templates
- Custom Elements
- Shadow DOM

![](http://img.blog.csdn.net/20150319170655718)

虽然大部分浏览器还不支持 Web Components ，但是有个叫做 [webcomponents.js](http://webcomponents.org/) 的兼容库，可以让 Web Components 在不支持它的浏览器上运行起来。只要你在项目中引入这个库，就可以在其他浏览器中将 Web Components 用起来。

### HTML Imports
通过`<link>`标签来引入 HTML 文件，使得我们可以用不同的物理文件来组织代码。

```
<link rel="import" href="http://example.com/component.html" >
```
> 注意：受浏览器同源策略限制，跨域资源的 import 需要服务器端开启 CORS。
> `Access-Control-Allow-Origin: example.com`

通过`import`引入的 HTML 文件是一个包含了 html, css, javascript 的独立 component。

```
<template>
    <style>
        .coloured {
            color: red;
        }
    </style>
    <p>My favorite colour is: <strong class="coloured">Red</strong></p>
</template>
<script>
    (function() {
        var element = Object.create(HTMLElement.prototype);
        var template = document.currentScript.ownerDocument.querySelector('template').content;
        element.createdCallback = function() {
            var shadowRoot = this.createShadowRoot();
            var clone = document.importNode(template, true);
            shadowRoot.appendChild(clone);
        };
        document.registerElement('favorite-colour', {
            prototype: element
        });
    }());
</script>
```

### HTML Templates
关于 HTML 模板的作用不用多讲，用过 mustache、handlbars 模板引擎就对 HTML 模板再熟悉不过了。但原来的模板要么是放在 `script` 元素内，要么是放在 `textarea` 元素内，HTML 模板元素终于给了模板一个名正言顺的名分: `<template>`  

原来的模板形式：

- script 元素

```
<script type="text/template">
	<div>
		this is your template content.
	</div>
</script>
```
- textarea 元素

```
<textarea style="display:none;">
	<div>
		this is your template content.
	</div>
</textarea>
```

现在的模板形式：

- template 元素

```
<template>
	<div>
		this is your template content.
	</div>
</template>
```
主要有四个特性：

1. 惰性：在使用前不会被渲染；
2. 无副作用：在使用前，模板内部的各种脚本不会运行、图像不会加载等；
3. 内容不可见：模板的内容不存在于文档中，使用选择器无法获取；
4. 可被放置于任意位置：即使是 HTML 解析器不允许出现的位置，例如作为 `<select>` 的子元素。

### Custom Elements
自定义元素允许开发者定义新的 HTML 元素类型。带来以下特性：

1. 定义新元素
2. 元素继承
3. 扩展原生 DOM 元素的 API

#### 定义新元素
使用 `document.registerElement()` 创建一个自定义元素：

```
var Helloworld = document.registerElement('hello-world', {
  prototype: Object.create(HTMLElement.prototype)
});

document.body.appendChild(new Helloworld());
```
标签名必须包含连字符 ' - '

- 合法的标签名：`<hello-world>`, `<my-hello-world>`
- 不合法的标签名：`<hello_world>`, `<HelloWorld>`

#### 元素继承
如果 `<button>` 元素不能满足你的需求，可以继承它创建一个新元素，来扩展 `<button>` 元素：

```
var MyButton = document.registerElement('my-button', {
  prototype: Object.create(HTMLButtonElement.prototype)
});
```

#### 扩展原生 API

```
var MyButtonProto = Object.create(HTMLButtonElement.prototype);

MyButtonProto.sayhello = function() {
  alert('hello');
};

var MyButton = document.registerElement('my-button', {
  prototype: MyButtonProto
});


var myButton = new MyButton();
document.body.appendChild(myButton);

myButton.sayhello(); // alert: "hello"
```

#### 实例化
使用 `new` 操作符：

```
var myButton = new MyButton();
myButton.innerHTML = 'click me!';
document.body.appendChild(myButton);
```

或，直接在页面插入元素：

```
<my-button>click me!</my-button>
```

#### 生命周期
元素可以定义特殊的方法，来注入其生存周期内的关键时间点。生命周期的回调函数名称和时间点对应关系如下：

- createdCallback: 创建元素实例时
- attachedCallback: 向文档插入实例时
- detachedCallback: 从文档移除实例时
- attributeChangedCallback(attrName, oldVal, newVal): 添加，移除，或修改一个属性时

```
var MyButtonProto = Object.create(HTMLButtonElement.prototype);

MyButtonProto.createdCallback = function() {
  this.innerHTML = 'Click Me!';
};

MyButtonProto.attachedCallback = function() {
  this.addEventListener('click', function(e) {
    alert('hello world');
  });
};

var MyButton = document.registerElement('my-button', {
  prototype: MyButtonProto
});

var myButton = new MyButton();
document.body.appendChild(myButton);
```

### Shadow DOM
Shadow DOM 是一个 HTML 的新规范，其允许开发者封装自己的 HTML 标签、CSS 样式和 JavaScript 代码。Shadow DOM 使得开发人员可以创建类似 `<input type="range">` 这样自定义的一级标签。

web 开发经典问题：封装。如何保护组件的样式不被外部 css 样式侵入，如何保护组件的 dom 结构不被页面的其他 javascript 脚本修改。大家都用过 Bootstrap，如果要使用其中的某些组件，例如 modal，通常会把组件的 DOM 结构复制过来。

```
<div class="modal fade">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
        <h4 class="modal-title">Modal title</h4>
      </div>
      <div class="modal-body">
        <p>One fine body&hellip;</p>
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
        <button type="button" class="btn btn-primary">Save changes</button>
      </div>
    </div><!-- /.modal-content -->
  </div><!-- /.modal-dialog -->
</div><!-- /.modal -->
```
这样一坨复制过来的代码，大多数时候并没有仔细了解，任何时候一个不小心都有可能覆盖了其中的一个 class 样式，这里面可能潜在很多小 bug。Shadow Dom 可以很好的解决组件封装问题。

#### 一个例子说明，什么是 Shadow DOM ？
浏览器渲染 `<input type="range">` 标签，显示结果如下：  

<input type="range">

看起来似乎很简单，只有一个 `input` 标签而已。页面代码如下：  
![](http://g02.a.alicdn.com/kf/HTB1sAa_IpXXXXaLXFXX760XFXXXu.png)  
但实际上是这样的：  
![](http://g02.a.alicdn.com/kf/HTB1laKYIpXXXXc8XVXX760XFXXXd.png)

显示 shadow dom 需要开启 Chrome 开发者工具的 'Show user agent shadow DOM'  
![](http://g02.a.alicdn.com/kf/HTB1r4LiIpXXXXXkXpXX760XFXXX4.png)

#### 创建 Shadow DOM
使用 `createShadowRoot` 创建影子根节点，其余的操作跟普通 DOM 操作没有太大区别。

```
<div class="widget">Hello, world!</div>  
<script>  
    var host = document.querySelector('.widget');
    var root = host.createShadowRoot();
    
    var header = document.createElement('h1');
    header.textContent = 'Hello, I am Shadow DOM.';

    var paragraph = document.createElement('p');
    paragraph.textContent = 'This is the content.';

    root.appendChild(header);
    root.appendChild(paragraph);
</script> 
```
宿主节点的原有内容 `Hello, world!` 不会被渲染，取而代之的是 shadow root 里的内容。

#### 使用 content 标签

```
<div class="widget">shadow dom</div>  
<template>
<h1>Hello, I am <content></content></h1>
</template>
<script>  
   var host = document.querySelector('.widget');
   var root = host.createShadowRoot();
    
	var	 template = document.querySelector('template').content;
	root.appendChild(document.importNode(template, true));
</script> 
```
使用 `<content>` 标签，我们创建了一个插入，其将 `.widget` 中的文本投射出来，使之得以在我们的影子节点 `<h1>` 中展示。上面的例子最终渲染成 `Hello, I am shadow dom`。

#### Shadow DOM 样式
Shadow DOM 和常规 DOM 之间存在一个边界，这个边界能防止常规 DOM 的样式泄露到 Shadow DOM 中来。

```
<style>
p.normal, p.shadow {
  color: red;
  font-size: 18px;
}
</style>
<p class="normal">我是一个普通文本</p>
<p class="shadow"></p>

<script>
	var host = document.querySelector('.shadow');
	var root = host.createShadowRoot();
	root.innerHTML = `
	<style>
	p { 
	  color: blue; 
	  font-size: 24px; 
	} 
	</style>
	<p>我是一个影子文本</p>`;
</script>
```

#### :host 选择器
通过 `:host` 选择器可以设置宿主元素的样式。

```
<style>
p {
  color: red;
  font-size: 18px;
}
</style>
<p class="normal">我是一个普通文本</p>
<p class="shadow"></p>

<script>
  var host = document.querySelector('.shadow');
  var root = host.createShadowRoot();
  root.innerHTML = `
  <style>
  :host(p.shadow) { 
    color: blue; 
    font-size: 24px; 
  } 
  </style>
  我是一个影子文本`;
</script>
```
注意上例中 shadow DOM 内的选择器是 `:host(p.shadow)`，而不是跟外部平级的 `:host(p)`。 因为`:host(p)` 的优先级低于外部的 `p` 选择器，所以不会生效。需要使用 `:host(p.shadow)` 提升优先级，才能将 `.shadow` 中的样式覆盖。

#### ::shadow 伪类选择器
有时你可能会想让使用者打破影子边界的壁垒，让他们能够给你的组件添加一些样式，使用 ::shadow 伪类选择器我们可以赋予用户重写我们默认定义的自由。

```
<style>
p span, 
p::shadow span {
  color: red;
  font-size: 18px;
}
</style>
<p class="normal"><span>我是一个普通文本</span></p>
<p class="shadow"></p>

<script>
  var host = document.querySelector('.shadow');
  var root = host.createShadowRoot();
  root.innerHTML = `
  <style>
  span { 
    color: blue; 
    font-size: 24px; 
  } 
  </style>
  <span>我是一个影子文本</span>`;
</script>
```



## 参考文献
- [Web Components 是什么？它为什么对我们这么重要？](http://www.html-js.com/article/2779)
- [Shadow DOM 综述](http://www.ituring.com.cn/article/179915)
- [Custom Elements](http://www.html5rocks.com/zh/tutorials/webcomponents/customelements/)