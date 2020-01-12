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
