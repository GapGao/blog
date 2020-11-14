---
title: vue3之初体验
date: 2020-10-19 00:08:46
tags: [vue3, 前端]
---

前段时间，神秘的尤爸爸终于发布了vue3.0。因为发布个vue3，尤爸爸还一度成为网红，积攒了一大波粉丝，又是搞直播又是搞宣传什么的，哈哈。今天让我们来看看vue3相较于vue2有哪些升级和不同吧~

<!--more-->

## 6大亮点

1. Performance：性能更比Vue 2.0强。
2. Tree shaking support：可以将无用模块“剪辑”，仅打包需要的。
3. Composition API：组合API
4. Fragment, Teleport, Suspense
5. Better TypeScript support：更优秀的Ts支持
6. Custom Renderer API：暴露了自定义渲染API

## break change 重大变化

### 全局API

1. Global Vue API is changed to use an application instance
   人话就是，在vue2中，创建一个全局组件或自定义指令都是挂载全局Vue下的，例如 `Vue.directive`，`Vue.component`等，这样可以使该Vue应用全局都可以使用。但是弊端是，对于同一个Vue，new 的两个实例应用，都会带上同样的全局属性，难以分别，会引起诸多不便。所以这些在Vue3中得到了优化，会将相关操作转而挂在应用下，也就是new 出来的实例下，有不同作用域。
2. Global and internal APIs have been restructured to be tree-shakable
   全局和内部的API 可以 tree-shaking

### 模板指令

1. v-model usage on components has been reworked
   v-model 用法重新设计优化
2. key usage on `<template v-for>` and non-v-for nodes has changed
   key 在 `<template v-for>`和 非v-for节点上的使用情况已更改
   key在v-if/ v-else/v-else-if分支上不再需要
   key 在2中可以放在v-for的子元素上，现在必须要放在 v-for的元素上
3. v-if and v-for precedence when used on the same element has changed
   v-if和v-for在同一元素上使用时的优先级已更改
   如果用于同一元素上时，则v-if优先级高于v-for
4. v-bind="object" is now order-sensitive
   v-bind的绑定顺序将影响渲染结果
   vue2时，有同样的属性或绑定时会有原生优先级的区别，但vue3，则完全按生命顺序进行合并
5. ref inside v-for no longer register an array of refs
   v-for内部使用ref时，不再注册数组，而是提供一个:ref方法，自己处理ref的存储格式和维护

### 组件

1. Functional components can only be created using a plain function
   Functional组件只能使用普通函数创建，在vue2时，Functional组件主要为了提升组件性能，因为它的初始化和有状态组件相比要快的多，但是vue3已经将两者优化的性能几乎无差距，所以Functional组件存在的意义就不是那么大了，只用于一些小组件
2. functional attribute on single-file component (SFC) `<template>` and functional component option are deprecated
   functional单文件组件（SFC）上的属性`<template>`和functional组件选项已弃用
3. Async components now require defineAsyncComponent method to be created
   异步组件现在需要有defineAsyncComponent方法包裹创建，并且之前的高级用法里的component字段改为loader

### 渲染功能

1. Render function API changed
   render函数变动，vue2 render会接收一个h参数即createElement方法，而后再调用。vue3，render不再接收h，而是从Vue中import进来再调用，且VNode结构变的更加扁平
2. $scopedSlots property is removed and all slots are exposed via $slots as functions
   this.$scopedSlots属性被删除，所有插槽都通过$slots函数公开

### 自定义元素

1. Custom elements whitelisting is now performed during template compilation
2. Special is prop usage is restricted to the reserved `<component>` tag only
这俩不知道说啥呢，可能是高级用法，我也没整明白

### 其他小的变化

1. destroyed => unmounted
2. beforeDestroy => beforeUnmount
3. Props default factory function no longer has access to this context
   props的default定义 函数 将不能在访问this 但可以通过inject方法实现
4. Custom directive API changed to align with component lifecycle
   自定义指令会有生命周期 在元素不同情况下可以有不同的调用 与 组件生命周期统一
5. The data option should always be declared as a function
   vue2 data 可以用 object定义 也可以用 function return object 定义，但vue3必须用function return object
6. The data option from mixins is now merged shallowly
   mixins 中的 data 现在会被浅合并
7. Attributes coercion strategy changed
   html 的 attr 的绑定 false 不再强制删除，而是置为`'false'`，使用null或undefined进行去除，这样就统一了输出
8. Some transition classes got a rename
   v-enter => v-enter-from
   v-leave => v-leave-from
9. When watching an array, the callback will only trigger when the array is replaced. If you need to trigger on mutation, the deep option must be specified.
    watch 方法在监听 array 时，仅在数组完全赋值替换的时候才会触发，如果需要在单项改变时触发，必须给一个deep: true字段进行配置。
10. `<template>` tags with no special directives (v-if/else-if/else, v-for, or v-slot) are now treated as plain elements and will result in a native `<template>` element instead of rendering its inner content.
11. In Vue 2.x, application root container's outerHTML is replaced with root component template (or eventually compiled to a template, if root component has no template/render option). Vue 3.x now uses application container's innerHTML instead - this means the container itself is no longer considered part of the template.
后边这俩没看懂。。。

### 删除的API

1. keyCode support as v-on modifiers
   config.keyCodes keycode别名定义，不再受支持
   v-on不再支持使用数字（即keyCodes）作为修饰符 例如：`<input v-on:keyup.13="submit" />`
   主要是因为 KeyboardEvent.keyCode 已弃用，推荐用 KeyboardEvent.key，但又说会有兼容性问题，难。
2. $on, $off and $once instance methods
   这几个删了 主要是用于 eventBus，但是emit还在。
3. Filters
   筛选器删了，特么难受，我觉得挺好用的啊，我经常用，推荐使用computed和function调用替换
4. Inline templates attributes
   没用过不知道
5. $destroy instance method. Users should no longer manually manage the lifecycle of individual Vue components.
   删除了$destroy实例方法， 用户不应再手动管理各个Vue组件的生命周期

终于把migration文档看完了。。

### Options API 和 Composition API

在vue2中，我们会在一个vue文件中methods，computed，watch，data中等等定义属性和方法，共同处理页面逻辑，我们称这种方式为Options API。
缺点是：一个功能往往需要在不同的vue配置项中定义属性和方法，比较分散，项目小还好，清晰明了，但是项目大了后，一个methods中可能包含20多个方法，你往往分不清哪个方法对应着哪个功能

vue3中的Composition API就是用来解决这个问题的
在vue3 Composition API 中，我们的代码是根据逻辑功能来组织的，一个功能所定义的所有api会放在一起（更加的高内聚，低耦合），这样做，即时项目很大，功能很多，我们都能快速的定位到这个功能所用到的所有API，而不像vue2 Options API 中一个功能所用到的API都是分散的，需要改动功能，到处找API的过程是很费劲的。

为什么要使用 Composition API？
Composition API 是根据逻辑相关性组织代码的，提高可读性和可维护性
基于函数组合的 API 更好的重用逻辑代码（在vue2 Options API中通过Mixins重用逻辑代码，容易发生命名冲突且关系不清）

## 切入正题，专注搞事

### 引入方式

1. CDN方式<script src="https://unpkg.com/vue@next"></script>
2. 通过Vite脚手架 npm init vite-app hello-vue3 OR yarn create vite-app hello-vue3
3. 通过vue-cli脚手架 OR yarn global add @vue/cli
   npm install -g @vue/cli
   vue create hello-vue3
   select vue 3 preset

### vue-cli引入
我们就用最熟悉的vue-cli引入吧，折腾一下
执行 `vue create hello-vue3`
![avatar](/img/vue3/1.png)
我们选择使用vue3，激动
目录结构如下
![avatar](/img/vue3/2.png)
执行 `npm run server`
8080端口起了服务，我终于用上了网红框架，vue3哈哈哈。
![avatar](/img/vue3/3.png)

看看入口

```javascript
// main.js
import { createApp } from 'vue'
import App from './App.vue'

// 可以看到 vue 跟应用的创建方式已经不是 new Vue了，而是createApp，创建一个应用，别切应用间配置可以相互隔离
createApp(App).mount('#app')
// App.vue
<template>
  <img alt="Vue logo" src="./assets/logo.png">
  <HelloWorld msg="Welcome to Your Vue.js App"/>
</template>

// 感觉这代码规范天天变 之前的规范还是 <hello-world> 呢
```

```javascript
import { ref, onMounted, onUpdated, onUnmounted } from "vue";

function useCounter() {
  const count = ref(0);
  function increment() {
    count.value++;
  }

  return {
    count,
    increment,
  };
}

export default {
  name: "App",
  components: {
    HelloWorld,
  },
  data() {
    return {
      a: 1,
      b: 2,
    };
  },
  mounted() {
    console.log(this.a, this.b);
  },
  setup() {
    const { count, increment } = useCounter();

    onMounted(() => {
      console.log("mounted!");
    });
    onUpdated(() => {
      console.log("updated!");
    });
    onUnmounted(() => {
      console.log("unmounted!");
    });

    return {
      count,
      increment,
    };
  },
};
```

我写了如上一段代码，是可以正常运行的，所以vue3在向后（vue2 Option API）兼容的基础上，新增了 Composition API，并且他们是可以同时存在的。害，我还以为vue2一些语法不能用了呢。但是，根据运行结果来看，setup 里的hook的执行是优先于mounted的，毕竟 setup 是主要方向。

```javascript
// 可以给创建的应用单独配置设置和添加应用内全局指令或其他功能
// 不会影响其他应用
const app = createApp(App).mount('#app');
app.config = {...}
app.directive("my-directive", {
  // Directive has a set of lifecycle hooks:
  // called before bound element's parent component is mounted
  beforeMount() {},
  // called when bound element's parent component is mounted
  mounted() {},
  // called before the containing component's VNode is updated
  beforeUpdate() {},
  // called after the containing component's VNode and the VNodes of its children // have updated
  updated() {},
  // called before the bound element's parent component is unmounted
  beforeUnmount() {},
  // called when the bound element's parent component is unmounted
  unmounted() {},
});
```

## 总结

vue3在大部分兼容vue2的基础上，做了很多的优化和更新（最基础的响应式原理，我们之前已经研究过了），当然也有很多breaking change。这次我们就先大致了解它的升级变化，基本用法，更高级的用法，比如setup 花式hook之类的，我们后续再去深入研究。

## 引用

>https://v3.vuejs.org/guide/migration/introduction.html