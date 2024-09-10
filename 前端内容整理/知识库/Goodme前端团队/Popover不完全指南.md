![cover_image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicGuPwtriaKMd86RYUia6qOZ29hhzibfbicGPTse189LycOX7xe0kuiagGR8ibvUCIIRaXR5ib96SQru11h4w/0?wx_fmt=jpeg)

#  Popover 不完全指南

原创  杨华云  [ Goodme前端团队 ](javascript:void\(0\);)

__ _ _ _ _

##  前言

说到 popover
，大家可能还有点陌生，但是当我们说到弹窗时，大家也就再熟悉不过了，它只是popover的一种表现形态。除了常见的弹窗，我们常用的组件比如 antd 中的
popover 组件以及移动端常用的 toast 组件也算是 popover
的一种实践。这些组件虽然功能比较简单，但是细节可一点也不少，就拿弹窗而言，zIndex层级冲突就是一个老大难的问题，相信大家没少用类似z-
index:9999的黑魔法。  
有了 Popover API，我们便可更优雅的实现类似的交互，其原生拥有以下特性：

  * **提升到顶层。** 弹出式窗口将出现在页面其余部分的 Top Layer，因此您无需尝试 z-index。 
  * **轻度关闭功能** 。点击弹出式窗口区域以外的位置会关闭弹出式窗口并返回焦点。 
  * **默认的焦点管理** 。默认情况下只能同时打开一个popover，打开一个popover时，会自动关闭另外一个。 
  * **无障碍键盘绑定。** 点按 esc 键或进行双重切换会关闭弹出式窗口并返回焦点。 
  * **可访问的组件绑定** 。从语义上将弹出式窗口元素连接到弹出式窗口触发器。 

##  快速上手

作为 web 新添加的基础能力，甚至开发者无需 Javascript 仅仅使用 HTML 即可创建 popover。创建 popover
非常简单，默认情况下，只需要一个用于触发弹出式窗口的 button 以及一个要触发的元素即可。

  * 首先，在要作为弹出式窗口的元素上设置 popover 属性。 
  * 然后，在弹出式窗口元素上添加一个唯一的 id。 
  * 最后，将按钮的 **popovertarget** 设置为 popover 的 id，即可将按钮和 popover 关联起来 

    
    
    <button popovertarget="my-popover">Open Popover</button>  
      
    <div id="my-popover" popover>  
      <p><p>I am a popover with more information. Hit <kbd>esc</kbd> or click away to close me.<p></p>  
    </div>  
      
    

对于按钮，我们称之为触发器，其除了通过 popovertarget 属性声明要控制的 popover 元素外，还可以通过
popovertargetaction 控制触发器的行为，其枚举值如下：

  * **popovertargetaction** ：hide || show || toggle 

要想让一个元素成为popover元素，只需添加 popover属性即可

  * **popover** ：auto || manual 

其值默认是 auto，auto 和 manual 最大的区别在于，auto 模式时，点击元素以外的区域的会关闭
popover，manual则不会，同时比较诡异的是，当你给 popover 设置了 overlay 以后，点击 overlay 依旧触发事件穿透。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGuPwtriaKMd86RYUia6qOZ29ZhsT97Eh73g3FqCbrLb0Q8Osc2fiaUtA1PbLXPgjLic3jtRdyDaWzPZw/640?wx_fmt=png&from=appmsg)
image.png

  
从开发者工具中也可以看到 popover 是以 top layer 的形式存在的，因此无法用 z-index 来调整 popover
与其他元素的层级，通过此特性，或许开发者以后再也无需使用 React Portal 了。

  

##  命令式控制

虽然可以不用 Javascript 也能实现 popover，但是 popover 元素上还是有方法可以控制 popover
的打开与关闭，同时也提供了beforetoggle 及 toggle事件，用于获取当前popover的状态。命令式打开弹窗的开发者表示狂喜😁。  

**实例方法**

  * showPopover 
  * hidePopover 
  * togglePopover 

**事件**

  * beforetoggle 
  * toggle 

    
    
    <div class="action">  
      <button data-acion="hide">关闭modal</button>  
      
      <button data-acion="show">显示modal</button>  
      
      <button data-acion="toggle">切换modal</button>  
    </div>  
      
    <div class="modal" id="my-popover" popover>  
      <button popovertargetaction="hide" popovertarget="my-popover">关闭</button>  
      <div>这里是一个正经弹窗</div>  
    </div>  
      
    
    
    
    const modal = document.querySelector('.modal');  
    const actions = document.querySelector('.action');  
      
    actions.addEventListener('click', (e) => {  
      switch (e.target.dataset.acion) {  
        case 'hide':  
          modal.hidePopover()  
          break;  
        case 'show':  
          modal.showPopover()  
          break;  
        case 'toggle':  
          modal.togglePopover()  
          break;  
        default:  
          break;  
      }  
    })  
      
    modal.addEventListener('beforetoggle', function (e) {  
      console.log("beforetoggle", e)  
    })  
      
    modal.addEventListener('toggle', function (e) {  
      console.log("toggle", e)  
    })  
    

  

##  嵌套的 popover

上文说到，默认情况下，多个popover不能同时并存，打开一个popover的时候会把其余的popover给关掉。那有没有办法同时打开多个popover呢，常见的
case 就是弹窗套弹窗，亦或者是级联菜单。答案是肯定的，毕竟弹窗嵌套也是经常用到的交互之一，虽然不怎么优雅。popover 也提供了3种方式实现
popver 嵌套。

  1. 直接通过 DOM 嵌套实现 

    
    
    <div popover>  
      Parent  
      <div popover>Child</div>  
    </div>  
    

  2. 将控制器放在 popover 元素中 

    
    
    <div popover>  
      Parent  
      <button popovertarget="foo">Click me</button>  
    </div>  
      
    <div popover id="foo">Child</div>  
    

  

##  让 popover 动起来

没有动画的弹窗是没有灵魂的，我们可以给弹窗加上一点动画给应用增加一些灵动感。  
在以往的开发经验中，我们可能会通过 transisition 或者 animation 给弹窗增加对应的动画，但是这有个前提，弹窗元素的 display
属性的值必须是 block。也就是说元素必须实实在在渲染在页面中，动画才能生效。  
对于无中生有的元素我们也可以在元素插入DOM
后，通过js设置一个延迟增加进场动画,同时退场动画也是类似的，关闭的时候先增加一个退场动画，然后动画结束的时候，再将元素销毁。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGuPwtriaKMd86RYUia6qOZ29nIM1pagEcd49GEBwzB6FzzODA7cnKffTuAJK9Ag4Sfv0TRUogJRq1w/640?wx_fmt=png&from=appmsg)
image.png

对于popover来讲，未显示之前 display 为 none，在显示的时候，其值才为
block，而且这个值并不受开发者控制，我们不能像之前那般通过设置延时来实现对应的动画效果。因此想要在已有的动画能力基础上给 popover
增加进出场动画也不是一个简单的事。幸而 css 提供了 **discrete property transitions （离散属性过渡）**
来解决类似的问题。  
下面来看下如何给 popover 增加进出场动画

    
    
    <button popovertarget="my-popover"> 打开弹窗 </button>  
      
    <div id="my-popover" popover>  
      模态弹窗会打断用户的正常操作，要求用户必须对其进行回应，否则不能继续其它操作；  
    </div>  
      
    
    
    
    button {  
      font-size: 100%;  
      padding: 0.75rem;  
      background: white;  
      transition-duration: 0.5s;  
      border: 4px solid plum;  
      background: lavenderblush;  
      border-radius: 1rem;  
      
      &:hover,  
      &:focus {  
        background: plum;  
        color: white;  
      }  
    }  
      
    [popover] {  
      background: black;  
      color: white;  
      font-weight: 400;  
      padding: 1rem 1.5rem;  
      border-radius: 1rem;  
      max-width: 20ch;  
      line-height: 1.4;  
      top: 2rem;  
      margin: 0 auto;  
    }  
      
    body {  
      background: #fcf9fb;  
      display: grid;  
      font-size: 1.5rem;  
      font-family: system-ui, sans-serif;  
      place-items: center;  
      height: 100dvh;  
    }   
      
    /*   弹窗展示时的样式  */  
    [popover]:popover-open {  
      translate: 0 0;  
    }  
      
    /*  弹窗关闭时的样式  */  
    [popover] {  
      transition: translate 0.7s ease-out, display 0.7s ease-out allow-discrete;  
      translate: 0 -22rem;  
    }  
      
    /*   弹窗初始的样式  */  
    @starting-style {  
      [popover]:popover-open {  
        translate: 0 -22rem;  
      }  
    }  
    

在上面的代码中，通过 @starting-style 声明了 popover的初始位置，然后通过45行增加 **allow-discrete**
属性让display none 到 block 也可以实现动画效果。感兴趣的可以试着删除这个属性再对比下。

效果如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/TpB2QHJbiaicGuPwtriaKMd86RYUia6qOZ290g6eVRicLkdCgyzHg6MnXR9BQiaRqVW5fDEGox5d64alWO6jKvQbLcWA/640?wx_fmt=gif&from=appmsg)

##  兼容性

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGuPwtriaKMd86RYUia6qOZ29oONiaTDGy3D6x0BjsJfzONJDnjic4qhS3Jtv6vy0ibOKXsPSPdOWgU6qA/640?wx_fmt=png&from=appmsg)

毫无疑问，对于新出的功能，其兼容性都不用说了，只有最新的浏览器才支持，再者因为特殊的环境，估计猴年马月才能在业务上用上，不过这并不妨碍我们去了解 web
的最新发展。了解最新的技术趋势不仅可以帮助我们在未来更快地采用这些新功能，还能提升我们的开发技能和技术视野。

不过庆幸的这一次，居然三大浏览器都同时支持了。当然对于喜欢尝鲜的同学可以试着用 popover API
构建自己的组件。不过使用之前还是推荐先用以下脚本做下功能检测。

    
    
    function supportsPopover() {  
      return HTMLElement.prototype.hasOwnProperty("popover");  
    }  
    

##  小茗推荐

  * [ 写一个VS Code 插件：Color to See 文章 ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247485729&idx=1&sn=2d9fdd59ee023597d06c6d172a3b8dea&chksm=cfe58e26f89207306f9440f922a8d77a30be5240d82134bedbbd7557e9156169bd1470d4cea7&scene=21#wechat_redirect)
  * [ 探究前端包管理工具：npm、yarn 和pnpm ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247485637&idx=1&sn=ba3ee4db0190d6458b0fcc05a600759c&chksm=cfe58fc2f89206d48344d136d82afef51ff149af722bf7e6f49f6755461bc957961ca974bdde&scene=21#wechat_redirect)   

  * [ 古茗 Mars 预编译技术方案探索 ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247485675&idx=1&sn=0d53b5441b29bbced27c0cf0544dfaab&chksm=cfe58fecf89206fa1ea0ba6af43f7bcb78c6119ae92d6105b1664f8bf6597612f5abd3f2ca29&scene=21#wechat_redirect)

##  最后

关注公众号「Goodme前端团队」，获取更多干货实践，欢迎交流分享。

  

预览时标签不可点

微信扫一扫  
关注该公众号





****



****



×  分析

  收藏

