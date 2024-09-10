![cover_image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicFBGBt3jygNkZFmdo59EDtGJQ9tUXYG5l8icWEXxDKArFZCg0mqRrmwCXIl16ia7RAJA7PHatoNauDA/0?wx_fmt=jpeg)

#  如何优化 electron 应用在低配电脑秒启动

原创  周杰  [ Goodme前端团队 ](javascript:void\(0\);)

__ _ _ _ _

#  背景

古茗门店使用的收银机，有些会因为使用年限长、装了杀毒软件、配置低等原因性能较差，导致进钱宝启动响应较慢。然后店员在双击进钱宝图标后，发现没反应，就会重复点击

因此我们希望优化到即使在这些性能不太好的收银机上，也能让进钱宝有较快的启动体验
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/TpB2QHJbiaicFBGBt3jygNkZFmdo59EDtG9C4JL2n4lryPVCWbIBfUAUTKKRfuibAvaHfrzE5HwhFHhGZUuJ3bQGA/640?wx_fmt=gif)

#  优化思路

  * 测量，得到一个大概的优化目标，并发现可优化的阶段 
  * 主要方向是优化主进程创建出窗口的时间、让渲染进程页面尽快显示 
  * 性能优化好后，尽量让人感觉上更快点 
  * 上报各阶段耗时，建立监控机制，发现变慢了及时优化 

#  测量

##  测量主进程

编写一个 bat文件 放到应用根目录，通过bat启动程序并获取初始启动时间：

    
    
    @echo off  
      
    set "$=%temp%\Spring"  
    >%$% Echo WScript.Echo((new Date()).getTime())  
    for /f %%a in ('cscript -nologo -e:jscript %$%') do set timestamp=%%a  
    del /f /q %$%  
    echo %timestamp%  
    start yourAppName.exe  
      
    pause  
    

项目内可以使用如下api打印主进程各时间节点：

    
    
    this.window.webContents.executeJavaScript(  
      `console.log('start', ${start});console.log('onReady', ${onReady});console.log('inCreateWindow', ${inCreateWindow});console.log('afterCreateWindow', ${afterCreateWindow});console.log('beforeInitEvents', ${beforeInitEvents});console.log('afterInitEvents', ${afterInitEvents});console.log('startLoad', ${startLoad});`  
    );  
    

如果发现主进程有不正常的耗时，可以通过v8-inspect-profiler捕获主进程执行情况，最终生成的文件可以放到浏览器调试工具中生成火焰图

##  测量渲染进程

1、可以console打印时间点，可以借助preformance API获取一些时间节点

2、可以使用preformance工具测白屏时间等

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicFBGBt3jygNkZFmdo59EDtGeicX601HIpS8MJcT0S4HZ44OHe5ukLLpjIMUQbcqJjzLiaq10FMRncEg/640?wx_fmt=png)

##  进钱宝测量结果

以下测量结果中每一项都是时间戳，括号里是距离上一步的时间（ms）

最简单状态(主进程只保留唤起主渲染进程窗口的逻辑)：

执行exe（指双击应用图标）  |  开始执行主进程代码  |  主进程ready事件  |  开始初始化渲染进程窗口  |  开始加载渲染进程资源   
---|---|---|---|---  
1677666141619  |  1677666142152(+533)  |  1677666142224(+72)  |  1677666142364(+140)  |  1677666142375(+11)   
  
未优化状态：

执行exe  |  开始执行主进程代码  |  主进程ready事件  |  开始初始化渲染进程窗口  |  开始加载渲染进程资源   
---|---|---|---|---  
1677669414886  |  1677669417742(+2856)  |  1677669417856(+114)  |  1677669418043(+187)  |  1677669418061(+18)   
  
通过上述数据，能看出主进程最大的卡点是执行exe到开始执行代码之间

渲染进程的白屏时间，最初测试大概是1000ms

_**那么我们的优化目标，就是往最简单应用的时间靠齐，优化重点就是主进程开始执行代码时间，和渲染进程白屏时间** _

#  优化步骤

##  一、让主进程代码尽快执行

使用常见的方式，打包、压缩、支持tree-shaking，让代码体积尽可能的小；

可以把一些依赖按需加载，减少初始包体积

###  代码压缩

使用electron的一个好处是：chrome版本较高，不用pollyfill，可以直接使用很新的es特性

_**直接编译目标 ecma2020!!** _

###  优化tree-shaking

主进程存在global对象，但一些配置性的变量尽量不要挂载在global上，可以放到编译时配置里，以支持更好的tree-shaking

    
    
    const exendsGlobal = {  
      __DEV__,  
      __APP_DIR__,  
      __RELEASE__,  
      __TEST__,  
      __LOCAL__,  
      __CONFIG_FILE__,  
      __LOG_DRI__,  
      GM_BUILD_ENV: JSON.stringify(process.env.GM_BUILD_ENV),  
    };  
      
    // 这里把一些变量挂载在global上，这样不利于tree-shaking  
    Object.assign(global, exendsGlobal);  
    

###  慎用注册快捷方式API

实测这样的调用是存在性能损耗的

    
    
    globalShortcut.register('CommandOrControl+I', () => {  
      this.window.webContents.openDevTools();  
    });  
    // 这个触发方式，我们改为了在页面某个地方连点三下，因为事件监听基本没性能损耗  
    // 或者把快捷方式的注册在应用的生命周期中往后移，尽量不影响应用的启动  
    

###  优化require

因为require在node里是一个耗时操作，而主进程最终是打包成一个cjs格式，里面难免有require

可以使用 ` node --cpu-prof --heap-prof -e "require('request')" `
获取一个包的引用时长。如下是一些在我本机的测量结果：

包  |  时长(ms)   
---|---  
fs-extra  |  83   
event-kit  |  25   
electron-store  |  197   
electron-log  |  61   
v8-compile-cache  |  29   
  
具体理论分析可以看这里：如何加快 Node.js 应用的启动速度

因此我们可以通过一些方式优化require

  * 把require的包打进bundle 
    * bundle体积会增加，这样还是会影响代码编译和加载时间 
    * 有些库是必须require的，像node和electron的原生api；就进钱宝来说，我们可以通过其他方式优化掉require，因此没使用这种方式 
    * 有两个问题 
  * 按需require 
  * v8 code cache / v8 snapshot 
  * 对应用流程做优化，通过减少启动时的事务，来间接减少启动时的require量 

####  按需require

比如fx-extra模块的按需加载方式：

    
    
    const noop = () => {};  
      
    const proxyFsExtra = new Proxy(  
      {},  
      {  
        get(target, property) {  
          return new Proxy(noop, {  
            apply(target, ctx, args) {  
              const fsEx = require('fs-extra');  
              return fsEx[property](...args);  
            },  
          });  
        },  
      }  
    );  
      
    export default proxyFsExtra;  
    

前面的步骤总是做了没坏处，但这个步骤因为要重构代码，因此要经过验证

因此我们测量一下：

执行exe  |  开始执行主进程代码  |  主进程ready事件  |  开始初始化渲染进程窗口  |  开始加载渲染进程资源   
---|---|---|---|---  
1677674087344  |  1677674089485(+2141)  |  1677674089606  |  1677674089864  |  1677674089934   
  
可以看出，主进程开始执行时间已经有了较大优化（大概700ms）

####  v8-compile-cache

可以直接用 v8-compile-cache 这个包做require缓存

简单测试如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicFBGBt3jygNkZFmdo59EDtGO7Gdd4JPCdBewhnibtkicx8P0ibKg2ibNib7MSHiavqMTQrUtaIsEHNFYoaQ/640?wx_fmt=png)

脚本执行时间从388到244，因此这个技术确实是能优化执行时间的

但也有可能没有优化效果：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicFBGBt3jygNkZFmdo59EDtGlbzEPRaP616icOv3FhPClK3Lz4dykg072bM8MAyEHEvOHRibGq605PLQ/640?wx_fmt=png)

在总require较少，且包总量不大的情况下，做cache是没有用的。实测对进钱宝也是没用的，因为经过后面的流程优化步骤，进钱宝代码的初始require会很少。因此我们没有使用这项技术

但我们还是可以看下这个包的优化机制，这个包核心代码如下，其实是重写了node的Module模块的_compile函数，编译后把V8字节码缓存，以后要执行时直接使用缓存的字节码省去编译步骤

    
    
    Module.prototype._compile = function(content, filename) {  
     ...  
      
      // 读取编译缓存  
      var buffer = this._cacheStore.get(filename, invalidationKey);  
      
      // 这一步是去编译代码，但如果传入的cachedData有值，就会直接使用，从而跳过编译  
      // 如果没传入cachedData，这段代码就会产生一份script.cachedData  
      var script = new vm.Script(wrapper, {  
        filename: filename,  
        lineOffset: 0,  
        displayErrors: true,  
        cachedData: buffer,  
        produceCachedData: true,  
      });  
      
      // 上面的代码会产生一份编译结果，把编译结果写入本地文件  
      if (script.cachedDataProduced) {  
        this._cacheStore.set(filename, invalidationKey, script.cachedData);  
      }  
      
      // 运行代码  
      var compiledWrapper = script.runInThisContext({  
        filename: filename,  
        lineOffset: 0,  
        columnOffset: 0,  
        displayErrors: true,  
      });  
      
      ...  
    };  
    

这里有个可能的优化点：v8-compile-cache
只是缓存编译结果，但require一个模块除了编译，还有加载这个io操作，因此是否可以考虑连io一起缓存

####  v8-snapshot

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicFBGBt3jygNkZFmdo59EDtG4ABSQtsGjZibQbibjCcnJwuQCxQ7OUiaXMe8XwlGTPYiaZ54cj8KTibjHDg/640?wx_fmt=png)

原理是：把代码执行结果的内存，做一个序列化，存到本地，真正执行时，直接加载然后反序列化到内存中

这样跳过了代码编译和执行两个阶段，因此可以提升应用的初始化速度。

优化效果：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicFBGBt3jygNkZFmdo59EDtGibicHd9icxVmKEM48TdpW0dqdSJnn42N0o5XIJQiceO0ydVYbytrDRTvpA/640?wx_fmt=png)

对react做快照后，代码中获取的react对象如下图，实际上获得的是一份react库代码执行后的内存快照，跟正常引入react库没什么区别：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicFBGBt3jygNkZFmdo59EDtGofRjFAmxuHmwiawh69NuYpwZDdvQA4yIhAD1MvBMSHbYLmDhS4Q8uvQ/640?wx_fmt=png)

_**这个方案看起来很香，但也存在两个小问题：** _

_**1、不能对有副作用的代码做snapshot** _

因为只是覆写内存，而没有实际代码执行，因此如果有 读写文件、操作dom、console 等副作用，是不会生效的

因此这个步骤更多是针对第三方库，而不是业务代码

_**2、需要修改打包配置** _

目前项目一般通过import引用各种包，最终把这些包打包到bundle中；但该方案会在内存直接生成对象，并挂载在全局变量上，因此要使用snapshot，代码中包引用方式需要修改，这个可以通过对编译过程的配置实现

这个技术看起来确实能有优化效果，但考虑如下几点，最后我们没有去使用这项技术：

  * 对主进程没用，因为主进程刚进来就是要做打开窗口这个副作用； 
  * 对渲染进程性价比不高，因为 
    * 我们的页面渲染已经够快(0.2s) 
    * 启动时，最大的瓶颈不在前端，而在服务端初始化，前端会长时间停留在launch页面等待服务端初始化，基于这一点，对渲染进程js初始化速度做极限优化带来的收益基本没有，我们真实需要的是让渲染进程能尽快渲染出来一些可见的东西让用户感知 
    * 维护一个新模块、修改编译步骤、引入新模块带来的潜在风险 

snapshot具体应用方式可看文尾参考文章

##  二、优化主进程流程，让应该先做的事先做，可以后做的往后放

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicFBGBt3jygNkZFmdo59EDtGrsgLzOA3t6LZB7WMNNicmkPuPMOlLC0tb8x4ibfEWUYLY8cUjYeEmYrA/640?wx_fmt=png)
基于上图的思想，我们对bundle包做了拆分：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicFBGBt3jygNkZFmdo59EDtGgribHmYnhcP6hJbBIulB1M2JrnD6Jnibo2ib7PCpHuHlAEDqle2YeOnow/640?wx_fmt=png)

新的测量数据：

执行exe  |  开始执行主进程代码  |  主进程ready事件  |  开始初始化渲染进程窗口  |  开始加载渲染进程资源   
---|---|---|---|---  
1677911394516  |  1677911395044(+528)  |  1677911395133(+89)  |  \-  |  \-   
  
可以看出，到这里主进程已经跟最简单状态差不多了。而且这一步明显优化非常明显。而这一步做的事情核心就是减少初始事务，从而减少了初始代码量以减少编译和加载负担，也避免了初始时过多比较耗性能的API的执行（比如require，比如new
BrowserWindow()）。

那么我们主进程优化基本已经达到目标

##  三、让渲染进程尽快渲染

####  requestIdleCallback

程序刚启动的时候，CPU占用会很高（100%），因此有些启动任务可以通过requestIdleCallback，在浏览器空闲时间执行，让浏览器优先去渲染

####  去掉或改造起始时调用sendSync以及使用electron-store的代码

原因是sendSync是同步执行，会阻塞渲染进程

而electron-store里面初始时会调用sendSync

####  只加载首屏需要的css

对首屏不需要的ui库、components做按需加载，以减少初始css量，首屏尽量只加载首屏渲染所需的css

> **因为css量会影响页面的渲染性能**
>
> 使用 tailwind 的同学可能会发现一个现象：如果直接加载所有预置css，页面动画会非常卡，因此 tailwind 会提供 Purge
> 功能自动移除未使用的css

####  少用或去掉modulepreload

我们使用的是vite，他会自动给一些js做modulepreload。但实测modulepreload（不是preload）是会拖慢首屏渲染的，用到的同学可以测测看

##  四、想办法让应用在体验上更快

####  使用骨架屏提升用户体感

程序开始执行 -> 页面开始渲染， 这段时间内可以使用骨架屏让用户感知到应用在启动，而不是啥都没有

我们这边用c++写了个只有loading界面的exe，在进钱宝启动时首先去唤起这个exe，等渲染进程渲染了，再关掉他（我们首屏就是一个很简单的页面，背景接近下图的纯色，因此loading界面也做的比较简单）

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/TpB2QHJbiaicFBGBt3jygNkZFmdo59EDtGQ7Yd3FfUxibibXfqoPTGXYNZFPtpdaQ7uEKIILic2pFbTibjFuHqaQrjrQ/640?wx_fmt=gif)
动画.gif

####  渲染进程骨架屏

渲染进程渲染过程：加载解析html -> 加载并执行js渲染

在js最终执行渲染前，就是白屏时间，可以在html中预先写一点简单的dom来减少白屏时间

####  一个白屏优化黑科技

我们先看两种渲染效果：

#####  渲染较快的

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicFBGBt3jygNkZFmdo59EDtGicrBLQIbbCAiaYsTg36homM748RA9nxGxcZlzvNHdKlT6GESwMRO8g1g/640?wx_fmt=png)
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicFBGBt3jygNkZFmdo59EDtGiaIXfRb4bcUbs8ZKXOzcTmklFAibLQUYmPIzp40ich8ZWIVLVGPMEY5LQ/640?wx_fmt=png)

#####  渲染较慢的

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicFBGBt3jygNkZFmdo59EDtGcOBW6LrqSwM4Nz5FyujDshAkOSDjAicgicvNupGk4ianlowmwvYwqOTTA/640?wx_fmt=png)
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicFBGBt3jygNkZFmdo59EDtGibKmLWX35WiaUcFc627IAXJvmnVWdFa7OPVU6UsuUSOZAia71xwX6ITrg/640?wx_fmt=png)

接下来看下代码区别：

    
    
    快的代码：  
    <div id="root">  
        <span style="color: #000;">哈哈</span> <!-- 就比下面那个多了这行代码 -->  
        <div class="container">  
          <div class="loading">  
            <span></span>  
          </div>  
        </div>  
      </div>  
      
    慢的代码：  
    <div id="root">  
      
        <div class="container">  
          <div class="loading">  
            <span></span>  
          </div>  
        </div>  
      </div>  
    

**就是多了一行文字，就会更快地渲染出来**

从下图可以看到，文字渲染出来的同时，背景色和loading动画（就中间那几个白点）也渲染出来了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicFBGBt3jygNkZFmdo59EDtGy5saoaL23Yklb6khicbkoBOmfwGNfZxibCmicbdWqbtHicEJz2K7Qianzww/640?wx_fmt=png)

有兴趣的可以测一下淘宝首页，如果去掉所有文字，还是会较快渲染，但如果再去掉加载的css中的一个background:
url(.....jpg)，首次渲染就会变慢了

我猜啊。。。这个叫信息优先渲染原则。。。🐶就是文字图片可以明确传递信息，纯dom不知道是否传递信息，而如果页面里有明确能传递信息的东西，就尽快渲染出来，否则，渲染任务就可能排到其他初始化任务后面了。

当然了，这只是我根据测试结果反推出来的猜测🐶

好了，现在我们也可以让渲染进程较快的渲染了（至少能先渲染出来一个骨架屏🤣）

##  五、其他

####  升级electron版本

electron 官方也是在不断优化bug和性能的

####  保证后续的持续优化

因为经过后续的维护，比如有人给初始代码加了些不该加的重量，是有可能导致性能下降的

因此我们可以对各节点的数据做上报，数据大盘，异常告警，并及时做优化，从而能持续保证性能

#  总结

本文介绍了electron应用的优化思路和常见的优化方案。并在进钱宝上取得了实际效果，我们在一台性能不太好的机器上，把感官上的启动时间从10s优化到了1s（可能有人会提个问题，上面列的时间加起来没有10s，为啥说是10s。原因是我们最初是在渲染进程的did-
finish-load事件后才显示窗口的，这个时间点是比较晚的）

这其中最有效的步骤是优化流程，让应该先做的事先做，可以往后的就往后排，根据这个原则进行拆包，可以使得初始代码尽可能的简单（体积小，require少，也能减少一些耗性能的动作）。

另外有些网上看起来很秀的东西，不一定对我们的应用有用，是要经过实际测量和分析的，比如code-cache 和 snapshot

还有个点是，如果想进一步提升体验，可以先启动骨架屏应用，再通过骨架屏应用启动进钱宝本身，这样可以做到ms级启动体验，但这样会使骨架屏显示时间更长点（这种体验也不好），也需要考虑win7系统会不会有dll缺失等兼容问题

##  最后

关注公众号「Goodme前端团队」，获取更多干货实践，欢迎交流分享~

#  参考文档

##  v8 code cache

https://v8.dev/blog/improved-code-caching  
https://v8.dev/blog/code-caching-for-devs  
https://fed.taobao.org/blog/taofed/do71ct/wpsf10/  
https://blog.csdn.net/szengtal/article/details/89636938

##  v8 snapshot

https://www.javascriptcn.com/post/5eedbcf2b5cbfe1ea0611a5d  
https://blog.inkdrop.app/how-to-make-your-electron-app-launch-1000ms-
faster-32ce1e0bb52c  
https://github.com/inkdropapp/electron-v8snapshots-example

##  其他

https://zhuanlan.zhihu.com/p/420238372

https://blog.csdn.net/qq_37939251/article/details/108122157

https://medium.com/@felixrieseberg/javascript-on-the-desktop-fast-and-
slow-2b744dfb8b55

https://zhuanlan.zhihu.com/p/376638202

  

预览时标签不可点

微信扫一扫  
关注该公众号





****



****



×  分析

  收藏

