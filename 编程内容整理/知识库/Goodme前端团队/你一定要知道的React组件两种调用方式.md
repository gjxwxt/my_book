![cover_image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicH7vwL6LHHVM0AT1oHzo2OHaTrnu5UEF2sH8YQUVqCOYWyNIWntFViblyrqxvaCVyutrkXjc9VeMgQ/0?wx_fmt=jpeg)

#  你一定要知道的「React组件」两种调用方式

原创  盛靖  [ Goodme前端团队 ](javascript:void\(0\);)

__ _ _ _ _

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicH7vwL6LHHVM0AT1oHzo2OHnEMnqN02IzASd2KOjpicl1JicvpL5e11icYxTibN8t2ibKKxv93mGf2XyDA/640?wx_fmt=png&from=appmsg)

##  前言

使用React框架的开发过程中，我们常常会使用两种方式调用组件：一种是组件式，另一种是函数式，但是这两种用法究竟有何不同？

##  一、代码示例

**App.jsx**

    
    
    // 当作普通组件使用  
    function MyComponent() {  
      return <div>Hello Guming!</div>  
    };  
      
    function App() {  
      return (  
        <div className="App">  
          <MyComponent />  
        </div>  
      );  
    }  
      
    export default App;  
      
    
    
    
    // 当作函数使用  
    function MyComponent() {  
      return <div>Hello Guming!</div>  
    };  
      
    function App() {  
      return (  
        <div className="App">  
          {MyComponent()}  
        </div>  
      );  
    }  
      
    export default App;  
      
    

这是一个简单的例子，还没有涉及到状态。在App.jsx文件中，声明了一个MyComponent组件，并在App入口函数中使用。我们知道，上述代码并不能直接执行，需要先被babel插件编译。

  
**编译后的结果：**

    
    
    // 当作普通组件使用  
    function MyComponent() {  
      return React.createElement("div", null, "Hello Guming!");  
    };  
      
    function App() {  
      return React.createElement("div", {  
        className: "App"  
      }, React.createElement(MyComponent, null));  
    }  
    export default App;  
    
    
    
    // 当作函数使用  
    function MyComponent() {  
      return React.createElement("div", null, "Hello Guming!");  
    };  
      
    function App() {  
      return React.createElement("div", {  
        className: "App"  
      }, MyComponent());  
    }  
    export default App;  
    

React.createElement的作用是创建ReactElement对象，它接收 **三个参数** ：

  1. **type — 元素类型；**
  2. **config — 元素属性；**
  3. **children — 元素子节点；**

在两种用法当中，我们可以看到经过babel插件编译后的区别在于函数的最后一个参数。  
当我们使用组件的方式时，babel插件会将“  "编译成React.createElement(MyComponent,
null)；而对于函数的用法，则变成了一次函数调用，而执行后，会返回

    
    
    React.createElement("div", null, "Hello Guming!")  
    

  

##  二、生成Fiber树结构的不同

我们知道，React会创建Fiber树，而Fiber节点创建的依据就是ReactElement对象。每次更新时，React都会创建一个新的workInProgress树，期间包d含diff和复用的逻辑(后面会提到)，然后根据workInProgress树依次创建dom节点，并将根节点的current指向新建的树的根节点(RootFiber)上。  
render阶段的工作，可以分为“递”(beginWork)与“归”(completeWork)。在"递"阶段，React会根据依次根据当前节点创建其子节点，直至构建出完整的Fiber树；而在“归”阶段，则是进行一些元素标记和标记的“冒泡”。在mount和update时，这“递”与“归”流程也有所不同，大体如下图所示：

**“递”阶段**  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicH7vwL6LHHVM0AT1oHzo2OHASjrDJf2GMfQ2oOht6DkJzM5PrdZ4rZMXx50NbpLW2xQOmlt8jicnEg/640?wx_fmt=jpeg&from=appmsg)

**“归”阶段**  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicH7vwL6LHHVM0AT1oHzo2OHptibZqUTkJBVQ2jcB2w8ITFRLZAuDk09A3z0aUfZKLMkbMVrOxp5xYA/640?wx_fmt=jpeg&from=appmsg)

回到例子当中来，在创建好React
Element之后，接下来需要构建workInProgress树。相比函数式的使用方式，使用组件的方法多创建了一个Fiber节点：  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicH7vwL6LHHVM0AT1oHzo2OHz9t2tNR3SmnYq3HJ8ibJluMsaE3gOeWOWOUZqqSrczDYDfC8R8Y20tw/640?wx_fmt=jpeg&from=appmsg)

**MyComponent组件的Fiber节点**  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicH7vwL6LHHVM0AT1oHzo2OHtubXbrbJzNibQNbIlaQ9lFny3ACpIXvIMzbp0BxJK0kfZWFicT4z0p3w/640?wx_fmt=png&from=appmsg)  
从上图可以看出来，这个节点的type(类型)就是MyComponent函数地址。

在completeWork时，会根据Fiber创建对应的dom节点，由于MyComponent对应fiberNode的tag为FunctionComponent，因此在执行完bubbleProperties函数后会不会创建dom
element而是直接返回null。bubbleProperties函数主要作用是节点属性的冒泡。  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicH7vwL6LHHVM0AT1oHzo2OHecJt7P4kMdODYy2s02kEVoCPIVkLuenTlz9ngnJnNpVVpNh08zXS2w/640?wx_fmt=png&from=appmsg)

##  三、组件的函数执行的时机不同

对于FunctionComponent，生成的fiberNode的type就是组件的执行函数本身。  
在beginWork时，如果是在mount时，则会调用mountIndeterminateComponent函数；如果是在update时，则会调用updateFunctionComponent函数，而在两个函数执行过程中，都会调用renderWithHooks这个函数。而在renderWithHooks函数中，有这样一段代码：  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicH7vwL6LHHVM0AT1oHzo2OH0v4Cpy8PApr8fS824tCoLuP3a4bRl2ATz9dkiaOxOZCiajBsG7pINuOA/640?wx_fmt=png&from=appmsg)  
而Component就是组件函数本身。在renderWithHooks函数打上断点，我们可以发现：  
对于组件使用的方式，当前处于beginWork的节点是MyComponent的fiberNode:  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicH7vwL6LHHVM0AT1oHzo2OH3Ka7WcHOjQrMLPZOXCBiaBS9xYfwBcKoGftt8hdQ6j84fZZ5gGIdxlQ/640?wx_fmt=png&from=appmsg)  
而对于函数使用的方式，当前处于beginWork的节点是App对应的fiberNode:  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicH7vwL6LHHVM0AT1oHzo2OHgDqChQJjicj2icBqYmcO7dXnX1Zqzg9csfC8gsbbZBuIfcqthRvFicuibQ/640?wx_fmt=png&from=appmsg)

##  四、进行某些算法比较时的流程不同

  

###  1.bailout策略-节点复用

在update时，出于性能考虑，React会进行一些优化。在update时，如果满足以下条件：

  * 新旧节点的属性相同 
  * 当前节点的类型没有发生改变(例如没有从div变成p) 
  * 当前节点不存在更新 
  * context没有发生变化(由于代码中没有使用Context相关API，因此这个条件一直满足) 

那么React会对子节点进行复用，如果当前节点的子节点不存在任何更新，则会跳过整个子树的beginWork。  
有一个点需要注意，在对比新旧节点属性时，由于每次render时，创建的JSX对象都是新的地址，因此新旧节点属性比较并不相同，除非当前节点的父节点满足bailout逻辑，复用了当前节点。

接下来改造一下代码，引入状态更新：

    
    
    import { useState } from "React";  
      
    // 当作普通组件使用  
    function MyComponent({count}) {  
      return <div>I has drunk {count} guming milky tea!</div>  
    };  
      
    function App() {  
      const [num, setNum] = useState(0);  
      
      return (  
        <div className="App" onClick={() => {setNum(1)}}>  
          <MyComponent count={num}/>  
        </div>  
      );  
    }  
      
    export default App;  
      
    
    
    
    import { useState } from "React";  
      
    // 当作函数使用  
    function MyComponent(count) {  
      return <div>I has drunk {count} guming milky tea!</div>  
    };  
      
    function App() {  
      const [num, setNum] = useState(0);  
      
      return (  
        <div className="App" onClick={() => {setNum(1)}}>  
          {MyComponent(num)}  
        </div>  
      );  
    }  
      
    export default App;  
      
    

(1).我们第一次点击文字时，会时num值加1，此时第一个进入beginWork的节点是HostFiberRooter，此时肯定满足以下条件：

  * 新旧节点的属性相同(全等) 
  * 当前节点的类型没有发生改变(例如没有从div变成p) 
  * 当前节点不存在更新 
  * context没有发生变化 

因此App对应的fiberNode被复用。  

(2). 接下来进入App对应fiberNode的beginWork，由于我们点击文字触发了更新，因此App存在更新，不满足bailout逻辑。  

(3).
再创建完div对应的fiberNode后，进入div的beginWork。由于其父节点App没有被复用，因此在比较当前节点的新旧属性时不相等，不满足bailout逻辑。  

(4). **对于组件使用的方式**
，由于多了一个MyComnonet节点，因此下一个进入beginWork的是MyComponent。由于其父节点div没有被复用，并且节点属性发生变化(count属性由0变为1)，因此不会复用子节点；
**对于函数调用方式** ，下一个beginWork的节点则是div，同样由于副节点未被复用，导致在比较oldProps和newProps时不相同。  

(5).除了MyComponent，由于两者之后的fiber树的构造相同，后面节点的beginWork流程也是一样，同样无法被复用；

###  2.diff算法

在beginWork时，还设计到diff，在React源码中，主要diff逻辑reconcileSingleElement方法中。网上已经存在很多关于React
diff算法的文章，这里不过多讲解，有兴趣的读者可以参考这篇：  
【动图+大白话🍓解析React源码】Render阶段中Fiber树的初始化与对比更新～ - 掘金

以下是reconcileSingleElement函数的内容

    
    
    function reconcileSingleElement(returnFiber, currentFirstChild, element, lanes) {  
      var key = element.key;  
      var child = currentFirstChild;  
      
      while (child !== null) {  
        // TODO: If key === null and child.key === null, then this only applies to  
        // the first item in the list.  
        if (child.key === key) {  
          var elementType = element.type;  
      
          if (elementType === REACT_FRAGMENT_TYPE) {  
            if (child.tag === Fragment) {  
              deleteRemainingChildren(returnFiber, child.sibling);  
              var existing = useFiber(child, element.props.children);  
              existing.return = returnFiber;  
      
              {  
                existing._debugSource = element._source;  
                existing._debugOwner = element._owner;  
              }  
      
              return existing;  
            }  
          } else {  
            if (child.elementType === elementType || ( // Keep this check inline so it only runs on the false path:  
              isCompatibleFamilyForHotReloading(child, element) ) || // Lazy types should reconcile their resolved type.  
                // We need to do this after the Hot Reloading check above,  
                // because hot reloading has different semantics than prod because  
                // it doesn't resuspend. So we can't let the call below suspend.  
                typeof elementType === 'object' && elementType !== null && elementType.$$typeof === REACT_LAZY_TYPE && resolveLazy(elementType) === child.type) {  
              deleteRemainingChildren(returnFiber, child.sibling);  
      
              var _existing = useFiber(child, element.props);  
      
              _existing.ref = coerceRef(returnFiber, child, element);  
              _existing.return = returnFiber;  
      
              {  
                _existing._debugSource = element._source;  
                _existing._debugOwner = element._owner;  
              }  
      
              return _existing;  
            }  
          } // Didn't match.  
      
      
          deleteRemainingChildren(returnFiber, child);  
          break;  
        } else {  
          deleteChild(returnFiber, child);  
        }  
      
        child = child.sibling;  
      }  
      
      if (element.type === REACT_FRAGMENT_TYPE) {  
        var created = createFiberFromFragment(element.props.children, returnFiber.mode, lanes, element.key);  
        created.return = returnFiber;  
        return created;  
      } else {  
        var _created4 = createFiberFromElement(element, returnFiber.mode, lanes);  
      
        _created4.ref = coerceRef(returnFiber, currentFirstChild, element);  
        _created4.return = returnFiber;  
        return _created4;  
      }  
    }  
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicH7vwL6LHHVM0AT1oHzo2OHEtqxSRayKAddh8NeaOcQcCEnZ5ibblItLSWcudibtZd4t0PsfjiarE00w/640?wx_fmt=png&from=appmsg)  
当执行到MyComponent对应fiberNode的beginWork时，我们可以看到，其child属性为null，并不会进入下面的while循环，因此MyComponent是不参与diff比较的。  
在update时，completeWork工作主要是针对前后属性有变化的节点进行标记，MyComponent对diff的结果没有影响，所以对最后diff的结果并没有影响。

##  五、结论

本文通过几个例子，简单讲解了用组件方式和函数方式使用自定义组件的三个区别：

  * 在创建workInProgress树时，组件使用的方式会比函数使用的方式多一个fiber节点； 
  * 在组件函数执行的时机不同； 
  * 在进行diff算法比较时，MyComponent对应的节点会被忽略； 
  * 在bailout的复用逻辑中，尽管复用时多了一次节点比较，但是并不会影响最终的结果； 

React的流程都是基于Fiber的，而上述差异都是基于此点。尽管两种使用方式生成的Fiber树结构不相同，但是对于React核心的流程并没有太大影响。当然，实际的使用情况远远比本文中提出的例子更复杂，本文并没有针对所有场景和流程逐一分析。

##  最后

📚 小茗文章推荐：

  * [ 中后台业务开发（一）「表单原理」 ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247484905&idx=1&sn=88e09add6d0df02ab6217c3f9d6a6b96&chksm=cfe582eef8920bf8279f085b85ac43205d34ef15106a864a172baf318af39588f07ab41eccfe&scene=21#wechat_redirect)
  * [ 手摸手教运营小姐姐搭建一个表单 ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247484883&idx=1&sn=3ad4019b966f8fad6a6f64855755f664&chksm=cfe582d4f8920bc27f6d4522d28a7388ca254547d8ea54712573fbcc1f52525e22943c0fed01&scene=21#wechat_redirect)
  * [ 前端开发者需要了解的「设计体系」 ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247484803&idx=1&sn=42f0e93bf971b9f048cf5214fc97a506&chksm=cfe58284f8920b92927b22955c22c30345372b1505cebdc26f337b2171dbe09d0e3dbfdf1415&scene=21#wechat_redirect)

关注公众号「Goodme前端团队」，获取更多干货实践，欢迎交流分享~

  

  

预览时标签不可点

微信扫一扫  
关注该公众号





****



****



×  分析

  收藏

