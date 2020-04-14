---
title: Nuxt+express踩坑日记
date: 2019-07-20 15:24:01
tags: [Nuxt, express, vue]
---

# 背景介绍
公司企业官网要更新改版，老版本官网使用express+pug+jq及一些原生js搭建，不利于项目的维护和迭代，一些ui组件难复用，开发周期长。继而调研ssr（Server Side Render）框架，Nuxt。能够很好地解决以上问题，且保持良好的seo效果。

<!--more-->

# ssr个人理解
出现ssr的原因就是，vue或react等库，在生成的静态文件中，index.html只是一个容器，而实际的界面ui及文案等都在引用的js文件中，而这对于搜索引擎的seo是很不友好的，因为搜索引擎只抓取html。而引用以上库之后的项目，请求之后吐出的只有一个空壳子需要运行js之后才会填充ui界面。所以ssr，服务端渲染应用框架应运而生，它会在服务端运行实例的部分生命周期，完成首次渲染，吐出已填充好的html，并将实例的另一部分生命周期交给client接管，运行后边的代码及逻辑。

# Nuxt官方介绍
>Nuxt.js 是一个基于 Vue.js 的通用应用框架，与之对应的react ssr框架是next.js（早于nuxt）。
通过对客户端/服务端基础架构的抽象组织，Nuxt.js 主要关注的是应用的 UI渲染。
我们的目标是创建一个灵活的应用框架，你可以基于它初始化新项目的基础结构代码，或者在已有 Node.js 项目中使用 Nuxt.js。
Nuxt.js 预设了利用 Vue.js 开发服务端渲染的应用所需要的各种配置。
除此之外，我们还提供了一种命令叫：nuxt generate ，为基于 Vue.js 的应用提供生成对应的静态站点的功能。
我们相信这个命令所提供的功能，是向开发集成各种微服务（Microservices）的 Web 应用迈开的新一步。
作为框架，Nuxt.js 为 客户端/服务端 这种典型的应用架构模式提供了许多有用的特性，例如异步数据加载、中间件支持、布局支持等。
Nuxt是基于 Vue、Webpack 和 Babel

>Nuxt.js 集成了以下组件/框架，用于开发完整而强大的 Web 应用：
Vue 2
Vue-Router
Vuex (当配置了 Vuex 状态树配置项 时才会引入)
Vue 服务器端渲染 (排除使用 mode: 'spa')
Vue-Meta
另外，Nuxt.js 使用 Webpack 和 vue-loader 、 babel-loader 来处理代码的自动化构建工作（如打包、代码分层、压缩等等）。

# 项目搭建
>由于是旧项目改版，且市场部设计只设计了pc端页面，移动端未设计，所以需求是pc端用新版官网，而移动端用旧版官网，也是有点奇葩的。所以只好用Nuxt与就官网的express+pug混合加翻新。

>Nuxt的create-nuxt-app官方demo安装好之后，可以看到很多选项，根据需求和现状，选择使用多页面应用（mode: Universal），express服务器。

## dev环境
>安装依赖
由于旧版官网的打包及babel等依赖的版本老旧，而Nuxt@2.6.x依赖webpack@4、babel@7，注意babel@7与旧版本的不同，babel@7将其下相关依赖整合，@babel/core，@babel/polyfill，@babel/preset-env，@babel/register等，整合之后清晰简洁不少，引用方便。且对应babel-loader@8，注意polyfill，preset-env，register引用方式的替换，都变成了类似@babe/xxx。
由于老版官网使用webpack3，升级了wabpack之后，相应对webpack配置文件用新版本写法更新，配置好各种依赖、loader、plugin等用于旧版官网的兼容打包处理。

>将Nuxt项目文件夹与旧版官网的目录分好之后，去nuxt.config.js(Nuxt的webpack及vue等已内置，且有一些他们的基础配置和nuxt本身的一些基础配置，此文件就是来补充一些自定义的配置merge进默认配置)，配置Nuxt的srcDir路径（项目资源目录）。所以需要两个webpack配置，一个是旧官网的配置，一个是Nuxt的webpack的自定义配置，最终项目也会以不同的打包方式生成并吐给client。
由于官网还要render旧版官网，所以选用了Nuxt的express起服务器，在官方demo生成之后，研究了下它的index.js起服务的文件，发现是引用了Nuxt库里的基类，以nuxt.config.js配置实例化一个Nuxt实例，并调用其内部的builder及render方法来实现服务器的请求及相应，先等待builder的打包，在经过Nuxt的render方法render。

>继而将Nuxt的express服务器代码和传统express服务器代码混合改造。先Nuxt build然后判断是pc端还是移动端，pc端直接app.use(nuxt.render)，否则使用旧官网app.use('/', router)。public文件夹及其他不变，可以理解为如果是pc端直接用nuxt渲染，移动端使用传统express渲染。

经过以上配置，node index.dev.js，启动dev服务器后则看到pc端及移动端不同渲染方式吐出的页面了。

## prod环境
由于Nuxt支持指令式操作 nuxt build，但是此操作只对Nuxt项目管用，对与旧官网要用另一个webpack配置来打包。所以考虑沿用旧官网的gulp来调用各自的build方法。这里顺便升级了gulp到4，使用了gulp的一些新的串行并行的写法，确实简洁了不少。在打包旧官网的同时，并行打包新官网。在Nuxt源码中找到了builder和generator的源码，因为在官网没找到这个api，很尴尬。

```javascript
  gulp.task('build:new-client:compile', function(callback) {
    nuxtConfig.dev = false;
    const Nuxt = new nuxt.Nuxt(nuxtConfig);
    const builder = new nuxt.Builder(Nuxt);
    const generator = new nuxt.Generator(Nuxt, builder);
    generator.generate();
    Nuxt.hook('generate:done', function() {
        callback();
    });
  });
```

在调用Builder之后，会生成一个编译后的项目文件夹.nuxt，这包括前端和服务器端的代码，所以要提取出静态资源还要调用Generator，将其输出至指定文件夹下。

以下为nuxt.config.js对应generate的配置

```json
  generate: {
    dir: 'new-client-dist',
  }
```

这样gulp build之后就可以输出新版及旧版官网的静态文件了。

翻翻看Nuxt生成的.html文件，里边确实已经包含了一些实际的内容，并不是一个空容器。然后再对生产环境的服务器代码进行改造，主要是路由的改造，就可以跑起来了。

# 一些觉得挺好的小知识点

其实都是来自于官方文档，但是这里提炼出一些我在开发过程中觉得很好用的，或者一开始没在文档中找到的知识点。

> 1.nuxt.config.js

```json
  modules: [
    '@nuxtjs/style-resources',
  ],
  // 可以随处直接使用定义过的变量或函数
  styleResources: {
    stylus: ['./assets/palette.styl', './assets/util.styl', './assets/product_page.styl'],
  }
```

modules是加载一些module引用的plugin，使得后边的那些styl可以在每个.vue中均被自动引入，less啊saas啊都行，觉得比较好用。

> 2.app.html可以写一些比如在线客服啊，埋点啊，百度或头条等seo的一些东西。
> 3.assest 放一些.styl，.saas，.less，.ts，需要打包或编译的一些资源文件，static放一些静态文件，middleware和plugin我还没用过哈哈。
> 4.layout里边放一些共用的容器模板文件，每个路由对应的.vue文件会自动填充在对应容器内，当然，填充在哪个容器内是可以配置的。在路由对应的.vue文件里可以配置

```javascript
export default {
  layout: 'xxx',
  layout: (context) => {
    // 此声明周期在后端，可以打log在后端看看
    return 'xxx';
  },
}
```

这样就可以配置自定义容器模板啦，默认是default.vue

> 5.Nuxt的路由跟next.js、umi啊之类的是一致的，是约定式的路由，即文件即路由，在pages下创建好对应文件及文件名之后，就会以文件名为路由了，当然一些比较复杂或重复引用的路由还是要去手动配置。

# 总结
这里我写了接到一个官网改版的需求之后的调研及项目搭建，还有一些Nuxt使用心得，比较浅显与片面，后续还会抽时间更深入的学习ssr原理，并实践更多的ssr及其他框架的项目，积累更多的项目经验。
