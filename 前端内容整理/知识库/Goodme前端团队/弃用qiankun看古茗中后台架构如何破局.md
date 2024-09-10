![cover_image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicFRoSkWoKopk57AibBSuCKR1T4Vd58eSSDYDEcMBRF0FuTNpKzOhibDL5YcMOjrjXjrAV3zca6Zh3Yg/0?wx_fmt=jpeg)

#  弃用qiankun！看古茗中后台架构如何破局

陈杰  [ Goodme前端团队 ](javascript:void\(0\);)

__ _ _ _ _

> 温馨Tips：阅读本文大约需要 10 分钟～

##  引言

我们团队之前发布过一篇文章，介绍了古茗前端到底在做什么，当然这只包含了我们团队内做的一部分事情。以端侧来划分，主要包括：中后台 web 端、H5 端、4
端小程序、Electron PC 端、Android/Flutter
客户端，我们需要为这些端侧方向做一些通用能力的解决方案，同时也需要深入每个端侧细分领域做它们特有的技术沉淀。本文主要介绍古茗在中后台技术方向的一些思考和技术沉淀。

##  业务现状

古茗目前有大量的中后台业务诉求，包括：权限系统、会员系统、商品中心、拓展系统、营运系统、财务系统、门店系统、供应链系统等多个子系统，服务于内部产品、技术、运营，及外部加盟商等用户群体，这些子系统分别由不同业务线的同学负责开发和维护，而这些子系统还没有系统性的技术沉淀。随着业务体量和业务复杂度的不断增大，在“降本增效”的大背景下，如何保证业务快速且高质量交付是我们面临的技术挑战。

##  技术演进

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicFWs37lGoibELp3Fia5flwaPvUIrBFAXGibflpc4bKicRB6oWibQ2sTgk153B1tV5P1L29vZzH8yK4AmNA/640?wx_fmt=png)
evolution

如上述技术演进路线图所示，我们的中后台技术架构大致经历了 4
个阶段：当业务起步时，我们很自然的使用了单应用开发模式；当业务发展到一定的体量，多人协同开发变得困难时，我们拆分了多个子系统维护，并使用了 systemjs
来加载子系统资源（微前端模式的雏形）；当遇到有三方库的多版本隔离诉求时，我们又引入了 qiankun
微前端框架来隔离多个子系统，保证各子系统间互不影响；那是什么原因让我们放弃 qiankun，转向完全自研的一套中后台解决方案？

###  弃用 qiankun？

其实准确来说，qiankun 并不能算是一个完整的中后台解决方案，而是一个微前端框架，它在整个技术体系里面只充当子应用管理的角色。我们在技术演进过程中使用了
qiankun 尝试解决子应用的隔离问题，但同时也带来了一些新的问题：某些场景跳转路由后视图无反应；某些三方库集成进来后导致奇怪的
bug...。同时，还存在一些问题无法使用 qiankun 解决，如：子应用入口配置、样式隔离、运维部署、路由冲突、规范混乱、要求资源必须跨域等。

###  探索方向

我们重新思考了古茗中后台技术探索方向是什么，也就是 **中后台技术架构到底要解决什么问题** ，并确定了当下的 2 大方向： **研发效率** 、
**用户体验** 。由此我们推导出了中后台技术探索要做的第一件事情是“ **统一** ”，这也为我们整个架构的基础设计确立了方向。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicFWs37lGoibELp3Fia5flwaPv4MicN7HqjZbJKp4DytsQolmLicvx53yaRjoyXqXnhySEosoEoiasncSXg/640?wx_fmt=png)
goal

##  架构设计

我们的整体架构是围绕着“统一”和“规范” 2 大原则来设计， **目标是提升整个团队的研发效能** 。我们认为好的架构应该是 **边界清晰**
的，而不是一味的往上面堆功能，所以我们思考更多的是，如果没有这个功能，能否把这件事情做好。

###  取个“好”名字

我老板曾说过一个好的（技术）产品要做的第一件事就是取个“好”名字。我们给这套中后台架构方案取名叫「 **Mars** 」，将相关的 NPM
包、组件库、SDK、甚至部署路径都使用 mars
关键字来命名，将这个产品名字深入人心，形成团队内共同的产品认知。这样的好处是可以加强团队对这套技术方案的认同感，以及减少沟通负担，大家一提到
mars，就都知道在说哪一件事。

###  框架设计

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicFWs37lGoibELp3Fia5flwaPveQBQFn8Kwqhe6y9mzRD5nCTx0TcnSS588dlyHbFjhINk6iaxIeiaJSuA/640?wx_fmt=png)
mars-arch

正如上述大图所示，我们是基于微前端的思路做的应用框架设计，所以与市面上大多数的微前端框架的设计思路、分层结构大同小异。这里还是稍微介绍一下整个流程：当用户访问网站时，首先经过网关层，请求到基座应用的资源并渲染基础布局和菜单，当监听路由变化时，加载并渲染路由关联的子应用及页面。

但是，市面上大部分的微前端框架往往需要在基座应用上配置子应用的 ` name ` 、 ` entry ` 、 ` avtiveRule `
等信息，因为框架需要根据这些信息来决定什么时机去加载哪一个子应用，以及如何加载子应用的资源。这就意味着每新增一个子应用，都需要去基座维护这份配置。更要命的是，不同环境的
` entry ` 可能还要根据不同的环境做区别判断。如果遇到本地开发时需要跳转至其他子应用的场景，那也是非常不好的开发体验。所以，这是我们万万不能接受的。

针对这一痛点，我们想到了 2 种解决思路：

  1. 把基座应用的配置放到云端，用 node 作为中间层来维护各子应用的信息，每次新增应用、发布资源后同步更新，问题就是这套方案实现成本比较高，且新增子应用还是需要维护子应用的信息，只是转移到在云端维护了。 
  2. 使用约定式路由及部署路径，当我们识别到一个约定的路由路径时，可以反推它的应用 ID 及部署资源路径，完全 0 配置。很明显，我们选择了这种方案。 

###  约定式路由及部署路径

####  路由约定

我们制定了如下的 **标准 Mars 路由规范** ：

    
    
      /mars/appId/path/some?name=ferret  
       \_/   \_/   \_____/   \_______/  
        |     |       |          |  
       标识  appId   path       query  
    

  1. 路由必须以 ` /mars ` 开头（为了兼容历史路由包袱） 
  2. 其后就是 ` appId ` ，这是子应用的唯一标识 
  3. 最后的 ` path ` 和 ` query ` 部分就是业务自身的路由和参数 

####  部署路径约定

我们制定了如下的 **标准 Mars 子应用部署路径规范** ：

    
    
      https://cdn.example.com/mars/[appId]/[env]/manifest.json  
        \__________________/  \_/   \___/   \_/   \________/  
                  |            |      |      |        |  
               cdn 域名        标识   appId  环境   入口资源清单  
    

从上述部署路径规范可以看出，整个路径就 ` appId ` 和 ` env ` 2 个变量是不确定的，而 ` env ` 可以在发布时确定，因此可由 `
appId ` 推导出完整的部署路径。而根据路由约定，我们可以很容易的从路由中解析出 ` appId ` ，由此就可以拿到完整的 `
manifest.json ` 部署路径 ，并以此获取到整个子应用的入口资源信息。

###  编译应用

虽然制定了上述 2 大规范，但是如何保障规范落地，防止规范腐化也是非常重要的一个问题。我们是通过编译手段来强制约束执行的（毕竟“人”往往是靠不住的😄）。

####  依赖工程化体系

> 提示：Kone 是古茗内部前端工程化的工具产品。

首先，子应用需要配置一个工程配置文件，并注册 ` @guming/kone-plugin-mars `
插件来完成子应用的本地开发、构建、发布等工程化相关的任务。其中：配置项 ` appId ` 就代表约定路由中的 appId 和 部署路径中的
appId，也是子应用的唯一标识。

**工程配置文件：kone.config.json**

    
    
    {  
      "plugins": ["@guming/kone-plugin-mars"],  
      "mars": {  
        "appId": "demo"  
      }  
    }  
    

####  编译流程

然后，子应用通过静态化配置式（json 配置）注册路由，由编译器去解析配置文件，注册路由，以及生成子应用 ` mount ` 、 ` unmount `
生命周期方法。这样实现有以下 3 个好处：

  * 完整的路由 path 由编译生成，可以非常好的保障约定式路由落地 
  * 生命周期方法由编译生成，减少项目中的模板代码，同样可以约束子应用的渲染和卸载按照预定的方式执行 
  * 可以约束不规范的路由 path 定义，例如我们会禁用掉 ` :param ` 形式的动态路由 

**应用配置文件：src/app.json**

    
    
    {  
      "routes": [  
        {  
          "path": "/some/list",  
          "component": "./pages/list",  
          "description": "列表页"  
        },  
        {  
          "path": "/some/detail",  
          "component": "./pages/detail",  
          "description": "详情页"  
        }  
      ]  
    }  
    

> 上述示例最终会生成路由： ` /mars/demo/some/list ` 、 ` /mars/demo/some/detail ` 。

**webpack-loader 实现** ：

解析 ` src/app.json ` 需要通过一个自定义的 webpack-loader 来实现，部分示例代码如下：

    
    
    import path from 'path';  
    import qs from 'qs';  
      
    export default function marsAppLoader(source) {  
      const { appId } = qs.parse(this.resourceQuery.slice(1));  
      let config;  
      try {  
        config = JSON.parse(source);  
      } catch (err) {  
        this.emitError(err);  
        return;  
      }  
      
      const { routes = [] } = config;  
      
      const routePathSet = new Set();  
      const routeRuntimes = [];  
      const basename = `/mars/${appId}`;  
      
      for (let i = 0; i < routes.length; i++) {  
        const item = routes[i];  
        if (routePathSet.has(item.path.toLowerCase())) {  
          this.emitError(new Error(`重复定义的路由 path: ${item.path}`));  
          return;  
        }  
      
        routeRuntimes.push(  
          `routes[${i}] = { ` +  
            `path: ${JSON.stringify(basename + item.path)}, ` +  
            `component: _default(require(${JSON.stringify(item.component)})) ` +  
            `}`  
        );  
        routePathSet.add(item.path.toLowerCase());  
      }  
      
      return `  
    const React = require('react');  
    const ReactDOM = require('react-dom');  
      
    // 从 mars sdk 中引入 runtime 代码  
    const { __internals__ } = require('@guming/mars');  
    const { defineApp, _default } = __internals__;  
      
    const routes = new Array(${routeRuntimes.length});  
    ${routeRuntimes.join('\n')}  
      
    // define mars app: ${appId}  
    defineApp({  
      appId: '${appId}',  
      routes,  
    });  
      
      `.trim();  
    }  
    

将 ` src/app.json ` 作为编译入口并经过此 webpack-loader
编译之后，将自动编译关联的路由组件，创建子应用路由渲染模板，注册生命周期方法等，并最终输出 ` manifest.json ` 文件作为子应用的入口（类似
` index.html ` ），根据入口文件的内容就可以去加载入口的 js、css 资源并触发 ` mount ` 生命周期方法执行渲染逻辑。生成的 `
manifest.json ` 内容格式如下：

    
    
    {  
      "js": [  
        "https://cdn.example.com/mars/demo/prod/app.a0dd6a27.js"  
      ],  
      "css": [  
        "https://cdn.example.com/mars/demo/prod/app.230ff1ef.css"  
      ]  
    }  
    

###  聊聊沙箱隔离

一个好的沙箱隔离方案往往是市面上微前端框架最大的卖点，我们团队内也曾引入 qiankun
来解决子应用间隔离的痛点问题。而我想让大家回归到自己团队和业务里思考一下：“我们团队需要隔离？不做隔离有什么问题”。而我们团队给出的答案是： **不隔离
JS，要隔离 CSS** ，理由如下：

  1. 不隔离 JS 可能会有什么问题： ` window ` 全局变量污染？能污染到哪儿去，最多也就内存泄露，对于现代 B 端应用来说，个别内容泄露几乎可以忽略不计；三方库不能混用版本？如文章开头所提及的，我们要做的第一件事就是 **统一** ，其中就包括统一常用三方库版本，在统一的前提下这种问题也就不存在了。当然也有例外情况，比如高德地图 sdk 在不同子系统需要隔离（使用了不同的 key），针对这种问题我们的策略就是专项解决；当然，最后的理由是一套非常好的 JS 隔离方案实现成本太高了，需要考虑太多的问题和场景，这些问题让我们意识到隔离 JS 带来的实际价值可能不太高。 
  2. 由于 CSS 的作用域是全局的，所以非常容易造成子应用间的样式污染，其次，CSS 隔离是容易实现的，我们本身就基于编译做了很多约束的事情，同样也可以用于 CSS 隔离方案中。实现方案也非常简单，就是通过实现一个 postcss 插件，将子应用中引入的所有 css 样式都加上特有的作用域前缀，例如： 

    
    
    .red {  
      color: red;  
    }  
    

将会编译成：

    
    
    .mars__demo .red {  
      color: red;  
    }  
    

当然，某些场景可能就是需要全局样式，如 antd
弹层内容默认就会在子应用内容区外，造成隔离后的样式失效。针对这种场景，我们的解法是用隔离白名单机制，使用也非常简单，在最前面加上 ` :global `
选择器，编译就会直接跳过，示例：

    
    
    :global {  
      .some-modal-cls {  
        font-size: 14px;  
      }  
    }  
    

将会编译成：

    
    
    .some-modal-cls {  
      font-size: 14px;  
    }  
    

除此之外，在子应用卸载的时候，还会禁用掉子应用的 CSS 样式，这是如何做到的？首先，当加载资源的时候，会找到该资源的 ` CSSStyleSheet `
对象：

    
    
    const link = document.createElement('link');  
    link.setAttribute('href', this.url);  
    link.setAttribute('rel', 'stylesheet');  
    link.addEventListener('load', () => {  
      // 找到当前资源对应的 CSSStyleSheet 对象  
      const styleSheets = document.styleSheets;  
      for (let i = styleSheets.length - 1; i >= 0; i--) {  
        const sheet = styleSheets[i];  
        if (sheet.ownerNode === this.node) {  
          this.sheet = sheet;  
          break;  
        }  
      }  
    });  
    

当卸载资源的时候，将该资源关联的 ` CSSStyleSheet ` 对象的 ` disabled ` 属性设置为 ` true ` 即可禁用样式：

    
    
    if (this.sheet) {  
      this.sheet.disabled = true;  
    }  
    

###  框架 SDK 设计

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicFWs37lGoibELp3Fia5flwaPv4zn2kj4ffQibkO3aDoctjUKU6TibCyxlLVohHCgA07xdPT57t91ahHZA/640?wx_fmt=png)
sdk

框架 SDK 按照使用场景可以归为 3 类，分别是：子应用、基座应用、编译器。同样的遵循我们的一贯原则，如果一个 API 可以满足诉求，就不会提供 2 个
API，尽可能保证团队内的代码风格都是统一的。例如：路由跳转 SDK 只提供唯一 API，并通过编译手段禁用掉其他路由跳转方式（如引入 ` react-
router-dom ` 的 API 会报错）：

    
    
    import { mars } from '@guming/mars';  
      
    // 跳转路由：/mars/demo/some/detail?a=123  
    mars.navigate('/mars/demo/some/detail', {   
      params: { a: '123' }   
    });  
      
    // 获取路由参数  
    const { pathname, params } = mars.getLocation();  
    // pathname: /mars/demo/some/detail  
    // params: { a: '123' }  
    

当然，我们也会根据实际情况提供一些便利的 API，例如：跳转路由要写完整的 ` /mars/[appId] `
路由前缀太繁琐，所以我们提供了一个语法糖来减少样板代码，在路由最前面使用 ` : ` 来代替 ` /mars/[appId] `
前缀（仅在当前子应用内跳转有效）：

    
    
    import { mars } from '@guming/mars';  
      
    // 跳转路由：/mars/demo/some/detail?a=123  
    mars.navigate(':/some/detail', {   
      params: { a: '123' }   
    });  
    

另外，值得一提的是在基座应用上使用的一个 API ` bootstrap() ` ，得益于这个 API
的设计，我们可以快速创建多个基座应用（不同域名），还能使用这个 API 在本地开发的时候启动一个基座来模拟本地开发环境，提升开发体验。

###  本地开发体验

####  开发模拟器

为更好的支持本地开发环境，我们提供了一套本地开发模拟器，在子应用启动本地服务的时候，会自动启动一个模拟的基座应用，拥有和真实基座应用几乎一样的布局，运行环境，集成的登录逻辑等。除此之外，开发模拟器还提供了辅助开发的「debug
小组件」，比如通过 debug 工具可以动态修改本地开发代理规则，保存之后立即生效。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicFWs37lGoibELp3Fia5flwaPvHL8CpI7bVDXJMX2ACdf0UYQ6LRRUNACjaqV6sDr407fPACPnZ0dfFg/640?wx_fmt=png)
simulator

####  IDE 支持

为了提升开发体验，我们分别开发了 Webstorm 插件 和 VSCode 插件服务于 Mars
应用，目前支持路由组件配置的路径补全、点击跳转、配置校验等功能。此外，我们会为配置文件提供 json schema，配置后将会获得 IDE
的自动补全能力和配置校验能力。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicFNhKRy6dUUpFtmibYG8nLhUBNCM3vWOaj99ibSibP0hnzhUZwLUylP3VU3WIneYtnicibbzyrABWxp5jg/640?wx_fmt=other)

##  历史项目迁移

技术架构演进对于业务项目来说最大的问题在于，如何完成历史项目向新架构的迁移改造？我们团队投入 2 个人花了 2 个月时间将 12
个历史项目全部迁移至最新的架构上，这里分享一些我们在迁移过程中的经验。

###  定目标

首先要确定历史项目迁移这件事情一定是要做的，这是毋庸置疑的，而且要越快越好。所以我们要做的第一件事就是制定迁移计划，最后确定投入 2 个人力，大约花 2
个月时间将历史项目全部迁移至新的架构。第二件事情就是确定改造的范围（ **确定边界，不做非目标范围内的改造** ），对于我们的业务现状来说，主要包括：

  * 统一 ` react ` 、 ` react-dom ` 版本为 ` 17.0.2 `
  * 统一 ` antd ` 版本为 ` 4.24.8 `
  * 统一路由 
  * 统一接入 request 请求库 
  * 统一接入工程化体系 
  * 统一环境变量 

###  梳理 SOP

因为迁移的流程不算简单，迁移要做的事情还挺多的，所以接下来要做的一件事就是 **梳理迁移流程 SOP** ，SOP
文档要细化到每种可能的场景，以及遇到问题对应的解法，让后续项目的迁移可以傻瓜式的按照标准流程去操作即可。我们的做法是，先以一个项目作为试点，一边迁移一边梳理
SOP，如果在迁移其他项目中发现有遗漏的场景，再持续补充这份 SOP 文档。

例如：之前项目中使用了 dva 框架，但是它的 ` router ` 和 ` model `
是耦合的，这样就无法使用我们制定的统一路由方案，对此我们的解法是，通过 hack dva 的源代码，将 ` model `
前置注入到应用中，完成与路由的解耦。

###  上线方案

由于业务迭代频繁，所以我们代码改造持续的时间不能太长，否则要多次合并代码冲突，而我们的经验就是，项目改造从拉分支到发布上线，要在 1
周内完成。当然，整个上线过程还遇到许多需要解决的问题，比如在测试参与较少的情况下如何保障代码质量，包括：业务回归的策略，回滚策略，信息同步等等。

##  总结

之前看到 [ Umi 4 设计思路文字稿
](https://mp.weixin.qq.com/s?__biz=MjM5NDgyODI4MQ==&mid=2247484533&idx=1&sn=9b15a67b88ebc95476fce1798eb49146&scene=21#wechat_redirect)
里面有句话我觉得特别有道理：“ **社区要开放，团队要约束** ”，我们团队也在努力践行“ **团队约束** ”这一原则，因为它为团队带来的收益是非常高的。

没有最完美的方案，只有最适合自己的方案，以上这套架构方案只是基于当下古茗前端团队现状做的选择后的结果，可能并不适合每个团队，希望本文的这些思考和技术沉淀能对您有所帮助和启发。

##  最后

关注公众号「Goodme前端团队」，获取更多干货实践，欢迎交流分享~

  

预览时标签不可点

微信扫一扫  
关注该公众号





****



****



×  分析

  收藏

