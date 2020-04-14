---
title: 搞懂CJS & UMD & ESM
date: 2020-02-23 19:20:22
tags: [CJS, UMD, ESM]
---

CommonJS & UMD & ES Module

最近在看 React.js，配置 webpack 的时候，注意到 react、react-router 等库，构建配置了多个不同的模块输出形式，分别是：CommonJS、UMD 以及 ES Module ，那么他们有什么区别呢，各自用在哪种场景里。

![avatar](/img/cjs/cjs.png)

<!--more-->

## CommonJS 用于后端的 node

Node 应用采用的模块规范，主要用于后端的 node。
每个文件就是一个模块，有自己的作用域。在一个文件里面定义的变量、函数、类，都是私有的，对其他文件不可见。

> CommonJS 特点：
> 所有代码都运行在模块作用域，不会污染全局作用域。
> 模块可以多次加载，但是只会在第一次加载时运行一次，然后运行结果就被缓存了，以后再加载，就直接读取缓存结果。要想让模块再次运行，必须清除缓存。
> 模块加载的顺序，按照其在代码中出现的顺序。
> 值得一提的是，CommonJS 规范加载模块是同步的，也就是说，只有加载完成，才能执行后面的操作。
> 通过 module.exports 暴露模块接口，通过 require 引入模块

```javascript
var express = require("express");
var router = require("./router");

var app = express();

module.exports = app;
```

## AMD/CMD

### AMD (Asynchronous Module Definition ，异步模块定义)

AMD 规范则是非同步加载模块，允许指定回调函数。由于 Node.js 主要用于服务器编程，模块文件一般都已经存在于本地硬盘，所以加载起来比较快，不用考虑非同步加载的方式，所以 CommonJS 规范比较适合。但是，如果是浏览器环境，要从服务器端加载模块，这时就必须采用非同步模式，因此浏览器端一般采用 AMD 规范。

AMD 使用 define 定义模块，使用 require 加载模块，特点：依赖前置，提前执行。
运行在浏览器端模块化开发的规范，使用该规范的有requireJs。

```javascript
// 引入模块加载器：
<script src="js/require.js" data-main="js/main"></script>

/** main.js 入口文件/主模块 **/
// 首先用config()指定各模块路径和引用名
require.config({
  baseUrl: "js/lib",
  paths: {
    "jquery": "jquery.min",  //实际路径为js/lib/jquery.min.js
    "underscore": "underscore.min",
  }
});
// 执行基本操作
require(["jquery","underscore"],function($,_){
  // some code here
});
---------------------
// 定义math.js模块 一个依赖underscore.js的模块 以下为math.js内容
define('名字 或 id', ['underscore', './xxx.js'],function(_, xxx){
  var classify = function(list){
    _.countBy(list,function(num){
      return num > 30 ? 'old' : 'young';
    })
  };
  return {
    classify :classify
  };
})
----------------------
// 引用模块，将模块放在[]内
// jquery 为require.config里的全局变量 ./math.js为路径
require(['jquery', './math'],function($, math){
  var sum = math.add(10,20);
  $("#sum").html(sum);
});

```

### CMD (Common Module Definition，通用模块定义)

CMD 使用define 来定义一个模块，使用require 来加载一个模块，特点：尽可能懒执行。
运行在浏览器端模块化开发的规范，使用该规范的有SeaJs

```javascript
// 引入模块加载器：
<script src="base/lib/sea.js" type="text/javascript"></script>
// 模块加载器配置：
seajs.config();
-------------------------
// 依赖并定义模块： xxx.js文件内容
// 一个文件里可以定义多个模块，需要有不同的ID
// 所有模块用 define 定义
define(
"模块ID", // 模块名
['依赖模块路径 或 ID'],
function(require, exports, module) {
  var otherModule = require(module.dependencies[0]);  // 导入模块
  var otherModule2 = require('./xx.js');  // 导入模块


  exports.xxx = 1;  // 导出模块
  module.exports = {
    xxx: 1,
  }; // 或导出模块
});
-------------------------
使用模块：
seajs.use('./xxx.js', function(模块实例){})
```

### AMD与CMD区别

>模块定义时对依赖的处理不同：
1.AMD推崇依赖前置，在定义模块的时候就要声明其依赖的模块
2.CMD推崇就近依赖，只有在用到某个模块的时候再去require
>同为异步加载模块的区别：
1.AMD在加载模块完成后就会执行该模块
2.CMD加载完某个依赖模块后并不执行，只是下载而已,这样模块的执行顺序和书写顺序是完全一致的。性能较好，只有用户需要时才执行

## UMD (Universal Module Definition，万能模块定义)

UMD 提供了支持多种风格的“通用”模式，在兼容 CommonJS 和 AMD 规范的同时，还兼容全局引用的方式

UMD 实现原理很简单：
先判断是否支持 AMD（define 是否存在），存在则使用 AMD 方式加载模块；
再判断是否支持 Node.js 模块格式（exports 是否存在），存在则使用 Node.js 模块格式；
前两个都不存在，则将模块公开到全局（window 或 global）
UMD 使得你可以直接使用`<script>`标签引用

```javascript
// if the module has no dependencies, the above pattern can be simplified to
(function(root, factory) {
  if (typeof define === 'function' && define.amd) {
    // AMD. Register as an anonymous module
    define([], factory);
  } else if (typeof exports === 'object') {
    // Node. Does not work with strtic CommonJS, but
    // only CommonJS-like environments that support module.exports,
    // like Node
    module.exports = factory();
  } else {
    // Browser globals (root is window)
    root.returnExports = factory();
  }

}(this, function() {

  // Just return a value to define the module export.
  // This example returns an object, but the module
  // can return a function as the exported value.
  return {};
}))
```

## ES Module

ECMAScript 6 的一个目标是解决作用域的问题，也为了使 JS 应用程序显得有序，于是引进了模块。目前部分主流浏览器已原生支持 ES Module，使用 type = module 指定为模块引入即可
注意：使用该方式执行 JS 时自动应用 defer 属性。

```javascript
// Default exports and named exports
import theDefault, { named1, named2 } from './lib';
// Renaming
import { named1 as myNamed1, named2 } from './lib';
// Importing the module as an object
// with one property per named export
import * as mylib from './lib';

// Only load the module, don't import anything
import from './lib';
------------------------
export const myVar1 = '';
export function myFunc(){};
------------------------
const MY_CONST = '';
function myFunc(){};

export {MY_CONST, myFunc};
export {MY_CONST as THE_CONST, myFunc};
export * from './lib';
export { foo as myFoo, bar } from './lib';
------------------------
export default function() {};
```

## ES Module 与 CommonJS 模块的差异

​1.CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
> CommonJS 模块输出的是值的拷贝，也就是说，一旦输出一个值，模块内部的变化就影响不到这个值。
ES6 模块的运行机制与 CommonJS 不一样。JS 引擎对脚本静态分析的时候，遇到模块加载命令import，就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里面去取值。换句话说，ES6 的import有点像 Unix 系统的“符号连接”，原始值变了，import加载的值也会跟着变。因此，ES6 模块是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块。

2.CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。
>运行时加载: CommonJS 模块就是对象；即在输入时是先加载整个模块，生成一个对象，然后再从这个对象上面读取方法，这种加载称为“运行时加载”。
  编译时加载: ES6 模块不是对象，而是通过 export 命令显式指定输出的代码，import时采用静态命令的形式。即在import时可以指定加载某个输出值，而不是加载整个模块，这种加载称为“编译时加载”。

CommonJS 加载的是一个对象（即module.exports属性），该对象只有在脚本运行完才会生成。而 ES6 模块不是对象，它的对外接口只是一种静态定义，在代码静态解析阶段就会生成

## webpack 配置

通过配置output.library及output.libraryTarget，可以配置如何暴露 你写的 library。
library 是你起的模块名字，libraryTarget是你要暴露模块的方式，值有

1.暴露为一个变量：var、assign；
2.通过在对象上赋值暴露：this、window、global、commonjs；
3.模块定义系统：commonjs2、amd、umd；
4.其他 Targets：jsonp；

> 从 webpack 3.1.0 开始，你可以将 library 指定为一个对象，用于给每个 target 起不同的名称：

```javascript
module.exports = {
  //...
  output: {
    library: {
      root: 'MyLibrary',
      amd: 'my-library',
      commonjs: 'my-common-library'
    },
    libraryTarget: 'umd'
  }
};
```

> 详细配置的说明请看 <https://webpack.docschina.org/configuration/output/#output-librarytarget>
