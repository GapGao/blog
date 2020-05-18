---
title: webpack-rollup-parcel
date: 2020-05-17 20:12:25
tags: [webpack, rollup, parcel, 打包]
---

webpack rollup parcel 应该用哪个？

<!--more-->

## 介绍

### webpack

webpack  是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个  bundle。为处理资源管理和分割代码而生，可以包含任何类型的文件。灵活，插件多。

### rollup

Rollup 是一个 JavaScript 模块打包器，可以将小块代码编译成大块复杂的代码，例如 library 或应用程序。用标准化的格式（es6）来写代码，通过减少死代码尽可能地缩小包体积。

### parcel

Parcel 是 Web 应用打包工具，适用于经验不同的开发者。它利用多核处理提供了极快的速度，并且不需要任何配置。超快的打包速度，多线程在多核上并发编译，不用任何配置。

## 配置对比

webpack 和 rollup 都需要配 config 文件，指明 entry, output, plugin，transformations。二者的细微区别在于： rollup 有对 import/export 所做的 node polyfills，webpack 没有  rollup 支持相对路径，而 webpack 没有，所以得使用 path.resolve/path.join。
parcel 则是完全开箱可用的，不用配置。

## 入口文件对比

webpack 只支持 js 文件作为入口文件，如果要以其他格式的文件作为入口，比如 html 文件为入口，如要加第三方 Plugin。
rollup 可以用 html 作为入口文件，但也需要 plugin，比如 rollup-plugin-html-entry。
parcel 可以用 index.html 作为入口文件，而且它会通过看 index.html 的 script tag 里包含的什么自己找到要打包生成哪些 js 文件。

## transformations 对比

transformations 指的是把其他文件转化成 js 文件的过程，需要经过 transformation 才能够被打包。
webpack 使用 Loaders 来处理。
rollup 使用 plugins 来处理。
parcel 会自动去转换，当找到配置文件比如.babelrc, .postcssrc 后就会自动转。

## tree-shaking 对比

摇树优化是 webpack 的一大特性。需要 1，用 import/export 语法，2，在 package.json 中加副作用的入口，3，加上支持去除死代码的缩小器（uglifyjsplugin）。
rollup 会统计引入的代码并排除掉那些没有被用到的。这使得可以在现有工具和模块的基础上构建，而不会添加额外的依赖项或膨胀项目的大小。
parcel 不支持摇树优化。

## dev server 对比

webpack 用 webpack-dev-server。
rollup 用 rollup-plugin-serve 和 rollup-plugin-livereload 共同作用。
parcel 内置的有 dev server。

## 热更新对比

webpack 的 wepack-dev-server 支持 hot 模式。
rollup 不支持 hmr。
parcel 有内置的 hmr。

## 代码分割对比

webpack 通过在 entry 中手动设置，使用 CommonsChunkPlugin，和模块内的内联函数动态引入来做代码分割。
rollup 有实验性的代码分割特性。它是用 es 模块在浏览器中的模块加载机制本身来分割代码的。需要把 experimentalCodeSplitting 和 experimentalDynamicImport 设为 true。
parcel 支持 0 配置的代码分割。主要是通过动态 improt。

## DEV 打包时间对比

|         | DEV BUILD TIME ONE | DEV BUILD TIME TWO | DEV BUILD TIME THREE | DEV BUILD TIME AVG |
| ------- | ------------------ | ------------------ | -------------------- | ------------------ |
| Webpack | 1.249 S            | 1.126 S            | 1.218 S              | 1.197 AVG S        |
| Rollup  | 0.618 S            | 0.570 S            | 0.583 S              | 0.590 AVG S        |
| Parcel  | 8.61 S             | 1.68 S             | 1.74 S               | 4.01 AVG S         |

## PROD 打包时间对比

|         | PROD BUILD TIME ONE | PROD BUILD TIME TWO | PROD BUILD TIME THREE | PROD BUILD TIME AVG |
| ------- | ------------------- | ------------------- | --------------------- | ------------------- |
| Webpack | 1.551 S             | 1.149 S             | 1.226 S               | 1.308 AVG S         |
| Rollup  | 0.785 S             | 0.739 S             | 0.754 S               | 0.759 AVG S         |
| Parcel  | 6.60 S              | 0.456 S             | 0.422 S               | 2.492 AVG S         |

## 结论

首次运行后，Parcel 的缓存功能大大减少了时间消耗。对于较小的频繁发生小改动的项目，Parcel 是一个不错的选择。
Rollup 提供了比 Webpack4 更简单的配置，并且具有许多预配置的插件，可以轻松地将它们集成到项目中。Rollup 也是构建工具阶段中最快的。
Rollup 还提供了方便的 sourcemap，可以帮助调试。
webpack 4 的使用变得更加容易，尤其是通过便捷的 mode 属性（现在设置为“production”时将强制最小化）。

## 总结

Webpack 适⽤于⼤型复杂的前端站点构建: webpack 有强⼤的 loader 和插件⽣态,打包后的⽂件实际上就是⼀个⽴即执⾏函数，这个⽴即执⾏函数接收⼀个参数，这个参数是模块对象，键为各个模块的路径，值为模块内容。⽴即执行函数内部则处理模块之间的引⽤，执⾏模块等,这种情况更适合⽂件依赖复杂的应⽤开发.
Rollup 适⽤于基础库的打包，如 vue、d3 等: Rollup 就是将各个模块打包进⼀个⽂件中，并且通过 Tree-shaking 来删除⽆⽤的代码,可以最⼤程度上降低代码体积,但是 rollup 没有 Webpack 如此多的的如代码分割、按需加载等⾼级功能，其更聚焦于库的打包，因此更适合库的开发.（Rollup 已被许多主流的 JavaScript 库使用，也可用于构建绝大多数应用程序。但是 Rollup 还不支持一些特定的高级功能，尤其是用在构建一些应用程序的时候，特别是代码拆分和运行时态的动态导入  dynamic imports at runtime. 如果你的项目中更需要这些功能，那使用  Webpack 可能更符合你的需求。）
Parcel 适⽤于简单的实验性项⽬: 他可以满⾜低⻔槛的快速看到效果,但是⽣态差、报错信息不够全⾯都是他的硬 伤，除了⼀些玩具项⽬或者实验项⽬不建议使⽤
