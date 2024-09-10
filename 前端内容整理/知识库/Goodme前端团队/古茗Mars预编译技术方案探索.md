![cover_image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicGdatIJficZGja3v0M9uYV22v4oxAArqP62mYGnmVUotbkv3e3PSia94eodPKY8ULVE2Kia3FZEPKlxA/0?wx_fmt=jpeg)

#  古茗 Mars 预编译技术方案探索

原创  陈杰  [ Goodme前端团队 ](javascript:void\(0\);)

__ _ _ _ _

##  前言

提升编译效率是前端基建中一个绕不开的话题，古茗自从在团队中落地 中后台框架 Mars 后也在积极探索有效的编译提速方案。

##  探索方向

通常来说，提升前端编译效率可以从以下 3 个方向入手，俗称“编译提速 3 板斧”：

  * **使用缓存** ：将编译结果缓存起来，下次就可以跳过编译，直接使用缓存结果，缓存又可以分为 **内存缓存** 和 **持久化缓存** 。常见的 babel-loader、webpack 都可以启用缓存以提升编译效率。 
  * **Native code** ：对于编译这种 cpu 密集型任务，js 的执行效率通常不如 c/c++、rust、go 等这类可以编译成原生二进制的语言，所以使用这类语言编写的编译器效率会更高，例如 swc、esbuild、rspack 等工具都是借助了该类语言的特性来提速。 
  * **少编译** ：通常是使用“按需编译”的策略来减少编译的范围，例如 vite 就是该类工具的代表。 

以上三个优化方向我们或多或少都有所涉及，本文主要是分享我们在基于 native code 进行依赖预编译方向上的探索过程。

##  预编译方案

古茗 Mars 预编译的实现方案参考了 Taro 依赖预编译、UmiJS MFSU 等社区方案，并结合实际场景加以优化实现。 **大致流程如下** ：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGdatIJficZGja3v0M9uYV22p8ydDOYFMH7JibMYlm6KeQxpXxCQARxAkrQjVcuicYq3axQnGo9DhrAw/640?wx_fmt=png&from=appmsg)
流程图

###  1\. 扫描依赖

使用 esbuild 编译项目源代码，并通过实现一个自定义 esbuild 插件来收集所有的 ` node_modules `
目录下的依赖模块，插件的示例代码如下：

    
    
    import esbuild from 'esbuild';  
      
    const deps = new Set<string>();  
      
    // 扫描插件  
    export function getScanImportsPlugin() {  
      return {  
        name: 'ScanImportsPlugin',  
        setup: (build) => {  
          // 解析 npm 包模块路径  
          build.onResolve({ filter: /^[^./\\][^:]/ }, async (args) => {  
            const { path: id, resolveDir } = args;  
      
            // 解析模块绝对路径  
            const filePath = require.resolve(id, { paths: [resolveDir] });  
      
            // 如果路径包含 node_modules，则识别为预编译的模块  
            if (filePath.includes('node_modules')) {  
              deps.add(id);  
              return { external: true };  
            }  
      
            return { path: filePath };  
          });  
      
          // 处理静态资源  
          build.onResolve({ filter: /\.(css|less|sass|scss|png|jpg|jpeg|gif|svg|json)$/i }, () => {  
            return { external: true };  
          });  
        },  
      };  
    }  
    

在上述示例中，解析模块路径暂时通过 ` require.resolve() ` 方法来简单处理了，但是在实际场景中会遇到很多问题。主要是在以下 2 类场景：

  * 用户自定义配置的 alias 别名 
  * 定义在 npm 包 ` package.json ` 中的一些字段会影响模块路径解析，例如： ` module ` 、 ` main ` 、 ` browser ` 、 ` exports ` 等，而且可能还要兼容历史的规范 

针对以上问题，我们使用了 enhanced-resolve 来解析模块路径，这也是 webpack 内置的模块解析工具，可以更容易与 webpack
的模块解析行为保持一致。

###  2\. 编译依赖

再次使用 esbuild 工具编译第 1 步中扫描到的依赖模块。这个过程核心需要关注的问题是， **如何将预编译的产物和正常编译的产物进行结合**
。对于正常编译（webpack）来说，就是要让 webpack 将已经预编译的模块排除掉，并能在预编译产物中找到对应的模块。

通常来说，有以下 3 种方式来达到这一目的：

  * 通过 webpack ` externals ` 排除预编译的模块，并将预编译产物的模块变量暴露在全局 ` window ` 对象上并与之关联。这是一种暴力解法，不优雅，会暴露太多全局变量 
  * 使用 webpack5 模块联邦做桥接，Taro 的依赖预编译方案就是使用此方式。但需要魔改模块联邦插件，实现复杂度高，最关键的是需要额外使用 webpack 将预编译产物构建一次，这会降低预编译效率 
  * 使用 DllReferencePlugin 插件和 DllPlugin 插件做桥接。同样的，这种方式也需要额外将预编译产物使用 webpack 构建一次 

最终调研下来发现使用 ` DllReferencePlugin ` 插件去桥接这种方式比较合适。但是不同之处在于，我们 **不会使用` DllPlugin
` 插件去构建预编译产物，而是使用 esbuild 编译直出产物（通过伪造成 DllPlugin 产物的方式） ** ，这种方式会大幅提升预编译的构建效率。

####  伪造 ` DllPlugin ` 插件产物

尝试使用 ` DllPlugin ` 插件去构建一个 demo，发现主要会生成一个 dll.js 文件（bundle）和一个 manifest.json
清单文件，文件代码大致如下：

**dll.js**

    
    
    var dll = (function(e) {  
      // ...   
    }({  
      0: function(e, t, n) {  
        // 模块 ID 为 0 的模块对应的代码  
      },  
      141: function(e, t, n) {  
        // 模块 ID 为 141 的模块对应的代码  
      },  
      // ... 此处省略剩下的模块对应的代码   
    }));  
    

**manifest.json**

    
    
    {  
      "name": "react_dll",  
      "content": {  
        "./node_modules/react/index.js": { "id": 0, "buildMeta": { "providedExports": true }},  
        "./node_modules/object-assign/index.js": { "id": 141, "buildMeta": { "providedExports": true }}  
      }  
    }  
    

稍加观察可以发现一些规律，如在 manifest.json 文件中 ` content ` 的 key 是模块的本地模块相对路径，而它的 ` id ` 又与
js 产物的模块 id 关联…，所以我们就可以基于第一步扫描出来的依赖伪造出 **manifest.json** 文件及构建产物 **vendor.js**
。

预编译产物 vendor.js 是由 esbuild 构建一次生成，以下是使用 esbuild 进行预编译的入口代码示例，可以由 esbuild
插件去动态组装入口代码：

    
    
    // 预编译的模块映射  
    const modules = {  
      './node_modules/foo.js': /* module implement for: foo.js */,  
      './node_modules/bar.js': /* module implement for: bar.js */,  
      // ...更多预编译模块  
    };  
      
    const cache_exports = {};  
      
    module.exports = (id) => {  
      let value = cache_exports[id];  
      if (value) return value;  
      
      const module_require = modules[id];  
      if (module_require) {  
        return (cache_exports[id] = module_require());  
      }  
      
      throw new Error('Cannot found pre-bundled module: ' + id);  
    }  
    

####  预编译缓存

从上述生成 **伪造` DllPlugin ` 插件产物 **
的流程中可以看出来，唯一影响预编译产物的因素是扫描出来的依赖模块。所以可以根据这一特征做预编译缓存，进一步提升预编译效率。

于是我们做了这样的 **缓存 hash 值生成策略** ：

  1. 将扫描到的预编译模块排序，避免由引入顺序的不同而导致破坏缓存结果 
  2. 依次读取依赖模块的 npm 包名称 ( ` name ` ) 和版本号 ( ` version ` )，如果无法找到时会降级使用模块文件内容替代，将它们依次添加至生成 hash 的源字符中 
  3. 将其他可能影响缓存结果的变量添加至 hash 源字符中，例如：环境变量、编译器版本、配置文件等 
  4. 使用 md5 摘要算法将源字符生成为缓存 hash 字符串 

缓存 hash 值生成完成后，保存在 manifest.json 中。每次扫描依赖完成，与之前生成的缓存 hash
值进行对比，如果对比一致则可以复用预编译产物，否则就需要重新执行预编译构建。

###  3\. 适配 webpack 编译

主要分成 2 个部分：

  1. 根据 **manifest.json** 配置 ` DllReferencePlugin ` 插件，将预编译的模块排除构建 
  2. 将 **vendor.js** 发送至产物目录，并作为前置依赖优先被 html 加载即可。这里我们通过一个 webpack 插件，调用 compilation.emitAsset 或 compilation.updateAsset API 发送一个新的产物文件，使用 add-asset-html-webpack-plugin 插件可以将预编译产物的 js 文件添加至 html 中优先被加载。 

###  4\. 其他

以上 3 步介绍了预编译方案的总体流程，实际落地过程中还遇到了很多问题，我们把几个关键的问题以 Q&A 的形式梳理在下面：

####  Q: 依赖中引入的 CSS、图片等资源文件如何处理

**A** : 由于预编译的实际产物只有 **vendor.js** 文件，所以是无法直接处理其他资源文件。对此，我们的解法是把这类资源文件作为 `
externals ` 记录在 **manifest.json** 中，同时在 webpack 编译入口处添加如下代码：

    
    
    const host = _mpb_app_host_;  
      
    // Register pre-bundle external modules  
    host["./node_modules/foo.css"] = require("../foo.css");  
    host["./node_modules/bar.png"] = require("../bar.png");  
    

借助 webpack 的模块构建能力去处理资源文件。当 webpack 编译完成后，所有预编译依赖的资源文件模块将被保存至 ` _mpb_app_host_
` 全局变量中，这样在使用 esbuild 编译静态资源的时候，只需引用 ` _mpb_app_host_[id] ` 即可。

####  Q: 如何处理 webpack 自定义的 ` externals ` 配置

**A** : webpack 会将 ` externals `
配置中的模块排除构建，并支持通过各种模块规范或全局变量等方式引入该模块。为简化实现复杂度，我们仅支持通过全局变量方式引入。预构建的 esbuild
插件伪代码如下：

    
    
    export function getPreBundlePlugin() {  
      return {  
        name: 'PreBundlePlugin',  
        setup: (build) => {  
          // load external module  
          build.onLoad({ filter: /^!external\// }, async () => {  
            // 省略一些逻辑，将 externals 转换配置成 runtime 代码  
      
            // Note: `someGlobalVar` 替换成真实场景的全局变量  
            return {  
              contents: `module.exports = window.someGlobalVar`,  
            };  
          });  
        },  
      };  
    }  
    

####  Q: 如何复用 ` @babel/runtime ` 运行时

**A** : 在 npm 包中有可能会引入 ` @babel/runtime ` 运行时（只要使用 babel 编译的 npm 包），而项目中也会引入 `
@babel/runtime ` ，所以这 2 部分的 ` @babel/runtime ` 运行时代码可能会重复引入，导致代码体积增大。

在开发环境中，最注重的是编译效率，而非代码体积，所以开发环境会直接忽略这个问题。但在构建压缩环境时，会更注重代码体积，这时会把 `
@babel/runtime ` 模块当作资源文件模块，交给 webpack 去编译。

####  Q: 如何调试 ` node_modules ` 目录下的 npm 包模块

**A** : 预编译后 ` node_modules ` 模块会被缓存且会被 webpack
忽略编译，这就意味着在开发阶段编辑这些文件时将不会触发热更新，无法实时调试 npm 包的模块代码。这个问题我们为 MacOS 和 Windows 系统使用了
2 套不同的策略：

  * **MacOS** : 预编译完成后，使用 chokidar 监听依赖目录变化，当监听到变化时重新执行预编译，再触发 webpack 编译 
  * **Windows** : 最开始也使用 chokidar 监听方案，但实测下来发现 windows 系统会占用大量内存。所以不得不使用手动模式，监听控制台输入 ( ` process.stdin ` ): ** ` PB ` **，监听输入后重新执行预编译逻辑 

##  优化效果

分别对小型项目（18个页面）和大型项目（108个页面）进行实测，得到如下的基准测试数据（测试电脑为：2018款 MacBook Pro i7 处理器）：

模式  |  页面数量（个）  |  无缓存  |  有缓存   
---|---|---|---  
普通编译  |  18  |  **26.5s** |  **5.9s**  
启用预编译  |  18  |  1.9+13= **14.9s** |  0.2+2.9= **3.1s**  
普通编译  |  108  |  **95.8s** |  **9.7s**  
启用预编译  |  108  |  3.4+56.9= **60.3s** |  0.7+4.9= **5.6s**  
  
> **提示** ：启用预编译后的编译时间为 **预编译时间** \+ **webpack 编译时间** ，相加之和为最终的实测编译时间。

**结论** ：从上面的实测数据可以看出：

  * 启用缓存对编译效率的提升是非常大的（这里主要是 webpack、babel 持久化缓存带来的收益） 
  * 启用预编译后大约可以提升 **40%** 的编译速度 
  * 预编译的耗时是非常少的，实测数据（无缓存）大致在 **2s ~ 4s** 范围内 

##  总结

本文主要介绍了古茗在中后台 Mars 框架下的预编译方案，在此要特别感谢 Taro 依赖预编译 提供给的灵感。此外，基于 Mars
框架的静态化路由设计，我们在团队内部还使用了 **路由按需编译** 策略来优化首次编译效率，使大型项目首次无缓存编译时间 **从 60s 减少至 20s**
。

##  小茗推荐

  * 门店：“电脑又双叒叕中病毒了” 
  * 钉钉小程序+F2图表库的踩坑指南 
  * 去掉状态管理工具（formily）的 observer 

##  最后

关注公众号「Goodme前端团队」，获取更多干货实践，欢迎交流分享。

  

预览时标签不可点

微信扫一扫  
关注该公众号





****



****



×  分析

  收藏

