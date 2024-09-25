![cover_image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicGQDIquJvfSiaGOe9T5rHyA3eYicdlRYPKTPKlYc1K1Dpn7OjicrWdpSRGgw9WBhL6P5jl2LscCjUDAw/0?wx_fmt=jpeg)

#  转 C 端后，我在古茗做的第一个需求

原创  何遇  [ Goodme前端团队 ](javascript:void\(0\);)

__ _ _ _ _

> 作者：何遇

做了 6 年的 toB SaaS 系统，如今转 C 端，我将用这篇文章分享我在古茗开发第一个需求的心路路程。

##  需求描述

用微信、支付宝和抖音扫描门店展示的同一个二维码进入不同的页面，具体来说，支付宝和抖音端进入点单小程序的点单页，并定位到二维码所绑定的门店，微信端则进入二码合一页面。二码合一名称的由来是，将原来的点餐码和社群码统一成一个二维码，微信扫描之后进入二码合一页面，在该页面判断用户是否加过社群，根据结果在界面上展示不同的内容。

> 注意：在后续的内容中将二码合一页面简称为二码合一。

二码合一有一个基本的功能，即：长按社群码，微信自动打开它的加群页面。经调研，小程序无法做到这一点，于是我们决定用微信公众号网页开发二码合一，该页面需要包含如下功能：

  1. 长按社群码，微信自动打开它的加群页面让用户加入古茗的社群。 
  2. 判断用户是否已加入古茗社群 
  3. 自动跳转到小程序点单页 
  4. 点击按钮跳转到小程序点单页 

##  加社区

社群码是一个二维码图片，将其显示在页面上，长按图片，微信能自动识别其中的内容并打开对应的加群页面。

##  用户是否已加入社群

前面的内容曾提到，二码合一是微信公众号网页，而扫二维码的用户最终要跳到微信小程序去下单，如何将进入二码合一的用户与进入点单小程序的用户识别为同一用户？这要借助微信公众平台的
unionid。

  1. openid：每个用户针对每个公众号产生唯一的标识。无需用户授权，也无需用户关注公众号便能获得 openid。 
  2. unionid：一个用户虽然对多个公众号和应用有多个不同的 openid，如果将这些公众号和应用绑定到同一个开发平台账号下，那么他对所有这些同一开放平台账号下的公众号和应用，只有一个unionid。用户关注了公众号或通过用户授权才能得到 unionid。 

在这一步了解到，拿到用户的 unionid 便能调用后续的接口得知用户是否加入了古茗的社群。继续调研如何获取 unionid。

##  网页授权

###  微信公众号网页授权

关于网页授权，我们首先想到的是微信公众号网页授权。步骤大致如下：

  1. 用户同意授权，获取code。 
  2. 通过 code 换取网页授权 access_token。在这一步能得到 unionid。 

以往开放 SaaS
系统的时候，域名的数量完全受自己控制，开发微信生态的产品则不然，业务上需要使用哪些域名必须预先到微信公众平台配置。域名受控这一点是最先需要转变的认知。下面是获取
code 的时序图。
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicGQDIquJvfSiaGOe9T5rHyA3e1eu1mkiay7MiaA15OEyJeRnQtKU7JRCpucBtl7LEPzibpA1NaZylNyrg/640?wx_fmt=jpeg&from=appmsg)
参数解释：

  1. appid：微信公众号的 appid 
  2. scope：应用授权作用域，取值为 snsapi_base 和 snsapi_userinfo。snsapi_base：不弹出授权页面，直接跳转，只能获取用户 openid；snsapi_userinfo：弹出授权页面，可拿到 unionid，即使在未关注公众号的情况下，只要用户授权也能获取其信息 。 
  3. redirect_uri1：二码合一的地址，以便于授权之后能跳回二码合一。 
  4. redirect_uri2：发起授权的网页的地址，已便于授权后能跳回发起授权的网页。 

发起网页授权的链接如下：

    
    
    https://open.weixin.qq.com/connect/oauth2/authorize?appid=wx807d86fb6b3d4fd2&redirect_uri=http%3A%2F%2Fdevelopers.weixin.qq.com&response_type=code&scope=snsapi_userinfo&state=STATE#wechat_redirect  
    

在网页授权这部分，我最初的疑问是，为什么不能在二码合一发起授权，而需要在授权中间页发起授权。答案是：网页授权需要在微信公众号后台配置网页授权域名，且只能配置一个。二码合一分为开发环境、测试环境和生产环境等，至少有
3 个域名，这不符合公众号后台的要求。

在网页授权这部分我们遇到了一个问题，即，要想得到 unionid，授权作用域必须是
snsapi_userinfo，这会弹出授权页面。为了更好的用户体验，我们的交互链路不允许弹出授权页面。

###  企业微信网页授权

微信公众号网页授权不满足要求，我们便开始调研企业微信网页授权。从前端视角来看，企业微信网页授权的时序图与微信公众号网页授权的相同，至少授权时携带的参数不同。授权的链接如下：

    
    
    https://open.weixin.qq.com/connect/oauth2/authorize?appid=CORPID&redirect_uri=REDIRECT_URI&response_type=code&scope=snsapi_base&state=STATE&agentid=AGENTID#wechat_redirect  
    

在这里我们重点要关注的是授权范围 scope 的值，它有两种取值：snsapi_base 和
snsapi_privateinfo。snsapi_base：静默授权，可获取成员的基础信息（UserId）；snsapi_privateinfo：手动授权，可获取成员的详细信息，包含头像、二维码等敏感信息。阅读企业微信的文档我们发现，当
scope 为 snsapi_base 时，我们通过此时返回的 code 去获取用户信息，将得到如下两种结果： **当用户为企业成员时**

参数  |  说明   
---|---  
errcode  |  返回码   
errmsg  |  对返回码的文本描述内容   
userid  |  成员UserID。若需要获得用户详情信息，可调用通讯录接口： **读取成员。 如果是互联企业/企业互联/上下游，则返回的UserId格式如：  CorpId/userid。  **  
user_ticket  |  成员票据，最大为512字节，有效期为1800s。   
scope为snsapi_privateinfo，且用户在应用可见范围之内时返回此参数。  
后续利用该参数可以获取用户信息或敏感信息，参见  **"获取访问用户敏感信息"。 暂时不支持上下游或/企业互联场景。  **  
  
**非企业成员时**

参数  |  说明   
---|---  
errcode  |  返回码   
errmsg  |  对返回码的文本描述内容   
openid  |  非企业成员的标识，对当前企业唯一。不超过64字节   
external_userid  |  外部联系人id，当且仅当用户是企业的客户，且跟进人在应用的可见范围内时返回。如果是第三方应用调用，针对同一个客户，同一个服务商不同应用获取到的id相同   
  
本需求扫码二维码的一定是非企业成员，因此我们能得到
external_userid，通过它判断用户是否加入了社群。到目前为止，我们找到了判断用户是否加入了社群的方案，即：企业微信网页授权获取 code，再获取
external_userid。

##  跳转小程序

微信客户端网页跳转小程序最常见的方式是开放标签：wx-open-launch-
weapp，我们的需求之一是，当用户已经加入社群，则自动跳转到点单小程序，在这里使用 URL Scheme 完成该功能。URL Scheme
虽好，但它有一个限制——每个小程序每天 URL Scheme 和 URL Link 总打开次数上限为 300
万，因此我们使用开放标签作为兜底选项。在本地环境无法调试开放标签，必须将代码发布到测试环境，这与 JS 接口安全域名最多设置 3 个有关。这块我还踩了一个坑
—— 在 wx.config({...}) 调用正确的情况下，开放标签不显示，而其他 API 能成功调用。原因是，index.html 引入了 1.3.0
版本的 JS-SDK，在 es module 中引入 1.6.0 将不生效，而开发标签要求的最低版本是 1.6.0。

##  单位与 CSS 尺寸

本需求有一张全屏背景图，图上有按钮，我们需要将热区定位到按钮区域，点击按钮实现跳转。怎么定位呢？我的想法是，假设手机屏幕的尺寸为 430 * 932
px，原图的尺寸为 750 * 1548 px，以图片左上角为原点，按钮的坐标为 (600,1000)，宽度为 100 px，高度为 80
px。我的疑问如下：原图的宽高比和手机尺寸的宽高比不同，在铺平背景的时候以宽度铺满还是以高度铺满，如果不适应高宽比，只是粗暴的将屏幕铺满，图片将变形，这一定不是需求方想要的效果。后来与设计师确认铺满宽度，高度自适应。那么图片的显示宽度为
430，按钮的显示宽度为 430 / 750 * 100 === 57.3，高度为 430 / 750 * 80 === 45.8。按钮的坐标为 (430
/ 750 * 600, 430 / 750 * 1000) ===(344,573.3)。得到上述结果后，可以使用 CSS
绝对定位将热区定到相应的位置，由于手机有多种分辨率，因此上述数据必须根据实际情况动态计算。和同事交流思路之后，他说不用自己计算，可以直接使用
Taro.pxTransform 来做运行时的尺寸转换。下面我们对 Taro.pxTransform 一探究竟。

###  Taro.pxTransform

Taro.pxTransform 用于在 JS 中转换行内样式，其类型定义为 pxTransform(size: number): string，参数
size 是设计稿中量出的尺寸大小，pxTransform 的源代码如下：

    
    
    function (size) {  
        const config = taro.config || {}  
        // H5 字体尺寸大小基准值  
        const baseFontSize = config.baseFontSize  
        const deviceRatio = config.deviceRatio || defaultDesignRatio  
        // 计算 designWidth 的大小，默认值是 750  
        const designWidth = ((input = 0) => isFunction(config.designWidth)  
          ? config.designWidth(input)  
          : config.designWidth || defaultDesignWidth)(size)  
        if (!(designWidth in deviceRatio)) {  
          throw new Error(`deviceRatio 配置中不存在 ${designWidth} 的设置！`)  
        }  
        // 转换后的单位，H5 的目标单位是 rem  
        const targetUnit = config.targetUnit || defaultTargetUnit  
        // 保留多少位小数，默认值为 5  
        const unitPrecision = config.unitPrecision || defaultUnitPrecision  
        const formatSize = ~~size  
        // 在上述例子中，designWidth 的值为 750，因此 rootValue 的值为 1  
        let rootValue = 1 / deviceRatio[designWidth]  
        switch (targetUnit) {  
          case 'rem':  
            // rootValue 为 2 倍 H5 字体大小基准值  
            rootValue *= baseFontSize * 2  
            break  
          case 'px':  
            rootValue *= 2  
            break  
        }  
          
        let val = formatSize / rootValue  
        if (unitPrecision >= 0 && unitPrecision <= 100) {  
          val = Number(val.toFixed(unitPrecision))  
        }  
        return val + targetUnit  
      }  
    

在上述例子中，pxTransform 计算的结果为 size / (2 * baseFontSize)。baseFontSize 表示 H5
字体大小基准值，可在 config/index.js 中配置，默认值为 20。H5 使用的单位是 rem，相对于根元素(即 html 元素) font-
size 计算值的倍数。假设 size 的值为 430 ，那么 Taro.pxTransform 的返回值为 10.75 rem (430 / (2 *
20) === 10.75)

###  根元素的字体大小

下面是 Taro 计算根元素字体大小的代码

    
    
    // H5 字体大小基准值  
    const baseFontSize = options?.baseFontSize || 20;  
    const designWidth = options?.designWidth || 750  
    // config.deviceRatio = {  
    //   640: 2.34 / 2,  
    //   750: 1,  
    //   828: 1.81 / 2  
    // }  
      
    // rootValue 的默认值是 20 / 1 * 2 === 40  
    const rootValue = baseFontSize / config.deviceRatio![designWidth!] * 2  
      
    function f(){  
        var e = window.document.documentElement;  
        var width = Math.floor(e.getBoundingClientRect().width);  
          
        if(width < 600){  
          var x = rootValue * width / designWidth;  
          e.style.fontSize= x + "px"  
        }else if(width < 840){  
          width = designWidth / 2;  
          var x = rootValue * width / designWidth;  
          e.style.fontSize= x + "px"  
        }else if(width < 1440){  
          width = designWidth / 2;  
          var x= rootValue * width / designWidth;  
          e.style.fontSize = x + "px"  
        }else{  
          width = designWidth / 2;  
          var x = rootValue * width / designWidth;  
          e.style.fontSize= x + "px"  
        }  
    }  
      
    window.addEventListener("resize",f);  
      
    f()  
    

根元素字体大小可以用公式表示为：rootValue * width / designWidth，其中 rootValue 通过 baseFontSize
计算而来，后续将进一步介绍它，width 是屏幕宽度，designWidth 是设计稿的宽度。从上述代码可以看出，在一个水平宽度为 430px
的设备上，根元素字体大小为 22.9333px (40 * 430 / 750)。现在将本小节开头例举的按钮坐标 (600,1000)，宽度 100，高度
80 分别带入 Taro.pxTransform，将各自的结果乘以根元素字体大小(在这里是 22.9333)，可以发现与我在前面手动计算的值相当。

###  公式推导

现在疑问是，为什么用 Taro.pxTransform(size) * rootFontSize
得到的结果与我们手动计算的结果一样呢？现在来推导公式。自己计算按钮位置和尺寸使用的公式如下：

    
    
    const designWidth = 750; // 背景图的宽度  
    const clientWidth = 430; // 屏幕可视区域的宽度，也是背景的显示宽度  
    const sw = 100; // 按钮在原图上的宽度  
    const sh = 80;  // 按钮在原图上的高度  
    const sx = 600; // 按钮在原图上的 x 坐标  
    const sy = 1000; // 按钮在原图上的 y 坐标  
      
    const ratio = clientWidth / designWidth;  
    const dw = sw * ratio;  
    const dh = sh * ratio;  
    const dx = sx * ratio;  
    const dy = sy * ratio;  
    

从上述代码可以看出，目标值 = 原始值 * ratio；使用 Taro.pxTransform 获取转换后的尺寸，再结合根元素的字体大小，转化成公式如下：

    
    
    const dSize = Taro.pxTransform(size) * rootFontSize  
    === Taro.pxTransform(size) * (rootValue * clientWidth / designWidth)  
    === size / (2 * baseFontSize) * (rootValue * clientWidth / designWidth)  
    === size / (2 * baseFontSize) * (baseFontSize * 2 * clientWidth / designWidth)  
    === size * (clientWidth / designWidth)  
    === size * ratio  
    

从上述换算可知，目标值依然 = 原始值 * ratio。clientWidth 是用户设备的可视区域宽度，其值可变，designWidth
是背景图的宽度，也是设计稿的宽度。在开发 C 端应用时，我们首先需要与平面设计师确定设计稿的尺寸宽度，使其与 Taro 的换算标准一致，Taro
默认的换算标准为 750，其值可配。

###  字体大小基准

从前面的内容可知，Taro 在做尺寸换算时，baseFontSize 起到了重要的作用，它的默认值是 20，其值可配，该由谁确定 baseFontSize
的大小呢？答案是平面设计师，除了字体大小基准，他还需要确定设计稿标准尺寸宽度。

##  总结

开发本需求期间，最让我头痛的是调试 JS-SDK 开放标签，每一次修改都需要发布到测试环境后才能看到结果，这是因为本地服务的域名未被设置成 JS
安全域名。另一个让我疑惑的点是，为什么 Taro.pxTransform(size)
计算的热区显示在界面上与我手动计算的位置相同，其原因我通过介绍源码加推导公式的方式已加以说明。最后，Taro
计算根元素字体大小的过程比较复杂，不像网上的大部分文章那么直接确认一个值，Taro 通过 baseFontSize 再结合屏幕宽度动态计算出它的值。

  

预览时标签不可点

微信扫一扫  
关注该公众号





****



****



×  分析

  收藏

