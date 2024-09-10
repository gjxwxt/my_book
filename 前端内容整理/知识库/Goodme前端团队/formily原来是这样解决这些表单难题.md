![cover_image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicGib9H0ffNa3zIX2tBToC6QeTmqIrjWlM0OuTVet7EqIkCPuiaugBL556iaYU52UeMWyU6Riccmvede9g/0?wx_fmt=jpeg)

#  formily原来是这样解决这些表单难题

原创  陈南晓  [ Goodme前端团队 ](javascript:void\(0\);)

__ _ _ _ _

> 作者：陈南晓

##  背景

古茗在中后台的场景中大量的使用 formily 来解决问题。在小陈首次使用 formily 时第一感受，这玩意儿咋那么难用，能不用吗？（  现在改成
antd 来写还来得及不  ）跟老李反馈了初识 formily
遇到的难处，老李听后细心解答了古茗的中后台在使用表单始终会存在几个问题（表单数据管理、表单字段依赖、表单精准更新等等）。  

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/TpB2QHJbiaicGib9H0ffNa3zIX2tBToC6Qext9QzBwsxsResJAtayrj8LOkbBCzUsyFze1biaj4w6N80q91fiamavjA/640?wx_fmt=gif&from=appmsg)
IMG_2076.GIF

formily 仅仅作为一个 **数据载体** 帮我们去处理这类问题，不使用 formily
也照样存在你反馈的难处（但上手成本会低点🐶）。小陈回头一想是那么一回事，OK,I'm fine 是真的被他打败。那我们来看一下 formily
是如何解决中后台表单数据管理、字段依赖、精准更新问题吧～

##  表单数据管理

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGib9H0ffNa3zIX2tBToC6QekeU6zGA1hqVQk7BLeFBbnxRF9vWmSJa8JTABFoQF2MdyNZpecbEzAA/640?wx_fmt=png&from=appmsg)
当使用 createForm 创建出一个表单模型，主要包括 FormGraph 和 FormHeart 两个部分。  
其中：

  * FormGraph：表单是一个 Form 对象，表单字段是一个 Field 对象（每一个表单字段都对应一个独立的 Field ），它们的状态都由各自内部维护。这些都可以看成一个个的节点，最终都统一由 FormGraph 进行管理。 
  * FormHeart：管理的是表单的生命周期，里面包括了表单的挂载，字段状态改变等等生命周期事件。 

我们知道在 Input 组件中输入一个值，Input 组件对应的 Field 字段的 value 会更新以及 Form 中的 value 也会更新，反过来
Form 或 Field 中的 value 改变也会更新 Input 组的值，我们来看下它们是如何进行通信的  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGib9H0ffNa3zIX2tBToC6QeEFXZgfXx1wppSQpOZSx22gD1tJs3U0MyiakgB1kBNMSJdQgS0hg1wRw/640?wx_fmt=png&from=appmsg)
其中：

  * formState：维护着表单所有字段的 value 
  * fieldState：维护着当前字段的 value 
  * component：是每个字段对应的展示层组件，可以是 Input 或者 Select，也可以是其它的自定义组件 

###  Form和Field数据通信

Form 和 Field 是通过发布订阅的模式进行通信，当 field.value 改变会通知 form 表单更新 value , form 中的
value 改变也会通知 field 更新 value 。  
formily 内部实现

    
    
    // form.value 变化通知 field  
    const makeReactive = {  
      const triggerFormValuesChange = (form: Form, change: DataChange) => {  
        // 比较 form.value 是否存在变动  
        if(contains(form.values, change.object)) {  
          // 通知 field 组件，form.value 改变   
          form.notify(LifeCycleTypes.ON_FORM_VALUES_CHANGE)  
        }  
      }  
      
      observe(form,  
              (change) => {  
                xxxx  
                triggerFormValuesChange(form, change) },   
              true)  
    }  
      
    // field.value 变动通知表单  
    const onInput = (...args) => {  
      const values = getValues(args)  
      const value = values[0]  
      this.inputValue = value  
      this.value = value  
      // 通知表单 field.value 改变  
      this.notify(LifeCycleTypes.ON_FIELD_INPUT_VALUE_CHANGE)  
    }  
    

###  Field与Component数据通信

一个 Field 对应一个 Component ，当用户在 Component 中输入时，会触发对应的 onChange
事件，该事件内部把用户输入的值传递给 Field 里。在 Field 值变动时，会将 field.value 通过 props.value 的形式传递给
Component，最终达到双向绑定的效果。  
formily 内部实现

    
    
    const renderComponent = () => {  
      const events = {} as Record<string, any>  
      // 设置 field 中的 onChange 事件  
      events.change = (...args: any[]) => {  
        if (!isVoidField(field)) field.onInput(...args)  
        originChange?.(...args)  
      }  
      const componentData = {  
        attrs: {  
          // 获取 field 中的value  
          value: !isVoidField(field) ? field.value : undefined,  
        },  
        on: events,  
      }  
      // 渲染 field 的 component 组件  
      return h(component, componentData, mergedSlots)  
    }  
      
    // field.value   
    class Field {  
      construct(props) {  
        this.value = props.value;  
        this.makeObservable()  
      }  
      
      makeObservable() {  
        define(this,{  
          ...,  
          // 将 this.value 变成响应式  
          value: observable.computed,  
        })  
      }  
    }  
    

这里的 onChange 事件 field.onInput 会把用户输入的 value 赋值给 field.value 。反过来 field.value
改变会通过响应式知道哪个组件依赖它，并对其重新渲染。

##  表单字段联动

字段联动是指表单中一个字段依赖于其他字段的值，当其值发生变化时，相关联的表单元素会做出相应的变化或更新。那在 formily 中是如何实现表单联动的呢？  
在 formily 中表单的联动效果，本质上也是一个“响应式”模型，实现原理借助了 formily/core ➕ formily/reactive 。

  1. 依赖收集 

在 formily/core 中实例化 Field 实例中会执行 createReactions ，实现对依赖的收集

    
    
    const createReactions = (field: GeneralField) => {  
      const reactions = toArr(field.props.reactions)  
      // 在表单中注册该字段值变动的生命周期  
      field.form.addEffects(field, () => {  
        reactions.forEach((reaction) => {  
          if (isFn(reaction)) {  
            field.disposers.push(  
              // 收集依赖  
              ....  
              autorun(reaction(field))  
            )  
          }  
        })  
      })  
    }  
    

我们再进一步看看 autorun 是个啥玩意，怎么就能够收集依赖了呢？这里实现简易版的
autorun（关于“响应式”模型感兴趣的小伙伴可以看看这篇从零开始撸一个「响应式」框架

    
    
    let ReactionStack;  
    const RawReactionsMap = new WeakMap();  
      
    export function observable(value) {  
      return new Proxy(value, baseHandler);  
    }  
      
    const baseHandler: any = {  
      get(target, key) {  
        const result = target[key];  
        const current = ReactionStack  
        if (current) {  
          // 当前存在响应器  
          addRawReactionsMap(target, key, current);  
        }  
        return result;  
      },  
      set(target, key, value) {  
        target[key] = value;  
        RawReactionsMap.get(target)  
          ?.get(key)  
          ?.forEach((reaction) => reaction());  
        return true;  
      },  
    };  
      
    function addRawReactionsMap(target, key, reaction) {  
      const reactionsMap = RawReactionsMap.get(target);  
      
      if (reactionsMap) {  
        const reactions = reactionsMap.get(key);  
        if (reactions) {  
          reactions.push(reaction);  
        } else {  
          reactionsMap.set(key, [reaction]);  
        }  
        return reactionsMap;  
      } else {  
        const reactionsMap = new Map();  
        reactionsMap.set(key, [reaction]);  
        RawReactionsMap.set(target, reactionsMap);  
        return reactionsMap;  
      }  
    }  
      
    export function autorun(tracker) {  
      // reaction作为响应器，它也是一个函数  
      const reaction = () => {  
        ReactionStack = reaction;  
        tracker();  
        ReactionStack = null;  
      };  
      reaction();  
    }  
    

由于 reaction(field) 中有引用 fieldState ，所以触发 baseHandler 中的 get 属性，实现对依赖的收集。

  2. 依赖监听 

在上述 autorun 中实现的 get 属性实现了对依赖的收集，其中的 set 属性就是对依赖的监听，当依赖的值发生改变时会触发 set，执行
reaction 更新表单字段。  
表单的联动在表单中是非常重要的，包含了字段间的各种关系。同时字段与字段关联时，还要保证不影响表单性能，下面我们再走进 formily
深处，看看它凭啥是表单高性能的解决方案。

##  表单精准更新

表单的精准更新本质上也是响应式原理，类似于表单组件里面引用了子组件，该如何正确的知道是哪个组件依赖了当前表单字段，另外当表单字段更新时如何做到只更新对应的子组件。  
formily 中把上述的 autorun 中的全局变量 ReactionStack 改成一个栈形式，记录调用依赖的函数。下面我们来看一个栗子，假如 A
组件中引用了 B 组件，B 组件中引用了 C 组件。最终的形式 ** ** ，然而只有 C 组件使用的该依赖值，我们只需要重新渲染 C 组件即可，A、B
组件不需要重新渲染。  
在依次执行 A、B、C 组件时，当执行到 C 组件时，发现有对依赖进行引用，对其进行依赖收集，即
ReactionStack[ReactionStack.length - 1] 此时为 C
组件，初始化结束后依次按顺序出栈。之后依赖的值更改时由于收集到的依赖是 C 组件，所以当依赖值改变时，只会重新渲染 C 组件。

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/TpB2QHJbiaicGib9H0ffNa3zIX2tBToC6QemeldViaLeicwP7QxUYvlcNwjU8IHTB3qic4nFwc89NtRIGibuvdjviaB5JQ/640?wx_fmt=gif&from=appmsg)
我的影片.gif

    
    
    // 由于代码过多，只列出如何实现精准刷新的与上文的区别  
    let ReactionStack = [];  
      
    export function observable(value) {  
      return new Proxy(value, baseHandler);  
    }  
      
    const baseHandler: any = {  
      get(target, key) {  
        const result = target[key];  
        // current 表示当前依赖所在的执行函数  
        const current = ReactionStack[ReactionStack.length - 1]  
        if (current) {  
          // 当前存在响应器  
          addRawReactionsMap(target, key, current);  
        }  
        return result;  
      },  
      ...  
        };  
      
    export function autorun(tracker) {  
      // reaction作为响应器，它也是一个函数  
      const reaction = () => {  
        ReactionStack.push(reaction);  
        tracker();  
        ReactionStack.pop();  
      };  
      reaction();  
    }  
    

##  总结

在中后台的业务开发中与表单的邂逅是不可避免的，社区上也有各式各样的表单解决方案，正式练习快满一年的小陈同学从一开始对 formily 的疯狂 diss
，到现在发现更多只是使用方式的差别。🙏大家的阅读，如文章中有错误👏评论，还有如果有建议或者不同的想法，欢迎留言 也可以期待一下我们的下一篇文章
salute🫡~~

##  最后

📚 小茗文章推荐：

  * [ 古茗是如何将小程序编译速度提升3倍的 ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247485381&idx=1&sn=cb63b78c665a492408dcd353a821e9ec&chksm=cfe580c2f89209d40945633c906bfb73dd6c0e976e745d4f0c512a443ec1a404d21e81103e09&scene=21#wechat_redirect)
  * [ 钉钉小程序实现签名板 ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247485317&idx=1&sn=a43f9c932a7989c0d3f8714a73bb538c&chksm=cfe58082f8920994cd45f953dec28a653768f312a9ee8728edf13bc141ec3f5bc48be4cc5dfd&scene=21#wechat_redirect)
  * [ 小程序主包体积的优化方案与技术实现 ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247485223&idx=1&sn=013a3cd62449303136a8e8556b688f3e&chksm=cfe58020f89209365a3a0620e7269b9ac7243cd7f793fc1802a863c3d40cb7b6bad462d05064&scene=21#wechat_redirect)   

关注公众号「Goodme前端团队」，获取更多干货实践，欢迎交流分享~

  

  

预览时标签不可点

微信扫一扫  
关注该公众号





****



****



×  分析

  收藏

