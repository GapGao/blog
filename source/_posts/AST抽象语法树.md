---
title: AST抽象语法树
date: 2020-05-17 17:27:51
tags: [javascript, AST]
---

AST-抽象语法树

同事写了一个脚本，把所有引入某一组件的 js 文件都找出来了，我问他是咋做的，字符串正则匹配吗，他说用 AST 做的，今天我们就来研究一下。
我也是人生第一次。

<!--more-->

## 先看看概念

抽象语法树（abstract syntax code，AST）是源代码的抽象语法结构的树状表示，树上的每个节点都表示源代码中的一种结构，这所以说是抽象的，是因为抽象语法树并不会表示出真实语法出现的每一个细节，比如说，嵌套括号被隐含在树的结构中，并没有以节点的形式呈现。抽象语法树并不依赖于源语言的语法，也就是说语法分析阶段所采用的上下文无文文法，因为在写文法时，经常会对文法进行等价的转换（消除左递归，回溯，二义性等），这样会给文法分析引入一些多余的成分，对后续阶段造成不利影响，甚至会使合个阶段变得混乱。因些，很多编译器经常要独立地构造语法分析树，为前端，后端建立一个清晰的接口。

抽象语法树在很多领域有广泛的应用，比如浏览器，智能编辑器，编译器。

其实，我也没看懂。。

> 引用 <https://blog.csdn.net/philosophyatmath/article/details/38170131>

## AST 有什么用

![avatar](/img/ast/1.png)

1. 解析(PARSE)：将代码字符串解析成抽象语法树。

2. 转换(TRANSFORM)：对抽象语法树进行转换操作。

3. 生成(GENERATE): 根据变换后的抽象语法树再生成代码字符串。

javascript 程序是由一系列的字符组成的，每一个字符都有一些含义，比如我们可以使用匹配的字符([], {}, ()), 或一些其他成对的字符('', "")和代码缩进让程序解析更加简单，但是对计算机并不适用，这些字符在内存中仅仅是个数值，但是计算机并不知道一个程序内部有多少个变量这些高级问题，这个时候我们需要寻找一些能让计算机理解的方式，这个时候，抽象语法树诞生了。

## Parse

我们通过上面知道，Babel 的工作的第一步是 解析操作，将代码字符串解析成抽象语法树，那么抽象语法树就是在解析过程中产生的。其实解析又可以分成两个步骤： 1.：将整个代码字符串分割成 语法单元数组。 2.：在分词结果的基础之上分析 语法单元之间的关系。

那么 JS 代码中有哪些语法单元呢？

1. 空白。JS 中连续的空格，换行，缩进等这些如果不在字符串里面，就没有任何实际的意义，因此我们可以将连续的空白组合在一起作为一个语法单元。
2. 注释。行注释或块注释，对于编写人或维护人注释是有意义的，但是对于计算机来说知道这是个注释就可以了，并不关心注释的含义，因此我们可以将注释理解为一个不可拆分的语法单元。
3. 字符串。对计算机而言，字符串的内容会参与计算或显示，因此有可以为一个语法单元。
4. 数字。JS 中有 16，10，8 进制以及科学表达式等语法，因此数字也可以理解一个语法单元。
5. 标识符。没有被引号括起来的连续字符，可包含字母 \_, \$ 及数字，或 true, false 等这些内置常量，或 if，return，function 等这些关键字。
6. 运算符： +, -, \*, /, >, < 等。
7. 还有一些其他的，比如括号，中括号，大括号，分号，冒号，点等等。

程序代码本身可以被映射成为一棵语法树，而通过操纵语法树，我们能够精准的获得程序代码中的某个节点。例如声明语句，赋值语句，而这是用正则表达式所不能准确体现的地方。所以在正则表达式做不了的时候考虑下 AST 哦。

这里有两个在线 AST 解析工具

1. <https://esprima.org/demo/parse.html>
2. <https://astexplorer.net/>

个人认为第二个网站更牛批一些，里边有各种语言的 AST 解析和一些三方库，并且还有一些可视化的模块划分，鼠标悬浮，就能清晰的看出，这块数据代表的是哪块代码。

## 我的第一次 AST 解析

```javascript
/**
 * 这是我的第一个 AST 解析
 */

var gap = "g" + "a" + "p";
```

这里边包含了注释，变量定义，字符串，空格，运算符等等，我们来看看 AST 能解析出啥。（通过<https://astexplorer.net/>这个解析的）

```json
{
  "type": "File",
  "start": 0,
  "end": 54,
  "loc": {
    "start": {
      "line": 1,
      "column": 0
    },
    "end": {
      "line": 6,
      "column": 0
    }
  },
  "errors": [],
  "program": {
    "type": "Program",
    "start": 0,
    "end": 54,
    "loc": {
      "start": {
        "line": 1,
        "column": 0
      },
      "end": {
        "line": 6,
        "column": 0
      }
    },
    "sourceType": "module",
    "interpreter": null,
    "body": [
      {
        "type": "VariableDeclaration",
        "start": 27,
        "end": 53,
        "loc": {
          "start": {
            "line": 5,
            "column": 0
          },
          "end": {
            "line": 5,
            "column": 26
          }
        },
        "declarations": [
          {
            "type": "VariableDeclarator",
            "start": 31,
            "end": 52,
            "loc": {
              "start": {
                "line": 5,
                "column": 4
              },
              "end": {
                "line": 5,
                "column": 25
              }
            },
            "id": {
              "type": "Identifier",
              "start": 31,
              "end": 34,
              "loc": {
                "start": {
                  "line": 5,
                  "column": 4
                },
                "end": {
                  "line": 5,
                  "column": 7
                },
                "identifierName": "gap"
              },
              "name": "gap"
            },
            "init": {
              "type": "BinaryExpression",
              "start": 37,
              "end": 52,
              "loc": {
                "start": {
                  "line": 5,
                  "column": 10
                },
                "end": {
                  "line": 5,
                  "column": 25
                }
              },
              "left": {
                "type": "BinaryExpression",
                "start": 37,
                "end": 46,
                "loc": {
                  "start": {
                    "line": 5,
                    "column": 10
                  },
                  "end": {
                    "line": 5,
                    "column": 19
                  }
                },
                "left": {
                  "type": "StringLiteral",
                  "start": 37,
                  "end": 40,
                  "loc": {
                    "start": {
                      "line": 5,
                      "column": 10
                    },
                    "end": {
                      "line": 5,
                      "column": 13
                    }
                  },
                  "extra": {
                    "rawValue": "g",
                    "raw": "'g'"
                  },
                  "value": "g"
                },
                "operator": "+",
                "right": {
                  "type": "StringLiteral",
                  "start": 43,
                  "end": 46,
                  "loc": {
                    "start": {
                      "line": 5,
                      "column": 16
                    },
                    "end": {
                      "line": 5,
                      "column": 19
                    }
                  },
                  "extra": {
                    "rawValue": "a",
                    "raw": "'a'"
                  },
                  "value": "a"
                }
              },
              "operator": "+",
              "right": {
                "type": "StringLiteral",
                "start": 49,
                "end": 52,
                "loc": {
                  "start": {
                    "line": 5,
                    "column": 22
                  },
                  "end": {
                    "line": 5,
                    "column": 25
                  }
                },
                "extra": {
                  "rawValue": "p",
                  "raw": "'p'"
                },
                "value": "p"
              }
            }
          }
        ],
        "kind": "var",
        "leadingComments": [
          {
            "type": "CommentBlock",
            "value": "*\n * 这是我的第一个 AST 解析\n ",
            "start": 0,
            "end": 25,
            "loc": {
              "start": {
                "line": 1,
                "column": 0
              },
              "end": {
                "line": 3,
                "column": 3
              }
            }
          }
        ]
      }
    ],
    "directives": []
  },
  "comments": [
    {
      "type": "CommentBlock",
      "value": "*\n * 这是我的第一个 AST 解析\n ",
      "start": 0,
      "end": 25,
      "loc": {
        "start": {
          "line": 1,
          "column": 0
        },
        "end": {
          "line": 3,
          "column": 3
        }
      }
    }
  ]
}
```

我的妈呀真是有够长的，我们简单看看都有些什么内容。
首先是 File 对象，即当前文件，有文本的起点位置重点位置，起点行终点行（loc）之类的。然后分为`program`和`comments`。
咱们先说`comment`s，它是一个数组，当有多个注释的块的时候，它会变多，并且标记好不同的位置信息，及内容。
`program`是个对象，应该是整个文件都属于它的范畴，里边的`body`是个数组，它其实包含着我们每一个代码块，可以看到里边的，`type:VariableDeclaration`，其实就是表示一个变量声明代码块。一些基本的位置信息之后是`declaration`，具体的定义信息，变量名字模块`id`，`init`里是递归嵌套的具体的操作运算类`BinaryExpression`，里边会有`left`个`right`分别代表运算符两边的变量，而`left`仍然可以是`BinaryExpression`类，这样就递归下去了，直至将整个运算都表示完全。`body`里每一个代码块还有一个`leadingComments` 和 `trailingComments`分别代表自己前边的注释和后边的注释，都是数组。

这样这么简单的一段代码就被解析成这么复杂的 json 给展示出来了。
虽然复杂，但是绝对最低粒度的展示了代码的本质。

## 纯文本转 AST 的实现

上述从纯文本转换成树形结构的数据，每个条目和树中的节点一一对应。
那么纯文本是如何转成 AST 的呢？

### 第一步：词法分析，也叫扫描 scanner

它读取我们的代码，然后把它们按照预定的规则合并成一个个的标识 tokens。同时，它会移除空白符、注释等。最后，整个代码将被分割进一个 tokens 列表（或者说一维数组）。<https://esprima.org/demo/parse.html>这个里边可以解析的到。

```javascript
const a = 'gap';

// 转换成

[{value: 'const', type: 'keyword'}, {value: 'a', type: 'identifier'}, ...]
```

当词法分析源代码的时候，它会一个一个字母地读取代码，所以很形象地称之为扫描 - scans。当它遇到空格、操作符，或者特殊符号的时候，它会认为一个话已经完成了。

### 第二步：语法分析，也称解析器

它会将词法分析出来的数组转换成树形的形式，同时，验证语法。语法如果有错的话，抛出语法错误。

```javascript
[{value: 'const', type: 'keyword'}, {value: 'a', type: 'identifier'}, ...]

// 语法分析后的树形形式

{

  type: "VariableDeclarator",

  id: {

  type: "Identifier",

  name: "a"

  },
  ...
}
```

当生成树的时候，解析器会删除一些没必要的标识 tokens（比如：不完整的括号），因此 AST 不是 100% 与源码匹配的。

> 解析器 100%覆盖所有代码结构生成树叫做 CST（具体语法树）

## 看看别人怎么吹 AST 的

Javascript 就像一台精妙运作的机器，我们可以用它来完成一切天马行空的构思。

我们对 javascript 生态了如指掌，却常忽视 javascript 本身。这台机器，究竟是哪些零部件在支持着它运行？

AST 在日常业务中也许很难涉及到，但当你不止于想做一个工程师，而想做工程师的工程师，写出 vue、react 之类的大型框架，或类似 webpack、vue-cli 前端自动化的工具，或者有批量修改源码的工程需求，那你必须懂得 AST。AST 的能力十分强大，且能帮你真正吃透 javascript 的语言精髓。

事实上，在 javascript 世界中，你可以认为抽象语法树(AST)是最底层。 再往下，就是关于转换和编译的“黑魔法”领域了。

> <https://segmentfault.com/a/1190000016231512?utm_source=tag-newest>

## 开搞：利用 AST 实现一个组件引用文件查找的脚本

### 依赖工具安装

这里我们使用`acorn`这个库，这个还是比较权威的。

### 试用一下功能

安装好依赖之后，创建一个 index.js 文件

```javascript
let acorn = require("acorn");
console.log(acorn.parse("1 + 1"));
```

运行 node index.js 输出

```json
Node {
  type: 'Program',
  start: 0,
  end: 5,
  body:
   [ Node {
       type: 'ExpressionStatement',
       start: 0,
       end: 5,
       expression: [Object] } ],
  sourceType: 'script' }
```

这种方式是在代码中去解析一个 string，即`纯文本转 AST`。
你也可以执行命令行，`npx acorn ./index.js`
输出：

```json
{
  "type": "Program",
  "start": 0,
  "end": 65,
  "body": [
    {
      "type": "VariableDeclaration",
      "start": 0,
      "end": 29,
      "declarations": [
        {
          "type": "VariableDeclarator",
          "start": 4,
          "end": 28,
          "id": {
            "type": "Identifier",
            "start": 4,
            "end": 9,
            "name": "acorn"
          },
          "init": {
            "type": "CallExpression",
            "start": 12,
            "end": 28,
            "callee": {
              "type": "Identifier",
              "start": 12,
              "end": 19,
              "name": "require"
            },
            "arguments": [
              {
                "type": "Literal",
                "start": 20,
                "end": 27,
                "value": "acorn",
                "raw": "\"acorn\""
              }
            ]
          }
        }
      ],
      "kind": "let"
    },
    {
      "type": "ExpressionStatement",
      "start": 30,
      "end": 64,
      "expression": {
        "type": "CallExpression",
        "start": 30,
        "end": 63,
        "callee": {
          "type": "MemberExpression",
          "start": 30,
          "end": 41,
          "object": {
            "type": "Identifier",
            "start": 30,
            "end": 37,
            "name": "console"
          },
          "property": {
            "type": "Identifier",
            "start": 38,
            "end": 41,
            "name": "log"
          },
          "computed": false
        },
        "arguments": [
          {
            "type": "CallExpression",
            "start": 42,
            "end": 62,
            "callee": {
              "type": "MemberExpression",
              "start": 42,
              "end": 53,
              "object": {
                "type": "Identifier",
                "start": 42,
                "end": 47,
                "name": "acorn"
              },
              "property": {
                "type": "Identifier",
                "start": 48,
                "end": 53,
                "name": "parse"
              },
              "computed": false
            },
            "arguments": [
              {
                "type": "Literal",
                "start": 54,
                "end": 61,
                "value": "1 + 1",
                "raw": "\"1 + 1\""
              }
            ]
          }
        ]
      }
    }
  ],
  "sourceType": "script"
}
```

是对整个 js 文件进行 AST 解析。

### 举个 🌰

我们要找出来所有引用了`lodash`库的文件，下边是个例子
本例是一个普通 js，如果是 react、vue、es6、ts 代码，还需要进行 babel 转换之后再进行 AST 解析。

```javascript
const _ = require("lodash");

const xxx = { a: { b: { c: 3 } } };

const yyy = _.get(a, ["b", "c"]);
```

所以我们要写一个脚本，要对解析的内容进行判断区别，然后记录文件名字。

#### 首先我们要搞一些测试文件或文件夹：如下

```javascript
- src
 |- dir
 | |- dir
 | | |- yyy.js // 含有lodash
 | |- demo.js // 含有lodash
 | |- xxx.css
 | |- xxx.js // 不含有
 |- index.js // 含有lodash
 |- script.js // 脚本
```

#### 然后要写一个遍历文件读取的脚本

```javascript
const path = require("path");
const fs = require("fs");

const rootPathName = path.resolve(__dirname, "./src");
const outputPathName = path.resolve(__dirname, "./hasLodashFileList.txt");

function traverseDir(pathName) {
  // 读取文件夹 输出目录
  fs.readdir(pathName, function (err, files) {
    // 遍历文件 判断是文件还是文件夹，如果是文件是否是js文件
    // 如果是文件夹 则 递归深入遍历
    files.forEach((fileName) => {
      const filePath = path.join(pathName, fileName);
      fs.stat(filePath, function (err, data) {
        if (data.isDirectory()) {
          // 如果是文件夹 则递归
          traverseDir(filePath);
        } else if (data.isFile() && fileName.endsWith(".js")) {
          // 如果是文件 且是js文件则 AST 解析 检查是否引入了 lodash
          fs.readFile(filePath, "utf-8", function (err, data) {
            const result = false; // 需要在这里进行AST解析
            if (result) {
              // 将结果输出到一个文本里
              fs.appendFileSync(outputPathName, `${filePath}\n`);
            }
          });
        }
      });
    });
  });
}

traverseDir(rootPathName);
```

#### log 一下解析内容

```javascript
fs.readFile(filePath, "utf-8", function (err, data) {
  console.log(acorn.parse(data));
});

// 输出
Node {
  type: 'Program',
  start: 0,
  end: 101,
  body:
   [ Node {
       type: 'VariableDeclaration',
       start: 0,
       end: 28,
       declarations: [Array],
       kind: 'const' },
     Node {
       type: 'VariableDeclaration',
       start: 30,
       end: 65,
       declarations: [Array],
       kind: 'const' },
     Node {
       type: 'VariableDeclaration',
       start: 67,
       end: 100,
       declarations: [Array],
       kind: 'const' } ],
  sourceType: 'script' }

// 发现有些遍历被包含住了看不到，所以
console.log(JSON.stringify(acorn.parse(data)));
// 解析之后，按照type argument 等字段的判断 我们就可以判断出此文件是否引入了 lodash
```

#### 最终代码

```javascript
const path = require("path");
const fs = require("fs");
const acorn = require("acorn");

const rootPathName = path.resolve(__dirname, "./src");
const outputPathName = path.resolve(__dirname, "./hasLodashFileList.txt");

function traverseDir(pathName) {
  fs.readdir(pathName, function (err, files) {
    files.forEach((fileName) => {
      const filePath = path.join(pathName, fileName);
      fs.stat(filePath, function (err, data) {
        if (data.isDirectory()) {
          // 如果是文件夹 则递归
          traverseDir(filePath);
        } else if (data.isFile() && fileName.endsWith(".js")) {
          // 如果是文件 且是js文件则 AST 解析 检查是否引入了 lodash
          fs.readFile(filePath, "utf-8", function (err, data) {
            const ast = acorn.parse(data);
            const result = ast.body.some((line) => {
              // 找到类型是变量定义的那一行
              if (line.type === "VariableDeclaration") {
                // 遍历每一个 变量定义 块 判断是否 引入(require) 了 lodash
                return line.declarations.some((item) => {
                  if (
                    item.type === "VariableDeclarator" &&
                    item.init.callee &&
                    item.init.callee.name === "require" &&
                    item.init.arguments &&
                    item.init.arguments.some((arg) => arg.value === "lodash")
                  ) {
                    return true;
                  }
                  return false;
                });
              }
              return false;
            });
            if (result) {
              fs.appendFileSync(outputPathName, `${filePath}\n`);
            }
          });
        }
      });
    });
  });
}

traverseDir(rootPathName);
```
