![cover_image](https://mmbiz.qpic.cn/mmbiz_jpg/TpB2QHJbiaicG9kF3hfxkxwwNzMRprBDzN1VjRNibfczy19yHYKVOP9vLo3icalLZb3SDUawbVzlhtjfxqEAGibBYlQ/0?wx_fmt=jpeg)

#  如何在 taro 中通过动态插入节点实现 API Toast

潘宇  [ Goodme前端团队 ](javascript:void\(0\);)

__ _ _ _ _

##  什么是动态插入节点

在 ` web ` 端加入我们需要实现一个可以通过API调用的 ` Toast ` ，我们可以这样实现:

    
    
    const Toast = (props)=>{  
        return <div>{props.msg}</div>  
    }  
    const showToast = (options)=>{  
        const div = document.createElement('div');  
        div.setAttribute('id', 'toast');  
        document.body.appendChild(div);  
        ReactDOM.render(React.createElement(Toast, options), div);  
    }  
    const hideTosat = ()=>{  
        if(!document.getElementById("toast")) return;  
        document.getElementById("toast").remove()  
    }  
    

我们依赖的就是浏览器 ` Document ` 对象,提供的可以操作 ` dom ` 的方法。但是我们都知道小程序没有提供 ` Document `
对象，那么我们该如何去实现类似的功能呢？

##  在 ` taro ` 中动态插入节点的实现

    
    
    import React from 'react';  
    import Taro from '@tarojs/taro';  
    import { render, unmountComponentAtNode } from '@tarojs/react';  
    import { document, TaroRootElement } from '@tarojs/runtime';  
    import { View } from '@tarojs/components';  
      
    const Demo = ({ msg }) => {  
      return <View style={{ position: 'fixed', top: 0 }}>{msg}</View>;  
    };  
      
    export const createNotification = (msg: string) => {  
        const view = document.createElement('view');  
        const currentPages = Taro.getCurrentPages();  
        const currentPage = currentPages[currentPages.length - 1];// 获取当前页面对象  
        const path = currentPage.$taroPath;  
        const pageElement = document.getElementById<TaroRootElement>(path);  
        render(<Demo msg={msg} />, view);  
        pageElement?.appendChild(view);  
    };  
      
      
    export const destroyNotification = (node) => {  
        const currentPages = Taro.getCurrentPages();  
        const currentPage = currentPages[currentPages.length - 1];  
        const path = currentPage.$taroPath;  
        const pageElement = document.getElementById<TaroRootElement>(path);  
        unmountComponentAtNode(node);  
        pageElement?.remove(node);  
    };  
    

##  为什么可以这样实现？

###  引文

在使用 React 时，我们会引用 ` react ` 和 ` react-dom ` 。而在 ` react-dom ` 中依赖 ` react-
reconciler ` 。

那么三者各自负责什么部分，又有什么联系呢？推荐阅读 react-dom 与 react 之间的关系

总结来说： ` react ` 负责描述特性，提供 ` React API ` ， ` react-dom ` 负责实现特性操作真实的 ` dom `
节点。

###  taro 中的 react-dom

我们知道 web端 有 ` react-dom ` , 移动端 有 ` react-native ` 。那么 taro 又是通过什么操作小程序 dom
的呢？答案是 ` @tarojs/react ` ，与 web端、移动端一样 ` @tarojs/react ` 依赖 ` react-reconciler
` 。

但我们都知道小程序并没有提供像 web端 一样的操作 dom 的方法。那么 ` @tarojs/react ` 又是怎么实现对小程序 ` dom `
的操作的呢？答案是 ` @tarojs/runtime ` 。

####  源码

> 以下源码都省略了一些实现细节，只剩下了核心的实现逻辑伪代码。

#####  TaroDocument

> 就是上述 ` demo ` 中所引入的 ` document ` 对象
    
    
    export class TaroDocument extends TaroElement {  
        // ...  
    }  
    

#####  TaroElement

    
    
    export class TaroElement extends TaroNode {  
        // ...  
    }  
    

#####  TaroNode

通过层层继承关系最终 ` document.appendChild() ` 方法调用的是 ` TaroNode ` 类中的 ` appendChild `
。而 ` TaroNode ` 类中的 ` appendChild ` 方法通过层层逻辑调用最终调用了 `
this._root?.enqueueUpdate(payload) ` 即根节点中的 ` enqueueUpdate ` 方法，也就是 `
TaroRootElement ` 类中的 ` enqueueUpdate ` 方法

    
    
    export class TaroNode extends TaroEventTarget {  
      
      /**  
       * @doc https://developer.mozilla.org/zh-CN/docs/Web/API/Node/appendChild  
       * @scenario  
       * [A,B,C]  
       *   1. append C, C has no parent  
       *   2. append C, C has the same parent of B  
       *   3. append C, C has the different parent of B  
       */  
      public appendChild (newChild: TaroNode) {  
        return this.insertBefore(newChild)  
      }  
        
        /**  
       * @doc https://developer.mozilla.org/zh-CN/docs/Web/API/Node/insertBefore  
       * @scenario  
       * [A,B,C]  
       *   1. insert D before C, D has no parent  
       *   2. insert D before C, D has the same parent of C  
       *   3. insert D before C, D has the different parent of C  
       */  
      public insertBefore<T extends TaroNode> (newChild: T, refChild?: TaroNode | null, isReplace?: boolean): T {  
        
         // ... 省略部分逻辑 ...  
        
         // Serialization  
        if (this._root) {  
          if (!refChild) {  
            // appendChild  
            const isOnlyChild = this.childNodes.length === 1  
            if (isOnlyChild) {  
              this.updateChildNodes() // 更新节点  
            } else {  
              this.enqueueUpdate({  
                path: newChild._path,  
                value: this.hydrate(newChild)  
              }) // 更新节点  
            }  
          } else if (isReplace) {  
            // replaceChild  
            this.enqueueUpdate({  
              path: newChild._path,  
              value: this.hydrate(newChild)  
            }) // 更新节点  
          } else {  
            // insertBefore  
            this.updateChildNodes() // 更新节点  
          }  
        }  
          
        // ... 省略部分逻辑 ...  
      }  
        
      private updateChildNodes (isClean?: boolean) {  
        const cleanChildNodes = () => []  
        const rerenderChildNodes = () => {  
          const childNodes = this.childNodes.filter(node => !isComment(node))  
          return childNodes.map(hydrate)  
        }  
      
        this.enqueueUpdate({  
          path: `${this._path}.${CHILDNODES}`,  
          value: isClean ? cleanChildNodes : rerenderChildNodes  
        }) // 更新节点  
      }  
        
      // ... 省略部分逻辑 ...  
        
      public enqueueUpdate (payload: UpdatePayload) {  
        this._root?.enqueueUpdate(payload)  
      }  
        
      public get _root (): TaroRootElement | null {  
        return this.parentNode?._root || null  
      }  
      
    }  
    

#####  TaroRootElement

通过层层调用 ` enqueueUpdate ` 方法最终调用 ` performUpdate ` 方法。而 ` performUpdate `
方法的核心是 ` ctx.setData() ` 是不是感觉和微信页面的 ` setData ` 很像？其实不止是像，他调用的就是微信的 ` setData
` 。

    
    
    export class TaroRootElement extends TaroElement {  
        
      // ... 省略部分逻辑 ...  
        
      public ctx: null | MpInstance = null  
        
      public get _root (): TaroRootElement {  
        return this  
      }  
      
      public enqueueUpdate (payload: UpdatePayload): void {  
        this.updatePayloads.push(payload)  
      
        if (!this.pendingUpdate && this.ctx) {  
          this.performUpdate()  
        }  
      }  
        
      public performUpdate (initRender = false, prerender?: Func) {  
        this.pendingUpdate = true  
      
        const ctx = this.ctx!  
          
        // ... 省略部分逻辑 ...  
           
        // custom-wrapper setData  
        if (customWrpperCount) {  
          customWrapperMap.forEach((data, ctx) => {  
            if (process.env.NODE_ENV !== 'production' && options.debug) {  
              // eslint-disable-next-line no-console  
              console.log('custom wrapper setData: ', data)  
            }  
            ctx.setData(data, cb)  
          })  
        }  
      
        // page setData  
        if (isNeedNormalUpdate) {  
          if (process.env.NODE_ENV !== 'production' && options.debug) {  
            // eslint-disable-next-line no-console  
            console.log('page setData:', normalUpdate)  
          }  
          ctx.setData(normalUpdate, cb)  
        }      
      }  
        
      // ... 省略部分逻辑 ...  
    }  
    

#####  createPageConfig

仔细阅读以下源码可知

  * 获取页面根节点的方法为 ` document.getElementById<TaroRootElement>($taroPath) ` 。 
  * ` TaroRootElement ` 中 ` ctx ` 的值为页面 ` this ` ，所以 ` TaroRootElement ` 中 ` ctx.setState ` 其实就是微信页面的 ` this.setState `

    
    
    export function createPageConfig (component: any, pageName?: string, data?: Record<string, unknown>, pageConfig?: PageConfig) {  
      // 小程序 Page 构造器是一个傲娇小公主，不能把复杂的对象挂载到参数上  
      const id = pageName ?? `taro_page_${pageId()}`  
        
      // ... 省略部分逻辑 ...  
        
      const config: PageInstance = {  
          [ONLOAD] (this: MpInstance, options: Readonly<Record<string, unknown>> = {}, cb?: Func) {  
          hasLoaded = new Promise(resolve => { loadResolver = resolve })  
      
          perf.start(PAGE_INIT)  
      
          Current.page = this as any  
          this.config = pageConfig || {}  
      
          // this.$taroPath 是页面唯一标识  
          const uniqueOptions = Object.assign({}, options, { $taroTimestamp: Date.now() })  
          const $taroPath = this.$taroPath = getPath(id, uniqueOptions)  
          if (process.env.TARO_ENV === 'h5') {  
            config.path = $taroPath  
          }  
          // this.$taroParams 作为暴露给开发者的页面参数对象，可以被随意修改  
          if (this.$taroParams == null) {  
            this.$taroParams = uniqueOptions  
          }  
      
          setCurrentRouter(this)  
      
          const mount = () => {  
            Current.app!.mount!(component, $taroPath, () => {  
              pageElement = env.document.getElementById<TaroRootElement>($taroPath) // 在这里我们知道页面根节点的查找方式  
      
              ensure(pageElement !== null, '没有找到页面实例。')  
              safeExecute($taroPath, ON_LOAD, this.$taroParams)  
              loadResolver()  
              if (process.env.TARO_ENV !== 'h5') {  
                pageElement.ctx = this // 在这里为 TaroRootElement 注入 ctx 的值为页面 this  
                pageElement.performUpdate(true, cb)  
              } else {  
                isFunction(cb) && cb()  
              }  
            })  
          }  
          if (unmounting) {  
            prepareMountList.push(mount)  
          } else {  
            mount()  
          }  
        },  
        // ... 省略部分逻辑 ...  
      }  
      // ... 省略部分逻辑 ...  
    }  
    

###  总结

    
    
    <import src="../../base.wxml"/>  
    <template is="taro_tmpl" data="{{root:root}}" />  
    

最后我们可以看看我们的小程序编译结果，其中有一个 ` base.wxml ` 是每一个页面的 ` index.wxml ` 依赖的，而每一个页面的 `
index.wxml ` 都和上面列出的代码一致。所以 ` taro ` 渲染的真相就是通过对小程序的 ` data ` 进行循环得出而 ` dom `
更新的真相就是 ` setState ` 。那么我们同样可以通过 ` @tarojs/react ` 和 ` @tarojs/runtime `
动态操作小程序 ` dom ` 与web端的体验一致。

##  最后

关注公众号「Goodme前端团队」，获取更多干货实践，欢迎交流分享~

  

预览时标签不可点

微信扫一扫  
关注该公众号





****



****



×  分析

  收藏

