# rasterizeHTML.js

<a href="https://www.npmjs.org/package/rasterizehtml">
    <img src="https://badge.fury.io/js/rasterizehtml.svg"
        align="right" alt="NPM version" height="18">
</a>

把 html 绘制到 canvas 上.

使用文档的[API](https://github.com/cburgmer/rasterizeHTML.js/wiki/API).

## 安装

    $ npm install rasterizehtml

通过脚本引入 `node_modules/rasterizehtml/dist/rasterizeHTML.allinone.js` 或者通过 [browserify](https://github.com/substack/node-browserify)加载.

## 例子

```javascript
var canvas = document.getElementById("canvas");
rasterizeHTML.drawHTML(
  "Some " +
    '<span style="color: green; font-size: 20px;">HTML</span>' +
    ' with an image <img src="someimg.png">',
  canvas
);
```

看 [这个例子](https://github.com/cburgmer/rasterizeHTML.js/wiki/Examples). 这个代码 [也有一些列子](https://github.com/cburgmer/rasterizeHTML.js/tree/master/examples), 确保首先执行了 `npm test` 去编译这个库.

## 怎么工作

由于安全性的原因，把 html 渲染到 canvas 上，有很多限制。Firefox 提供了一个`ctx.drawWindow`的函数方法。但也只有 Chrome 使用支持（可以看这个[文档](https://developer.mozilla.org/en/Drawing_Graphics_with_Canvas)）

寻找了一些其他的方法，在这两个文档中都有讲述：

- [Drawing DOM Content To Canvas](http://robert.ocallahan.org/2011/11/drawing-dom-content-to-canvas.html)
- [Drawing DOM objects into a canvas](https://reference.codeproject.com/book/dom/canvas_api/drawing_dom_objects_into_a_canvas)

它可以通过`<foreignObject>`把 html 包装成一个 SVG 图片，然后把结果的图片通过`ctx.drawImage()`绘制到 canvas 上.

另外，SVG 是不允许链接到外部的资源，如果 rasterizeHTML.js 需要加载额外的图片、文字、样式这些，那么就以内联的方式处理这些资源（比如图片的[data: URIs](http://en.wikipedia.org/wiki/Data_URI_scheme)，或者样式的内联）

## 限制

所有绘制页面所需要的资源（html 页面、css、images、fonts 和 js）都只能通过[同源](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Same_origin_policy_for_JavaScript)加载，除非使用[CORS](http://enable-cors.org)，`drawURL`只能加载来自同域的页面作为当前页面，所有的绘制方法同样地也只能签到来自该域的样式和图像

代码已经在 Firforx、Chrome 以及 Safari 中测试完成。

基本支持微软的 Edge，然而[不支持任何的 IE 版本](https://github.com/cburgmer/rasterizeHTML.js/wiki/Limitations#ie)

当把 html 包装成 SVG 渲染到 canvas 上的时候，对于不同的浏览器也有一些独特的问题。

[这儿有所有浏览器不同的限制](https://github.com/cburgmer/rasterizeHTML.js/wiki/Limitations).

## TypeScript

引入类型如下:

```ts
import * as rasterizeHTML from "rasterizehtml";
```

## 开发

运行 `npm test`. 安装需要使用的依赖.

在不同的浏览器中打开`test/SpecRunner.html`做测试, 为了一样的结果，需要在 Safari 中打开 `test/manualIntegrationTestForWebkit.html` (在 Chrome 中要么打开`--allow-file-access-from-files` 选项，要么启动本地 server 打开页面).

[![Build Status](https://travis-ci.org/cburgmer/rasterizeHTML.js.svg?branch=master)](https://travis-ci.org/cburgmer/rasterizeHTML.js)

## 哪儿被用了?

- [CSS Critic](https://github.com/cburgmer/csscritic),一个轻量的瀑布流的样式库的回归测试
- ...

## API

- 绘制一个页面到 canvas 上

```js
rasterizeHTML.drawURL( url [, canvas] [, options] )
```

- 绘制一个 html 字符串到 canvas 上

```js
rasterizeHTML.drawHTML( html [, canvas] [, options] )
```

- 绘制一个 document 文档到 canvas 上

```js
rasterizeHTML.drawDocument( document [, canvas] [, options] )
```

#### 参数

- 一个 URL
- 一个 html 的字符串
- document 的文档对象

#### 参数配置选项

- canvas 表示需要绘制到哪个的 canvas 节点
- options 对象属性：
  - width 视图的宽度，默认是 300
  - height 视图的高度，默认是 200
  - baseUrl html 文档中使用的资源基于这个链接来处理，默认为 null
  - executeJs 如果设置成了 true，在绘制页面，挥着绘制 html 字符串的时候，他就会执行，然后等待 onload 事件之后，再绘制内容（绘制 document 文档不支持），默认为 false
  - executeJsTimeout 将会在给定的时间之后，再停止执行 js，如果没有 js 执行，也会等待，默认是 0
  - zoom 一个缩放展示内容的配置，[默认是 1](https://github.com/cburgmer/rasterizeHTML.js/wiki/Limitations)
  - hover 该选择器会匹配一个模拟鼠标悬停的效果,默认为 null
  - active 该选择器会匹配一个模拟鼠标激活的状态，默认为 null
  - focus 该选择器会匹配一个模拟鼠标聚焦的状态，默认为 null
  - target 该选择器会匹配一个模拟数据锚点点击的状态，默认为 null
  - cache 允许微调缓存行为：
    - 'none' 通过在每一个请求上加戳"?\_=[timestamp]"来让请求不会被缓存
    - 'repeated' 强制对初始调用进行非缓存加载，同时允许浏览器对于相同的 URL 使用缓存
    - 'all' 不会使用任何缓存清除功能（默认）
- cacheBucket 一个对象保存库自己本身的内存缓存，只有在多次调用 API 并且 cache 设置了'none'以外的值，初始化是{}。

#### 返回值

所有的方法都返回一个 promise，只要这个内容被渲染了，它就是 fulfilled，如果渲染失败了，他就是返回 rejected 状态。它的使用模式基本如下：

```javascript
rasterizeHTML.drawURL(url, canvas, options)
    .then(function success(renderResult) {
         ...
    }, function error(e) {
        ...
    });
```

#### 成功回调

渲染成功之后的返回结果对象包含：

- image 被渲染到 canvas 上的结果的图片，如果内容超出了给定的视图大小，那么图像将会有更大的尺寸
- svg 呈现内容内部的 svg 表现形式
- errors 下载失败的资源列表

#### 失败回调

在 error 函数接受到一个 e 的对象，由如下组成：

```js
{
  message: "THE_MESSAGE",
  originalError: obj
}
```

这里的 message 可能会有其他几种情况：

- "Unable to load page" 如果给定的 URL 不能通过 drawURL 绘制
- "Error rendering page" 如果整个 document 渲染失败，就会出现这个错误
- "Invalid source" 如果这个资源是无效的（更具体地说，不能转换为呈现 HTML 所需的中间 XHTML 格式）

#### 资源失败

这个资源失败的的列表出错结构如下：

```js
{
  resourceType: "TYPE_OF_RESOURCE",
  url: "THE_FAILED_URL",
  msg: "A_HUMAN_READABLE_MSG"
}
```

资源类型：

- 对于图片—— `<img href="">`或者`<input type="image" src="">`
- 对于样式—— `<link rel="stylesheet" href="">`或者`@import url("");`
- 对于背景图—— `background-image: url("");`
- 对于字体文件—— `@font-face { src: url("") }`
- 对于脚本—— `<script src="">`
- 脚本执行的过程的错误信息

## 不同浏览器中的一些问题

#### Firfox

- XMLSerializer 的 HTML 转换功能有限，所以使用了自己[实现的](http://cburgmer.github.io/xmlserializer/)
- 经由 XHR2 加载和解析的 HTML [文档不能和本地文件工作](https://bugzilla.mozilla.org/show_bug.cgi?id=942138)
- 通过 XHR2 加载的文件，[不能够通过 CSSOM 访问到样式表](https://bugzilla.mozilla.org/show_bug.cgi?id=925493)

#### Webkit 的一些原因

- margin 塌陷通过`<foreignObject>`([Safari 的情况](https://bugs.webkit.org/show_bug.cgi?id=23963))有不同的实现
- XMLSerializer 接口不需要将 HTML 文档序列化为 XHTML，[因此 Webkit 不支持此功能](https://bugs.webkit.org/show_bug.cgi?id=47768)，就使用自己实现的序列化功能

#### Chrome

- [通过 Blob URL 绘制 SVG 后从画布读取失败](https://code.google.com/p/chromium/issues/detail?id=294129),可以回退到使用 Data:uri

#### Safari

- `<style>`标签在 SVG 中在 Safari 里面，[它的 sheet 属性是 undefined](https://bugs.webkit.org/show_bug.cgi?id=163324),基于 hack 的方式修复了这个[bug](https://github.com/cburgmer/rasterizeHTML.js/issues/158)

## 限制

对于 rasterizaHTML.js 使用的限制的原因是由于不同浏览器具体实现上的一些 bug。

### 一般限制

- 所有需要在绘制的时候使用到的资源（html 页面、css、images、fonts 和 js）要么是来自同源的，要么是用过 CORS 来请求的
- 不会加载额外通过 js 来创建的资源（比如：懒加载图片）。如果呀使用这些资源，同意通过内联的方式来处理
- 配置项：hover、active、focus 只能够用于用户特指的伪类样式，而不是用户代理自己配置的。比如在 chorme 中的 button 边框或者在 Firefox 处于活跃状态的时候的左内边距的距离
- 操作 ScrollTop 或者类似的属性将不会影响可视化区域的位置

### 当前不支持的 html

- canvas 的内容和实际大小都不能都不能正确的渲染
- 不支持 iframes
- 通过内联样式定义的外部的背景图片也不能渲染（背景图内联处理）
- 通过 srcset 定义的图片，不能被渲染
- 通过 media 查询定义的 link 标签会被忽略 `<link rel="stylesheet" media="..." />`
- 通过 media 查询动态 import 进来的规则会被忽略`@import url("...") screen and (orientation:landscape);`

### 浏览器的问题

#### 所有浏览器

- 设置在 HTML 或者 BODY 元素上的背景图是应该铺满整个 canvas，但是目前只是实现了文档实际自身的大小，[没有铺满全部](https://bugzilla.mozilla.org/show_bug.cgi?id=1143405)。

#### Webkit origin

- zoom 配置选项不能够正常的生成[叠层上下文](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Understanding_z_index/The_stacking_context)，比如由于一些[bug](https://bugs.chromium.org/p/chromium/issues/detail?id=467308),造成一些属性错位（position、transform、opacity）
- [复合变换不能够正确的渲染](https://github.com/cburgmer/rasterizeHTML.js/issues/85)
- Chrome＆Safari 使用小写的标记名称选择器，可匹配任何独立于字母大小写的元素。但是，Firefox 仅对 HTML 元素不区分大小写。

#### Safari

- webkit 不能获取 canvas 上的内容
- 在一些不清楚的情况下，不会加载图像（img 和 background-img）,预加载相关图像可能会有帮助，比如，[将背景图像作为高度 0 的 div](https://github.com/cburgmer/rasterizeHTML.js/issues/81)

#### IE

- 由于缺少对 SVG 的`<forrignObject>`支持，因此，基本上没有支持
