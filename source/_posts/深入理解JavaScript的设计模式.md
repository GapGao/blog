---
title: 深入理解JavaScript的设计模式
date: 2019-11-21 22:42:08
tags: [javascript, 前端]
---

使用适当的设计模式可以帮助你编写更好、更易于理解的代码。这样的代码也更容易维护。但是，重要的是不要过度使用它们。在使用设计模式之前，你应该仔细考虑你的问题是否符合设计模式。

当你开始一个新的项目时，你不会立即开始编码。你必须定义项目的目的和范围，然后列出项目特性或规格说明。之后，你可以开始编写代码，或者，如果你正在参与的是一个更复杂的项目，那么你应该选择一个最适合项目的设计模式。

<!--more-->

## 什么是设计模式？

在软件工程中，设计模式是软件设计中常见问题的可重用解决方案。设计模式代表了经验丰富的软件开发人员所使用的最佳实践。设计模式可以看作是编程模板。

## 为什么要使用设计模式？

有许多程序员，他们要么认为设计模式浪费时间，要么不知道如何恰当地应用它们。但是，使用适当的设计模式可以帮助你编写更好、更易于理解的代码。这样的代码也更容易维护。

最重要的是，设计模式为软件开发人员提供了一个可以谈论的通用词汇表。它们可以让学习代码的人快速了解代码的意图。

例如，如果你在项目中使用了装饰模式，那么新程序员就会立即知道那段代码在做什么，他们可以把更多的精力放在解决业务问题上，而不是试图理解那段代码在做什么。

现在我们知道了什么是设计模式，以及为什么它们很重要。接下来，让我们深入探讨下应用于 JavaScript 的各种设计模式。

## 模块模式

模块是一段自包含的代码，因此，我们可以在不影响代码其他部分的情况下更新模块。模块还允许我们通过为变量创建单独的作用域来避免命名空间污染。当模块与其他代码片段松耦合时，我们还可以在其他项目中重用它们。

模块是任何现代化 JavaScript 应用程序的组成部分，有助于保持代码的整洁、隔离和条理性。使用 JavaScript 创建模块有很多方法，其中之一就是模块模式。

像 Bit 这样的平台可以帮助你将模块和组件转换成共享的构建块，可以在任何项目中共享、发现和开发。不需要任何重构，就可以使用一种快速且可扩展的方式共享和重用代码。

与其他编程语言不同，JavaScript 没有访问修饰符，也就是说，不能将变量声明为 private 或 public。因此，模块模式也被用来模拟封装的概念。

该模式使用 iife（即时调用函数表达式）、闭包和函数作用域来模拟这个概念，例如：

```javascript
const myModule = (function () {
  const privateVariable = "Hello World";

  function privateMethod() {
    console.log(privateVariable);
  }
  return {
    publicMethod: function () {
      privateMethod();
    },
  };
})();
myModule.publicMethod();
```

由于是 iife，所以代码会立即执行，返回的对象赋给 myModule 变量。由于是闭包，所以返回的对象仍然可以访问在 iife 中定义的函数和变量，即使在 iife 结束之后。

因此，在 iife 中定义的变量和函数本质上是对外部作用域隐藏的，这使得它成为 myModule 变量私有的。

执行代码后，myModule 变量如下：

```javascript
const myModule = {
  publicMethod: function () {
    privateMethod();
  },
};
```

因此，我们可以调用 publicMethod()，而它又会调用 privateMethod()，例如：

```javascript
// 打印'Hello World'
module.publicMethod();
```

## 揭示模块模式

揭示模块模式是经 Christian Heilmann 略微改进的模块模式。模块模式的问题是，我们必须创建新的公共函数来调用私有函数和变量。

在这个模式中，我们将把返回对象的属性映射到我们想要公开的私有函数。这就是为什么它被称为揭示模块模式，例如：

```javascript
const myRevealingModule = (function () {
  let privateVar = "Peter";
  const publicVar = "Hello World";
  function privateFunction() {
    console.log("Name: " + privateVar);
  }

  function publicSetName(name) {
    privateVar = name;
  }
  function publicGetName() {
    privateFunction();
  }
  /** 把希望公开的方法和变量赋给对象属性 */
  return {
    setName: publicSetName,
    greeting: publicVar,
    getName: publicGetName,
  };
})();
myRevealingModule.setName("Mark");
// 打印姓名：Mark
myRevealingModule.getName();
```

这种模式使我们更容易理解哪些函数和变量可以公开访问，这有助于提高代码的可读性。

执行代码之后，myRevealingModule 是下面这个样子：

```javascript
const myRevealingModule = {
  setName: publicSetName,
  greeting: publicVar,
  getName: publicGetName,
};
```

我们可以调用 myrevealingmodule. setname ('Mark')，它是对方法 publicSetName 的引用，而 myRevealingModule.getName() 是对内部方法 publicGetName 的引用，例如:

```javascript
myRevealingModule.setName("Mark");
// 打印姓名：Mark
myRevealingModule.getName();
```

### 与模块模式相比，揭示模块模式的优点如下

- 通过修改 return 语句中的一行代码，我们就可以将成员从 public 更改为 private，反之亦然。
- 返回的对象不包含任何函数定义，所有右侧表达式都在 iife 中定义，这使得代码清晰且易于阅读。

## ES6 模块

在 ES6 之前，JavaScript 没有内置模块，因此，开发人员不得不依赖第三方库或模块模式来实现模块。但是在 ES6 中，JavaScript 有了本地模块。

ES6 模块存储在文件中。每个文件只能有一个模块。默认情况下，模块中的所有内容都是私有的。函数、变量和类都是使用 export 关键字公开的。模块内的代码总是在严格模式下运行。

## 输出模块

有多种方法可以公开函数和变量声明：

在函数和变量声明前添加 export 关键字，例如：

```javascript
// utils.js
export const greeting = "Hello World";
export function sum(num1, num2) {
  console.log("Sum:", num1, num2);
  return num1 + num2;
}
export function subtract(num1, num2) {
  console.log("Subtract:", num1, num2);
  return num1 - num2;
}
// 这是一个私有函数
function privateLog() {
  console.log("Private Function");
}
```

在代码末尾添加 export 关键字，其中包含我们希望公开的函数名和变量名，例如：

```javascript
// utils.js
function multiply(num1, num2) {
  console.log("Multiply:", num1, num2);
  return num1 * num2;
}
function divide(num1, num2) {
  console.log("Divide:", num1, num2);
  return num1 / num2;
}
// 这是一个私有函数
function privateLog() {
  console.log("Private Function");
}
export { multiply, divide };
```

### 导入模块

和输出模块类似，借助 import，有多种方法可以导入模块：

一次导入多个项：

```javascript
// main.js
// 导入多个项
import { sum, multiply } from "./utils.js";
console.log(sum(3, 7));
console.log(multiply(3, 7));
```

导入所有模块：

```javascript
// main.js
// 导入所有模块
import * as utils from "./utils.js";
console.log(utils.sum(3, 7));
console.log(utils.multiply(3, 7));
```

### 导入和输出的别名

如果你想要避免命名冲突，则可以在输出和导入时更改名称，例如：

重命名输出：

```javascript
// utils.js
function sum(num1, num2) {
  console.log("Sum:", num1, num2);
  return num1 + num2;
}
function multiply(num1, num2) {
  console.log("Multiply:", num1, num2);
  return num1 * num2;
}
export { sum as add, multiply };
```

重命名导入：

```javascript
// main.js
import { add, multiply as mult } from "./utils.js";
console.log(add(3, 7));
console.log(mult(3, 7));
```

## 单例模式

单例对象是只能实例化一次的对象。如果一个类的实例不存在，单例模式就会创建一个新的类实例。如果实例存在，它只返回对该对象的引用。对构造函数的任何重复调用都会获取相同的对象。

JavaScript 语言一直都内置了的单例，只是我们不把它们叫做单例，我们称它们为对象字面量，例如：

```javascript
const user = {
  name: "Peter",
  age: 25,
  job: "Teacher",
  greet: function () {
    console.log("Hello!");
  },
};
```

因为 JavaScript 中的每个对象都占用一个唯一的内存位置，当我们调用 user 对象时，我们本质上是返回了该对象的引用。

如果我们试图将 user 变量复制到另一个变量中并修改该变量，例如：

```javascript
const user1 = user;
user1.name = "Mark";
```

我们会看到，两个对象都被修改了，因为在 JavaScript 中，对象是通过引用传递的，而不是值。因此，内存中只有一个对象，例如：

```javascript
// 打印'Mark'
console.log(user.name);
// 打印'Mark'
console.log(user1.name);
// 打印 true
console.log(user === user1);
```

单例模式可以使用构造函数实现，例如：

```javascript
let instance = null;
function User() {
  if (instance) {
    return instance;
  }
  instance = this;
  this.name = "Peter";
  this.age = 25;

  return instance;
}
const user1 = new User();
const user2 = new User();
// 打印 true
console.log(user1 === user2);
```

当调用这个构造函数时，它会检查实例对象是否存在。如果对象不存在，它就将这个变量赋给实例变量。如果对象存在，它只返回那个对象。

单例也可以使用模块模式实现，例如：

```javascript
const singleton = (function () {
  let instance;

  function init() {
    return {
      name: "Peter",
      age: 24,
    };
  }
  return {
    getInstance: function () {
      if (!instance) {
        instance = init();
      }

      return instance;
    },
  };
})();
const instanceA = singleton.getInstance();
const instanceB = singleton.getInstance();
// 打印 true
console.log(instanceA === instanceB);
```

在上面的代码中，我们通过调用 singleton.getInstance 方法来创建一个新实例。如果实例已经存在，则该方法只是返回这个实例，如果实例不存在，则调用 init() 函数创建一个新的实例。

## 工厂模式

工厂模式使用工厂方法创建对象，而不指定所创建对象的确切类或构造函数。

工厂模式用于创建对象，而不公开实例化逻辑。当我们需要根据特定条件生成不同的对象时，可以使用此模式，例如：

```javascript
class Car {
  constructor(options) {
    this.doors = options.doors || 4;
    this.state = options.state || "brand new";
    this.color = options.color || "white";
  }
}
class Truck {
  constructor(options) {
    this.doors = options.doors || 4;
    this.state = options.state || "used";
    this.color = options.color || "black";
  }
}
class VehicleFactory {
  createVehicle(options) {
    if (options.vehicleType === "car") {
      return new Car(options);
    } else if (options.vehicleType === "truck") {
      return new Truck(options);
    }
  }
}
```

这里，我创建了一个 Car 类和一个 Truck 类（带有一些默认值），用于创建新的 Car 和 Truck 对象。我还定义了一个 VehicleFactory 类，基于 options 对象中接收到的 vehicleType 属性创建和返回一个新的对象。

```javascript
const factory = new VehicleFactory();
const car = factory.createVehicle({
  vehicleType: "car",
  doors: 4,
  color: "silver",
  state: "Brand New",
});
const truck = factory.createVehicle({
  vehicleType: "truck",
  doors: 2,
  color: "white",
  state: "used",
});
// 打印 Car {doors: 4, state: "Brand New", color: "silver"}
console.log(car);
// 打印 Truck {doors: 2, state: "used", color: "white"}
console.log(truck);
```

我已经创建了一个新的 VehicleFactory 类的对象工厂。之后，我们可以调用 factory.createVehicle 方法，传入一个 vehicleType 属性值为 car 或 truck 的 options 对象。

## 装饰模式

装饰模式用于扩展对象的功能，而不修改现有的类或构造函数。该模式可用于向对象添加特性，而不修改使用它们的底层代码。

下面是这个模式的一个简单例子：

```javascript
function Car(name) {
  this.name = name;
  // 默认值
  this.color = "White";
}
// 新建一个需要装饰的对象
const tesla = new Car("Tesla Model 3");
// 使用新功能装饰对象
tesla.setColor = function (color) {
  this.color = color;
};
tesla.setPrice = function (price) {
  this.price = price;
};
tesla.setColor("black");
tesla.setPrice(49000);
// 打印 black
console.log(tesla.color);
```

对于这种模式，一个更实际的例子是，比方说，一辆车的价格取决于它有多少功能。如果没有装饰模式，我们将不得不为不同的特性组合创建不同的类，每个类都有计算成本的 cost 方法，例如：

```javascript
class Car() {
}
class CarWithAC() {
}
class CarWithAutoTransmission {
}
class CarWithPowerLocks {
}
class CarWithACandPowerLocks {
}
```

但是使用装饰模式，我们可以创建一个基类 Car，并使用装饰函数将不同配置的成本添加到它的对象中，例如

```javascript
class Car {
  constructor() {
    // 默认值
    this.cost = function () {
      return 20000;
    };
  }
}
// 装饰函数
function carWithAC(car) {
  car.hasAC = true;
  const prevCost = car.cost();
  car.cost = function () {
    return prevCost + 500;
  };
}
// 装饰函数
function carWithAutoTransmission(car) {
  car.hasAutoTransmission = true;
  const prevCost = car.cost();
  car.cost = function () {
    return prevCost + 2000;
  };
}
// 装饰函数
function carWithPowerLocks(car) {
  car.hasPowerLocks = true;
  const prevCost = car.cost();
  car.cost = function () {
    return prevCost + 500;
  };
}
```

首先，我们创建一个创建 Car 对象的基类 Car。然后，为要添加的功能创建装饰，并将 Car 对象作为参数传递。然后，我们重写这个对象的 cost 函数，该函数返回更新后的汽车成本，并向该对象添加一个新属性，表明添加了哪些功能。

要添加新功能，我们可以这样做：

```javascript
const car = new Car();
console.log(car.cost());
carWithAC(car);
carWithAutoTransmission(car);
carWithPowerLocks(car);
```

最后，我们可以像下面这样计算汽车的成本：

```javascript
// 计算汽车的总成本
console.log(car.cost());
```

## 小结

我们已经了解了 JavaScript 中使用的各种设计模式，但还有一些可以用 JavaScript 实现的设计模式我在这里没有涉及。

虽然了解各种设计模式很重要，但同样重要的是不要过度使用它们。在使用设计模式之前，你应该仔细考虑你的问题是否符合设计模式。要知道一个模式是否适合你的问题，你应该研究设计模式以及该设计模式的应用。

## 引用

> https://blog.bitsrc.io/understanding-design-patterns-in-javascript-13345223f2dd?br=ro&
