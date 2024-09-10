![cover_image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicEBmtHQ5fibGzX3vrG51AXDpV7kr7nwZOfjI1SEvPiaL9DbJqKE9hNyoVZlMFE8A7kdnRFydz3ZDxqQ/0?wx_fmt=jpeg)

#  5分钟回顾webpack的前世今生

原创  章文亮  [ Goodme前端团队 ](javascript:void\(0\);)

__ _ _ _ _

##  引言

模块化编程是软件设计的一个重要思想。在JavaScript中，处理模块一直是个问题，由于浏览器只能执行JavaScrip、CSS、HTML
代码，所以模块化的前端代码必须进行转换后才能运行。例如 CommonJS 或 AMD，甚至是ECMA 提出的 JavaScript 模块化规范——ES6
模块，这些模块系统要么是在浏览器无法运行，要么是无法被浏览器识别和加载，所以针对不同的模块系统，就需要使用专门的工具将源代码转换成浏览器能执行的代码。

整个转化过程被称为构建，构建过程就是“模块捆绑器”或“模块加载器”发挥作用的地方。

Webpack是JavaScript模块捆绑器。在Webpack之前，已经有针对各类型的代码进行编译和构建的流程，例如使用Browserify对CommonJS模块进行编译和打包，然后将打包的资源通过HTML去加载；或者通过gulp进行任务组排来完成整个前端自动化构建。

但是这些方式的缺点是构建环节脱离，编译、打包以及各类资源的任务都分离开。

Webpack模块系统的出现，能将应用程序的所有资源（例如JavaScript、CSS、HTML、图像等）作为模块进行管理，并将它们打包成一个或多个文件并进行优化。Webpack的强大和灵活性使得其能够处理复杂的依赖关系和资源管理，已经成为了构建工具中的首选。

本文主要来扒一扒Webpack的发展进阶史，一起来看看Webpack是如何逐渐从一个简单的模块打包工具，发展成一个全面的前端构建工具和生态系统。

##  webpack发展历程

webpack从2012年9月发布第一个大版本至2020年10月一共诞生了5个大的版本，我们从下面一张图可以清晰具体地看到每一个版本的主要变化

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicEBmtHQ5fibGzX3vrG51AXDpOVD4ia6enKLCiciaK8ciaTOPYwndcicJTnQonicQMoiajkdhNC89iaZmhlKJVQ/640?wx_fmt=png)
Webpack发展史.png

##  Webpack 版本变化方向

  1. **Webpack 1** ：在此之前多是用 ` gulp ` 对各个类型的编译任务进行编排，最后在Html文件中将各种资源引用进来，而 ` Webpack ` 的初始版本横空出世，凭借如下其功能、理念、内核等优点成为众多前端构建工具的最新选择。 

  * **理念** ：一切皆资源，在代码中就能能对 ` Html、Js、Css、图片、文本、JSON ` 等各类资源进行模块化处理。 
  * **内核** ：实现了独有的模块加载机制，引入了模块化打包和代码分割的概念。 
  * **功能** ：集合了编译、打包、代码优化、性能改进等以前各类单一工具的功能，成为前端构建工具标准选择。 
  * **特点** ：通过配置即可完成前端构建任务，同时支持开发者自定义 ` Loader ` 和 ` Plugin ` 对Webpack的生态进行更多的扩展。 

  2. **Webpack 2：** Webpack 2的在第一个版本后足足过了4年，其重点在于满足更多的打包需求以及少量对打包产物的优化 

  * 引入对 ` ES6 ` 模块的本地支持。 
  * 引入 ` import ` 语法，支持按需加载模块。 
  * 支持 ` Tree Shaking ` （无用代码消除）。 

  3. **Webpack 3：** ` Webpack 3 ` 提供了一些优化打包速度的配置，同时对打包体积的优化再次精益求精 

  * 引入 ` Scope Hoisting ` （作用域提升），用于减小打包文件体积。 
  * 引入 ` module.noParse ` 选项，用于跳过不需要解析的模块。 

  4. **Webpack 4：** ` Webpack 4 ` 带来了显著的性能提升，同时侧重于用户体验，倡导开箱即用 

  * 引入了 ` mode ` 选项，用于配置开发模式或生成模式，减少用户的配置成本，开箱即用 
  * 内置 ` Web Workers ` 支持，以提高性能 

  5. **Webpack 5：** ` Webpack 5 ` 继续在构建性能和构建输出上进行了改进，且带来跨应用运行时模块共享的方案 

  * 支持 ` WebAssembly ` 模块，使前端能够更高效地执行计算密集型任务。 
  * 引入了文件系统持久缓存，提高构建速度 
  * 引入 ` Module Federation ` （模块联邦），允许多个Webpack应用共享模块 

##  webpack打包后的代码分析

为了更方便理解后续章节，我们先看一下Webpack打包后的代码长什么样（为了方便理解，这里以低版本Webpack为例，且不做过多描述）

    
    
    /******/ (function(modules) { // webpackBootstrap  
    /******/  // The module cache  
    /******/  var installedModules = {};  
      
    /******/  // The require function  
    /******/  function __webpack_require__(moduleId) {  
    /******/  /* 省略 */  
    /******/  }  
      
    /******/  // expose the modules object (__webpack_modules__)  
    /******/  __webpack_require__.m = modules;  
      
    /******/  // expose the module cache  
    /******/  __webpack_require__.c = installedModules;  
      
    /******/  // __webpack_public_path__  
    /******/  __webpack_require__.p = "";  
      
    /******/  // Load entry module and return exports  
    /******/  return __webpack_require__(0);  
    /******/ })  
    /************************************************************************/  
    /******/ ([  
    /* 0 */  
    /***/ (function(module, __webpack_exports__, __webpack_require__) {/*省略*/})  
    /* 1 */  
    /***/ (function(module, __webpack_exports__, __webpack_require__) {/*省略*/})  
    /* 2 */  
    /***/ (function(module, __webpack_exports__, __webpack_require__) {/*省略*/})  
    /******/ ]);  
    

可以看到其实入口文件就是一个 ` IIFE ` （立即执行函数），在这个IIFE里核心包括两块：

  1. **模块系统** ：Webpack 在IIFE里实现了模块系统所需要的 ` Module、Require、export ` 等方法组织代码。每个模块都被包装在一个函数内，这个函数形成了一个闭包，模块的作用域在这个闭包内。 
  2. **模块闭包** ： ` IIFE ` 的入参即是 ` Modules ` ，它是一个数组，数组的每一项则是一个模块，每个模块都有自己的作用域。模块和模块之间通过 ` Webpack ` 的模块系统可以进行引用。 

##  webpack的发展长河中，笑到最后和沦为历史

###  笑到最后：OccurrenceOrderPlugin

有趣的是该插件在 ` Webpack 1 ` 叫做 ` OccurenceOrderPlugin ` ， ` Webpack 2 ` 才更名为 `
OccurrenceOrderPlugin ` ， ` Webpack 3 ` 则不需要手动配置该插件了。

**插件作用** ：用于优化模块的顺序，以减小输出文件的体积。其原理基于模块的使用频率，将最常用的模块排在前面，以便更好地利用浏览器的缓存机制。

有了前面对于Webpack打包后的代码分析， ` OcurrenceOrderPlugin ` 的优化效果也就很好理解了。它的原理主要基于两个概念：
**模块的使用频率** 和 **模块的ID**

  1. **模块的使用频率** ：OccurrenceOrderPlugin 插件会分析在编译过程中每个模块的出现次数。这个出现次数是指模块在其他模块中被引用的次数。插件会统计模块的出现次数，通常情况下，被引用次数更多的模块将被认为更重要，因此会更早地被加载和执行。 
  2. **模块的 ID** ：Webpack 使用数字作为模块的 ID，OccurrenceOrderPlugin 插件会根据模块的出现次数，为每个模块分配一个优化的 ID。这些 ID 的分配是按照出现次数从高到低的顺序进行的，以便出现次数较多的模块获得较短的 ID，这可以减小生成的 JavaScript 文件的大小。假设一共有100个模块，最高的频率为被引用100次，则减小文件体积200B。（确实好像作用很小，但是作为最贴近用户体验的前端er，不应该是追求精益求精嘛） 

这个插件的主要目标是减小 JavaScript 文件的体积，并提高加载性能，因为浏览器通常更倾向于缓存较小的文件。通过将频繁使用的模块分配到较短的
ID，可以减小输出文件的体积，并提高缓存的效率。

###  笑到最后：Scope Hoisting

过去 Webpack 打包时的一个取舍是将 bundle 中各个模块单独打包成闭包。这些打包函数使你的 JavaScript
在浏览器中处理的更慢。相比之下，一些工具像 Closure Compiler 和 RollupJS
可以提升(hoist)或者预编译所有模块到一个闭包中，提升你的代码在浏览器中的执行速度。

而Scope Hoisting
就是实现以上的预编译功能，通过静态分析代码，确定哪些模块之间的依赖关系，然后将这些模块合并到一个函数作用域中。这样，多个模块之间的函数调用关系被转化为更紧凑的代码，减少了函数调用的开销。这样不仅减小了代码体积，同时也提升了运行时性能。

Scope Hoisting 的原理是在 Webpack 的编译过程中自动进行的，开发人员无需手动干预。要启用 Scope Hoisting，你可以使用
Webpack 4 版本中引入的 **` moduleConcatenation ` ** 插件。在 Webpack 5 及更高版本中，Scope
Hoisting 是默认启用的，不需要额外的配置。

###  CommonsChunkPlugin的作用和不足，为何会被optimization.splitChunks所取代

CommonsChunkPlugin 插件，是一个可选的用于建立一个独立chunk的功能，这个文件包括多个入口 chunk 的公共模块。主要配置项包含

    
    
    {  
      name: string, // or  
      names: string[],  
      filename: string,  
      minChunks: number|Infinity|function(module, count) => boolean,  
      chunks: string[],  
      // 通过 chunk name 去选择 chunks 的来源。chunk 必须是  公共chunk 的子模块。  
      // 如果被忽略，所有的，所有的 入口chunk (entry chunk) 都会被选择。  
      
      children: boolean,  
      deepChildren: boolean,  
    }  
    

通过上面的配置项可以看到虽然CommonsChunkPlugin将一些重复的模块传入到一个公共的chunk，以减少重复加载的情况，尤其是将第三方库提取到一个单独的文件中，但是其首要依赖是通过Entry
Chunk进行的。在Webpack4以及更高的版本当中被 ` optimization.splitChunks `
所替代，其提供了配置让webpack根据策略来自动进行拆分，被替代的原因主要有以下几点：

  1. **灵活度不足** ：在配置上相对固定，只能将指定 Entry Chunk的共享模块提取到一个单独的chunk中,可能无法满足复杂的代码拆分需求。 
  2. **配置复杂** ：需要手动指定要提取的模块和插件的顺序，配置起来相对复杂，开发者需要约定好哪些chunk可以被传入，有较高的心智负担。而 ` optimization.splitChunks ` 只需要配置好策略就能够帮你自动拆分。 

因此在Webpack 4这个配置和开箱即用的版本里，它自然也就“香消玉损”。只能遗憾地看到一句：

_the CommonsChunkPlugin 已经从 Webpack v4 legato 中移除。想要了解在最新版本中如何处理 chunk，请查看_
SplitChunksPlugin _。_

###  被移除的DedupePlugin

这是 Webpack 1.x
版本中的插件，用于在打包过程中去除重复的模块（deduplication），其原理不知道是通过内容hash，还是依赖调用关系图。但是在Webpack
2中引入了Tree Shaking功能，则不再需要了。原因有以下几点：

  * **Tree Shaking控制更精确** ：能通过静态分析来判断哪些代码是不需要的，实现了更细力度的优化。 
  * **Scope Hositing减少了重复模块** ：Webpack 3引入了 ` Scope Hositing ` ，将模块包裹在函数闭包中，进一步减少了重复模块的依赖 

因此我们在 ` Webpack ` 的文档中看到：DedupePlugin has been removed

不再需要 Webpack.optimize.DedupePlugin。请从配置中移除。

##  总结

或许有些插件你已经看不到它的身影，有些特性早已被webpack内置其中。webpack从第一个版本诞生后一直致力于以下几个方面的提升：

  1. **性能优化** ：通过去除重复代码、作用域提升、压缩等方式减少代码体积和提高运行时性能。 
  2. **构建提效** ：通过增量编译、缓存机制、并行处理等提升打包速度。 
  3. **配置简化** ：通过内置必要的特性和插件以及简化配置提升易用性。 

##  最后

关注公众号「Goodme前端团队」，获取更多干货实践，欢迎交流分享~

  

预览时标签不可点

微信扫一扫  
关注该公众号





****



****



×  分析

  收藏

