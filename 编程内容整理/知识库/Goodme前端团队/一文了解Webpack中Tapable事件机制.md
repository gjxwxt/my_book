![cover_image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1CtskVib1QOxNreeVaAnRfhu7zh4QWJvQWOGm80jZS6Eccjc2mnqOLtiaOA/0?wx_fmt=jpeg)

#  一文了解Webpack中Tapable事件机制

原创  刘锦泉  [ Goodme前端团队 ](javascript:void\(0\);)

__ _ _ _ _

##  引言

` Webpack ` 是前端工程化常用的静态模块打包工具。在合适的时机通过 ` Webpack ` 提供的 API 改变输出结果，使 ` Webpack
` 可以执行更广泛的任务，拥有更强的构建能力。

` Webpack ` 的插件机制本质上是一种事件流的机制，它的工作流程就是将各个插件串联起来，而实现这一切的核心就是 ` Tapable ` ， `
Webpack ` 中最核心的负责编译的 ` Compiler ` 和负责创建bundles的 ` Compilation ` 都是 ` Tapable `
的实例。

本文将介绍 ` Tapable ` 的基本使用以及底层实现。

##  Tapable

` Tapable ` 是一个类似于 ` Node.js ` 中的 ` EventEmitter ` 的库，但它 **更专注于自定义事件的触发和处理**
。通过 ` Tapable ` 我们可以注册自定义事件，然后在适当的时机去执行自定义事件。这个和我们所熟知的生命周期函数类似，在特定的时机去触发。

我们先看一个 简单 ` Tapable ` 的 例子:

    
    
    const { SyncHook } = require("tapable");  
      
    // 实例化 钩子函数 定义形参  
    const syncHook = new SyncHook(["name"]);  
      
    //通过tap函数注册事件  
    syncHook.tap("同步钩子1", (name) => {  
      console.log("同步钩子1", name);  
    });  
      
    //同步钩子 通过call 发布事件  
    syncHook.call("古茗前端");  
      
    

通过上面的例子，我们大致可以将 ` Tapable ` 的使用分为以下三步:

  * 实例化钩子函数 
  * 事件注册 
  * 事件触发 

**事件注册**

  * 同步的钩子要用 ` tap ` 方法来注册事件 
  * 异步的钩子可以像同步方式一样用 ` tap ` 方法来注册，也可以用 ` tapAsync ` 或 ` tapPromise ` 异步方法来注册。 
    * ` tapAsync ` ：使用用 ` tapAsync ` 方法来注册 ` hook ` 时，必须调用callback 回调函数。 
    * ` tapPromise ` ：使用 ` tapPromise ` 方法来注册 ` hook ` 时，必须返回一个 ` pormise ` ，异步任务完成后 ` resolve ` 。 

**事件触发**

  * 同步的钩子要用 ` call ` 方法来触发 
  * 异步的钩子需要用 ` callAsync ` 或 ` promise ` 异步方法来触发。 
    * ` callAsync ` ：当我们用 ` callAsync ` 方法来调用 ` hook ` 时，第二个参数是一个回调函数，回调函数的参数是执行任务的最后一个返回值 
    * ` promise ` ：当我们用 ` promise ` 方法来调用 ` hook ` 时，需要使用 ` then ` 来处理执行结果，参数是执行任务的最后一个返回值。 

###  Tapable Hook 钩子

tapable 内置了 9 种 hook 。分为 ` 同步 ` 、 ` 异步 ` 两种执行方式。异步执行 Hook 又可以分为 ` 串行 ` 执行和 `
并行 ` 执行。除此之外，hook 可以根据执行机制分为 ` 常规 ` ` 瀑布模式 ` ` 熔断模式 ` ` 循环模式 ` 四种执行机制。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1CtGpgFYb3q4K6bX9VXMiaW4akRAPZLrLzTT1PrlmEicNdbkCoiaOicD96jfQ/640?wx_fmt=png)
image.png

####  同步钩子

> 同步钩子顾名思义：同步执行，上一个钩子执行完才会执行下一个钩子。

示例代码如下：

    
    
    const { SyncHook } = require("tapable");  
      
    // 实例化 钩子函数 定义形参  
    const syncHook = new SyncHook(["name"]);  
      
    //通过tap函数注册事件  
    syncHook.tap("同步钩子1", (name) => {  
      console.log("同步钩子1", name);  
    });  
      
    //该监听函数有返回值  
    syncHook.tap("同步钩子2", (name) => {  
      console.log("同步钩子2", name);  
    });  
      
      
    //同步钩子 通过call 发布事件  
    syncHook.call("古茗前端");  
    

执行结果如下所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1CteMiaVRgaRa4v2aVWjwdxtVvs6xs3kahN1mRdFqDa1bwe6Z2wVQXAe7g/640?wx_fmt=png)
image.png

####  异步钩子

> 异步钩子分为: 串行执行和并行执行。在串行执行中，如果上一个钩子没有调用callback 回调函数，下一个钩子将不会触发对应的事件监听。

示例代码如下：

    
    
    const { AsyncParallelHook, AsyncSeriesHook } = require("tapable");  
      
      
    const asyncParallelHook = new AsyncParallelHook(["name"]);  
      
    const asyncSeriesHook = new AsyncSeriesHook(["name"]);  
      
      
    //通过tap函数注册事件  
    asyncParallelHook.tapAsync("异步并行钩子1", (name, callback) => {  
      setTimeout(() => {  
        console.log("异步并行钩子1", name);  
      }, 3000);  
    });  
      
    //该监听函数有返回值  
    asyncParallelHook.tapAsync("异步并行钩子2", (name, callback) => {  
      setTimeout(() => {  
        console.log("异步并行钩子2", name);  
      }, 1500);  
    });  
      
    //通过tap函数注册事件  
    asyncSeriesHook.tapAsync("异步串行钩子1", (name, callback) => {  
      setTimeout(() => {  
        console.log("异步串行钩子1", name);  
      }, 3000);  
    });  
      
    //该监听函数有返回值  
    asyncSeriesHook.tapAsync("异步串行钩子2", (name, callback) => {  
      setTimeout(() => {  
        console.log("异步串行钩子2", name);  
      }, 1500);  
    });  
      
      
    // 异步并行钩子 通过 callAsync 发布事件  
    asyncParallelHook.callAsync("古茗前端", () => {  
      console.log("1111");  
      return "1122";  
    });  
      
    // 异步串行钩子 通过 callAsync 发布事件  
    asyncSeriesHook.callAsync("古茗前端", () => {  
      console.log("1111");  
      return "1122";  
    });  
      
    

控制台输出结果如下图所示:

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1Ctic28aLW0ickOD15fBicL3ibovXuTNGB6AruiceCuspCibwaMicQia6gczxLM9A/640?wx_fmt=png)
image.png

串行钩子1没有调用callback, 所以串行钩子2没有触发。添加callback后，控制台输出结果：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1CtkBWq67V9h7V0jO1I15nT3Shch6E7o2hYkBhuLUny0AiasAIUuwnjrYw/640?wx_fmt=png)
image.png

####  熔断类

> ` AsyncSeriesBailHook ` 是一个异步串行、熔断类型的 ` Hook `
> 。在串行的执行过程中，只要其中一个有返回值，后面的就不会执行了。

示例代码如下：

    
    
    const { SyncBailHook, AsyncSeriesBailHook } = require("tapable");  
      
    const syncBailHook = new SyncBailHook(["name"]);  
    const asyncSeriesBailHook = new AsyncSeriesBailHook(["name"]);  
      
    syncBailHook.tap("保险类同步钩子1", (name) => {  
      console.log("保险类同步钩子1", name);  
    });  
      
    syncBailHook.tap("保险类同步钩子2", (name) => {  
      console.log("保险类同步钩子2", name);  
      return "有返回值";  
    });  
      
    syncBailHook.tap("保险类同步钩子3", (name) => {  
      console.log("保险类同步钩子3", name);  
    });  
      
    asyncSeriesBailHook.tapAsync("保险类异步串行钩子1", (name, callback) => {  
      setTimeout(() => {  
        console.log("保险类异步串行钩子1", name);  
        callback();  
      }, 3000);  
    });  
      
    asyncSeriesBailHook.tapAsync("保险类2步串行钩子1", (name, callback) => {  
      setTimeout(() => {  
        console.log("保险类异步串行钩子2", name);  
        callback("有返回值");  
      }, 2000);  
    });  
      
    asyncSeriesBailHook.tapAsync("保险类异步串行钩子3", (name) => {  
      setTimeout(() => {  
        console.log("保险类异步串行钩子3", name);  
      }, 1000);  
    });  
      
    syncBailHook.call("古茗前端");  
    asyncSeriesBailHook.callAsync("古茗前端", (result) => {  
      console.log("result", result);  
    });  
      
    

控制台输出结果如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1Ctv6GQXUuPGsuLVlLsBp18xyfT6dnia3QwkBTJqZemJIBUrL3U3XR4clw/640?wx_fmt=png)
image.png

####  循环类

` SyncLoopHook ` 是一个同步、循环类型的 ` Hook ` 。循环类型的含义是不停的循环执行事件函数，直到所有函数结果 ` result
=== undefined ` ，不符合条件就调头重新开始执行。

示例代码：

    
    
    const { SyncLoopHook } = require("tapable");  
      
    const syncLoopHook = new SyncLoopHook(["name"]);  
      
    let count = 4;  
      
    syncLoopHook.tap("循环钩子1", (name) => {  
      console.log("循环钩子1", count);  
      return count <= 3 ? undefined : count--;  
    });  
      
    syncLoopHook.tap("循环钩子2", (name) => {  
      console.log("循环钩子2", count);  
      return count <= 2 ? undefined : count--;  
    });  
      
    syncLoopHook.tap("循环钩子3", (name) => {  
      console.log("循环钩子3", count);  
      return count <= 1 ? undefined : count--;  
    });  
      
      
    syncLoopHook.call();  
      
    

控制台输出结果：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1CtUapqaqaBy3SibEcqtv0QdN52MzD6E73bqyAJWTs2x5JzpnDRR5PEDicQ/640?wx_fmt=png)
image.png

####  瀑布类

` AsyncSeriesWaterfallHook ` 是一个异步串行、瀑布类型的 ` Hook ` 。如果前一个事件函数的结果 ` result !==
undefined ` ，则 ` result ` 会作为后一个事件函数的第一个参数（也就是上一个函数的执行结果会成为下一个函数的参数）。

示例代码：

    
    
    const { AsyncSeriesWaterfallHook, SyncWaterfallHook } = require("tapable");  
      
    const syncWaterfallHook = new SyncWaterfallHook(["name"]);  
      
    const asyncSeriesWaterfallHook = new AsyncSeriesWaterfallHook(["name"]);  
      
    syncWaterfallHook.tap("瀑布式同步钩子1", (name) => {  
      console.log("瀑布式同步钩子1", name);  
      
      return "古茗前端1";  
    });  
      
    syncWaterfallHook.tap("瀑布式同步钩子2", (name) => {  
      console.log("瀑布式同步钩子2", name);  
    });  
      
    syncWaterfallHook.tap("瀑布式同步钩子3", (name) => {  
      console.log("瀑布式同步钩子3", name);  
      
      return "古茗前端3";  
    });  
      
    asyncSeriesWaterfallHook.tapAsync("瀑布式异步串行钩子1", (name, callback) => {  
      setTimeout(() => {  
        console.log("瀑布式异步串行钩子1", name);  
      
        callback();  
      }, 1000);  
    });  
      
    asyncSeriesWaterfallHook.tapAsync("瀑布式异步串行钩子2", (name, callback) => {  
      console.log("瀑布式异步串行钩子2", name);  
      setTimeout(() => {  
        callback();  
      }, 2000);  
    });  
      
    asyncSeriesWaterfallHook.tapAsync("瀑布式异步串行钩子3", (name, callback) => {  
      console.log("瀑布式异步串行钩子3", name);  
      setTimeout(() => {  
        callback("古茗前端3");  
      }, 3000);  
    });  
      
    syncWaterfallHook.call("古茗前端");  
      
    asyncSeriesWaterfallHook.callAsync("古茗前端", (result) => {  
      console.log("result", result);  
    });  
      
      
    

控制台输出结果：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1CtS5qFOtnv9v59TJ8n0spTmAuZcqV7dIbP8npYONOvb6iaWFeM4gwepwA/640?wx_fmt=png)
image.png

###  Tapable 高级特性

####  Intercept

除了通常的 ` tap/call ` 之外，所有 ` hook ` 钩子都提供额外的拦截API。— ` intercept ` 接口。

` intercept ` 支持的中间件如下图所示:

intercept  |  类型  |  描述   
---|---|---  
**call** |  ` (...args) => void ` |  当钩子被触发时，向拦截器添加调用将被触发。您可以访问hooks参数   
**tap** |  ` (tap: Tap) ` |  将tap添加到拦截器将在插件点击钩子时触发。提供的是Tap对象。无法更改Tap对象   
**loop** |  ` (...args) => void ` |  向拦截器添加循环将触发循环钩子的每个循环。   
**register** |  ` (tap: Tap) => Tap 或 undefined ` |  将注册添加到拦截器将触发每个添加的Tap，并允许对其进行修改。   
  
**context**

插件和拦截器可以选择访问可选的上下文对象，该对象可用于向后续插件和拦截器传递任意值。

我们通过下面的小案例，来帮助我们理解。示例代码如下：

    
    
    const { SyncHook } = require("tapable");  
      
    // 实例化 钩子函数 定义形参  
    const syncHook = new SyncHook(["name"]);  
      
    syncHook.intercept({  
      context: true,  
      register(context, name) {  
        console.log("every time tap", context, name);  
      },  
      call(context, name) {  
        console.log("before call", context, name);  
      },  
      loop(context, name) {  
        console.log("before loop", context, name);  
      },  
      tap(context, name) {  
        console.log("before tap", context, name);  
      },  
    });  
      
    //通过tap函数注册事件  
    syncHook.tap("同步钩子1", (name) => {  
      console.log("同步钩子1", name);  
    });  
      
    //通过tap函数注册事件  
    syncHook.tap("同步钩子2", (name) => {  
      console.log("同步钩子2", name);  
    });  
      
    //同步钩子 通过call 发布事件  
    syncHook.call("古茗前端");  
    syncHook.call("古茗前端 call2");  
      
    

控制台输入结果如图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1CtKsxWFddvhA0IPYwyib74CxoWnagbxBgvw4lGFvZzia9mMFu9X3Uuc5qg/640?wx_fmt=png)
image.png

由上面的案例结果，我们可以知道。 ` intercept ` 中的 ` register ` 会在每一次的 ` tap ` 触发。有几个 ` tap `
就会触发几次 ` register ` 。然后依次执行钩子里面的 ` call ` 、 ` tap ` .

` intercept ` 特性在 webpack 内主要被用作进度提示，如 ` webpack/lib/ProgressPlugin ` 插件中，分别对
` compiler.hooks.emit ` 、 ` compiler.hooks.afterEmit ` 钩子应用了记录进度的中间件函数。

####  HookMap

` HookMap ` HookMap是具有Hooks的Map的辅助类.提供了一种集合操作能力，能够降低创建与使用的复杂度，用法比较简单：

    
    
    const { SyncHook, HookMap } = require("tapable");   
    const syncMap = new HookMap(() => new SyncHook());   
      
    // 通过 for 函数过滤集合中的特定钩子   
    syncMap.for("some-key").tap("MyPlugin", (arg) => { /* ... */ });  
    syncMap.for("some-key").tapAsync("MyPlugin", (arg, callback) => { /* ... */ });  
    syncMap.for("some-key").tapPromise("MyPlugin", (arg) => { /* ... */ });  
      
    // 触发 guming-test 类型的钩子   
    syncMap.get("guming-test").call();  
      
    

##  Tapable 底层原理

我们先将 ` Tapable ` 工程源码克隆到本地, 执行如下指令：

    
    
    $ git clone https://github.com/webpack/tapable.git  
    

` Tapable ` 源码的 ` lib ` 目录结构， 如下所示：

    
    
    lib  
    ├─ AsyncParallelBailHook.js  
    ├─ AsyncParallelHook.js  
    ├─ AsyncSeriesBailHook.js  
    ├─ AsyncSeriesHook.js  
    ├─ AsyncSeriesLoopHook.js  
    ├─ AsyncSeriesWaterfallHook.js  
    ├─ Hook.js  
    ├─ HookCodeFactory.js  
    ├─ HookMap.js  
    ├─ MultiHook.js  
    ├─ SyncBailHook.js  
    ├─ SyncHook.js  
    ├─ SyncLoopHook.js  
    ├─ SyncWaterfallHook.js  
    ├─ index.js  
    └─ util-browser.js  
    

除了上面我们所提及的基本 ` hooks ` 函数、 ` HookMap ` 高级特性，还会有一些 ` HookCodeFactory ` 、 ` Hook
` 这些文件。我们简单过一下 ` hooks ` 函数内的内容， 会发现所有的 ` hooks ` 函数都会引用 ` HookCodeFactory ` 和
` Hook ` 这两个文件所导出的对象实例。

我们以 ` syncHook ` 钩子函数为例， 如下图所示:

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1CtLLDnibkicCJyuWr1NVJ94fnwtWCoF9PgCE2UhzIdNjGPPNCycjGiaVicvw/640?wx_fmt=png)
image.png

我们大致能够知道 一个 ` hooks ` 函数 会由一个 ` CodeFactory ` 代码工厂 以及 ` Hook ` 实例组成。 ` Hook `
实例会针对不同场景的 ` hooks ` 函数， 更改其对应的 **注册钩子(` tapAsync ` , ` tap ` , ` tapPromise `
) **，** 事件触发钩子( ` call ` , ` callAsync ` ) ** , **编译函数(complier)**。 ` Complier
` 函数会由我们 ` HookCodeFactory ` 实现。

接下来，我们将通过分析 ` HookCodeFactory ` 及 ` Hook ` 的内部实现来了解 ` Tapable ` 的内部实现机制。

###  Hook 实例

` Hook ` 实例会生成我们 ` hooks ` 钩子函数通用的 ` 事件注册 ` ， ` 事件触发 ` 。核心逻辑，我们大致可以分为以下三个部分:

  1. 实例初始化构造函数 
  2. 事件注册 的实现 
  3. 事件触发的实现 

####  构造函数

构造函数会对实例属性初始化赋值。代码如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1Ctqu0m79OKl1F0FibQukuKJDqqnQEj0EHoIpN9WgLJEHBnkFYA7DmxzPQ/640?wx_fmt=png)
image.png

####  注册事件

注册事件主要分为两块，一块是 ` 适配器注册调用 ` ， 第二块是 ` 触发事件注册 ` 。核心逻辑在 ` _tap ` 函数内部实现，代码如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1CtKAuicZkmzYI0ngHpomJ7Inj8yQzaUoAibEDdZ5Bxot7iblT9QfXgk1rEA/640?wx_fmt=png)
image.png

**适配器调用**

在这里会对携带 ` register ` 函数的适配器进行调用，更改 ` options ` 配置，返回新的 ` options ` 配置。代码如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1CtdAzPcWfk3CYMzPfmhmvJWzee1kB1B4mkI4fJ1dpyWHibIWzBmUEDetg/640?wx_fmt=png)
image.png

**触发事件注册**

` Hook ` 实例的 ` taps ` 会存储我们的注册事件， 同时会根据，注册事件配置的执行顺序去存储对应的注册事件。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1CtdFPmtkCVUDF95jejgaDg289TGdlu2Pg7librRyWzNgvQwKl0lgM5usQ/640?wx_fmt=png)
image.png

####  触发事件

触发事件会通过调用内部 的 ` _createCall ` 函数，函数内部会调用实例的 ` compile ` 函数。我们会发现： ` Hook `
实例内部不会去实现 ` complier ` 的逻辑， 不同钩子的 ` complier ` 函数会通过通过对应的 继承 ` HookCodeFactory
` 的实例去实现。代码如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1CtRUQiasdqM2yfxU3c8GkdDbfvPj40whrtwcNdFCNJzEFW8aMpvFwbZJA/640?wx_fmt=png)
image.png
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1CtTfIo9H32eV1d2I68j9UlWFPSTYNBOzRxfz1cG8QgtPyQYd8iaKN5m4w/640?wx_fmt=png)
image.png

接下来，我们继续 探究 ` HookCodeFactory ` 实例，了解 ` Tapable ` 事件触发的逻辑。

###  HookCodeFactory 实例

` HookCodeFactory ` 实例会根据我们传入的事件触发类型 ( ` sync, async, promise ` )以及我们的触发机制类型 (
` 常规 ` ` 瀑布模式 ` ` 保险模式 ` ` 循环模式 ` ), 生成事件触发函数的函数头， 函数体。通过 ` new Function `
构造出事件触发函数。

> ` Tapable ` 事件触发的执行，是动态生成执行代码, 包含我们的参数，函数头，函数体，然后通过 ` new Function `
> 来执行。相较于我们通常的遍历/递归调用事件，这无疑让 ` webpack ` 的整个事件机制的执行有了一个更高的性能优势。

由上面我们可知， ` Hook ` 实例 的 ` complier ` 函数是 ` HookCodeFactory ` 实例 ` create ` 函数
的返回。

接下来，我们就从 ` create ` 函数 一步步揭秘 ` Tapable ` 的动态生成执行函数的核心实现。

####  create

` create ` 函数通过对应的 ` 函数参数 ` ， ` 函数 header ` , ` 函数 content `
方法构造出我们事件触发的函数的内容， 通过 ` new Function ` 创建出我们的触发函数。

create 函数会根据事件的触发类型 ( sync、async、promise)，进行不同的逻辑处理。代码如下图所示:
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1CtlMla9AKIEzDLUezB0nBGeraic0rAGd6B5FzSWYI0rGaI6oiaz4BpPk1Q/640?wx_fmt=png)

每一种触发机制，都会由 ` this.args ` , ` this.header ` , ` this.contentWithInterceptors `
三个函数去实现 动态函数的 ` code ` 。代码如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1CtRSwBxJTJbPWrX3fHgxibkeibiaYnvrSqQy1PDr56S6HYK3VKno63AZQ3A/640?wx_fmt=png)
image.png

接下来我们看一看 ` this.contentWithInterceptors ` 函数如何生成我们事件触发函数的函数体。

####  contentWithInterceptors

` contentWithInterceptors ` 函数里包含两个模块， 一个是适配器 ( ` interceptor ` ), 一个 `
content ` 生成函数。同时， ` HookCodeFactory ` 实例本身不会去实现 ` content `
函数的逻辑，会由继承的实例去实现。整体结构代码如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1CtPrZ3gJUpF4I9YxdcctvcwRBUElycODrVq47I1mDvTAHVa1UxKDM9VA/640?wx_fmt=png)
image.png

每个 ` hooks ` 钩子函数的 ` CodeFactory ` 实例会去实现 ` content ` 函数。 ` content ` 函数会调用 `
HookCodeFactory ` 实现的不同运行机制的方法( ` callTap ` 、 ` callTapsSeries ` 、 `
callTapsLooping ` 、 ` callTapsParallel ` )， 构造出最终的函数体。实现代码如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1CtGcc8MKibHEvfrb1cPKuLicWx9p65GN7lYiaoZINrYlDdKK7re1niaufgcg/640?wx_fmt=png)
image.png

接下来，就是不同运行机制，根据不同的调用方式 ( ` sync ` , ` async ` , ` promise ` ) 生成对应的执行代码。

####  动态函数

我们通过下面一个简单案例，看 ` New Function ` 输入的内容是什么？

实例代码如下:

    
    
      
    const { SyncHook, AsyncSeriesHook } = require("tapable");  
      
    // 实例化 钩子函数 定义形参  
    const syncHook = new SyncHook(["name"]);  
    const asyncSeriesHook = new AsyncSeriesHook(["name"])  
      
    //通过tap函数注册事件  
    syncHook.tap("同步钩子1", (name) => {  
      console.log("同步钩子1", name);  
    });  
      
    //通过tap函数注册事件  
    asyncSeriesHook.tapAsync("同步钩子1", (name) => {  
      console.log("同步钩子1", name);  
    });  
      
    //同步钩子 通过call 发布事件  
    syncHook.call("古茗前端sync");  
    asyncSeriesHook.callAsync("古茗前端async");  
    asyncSeriesHook.promise("古茗前端promise")  
      
    

在 ` HookCodeFactory ` 的 ` create ` 打印 ` fn ` , 实例代码如图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1Ct7CjwDQVtzfRKJ0DyYyYNzqfHG180QYPaia0CoibThuF7rS37lBSqrnWA/640?wx_fmt=png)
image.png

` sync ` 同步调用的输出结果如下：

    
    
     function anonymous(name  
    ) {  
    "use strict";  
    var _context;  
    var _x = this._x;  
    var _fn0 = _x[0];  
    _fn0(name);  
      
    }  
    

` async ` 异步调用的输出结果如下：

    
    
    function anonymous(name, _callback  
    ) {  
    "use strict";  
    var _context;  
    var _x = this._x;  
    var _fn0 = _x[0];  
    _fn0(name, (function(_err0) {  
    if(_err0) {  
    _callback(_err0);  
    } else {  
    _callback();  
    }  
    }));  
      
    }  
    

` promise ` 调用的输出结果如下

    
    
     function anonymous(name  
    ) {  
    "use strict";  
    var _context;  
    var _x = this._x;  
    return new Promise((function(_resolve, _reject) {  
    var _sync = true;  
    function _error(_err) {  
    if(_sync)  
    _resolve(Promise.resolve().then((function() { throw _err; })));  
    else  
    _reject(_err);  
    };  
    var _fn0 = _x[0];  
    _fn0(name, (function(_err0) {  
    if(_err0) {  
    _error(_err0);  
    } else {  
    _resolve();  
    }  
    }));  
    _sync = false;  
    }));  
      
    }  
    

三种不同方式调用，内部代码实现差异还是比较清晰的。 ` async ` 调用相较于 ` sync ` 多了回调函数。 ` async ` 和 `
promise ` 的区别再去返回 ` promise ` 还是回调函数。

最后，我们来用一些结构图来总结一下 ` Tapable ` 中事件触发的逻辑。

` HookCodeFactory ` 会根据我们触发的方式，生成我们对应 ` new Function ` 里面的 ` content ` , `
args ` , ` header ` 。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1CtctHM3mqRpKflmjibETfBF18ibtm5wEMCNkAlShUL8yyBPKfHTUorzic4Q/640?wx_fmt=png)
image.png

` content ` 最终会由 ` callTapsSeries ` 、 ` callTapsLooping ` 、 ` callTapsParallel
` 生成。每种生成方式都会包含 ` Done ` 处理、 ` Error ` 处理以及 ` Result ` 处理。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGufIJGlkfzxe97zRUaz1CteFJD1qeFKOeCNxGOTMzeJSwicMOSXmC0grYvBIO54Giank6nMgAmS8sw/640?wx_fmt=png)
image.png

##  总结

其他机制的 ` Hooks ` 钩子实现原理大致是相同的， 这里就不一一赘述了。 ` Tapable `
是一个非常优秀的库，灵活扩展性高，许多优秀的开源项目的插件化设计都借鉴或采纳了 ` Tapable ` 的设计思想，是一个非常值得推荐学习的一个开源工具库。

##  最后

📚 小茗文章推荐：

  * [ 古茗打印机技术的演进 ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247484498&idx=1&sn=79ba32d922199176369d4f4d05e2faea&chksm=cfe58355f8920a43a9b889d96ab2d582f1fbb9b7b2366d7f8431326b82cdaecc58164f2e25eb&scene=21#wechat_redirect)
  * [ 5分钟带你了解，古茗的代码发布与回滚 ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247484454&idx=1&sn=57684faf0e2988d922339f45d118efa3&chksm=cfe58321f8920a3718c7e6e1ee69a3380a1775f3c4bcffb75c26ad21fec5e12254b22954358b&scene=21#wechat_redirect)
  * [ 深入Git：4个关键步骤解锁版本控制机制 ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247484446&idx=1&sn=68853d15dfa21f2935e4bcd9358b11ba&chksm=cfe58319f8920a0fe760a76b50debc2fc97ea310d25bae5d4f02b6e50b523804ae26e1862f34&scene=21#wechat_redirect)   

关注公众号「Goodme前端团队」，获取更多干货实践，欢迎交流分享~

预览时标签不可点

微信扫一扫  
关注该公众号





****



****



×  分析

  收藏

