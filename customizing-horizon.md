# 如何定制 Horizon

本文译自 [openstack-horizon 官方文档](https://github.com/openstack/horizon/blob/master/doc/source/topics/customizing.rst)


## 主题

从 Kilo 版本开始，Horizon 支持通过主题来定制样式。主题内包含一个 ``_variables.scss`` 文件用来覆写颜色，还有一个 ``_styles.scss`` 文件用来添加样式，这两个文件在 dashboard 样式之后加载。

从 Mitaka 版本开始，Horizon 支持配置多个主题给用户选择，通过浏览器 cookie 记录用户设置的主题。默认情况下提供 'default' 和 'material' 两个标准主题。

主题配置在 ``local_settings.py`` 文件的 ``AVAILABLE_THEMES`` 变量中，是一个包含 ``('name', 'label', 'path')`` 的元组。

``name``
  保存在 cookie 中的主题名称

``label``
  展示在用户界面列表中的主题名称

``path``
  包含主题的文件夹路径，必须是相对于 ``openstack_dashboard`` 目录
  的相对路径或者操作系统的绝对路径

多主题的例子
```
  AVAILABLE_THEMES = [
      ('default', 'Default', 'themes/default'),
      ('material', 'Material', 'themes/material'),
  ]
```
单主题的例子
```
  AVAILABLE_THEMES = [
      ('default', 'Default', 'themes/default'),
  ]
```
Dashboard 自定义变量和 Bootstrap 变量都可以被覆写。在``openstack_dashboard/static/dashboard/scss/_variables.scss`` 这个文件查看所有的 SCSS 变量。

一个自定义主题必须包含 ``_variables.scss`` 和 ``_styles.scss`` 两个文件，其中``_variables.scss`` 必须包含所有的 Bootstrap 变量。


### 主题继承


自定义主题必须定义 ``_variables.scss`` and ``_styles.scss`` 中所有的 Bootstrap 变量。你可以从已有的 default 主题继承这些变量，只覆写你想要修改的内容，把以下代码放在自定义主题的  ``_variables.scss`` 文件中
```
   @import "/themes/default/variables";
```
修改了样式之后需要重新执行如下命令，重新生成静态文件 ``./run_tests.sh -m collectstatic``.

运行  `collectstatic` 命令时，在 ``AVAILABLE_THEMES`` 变量中的所有主题都会重新生成，更新到 `static/themes` 路径下，这个目标路径也可以通过 ``local_settings.py`` 中的``THEME_COLLECTION_DIR`` 变量来指定。

以下是一个引用 material 主题中定义的变量的例子
```
  @import "/themes/material/variables";
  @import "/themes/material/styles";
```

### 主题包的目录结构

根据自定义的程度不同，自定义主题的目录可以是不同的形式，类似 Django 的模板体系一样引用静态文件。可以包含 ``static``, ``templates`` 和 ``img`` 三个子目录。

#### ``static`` 目录

如果主题目录下包含 ``static`` 文件夹，则该文件夹会被当做 **主题的静态文件根目录**。例如，Horizon 会在该文件夹下查找 _variables.scss 和 _styles.scss 文件。The contents of this folder will also be served up at ``/static/custom``.

#### ``templates`` 目录

如果主题目录下包含 ``templates`` 文件夹，则该文件夹的路径会添加到 ``TEMPLATE_DIRS`` 元组的前面，这样主题就可以自定义模板。

#### 使用 ``templates`` 目录

在Horizon中，任何 Django 模板都可以被主题覆写，提供了高度可定制的能力。覆写的模板必须和被覆写的模板保持一样的目录结构。

例如，如果你要自定义 sidebar，必须把在 templates 目录下也新建一个 ``horizon/_sidebar.html`` 文件，这样就可以在 ``{ theme_path }/templates/horizon/_sidebar.html`` 引用中生效。

#### ``img`` 目录

如果主题的静态文件根目录下包含 ``omg`` 文件夹，则所有用 {% themable_asset %} 模板标签引入的图片都会被覆写，包括 logo.png, splash-logo.png and favicon.ico，但还不支持覆写 `dashboard/img` 下的被 Heat 组件使用的 SVG/GIF 文件。

### 自定义 Logo

#### 简单的

如果你想自定义启动界面或顶部导航栏的logo，你需要在主题的 static 根目录下创建一个 ``img`` 文件夹，将自定义的 ``logo.png`` 和 ``logo-splash.png`` 图片放在里面。

如果你的 ``logo.png`` 的高度比顶部导航栏的高度更大，图片会被压缩至导航栏的高度。你可以通过 SCSS 变量 ``$navbar-height`` 来定制顶部导航栏的高度。 如果图片的高度比顶部导航栏的高度小，那么会垂直居中显示。

Kilo 之前的版本，需要将 Horizon 中的图片替换成你自己的图片，或者把样式表中的图片指向你的图片路径。

#### 高级的

如果你想做更多的定制化，可以在主题根目录下添加 ``templates/header/_brand.html`` ，然后修改里面的内容。参考：``openstack_dashboard/themes/material/templates/header/_brand.html``

启动界面也可以定制，通过添加 ``templates/auth/_splash.html`` 文件实现。参考：``openstack_dashboard/themes/material/templates/auth/_splash.html``


## 设计 Horizon 的品牌风格

从 Liberty 版本发布以来，Horizon 严格遵守着 Bootstrap 设计规范，努力创造更好的响应式网页设计，也减轻未来每次版本改变风格的负担。

### 支持的组件

以下组件根据版本号排列，充分利用了 Bootstrap 的主题结构

#### 8.0.0 (Liberty)

* `Top Navbar`
* `Side Nav`
* `Pie Charts`

#### 9.0.0 (Mitaka)

* Tables
* Bar Charts
* Login
* Tabs
* Alerts
* Checkboxes

### 第一步

创建品牌化主题的第一步是创建自定义的 Bootstrap 主题。有很多辅助工具，其中包括：

- `Bootswatchr`
- `Paintstrap`
- `Bootstrap`


    Bootstrap 使用 LESS 但我们用 SCSS，以上的工具都会提供 
    ``variables.less`` 文件，需要手动转换为  ``_variables.scss`` 

### Top Navbar

现在的 Horizon 的顶部导航栏用 Bootstrap 原生的 ``navbar``。在   ``_variables.scss`` 文件的 **Navbar** 区块内查看哪些变量可以自定义。

导航栏用了原生的 Bootstrap dropdowns 组件，也可以自定义其中的变量值，参考  ``_variables.scss`` 文件中的 **Dropdowns** 区块进行自定义。

顶部导航栏可以自适应小屏幕。

### Side Nav

侧边栏组件也已经基于原生的 Stacked Pills 元素重构了，参考 ``_variables.scss`` 文件中的 **Pills** 区块进行自定义。

### Charts


#### Pie Charts

饼图由 SVG 元素构成，SVG 元素可以接受基础的 CSS 的自定义样式。(例如 colors, size)

Bootstrap 中没有饼图的原生元素，所以 Horizon 的图表样式是由主题样式定义。参考 ``_pie_charts.scss``


#### Bar Charts

柱状图可以是 Bootstrap Progress Bar 也可以是 SVG 元素，两种情况都使用了 Bootstrap Progress Bar 的样式。

SVG 实现的柱状图无法自定义高度，所以推荐使用基于 Bootstrap Progress Bar 实现的柱状图。

参考 ``_variables.scss`` 文件中的 **Progress bars** 区块进行自定义样式，SVG 版的在  ``_bar_charts.scss`` 里面自定义。

### Tables

标准的 Django 表格使用了原生的 Bootstrap table 标签，参考 ``_variables.scss`` 文件中的 **Tables** 区块进行自定义。

标准的 Bootstrap 表格是无边框的，如果想要添加边框，以 ``default`` 主题为例，参考 ``openstack_dashboard/themes/default/horizon/components/_tables.scss`` 文件。


### Login

#### Login Splash Page

登录页面使用了标准的 Bootstrap panel 的实现，参考 ``_variables.scss`` 文件中的 **Panels** 区块进行自定义。

#### Modal Login

登录弹窗使用了标准的 Bootstrap dialog，参考 ``_variables.scss`` 文件中的 **Modals** 区块进行自定义。

### Checkboxes

Horizon 使用 icon fonts 来实现 checkboxs，只需要覆写  standard scss 来实现自定义。例如 ``themes/material/static/horizon/components/_checkboxes.scss``。

### Bootswatch and Material Design

`Bootswatch`_ 是一系列免费的 Bootstrap 主题。Horizon 包含了另一个主题 ``material``，遵循 `Google's Material Design`_ 风格，基于 Bootswatch 的 **Paper** 主题。

Bootswatch 提供了一系列其他的主题，Horizon 是完全主题化的开发，可以很方便的切换主题、自定义主题。

### Development Tips

每次修改主题后，需要手动生成 `static` 目录里的内容，然后重启服务器。如果你不想每次都重启，可以按如下方式修改 ``local_settings.py`` 文件::

  COMPRESS_OFFLINE = False
  COMPRESS_ENABLED = False

## 修改站点标题

在 ``local_settings.py`` 文件添加  ``SITE_BRANDING`` 变量来自定义站点标题。

## 修改首页链接

在 ``local_settings.py`` 文件添加  ``SITE_BRANDING_LINK`` 变量来自定义站点首页链接。

## 自定义页脚

在主题目录下的 template 文件夹添加 ``_footer.html`` 以自定义全局页脚，添加 ``_login_footer.html`` 以自定义登录页页脚。

## 修改 Dashboards 和 Panels

你可以指定一个自定义的 python 模块作为 dashboard 或 panel，常见的站点定制需求如下：

* 从 dashboard 注册或注销 panels
* 修改 dashboard 和 panel 的名称
* 对 panel 进行重新排序


默认加载的 panel 在 openstack_dashboard/enabled/ 目录下，根据文件名顺序排序加载。文件名以 .example 后缀结尾的文件是一些示例。开发者和维护者最好也按照这种方式来组织，请不要胡乱覆写文件和打补丁。

## Icons


Horizon 使用 Font Awesome 的字体图标。参阅 `Font Awesome`_。
使用 icon 属性给表格添加 Action。例如
```
    class CreateSnapshot(tables.LinkAction):
        name = "snapshot"
        verbose_name = _("Create Snapshot")
        icon = "camera"
```
另外，全站的默认按钮样式修改，可以在 ``local_settings.py`` 文件中的 ``ACTION_CSS_CLASSES`` 中添加 class 类。


## 自定义样式


Horizon 可以自定义 dashboard 的样式，基础模板 ``openstack_dashboard/templates/base.html`` 中定义的 block 都可以被覆写。

创建一个 dashboard 的 templates 文件夹，从 Horizon 的基础模板继承，例如 ``openstack_dashboard/dashboards/my_custom_dashboard/templates/my_custom_dashboard/base.html``，然后就可以重新定义这个基础模板中的 block css。（别忘了引入 ``_stylesheets.html``，它包含了 Horizon 的所有默认样式 ）::

    {% extends 'base.html' %}

    {% block css %}
      {% include "_stylesheets.html" %}

      {% load compress %}
      {% compress css %}
      <link href='{{ STATIC_URL }}my_custom_dashboard/scss/my_custom_dashboard.scss' type='text/scss' media='screen' rel='stylesheet' />
      {% endcompress %}
    {% endblock %}


自定义的样式文件放在 dashboard 的 ``static`` 目录下 ``openstack_dashboard/dashboards/my_custom_dashboard/static/my_custom_dashboard/scss/my_custom_dashboard.scss``.

所有的 template 都必须继承自 dashboard 的基础模板
```
    {% extends 'my_custom_dashboard/base.html' %}
```


## 自定义 Javascript

页面所有的 js 文件都在 ``openstack_dashboard/templates/horizon/_scripts.html`` 模板中引入，这个模板在 base 模板的 ``block js`` 中被引用。


在你的 dashboard 中创建一个 ``openstack_dashboard/dashboards/my_custom_dashboard/
templates/my_custom_dashboard/_scripts.html`` 模板，继承自 ``horizon/_scripts.html``，在这个模板中覆写 ``block custom_js_files``，添加你自己的 javascript 文件
```
    {% extends 'horizon/_scripts.html' %}

    {% block custom_js_files %}
        <script src='{{ STATIC_URL }}my_custom_dashboard/js/my_custom_js.js' type='text/javascript' charset='utf-8'></script>
    {% endblock %}
```

在你自己的 dashboard 的基础模板 ``openstack_dashboard/dashboards/my_custom_dashboard/templates/my_custom_dashboard/base.html`` 中覆写 ``block js``，包含你自己的 ``_scripts.html``
```
    {% block js %}
        {% include "my_custom_dashboard/_scripts.html" %}
    {% endblock %}
```
输出结果是一个包含了 Horizon 和 dashboard 自定义脚本的压缩文件。

另外，有些分析采集脚本需要在 <head> 中加载，这种情况可以在 ``horizon/_custom_head_js.html`` 添加。像上文提到的 ``_scripts.html`` 做法一样，直接添加链接
```
    <script src='{{ STATIC_URL }}/my_custom_dashboard/js/my_marketing_js.js' type='text/javascript' charset='utf-8'></script>
```
也可以把脚本直接拷贝到模板里面::

  <script type="text/javascript">
  //some javascript
  </script>


## 自定义 Meta 属性

把你自定义的 Meta 添加到 ``horizon/_custom_meta.html`` 文件中，此文件的内容将会被插入到页面的 <head> 里。

## 相关链接
Bootswatch: http://bootswatch.com
Bootswatchr: http://bootswatchr.com/create#!
Paintstrap: http://paintstrap.com
Bootstrap: http://getbootstrap.com/customize/
Google's Material Design: https://www.google.com/design/spec/material-design/introduction.html
Font Awesome: https://fortawesome.github.io/Font-Awesome/
