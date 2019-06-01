---
title: umi实战笔记
date: 2019-05-09 21:42:45
categories: web
tags: [react, umi, antd]
---

# 介绍
umi 是蚂蚁金服的底层前端框架
umi，中文可发音为乌米，是一个可插拔的企业级 react 应用框架。umi 以路由为基础的，支持类 next.js 的约定式路由，以及各种进阶的路由功能，并以此进行功能扩展，比如支持路由级的按需加载。然后配以完善的插件体系，覆盖从源码到构建产物的每个生命周期，支持各种功能扩展和业务需求，目前内外部加起来已有 50+ 的插件。

<!--more-->

# 特性
📦 开箱即用，内置 react、react-router 等
🏈 类 next.js 且功能完备的路由约定，同时支持配置的路由方式
🎉 完善的插件体系，覆盖从源码到构建产物的每个生命周期
🚀 高性能，通过插件支持 PWA、以路由为单元的 code splitting 等
💈 支持静态页面导出，适配各种环境，比如中台业务、无线业务、egg、支付宝钱包、云凤蝶等
🚄 开发启动快，支持一键开启 dll 和 hard-source-webpack-plugin 等
🐠 一键兼容到 IE9，基于 umi-plugin-polyfills
🍁 完善的 TypeScript 支持，包括 d.ts 定义和 umi test
🌴 与 dva 数据流的深入融合，支持 duck directory、model 的自动加载、code splitting 等等
后边的不复制了 -> <https://umijs.org/zh/guide> umi官方文档

# 安装
> 1.安装node 8.1x
> 2.yarn global add umi 或者 npm i umi -g 推荐使用yarn 本文后续都使用yarn进行管理

# 项目搭建
> 1.mkdir myapp && cd myapp
> 2.使用脚手架搭建项目
yarn create umi 使用create-umi脚手架来快速创建项目代码

```bash
? Select the boilerplate type (Use arrow keys)
  ant-design-pro  - 包含 antd-pro布局的脚手架
❯ app             - 通用项目脚手架
  block           - Create a umi block.
  library         - 区块脚手架，页面级别的可复用的代码，可以实现页面的按需加载，目前暂不清楚区块和插 件的区别
  plugin          - umi插件框架

  Do you want to use typescript? (y/N) 本博主暂时不会 后面会学习使用 选N
  (然后选择需要的功能,这里列出的都是umi的默认插件，根据需要选择)

? What functionality do you want to enable?
  (Press <space> to select, <a> to toggle all, <i> to invert selection)
❯ ◯ antd                  - antd组件库 选中
  ◯ dva                   - dva 首先是一个基于 redux 和 redux-saga 的数据流方案，
                            然后为了简化开发体验，dva 还额外内置了 react-router 和 fetch，所以也可以理解为一个
                            轻量级的应用框架。 选中
  ◯ code splitting        - webpack 实现按需加载的
  ◯ dll                   - 通过 webpack 的 dll 插件预打包一份 dll 文件来达到二次启动提速的目的。
```

> 4.这里为了快速熟悉umi先用antd(ant-design-pro)布局的搭建一个小项目
>> 1.创建好之后 执行 yarn 安装依赖
>> 文件目录：
>> ![avatar](/img/antd-design-pro-menu.png)
>> 2.yarn start 启动项目
>> 效果：
>> ![avatar](/img/antd-umi.png)
>> ...
>> 看完这个项目之后，我发现都是一些比较个性化的配置，不适合初学者使用学习，
>> 放弃，回归到选用app，搭建一个通用项目来学习，很尴尬

> 5.搭建一个通用项目
>> 1.yarn create umi
>> 2.选择 app -> use typescript N -> [antd, dva, code splitting]
>> 3.yarn 安装依赖
>> 目录：
>> ![avatar](/img/app-menu.png)
>> 4.yarn start
>> 效果：
>> ![avatar](/img/app-umi.png)

# 文件目录介绍

## 目录
```bash
.
├── dist/                          // 默认的 build 输出目录
├── mock/                          // mock 文件所在目录，基于 express
├── config/
    ├── config.js                  // umi 配置，同 .umirc.js，二选一
└── src/                           // 源码目录，可选
    ├── layouts/index.js           // 全局布局
    ├── pages/                     // 页面目录，里面的文件即路由
        ├── .umi/                  // dev 临时目录，需添加到 .gitignore
        ├── .umi-production/       // build 临时目录，会自动删除
        ├── document.ejs           // HTML 模板
        ├── 404.js                 // 404 页面
        ├── page1.js               // 页面 1，任意命名，导出 react 组件
        ├── page1.test.js          // 用例文件，umi test 会匹配所有 .test.js 和 .e2e.js 结尾的文件
        └── page2.js               // 页面 2，任意命名
    ├── global.css                 // 约定的全局样式文件，自动引入，也可以用 global.less
    ├── global.js                  // 可以在这里加入 polyfill
    ├── app.js                     // 运行时配置文件
├── .umirc.js                      // umi 配置，同 config/config.js，二选一
├── .env                           // 环境变量
└── package.json
```

## ES6 语法
配置文件、mock 文件等都有通过 @babel/register 注册实时编译，所以可以和 src 里的文件一样，使用 ES6 的语法和 es modules 。

## dist
默认输出路径，可通过配置 outputPath 修改。

## mock
此目录下所有的 .js 文件（包括 _ 前缀的）都会被解析为 mock 文件。
比如，新建 mock/users.js，内容如下：

```javascript
export default {
  '/api/users': ['a', 'b'],
}
然后在浏览器里访问 'http://localhost:8000/api/users' 就可以看到 ['a', 'b'] 了。
```

## src
约定 src 为源码目录，如果不存在 src 目录，则当前目录会被作为源码目录。
比如：下面两种目录结构的效果是一致的。

```bash
+ src
  + pages
    - index.js
  + layouts
    - index.js
- .umirc.js
+ pages
  - index.js
+ layouts
  - index.js
- .umirc.js
```

## src/layouts/index.js
> 注：配置式路由下无效。

全局布局，在路由外面套的一层路由。
比如，你的路由是：

```json
[
  { path: '/', component: './pages/index' },
  { path: '/users', component: './pages/users' },
]
如果有 layouts/index.js，那么路由就会变为：

[
  { path: '/', component: './layouts/index', routes: [
    { path: '/', component: './pages/index' },
    { path: '/users', component: './pages/users' },
  ] }
]
```

## src/pages
约定 pages 下所有的 js、jsx、ts 和 tsx 文件即路由。
>注：配置式路由下无效。


## src/pages/404.js
404 页面。注意开发模式下有内置 umi 提供的 404 提示页面，所以只有显式访问 /404 才能访问到这个页面。

## src/pages/document.ejs
有这个文件时，会覆盖默认的 HTML 模板。
模板里需至少包含根节点的 HTML 信息

```html
<div id="root"></div>
```

## src/pages/.umi
这是 umi dev 时生产的临时目录，默认包含 umi.js 和 router.js，有些插件(dva, babel)也会在这里生成一些其他临时文件。可以在这里做一些验证，但请不要直接在这里修改代码，umi 重启或者 pages 下的文件修改都会重新生成这个文件夹下的文件。

## src/pages/.umi-production
同 src/pages/.umi，但是是在 umi build 时生成的，umi build 执行完自动删除。

## .test.(js|ts) 和 .e2e.(js|ts)
测试文件，umi test 会查找所有的 .test.js 和 .e2e.js 文件来跑测试。

## src/global.(js|ts)
此文件会在入口文件的最前面被自动引入，可以在这里加载补丁，做一些初始化的操作等。

## src/global.(css|less|sass|scss)
此文件不走 css modules，且会自动被引入，可以在这里写全局样式，以及做样式覆盖。

## src/app.(js|ts)
运行时配置文件，可以在这里扩展运行时的能力，比如修改路由、修改 render 方法等。

## .umirc.(js|ts) 和 config/config.(js|ts)
编译时配置文件，二选一，不可共存。

## .env
环境变量配置文件，比如：

```bash
CLEAR_CONSOLE=none
BROWSER=none
```

# 路由（重点，umi已约定式路由为基础拓展）
> 路由使用可以在 umi-examples/routes 和 umi-examples/routes-via-config 里找到示例代码。
umi 会根据 pages 目录自动生成路由配置。

## 约定式路由
### 基础路由
假设 pages 目录结构如下：

```bash
+ pages/
  + users/
    - index.js
    - list.js
  - index.js
```

那么，umi 会自动生成路由配置如下：

```json
[
  { path: '/', component: './pages/index.js' },
  { path: '/users/', component: './pages/users/index.js' },
  { path: '/users/list', component: './pages/users/list.js' },
]
```

### 动态路由
umi 里约定，带 $ 前缀的目录或文件为动态路由。

比如以下目录结构：

```bash
+ pages/
  + $post/
    - index.js
    - comments.js
  + users/
    $id.js
  - index.js
```

会生成路由配置如下：

```json
[
  { path: '/', component: './pages/index.js' },
  { path: '/users/:id', component: './pages/users/$id.js' },
  { path: '/:post/', component: './pages/$post/index.js' },
  { path: '/:post/comments', component: './pages/$post/comments.js' },
]
```

### 可选的动态路由
umi 里约定动态路由如果带 $ 后缀，则为可选动态路由。

比如以下结构：

```bash
+ pages/
  + users/
    - $id$.js
  - index.js
```

会生成路由配置如下：

```json
[
  { path: '/': component: './pages/index.js' },
  { path: '/users/:id?': component: './pages/users/$id$.js' },
]
```

### 嵌套路由
umi 里约定目录下有 _layout.js 时会生成嵌套路由，以 _layout.js 为该目录的 layout 。

比如以下目录结构：

```bash
+ pages/
  + users/
    - _layout.js
    - $id.js
    - index.js
```

会生成路由配置如下：

```json
[
  { path: '/users', component: './pages/users/_layout.js',
    routes: [
     { path: '/users/', component: './pages/users/index.js' },
     { path: '/users/:id', component: './pages/users/$id.js' },
   ],
  },
]
```

### 全局 layout
约定 src/layouts/index.js 为全局路由，返回一个 React 组件，通过 props.children 渲染子组件。

比如：

```javascript
export default function(props) {
  return (
    <>
      <Header />
      { props.children }
      <Footer />
    </>
  );
}
```

### 不同的全局 layout
你可能需要针对不同路由输出不同的全局 layout，umi 不支持这样的配置，但你仍可以在 layouts/index.js 对 location.path 做区分，渲染不同的 layout 。

比如想要针对 /login 输出简单布局，

```javascript
export default function(props) {
  if (props.location.pathname === '/login') {
    return <SimpleLayout>{ props.children }</SimpleLayout>
  }

  return (
    <>
      <Header />
      { props.children }
      <Footer />
    </>
  );
}
```

### 404 路由
约定 pages/404.js 为 404 页面，需返回 React 组件。

比如：

```javascript
export default () => {
  return (
    <div>I am a customized 404 page</div>
  );
};
```

> 注意：开发模式下，umi 会添加一个默认的 404 页面来辅助开发，但你仍然可通过精确地访问 /404 来验证 404 页面。

### 通过注释扩展路由
约定路由文件的首个注释如果包含 yaml 格式的配置，则会被用于扩展路由。

比如：

```bash
+ pages/
  - index.js
如果 pages/index.js 里包含：

/**
 * title: Index Page
 * Routes:
 *   - ./src/routes/a.js
 *   - ./src/routes/b.js
 */
 ```

则会生成路由配置：

```json
[
  { path: '/', component: './index.js',
    title: 'Index Page',
    Routes: [ './src/routes/a.js', './src/routes/b.js' ],
  },
]
```

## 配置式路由
如果你倾向于使用配置式的路由，可以配置 routes ，此配置项存在时则不会对 src/pages 目录做约定式的解析。（咱们先按约定式路由搞一遍，然后后边再来一套配置式路由的）

比如：

```javascript
export default {
  routes: [
    { path: '/', component: './a' },
    { path: '/list', component: './b', Routes: ['./routes/PrivateRoute.js'] },
    { path: '/users', component: './users/_layout',
      routes: [
        { path: '/users/detail', component: './users/detail' },
        { path: '/users/:id', component: './users/id' }
      ]
    },
  ],
};
```

> 注意：
component 是相对于 src/pages 目录的

## 权限路由
umi 的权限路由是通过配置路由的 Routes 属性来实现。约定式的通过 yaml 注释添加，配置式的直接配上即可。

比如有以下配置：

```json
[
  { path: '/', component: './pages/index.js' },
  { path: '/list', component: './pages/list.js', Routes: ['./routes/PrivateRoute.js'] },
]
```

然后 umi 会用 ./routes/PrivateRoute.js 来渲染 /list。

./routes/PrivateRoute.js 文件示例：

```javascript
export default (props) => {
  return (
    <div>
      <div>PrivateRoute (routes/PrivateRoute.js)</div>
      { props.children }
    </div>
  );
}
```

## 路由动效
路由动效应该是有多种实现方式，这里举 react-transition-group 的例子。

先安装依赖，

$ yarn add react-transition-group
在 layout 组件（layouts/index.js 或者 pages 子目录下的 _layout.js）里在渲染子组件时用 TransitionGroup 和 CSSTransition 包裹一层，并以 location.pathname 为 key，

```javascript
import withRouter from 'umi/withRouter';
import { TransitionGroup, CSSTransition } from "react-transition-group";

export default withRouter(
  ({ location }) =>
    <TransitionGroup>
      <CSSTransition key={location.pathname} classNames="fade" timeout={300}>
        { children }
      </CSSTransition>
    </TransitionGroup>
)
```

上面用到的 fade 样式，可以在 src 下的 global.css 里定义：

```css
.fade-enter {
  opacity: 0;
  z-index: 1;
}

.fade-enter.fade-enter-active {
  opacity: 1;
  transition: opacity 250ms ease-in;
}
```

## 面包屑
面包屑也是有多种实现方式，这里举 react-router-breadcrumbs-hoc 的例子。

先安装依赖，

$ yarn add react-router-breadcrumbs-hoc
然后实现一个 Breakcrumbs.js，比如：

```javascript
import NavLink from 'umi/navlink';
import withBreadcrumbs from 'react-router-breadcrumbs-hoc';

// 更多配置请移步 https://github.com/icd2k3/react-router-breadcrumbs-hoc
const routes = [
  { path: '/', breadcrumb: '首页' },
  { path: '/list', breadcrumb: 'List Page' },
];

export default withBreadcrumbs(routes)(({ breadcrumbs }) => (
  <div>
    {breadcrumbs.map((breadcrumb, index) => (
      <span key={breadcrumb.key}>
        <NavLink to={breadcrumb.props.match.url}>
          {breadcrumb}
        </NavLink>
        {(index < breadcrumbs.length - 1) && <i> / </i>}
      </span>
    ))}
  </div>
));
```

然后在需要的地方引入此 React 组件即可。

## 启用 Hash 路由
umi 默认是用的 Browser History，如果要用 Hash History，需配置：

```javascrip
export default {
  history: 'hash',
}
```

## Scroll to top
在 layout 组件（layouts/index.js 或者 pages 子目录下的 _layout.js）的 componentDidUpdate 里决定是否 scroll to top，比如：

```javascript
import { Component } from 'react';
import withRouter from 'umi/withRouter';

class Layout extends Component {
  componentDidUpdate(prevProps) {
    if (this.props.location !== prevProps.location) {
      window.scrollTo(0, 0);
    }
  }
  render() {
    return this.props.children;
  }
}

export default withRouter(Layout);
```

# 在页面间跳转
在 umi 里，页面之间跳转有两种方式：声明式和命令式。

## 声明式
基于 umi/link，通常作为 React 组件使用。

```javascript
import Link from 'umi/link';

export default () => (
  <Link to="/list">Go to list page</Link>
);
```

## 命令式
基于 umi/router，通常在事件处理中被调用。

```javascript
import router from 'umi/router';

function goToListPage() {
  router.push('/list');
}
```

更多命令式的跳转方法，详见 api#umi/router。

# 配置
## 配置文件
umi 允许在 .umirc.js 或 config/config.js （二选一，.umirc.js 优先）中进行配置，支持 ES6 语法。

为简化说明，后续文档里只会出现 .umirc.js。

比如：

```javascript
export default {
  base: '/admin/',
  publicPath: 'http://cdn.com/foo',
  plugins: [
    ['umi-plugin-react', {
      dva: true,
    }],
  ],
};
```

具体配置项详见配置。

## .umirc.local.js
.umirc.local.js 是本地的配置文件，不要提交到 git，所以通常需要配置到 .gitignore。如果存在，会和 .umirc.js 合并后再返回。

## UMI_ENV
可以通过环境变量 UMI_ENV 区分不同环境来指定配置。

举个例子，

```javascript
// .umirc.js
export default { a: 1, b: 2 };

// .umirc.cloud.js
export default { b: 'cloud', c: 'cloud' };

// .umirc.local.js
export default { c: 'local' };
不指定 UMI_ENV 时，拿到的配置是：

{
  a: 1,
  b: 2,
  c: 'local',
}
指定 UMI_ENV=cloud 时，拿到的配置是：

{
  a: 1,
  b: 'cloud',
  c: 'local',
}
```

# HTML 模板
## 修改默认模板
新建 src/pages/document.ejs，umi 约定如果这个文件存在，会作为默认模板，内容上需要保证有

```javascript
<div id="root"></div>
```

比如：

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <title>Your App</title>
</head>
<body>
  <div id="root"></div>
</body>
</html>
```

## 配置模板
模板里可通过 context 来获取到 umi 提供的变量，context 包含：

route，路由对象，包含 path、component 等
config，用户配置信息
publicPath2.1.2+，webpack 的 output.publicPath 配置
env，环境变量，值为 development 或 production
其他在路由上通过 context 扩展的配置信息
模板基于 ejs 渲染，可以参考 <https://github.com/mde/ejs> 查看具体使用。

比如输出变量，

```html
<link rel="icon" type="image/x-icon" href="<%= context.publicPath %>favicon.png" />
```

比如条件判断，

```html
<% if(context.env === 'production') { %>
  <h2>生产环境</h2>
<% } else {%>
  <h2>开发环境</h2>
<% } %>
```

# 针对特定页面指定模板
WARNING

此功能需开启 exportStatic 配置，否则只会输出一个 html 文件。

TIP

优先级是：路由的 document 属性 > src/pages/document.ejs > umi 内置模板

配置路由的 document 属性。

比如约定式路由可通过注释扩展 document 属性，路径从项目根目录开始找，

```javascript
/**
 * document: ./src/documents/404.ejs
 */
```

然后这个路由就会以 ./src/documents/404.ejs 为模板输出 HTML。

# Mock 数据
Mock 数据是前端开发过程中必不可少的一环，是分离前后端开发的关键链路。通过预先跟服务器端约定好的接口，模拟请求数据甚至逻辑，能够让前端开发独立自主，不会被服务端的开发所阻塞。

## 使用 umi 的 mock 功能
umi 里约定 mock 文件夹下的文件或者 page(s) 文件夹下的 _mock 文件即 mock 文件，文件导出接口定义，支持基于 require 动态分析的实时刷新，支持 ES6 语法，以及友好的出错提示，详情参看 mock-data。

```javascript
export default {
  // 支持值为 Object 和 Array
  'GET /api/users': { users: [1, 2] },

  // GET POST 可省略
  '/api/users/1': { id: 1 },

  // 支持自定义函数，API 参考 express@4
  'POST /api/users/create': (req, res) => { res.end('OK'); },
};
```

当客户端（浏览器）发送请求，如：GET /api/users，那么本地启动的 umi dev 会跟此配置文件匹配请求路径以及方法，如果匹配到了，就会将请求通过配置处理，就可以像样例一样，你可以直接返回数据，也可以通过函数处理以及重定向到另一个服务器。

比如定义如下映射规则：

```javascript
'GET /api/currentUser': {
  name: 'momo.zxy',
  avatar: imgMap.user,
  userid: '00000001',
  notifyCount: 12,
},
```

访问的本地 /api/users 接口：

请求头

返回的数据

## 引入 Mock.js
Mock.js 是常用的辅助生成模拟数据的第三方库，当然你可以用你喜欢的任意库来结合 roadhog 构建数据模拟功能。

```javascript
import mockjs from 'mockjs';

export default {
  // 使用 mockjs 等三方库
  'GET /api/tags': mockjs.mock({
    'list|100': [{ name: '@city', 'value|1-100': 50, 'type|0-2': 1 }],
  }),
};
```

## 添加跨域请求头
设置 response 的请求头即可：

```javascript
'POST /api/users/create': (req, res) => {
  ...
  res.setHeader('Access-Control-Allow-Origin', '*');
  ...
},
```

## 合理的拆分你的 mock 文件
对于整个系统来说，请求接口是复杂并且繁多的，为了处理大量模拟请求的场景，我们通常把每一个数据模型抽象成一个文件，统一放在 mock 的文件夹中，然后他们会自动被引入。

## 如何模拟延迟
为了更加真实的模拟网络数据请求，往往需要模拟网络延迟时间。

## 手动添加 setTimeout 模拟延迟
你可以在重写请求的代理方法，在其中添加模拟延迟的处理，如：

```javascript
'POST /api/forms': (req, res) => {
  setTimeout(() => {
    res.send('Ok');
  }, 1000);
},
```

## 使用插件模拟延迟
上面的方法虽然简便，但是当你需要添加所有的请求延迟的时候，可能就麻烦了，不过可以通过第三方插件来简化这个问题，如：roadhog-api-doc#delay。

```javascript
import { delay } from 'roadhog-api-doc';

const proxy = {
  'GET /api/project/notice': getNotice,
  'GET /api/activities': getActivities,
  'GET /api/rule': getRule,
  'GET /api/tags': mockjs.mock({
    'list|100': [{ name: '@city', 'value|1-100': 50, 'type|0-2': 1 }]
  }),
  'GET /api/fake_list': getFakeList,
  'GET /api/fake_chart_data': getFakeChartData,
  'GET /api/profile/basic': getProfileBasicData,
  'GET /api/profile/advanced': getProfileAdvancedData,
  'POST /api/register': (req, res) => {
    res.send({ status: 'ok' });
  },
  'GET /api/notices': getNotices,
};
```

// 调用 delay 函数，统一处理

```javascript
export default delay(proxy, 1000);
```

## 动态 Mock 数据
如果你需要动态生成 Mock 数据，你应该通过函数进行处理，

比如：

```javascript
// 静态的
'/api/random': Mock.mock({
  // 只随机一次
  'number|1-100': 100,
}),
// 动态的
'/api/random': (req, res) => {
  res.send(Mock.mock({
    // 每次请求均产生随机值
    'number|1-100': 100,
  }))
},
```

## 联调
当本地开发完毕之后，如果服务器的接口满足之前的约定，那么你只需要不开本地代理或者重定向代理到目标服务器就可以访问真实的服务端数据，非常方便。

# Use umi with dva(重点)
自>= umi@2起，dva的整合可以直接通过 umi-plugin-react 来配置。

## 特性
按目录约定注册 model，无需手动 app.model
文件名即 namespace，可以省去 model 导出的 namespace key
无需手写 router.js，交给 umi 处理，支持 model 和 component 的按需加载
内置 query-string 处理，无需再手动解码和编码
内置 dva-loading 和 dva-immer，其中 dva-immer 需通过配置开启
开箱即用，无需安装额外依赖，比如 dva、dva-loading、dva-immer、path-to-regexp、object-assign、react、react-dom 等

## 使用
用 yarn 安装依赖，

$ yarn add umi-plugin-react
如果你用 npm，执行 npm install --save umi-plugin-react。

然后在 .umirc.js 里配置插件：

```javascript
export default {
  plugins: [
    [
      'umi-plugin-react',
      {
        dva: true,
      },
    ]
  ],
};
```

推荐开启 dva-immer 以简化 reducer 编写，

```javascript
export default {
  plugins: [
    [
      'umi-plugin-react',
      {
        dva: {
          immer: true
        }
      }
    ],
  ],
};
```

## model 注册
model 分两类，一是全局 model，二是页面 model。全局 model 存于 /src/models/ 目录，所有页面都可引用；页面 model 不能被其他页面所引用。

规则如下：

>src/models/**/*.js 为 global model
src/pages/**/models/**/*.js 为 page model
global model 全量载入，page model 在 production 时按需载入，在 development 时全量载入
page model 为 page js 所在路径下 models/**/*.js 的文件
page model 会向上查找，比如 page js 为 pages/a/b.js，他的 page model 为 pages/a/b/models/**/*.js + pages/a/models/**/*.js，依次类推
约定 model.js 为单文件 model，解决只有一个 model 时不需要建 models 目录的问题，有 model.js 则不去找 models/**/*.js
举个例子，

```bash
+ src
  + models
    - g.js
  + pages
    + a
      + models
        - a.js
        - b.js
        + ss
          - s.js
      - page.js
    + c
      - model.js
      + d
        + models
          - d.js
        - page.js
      - page.js
```

如上目录：

>global model 为 src/models/g.js
/a 的 page model 为 src/pages/a/models/{a,b,ss/s}.js
/c 的 page model 为 src/pages/c/model.js
/c/d 的 page model 为 src/pages/c/model.js, src/pages/c/d/models/d.js

## 配置及插件
之前在 src/dva.js 下进行配置的方式已 deprecated，下个大版本会移除支持。

在 src 目录下新建 app.js，内容如下：

```javascript
export const dva = {
  config: {
    onError(e) {
      e.preventDefault();
      console.error(e.message);
    },
  },
  plugins: [
    require('dva-logger')(),
  ],
};
```

## FAQ
### url 变化了，但页面组件不刷新，是什么原因？
layouts/index.js 里如果用了 connect 传数据，需要用 umi/withRouter 高阶一下。

```javascript
import withRouter from 'umi/withRouter';

export default withRouter(connect(mapStateToProps)(LayoutComponent));
```

### 如何访问到 store 或 dispatch 方法？
window.g_app._store
window.g_app._store.dispatch
### 如何禁用包括 component 和 models 的按需加载？
在 .umirc.js 里配置：

```javascript
export default {
  plugins: [
    [
      'umi-plugin-react',
      {
        dva: {
          dynamicImport: undefined // 配置在dva里
        },
        dynamicImport: undefined   // 或者直接写在react插件的根配置，写在这里也会被继承到上面的dva配置里
      }
    ],
  ],
};
```

### 全局 layout 使用 connect 后路由切换后没有刷新？
需用 withRouter 包一下导出的 react 组件，注意顺序。

```javascript
import withRouter from 'umi/withRouter';
export default withRouter(connect()(Layout));
```

# 按需加载
出于性能的考虑，我们会对模块和组件进行按需加载。

## 按需加载组件
通过 umi/dynamic 接口实现，比如：

```javascript
import dynamic from 'umi/dynamic';

const delay = (timeout) => new Promise(resolve => setTimeout(resolve, timeout));
const App = dynamic({
  loader: async function() {
    await delay(/* 1s */1000);
    return () => <div>I will render after 1s</div>;
  },
});
```

## 按需加载模块
通过 import() 实现，比如：

```javascript
import('g2').then(() => {
  // do something with g2
});
```

# 运行时配置
## 为什么有运行时配置？
我们通过 .umirc.js 做编译时的配置，这能覆盖大量场景，但有一些却是编译时很难触及的。

比如：

在出错时显示个 message 提示用户
在加载和路由切换时显示个 loading
页面载入完成时请求后端，根据响应动态修改路由
这些在编译时就很难处理，或者不能处理了。

## 配置方式
umi 约定 src 目录下的 app.js 为运行时的配置文件。

```bash
+ src
  - app.js      # 运行时配置文件
- package.json
```

## 配置列表
### patchRoutes
用于运行时修改路由。
> 参数：
routes: Array，路由配置
e.g. 添加一个 /foo 的路由，

```bash
export function patchRoutes(routes) {
  routes[0].unshift({
    path: '/foo',
    component: require('./routes/foo').default,
  });
}
```

可能的场景：
和 render 配合使用，请求服务端根据响应动态更新路由
修改全部路由，加上某个前缀
...
注：

同样适用约定式路由
直接修改 routes 就好，不要返回新的路由对象
### render
用于改写把整个应用 render 到 dom 树里的方法。

参数：
oldRender， Function，原始 render 方法，需至少被调用一次
e.g. 延迟 1s 渲染应用，

```javascript
export function render(oldRender) {
  setTimeout(oldRender, 1000);
}
```

可能的场景：渲染应用之前做权限校验，不通过则跳转到登录页

### onRouteChange
用于在初始加载和路由切换时做一些事情。

> 参数：
Object
location：Object, history 提供的 location 对象
routes: Array, 路由配置
action: PUSH|POP|REPLACE|undefined，初次加载时为 undefined
e.g.

```javascript
export function onRouteChange({ location, routes, action }) {
  bacon(location.pathname);
}
```

> 可能的场景：埋点统计
> 注：初次加载时也会执行，但 action 为 undefined

### rootContainer
用于封装 root container，可以取一部分，或者外面套一层，等等。

> 参数：container，ReactComponent，React 应用最上层的根组件 e.g.

```javascript
export function rootContainer(container) {
  const DvaContainer = require('@tmp/DvaContainer').default;
  return React.createElement(DvaContainer, null, container);
}
```

> 可能的场景：dva、intl 等需要在外层有个 Provider 的场景

### modifyRouteProps
修改传给路由组件的 props。

> 参数：
props，Object，原始 props
Object
route，Object，当前路由配置
e.g.

```javascript
export function modifyRouteProps(props, { route }) {
  return { ...props, foo: 'bar' };
}
```

> 注：需返回新的 props

# 区块（区块开发就不赘述了，比较高级的使用方法）
在 umi 中，区块是指页面级别的可复用的代码。umi 定义了一个区块的规范，你可以通过 umi 能够快速简单的在你的项目中添加区块，用于快速的开始一个页面的开发。

# 部署
## 默认方案
umi@2 默认对新手友好，所以默认不做按需加载处理，umi build 后输出 index.html、umi.js 和 umi.css 三个文件。

## 不输出 html 文件
某些场景 html 文件交给后端输出，前端构建并不需要输出 html 文件，可配置环境变量 HTML=none 实现。

```javascript
$ HTML=none umi build
```

## 部署 html 到非根目录
经常有同学问这个问题：

为什么我本地开发是好的，部署后就没反应了，而且没有报错？

没有报错！ 这是应用部署在非根路径的典型现象。为啥会有这个问题？因为路由没有匹配上，比如你把应用部署在 /xxx/ 下，然后访问 /xxx/hello，而代码里匹配的是 /hello，那就匹配不上了，而又没有定义 fallback 的路由，比如 404，那就会显示空白页。

怎么解决？

可通过配置 base 解决。

```javascript
export default {
  base: '/path/to/your/app/root',
};
```

## 使用 hashHistory
可通过配置 history 为 hash 为解决。

```javascript
export default {
  history: 'hash',
};
```

## 按需加载
要实现按需加载，需装载 umi-plugin-react 插件并配置 dynamicImport。

```javascript
export default {
  plugins: [
    ['umi-plugin-react', {
      dynamicImport: true,
    }],
  ],
};
```

> 参数详见：umi-plugin-react#dynamicImport。

## 静态资源在非根目录或 cdn
这时，就需要配置 publicPath。至于 publicPath 是啥？具体看 webpack 文档，把他指向静态资源（js、css、图片、字体等）所在的路径。

```javascript
export default {
  publicPath: "http://yourcdn/path/to/static/"
}
```

## 使用 runtime 的 publicPath
对于需要在 html 里管理 publicPath 的场景，比如在 html 里判断环境做不同的输出，可通过配置 runtimePublicPath 为解决。

```javascript
export default {
  runtimePublicPath: true,
};
```

然后在 html 里输出：

```html
<script>
window.publicPath = <%= YOUR PUBLIC_PATH %>
</script>
```

## 静态化
在一些场景中，无法做服务端的 html fallback，即让每个路由都输出 index.html 的内容，那么就要做静态化。

比如上面的例子，我们在 .umirc.js 里配置：

```javascript
export default {
  exportStatic: {},
}
```

然后执行 umi build，会为每个路由输出一个 html 文件。

```bash
./dist
├── index.html
├── list
│   └── index.html
└── static
    ├── pages__index.5c0f5f51.async.js
    ├── pages__list.f940b099.async.js
    ├── umi.2eaebd79.js
    └── umi.f4cb51da.css
```

注意：静态化暂不支持有变量路由的场景。

## HTML 后缀
有些静态化的场景里，是不会自动读索引文件的，比如支付宝的容器环境，那么就不能生成这种 html 文件，

```bash
├── index.html
├── list
│   └── index.html
而是生成，

├── index.html
└── list.html
```

配置方式是在 .umirc.js 里，

```javascript
export default {
  exportStatic: {
    htmlSuffix: true,
  },
}
```

umi build 会生成，

```bash
./dist
├── index.html
├── list.html
└── static
    ├── pages__index.5c0f5f51.async.js
    ├── pages__list.f940b099.async.js
    ├── umi.2924fdb7.js
    └── umi.cfe3ffab.css
```

## 静态化后输出到任意路径

```javascript
export default {
  exportStatic: {
    htmlSuffix: true,
    dynamicRoot: true,
  },
}
```

# 实战-写一个登陆及管理后台menu demo

## 目录
```bash
+ mock
  - index.js mock数据
+ src
  + pages
    + index
      - index.js 根路由 即 登陆页
      - index.css
      - model.js
      - service.js
    + dashboard
      - index.js 登陆后首页
+ utils
  - request.js 定义请求对象
```

## 首先定义utils下的request请求对象

```javascript
  // dva 的fetch 相对底层一些，要自己去扩展很多方法
  import fetch from 'dva/fetch';
  import router from 'umi/router';
  import { message } from 'antd'; // 使用antd弹窗

  function parseJSON(response) {
    return response.json(); // 对于res的解析必须调这个方法 dva 里的 虽然有点奇怪
  }

  function checkStatus(response) {
    if (response.status === 200) {
      return response;
    } else if(response.status === 303){
      router.replace('/'); // cookie失效跳回登录页
    } else {
      response.json().then(message.error);
    }
  }

  // 对于query的format
  const parseQuery = (obj) => {
    let str = ''
    for (let key in obj) {
      const value = typeof obj[key] !== 'string' ? JSON.stringify(obj[key]) : obj[key]
      str += '&' + key + '=' + value
    }
    return str.substr(1)
  }
  /**
  * Requests a URL, returning a promise.
  *
  * @param  {string} url       The URL we want to request
  * @param  {object} [options] The options we want to pass to "fetch"
  * @return {object}           An object containing either "data" or "err"
  */
  const request = (url, method = 'get', data = {}) => {
    const options = {
      method,   // HTTP请求方法，默认为GET
      headers:{
        'Content-Type': 'application/json'
      },
      credentials: 'include' 
      // 是否携带cookie，默认为omit,不携带; same-origi,同源携带; include,同源跨域都携带
    }
    if (method === 'get') {
      url += '&' + parseQuery(data)
    } else {
      options.body = JSON.stringify(data);
    }
    return fetch(url, options)
    .then(checkStatus)
    .then(parseJSON)
    .then(data => (data))
    .catch(err => ({ err }));
  }

  export default {
    get: (url, data) => {
      return request(url, 'get', data);
    },
    post: (url, data) => {
      return request(url, 'post', data);
    },
    put: (url, data) => {
      return request(url, 'put', data);
    },
    delete: (url, data) => {
      return request(url, 'delete', data);
    },
  };
```

## 在service下export出在model中调用的api func

```javascript
  import request from '@/utils/request';

  export function login (user = {}) {
    return request.post('/login', user);
  };
```

## 定义model对象

```javascript
  import * as service from './service';

  export default {
    namespace: 'user', // 定义命名空间 调用时要指定命名空间

    state: { // 数据存储
      user: {},  //reducers中接收数据
    },

    subscriptions: { // 暂时没搞懂是干啥的
      setup({ dispatch, history }) {  // eslint-disable-line
      },
    },

    effects: { // actions
      // 是个promise 因为集成了redux-saga
      * login({ payload: { user, cb = () => {} }}, { call, put }) { // * 声明是个异步func
        // yield 将等待异步方法返回结果 类似 await
        // call是dva标准用法用来调用api，第一个参数是你的api func，第二个参数是你传入api func 的参数
        const result = yield call(service.login, user);// 如果使用  {参数}  ，则是一个对象
        //把请求的数据保存起来
        //数据更新会带动页面重新渲染
        yield put({
          type: 'setUser',  //reducers中的方法名
          payload: {
            user: result,  //网络返回的要保留的数据
          },
        });
        return result;
      }
    },

    reducers: {
      // 注意！！！effects中的方法名不能与reducers中的方法名重复，否则会死循环
      setUser(state, { payload: { user } }) {
        return {
          ...state, 
          user,
        };
      },
    },
  }
```

## mock/index.js加些mock数据

```javascript
  export default {
    // 支持值为 Object 和 Array
    'POST /login': (req, res) => {
      const { account, password } = req.body;
      if (account === 'umi' && password === '1234') {
        res.status(200).send({
          account: 'umi',
          name: '乌米',
          role: '超级管理员',
          permissions: [1, 2, 3],
        });
      } else {
        res.status(403).json('账号或密码错误');
      }
    },
  };
```

## login页面（antd） ， 接口调用及路由跳转

```javascript
  import React from 'react';
  import { connect } from 'dva';
  import router from 'umi/router';
  import { Form, Input, Icon, Checkbox, Button } from 'antd';

  import style from './index.css';

  let Login = class extends React.Component {
    handleSubmit = e => {
      e.preventDefault();
      this.props.form.validateFields((err, values) => {
        if (!err) {
          // actions 调用方式 命名空间/action名字
          // 是个promise 因为集成了redux-saga
          this.props.dispatch({
            type: 'user/login',
            payload: {
              user: {
                account: values.account,
                password: values.password,
              },
            },
          })
          .then(() => {
            if (values.remember) {
              window.localStorage.setItem('account', values.account);
              window.localStorage.setItem('password', values.password);
            }
            router.push('/dashboard');
          });
        }
      });
    };

    render() {
      const { getFieldDecorator } = this.props.form;
      return (
        <div className={style.loginPanel}>
          <div className={style.loginForm}>
            <Form onSubmit={this.handleSubmit}>
              <Form.Item>
                {getFieldDecorator('account', {
                  rules: [{ required: true, message: 'Please input your username!' }],
                })(
                  <Input
                    prefix={<Icon type="user" style={{ color: 'rgba(0,0,0,.25)' }} />}
                    placeholder="Username"
                  />,
                )}
              </Form.Item>
              <Form.Item>
                {getFieldDecorator('password', {
                  rules: [{ required: true, message: 'Please input your Password!' }],
                })(
                  <Input
                    prefix={<Icon type="lock" style={{ color: 'rgba(0,0,0,.25)' }} />}
                    type="password"
                    placeholder="Password"
                  />,
                )}
              </Form.Item>
              <Form.Item>
                {getFieldDecorator('remember', {
                  valuePropName: 'checked',
                  initialValue: true,
                })(<Checkbox className={style.remember}>Remember me</Checkbox>)}
                <Button type="primary" htmlType="submit" className="login-form-button">
                  Log in
                </Button>
              </Form.Item>
            </Form>
          </div>
        </div>
      );
    }
  }
  Login = Form.create({ name: 'normal_login' })(Login); // antd form 使用方法
  export default  connect()(Login);
  // 将actions connect进来 不需要再像react中一样手动加actions
  // 这里只需要手动connect mapStatusToProps
```
