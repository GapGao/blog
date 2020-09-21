---
title: 前端自动化测试（三）- E2E测试
date: 2020-09-21 12:06:56
tags: [前端, E2E测试测试, Cypress]
---

上一篇博客我们讲了讲前端单元测试及其选型、demo等，我还给团队做了技术分享，这次补上E2E测试实战。

<!--more-->

## 选型

说到自动化 e2e 测试，不得不说到 `Selenium` 和 `PhantomJS`。

1. Selenium 是一个用于 Web 应用程序测试的工具，又名webdriver。Selenium 测试直接运行在浏览器中，就像真正的用户在操作一样。支持的浏览器包括 IE（7, 8, 9, 10, 11），Mozilla Firefox，Safari，Google Chrome，Opera 等。selenium 是一套完整的 web 应用程序测试系统，包含了测试的录制（selenium IDE）,编写及运行（Selenium Remote Control）和测试的并行处理（Selenium Grid）。Selenium 的核心 Selenium Core 基于 JsUnit，完全由 JavaScript 编写，因此可以用于任何支持 JavaScript 的浏览器上。Selenium 可以模拟真实浏览器，自动化测试工具，支持多种浏览器，爬虫中主要用来解决 JavaScript 渲染问题。可以使用 Selenium 与其他测试框架（如 jest）配合使用，效果很不错。

2. 而 PhantomJS 使用无界面的 WebKit 浏览器，是一个基于 WebKit 的服务端 JavaScript API,支持 Web 而不需要浏览器支持，其快速、原生支持各种 Web 标准：Dom 处理，CSS 选择器，JSON 等等。PhantomJS 可以用用于页面自动化、网络监测、网页截屏，以及无界面测试。

PhantomJS 是无界面测试，咱们就先不选它了，枯燥乏味。
Selenium被说的神乎其神，确实功能强大，支持多种主流浏览器，它使用每个浏览器对自动化的原生支持直接调用浏览器，多语言，多平台，且社区庞大，但是学习曲线较高，而且我感觉它只是让你的代码有调用浏览器的能力，并不是一个完整的自动化测试框架，要形成测试框架估计还需要跟多的封装。最重要的是，我发现一个很不错的框架，很适合前端同学使用。

他就是 `Cypres`。而且它不使用Selenium或WebDriver。

说干就干。

## Cypress

打开Cypres官网，直接一行slogan
The web has evolved.
Finally, testing has too
看来这个团队还是很注重发展的。（我刚用的时候是4.x，没过两天就升5了）

为啥说很适合前端同学呢，因为他就是一个npm 的 包，只需`npm install cypress`就可以拥有它，也可以单独下载它的应用。

当然，如果项目不是js的项目，请略过。。

还有一点超级赞，就是它的文档都是带视频的，大大降低了使用难度，良心团队。
![avatar](/img/autotest/20.png)

### 特色功能

Time Travel: Cypress takes snapshots as your tests run. Hover over commands in the Command Log to see exactly what happened at each step.
Cypress在测试运行时拍摄快照。将鼠标悬停在命令日志中的命令上，以查看每一步发生了什么，还能查看当时的页面dom和交互，还可以前后对比。

Debuggability: Stop guessing why your tests are failing. Debug directly from familiar tools like Developer Tools. Our readable errors and stack traces make debugging lightning fast.
测试用例调试友好，直接在浏览器里调试，报错什么的都打印在了浏览器里，和平常开发没什么太大区别

Automatic Waiting: Never add waits or sleeps to your tests. Cypress automatically waits for commands and assertions before moving on. No more async hell.
不需要在代码里加 sleep 或 等待之类的方法，去执行等待接口响应之后的操作，cypress帮我们做了

Spies, Stubs, and Clocks: Verify and control the behavior of functions, server responses, or timers. The same functionality you love from unit testing is right at your fingertips.
没看懂

Network Traffic Control: Easily control, stub, and test edge cases without involving your server. You can stub network traffic however you like.
可以控制网络流量模拟真实场景

Consistent Results: Our architecture doesn’t use Selenium or WebDriver. Say hello to fast, consistent and reliable tests that are flake-free.
不使用Selenium

Screenshots and Videos: View screenshots taken automatically on failure, or videos of your entire test suite when run from the CLI.
查看失败时自动拍摄的屏幕截图，或从CLI运行时整个测试套件的视频，步步有记录，整体有视频。

Cross browser Testing: Run tests within Firefox and Chrome-family browsers (including Edge and Electron) locally and optimally in a Continuous Integration pipeline.
可以在本地火狐或者chrome系的浏览器中直接测试运行，也可以配置Continuous Integration pipeline 集成测试流程

### 安装使用

1. npm install cypress
   嚯，我一看已经5.2.0了，发展有点快距离上次使用不过两三周，但是下载超级慢，我们还是下4吧
   npm i cypress@4

2. 第二步启动 npx cypress open
![avatar](/img/autotest/21.png)
![avatar](/img/autotest/22.png)

可以看到直接弹出了一个cypress的应用，上边有一些examples，还有一些功能按钮，还可以切换浏览器，并且在项目目录下也多了很多文件，能基本看的出，每一个test example就是对应一个cypress/integration下边的一个文件，还有贴心的文档链接地址和技术支持。
可以直接看到右边噼里啪啦的在执行，访问跳转等等
![avatar](/img/autotest/23.png)
并且其中每一步的执行都有详细记录，和‘截屏’与之对应，方便你查看是哪一步出了问题，还可以比较操作前和操作后界面的变化。
甚至可以获取到‘截屏’里边的dom节点，感觉其实是把整个dom都做了备份，保留了交互功能，总之有点高端，没想到是怎么做的。
![avatar](/img/autotest/24.png)
3. 上边都是可视化执行的，启动和执行以及结果都在浏览器中呈现，下面是手动执行或者说静默执行
   npx cypress run
![avatar](/img/autotest/25.png)

发现项目目录里多了好多个video文件，对，没错，这就是cypress录制的整个执行过程，是不是很酷炫
4. Cypress平台
   cypress应用里边有一个 `Runs` tab，这里可以帮你配置生成一个cypres项目，并且生成唯一record key，在静默执行时，使用
cypress run --record --key `your record key`，启动，他就会自动把执行记录发往cypress云平台。

刚一启动，就已经同步到了平台。
![avatar](/img/autotest/26.png)
![avatar](/img/autotest/28.png)
执行结果页也可以查看输出、截图、视频等，不要太方便。

平台上还有一些分析报表的功能帮助我们改进。
![avatar](/img/autotest/27.png)

还有一些和github等工具的对接，集成ci，cd等，感觉功能确实比较完善和强大。

### 我的第一个e2e测试

我们实现一个最基本的测试。来模拟用户的使用。
访问网站-> 输入账号密码 -> 登录 -> 打开一个详情 -> 浏览 -> 退出

注意，写测试用例的时候要 `npx cypress open`启动cypress应用之后，并打开你的测试文件，在测试浏览器里边调试边写
cypress还有配套的功能，让你可以选择某个元素来选中，并自动生成代码，复制到你代码里就完事了，很高效。但是如果像一些class
经常发生变化的节点可能需要考虑另一个方式，就是固定 attr 属性 cypress 支持两种选中。

测试框架集成了 Mocha 和 Chai，所以可以随便用 describe it 断言等等
cypress 的 api 都在 cy 下

ps我感觉这玩意能写个抢票或者刷单软件。
废话不多说，上边了解了那么多强大的功能之后我们来写一个，尝尝

我们在cypress文件夹下写一个demo
ps：fixtures 文件夹下 放一些mock数据源之类的
plugins 下放一些外部插件 https://docs.cypress.io/plugins/
社区里有很多更实际的解决方案

![avatar](/img/autotest/29.png)
![avatar](/img/autotest/30.png)
选中元素，复制自动生成的代码，刷新重新跑，齐活。

```javascript
// 写一个访问我司网站 浏览一下
describe("My First Test", () => {
  beforeEach(() => {
    // reset and seed the database prior to every test
    // cy.exec("npm run db:reset && npm run db:seed");
  });

  it("Login Moka", function (done) {
    cy.visit("https://app.mokahr.com");
    // 类似jq
    cy.get(":nth-child(1) > .sd-Input-container-1MAQN > .sd-Input-input-3dqaS")
      .type("xxxx")
      .should("have.value", "xxxxx");

    cy.get(
      ":nth-child(2) > .sd-Input-container-1MAQN > .sd-Input-input-3dqaS"
    ).type("GAOBOai1314.");


    // 获取cookie
    cy.getCookie("user_email")
      .as("email")
      .then(() => {
        // 登录
        cy.get(".sd_global_focus_controller_class").click().wait(2000);
        console.log(this.email);
        if (this.email) {
          cy.get(".sd_global_focus_controller_class").click().wait(2000);
        }
      });

    // 打开详情
    cy.get(".choose_orgs_login_panel > :nth-child(3) > :nth-child(1)").click();

    cy.get("#\\30  > .info > .candidate-info").click();

    // 查看
    cy.get(".candidate-resume__container")
      .wait(1000)
      .scrollTo(0, 500)
      .wait(1000)
      .scrollTo(0, 1000)
      .wait(1000)
      .scrollTo(0, 500)
      .wait(1000)
      .scrollTo(0, 0);

    cy.get(".candidate-header__util > .sd-Icon-container-1BX72").click();

    // 登出
    cy.get(".hoverdown-btn > .avatar--10 > .avatar--name").trigger("mouseover");
    cy.get(".topbar__hoverdown-item__form").click();
    cy.clearCookies().then(() => done());
  });
});
```

## 配置

文件目录里还有一个叫cypress.json的文件，没错他就是cypress的配置文件。里边可以设置包括projectId，viewPort，文件目录，视频输出目录，浏览器设置，截屏设置，超时设置等。

```json
{
  "viewportWidth": 2000,
  "viewportHeight": 1320,
  "projectId": "xxxx"
}
```

## 总结

通常一次迭代中，QA 会花费大约 20~30%的时间进行回归测试，可见回归测试的重要性，但是很多情况下，由于项目时间紧迫，或者紧急发布等情况，被压缩和牺牲的往往是回归测试，而 e2e AT 正好可以覆盖一部分的回归测试，如果你的 AT case 覆盖率越高，则回归测试的覆盖率也越高，出品也就越稳定，如果 AT 能覆盖绝大部分的回归测试，而 AT 的执行效率又是人肉执行的数倍，那 QA 的工作量就被大大的降低。
Cypress 强烈推荐使用，它是一整套的解决方案，省时省力，开发便捷。
后续有时间研究下Cypress怎么配置CI/CD。

## 参考
> https://www.cypress.io/
> https://docs.cypress.io/
