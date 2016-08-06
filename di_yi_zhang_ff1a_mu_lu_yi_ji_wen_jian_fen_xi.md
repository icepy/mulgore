# 第一章：目录以及文件分析

> 混沌初开，从目录开始

先从下载源码开始 `git@github.com:facebook/react.git` ，然后找个编辑器去打开这个项目吧。第一眼，请不要被这个项目的目录结构吓坏了，其实你只需要阅读 `src` 目录就好。先从第一个文件 `ReactVersion.js` 开始吧，顾名思义它只有一个版本号的定义。

```JavaScript
'use strict';

module.exports = '15.1.0';
```

## React 是如何构建的

很不幸它使用了 `Grunt`，`Gulp`，`browserify` 以及 `npm scripts` 来构建 React，不过不要紧这些不是重点。如果，你有稍微看了一下 `React` 的源码，你应该对 `require('ReactComponent');` 会有印象，首先你要去查询的是这些别名都指向了哪一个文件。



