---
title: 初探Deno
date: 2020-06-28 19:25:11
tags: [Deno]
---

## 背景介绍

Node.js 的创建者 Ryan Dahl 觉得 Node.js 已经背离了自己的初衷，因为他已经将 Node 托管给别人很久了，当他回来时，发现物是人非，因为有些众所周知且无法忽视的问题，Ryan Dahl 决定再写一个替代品，彻底解决这些问题。它就是 Deno（Destroy Node.js），顾名思义，推翻 Node。Deno 在 2020 年 5 月 14 日也正式发布了 1.0 版本，它的更新非常之快，扶我起来，我还能学。

<!--more-->

## 为啥 Ryan Dahl 觉得 node 背离了自己的初衷呢

顾名思义其实就是 Node 有很多缺陷呗，其实这些缺陷也众所周知

1. 异步接口有 Promise 和回调函数两种写法（让我想到了 bluebird）。
2. 自己的模块格式 CommonJS 与不支持 ES 模块不兼容，需要借助 babel。
3. 模块管理工具 npm，及安装目录极其复杂，难以管理（还有 package-lock.json，还经常容易冲突）。
4. 不安全，安装了模块就任由别人的代码在自己的服务里运行进行各种读写操作（经典的 Antd 彩蛋事件，还有各种库的 bug）。
5. 功能不完善，没有内置构建工具，没有内置测试模块，lint，format 等，都需要自己或借助三方库搞，然后又没什么标准，导致三方库横行，开发者要学太多，搞的疲惫不堪。
   等等

## Deno 是啥

Deno 是使用 V8 并内置于 Rust 的 JavaScript 和 TypeScript 和 WebAssembly 的简单的，现代的且安全的运行时。
它内置了 V8 引擎，用来解释 JavaScript。同时，也内置了 tsc 引擎，解释 TypeScript。它使用 Rust 语言开发，由于 Rust 原生支持 WebAssembly，所以它也能直接运行 WebAssembly。它的异步操作不使用 libuv 这个库，而是使用 Rust 语言的 Tokio 库，来实现事件循环（event loop）。

为啥不是用 C++呢？据说是因为 Rust 提供了很多现成的模块，对 Deno 项目来说，可以节约很多开发时间。毕竟大佬的时间也很宝贵，看得出来大佬是急切的想 destory node。

## Deno 有哪些特点呢

1. 支持 js，ts，WebAssembly。
   原生支持 ts，可以直接运行，不必显示转码，Deno 会根据后缀名来判断是用 v8 引擎，还是先用 tsc 编译器，查看 Deno 的 github 代码就会发现，它默认都是使用 ts 编写的。<https://github.com/denoland/deno>
2. 相对安全，默认没有文件、网络或环境的访问权限，除非设置允许

   ```javascript
    --allow-read：打开读权限，可以指定可读的目录，比如--allow-read=/temp。
    --allow-write：打开写权限。
    --allow-net：允许网络通信，可以指定可请求的域，比如--allow-net=google.com。
    --allow-env：允许读取环境变量。
   ```

3. 去 packages 化，就是没有 package.json 和 node_modules
4. 使用标准库
   <https://deno.land/std/>在这里可以看到 Deno 核心团队开发且都没有用外部依赖，还可以加上版本号，比如https://deno.land/std@v1.0.0。
5. 使用 ES 模块，而不支持 CommonJs。
   没有 npm 和 node_modules，所有模块通过 URL 加载。因此，Deno 不需要一个中心化的模块储存系统，可以从任何地方加载模块。并且必须指定加载脚本的后缀。但是，Deno 下载模块以后，依然会有一个总的目录，在本地缓存模块，因此可以离线使用。
   Deno 也为 ES 模块提供了公共托管服务，可以在<https://deno.land/x>中找到。
6. 直接支持 async/await 不需要 babel，polify 什么鬼的，凡是异步操作都会返回 promise
7. 内置工具比如：Testing、依赖检查、构建工具、lint 和代码格式化等

   ```javascript
    deno bundle：将脚本和依赖打包
    deno eval：执行代码
    deno fetch：将依赖抓取到本地
    deno fmt：代码的格式美化
    deno help：等同于-h参数
    deno info：显示本地的依赖缓存
    deno install：将脚本安装为可执行文件
    deno repl：进入 REPL 环境
    deno run：运行脚本
    deno test：运行测试
    deno lint：检查代码
   ```

8. 兼容浏览器 API，它提供 window 这个全局对象，同时支持 fetch、webCrypto、worker 等 Web 标准，也支持 onload、onunload、addEventListener 等事件操作函数
9. 单文件可执行文件，所有操作都通过这个文件完成

## 安装上手（mac）

其他环境安装方式 <https://deno.land/>
`brew install deno` 或者 `curl -fsSL https://deno.land/x/install/install.sh | sh`，但是后者需要配置环境变量什么的

直接上图
![avatar](/img/deno/1.png)
![avatar](/img/deno/2.png)

可以看到-V 是 0.26.0，而且 deno 的 help 还是很完善的 有介绍有例子有用法。
Deno 现行最新版本是 1.1.2 但是低版本还不支持 deno upgrade，咳咳，还是老老实实用 curl 重装吧。

![avatar](/img/deno/3.png)
龟速下载 需要翻墙

还可以下载可执行文件 <https://github.com/denoland/deno/releases/tag/v1.1.2>
![avatar](/img/deno/5.png)
![avatar](/img/deno/6.png)

终于装好了，提示需要配置环境变量
![avatar](/img/deno/7.png)
执行`deno -version`的话，不仅能看到 deno 的版本还可以看到 v8 和 tsc 的版本
![avatar](/img/deno/8.png)

执行官方例子 `deno run https://deno.land/std/examples/welcome.ts`
居然可以直接执行网络上的脚本
![avatar](/img/deno/9.png)
log 了个 🦕

## 搞几个 demo 学习一下基本操作

1. Hello Word

```javascript
const a: string = "Hello word";
console.log(a);
```

执行使用 run 指令
![avatar](/img/deno/10.png)
可以看到 ts 可以直接执行。

2. 引入一个官方标准包并起一个服务

```javascript
// 去远程下载相关依赖
import { serve } from "https://deno.land/std/http/server.ts";

const PORT = 8080;
const server = serve({ port: PORT });

console.log(` Listening on <http://localhost>:${PORT}/`);

for await (const req of server) {
  console.log(req);
  req.respond({ body: "Hello Deno" });
}
```

![avatar](/img/deno/11.png)

可以看到它回去远程下载依赖，并且在执行是报错，让打开网络权限，没错就是`--allow-net`，
加上它再次执行（`deno run --allow-net ./demo.ts`）可以看到服务创建成功，访问之后会得到响应。
![avatar](/img/deno/12.png)

但是有没有发现它这个代码写的很奇怪，经过 log 之后，我发现访问一次，输出一次 req，所以它应该是一个 pending 状态的 promise，当有访问时就会 resolve 一个 req。然后循环等待。
![avatar](/img/deno/13.png)

执行 deno info 命令可以看到 缓存目录 但是目前还不知道怎么清除缓存
![avatar](/img/deno/15.png)

3. 网络请求

```javasccript
// 支持浏览器api fetch
const url = Deno.args[0];
const res = await fetch(url);

console.log(res);
```

执行`deno run --allow-net=www.baidu.com ./demo.ts https://www.baidu.com`
可以看到请求结果，记得要开启网络权限哦
![avatar](/img/deno/14.png)

4. 读写操作

```javascript
// 根据路径 获取文件 并控制台输出
const filename = Deno.args[0];
const file = await Deno.open(filename);
await Deno.copy(file, Deno.stdout);
file.close();
```

执行`deno run --allow-read ./demo.ts ./demo.ts`
可以看到打开的文件内容被打印，记得要开启读文件权限，写权限就略过了
![avatar](/img/deno/16.png)

5. test

```javascript
Deno.test("demo", () => {
  throw Error("bad request");
});
```

![avatar](/img/deno/17.png)

6. lint

```javascript
const arr = new Array();
arr.push(1);
```

执行`deno lint --unstable ./demo.ts`
![avatar](/img/deno/18.png)

7. 打包

```javascript
const filename = Deno.args[0];
const file = await Deno.open(filename);
await Deno.copy(file, Deno.stdout);
file.close();
```

执行`deno bundle ./demo.ts`
可以看到它打包出来的代码
![avatar](/img/deno/19.png)

8. format
   format 前
   ![avatar](/img/deno/20.png)
   format 后
   ![avatar](/img/deno/21.png)

9. doc

```javascript
/**
 *
 * @param {string} a
 * @param {string} b
 * @returns {string} a+b
 */
export function demo(a: string, b: string): string {
  return a + b;
}
```

执行`deno doc ./demo.ts`
可以看到 deno 对代码中注释的解析
![avatar](/img/deno/22.png)

10. 发现

打印了 window 和 Deno 对象之后，可以发现的是 Deno 这个属性是挂在 window 下的，而 Deno 下又有很多方法和对象，比如对于文件的操作对比 Node 的 fs，listen 服务监听，test 之类的基础 api。

## Deno 真的能替换 Nodejs 吗

我简单熟悉使用之后，感觉 Deno 更加清爽，不需要过多的关注代码之外的事，比如 package。基础 api 也比较丰富，有些官方也有标准库。我认为它的方向思路功能构想是更加先进简洁的，是颠覆的，对开发者更加友好的，比 Node 具有更多的优势，但是 Deno 毕竟也才 1.0，最近更新的版本也大都是 fixbug，如下图，所以它目前的稳定性还是一个很大的问题。另外它还很年轻，像是一个初生儿，几乎还没什么生态，所以用的时候肯定会缺少这样那样的模块库，它还需要时间去打磨和考验，需要更多的开发者去关注，去发展它的生态，我相信不久将来 Deno 一定会在开发圈里占有一席之地，就像当年的 Node 一样。
![avatar](/img/deno/4.png)

## 参考引用

> <http://www.ruanyifeng.com/blog/2020/01/deno-intro.html> > <http://liguixing.com/archives/1381>
