![cover_image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicFlyvylwBsapvcVkg0ey8y6SFyul8Oh9zfGKZQz08vdialPhgRJRYCcwcgMQ0wibSAty2Bs8GQUEWcg/0?wx_fmt=jpeg)

#  古茗如何做前端数据中心 - SDK 设计篇

原创  陆晨杰  [ Goodme前端团队 ](javascript:void\(0\);)

__ _ _ _ _

##  前言

在上一次中，我们谈到了古茗前端数据中心的整体的架构设计，今天我们来具体看一下 sdk 侧的具体设计。

> 我们先来回归一下上次的架构设计图，你还记得吗？不记得就再来回顾一下上次的内容吧！
> ![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicFlyvylwBsapvcVkg0ey8y657cRtlBOf4FMGko5Vn64QoF5pXduHSkv8J48pMMdT3vriazJRQI3yHA/640?wx_fmt=png&from=appmsg)

##  总体设计

###  概要设计图

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicFlyvylwBsapvcVkg0ey8y6g1cRlNRzvtn6z6p1OicvTdJkybZxBricFZibribd69m4JxC9wibEJQfFcBg/640?wx_fmt=png&from=appmsg)
架构图

###  使用

> don't talk, show you the code
    
    
    // 初始化  
    Track.init({  
      debug: false,  
      appId: 1,  
      initialExtra: () => {  
        return {  
       // 强行覆盖默认的 appId，用于微应用中识别子应用  
          appId: getCurrentApp()?.appId || 1,  
          userId,  
        };  
      },  
      integrations: {  
        InstrumentTrack: {  
          enable: false,  
          option: {},  
        },  
      },  
    });  
      
    // 埋点，两种埋点方式，对于无需参数的可以快捷埋点  
    function submitTrack(eventName: string, options?: { extends?: any }): void;  
    function submitTrack({  
     et: string;  
     e_name: string;  
     extends?: any;  
    }): void;  
      
    // 日志，消息内容会被 stringify  
    interface logger {  
     error(msg: any): void;  
     warn(msg: any): void;  
     info(msg: any): void;  
    }  
      
    

通常在日常使用，我们不会直接使用 core 包，为了方便开发的使用，我们已经基于平台二次封装好了 platform
包来对日志上报做了一些更客制化的需求，如：插件的集成、特有api的磨平等；尽可能的提升开发者的体验，做到开箱即食。

platform 暴露的函数只有 ` init ` ` submitTrack ` ` logger `
三个，分别用于初始化和埋点与日志的上报；具体的参数设计和实现我们在详细设计中再来讨论。

###  接口格式设计

在概要设计图中，我们知道整个 sdk 中，只存在一个 report 的接口的调用，report
接口是承载一切信息的基础，报文中包含了日志的所有的信息与云端配置的下发：

    
    
    // paylaod  
    {  
        "m": [  
            {  
                "time": 1710123186431,  
                "referer": "http://localhost:3000/",  
                "type": "log",  
                "data": {}  
            }  
        ],  
        "c": {  
            "appId": 382,  
            "env": "prod",  
            "app": "",  
            "app_version": "",  
            "platform": "",  
            "platform_version": "",  
            "model": "",  
            "brand": "",  
            "userId": "123",  
            “track”: {}  
        }  
    }  
    // response header  
    Track-Id: 111111aaaaaa222222bbbbbb  
    X-Track: a=1;b=25;c=31;d=0  
    

从上放方的 demo 数据可以看出， 接口核心内容为请求的 payload 和返回的 headers，payload 中包含了 ` m ` 和 ` c `
字段。

` m ` 是这次上报的所有日志信息，每一次的上报包含了多条日志，通过将多条日志合并至一次请求中发出以减少请求量
作为数据中心，平台不仅承载了埋点信息还包括了业务日志、资源信息、接口信息等不同类别的数据，所有的数据本质上都是一条日志，只通过 type
来区分了不同的业务属性，同时在 data 中添加特有的数据。上方 demo 中的 ` submitTrack ` 和 ` logger ` 本质上也只是
type 的不同。

` c ` 为本次上报的通用信息，appId 用来识别当前的应用，id 从公司的 devops 中获取，通过保持相同的
appId，来为以后多系统的数据打通做铺垫；同时sdk中的 ` initialExtra ` 方法，也可以向该字段中添加额外的数据：

> 为何上报的数据中有time字段？ 我们知道端侧的时间是不可信任的，那为何我们上报时还需要添加 time 字段呢？
> 那是因为在真正使用的场景时，可能会因为端侧堆栈未满、削峰、限流等原因导致的延迟上报，此时使用服务器的时间就会与真正日志产生的时间存在偏差，因此我们默认信任端侧的时间，使用端侧的时间对数据进行补偿或丢弃

####  模块

  * **CrossPlatform** : 提供需要跨平台的方法的实现，包含了 storage 存储、获取路由队列等方法 
  * **Reporter & Queue ** ：控制日志上报，管理了日志队列、削峰、并发管理等功能 
  * **ExtraInfo** ：额外信息，合并了 sdk 内置、插件内置、用户内置的通用信息 
  * **Configurator** ：远程配置相关模块，获取云端的相关 sdk 配置 
  * **Breadcrumb** ：用户行为日志记录，管理用户行为队列 
  * **Event** ：全局事件通信 
  * **Integrations** ：插件系统，控制插件的注册、使用等 

####  插件

  * **api** ：接口规范监控 & 接口异常监控 
  * **behavior** ：用户行为日志监控 
  * **cache** ：日志缓存模块，用于解决日志丢失问题 
  * **static** ：静态资源监控 
  * **track** ：埋点模块 

##  详细设计

###  模块拆封

####  Init

我们前文聊过，暴露出来的init方法其实就是将SDK类实例化过程的一个封装，我们先来看一下 Sdk 类和 init 方法的伪代码：

    
    
    // core  
    class SDK {  
     static get instance(): SDK;  
     // 实例化方法，sdk实例会存在 global 中，避免多次初始化导致重复埋点  
     static init(option) {  
         try {  
           _global.__sdk_instance_ = new SDK(option);  
         } catch (error) {  
           console.error('sdk init error: ', error);  
         }  
     }  
     // 远程配置管理    
     readonly configurator: Configurator;    
     // SDK 上报器    
     readonly reporter: Reporter;  
     // 各种内置模块  
     ...  
     // 插件列表    
     private integrations: Record<string, Integration> = {};  
     constructor(  
      this.configurator = new Configurator(this);  
      this.extraInfo = new ExtraInfo(this);  
      // 初始化各种模块 & 加载插件  
      ...  
      const totalSwitch = this.initIntegration();  
         /** 全局开关关闭时或有插件关闭时，强制刷新配置 */  
         const { report } = configurator.configuration;  
         if (report === false || (typeof report === 'number' && report !== totalSwitch)) {  
           configurator.forceUpdate();  
         }  
     )  
     // 各种工具函数  
     ...  
    }  
      
    // 提供便捷的上报日志和埋点的方法  
    export function logger(): Logger;  
    export function submitTrack(): SubmitTrack;  
      
    // platform  
      
    const DEFAULT_INTEGRATIONS = [  
      InstrumentBehavior,  
      InstrumentApi,  
      InstrumentStatic,  
      InstrumentTrack,  
      IntegrationCacheData,  
    ];  
      
    export init = (options) => {  
     // 处理options，添加默认值，转换参数格式等  
        SDK.init({  
         ...options,  
         transport,  
         corssPlatform,  
         integrations(this) {  
          return convertOptionsToIntegrations.call(this, DEFAULT_INTEGRATIONS, options.integrations);  
         },  
         initialExtra: () => inheritData(env.getTags.bind(env), initialExtra)  
        });  
    }  
    

core 中的 SDK 类并没有面向开发者进行设计，考虑的便是如何满足通用性的需求。在实例化过程中，是对各个在 core
中实现的模块的初始化，需要注意的是，我们模块的初始化顺序是有要求的，相关的配置、通用数据需要在最前初始化以供后面的进行模块使用。在实例化的最后，我们会去加载插件并更新远端配置。

platform 中已经内置好了默认插件、平台特有的 api 等，开发在使用时无需再对这些进行配置，只需要关注与自己应用相关的配置即可。

####  Configurator

为了控制大促等场景突发大流量不至于将系统打崩，云端会下发队列长度、削峰开关、限流参数等配置至客户端，该模块就是用来获取并处理云端下发的配置。

模块的定义如下：

    
    
    class Configurator {  
     private configuration: Configuration = {};  
     private stringConfiguration = '';  
     private storage: Required<CrossPlatform>['storage'];  
     /** 强制刷新配置 */  
     forceUpdate(): void;  
     /** 格式化配置 */  
     parse(input?: string, disableCache = false): Configuration;  
     /** 获取具体配置 */  
     get(key: string | string[]): any | Record<string, any>;  
    }  
    

日志的上报和配置的拉取都通过 report 接口进行，在将日志上报之后，对于端侧控制的配置也会在 header 中下发，配置的格式类似： `
a=2;b=25;c=31; ` 。

在获取了 header 中的值之后需要通过 parse 方法将字符串转换为 json
从而方便后续流程中消费配置；同时考虑到若将所有的日志上报禁用后将无法获取最新的配置，在每次sdk初始化后都会调用 forceUpdate
方法，手动的刷新一遍配置。

> 你知道吗？
> 通常在web中，我们会使用sendBeacon来上报日志，从而达到最好的体验；但是sendBeacon只会将数据接入到队列中，然后告知加入队列的成功与非，并不会告知是否发送成功的，更拿不到返回头之类的信息哦；
> 而且sendBeacon在chrome59～81，浏览器不允许设置 content-type 请求头为 application/x-www-form-
> urlencoded、multipart/form-data 或者 text/plain 以外的值。一旦出现了这种情况，sendBeacon
> 就会抛出异常哦！（没错，部分软件的webview会报错）

####  Reporter & Queue

Reporter 和 Queue是sdk的核心模块，他们决定了日志是否上报、何时上报，他们的格式如下：

    
    
    // Report & Queue是sdk的核心模块  
    class Reporter {  
     private queue: Queue;  
     getQueue(): Queue;  
     send(data: Data, options: {lazy?: boolean; reportType?: number});  
     private disabledReport(reportType?: number): boolean;  
    }  
      
    class Queue {  
     /** 普通队列 */  
     private stack = [];  
     /** 历史紧急上报队列信息 */  
     private immediatelyStack = [];  
     /** 采样队列 */  
     private samplingList = [];  
      
     /** 是否正在上报 */  
     private isFlushing = false;  
     private readonly micro: Promise<any>;  
     /** 上报任务轮训 */  
     private timer: NodeJS.Timer | null = null;  
     /** 异常重试次数 */  
     private retryTimes = 0;  
     /** 添加队列，包含采样等逻辑 */  
     add(rawData: data, options: {lazy?: boolean}): void;  
     /** 上报数据（理论外部不允许使用） */  
     report(data: Item[], options?: {lazy?: boolean}): void;  
     /** 获取当前堆栈 */  
     getStack(): Item[];  
     /** 添加上报轮训任务 */  
     private addListener(): void;  
     /** 添加队列时预处理上报信息 */  
     private generateStackItem(data: data): Item;  
     /** 采样上报队列 */  
     private sampling(data: Item): Item | undefined;  
     /** 添加队列 */  
     private _add(data: Item, lazy = true): void;  
     /** 是否需要延迟上报 */  
     private isLazy(lazy = true): boolean;  
     /** 轮训任务 */  
     private loop(time = 10000): void;  
     /** 清空某一种日志类型的采样队列并获取采样 */  
     private flushSampleItem(type: string, force = false): void;  
     /** 清空所有采样队列并获取采样 */  
     private flushSample(): void;  
     /** 清空普通队列并上报 */  
     private flushStack(): void;  
    }  
    

我们可以看到，每个 reporter 都会有一个属于自己的 queue ； reporter
本身只会通过远程配置来判断是否需要发送对应类型的日志，并调用队列的 add 方法加入到队列中，具体的限流等逻辑都在队列中实现，队列出发上报的时机有以下几种：

  1. 到达队列的长度：在添加队列后，若队列的长度已到达预设，queue就会将现在普通队列中所有的日志信息取出并上报，队列的长度默认为25，同时队列长度会受云端的配置影响 
  2. 定时上报：在初始化队列后，会创建一个10s的定时器，每个一段时间会清空队列进行上报，避免用户长时间不进行操作后导致日志的时间与当前时间差过大，清洗时需要对时间间隔比较大的历史数据进行补偿 
  3. 削峰上报：在云端开启削峰之后，会默认根据userId作为特征值进行削峰处理；此时队列的长度将为无限大，并且将特征值转换为10进制后，除以60取余的值为他在这分钟能上报的秒数，上报时会以队列长队 * 10分批上报 

同时队列还包含了采样和抽样逻辑。

**采样**
：当云端开启采样时，会下发采样队列的长度，日志会先添加到采样的队列，当采样的队列长度到达预设或定时器触法时，才会从采样队列中随机取其中1条添加到普通队列中，需要注意的是虽然云端会根据对应的采样配置将数据等比例翻倍，但是还是会失真；端侧的采样尽可能的少用。

**抽样**
：当云端开启抽样是，会根据特征值（默认userId）来判断对应用户是否需要抽样，与采样相比，抽样仅会等比例的丢失用户的数据，但是符合特征值的的用户的数据是全的。

####  Integrations

插件模块本身并没有负责的逻辑，仅负责外部插件的加载，在 sdk 初始化时，会执行一下一段脚本，将插件初始化并向插件提供 sdk 实例：

    
    
    (_integrations ?? []).forEach((Integration) => {    
     try {    
     const integrationInstance =    
     typeof Integration === 'object' ? Integration : new Integration(this);    
         
     integrationInstance.instrument();    
     this.integrations[Integration.constructor.name] = integrationInstance;    
     totalSwitch += integrationInstance.reportType || 0;    
     } catch (error: any) {    
     console.error(error);    
     this.logger.error(error?.message, 'sdk_error');    
     }    
    })  
    

###  插件

端侧所有的额外功能都是通过插件集成的，处理内置的插件，在初始化时也可以通过 integrations
参数添加开发同学自己定制的插件，每个插件都需要有一个自己特有的 key 和 type，key 为一个可以描述插件的字符串，type
为二进制数字，用于云端墙纸关闭插件：

> 云端可以下发插件开发强制关闭插件，不会为每个插件都下发一个插件开发，插件开关仅是一个数字，这也是为什么插件的type必须为唯一的二进制数字。
> 如，现在有两个插件，插件A的type为1，插件B的type为2 当我下发的开关的值为0b0010时，插件A为关，插件B为开，1 &
> 0b0010转换为布尔值为false，而2 & 0b0010 为true，因此我们可以通过一个数字来管理所有插件的开关

####  Cache

缓存插件，用于限流时，在关闭前将当次未上报的数据缓存，在下次打开程序时将历史数据进行补偿上报：

> 考虑到storage有内容大小限制，小程序暂时未支持 cache 模块，后续会通过 文件缓存 来实现 cache 模块。
    
    
    class Cache {  
     /** 初始化插件，db等 */  
     instrument(): void;  
     /** 监听关闭事件，缓存堆栈 */  
     addUnloadEvent(db: DB): void;  
     /** 初始化时添加堆栈 */  
     reInsertPrevList(db: DB): void;  
    }  
    class DB {  
     static isSupport: boolean = !!(_global && _global.indexedDB);    
     private db?: DBDatabase;  
     connect(options: {name: string; version?: number; tableSchema: Record<string, string | undefined>;}): Promise<void>;   
     getAll<E = any>(tableName: string): Promise<E[]>  
     batchAdd(tableName: string, value: any, key?: string): Promise<void>  
     clear(table: string): Promise<void>;  
    }  
    

Cache 模块初始化时，会初始化 DB 来存取队列信息；在 beforeunload 时将队列添加至 indexDB 中； DB 模块是基于
indexDB 的分装，在 connet 时 open indexDB，并确定 version 和 tableSchema 更新本地的表结构。

####  Static

static 模块会对静态资源的 onload 和 onerror 事件进行劫持上报，web 的劫持比较好处理，实例化
PerformanceObserver 后，对 type 为 resource 的资源进行监听即可；需要注意的是，部分的浏览器对于传 type
会报错，需要用 entryTypes 兼容（是谁不用多说。。。）。

    
    
    const observer = new PerformanceObserver((list) => {  
     // 处理资源  
    })  
    try {  
     observer.observe({type: 'resource', buffer: true});  
    } catch () {  
     observer.observe({ entryTypes: ['resource'] });  
    }  
      
    

小程序的 image 等标签虽然也有这些事件，但是手动的一个个写不现实，因此我们额外支持了 babel-plugin-jsx-inject
的babel插件，该插件支持对指定元素添加对应的属性或父元素，plugin 接受的参数为

    
    
    interface PluginOptionElement {    
     /** 唯一key */    
     identity: string;    
     /** 需要修改的元素的名称 */    
     name: string | RegExp | ((pathName: string, path: NodePath) => boolean);    
     /** 需要修改的属性的名称，仅修改属性时生效 */    
     attribute?: string;    
     /** 需要添加的属性或父元素的ast的模版 */    
     template: string | TemplateFn;    
     /** 需要import的依赖 */    
     require?: [];    
    }  
      
    /** 获取继承了原有方法的函数 */    
    export const getInheritFn = (attribute: string, attributes: Record<string, Node> = {}): string => {    
     if (!attribute) {    
      return '';    
     }    
     const spreadAttributes = getSpreadAttribute(attributes);    
     const inheritFn = attributes[attribute];    
     if (!spreadAttributes?.length) {    
      return inheritFn ? `%%${attribute}%%(event);` : '';    
     }    
         
     const params = (    
      !inheritFn    
       ? spreadAttributes    
       : [  
        ...spreadAttributes,    
        {  
         name: `%%${attribute}%%(event)`,    
         index: (inheritFn as any)._attr_index,    
        },    
         ]    
      )    
     .sort((a, b) => a.index - b.index)    
     .map(({ name }) => name)    
     .join(',');    
         
     return `_execInheritFn(event, "${attribute}", ${params})`;    
    };  
      
    // demo  
    const getTriggerFn = (attributes: Record<string, Node>) => {    
     return `    
     (event) => {    
      _gem_t('img:error', event);    
      ${getInheritFn('onError', attributes)}    
     }    
     `;    
    };    
        
    export const imgErrorInjectConfig = {    
     identity: 'img-error',    
     name: 'Image',    
     attribute: 'onError',    
     template: (attributes) => getTriggerFn(attributes),    
     require: ['import { trigger as _gem_t, _execInheritFn } from "@guming/global-event-mini"'],    
    };  
      
    

需要注意的是，添加属性时，不能将原有的属性覆盖了，getInheritFn 方法就是用来解决这个问题的，getInheritFn
包装了一个函数，将原有相同名称的属性，作为所有参数传入方法中，在 _execInheritFn 中，一个一个 pop，取出第一个匹配 attribute
名字的属性来执行。

####  Track

复用了原有了 [platform]_track 包，在插件中进行了初始化，插件并没有对 track 的逻辑做额外的功能，所有与埋点相关的逻辑都在 track
包中，track 主要对埋点字段的格式进行了处理，同时提供了 TrackScrollView 和 TrackSwiper 进行自动的曝光和点击的埋点：

    
    
    class Track {  
     instrument() {  
      initTrack({    
       ...options!,    
       eventQueueLimitNum: 1,    
       request: ({ data }) => {    
        this.report({    
         type: MeasurementType.Track,    
        data,    
        });    
       },    
      });  
      sdk.extraInfo.addExtra('track', () => {    
       return trackStore.getCommonInfo();    
      });  
      this.trackClick();  
     }  
      
     /** 劫持点击，有 data-track-option 属性的元素点击事件自动埋点 */  
     private trackClick(): void;  
    }  
    

##  总结

数据平台端侧的 sdk
除了需要考虑满足数据上报和埋点的需求外还要考虑性能、稳定性和拓展性等方面的因素；满足了在未来流量不断增加的场景下的可靠性，为产品提供更好的数据支持。希望这篇文章对大家能有一点灵感和帮助。

##  最后

📚 小茗文章推荐：

  * 小程序用户登录：安全性与用户体验的平衡 
  * formily原来是这样解决这些表单难题 
  * 古茗是如何将小程序编译速度提升3倍的 

关注公众号「Goodme前端团队」，获取更多干货实践，欢迎交流分享~

  

预览时标签不可点

微信扫一扫  
关注该公众号





****



****



×  分析

  收藏

