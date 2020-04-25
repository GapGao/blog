---
title: ES6-Proxy 能干点啥
date: 2020-04-25 16:49:12
tags: [es6, Proxy]
---

顾名思义，Proxy 的意思是代理，作用是为其他对象提供一种代理以控制对这个对象的访问。

<!--more-->

## Proxy 提供了哪些拦截方式

Proxy 一般是用来架设在目标对象之上的一层拦截，来实现对目标对象访问和修改的控制。Proxy 是一个构造函数，使用的时候需要配合 new 操作符，直接调用会报错。
![avatar](/img/proxy/1.webp)

Proxy 构造函数接收两个参数，第一个参数是需要拦截的目标对象，这个对象只可以是对象、数组或者函数；

第二个参数则是一个配置对象，提供了拦截方法，即使这个配置对象为空对象，返回的 Proxy 实例也不是原来的目标对象。

```javascript
const person = {
  name: 'tom',
};

// 如果第二个参数为空对象
const proxy = new Proxy(person, {});
console.log(proxy === person); // false

// 第二个参数不为空
const proxy = new Proxy(person, {
  get(target, prop) {
      console.log(`${prop} is ${target[prop]}`);
      return target[prop];
  }
})
proxy.name // 'name is tom'
```

Proxy 支持13种拦截操作，我们重点了解下其中四种。
![avatar](/img/proxy/2.webp)

1. get(target, prop, receiver)：拦截对象属性的访问。
2. set(target, prop, value, receiver)：拦截对象属性的设置，最后返回一个布尔值。
3. apply(target, object, args)：用于拦截函数的调用，比如 proxy()。
4. construct(target, args)：方法用于拦截 new 操作符，比如 new proxy()。为了使 new操作符在生成的Proxy对象上生效，用于初始化代理的目标对象自身必须具有 [[Construct]] 内部方法（即 new target 必须是有效的）。
5. has(target, prop)：拦截例如 prop in proxy的操作，返回一个布尔值。
6. deleteProperty(target, prop)：拦截例如 delete proxy[prop] 的操作，返回一个布尔值。
7. ownKeys(target)：拦截 Object.getOwnPropertyNames(proxy)、Object.keys(proxy)、for in 循环等等操作，最终会返回一个数组。
8. getOwnPropertyDescriptor(target, prop)：拦截 Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象。
9. defineProperty(target, propKey, propDesc)：拦截 Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs)，返回一个布尔值。
10. preventExtensions(target)：拦截 Object.preventExtensions(proxy)，返回一个布尔值。
11. getPrototypeOf(target)：拦截 Object.getPrototypeOf(proxy)，返回一个对象。
12. isExtensible(target)：拦截 Object.isExtensible(proxy)，返回一个布尔值。
13. setPrototypeOf(target, proto)：拦截 Object.setPrototypeOf(proxy, proto)，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。

## Proxy 对比 Object.defineProperty

在 Proxy 出现之前，JavaScript 中就提供过 Object.defineProperty，允许对对象的 getter/setter 进行拦截，那么两者的区别在哪里呢？

![avatar](/img/proxy/3.webp)

可见Object.defineProperty的功能支持较少有很多局限性，但是兼容性好，主要因为支持的较早。
随着浏览器的的升级换代，Proxy的兼容性已经不是什么问题了大多都支持了。
所以vue3也已经由ES6的Proxy来代替Object.defineProperty实现。

### Object.defineProperty 不能监听所有属性

`Object.defineProperty` 无法一次性监听对象所有属性，必须遍历或者递归来实现。

```javascript
let girl = {
  name: "marry",
  age: 22,
};

/* Proxy 监听整个对象*/
girl = new Proxy(girl, {
  get() {},
  set() {},
});

/* Object.defineProperty */
Object.keys(girl).forEach(key => {
  Object.defineProperty(girl, key, {
    set() {},
    get() {},
  })
});
```

### Object.defineProperty 无法监听新增加的属性

`Proxy` 可以监听到新增加的属性，而 `Object.defineProperty` 不可以，需要你手动再去做一次监听。因此，在 Vue 中想动态监听属性，一般用 `Vue.set(girl, "hobby", "game")` 这种形式来添加。

```javascript
let girl = {
  name: "marry",
  age: 22,
};

/* Proxy 监听整个对象*/
girl = new Proxy(girl, {
  get() {},
  set() {},
});

/* Object.defineProperty */
Object.keys(girl).forEach(key => {
  Object.defineProperty(girl, key, {
    set() {},
    get() {},
  })
});

/* Proxy 生效，Object.defineProperty 不生效 */
girl.hobby = "game"; 
```

### Object.defineProperty 无法响应数组操作

`Proxy` 可以监听数组的变化，`Object.defineProperty` 无法对 push、shift、pop、unshift 等方法进行响应。

```javascript
const arr = [1, 2, 3];

/* Proxy 监听数组*/
arr = new Proxy(arr, {
  get() {},
  set() {},
});

/* Object.defineProperty */
arr.forEach((item, index) => {
  Object.defineProperty(arr, `${index}`, {
    set() {},
    get() {},
  })
});

arr[0] = 10; // 都生效
arr[3] = 10; // 只有 Proxy 生效
arr.push(10); // 只有 Proxy 生效
```

对于新增加的数组项，`Object.defineProperty` 依旧无法监听到。因此，在 Mobx 中为了监听数组的变化，默认将数组长度设置为1000，监听 0-999 的属性变化...

```javascript
/* mobx 的实现 */
const arr = [1, 2, 3];

/* Object.defineProperty */
[...Array(1000)].forEach((item, index) => {
  Object.defineProperty(arr, `${index}`, {
    set() {},
    get() {},
  })
});

arr[3] = 10; // 生效
arr[4] = 10; // 生效
```

如果想要监听到 push、shift、pop、unshift等方法，该怎么做呢？在 Vue 和 Mobx 中都是通过重写原型实现的。
在定义变量的时候，判断其是否为数组，如果是数组，那么就修改它的 __proto__，将其指向 subArrProto，从而实现重写原型链。

```javascript
const arrayProto = Array.prototype;
const subArrProto = Object.create(arrayProto);
const methods = ['pop', 'shift', 'unshift', 'sort', 'reverse', 'splice', 'push'];

methods.forEach(method => {
  /* 重写原型方法 */
  subArrProto[method] = function() {
    arrayProto[method].call(this, ...arguments);
  };
  /* 监听这些方法 */
  Object.defineProperty(subArrProto, method, {
    set() {},
    get() {},
  });
});
```

### Proxy 拦截方式更多

`Proxy` 提供了13种拦截方法，包括拦截 `constructor`、`apply`、`deleteProperty` 等等，而 `Object.defineProperty` 只有 `get` 和 `set`。

### Object.defineProperty 兼容性更好

Proxy 是新出的 API，兼容性还不够好，不支持 IE 全系列。

## 语法

### get

`get` 方法用来拦截对目标对象属性的读取，它接收三个参数，分别是目标对象、属性名和 `Proxy` 实例本身。
基于 `get` 方法的特性，可以实现很多实用的功能，比如在对象里面设置私有属性（一般定义属性我们以 _ 开头表明是私有属性） ，实现禁止访问私有属性的功能。

```javascript
const person = {
  name: 'tom',
  age: 20,
  _sex: 'male',
};

const proxy = new Proxy(person, {
  get(target, prop) {
      if (prop[0] === '_') {
          throw new Error(`${prop} is private attribute`);
      }
      return target[prop];
  },
});

proxy.name; // 'tom'
proxy._sex; // _sex is private attribute
```

还可以给对象中未定义的属性设置默认值。通过拦截对属性的访问，如果是 `undefined`，那就返回最开始设置的默认值。

```javascript
let person = {
  name: 'tom',
  age: 20,
};

const defaults = (obj, initial) => {
  return new Proxy(obj, {
    get(target, prop) {
        if (prop in target) {
            return target[prop];
        }
        return initial;
    },
  });
};

person = defaults(person, 0);
person.name // 'tom'
person.sex // 0
person = defaults(person, null);
person.sex // null
```

### set

`set` 方法可以拦截对属性的赋值操作，一般来说接收四个参数，分别是目标对象、属性名、属性值、Proxy 实例。
下面是一个 `set` 方法的用法，在对属性进行赋值的时候打印出当前状态。

```javascript
const proxy = new Proxy({}, {
  set(target, key, value, receiver) {
    console.log(`${key} has been set to ${value}`);
    Reflect.set(target, key, value);
  },
});

proxy.name = 'tom'; // name has been setted ygy
```

第四个参数 `receiver` 则是指当前的 Proxy 实例，在下例中指代 `proxy`。

```javascript
const proxy = new Proxy({}, {
  set(target, key, value, receiver) {
    if (key === 'self') {
        Reflect.set(target, key, receiver);
    } else {
        Reflect.set(target, key, value);
    }
  },
});

proxy.self === proxy; // true
```

如果你写过表单验证，也许会被各种验证规则搞得很头疼。使用 Proxy 可以在填写表单的时候，拦截其中的字段进行格式校验。
通常来说，大家都会用一个对象来保存验证规则，这样会更容易对规则进行扩展。

```javascript
// 验证规则
const validators = {
  name: {
    validate(value) {
      return value.length > 6;
    },
    message: '用户名长度不能小于六',
  },
  password: {
    validate(value) {
      return value.length > 10;
    },
    message: '密码长度不能小于十',
  },
  moblie: {
    validate(value) {
      return /^1(3|5|7|8|9)[0-9]{9}$/.test(value);
    },
    message: '手机号格式错误',
  },
};
```

然后编写验证方法，用 set 方法对 form 表单对象设置属性进行拦截，拦截的时候用上面的验证规则对属性值进行校验，如果校验失败，则弹窗提示。

```javascript
// 验证方法
function validator(obj, validators) {
  return new Proxy(obj, {
    set(target, key, value) {
      const validator = validators[key]
      if (!validator) {
        target[key] = value;
      } else if (validator.validate(value)) {
        target[key] = value;
      } else {
        alert(validator.message || "");
      }
    },
  });
};

let form = {};
form = validator(form, validators);
form.name = '666'; // 用户名长度不能小于六
form.password = '113123123123123';
```

但是，如果这个属性已经设置为不可写，那么 `set` 将不会生效（但 set 方法依然会执行）。

```javascript
const person = {
  name: 'tom',
};

Object.defineProperty(person, 'name', {
  writable: false,
});

const proxy = new Proxy(person, {  
  set(target, key, value) {
    target[key] = 'jerry';
  },
});

proxy.name = '';
```

## apply

`apply` 一般是用来拦截函数的调用，它接收三个参数，分别是目标对象、上下文对象（this）、参数数组。

```javascript
function test() {
  console.log('this is a test function');
};

const func = new Proxy(test, {
  apply(target, context, args) {
    console.log('hello, world');
    target.apply(context, args);
  },
});

func();
```

通过 `apply` 方法可以获取到函数的执行次数，也可以打印出函数执行消耗的时间，常常可以用来做性能分析。

```javascript
function log() {}
const func = new Proxy(log, {
  _count: 0,
  apply(target, context, args) {
    target.apply(context, args);
    console.log(`this function has been called ${++this._count} times`);
  },
});

func();
```

可以做一些hook、节流防抖之类的功能。

## construct

`construct` 方法用来拦截 `new` 操作符。它接收三个参数，分别是目标对象、构造函数的参数列表、Proxy 实例，最后需要返回一个对象。
使用方式可以参考下面这么一个例子：

```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;
};

const P = new Proxy(Person, {
  construct(target, args, newTarget) {
    console.log('construct');
    return new target(...args);
  },
});

const p = new P('tom', 21); // 'construct'
```

我们知道，如果构造函数没有返回任何值或者返回了原始类型的值，那么默认返回的就是 this，如果返回的是一个引用类型的值，那么最终 new 出来的就是这个值。
因此，你可以代理一个空函数，然后返回一个新的对象。

```javascript
function noop() {};

const Person = new Proxy(noop, {
  construct(target, args, newTarget) {
    return {
      name: args[0],
      age: args[1],
    };
  },
});

const person = new Person('tom', 21); // { name: 'tom', age: 21 }
```

## Proxy 可以做哪些有意思的事情

Proxy 的使用场景非常广泛，可以用来拦截对象的 set/get 从而实现数据响应。在 Vue3 和 Mobx5 中都使用了 Proxy 代替 Object.defineProperty。那么接下来就来看看 Proxy 都可以做哪些事情吧。

### 骚操作：代理类

使用 construct 可以代理类，你可能会好奇，Proxy 不是只能代理 Object 类型吗？类该怎么代理呢？

其实类的本质也是构造函数和原型（对象）组成的，完全可以对其进行代理。
考虑有这么一个需求，需要拦截对属性的访问，以及计算原型上函数的执行时间，这样该怎么去做就比较清晰了。可以对属性设置 `get` 拦截，对原型函数设置 `apply` 拦截。

先考虑对下面的 `Person` 类的原型函数进行拦截。使用 `Object.getOwnPropertyNames` 来获取原型上面所有的函数，遍历这些函数并对其使用 `apply` 拦截。

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  say() {
    console.log(`my name is ${this.name}, and my age is ${this.age}`)
  }
};

const prototype = Person.prototype;
// 获取 prototype 上所有的属性名
Object.getOwnPropertyNames(prototype).forEach((name) => {
  Person.prototype[name] = new Proxy(prototype[name], {
    apply(target, context, args) {
      console.time();
      target.apply(context, args);
      console.timeEnd();
    },
  });
});
```

拦截了原型函数后，开始考虑拦截对属性的访问。前面刚刚讲过 `construct` 方法的作用，那么是不是可以在 new 的时候对所有属性的访问设置拦截呢？
没错，由于 new 出来的实例也是个对象，那么完全可以对这个对象进行拦截。

```javascript
new Proxy(Person, {
  // 拦截 construct 方法
  construct(target, args) {
    const obj = new target(...args);
    // 返回一个代理过的对象
    return new Proxy(obj, {
      get(target, prop) {
        console.log(`${target.name}.${prop} is being getting`);
        return target[prop];
      },
    });
  },
});
```

所以，最后完整的代码如下：

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  say() {
    console.log(`my name is ${this.name}, and my age is ${this.age}`)
  }
};

const proxyTrack = (targetClass) => {
  const prototype = targetClass.prototype;
  Object.getOwnPropertyNames(prototype).forEach((name) => {
    targetClass.prototype[name] = new Proxy(prototype[name], {
      apply(target, context, args) {
        console.time();
        target.apply(context, args);
        console.timeEnd();
      },
    });
  });

  return new Proxy(targetClass, {
    construct(target, args) {
      const obj = new target(...args);
      return new Proxy(obj, {
        get(target, prop) {
          console.log(`${target.name}.${prop} is being getting`);
          return target[prop];
        },
      });
    },
  });
};

const MyClass = proxyTrack(Person);
const myClass = new MyClass('tom', 21);
myClass.say();
myClass.name;
```

### 等不及可选链：深层取值（get）

平时取数据的时候，经常会遇到深层数据结构，如果不做任何处理，很容易造成 JS 报错。
为了避免这个问题，也许你会用多个 && 进行处理：

```javascript
const country = {
  name: 'china',
  province: {
    name: 'guangdong',
    city: {
      name: 'shenzhen',
    },
  },
};

const cityName = country.province
  && country.province.city
  && country.province.city.name;
```

但这样还是过于繁琐了，于是 Lodash 提供了 get 方法帮处理这个问题：

```_.get(country, 'province.city.name');```

虽然看起来似乎还不错，但总觉得哪里不太对（好是好，就是太丑了）。
最新的 ES 提案中提供了可选链的语法糖，支持我们用下面的语法来深层取值。

```country?.province?.city?.name```

但是这个特性只是处于 `stage3` 阶段，还没有被正式纳入 ES 规范中，更没有浏览器已经支持了这个特性。
所以，我们只能另辟蹊径。这时你可能会想到如果使用 Proxy 的 get 方法拦截对属性的访问，这样是不是就可以实现深层取值了呢？

预期结果

```javascript
const obj = {
  person: {},
};

get(obj)() === obj;
get(obj).person(); // {}
get(obj).person.name(); // undefined
get(obj).person.name.xxx.yyy.zzz(); // undefined
```

我们来实现一下：
首先，创建一个 get 方法，使用 Proxy 中的 get 对传入的对象进行拦截。

```javascript
function get (obj) {
  return new Proxy(obj, {
    get(target, prop) {
      return target[prop];
    },
  });
};

// 结果
get(obj).person; // {}
get(obj).person.name; // undefined
get(obj).person.name.xxx.yyy.zzz; // Cannot read property 'xxx' of undefined
```

前两个测试用例是成功了，但第三个还是不行，因为 `get(obj).person.name` 是 `undefined`, 所以接下来的重点是处理属性为 `undefined` 的情况。
对这个 get 方法进行一下简单的改造，这次不再直接返回 target[prop]，而是返回一个代理对象，让第三个例子不再报错。

```javascript
function get (obj) {
  return new Proxy(obj, {
    get(target, prop) {
      return get(target[prop]);
    },
  });
};
```

嗯，看起来有点儿高大上了，但是 `target[prop]` 为 `undefined` 的时候，传给 get 方法的就是 `undefined` 了，而 Proxy 第一个参数必须为对象，这样岂不是会报错？
所以，需要对 `obj` 为 `undefined` 的时候进行特殊处理，为了能够深层取值，只能对值为 `undefined` 的属性设置默认值为空对象。

```javascript
function get (obj = {}) {
    return new Proxy(obj, {
        get(target, prop) {
            return get(target[prop]);
        }
    })
}
get(obj).person; // {}
get(obj).person.name; // {}
get(obj).person.name.xxx.yyy.zzz; // {}
```

虽然不报错了，可是后两个返回值却不对了。不设置默认值为空对象就无法继续访问，设置默认值为空对象就会改变返回值。这可该怎么办呢？
仔细看一下上面的预期设计，是不是发现少了一个括号，这就是为什么每个属性都被当做函数来执行。
所以需要对这个函数稍加修改，让其支持 apply 拦截的方式。

```javascript
function noop() {}
function get (obj) {
  // 注意这里拦截的是 noop 函数
  return new Proxy(noop, {
      // 这里支持返回执行的时候传入的参数
      apply(target, context, [arg]) {
        return obj;
      },
      get(target, prop) {
        return get(obj[prop]);
      },
  });
};
```

所以这个 get 方法已经可以这样使用了。

```javascript
get(obj)() === obj; // true
get(obj).person.name(); // undefined
get(obj).person.name.xxx.yyy.zzz(); // Cannot read property 'xxx' of undefined
```

我们理想中的应该是，如果属性为 `undefined` 就返回 `undefined`，但仍要支持访问下级属性，而不是抛出错误。顺着这个思路来的话，很明显当属性为 `undefined` 的时候也需要用 Proxy 进行特殊处理。
所以我们需要一个具有下面特性的 get 方法：

```javascript
get(undefined)() === undefined; // true
get(undefined).xxx.yyy.zzz() // undefined
```

和前面的困扰不一样的地方是，这里完全不需要注意 get(undefined).xxx 是否为正确的值，因为想获取值必须要执行才能拿到。那么只需要对所有 undefined 后面访问的属性都默认为 undefined 就好了。

```javascript
function noop() {};
function get (obj) {
  if (obj === undefined) {
    return proxyVoid;
  }
  // 注意这里拦截的是 noop 函数
  return new Proxy(noop, {
    // 这里支持返回执行的时候传入的参数
    apply(target, context, [arg]) {
      return obj === undefined ? arg : obj;
    },
    get(target, prop) {
      if (
          obj !== undefined &&
          obj !== null &&
          obj.hasOwnProperty(prop)
      ) {
          return get(obj[prop]);
      }
      return proxyVoid;
    },
  });
};
```

接下来思考一下这个 `proxyVoid` 函数该如何实现呢？很明显它应该是一个代理了 `undefined` 后返回的对象。直接这样好不好？

```javascript
const proxyVoid = get(undefined);
```

但是这样很明显会造成死循环了，那么就需要判断临界值了，让 `get` 方法第一次接收 `undefined` 的时候不会死循环。

```javascript
let isFirst = true;
function noop() {}
let proxyVoid = get(undefined);
function get(obj) {
  if (obj === undefined && !isFirst) {
    return proxyVoid;
  }
  if (obj === undefined && isFirst) {
    isFirst = false;
  }
  // 注意这里拦截的是 noop 函数
  return new Proxy(noop, {
    // 这里支持返回执行的时候传入的参数
    apply(target, context, [arg]) {
      return obj === undefined ? arg : obj;
    },
    get(target, prop) {
      if (
          obj !== undefined &&
          obj !== null &&
          obj.hasOwnProperty(prop)
      ) {
          return get(obj[prop]);
      }
      return proxyVoid;
    },
  });
};
```

我们再来验证一下，这种方式是否可行：

```javascript
get(obj)() === obj; // true
get(obj).person.name(); // undefined
get(obj).person.name.xxx.yyy.zzz(); // undefined
```

这个方法完全实现了我们的需求。最后，完整的代码如下：

```javascript
let isFirst = true;
function noop() {}
let proxyVoid = get(undefined);
function get(obj) {
  if (obj === undefined || obj === null) {
    if (!isFirst) {
        return proxyVoid;
    }
    isFirst = false;
  }
  // 注意这里拦截的是 noop 函数
  return new Proxy(noop, {
    // 这里支持返回执行的时候传入的参数
    apply(target, context, [arg]) {
      return obj === undefined || obj === null ? arg : obj;
    },
    get(target, prop) {
      if (
          obj !== undefined &&
          obj !== null &&
          obj.hasOwnProperty(prop)
      ) {
        return get(obj[prop]);
      }
      return proxyVoid;
    },
  });
};
this.get = get;
```

### 管道

在最新的 ECMA 提案中，出现了原生的管道操作符 |>，在 RxJS 和 NodeJS 中都有类似的 pipe 概念。

使用 Proxy 也可以实现 pipe 功能，只要使用 get 对属性访问进行拦截就能轻易实现，将访问的方法都放到 stack 数组里面，一旦最后访问了 execute 就返回结果。

```javascript
const pipe = (value) => {
  const stack = [];
  return new Proxy({}, {
    get(target, prop) {
      if (prop === 'execute') {
        return stack.reduce(function (val, fn) {
          return fn(val);
        }, value);
      }
      stack.push(window[prop]);
      return proxy;
    },
  });
};

var double = n => n * 2;
var pow = n => n * n;

pipe(3).double.pow.execute;
```

* 注意：这里为了在 stack 存入方法，使用了 window[prop] 的形式，是为了获取到对应的方法。也可以将 double 和 pow 方法挂载到一个对象里面，用这个对象替换 window。
