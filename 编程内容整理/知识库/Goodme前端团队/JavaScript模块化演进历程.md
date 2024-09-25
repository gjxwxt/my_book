![cover_image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicFT4NZHp7L1Pmk4kTwDEzzDKMrgw5ibOZcaVxJ60wDAx5raVsuSkY0r07PXMCKc0UWQtTPGEBYNSmQ/0?wx_fmt=jpeg)

#  JavaScript 模块化演进历程

刘锦泉  [ Goodme前端团队 ](javascript:void\(0\);)

__ _ _ _ _

##  引言

回顾 ` JavaScript `
的发展历程，从最初的简单浏览器脚本语言，到如今构建互联网应用程序的现代编程语言，模块化技术在这一演变中发挥了关键作用。本篇文章将从 ` JavaScript
` 的模块化探索及模块化规范的持续演进两部分，回顾 ` JavaScript ` 模块化的演进历程。

##  模块化理念

在回顾 ` JavaScript ` 模块化技术的演进历程之前， 我们先了解一些基本的概念。何谓模块？且看维基百科给出来的定义：

> **模块** 是指由数个基础功能组件组成的特定功能组件，可用来组成具完整功能之系统、设备或程序。模块通常都会具有相同的  制程  [1]  或  逻辑
> [2]  ，更改其组成组件可调适其功能或用途。

>
> 我们可以简单地将模块理解为一个独立的文件或脚本，每个文件/脚本都被视为一个模块，包含特定的功能或逻辑。模块将代码分离到独立的单元中，方便管理、复用和维护。

那么，什么又是模块化呢？

>
> 模块化是现代软件工程的核心原则之一，它通过将大型的复杂系统拆解为更小、更容易管理和理解的部分——功能模块，来提高系统的可维护性和可拓展性。每个功能模块都是一个独立的单元，具有清晰定义的接口和职责，能够与其他模块交互以完成复杂的任务。

模块化是将复杂系统拆分为独立模块的设计方法，这些模块能够独立开发、测试和维护，并可以在不同项目或环境中复用。模块化在现代软件开发中起到了至关重要的作用，特别是在大型应用程序的开发中，模块化能够显著提高开发效率和代码质量。

##  JavaScript 模块化探索

我们知道，由于一些 历史的原因，很长一段时间 ` JavaScript ` 语言是没有模块化的概念的。

最早，我们就是直接在 ` script ` 标签里书写 ` JS ` 代码的：

    
    
      
    <script>  
      
     function add (a, b) {  
      
       return a + b;  
      
     };  
      
     add(1,2);  
      
    </script>  
      
    

随着代码量的增加，我们将业务逻辑拆分成多个 ` js ` 文件：

    
    
      
    /* a.js */  
      
      
      
    function add(a, b) {  
      
      return a + b;  
      
    }  
      
      
      
    /* b.js */  
      
      
      
    function average(a, b) {  
      
      return add(a, b) / 2;  
      
    }  
      
      
      
    /* main.js */  
      
      
      
    var result = average(5, 10);  
      
    console.log(result);  
      
      
      
    

然后在 HTML 中的引入如下：

    
    
      
    <script src="./a.js"></script>  
      
    <script src="./b.js"></script>  
      
    <script src="./main.js"></script>  
      
    

我们可以明显发现，由于这些 ` js ` 文件中的函数和变量直接暴露在全局作用域中，不仅容易导致命名冲突，还使得文件之间的依赖关系难以管理。

###  命名空间

随着代码量的增加，这些问题愈加严重。为了解决命名冲突并促进团队协作，命名空间的概念应运而生。

    
    
      
    var moduleA = {  
      
      name = 'moduleA'  
      
    }  
      
    moduleA.add = function(a, b) {  
      
        console.log(a + b)  
      
    }  
      
    moduleA.add(1, 2)  
      
    

引入命名空间后，命名冲突问题得到了部分缓解，模块间的依赖关系也更为清晰。然而，这种封装方式本质上是对象，对外仍然暴露了不应被访问的内容，安全性不足。

例如： ` b.js ` 模块的开发者，可以很方便的通过 ` moduleA.name ` 来取到 ` 模块A ` 中的名字，但是也可以通过 `
moduleA.name = 'rename' ` 来任意改掉模块A中的名字，而这件事情， ` 模块A ` 却毫不知情！这显然是不被允许的。

显然，这种方式并未解决根本问题，但模块化的概念已经开始萌芽，只是暂时通过命名空间来实现。

###  立即调用函数表达式（IIFE）

> 立即调用函数的匿名闭包是模块化实现的基石

为了更好地解决全局作用域污染和命名冲突问题，开发者引入了自执行函数和闭包的概念，这就是 ` IIFE ` （ ` Immediately Invoked
Function Expression ` ） 。

> **立即调用函数表达式** （IIFE）是一个在定义时就会立即执行的 JavaScript
> 函数。它是一种设计模式，也被称为自执行匿名函数，主要包含两部分：

> >   1. 第一部分是一个具有词法作用域的匿名函数，并且用圆括号运算符 ` () `
> 运算符闭合起来。这样不但阻止了外界访问自执行匿名函数中的变量，而且不会污染全局作用域。
>

>   1. 第二部分创建了一个立即执行函数表达式 ` () ` ，通过它，JavaScript 引擎将立即执行该函数。
>

` IIFE `
提供了一种简单而有效的方式来创建独立的作用域。通过将函数包装在括号内并立即调用，它创建了一个新的执行上下文，这样函数内部的变量和函数不会污染全局作用域。

` IIFE ` 示例代码如下:

    
    
      
    var utils = (function() {  
      
        var moduleA = {}  
      
        moduleA.add = function(a, b) {  
      
            console.log(a * b)  
      
        }  
      
        return moduleA  
      
    }())  
      
      
      
    utils.add(1,2);  
      
      
      
    

然而，这种实现并不完美，仍然需要手动维护依赖顺序。例如， ` jQuery ` 将所有函数都集中在全局对象 ` $ ` 中，项目必须确保 ` jQuery
` 加载完成后才能使用 ` $ ` 。

此时，我们认识到，模块化不仅需要解决全局变量污染和数据保护的问题，还必须有效地管理模块之间的依赖关系。

##  JavaScript 模块化规范的持续演进

随着 ` JavaScript ` 应用程序复杂性的提升，对模块化的需求也日益增长，推动了社区不断提出和发展新的模块化规范。

###  CommonJs

2009年1月， ` Mozilla ` 工程师 ` Kevin Dangoor ` 创建了名为 ` ServerJS `
的项目。到了2009年8月，这个项目更名为 ` CommonJS ` ，以彰显其 ` API ` 的广泛适用性。通过 ` CommonJS ` ， `
Node.js ` 实现了高效的模块加载、管理和组织，使服务器端 ` JavaScript ` 编程变得更加模块化和高效。

` CommonJS ` 约定：

  * 每个文件就是一个模块， 有自己的作用域 

  * 每个文件中定义的变量、函数、类都是私有的，对其它文件不可见 

  * 每个模块内部可以通过 ` exports ` 或者 ` module.exports ` 对外暴露接口 

  * 每个模块通过 ` require ` 加载另外的模块 

` CommonJs ` 使用代码示例如下：

    
    
      
    /* a.js */  
      
    function add(a, b) {  
      
      return a + b;  
      
    }  
      
      
      
    module.exports = {  
      
      add,  
      
    };  
      
      
      
    /* b.js */  
      
    const moduleA = require('./a.js');  
      
    const result = moduleA.add(5, 10);  
      
    console.log(result);  
      
    

` CommonJS ` 提供了统一的模块定义和管理方式，使模块间的依赖关系更加明确和易于管理。它标志着 ` JavaScript `
在服务器端模块化的初步发展，并为现代 ` JavaScript ` 生态系统奠定了基础。

` CommonJS `
规范在服务端实现了模块化，提供了同步加载模块的方式，这在本地磁盘读取模块时非常高效。然而，在浏览器中，同步加载可能导致浏览器假死，因为从服务器加载模块需要时间。因此，在客户端，异步加载模块成为更合理的选择，以避免阻塞和性能问题。

###  AMD

> AMD和CMD只是一种设计规范，而不是一种实现。

` AMD ` 即 ` Asynchronous Module Definition ` ，它采用异步的方式加载 ` JavaScript `
模块，模块的加载并不会影响它后面语句的运行。

` AMD ` 规范规定用 ` define ` 定义模块用 ` require ` 加载模块，语法如下：

    
    
      
    //  模块定义  
      
    define(id?, dependencies?, factory);  
      
      
      
    // 模块加载  
      
    require([module], callback);  
      
      
      
    

` AMD ` 提供了一种异步加载模块的机制，旨在解决前端模块化中的依赖管理和加载顺序问题，从而与 ` CommonJS `
在不同环境下的模块化需求形成了有益的补充。 ` RequireJS ` 遵循 ` AMD ` 规范，为客户端 ` JavaScript `
模块化开发提供了简化的解决方案。

###  CMD

` CMD ` （ ` Common Module Definition ` ）是玉伯在开发 ` SeaJS ` 的时候提出来的， ` SeaJS `
要解决的问题和 ` RequireJS ` 一样。不同于 ` AMD ` 的依赖前置， ` CMD ` 是就近依赖。且看下面这个案例：

    
    
      
    // AMD  
      
      
      
    define(['a', 'b'], function(a, b) {  
      
       // 模块 a 和 b 在这里就都执行好并可用了  
      
    })  
      
      
      
    // CMD  
      
      
      
    define(function(require, exports) {  
      
       // ...  
      
       var a = require('a')  // 模块 a 运行到此处才执行  
      
      
      
       // ...  
      
      
      
       if (false) {  
      
          var b = require('b')   // 当某些条件为 false 时，模块 b 永远也不会执行  
      
       }  
      
    })  
      
    

` AMD ` 强调异步加载和预定义依赖，适用于需要提前加载模块的场景。 ` CMD `
则支持按需加载，提供了更大的灵活性，允许在模块内部动态引入依赖。尽管两者有差异，但它们的出现都推动了客户端 ` JavaScript ` 模块化的发展。

###  UMD

` UMD ` 即通用模块定义（ ` Universal Module Definition ` ）。为了兼容 ` CommonJS ` ` 和AMD `
， ` UMD ` 规范被引入。它允许同一模块在不同环境中运行，既可以在 ` Node.js ` 中使用，也可以在浏览器中使用。

核心代码实现如下：

    
    
      
    (function (root, factory) {  
      
        if (typeof define === 'function' && define.amd) {  
      
            // AMD  
      
            define(['dependency'], factory);  
      
        } else if (typeof module === 'object' && module.exports) {  
      
            // Node, CommonJS-like  
      
            module.exports = factory(require('dependency'));  
      
        } else {  
      
            // Browser globals (root is window)  
      
            root.myModule = factory(root.dependency);  
      
        }  
      
    }(this, function (dependency) {  
      
        return {  
      
            greet: function() {  
      
                console.log('Hello, UMD');  
      
            }  
      
        };  
      
    }));  
      
      
      
    

` UMD ` 本质上就是帮你判断应该用 ` AMD ` 还是 ` CommonJS ` ，是哪个就用哪个方式来定义模块，都不是的话就挂到全局对象上。

###  ES Module

在 ` ES Module ` 之前， ` JavaScript ` 并没有官方的模块化机制，开发者依赖于像 ` CommonJS ` 、 ` AMD `
和 ` UMD `
等第三方规范来实现模块化。这些规范虽然有效，但都存在一些局限性，且在浏览器和服务器端的实现上有所不同，导致代码复用性和跨环境的兼容性变得复杂。

为了统一 ` JavaScript ` 的模块化标准，同时满足现代 ` Web ` 开发的需求，TC39（ ` ECMAScript ` 标准委员会）在 `
ES6 ` 中正式引入了 ` ES Module ` 。

> ES6 模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS 和 AMD
> 模块，都只能在运行时确定这些东西。比如，CommonJS 模块就是对象，输入时必须查找对象属性。

基本使用示例如下：

    
    
      
    /* a.js */  
      
    export const add = (a, b) => {  
      
      return a + b  
      
    }  
      
      
      
    /* main.js */  
      
      
      
    import { add } from "./a.js"  
      
    add(1, 2)  
      
      
      
    

` ES Module ` 的引入标志着 ` JavaScript `
正式进入了模块化开发的时代。它统一了模块的定义和使用方式，使得代码的复用性和维护性大大增强。此外， ` ES Module `
还推动了前端开发工具链的发展，比如 ` Webpack ` 、 ` Rollup ` 等工具都基于 ` ES Module `
进行打包和优化，极大地提升了开发效率和代码性能。

##  总结

从 ` IIFE ` 的模块化雏形到 ` ES Module ` 的成熟标准， ` JavaScript ` 模块化技术的发展不仅反映了 ` Web `
开发的不断演进，也预示着未来技术将更加模块化和组件化。同时，这一进程也体现了 ` JavaScript `
从简单脚本语言转变为支持复杂应用的现代编程语言的历程。

##  相关参考

前端模块化开发那点历史

Js模块化发展

RequireJs及AMD规范

  

预览时标签不可点

微信扫一扫  
关注该公众号





****



****



×  分析

  收藏

