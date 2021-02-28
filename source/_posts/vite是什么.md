---
title: vite是什么
date: 2021-02-28 20:14:31
tags: [vue, vite]
---

很久很久没写博客了，今天拾起来，干它一篇。最近也有很多不错的项目开源出来，今天我就来研究一下，最近很火的，尤大大刚发布的 vite。

<!--more-->

## 介绍

尤大大原话是：Vite，一个基于浏览器原生 ES imports 的开发服务器。利用浏览器去解析 imports，在服务器端按需编译返回，完全跳过了打包这个概念，服务器随起随用。同时不仅有 Vue 文件支持，还搞定了热更新，而且热更新的速度不会随着模块增多而变慢。针对生产环境则可以把同一份代码用 rollup 打包。虽然现在还比较粗糙，但这个方向我觉得是有潜力的，做得好可以彻底解决改一行代码等半天热更新的问题。
被称之为：Next Generation Frontend Tooling。

## ES imports

可以在 script 标签中 设置 type="module"参数，而引入 es modules，es modules 就是我们经常在项目中使用的 es modules,这种 es modules，在支持 es6 的浏览器中是可以直接使用的。
主流的 Edge, Chrome, Safari, and Firefox (+60)等浏览器都已经开始支持 es modules。

例如：

```html
<script type="module">
  import Vue from "https://cdn.jsdelivr.net/npm/vue@2.6.12/dist/vue.esm.browser.js";
  new Vue({
    el: "#container",
    data: {
      name: "Bob",
    },
  });
</script>
```

我们也可以通过 script 标签来直接引入相对路径或者绝对路径的模块。
如：

```html
<script type="module" src="/index.js"></script>
```

```javascript
//index.js
import Vue from "https://cdn.jsdelivr.net/npm/vue@2.6.12/dist/vue.esm.browser.js";
new Vue({
  el: "#container",
  data: { name: "Bob" },
});
```

如果浏览器直接引入 esmodules 的话，dev 环境下可以直接省略掉 打包（把所有用到的依赖或代码文件打包至一个或几个文件中） 部分，只需要`按需`，对需要的文件进行一些必要的 babel 编译拼接，即可使用，省去一大部分时间。

传统的 webpack 之类的打包工具，会将各模块提前打包进 bundle 里，但打包的过程是静态的——不管某个模块的代码是否执行到，这个模块都要打包到 bundle 里，这样的坏处就是随着项目越来越大打包后的 bundle 也越来越大，启动时间越来越长，就像我司项目，冷启动一次要 5 分钟，难受。

故而 vite 的特点是
快速的冷启动：不需要等待打包操作；
即时的热模块更新：替换性能和模块数量的解耦让更新飞起->快如闪电的 HMR；
真正的按需编译：不再等待整个应用编译完成，这是一个巨大的改变。

vite 功能实现
提供 web server：借用了 koa 来启动服务
模块解析：核心是拦截浏览器对模块的请求
支持 /@module/ ：判断路径是否以 /@module/ 开头，如果是取出包名，去 node_module 里找到这个库，基于 package.json 返回对应的内容
文件编译：拦截了对模块的请求并执行实时编译

## 安装使用

Vite 需要 Node.js 版本> = 12.0.0

```shell
npm init @vitejs/app
或者
yarn create @vitejs/app
```

![avatar](/img/vite/1.png)

执行后可以看到，可以选择 vue,vue-ts,react,react-ts,preact,preact-ts,lit-element,lit-element-ts 选项，我们这里选择了 react，因为比较熟悉，咳咳。

```shell
npm i
&&
npx vite
```

项目起来了

![avatar](/img/vite/2.png)

## index.html

index.html 是应用程序的入口

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="src/favicon.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite App</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

```jsx
import React from "react";
import ReactDOM from "react-dom";
import "./index.css";
import App from "./App";

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById("root")
);
```

可以看到 它引入了 路径为 `/src/main.jsx`的 esmodule。并且，项目根目录，即为静态资源根目录。引的居然直接是 jsx？？？我们在浏览器里看看，他输出的到底是什么代码。

## 输出

![avatar](/img/vite/3.png)

图中可以看到，vie 输出的 html，是在 index.html 模板基础上，注入了一些 script 的引入和执行，

```html
<script type="module" src="/@vite/client"></script>
<script type="module">
  import RefreshRuntime from "/@react-refresh";
  RefreshRuntime.injectIntoGlobalHook(window);
  window.$RefreshReg$ = () => {};
  window.$RefreshSig$ = () => (type) => type;
  window.__vite_plugin_react_preamble_installed__ = true;
</script>
```

引入了 node_modules 里的 vite 运行依赖，和 react-refresh，可能是热更新 plugin。

![avatar](/img/vite/4.png)
此图中可以看到`/src/main.jsx`的代码已经被`preset-react`处理过，害，我还以为直接输出了 jsx 呢，看了一眼，`content-type` 依然是 `application/javascript`，只要`content-type`对就行，后缀是啥无所谓。
紧接着又加载了 vite 依赖，`react`和`react-dom`，App.jsx 等，还真是解析到一个现加载一个。这样的话，就不要把所有代码都打包到一个文件里，最终一起引入，而是单个文件按需引入，解析到就发起请求，没解析到就不要加载，虽然可能会造成过多的请求，但是 dev 嘛，么得问题的，速度肯定刚刚的，毕竟按需引入是前端很重要的一个性能优化方向。并且服务器端拦截请求后，再去针对单个文件进行一些 babel 编译之类的工作，再返回给前端，真正的是按需引入和按需编译，毕竟每个文件都很小，编译速度极快。服务秒启动。

![avatar](/img/vite/5.png)

连 svg 都解析成了 esmodule，输出了引用路径。

![avatar](/img/vite/6.png)

这张图可以看到，根据 Referer 字段可以看出，每个请求都是由，哪个脚本发出去的，清晰明了。

对于 App.css 的请求

![avatar](/img/vite/9.png)

从图中可以看到，App.css 其实是被编译成了 js 脚本，把 style 编译成 string，通过 js，即 vite 提供的`updateStyle`方法，push 到 html 中，并给定 id。这样可以在热更新的时候，指定 id，删除 style 标签，重新生成新的 style 标签。

## hmr

首先 hmr 跟服务端的连接无外乎轮询和 ws，果然有一个 ws 连接，没 30s 发起一次心跳。
在修改 `Main.jsx` 之后，ws 收到一个 type 为 update 的消息推送，数据中说明了是 js 代码更新，并且标明了文件路径即请求路径

![avatar](/img/vite/7.png)

字段就不解释了，不言而喻。
而后，浏览器发起了一个 `http://localhost:3000/src/App.jsx?import&t=1614526800150`的请求，t 就是刚才的时间戳，重新请求`/src/App.jsx`。

对比了两次请求的代码之后，除了增加的一段 jsx 以外，好像并没有发现什么太多不一样的地方，同时还发现了之前没注意的

![avatar](/img/vite/7.png)

原来在转换为 `React.createElement`时，会加上代码行数`lineNumber`和列数`columnNumber`的参数.

另外可以看到代码中有很多 `import.meta.hot.accept`之类的语句，应该都是用来做热更新的，比如重复加载脚本之后执行回调，重新渲染之类的。下次再研究它的热更新实现。

## vite 如何处理 ESM 参考阿里巴巴淘系技术博客

既然 vite 使用 ESM 在浏览器里使用模块，那么这一步究竟是怎么做的？上文提到过，在浏览器里使用 ES module 是使用 http 请求拿到模块，所以 vite 必须提供一个 web server 去代理这些模块，上文中提到的 koa 就是负责这个事情，vite 通过对请求路径的劫持获取资源的内容返回给浏览器，不过 vite 对于模块导入做了特殊处理。

## @modules 是什么？参考阿里巴巴淘系技术博客

通过工程下的 index.html 和开发环境下的 html 源文件对比，发现 script 标签里的内容发生了改变，

```html
// 由
<script type="module">
  import { createApp } from "vue";
  import App from "/App.vue";
  createApp(App).mount("#app");
</script>
//变成了
<script type="module">
  import { createApp } from "/@modules/vue";
  import App from "/App.vue";
  createApp(App).mount("#app");
</script>

在 koa 中间件里获取请求 body 通过 es-module-lexer 解析资源 ast 拿到 import
的内容 判断 import 的资源是否是绝对路径，绝对视为 npm 模块
返回处理后的资源路径："vue" => "/@modules/vue"
```

那为啥需要 @modules？如果我们在模块里写下以下代码的时候，浏览器中的 esm 是不可能获取到导入的模块内容的，所以 vite 做了中间代理，在下一个 koa middleware 中，用正则匹配到路径上带有 @modules 的资源，再通过 require('xxx') 拿到 包的导出返回给浏览器。

**不过我想说的不是这些，而是**
目前最新版本

```html
<div id="root"></div>
<script type="module" src="/src/main.jsx"></script>
<script type="module">
  import Vue from "vue";
  console.log(Vue);
</script>
这端代码已经被编译成了

<div id="root"></div>
<script type="module" src="/src/main.jsx"></script>
<script type="module" src="/index.html?html-proxy&index=1.js"></script>
```

而 /index.html?html-proxy&index=1.js 是

```javascript
import Vue from "/node_modules/.vite/vue.js?v=6a7cef80";
console.log(Vue);
```

看来最新版已经已经对 html 中的引用的 node_modules 里的依赖，做了处理，让我们可以正常的像以往一样引用依赖，然后 vite，帮助我们处理请求，舒服。

![avatar](/img/vite/10.png)

并且对于引用过的 node_modules 里的依赖，都会被 vite 进行压缩处理，放到 node_modules/.vite 下进行缓存，后续无需再处理，直接使用，提高了加载速度。

并且还看到了 esbuild，总之就是，极致的效率体验~

## 结语

下次再研究一下它的热更新，以及是否能应用到我们的项目中或者参考它，提升一下我司项目的开发体验，因为它真的是太慢了。。。
虽然在短时间内 vite 不会替代 webpack，但是能够看到社区中多了一种方案还是很兴奋的。也佩服尤大大的勤奋和开源精神。希望他加油，赶紧更新出更加完备的 vite，我急用。。。

## 引用

> <https://vitejs.dev/>
> <https://www.zhihu.com/question/394062839/answer/1496127786>
