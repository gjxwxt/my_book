![cover_image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicGyz5jM0xIF7UeXkz3DwkzavXdToMu7swicSucjY7icPGqgGlOD8fWAqSytGwetQUTAzsDK3JP4RaqA/0?wx_fmt=jpeg)

#  探究前端包管理工具：npm、yarn 和pnpm

原创  宋永杰  [ Goodme前端团队 ](javascript:void\(0\);)

__ _ _ _ _

> 作者：宋永杰

##  引言

对于包管理器，不同语言其实都有自己的包管理器，比如：Python/Rust有自己的 **包管理器** （pip/cargo），还有如rpm、maven等。

同样在现代前端开发中，bower、npm、yarn、cnpm、pnpm等各种包管理器，简化了资源引用的依赖关系，提升了我们的开发效率。本文将从包管理器的发展史和当下主流的三种工具：npm、yarn和pnpm来做一个全方位的分析和对比，探讨各自优点和适用场景。

##  远古时期

nodejs诞生之前，我们想要引用一些三方资源库，比如jquery，经常使用以下方式：

  * 远程下载zip压缩包，解压以后将资源文件放入项目中，并进行引用。 
  * 通过cdn的方式，将资源链接以script标签引入到html中。 

随之就会出现版本管理混乱、项目文件过大、cdn资源失效、依赖升级等各种问题。

随着nodejs的爆火和模块化概念的诞生，npm出现了，最初npm只是用于服务端nodejs的包管理器，随着前端社区的不断发展，npm也使用在了客户端开发中。

那么当包管理工具出现后，是怎么逐步解决上述问题呢？这就得从它的发展史聊起了

##  包管理工具发展史

###  npm

####  npm v3之前

2011年7月，npm发布了1.0版本。当时的node_modules文件夹是什么样子呢？

    
    
    node_modules  
    └─ foo  
       ├─ index.js  
       ├─ package.json  
       └─ node_modules  
          └─ bar  
             ├─ index.js  
             └─ package.json  
    

#####  存在的问题

  * node_modules文件夹体积过大，比如当多个项目同时引用lodash，就会在node_modules多次安装lodash，很快就会把计算机的磁盘占满，不得不经常通过 rm -rf node_modules 来删除node_modules。 
  * 嵌套层级过深，只有当找到不依赖任何包的叶子节点，才会停止，会导致路径过长，在windows会出现删除node_modules失败问题。 
  * 安装速度很慢，有目录嵌套的原因，也有安装逻辑的问题，按照队列下载，这就会导致同一个时刻，只有一个模块在下载、解析、安装。 

####  npm V3

为了解决上述问题，npm团队认真思考了node_modules的结构，并提出了扁平化的策略，就是把嵌套过深的层级，通过扁平化的方式，将依赖包进行提升，使嵌套层级尽可能的变少。在npm
v3阶段，node_modules的结构如下：

    
    
    node_modules  
    ├─ foo  
    |  ├─ index.js  
    |  └─ package.json  
    └─ bar  
       ├─ index.js  
       └─ package.json  
    

虽然通过扁平化策略，确实减少了部分嵌套依赖太深和重复安装的问题，为什么说是解决部分问题呢？看下图：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicEFc5f4LoLBgaVMeJCpaTEicZWJbD8vkedcia5KLrKtbMbfHncFLwob7VOmrySe1aFJkm7yvOsh7QlQ/640?wx_fmt=png&from=appmsg)

可以看到，项目中同时依赖b1.0.0和b2.0.0，只有b1.0.0提升安装在顶层，b2.0.0照样还是会重复安装。实际上b2.0.0也有可能被提升安装在顶层，b1.0.0重复安装。并且决定这个提升顺序遵循的是先到先安装的策略，所以存在很多的不确定性。

#####  存在的问题

  * 没有彻底解决重复安装的问题; 
  * 存在幽灵依赖问题，比如在安装时把c@1.0.0提升到了顶层，但是在package.json中并没有声明，项目中照样可以引用c@1.0.0; 
  * 不支持离线缓存模式，安装速度慢; 

###  yarn

yarn的出现，可以说是从根本上解决了npm存在的很多问题，比如资源一致性、安装速度慢等问题。

####  资源一致性解决方案：版本锁定

yarn的出现，我觉着最大的贡献就是推出了yarn.lock，来解决依赖版本错乱的问题，npm在一年后的npm@5也跟上yarn的脚步，推出了package-
lock.json。

####  npm V5和yarn在处理扁平化的方式上的区别：

    
    
    // 在一个项目中存在如下依赖：  
    node_modules  
    ├─ htmlparser2@^3.10.1  
    |  ├─ entities@^1.1.1  
    └─ dom-serializer@^0.2.2  
    |  ├─ entities@^2.0.0  
    └─ entities@^2.1.0   
    

通过npm install安装依赖后，生成的package-lock.json和node_modules结构如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicEFc5f4LoLBgaVMeJCpaTEiccAzm08WwAA79j1DCHROqaDQEDUgvEEfoNVrXGTl1sg9SlYQCuAeLNg/640?wx_fmt=png&from=appmsg)

  

通过yarn安装依赖后，生成的yarn.lock和node_modules结构如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicEFc5f4LoLBgaVMeJCpaTEicCv9FicP3Hxn5fmNicPsF3EWl2v75Kic9mmwtb1iarnTT6sS85V5Ylogsicg/640?wx_fmt=png&from=appmsg)

  

对比可以看到：

  * yarn.lcok文件中，所有的依赖项描述都是扁平化的，结构简单明了; 
  * 在yarn.lock中，相同名称版本不同的依赖包，如果 **semver** 范围相同会被合并，同时会存在多个版本描述; 
  * yarn 在生成 **yarn.lock** 文件时，使用更严格的版本解析算法，会确切地记录每个依赖项的版本。这意味着无论何时重新安装依赖，yarn 都会使用相同的版本，从而确保了依赖版本的一致性; 

####  semver规范

SemVer 是指语义版本规范（Semantic Versioning），用来约定包版本格式。它由三部分组成：主版本号、次版本号和修订版本号。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicEFc5f4LoLBgaVMeJCpaTEicZ6iaX15g4QCfGtx5Cmbvo7jO4ib7bAxqR3w4YQibyPDibEvSfvwSo0ic5xQ/640?wx_fmt=png&from=appmsg)

  * **主版本号(MAJOR)：** 当升级API无法进行向下兼容，会破坏现有代码功能的时候，必须升级主版本号。 
  * **次版本号(MINOR)：** 添加了向下兼容的功能，可以升级次版本号。此时意味着增加了新功能，但是不影响现有功能使用。 
  * **修订版本号(PATCH)：** 当进行向下兼容的bug修复，可以升级修订版本号。这意味着新版本只是对之前版本中的错误进行了修复，没有添加新的功能，且与之前的版本兼容。 
  * **预发版本号和版本构建号(TAG)：** 通常使用连接符“-”和“+”来连接，比如：2.1.3-beat.1+build3.2 

####  yarn是否把npm拍死在沙滩上？

实际上，yarn本质上还是在下载npm包，只是针对npm v3中的痛点，针对性的做了优化：

#####  缓存机制：

  * yarn使用一个全局的缓存目录来存储所有依赖项，而npm使用分散的缓存目录结构。这样使得yarn更加易于管理和维护。- yarn拥有离线模式，当你在命令行敲下 **yarn install** 时，会首先尝试使用本地缓存，当你之前已经缓存过这些依赖项，那么在离线模式下也能安装。 
  * 并行安装：yarn在设计之初就考虑到了并行安装依赖，默认使用多线程来下载和安装依赖包，使得安装速度更快。 
  * 版本锁定更加稳定：如上分析，yarn.lock的文件更加扁平化和准确，能够最大限度避免多个版本依赖问题。 

社区里有人针对yarn和npm的性能做了对比（来源于：github.com/appleboy/npm-vs-yarn）：

  
|  npm install  |  npm ci  |  yarn   
---|---|---|---  
install without cache (without node_modules)  |  3m  |  3m  |  1m   
install with cache (without node_modules)  |  1m  |  18s  |  30s   
install with cache (with node_modules)  |  54s  |  21s  |  2s   
install without internet (with node_modules)  |  \-  |  \-  |  2s   
  
###  pnpm

####  为什么被称为最先进的包管理工具?

pnpm项目建立的初衷：

  * 节省磁盘空间 
  * 提高安装速度 
  * 创建一个非扁平的node_modules目录 

####  节省磁盘空间

在使用npm进行依赖安装的时候，不同项目有相同依赖的时候，都会被重复安装。在使用 pnpm
时，依赖会被存储在内容可寻址的存储仓库(store)中，采用store + hardLink的方式：

  * 当项目中引用了某个依赖项的不同版本，那么pnpm在安装的时候，只会将不同版本中的差异文件添加到store中。比如我们项目中的依赖的新版本只更改了其中1个文件，那么pnpm update的时候只会向store中更新这1个文件，而不会更改整个依赖包文件。 
  * 所有的依赖包文件都存储在全局的store目录下，当某个项目在安装依赖时，会通过硬链接的方式将依赖资源链接到项目中，而不会再次重复安装依赖包，不占用额外的磁盘空间。 

####  提高安装速度

上面提到过，pnpm采用store + hardLink的方式进行依赖的管理和安装。

  * 当一个项目中有多个相同的依赖包时，pnpm只需要下载一次，然后通过hardLink的方式进行不同项目中的引用。 
  * pnpm使用并行方式安装依赖项，可以同时下载多个依赖项，进一步提升安装速度。 

####  创建一个非扁平的node_modules目录

使用 npm 或 Yarn 安装依赖项时，所有的包都被提升到模块目录的根目录。这样就导致了一个问题，源码可以直接访问和修改依赖，而不是作为只读的项目依赖。

首先来看下pnpm是怎么解决嵌套依赖问题的：

    
    
    -> - a symlink (or junction on Windows)  
      
    node_modules  
    ├─ foo -> .registry.npmjs.org/foo/1.0.0/node_modules/foo  
    └─ .registry.npmjs.org  
       ├─ foo/1.0.0/node_modules  
       |  ├─ bar -> ../../bar/2.0.0/node_modules/bar  
       |  └─ foo  
       |     ├─ index.js  
       |     └─ package.json  
       └─ bar/2.0.0/node_modules  
          └─ bar  
             ├─ index.js  
             └─ package.json  
    

在pnpm 创建的 _node_modules_ 文件夹中，所有包都有自己的依赖项分组在一起，但目录树永远不会像 npm@2 那样深。pnpm
保持所有依赖关系平坦，但使用符号链接将它们分组在一起。

####  性能对比

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicEFc5f4LoLBgaVMeJCpaTEicgGckWzPNr5G8RUIAMTtJbv7jqRfj5JqRfgkONgZ5k5FTfhwo0tStKg/640?wx_fmt=png&from=appmsg)

####  pnpm的局限性

以下是来自官网的描述：

  1. npm-shrinkwrap.json 和 package-lock.json 被忽略。与 pnpm 不同，npm可以多次安装相同的 name@version ，并且具有不同的依赖项组合。npm 的锁文件旨在反映平铺的 node_modules 布局，但是，由于 pnpm 默认创建隔离布局，它无法由 npm 的锁文件格式反映出来。但是，如果您希望将锁定文件转换为 pnpm 的格式，请看 pnpm import (https://pnpm.io/zh/cli/import)。 
  2. Binstubs（在 node_modules/.bin中的文件）总是 shell 文件，而不是指向 JS 文件的符号链接。创建 shell 文件是为了帮助支持插件的 CLI 的程序在特殊的 node_modules 结构中能够正确地找到它们的插件。这是很少有的问题，如果您希望文件是 JS 文件，请直接引用原始文件，如 #736 (https://github.com/pnpm/pnpm/issues/736) 所示。 

##  总结

npm、yarn和pnpm都是当下十分优秀的包管理工具，具体选择哪个，还是要根据团队项目情况和个人喜好来决定。npm 是 Node.js
生态系统的一部分，yarn 提供了更快的依赖项安装和锁定文件功能，而 pnpm 则专注于减少磁盘空间的使用和安装时间。

附上三者在一些功能上的比较（https://pnpm.io/zh/feature-comparison）：

功能  |  pnpm  |  Yarn  |  npm   
---|---|---|---  
工作空间支持（monorepo）  |  ✔️  |  ✔️  |  ✔️   
隔离的 node_modules  |  ✔️ - 默认  |  ✔️  |  ✔️   
提升的 node_modules  |  ✔️  |  ✔️  |  ✔️ - 默认   
自动安装 peers  |  ✔️  |  ❌  |  ✔️   
Plug'n'Play  |  ✔️  |  ✔️ - 默认  |  ❌   
零安装  |  ❌  |  ✔️  |  ❌   
修补依赖项  |  ✔️  |  ✔️  |  ❌   
管理 Node.js 版本  |  ✔️  |  ❌  |  ❌   
有锁文件  |  ✔️ - pnpm-lock.yaml  |  ✔️ - yarn.lock  |  ✔️ - package-lock.json   
支持覆盖  |  ✔️  |  ✔️ - 通过 resolutions  |  ✔️   
内容可寻址存储  |  ✔️  |  ❌  |  ❌   
动态包执行  |  ✔️ - 通过 pnpm dlx  |  ✔️ - 通过 yarn dlx  |  ✔️ - 通过 npx   
辅助缓存  |  ✔️  |  ❌  |  ❌   
列出许可证  |  ✔️ - 通过 pnpm licenses list  |  ✔️ - 通过插件  |  ❌   
  
  

预览时标签不可点

微信扫一扫  
关注该公众号





****



****



×  分析

  收藏

