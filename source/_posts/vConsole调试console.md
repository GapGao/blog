---
layout: 移动端（微信等）使用
title: vConsole调试console
date: 2019-06-01 15:16:41
tags: [移动端, console, vconsole]
---

# 介绍
因为最近一直在弄移动端项目，由于在移动端无法打开控制台，所以想办法打印调试console的数据一直苦恼。之前用的是chrome的inspect调试，但是只能使用移动版的chrome查看数据，兼容不好，所以最近使用了vConsole 进行调试

<!--more-->

> 具体描述介绍啥的见[github](https://github.com/Tencent/vConsole "标题")

#使用
> npm install vconsole
> 使用webpack，然后js代码中

```javascript
import VConsole from 'vconsole/dist/vconsole.min.js'; //import vconsole
let vConsole = new VConsole(); // 初始化
```

> 或者找到这个模块下面的 dist/vconsole.min.js ，然后复制到自己的项目中

```html
<head>
    <script src="dist/vconsole.min.js"></script>
</head>
<!--建议在 `<head>` 中引入哦~ -->
<script>
  // 初始化
  var vConsole = new VConsole();
</script>
```

# 效果
## 页面效果
![avatar](/img/vconsole-ui.png)
## 点击后
![avatar](/img/vconsole-detail.png)

# 对应的webpack-plugin vconsole-webpack-plugin

## 安装
> npm install vconsole-webpack-plugin --save-dev

## 使用

```javascript
// 引入插件
var vConsolePlugin = require('vconsole-webpack-plugin');
module.exports = {
    ...
    plugins: [
        new vConsolePlugin({
            filter: [],  // 需要过滤的入口文件
            enable: true // 发布代码前记得改回 false
        }),
        ...
    ]
    ...
}
```
