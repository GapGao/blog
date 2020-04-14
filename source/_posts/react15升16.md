---
title: react15升16
date: 2020-01-31 17:47:23
tags: [react15, react16, migration]
---

React 16已经发布很长时间了 目前最新版本是 16.12.0 让我们来看看从15升级到16都要做哪些迁移吧~

<!--more-->

## 基础依赖升级

1. react 16.12.0
2. react-dom 16.12.0
3. prop-types 15.7.2

如果引用redux的话
4. redux 4.0.5
5. react-redux 7.1.3
6. redux-thunk 2.3.0

如果引用transition的话
7. react-transition-group 4.3.0

## 基础依赖安装

1.react-router-dom 5.1.2
2.react-router-config 5.1.1

## 三方依赖升级

一些组件库需要升级版本，如果不升级，依赖的不是react16的话，就会有一些生命周期函数的warning
这些生命周期函数将在17彻底废除掉

## migration 之 路由

升级之后发现跑不起来了，是因为react-router的写法完全变了react 和 react-router相互版本依赖 所以要一起升级，react-router已经到5了可怕而秉承万物即组件的理念 react-router 相较之前的写法有了很大的变化，迁移改动巨大。
那么使用路由组件的写法，一是改动成本大，二是不集中管理路由的话，又很难维护代，路由分散，对开发和debug查bug什么的对于新同学来说非常繁琐隐晦。
莫慌，我们还装了一个叫react-router-config的库，它可以帮助我们仍然可以集中管理路由，并且有点像vue一样的json写法，让你的路由配置变的神清气爽。
像这样

```javascript
import React, { lazy, Suspense } from 'react';
import { BrowserRouter } from 'react-router-dom';
import { renderRoutes } from 'react-router-config';

const App = lazy(() => import('./containers/App'));
const Organization = lazy(() => import('./containers/Organization'));
const ApplicationList = lazy(() => import('./containers/ApplicationList'));
const ExcelApplicationList = lazy(() => import('./containers/ExcelApplicationList'));
const JobList = lazy(() => import('./containers/JobList'));
const Job = lazy(() => import('./containers/Job'));
const CandidateInfo = lazy(() => import('./components/CandidateInfo'));
const Result = lazy(() => import('./containers/Result'));
const ExcelResult = lazy(() => import('./containers/ExcelResult'));

const routes = [
  {
    path: ['/headhunters/:orgId', '/headhunters'],
    // 是按顺序匹配
    // 如果exact为true 虽然会精确匹配 但是 触发子https://reacttraining.com/blog/react-router-v5-1/路由 父路由被精确掉了所以App就没了 子组件也就没有render了
    // 所以统一带id的放在前边 优先自动匹配
    // 有子路由的exact必须为false
    component: App,
    routes: [
      {
        path: '/headhunters/:orgId/org',
        component: Organization,
        routes: [
          {
            path: '/headhunters/:orgId/org/jobs',
            exact: true,
            component: JobList,
          },
          {
            path: '/headhunters/:orgId/org/applications',
            exact: true,
            component: ApplicationList,
          },
          {
            path: '/headhunters/:orgId/org/excel_applications',
            exact: true,
            component: ExcelApplicationList,
          },
        ],
      },
      {
        path: '/headhunters/:orgId/job/:jobId',
        exact: true,
        component: Job,
      },
      {
        path: '/headhunters/:orgId/recommend/:jobId',
        component: CandidateInfo,
        routes: [
          {
            path: '/headhunters/:orgId/recommend/:jobId/result',
            exact: true,
            component: Result,
          },
          {
            path: '/headhunters/:orgId/recommend/:jobId/excel_result',
            exact: true,
            component: ExcelResult,
          },
        ],
      },
      {
        path: '/headhunters/:orgId/update/:applicationId',
        component: CandidateInfo,
        routes: [
          {
            path: '/headhunters/:orgId/update/:applicationId/result',
            exact: true,
            component: Result,
          },
        ],
      },
    ],
  },
];
export default (
  <BrowserRouter>
    <Suspense fallback={null}>
      {renderRoutes(routes)}
    </Suspense>
  </BrowserRouter>
);
// 其实这里还用该有个 <Switch></Switch>组件，代表同级的路由只匹配一个，但是renderRoutes内部已经包含了该组件，这里就没有写
```

这样的路由配置结构清晰，嵌套清晰。

class组件的render方法，``withRouter``基本不变，但是可以看到export出去的路由多了很多东西。BrowserRouter组件代表使用history模式的路由，对，还有一个``HashRouter``，react-router已经内置了history模式，不需要再去引别的库了。

``this.props.router.`` 等都改为 ``this.props.history.``。

且react16引入了lazy 和 Suspense，分别代表懒加载模块，以及将来时的声明组件，两者配合使用，还可以fallback，在加载时降级展示的组件，react自己引入了代码分割功能。

> 具体用法参考 <https://react.docschina.org/docs/code-splitting.html>

renderRoutes就是react-router-config库的render方法了，源码很简单，就是递归你的配置路由，自己去看。使用这个方法将配置的路由render出来。

在子路由的render还是要改动，以前子路由的render可能是这样的

```javascript
{
  this.props.children && React.cloneElement(
  this.props.children,
  {
    ...
  })
}
```

现在改成这样

```javascript
  import { renderRoutes } from 'react-router-config';

  // 凡是路由组件，经过renderRoutes之后都会有一个route字段，即当前匹配的路由配置
  renderRoutes(route.routes, { ...props })
  // 只有匹配了子路由才会，子组件才会被render
```

```javascript
  import { renderRoutes, matchRoutes } from 'react-router-config';

  // 但是对于这种children是否存在，即是否匹配到子路由的情况的判断应该怎么判断呢
  // 上边的例子是匹配哪个子路由就render哪个子组件，没有匹配就不render所以不会有这样的情况
  // 但是这里需要在不匹配时render别的组件
  const renderDom = children || Component;

  // 可以这样写
  // matchRoutes是路由匹配方法，判断当前地址pathname是否匹配到了路由，匹配的路径是啥，返回的是一个数组
  // 我们可以直接以子路由为匹配输入参数 进行匹配，如果branch的length>0则说明当前地址匹配到了 当前层 路由的子路由，即有children
  const branch = matchRoutes(this.props.route.routes, this.props.location.pathname);
  const renderDom = branch.length ? children : Component;
```

* 需要注意的是，``exact``这个字段，默认为false，顾名思义精确匹配，且是按层级的精准匹配

```json
  {
    path: '/headhunters/:orgId/update/:applicationId',
    component: CandidateInfo,
    exact: true,
    routes: [
        {
        path: '/headhunters/:orgId/update/:applicationId/result',
        component: Result,
        },
    ],
  }
```

比如这个路由``/headhunters/:orgId/update/:applicationId``，exact为true时，它的子路由``/headhunters/:orgId/update/:applicationId/result``，只会显示``Result``这个组件，而不会显示``CandidateInfo``这个组件，因为没有精准匹配到``/headhunters/:orgId/update/:applicationId``这个路由，这对于嵌套路由父子组件都显示（例如列表组件上浮着的详情浮层）的期望是不匹配的，所以exact为true，一定不要给到有子组件的路由上，要给到无子组件的路由上（特殊情况除外）。

* 另外还有一个``strict``字段，默认为false，如果为true时，路由后面有斜杠而url中没有斜杠，是不匹配的
* 另外react-router5好像不支持那种带括号啊之类的路由了，比如``/headhunters(/:orgId)``（有没有orgId都会匹配），但是path可以接受一个数组，也就是这个数组里的路由会按顺序挨个去匹配，那么我们可以改写上边的路由，例如``path: ['/headhunters/:orgId', '/headhunters']``，以更精确的路由来配置。
* 对于function组件，在 react-router-dom 中有``let history = useHistory()``钩子🐶，可以使用，想咋使咋使。

> 具体用法参考 <https://reacttraining.com/blog/react-router-v5-1/>

## migration 之 params 和 query

params不再存在于this.props.location里了

location中只剩下这几个字段了，当前路由和search等

```json
  location: {
    pathname: "/headhunters/gaobo/org/jobs"
    search: "?aa=123"
    hash: ""
    state: undefined
  }
```

那params和query跑哪了嘞？

可以看到props里多了一个match字段，对，params跑这里去了，顾名思义，这个字段是路由匹配的结果，里边有

```json
  match: {
    path: "/headhunters/:orgId"
    url: "/headhunters/gaobo"
    isExact: false
    params: {orgId: "gaobo"}
  }
```

这些字段，path当前路由组件对应的路由配置，url是当前地址匹配到当前组件对应路由配置的部分，isExact略，params跑这了，所以之前有依赖这个字段的地方，会比较大。以前``this.props.params``，现在得``this.props.match.params``，虽然繁琐，但是对props的数据结构也更合理（改吧）。

query去哪了？对，没了。
在 v4 中不再能直接获取到 query 的值了。由于没有处理复杂 query 字符串的标准。所以 v4 决定把 query 字符串的处理权留给开发者，这样其实也挺好的，可以用qs等库自己处理，但是依赖query改变而触发不同render的情况，我还没改到那，但是我估计会比较麻烦，可能需要forceUpdate之类的吧。毕竟不算是props里的东西了，那它也不会引起组件的重新render。

* 对于function组件，在 react-router-dom 中有``let { xxx } = useParams()``，``let location = useLocation()``钩子，随便造，还是方便了很多的。

> 具体用法参考 <https://reacttraining.com/blog/react-router-v5-1/>

## migration 之 react-transition-group

迁移

```javascript
transitionName -> classNames
transitionEnterTimeout and transitionLeaveTimeout -> timeout={{ exit, enter }}
transitionAppear -> appear
transitionEnter -> enter
transitionLeave -> exit
```

原来

```javascript
  import CSSTransitionGroup from 'react/lib/ReactCSSTransitionGroup';
  // 或者
  import CSSTransitionGroup from 'react-addons-css-transition-group';
  // 这些都是低版本react的用法 里边包含着一些即将被废弃的生命周期函数（会报warning）所以需要一并升级

  <CSSTransitionGroup
    transitionName="toast-animation"
    transitionEnterTimeout={500}
    transitionLeaveTimeout={500}
  >
  {
    toasts.map((, index) => (
      <div key={index}></div>
    ))
  }
  </CSSTransitionGroup>
```

现在

```javascript
import { CSSTransition, TransitionGroup } from 'react-transition-group';
  <TransitionGroup>
  {
    toasts.map((, index) => (
      <CSSTransition
        classNames="toast-animation"
        timeout={{ exit: 500, enter: 500 }}
        key={index}
        // 还增加了一些生命周期函数
        onEnter={handleEnter}
        onEntering={handleEntering}
        onEntered={handleEntered}
        onExit={handleExit}
        onExiting={handleExiting}
        onExited={handleExited}
      >
        <div className={styles.item}></div>
      </CSSTransition>
    ))
  }
  </TransitionGroup>
```

> 具体用法参考 <https://github.com/reactjs/react-transition-group/blob/master/Migration.md>

## migration 之 生命周期函数

有一些生命周期函数在16里边会报warning，且到17的时候会被废弃掉不能用了。
比如 ```componentWillReceiveProps``` 改为 ```UNSAFE_componentWillReceiveProps```
将 ```componentWillMount``` 改为 ```UNSAFE_componentWillMount```等等

由于目前项目较大，完全更换生命周期函数，工作量和风险都比较大，目前先改成UNSAFE了。

## 总结

以上是对development环境下的react15升16的一些改造
对production环境，打包升级，还没开始做，后续做了会更新，尽情期待~

## 引用

> <https://react.docschina.org/docs/code-splitting.html>
> <https://reacttraining.com/blog/react-router-v5-1/>
> <https://github.com/reactjs/react-transition-group/blob/master/Migration.md>
