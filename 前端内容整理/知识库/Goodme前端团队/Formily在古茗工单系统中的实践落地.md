![cover_image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicE7X7iaykicoQCFgOCSzBtBtiaejrok8QgJJjFriaPSRVNI7TjIHP8iaqc76IDiaK478qT3VJH68GIw6ldQ/0?wx_fmt=jpeg)

#  Formily 在古茗工单系统中的实践落地

原创  简可  [ Goodme前端团队 ](javascript:void\(0\);)

__ _ _ _ _

> 温馨Tips：阅读本文大约需要 7 分钟，可以细细阅读～

##  背景

古茗是茶饮连锁品牌，所以我们会有很多业务围绕着加盟商展开，比如供应链、报货、财务、机料、上传下达等等，在这不一一列举了，前面的可能比较好理解，那上传下达是什么呢？其实就是古茗总部和门店沟通、信息传达的一种方式，本文就围绕着上传下达这个业务展开给大家介绍下我们技术实现的演进和落地。

##  上传下达

上传下达，分为门店上传和总部下达：

  * 门店上传：简单理解就是门店给总部反馈消息、申请售后等等的渠道。 
  * 总部下达：简而言之是总部给门店下发的消息、任务、公告等等。 

门店上传的主要承载形式是工单，但上传的工单的来源还有其他，比如人工客服；总部下达是总部下发给门店的任务，其实交互形式也类似工单，我们在业务上没有给他划分到工单，但在前端来说技术方案其实是相通的。
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicE7X7iaykicoQCFgOCSzBtBtialu8sv17oBpjgXsLCQ5KYjWRJh0gM017Vhv4m1qWrXlzF1FJVe5iaibDg/640?wx_fmt=png)

##  工单系统的演进

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicE7X7iaykicoQCFgOCSzBtBtiarLfxiaU5gcAJUyQqibRRPb7PIS0uyTCmq3tO43W41jtd7rMUBIzrxTibQ/640?wx_fmt=png)

在最开始时期简单粗暴：来一工单撸一个页面；但业务在发展，面对越来越多工单这种方式已经满足不了诉求，而且开发效率低，维护困难。

接下来我们探索了表单模型描述+渲染的方案，表单模型在三方平台简道云进行维护，简道云提供了可视化搭建表单的能力，前端获取到表单模型描述后在我们的页面进行渲染，但这种方案也逐步暴露了缺陷：

  1. 简道云表单模型对于前端来说很冗余，比如说简道云表单模型会有几十个字段，但需要用户填写的可能只需要几个字段。 
  2. 表单校验麻烦。 
  3. 简道云表单模型描述不了联动，比如在选择A时C字段显示，在选择B时C字段隐藏。 
  4. 前端实现渲染不优雅，会有很多if else的逻辑。 

所以我们需要一个支持联动、高性能、校验能力强、优雅的表单解决方案，这时候Formily进入了我们的视野，经过调研发现Formily完全满足我们的诉求，所以我们目前的表单解决方案就是Formily，并由Formily为核心打造了古茗的工单开发生态。

##  Formily工单生态

####  Formily

Formily 是一个数据+协议驱动的表单解决方案，构建了从基础表单到低代码领域的高性能通用基础能力，JSON
Schema独立存在，给UI桥接层消费，保证了协议驱动在不同UI框架下的绝对一致性，不需要重复实现协议解析逻辑，同时其配套的跨框架+跨终端组件生态体系，也能让开发人员更高效的开发日常业务表单，尽可能的减少了重复冗余的逻辑实现。

#####  核心优势

  * 高性能 
  * 开箱即用 
  * 联动逻辑实现高效 
  * 跨端能力，逻辑可跨框架，跨终端复用 
  * 动态渲染能力 

#####  核心劣势

  * 学习成本较高，虽然 2.x 已经在大量收敛概念，但还是存在一定的学习成本。 

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicE7X7iaykicoQCFgOCSzBtBtiadwlgDibyzBcUVtEqegExWFn7BrhUq6Isvk9KdHr8Mlt7o3yicbSd4jdQ/640?wx_fmt=png)

####  技术栈选型

从上面Formily的分层架构图上可以看出，我们首先需要选择UI桥接层和拓展组件库，UI桥接层我们选择了React，拓展组件库在中后台选择了Antd，在B端小程序出于我们使用Taro的解决方案考虑最开始选择的是Taro
UI，但在使用中发现Taro
UI并不是特别易用，加之为了满足设计团队的诉求和统一B端设计交互，我们前端团队自研了组件库Gudesion，然后把Formily的拓展组件库迁移到了Gudesion。

下面是组件库文档截图（如果大家感兴趣，可以邀请Gudesion作者做后续分享）

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicE7X7iaykicoQCFgOCSzBtBtiaxxAeEhibh4eCriaan2gMiaSA4DShSAzWEvn6ib3mTEWEqgr5sFiaLtSicu1w/640?wx_fmt=png)

####  对接拓展组件库(Gudesign)

Formily文档：https://react.formilyjs.org/zh-CN/api/shared/connect

    
    
    import { connect, mapReadPretty, mapProps } from '@formily/react';  
    import { getPrefixCls } from '@guming/formily-shared';  
    import { Input as GudInput, InputProps, TextareaProps } from '@guming/gudesign';  
    import cls from 'classnames';  
    import React from 'react';  
    import { PreviewText } from '../preview-text';  
    import { getFinallyLayout } from '../shared';  
      
    import './index.less';  
      
    export type FGInputProps = Omit<InputProps, 'required' | 'label' | 'defaultValue' | 'value'>;  
    export type FGTextAreaProps = Omit<TextareaProps, 'required' | 'defaultValue' | 'value'>;  
      
    export const Input: React.FC<FGInputProps> & {  
      TextArea?: React.FC<FGTextAreaProps>;  
    } = connect(  
      GudInput,  
      mapProps((props, field) => {  
        return {  
          ...(getFinallyLayout(field.componentType, field.decoratorProps.layout) === 'column'  
            ? { align: 'left' }  
            : { align: 'right' }),  
          placeholder: '请输入',  
          ...props,  
          className: cls(`${getPrefixCls('comp-input')}`, props.className),  
          // 不需要以下属性  
          required: undefined,  
          label: undefined,  
          defaultValue: undefined,  
        } as InputProps;  
      }),  
      mapReadPretty(PreviewText.Input)  
    );  
      
    export default Input;  
    

####  表单配置平台

上面介绍了Formily是一个数据+协议驱动的表单解决方案，所以我们也搭建了一个配置JSON Schema的表单描述配置平台。JSON
Schema的配置方式可以查阅Formily官网文档：https://react.formilyjs.org/zh-CN/api/shared/schema

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicE7X7iaykicoQCFgOCSzBtBtiaQRhRzOJzAjLs6D2ibZFQXG8Jgc3hicPV8MSBm88PgIGiccUM9SJqSe90w/640?wx_fmt=png)

####  业务场景落地

基础依赖工作准备好了，那想要具体在业务中去使用，还需要做什么呢？以下以总部给门店下发任务为例，列举一个简单的场景：

我们需要收集门店门头尺寸的情况，以便在给门店在制定菜单、宣传横幅等时能更加清晰明确统计各种尺寸的数量。

首先配置JSON Scheme，然后对端渲染成表单，让门店能填写和提交表单；

然后下发给门店的任务，不仅需要需要考虑是否每个门店都收到任务，还需要保证数据的完整性、正确性，所以还需要引入流程节点的能力；表现就是门店填写完任务表单后，会有一个角色来对门店提交的数据做正确性审核，以及督促门店及时完成任务，在古茗这个岗位叫督导，负责片区门店的管理、监督工作；当然其他可能还有其他后续节点，比如财务等等。

有了流程节点后，在每个流程节点内，可能还需要支持某些表单额外的诉求，举个简单例子，在门店节点时，有一个字段不需要门店填写，那就需要隐藏这个字段；然后到了督导审核的节点，这个字段需要展示给督导，那么就需要显示。

所以我们设计了在下发的任务工单的数据结构：

  1. **表单的JSON Schema描述** ：用于渲染表单。 
  2. **表单数据** ：用于初始化表单和表单提交后记录表单数据详情。 
  3. **节点定义：** 判断工单处于什么流程，门店节点(SHOP_TASK)、督导节点(SUPERVISOR_TASK)、财务节点等等。 
  4. **节点属性**
     * 流程处理逻辑 
     * 节点的自定义处理逻辑 
     * 节点操作的Hook 
     * 节点上下文 

#####  表单的数据结构

    
    
    {  
      jsonSchema // 表单描述  
      fromData // 表单数据  
      effects: {  
        nodeA: { // 节点，门店\督导  
          hook // 自定义处理逻辑  
          action // 流程处理  
          scope // js block 需要的上下文字段  
          beforeSubmit // 表单提交前勾子  
          afterSubmit // 表单提交后勾子  
        }  
      }  
    }  
    

#####  effects配置

  * hook ![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicE7X7iaykicoQCFgOCSzBtBtiaKZEoqiaycZFYCSWXmFogfW1OljSBrgK03A6wHKwtNfESx8U16AvMd9w/640?wx_fmt=png)

#####  执行流程

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicE7X7iaykicoQCFgOCSzBtBtiaHXmcm1Z3RicULeZMKicQEMzh9IKLEcbDlrxXxnLApZXPq1hnMZ5pYkzA/640?wx_fmt=png)

####  解析器

effects里面存在着自定义逻辑处理、流程处理、勾子，这些代码在给到表单渲染时都是字符串，那么怎么执行呢？我们第一反应肯定是想到eval、new
Function，但由于小程序安全机制「小程序对动态执行脚本的限制」，不支持eval、new Function这些常用的字符串表达式执行方式

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicE7X7iaykicoQCFgOCSzBtBtiaZGFGOHIFhYuAh5S5ic6wNo0FcR4CNgZd3vlcrM7IwCsxQLcpO4aco1A/640?wx_fmt=png)

所以我们需要探索能执行字符串表达式的方法：

#####  老方案

最开始我们需要支持的能力较简单时，比如解析单行表达式，

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicE7X7iaykicoQCFgOCSzBtBtiaYTrCSk2uAuAIsFhOcntm3qCST24f5QxUXT1hTN7SdaGhflHdLTQAYg/640?wx_fmt=png)

首先会获取到表达式，$deps[0]为Formily的内置表达式作用域

    
    
    $deps[0] === '3' ? 'visible' : 'none'  
    

  * 解析执行 

    
    
    import { Parser } from 'acorn';  
      
    export function CustomCompiler(input, scope) {  
      const parseNode = Parser.parse(input, {  
        ecmaVersion: 'latest',  
        locations: false,  
      });  
      const targetNode = parseNode?.body?.[0];  
      
      if (!targetNode) return false;  
      
      const customParser = new CustomParse({ node: targetNode, scope, input });  
      const result = customParser.execute();  
        
      return result;  
    }  
    

  * Parse：https://astexplorer.net/ 

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicE7X7iaykicoQCFgOCSzBtBtia3EDiad7XbMoxD6LBou7nMPnowA7W7Vgo5FyficbibAPuKywHnmp5CKNYQ/640?wx_fmt=png)

  * CustomParse类（长度原因，省略了代码） 

    
    
    import { PARSE_FN_MAP } from './config';  
      
    export class CustomParse {  
      constructor(props) {  
        const { node, scope, input } = props;  
        this.node = node;  
        this.scope = scope;  
        this.input = input;  
        this.nodeType = node.expression.type;  
      }  
      // a ? b : c  
      parseConditional(options) {  
        const {  
          currentNode: { test, consequent, alternate },  
        } = options;  
        const testVal = this[PARSE_FN_MAP[test.type]]({ currentNode: test });  
        const consequentVal = this[PARSE_FN_MAP[consequent.type]]({ currentNode: consequent });  
        const alternateVal = this[PARSE_FN_MAP[alternate.type]]({ currentNode: alternate });  
      
        return testVal ? consequentVal : alternateVal;  
      }  
      // 1  
      parseLiteral() {}  
      // a  
      parseIdentifier() {}  
      // a.b.c  
      parseMember() {}  
      // !a  
      parseUnary() {}  
      // a && b ...  
      parseLogical() {}  
      // a === b ...  
      parseBinary() {}  
      // a.b()  
      parseCall() {}  
      // a?.b || a?.b()  
      parseChain() {}  
      
      execute() {  
        const parseOptions = { currentNode: this.node.expression };  
        const fnName = PARSE_FN_MAP[this.nodeType];  
        if (!fnName) return;  
        return this[fnName](parseOptions);  
      }  
    }  
    

  * 配置 

    
    
    export const PARSE_FN_MAP = {  
      Identifier: 'parseIdentifier',  
      MemberExpression: 'parseMember',  
      LogicalExpression: 'parseLogical',  
      BinaryExpression: 'parseBinary',  
      Literal: 'parseLiteral',  
      ConditionalExpression: 'parseConditional',  
      UnaryExpression: 'parseUnary',  
      CallExpression: 'parseCall',  
      ChainExpression: 'parseChain',  
    };  
    

#####  目前方案

但当需要支持越来越复杂的表达式（支持多行语句、支持ES6等等）时我们就去社区拥抱了开源

    
    
    export function customCompiler(input, scope) {  
      const sandbox = new Sandbox();  
      const exec = sandbox.compile(`return ${input}`);  
      const result = exec({ ...scope, dayjs }).run();  
      
      return result;  
    }  
    

  * 用于解析执行json schema配置内的表达式 

    
    
    import { Schema } from '@formily/react';  
      
    Schema.registerCompiler(customCompiler);  
    

  * 用于解析执行effects内方法 

    
    
    import { customCompiler } from '@guming/***';  
    customCompiler(effects?.[nodeId]?.hook, {  
      form: f,  
      ...basicEffectsParams,  
      ...(effects?.[nodeId]?.scope || {}),  
    });  
    

但是目前的方式也有缺陷，上下文不稳定，所以大家有知道更好的开源解析器的话可以推荐给我们，感激不尽；

#####  推荐

如果对解析器感兴趣的这里推荐几篇文章，这里不做展开了：

https://github.com/peakchen90/how-to-write-an-ast-parser/blob/master/README.md

https://github.com/jamiebuilds/the-super-tiny-compiler/blob/master/README.md

####  预览

表单渲染效果如图

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicE7X7iaykicoQCFgOCSzBtBtia0swFGEOTgTsHpOyV8ptw9EqHsmzdxtibPFNxpWAuUdM4lJKUibJUfOuQ/640?wx_fmt=png)

##  未来

目前的表单配置还没有可视化搭建能力，所以都是由我们前端开发同学手动编写JSON
schema，由于Formily支持的能力很强大，有很多配置，再加上组件支持的属性配置，就会导致手工配置一个表单json
schema上手有难度，而且容易出错，会耗费大量的时间，更主要的业务方不能脱离技术独立使用，所以团队的同学已经着手支持配置平台可视化搭建能力。

如下图，主体分为左中右三块，左边的组件和右边的组件配置都是通过读取Gudesign组件库和自定义组件的typescript配置动态生成，保证了表单和组件能力的一致性；应该不久后就会上线交付业务方使用（如果大家感兴趣，也可以邀请作者做后续分享）

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicE7X7iaykicoQCFgOCSzBtBtiaBqWvPot6FEEnEtxvJUkRMoCo9SXBDcaBoibZT2oNgompepicJkWPswpw/640?wx_fmt=png)

##  总结

Formily在性能、低代码、跨端、联动上的能力给我们业务带来了很大的助力，如果你的业务也急需这些能力，不妨一试，还是很值得推荐的。

当然Formily不是一本万利，引入Formily也带来了其他问题，比如Formily的理解成本和上手难度比其他方案更高，当json
schema越来越多、越来越大时带来的的维护成本也会更高，简单业务场景的ROI可能不高。

最后叠下甲，我们的方案可能不是最完美的，只是目前适合我们当前的业务，我们会随着业务发展逐步探索更优的解决方案，大家如果有好的想法也可以沟通交流讨论。

##  最后

关注公众号「Goodme前端团队」，获取更多干货实践，欢迎交流分享~

  

预览时标签不可点

微信扫一扫  
关注该公众号





****



****



×  分析

  收藏

