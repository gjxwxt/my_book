![cover_image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicF7u2P0XpcPLZ4ZicmOFXICw2eeO5HRt3lddBdnI8dH5ibr6nNp43biadQ0HepbBQPPwpFhUicDV386MQ/0?wx_fmt=jpeg)

#  taro4.0支持vite？让我来试试

原创  张云重  [ Goodme前端团队 ](javascript:void\(0\);)

__ _ _ _ _

###  前言

` taro4.0 ` 已经beta版本好久了，去年底就关注到了相关的立项，据说今年的第二季度会发布正式版本。 ` 4.0 ` 我自己比较关注的升级有三处

  * 支持鸿蒙平台 
  * 微信小程序运行时支持 ` CompileMode `
  * 构建链路支持 ` vite `

前面两项直观上是能接受的，第三项什么鬼？仅支持H5还可以理解，但是看官方的意思是要全平台支持

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicF7u2P0XpcPLZ4ZicmOFXICwG6iaCeJV7eWKEzmHcemcX88GFwI3Lcia74CW9ribib7da0mFYjFPExxzxQ/640?wx_fmt=jpeg&from=appmsg)

要知道小程序是不支持原生 ` esm ` 也不支持加载外部 ` js ` 资源，偏偏 ` vite `
又是基于浏览器原生es模块能力来提升构建效率的，这么看来这俩东西根本就是水火不容啊，那 ` taro4 `
是如何进行支持的呢？真是极大的激起了我学习的欲望，让我来一探究竟。

###  创建taro4模版仓库

源码放一边，先来跑个 ` taro4 ` 的demo看看吧。

前置条件 ` node ` 版本需要 ` >= 18 ` ，然后通过 ` taro cli ` 创建模板

    
    
    $ yarn global add @tarojs/cli@beta   
    $ taro init taro4-demo  
    

安装beta版本会污染本地的taro版本，也可以指定一个文件夹安装

    
    
    $ mkdir taro4 && cd ./taro4   
    $ yarn add @tarojs/cli@beta   
    $ npx taro init taro4-demo  
    

下面是我创建模板时各条选项

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicF7u2P0XpcPLZ4ZicmOFXICwNlOgHiau72pvuBUq1yahhBVUJAcXEV8Ne92F6lUOsdk3bz6piaHfQicgQ/640?wx_fmt=jpeg&from=appmsg)

ok，然后我们来 ` run dev ` 吧，nice 熟悉的 ` vite log ` 出来了，说明构建跑通了，熟悉的 ` dist `
也已经产出了，然后IDE打开试试。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicF7u2P0XpcPLZ4ZicmOFXICwEg0flPrHfibqwt2L3rzb0H2JaSL9Lv1qZRkN4CMIb2BQEvxv8icCaIkA/640?wx_fmt=jpeg&from=appmsg)

如果不出意外就出意外了，用IDE是挂的（白屏报错）~(⊙︿⊙) 微信、支付宝都不行

这...还没开始已经结束了，默认模版就跑不起来

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicF7u2P0XpcPLZ4ZicmOFXICw9ibiaWflUzn7fClic2NsxqFRSBMvp4nLjNhpZFeBAwl1NXQBGxWN167wA/640?wx_fmt=jpeg&from=appmsg)

小小对比排查了一下，发现默认模版 ` babel target ` 配置有些问题，导致 ` dist/vendors.js ` 再给平台解析时出错了。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicF7u2P0XpcPLZ4ZicmOFXICwicwFvPXAp83X5dvplVHxCDokspfMUcDLCYcxicZeGrkviar7QIxyefjQA/640?wx_fmt=jpeg&from=appmsg)

替换成如下即可解决，或者第一步创建模板时不要选择默认模板，选择 ` mobx ` 模板

    
    
    ...  
    "browserslist": [  
      "last 3 versions",  
      "Android >= 4.1",  
      "ios >= 8"  
    ],  
    ...  
    

重新 ` run dev `
在试试，wk，居然真的run起来了，IDE页面也都一切正常；编辑一下代码保存，居然热更新也都ok，居然都可以跑通？怎么玩的？？？全是问号

此时此刻我的前端知识体系已经被摧毁了~~ ` taro ` 和 ` vite ` 还真能一家亲。别慌冷静，在接着分析看看

> 这里提示下，低版本微信小程序IDE热更新失效，需要升级到最新版本

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicF7u2P0XpcPLZ4ZicmOFXICwicXqZyib2jP4chSXTgXuVKP7Sgz1vPIlxZqIZiayOOhe57EmqGYfRXsJQ/640?wx_fmt=jpeg&from=appmsg)

###  产物分析

目录和之前 ` taro webpack ` 一毛一样，点开 ` dist/app.js ` 瞅瞅吧

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicF7u2P0XpcPLZ4ZicmOFXICwicWXrF8GaRtfJAbbRdCfrzoOe8nzjDcnmFicLquq7lwYaEPnOIYWF4eA/640?wx_fmt=jpeg&from=appmsg)

啊，这怎么和我理解的 ` vite ` 产出不一样呢？怎么还是 ` require ` ，不是应该满屏的 ` import `

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicF7u2P0XpcPLZ4ZicmOFXICwdTKGaHicOYp8UcSNA2SB0WeEibMKLGJ2elDmc5EzF38tJQ9gHeNY2ibHw/640?wx_fmt=jpeg&from=appmsg)
image.png

` vite dev ` 模式下的产出不应该全是标准的 ` es module ` 吗？下图的代码来自 ` vite ` 官方模版中的 ` main.js
`

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicF7u2P0XpcPLZ4ZicmOFXICwXBfGYx4IsXBicg5sj5mD8feeKL4Mv86fCUrlPEU8uEtic8DRWVV1IdCw/640?wx_fmt=jpeg&from=appmsg)
image.png

在来对比下 ` taro webpack ` 的产出看看呢？下图中除了 ` webpack module ` 相关的逻辑，其他逻辑和 ` taro vite
` 版本好像都是一致的

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicF7u2P0XpcPLZ4ZicmOFXICwicag5Cg11MibExP9FQhuIRaVGSB6a1SrKXZvKicicAEpW6lsN0DQmgUn5A/640?wx_fmt=jpeg&from=appmsg)
image.png

哦？难道打通了微信小程序在 ` require `
里面添加了黑科技吗？赶紧去看下network，失望透顶，空空荡荡一个外部资源都没有，看来肯定都是本地资源了

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicF7u2P0XpcPLZ4ZicmOFXICwkH9aaPD5Ru64QBcNINbsSDm6JGEoicPJGVvibN6IqnJFsWZ67uK0M9Qw/640?wx_fmt=jpeg&from=appmsg)
image.png

对比下 ` vite ` 中的 ` network ` 面板，对啊，这才是熟悉的 ` vite `

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicF7u2P0XpcPLZ4ZicmOFXICwIsKyLODDia7sTbhFDQmeEibabmyJqBNCuHRibPWY2Uv0BukngrEIWMibSw/640?wx_fmt=jpeg&from=appmsg)
image.png

这...

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicF7u2P0XpcPLZ4ZicmOFXICwFiaRlWQSUfwb2ItOwkqRspXw2ibHTZgHby0bsibCoGc6tLXMnzHO0RFcg/640?wx_fmt=other&from=appmsg)

那 ` taro4 vite ` 做了啥？我想象中 ` vite ` 的一切，esm、按需编译、更快的热更新速度... 在看到 ` require `
那一刻全都破灭了，难不成 ` taro4 vite ` 就是一个大号的 ` rollup ` 吗！！不行，我要再去看看源码确认下

###  源码 - vite-runner

关于 ` taro4 vite ` 核心的逻辑都在这个包里面 ` packages/taro-vite-runner ` -> `
@tarojs/vite-runner `

找到 ` vite build ` 入口，先来看看 ` vite config ` 吧

    
    
    // taro/packages/taro-vite-runner/src/index.mini.ts  
      
    ...  
    import { build } from 'vite'  
    ....  
      
    export default async function (appPath: string, rawTaroConfig: ViteMiniBuildConfig) {  
      ...  
      
      console.log(JSON.stringify(commonConfig, false, 2))  
      await build(commonConfig)  
    }  
      
    

😄 全是注册的 ` plugin ` ，封装的太好了

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicF7u2P0XpcPLZ4ZicmOFXICwdpDwWjRZLHxkj8qp5OZjiaKEj7GT8oO0JzaBDvU7qAkdsicpUCXeWSOA/640?wx_fmt=jpeg&from=appmsg)

接着找，发现最终的配置在 ` taro:vite-mini-config ` 中，拉出来看看

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicF7u2P0XpcPLZ4ZicmOFXICwRYh3172dFmRf1Eq8xPTAIboDjs88BroryfWdSRuGMufx6qjenV9fkA/640?wx_fmt=png&from=appmsg)

苦笑不得，破案了，原来是开启了 lib 构建 还真就是大号的 ` rollup `

相当于是把代码做了全量的编译，又通过各种定制 ` plugin ` 将代码转化成小程序代码

如果期待 ` vite ` 那些精髓特性的话，对不起， ` taro4 ` 暂时还做不到...

详细的代码转化逻辑推荐看这个几个文件，主要逻辑都在这里面了

  * taro/packages/taro-vite-runner/src/utils/compiler/base.ts 
  * taro/packages/taro-vite-runner/src/utils/compiler/mini.ts 
  * taro/packages/taro-vite-runner/src/mini/entry.ts 
  * taro/packages/taro-vite-runner/src/mini/page.ts 

###  路由按需 ⭐️

` taro4 vite ` 最期待的功能是路由切割按需编译，没有访问到（加载到的）路由以及路由对应依赖的资源不打包，当页面被访问时在进行构建，这也是 `
vite ` 的一个核心卖点。我们之前也聊过，整个 ` taro ` 的构建链路里面的最狠的 **刺客**
其实在小程序IDE那一步构建，往往小程序IDE的构建速度是 ` taro ` 构建速度的几倍， ` taro `
的构建实际上已经优化的很不错了并不慢，具体看 --->这篇文章。

如果能做到按需编译实际上就能从根上解决IDE构建慢的问题（感觉小程序IDE来接管路由按需构建是不是更合适！）

之前我们尝试过 react-loadable 的方案来实现运行时路由懒加载，尝试劫持小程序page代码，当路由没有被访问时 ` return null `
，当路由被激活（访问）时，在 ` return <VIew>....</View> ` ，然后触发 ` webpack rebuild `
；但是最终发现要改动的代码非常多，难点也很多，所以转头去拦截 ` app.config.js ` 来实现路由按需编译，这样简单很多。

希望后续 taro 大大们可以考虑考虑搞一个真正的路由按需编译~

###  我的观点

还是老老实实用 ` taro webpack ` 吧，当前的版本仅是通过 ` vite lib ` 模式进行全量构建， ` vite `
的核心特性并不能支持，构建速度上也并没有比 ` webpack ` 快多少。

源码推荐看一遍，还是能学到很多的。 ` taro ` 的分包做的是真合理啊，增加一个构建链路，居然只增加一个包就好了，没做多少大改，厉害了。

同时也期待 ` taro vite ` 后续的新升级，希望能把 ` vite ` 的精髓整合进 ` taro ` 。

  

预览时标签不可点

微信扫一扫  
关注该公众号





****



****



×  分析

  收藏

