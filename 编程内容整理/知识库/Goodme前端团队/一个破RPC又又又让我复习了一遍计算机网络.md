![cover_image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicGtDG715ePTbZo9W6dQqXqjNd6CAtRKCWBPqibiaXoRniaSx89icHicWcvfkQmlWg9L1yquaMbvjyoSlyQ/0?wx_fmt=jpeg)

#  一个破RPC，又又又让我复习了一遍计算机网络

于水增  [ Goodme前端团队 ](javascript:void\(0\);)

__ _ _ _ _

> 作者：于水增

如题，我又不换工作，为什么要写这篇文章🙄️。。。

###  背景

我所在的小组中有一部分同学是和设备打交道的，通过周会的分享会了解到设备的控制、设备间的通信设计等等。。。

另外以前在做需求的时候会经常从服务端同学口中听到RPC调用，那么浅浅了解下吧

先抛出几个问题

  1. RPC是谁？做什么工作的？芳龄几何...（跑题） 
  2. 怎样用，能玩出花吗？ 
  3. 我一个前端，跟我有毛关系？🙄️ 

###  先了解下RPC的概念

RPC 全称Remote Procedure
Call，即远程过程调用。在维基百科中简单理解就是，程序员无需关注网络通信的复杂性，在实际开发中调用远程方法就像调用本地程序一样简单。

说白了，就是我们经常干的函数调用，既然是远程调用，就需要解决不同进程或服务间相互数据传输也就是通信问题。

那么数据是如何在计算机中传输的？

###  聊聊数据是如何在不同应用之间传输的？

####  先回顾下 **OSI** 模型（ **开放式系统互联模型** ）

这个模型很牛X，因为它就像画了一个圈，然后业界大佬们就在这个圈圈中设计各种协议，来满足我们日益变化的需求。

根据Wiki中对各层的介绍，整理出下边图表，列出了每层中我们比较熟悉的协议，个别陌生的也做了相应的备注，但是并不是所有场景的通信都会走完每一层。比如服务与服务的通信可以直接跳过应用层、系统内部可以直接物理层、设备与设备通信也是在物理层（比如往U盘里边下点电影。。。）

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGtDG715ePTbZo9W6dQqXqjib6md3fFpibr5a666v7E2xojtmDpVjnxS4pcfw1mxcOSofKG9MZhAe7g/640?wx_fmt=png)

作为一个用户，无论是浏览器中打开一个链接（🈲️非法链接），还是打开微信去聊天。整个消息在网络上的传递是满足TCP/IP
4层概念模型的，数据层层传递（数据处理、建立连接、寻址。。。），最终满足用户的使用需求。

简单来看就是这样

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGtDG715ePTbZo9W6dQqXqjsZoc38eib9CYbITfy4E4gmOeooEF4Td6ibQuT7UF8OyS6zVSvFz38FRQ/640?wx_fmt=png)

其实，客户端与服务端的远比这个复杂，系统为我们做了很多事情，大体可以分为以下几个阶段

  1. 域名解析，查询IP（通过DNS服务查询IP地址） 
  2. 建立TCP连接（我们常说的三次握手是在这个阶段完成） 
    1. 调用socket组件创建套接字 
    2. 调用connect与服务端的套接字建立管道 
  3. 收发数据 
    1. 数据拆包 
    2. 数据确认（通过ACK号确认数据是否收到） 

  4. 断开连接并删除套接字 
    1. 四次挥手阶段。但是并不会立马删除套接字，主要防止像ACK号丢失的这种异常场景，比如客户端在发最后一个ACK后随即删除了套接字，但是服务端没有收到，所以服务端会再次发送一遍FIN，但是此时客户端已经删了呀，并且有个新的套接字被分配到了上次相同的端口号，那么FIN就会发给新的应用程序了，也就是我们常说的还没开始就结束了😂 

这部分内容很多，在每一步操作系统默默做了很多事，聊的很细的话根本聊不完。。

####  好了，可以了，我要说RPC了。。。

假如我们有下面这个场景

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGtDG715ePTbZo9W6dQqXqjjyHicfjH2wFNNrP2f2bIgiaQqLAjibEbslqQgW7AxILjJjzfT5ic1wHN1w/640?wx_fmt=png)

商品中心需要去校验库存，那么就需要两个服务进行通信，在这里就可以用RPC来实现，库存中心只需要提供一个校验库存的方法供商品中心调用。那为啥不用HTTP？别问，问就是性能不好。。。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGtDG715ePTbZo9W6dQqXqjqlrcO3Jia3hicwmxMsxp0BBXnibAJJFE71fmREx0gdArx9Qtaa3SiaT3DQ/640?wx_fmt=png)

首先，我们目前用的比较多的还是HTTP/1.1的版本，它有很多缺陷，比如消息头部增加了很多看似冗余的字段。同时还有为了方便内部解析加了很多的换行符、回车符等。另外还需要考虑浏览器的各种行为。总之服务与服务之间并不需要这些。

而RPC的灵活性、定制化程度更高。可优化的空间很大，所以很多微服务架构中都选择使用RPC。

在 **OSI**
模型图中我们发现RPC存在于会话层，其实它实现的方式有多种，可以基于HTTP/Websocket、TCP/UDP，甚至可以在物理层去实现。所以它更像一个规范、一个调用方式。目前大部分的RPC底层使用TCP。谷歌的gRPC是基于HTTP/2实现的。

####  基于HTTP来实现个RPC（虽然没什么用）

感觉基于HTTP/1.1版本来实现的RPC有点鸡肋。更像是约定了一个数据格式，本身还是通过JSON来序列化。HTTP/2改进了很多，甚至比很多RPC的性能还要好。谷歌的gRPC的底层实现就是HTTP/2。

这个小demo是用JSON-RPC来实现的，看下定义

> JSON-RPC
>
> 是一种轻量级的远程调用（RPC）协议，它使用JSON（Javascript Object
> Notation）作为数据交换格式，通过HTTP或其他传输协议在客户端和服务器之间进行通信，具有简单、轻量级、易于使用的特点。

JSON大家都知道，经常出现在项目配置文件，到Restful API规范里的响应数据、JSON Schema等。它具有易读写、易解析，具有很好的约束性。

详细的json rpc规范可参考 https://www.jsonrpc.org/specification

用HTTP实现JSON-RPC和普通HTTP请求有什么区别？

  1. 无非是将post请求中key/value格式替换为JSON格式 
  2. 服务端解析json，通过method和params两个属性进行处理业务/数据 
  3. 响应头中的Content-type变成了application/json-rpc 

###  Nodejs + HTTP + JSON-RPC

####  异常对象设计

当RPC调用遇到错误时，响应对象必须包含以下几个属性

    
    
    type ErrorBody = {  
      code: number; // 错误编码  
      message: string; // 异常描述  
      data?: any; // 附加信息的原始或结构化值，由服务器定义  
    };  
    

官方文档给出以下code

    
    
    /*  
    | code      |   message          | meaning |  
    | -32700    |   Parse error      | Invalid JSON was received by the server.An error occurred on the server while parsing the JSON text.  |  
    | -32600    |   Invalid Request  | The JSON sent is not a valid Request object.  |  
    | -32601    |   Method not found | The method does not exist / is not available.  |  
    | -32602    |   Invalid params   | Invalid method parameter(s).  |  
    | -32603    |   Internal error   | Internal JSON-RPC error.  |  
    | -32000 to -32099   |   Server error        | Reserved for implementation-defined server-errors.  |  
    */  
    

####  请求参数设计

    
    
    type RequestBody = {  
      jsonrpc: string; // 版本号，分为1.0和2.0两个版本，因存在兼容性问题，必须保证为2.0  
      method: string; // 方法名，以单词 rpc 开头  
      params?: any; // 一个结构化值，方法调用期间要使用的参数值  
      id: number｜string; // 唯一标识，由客户端建立  
    }  
    

####  响应结果设计

其中result与error为互斥关系，即成功只返回result，异常只返回error。

    
    
    type ResponseBody = {  
      jsonrpc: string; // 必须保证为2.0  
      result?: any;   
      error?: ErrorBody;  
     id: number｜string; // 必须与请求的ID相同  
    }  
    

####  封装一个JSONRPC

    
    
    export default class JSONRPC {  
      version: string = "2.0";  
      
      errorMsg = {  
        [-32700]: "Parse Error.",  
        [-32600]: "Invalid Request.",  
        [-32601]: "Method Not Found.",  
        [-32602]: "Invalid Params.",  
        [-32603]: "Internal Error.",  
      };  
      
      methods = {};  
      
      normalize(rpc, obj) {  
        obj.id = rpc && typeof rpc.id === "number" ? rpc.id : null;  
        obj.jsonrpc = this.version;  
        // 如果错误根据错误不存在错误信息的话代码获取错误信息  
        if (obj.error && !obj.error.message) {  
            obj.error.message = this.errorMsg[obj.error.code] || obj.error.message;  
        }  
        return obj;  
      }  
        
      /**  
       * JSONRPC 请求处理  
       * @param  {Object} rpc  
       * @param  {Function} response 响应回调  
       */  
      handleRequest(rpc, response) {  
        // ...版本与一些参数验证逻辑  
      
        //函数查找  
        const method = this.methods[rpc.method];  
        if (typeof method !== "function") {  
          return response(this.normalize(rpc, { error: { code: -32601 } })); // 函数或方法未找到  
        }  
          
        // 调用函数将其执行结果作为响应结果返回客户端  
        try {  
          response(this.normalize(rpc, { result: method.apply(this, rpc.params) }));  
        } catch (error: unknown) {  
          if (error instanceof Error) {  
            response(this.normalize(rpc, { error: { code: -32000, message: error.message } }));  
          } else {  
            response(  
              this.normalize(rpc, { error: { code: 0, message: "unknown error" } })  
            );  
          }  
        }  
      }  
    }  
    

####  起一个HTTP服务

    
    
    import { createServer } from "http";  
    import url from "url";  
    import JSONPRC from "./rpc";  
      
    const HOST = "localhost";  
    const PORT = 8080;  
    const RPC = new JSONPRC();  
    // 添加方法  
    RPC.methods = {  
      rpcDivide(a, b) {  
        if (b === 0) throw Error("Not allow 0");  
        return a / b;  
      },  
    };  
    // 路由设计，供客户端调用  
    const routes = {  
      "/rpc-divide": (request, response) => {  
        if (request.method === "POST") {  
          let data = "";  
          request.setEncoding("utf8");  
          request.addListener("data", (chunk) => {  
            data += chunk;  
          });  
          request.addListener("end", () => {  
            RPC.handleRequest(JSON.parse(data), (obj) => {  
              const body = JSON.stringify(obj);  
              response.writeHead(200, {  
                "Content-Type": "application/json",  
                "Content-Length": Buffer.byteLength(body),  
              });  
              response.end(body);  
            });  
          });  
        } else {  
          response.end("hello nodejs http server");  
        }  
      },  
    };  
      
    const server = createServer((request, response) => {  
      // 解析请求，包括文件名  
      const pathname = url.parse(request.url || "").pathname || "";  
      const route = routes[pathname];  
      if (route) {  
        route(request, response);  
      } else {  
        response.end("hello nodejs http server");  
      }  
    });  
      
    server.listen(PORT, HOST, 0, () => {  
      console.log(`server is listening on http://${HOST}:${PORT} ...`);  
    });  
    

####  运行效果

最终看下Postman的请求结果

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGtDG715ePTbZo9W6dQqXqjrQPapTbXhVWNVKylU6q8iciczZHsWG8Dh7rUdShbqibj9ZqzDpI2dnwiaA/640?wx_fmt=png)

出现非法请求时会跑出异常

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGtDG715ePTbZo9W6dQqXqjAaBnI0XwwCs8xYeicgclUcJmd1GVsQjf6Lc4arQgMhBadFFUHc5lCXA/640?wx_fmt=png)

完整实例代码在这里https://github.com/YSZ0927/node-json-rpc 感兴趣的可以试着调试下。。。

###  前端领域也能看到RPC的影子

####  Chrome远程调试工具

CDP（Chrome Devtools Protocol）是Chrome 的调试协议，用来调用 Chrome 内部的方法实现 js，css ，dom
的开发调试。它可以实现调试目标页面与控制台的相互通信。

首先Devtools由Fronend、Backend、Protocol、Message Channel构成。其中Protocol就是基于JSON-
RPC来实现的，而Fronend与Backend之间则通过Websocket实现消息通信

如何调试？

  1. 终端启动本地Chrome实例 

    
    
    sudo /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222 --remote-allow-origins=http://127.0.0.1:9222  
    

  2. 随便输入一个调试网址，然后通过http://127.0.0.1:9222/json来获取 websocket target 

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGtDG715ePTbZo9W6dQqXqjBDNwEsVrMGxVF98A2ibVkmkyFDHdkNXTthH3NPbq0OkHZBT6Eiaias6ibQ/640?wx_fmt=png)

  3. 新开tab页面http://127.0.0.1:9222/{devtoolsFrontendUrl} 

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGtDG715ePTbZo9W6dQqXqjQ0icSibeAjw0dqAicdiaauYb4MBWzXG8fgNmL7xMKMDibCMQ749KiaA0EUGw/640?wx_fmt=png)

感兴趣的可以在这篇文章 详细了解CDP的原理

####  JSON-RPC在门店业务中的应用场景

在门店里店员可以通过平板来操控设备。那就需要平板与设备与模组之间的通信

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGtDG715ePTbZo9W6dQqXqjzIdc5Spc5na3yL5SIiaPzwmvHGC2wEFktiaZ55nBKUM35z0OU8eSqRzw/640?wx_fmt=png)

###  总结

  * RPC的作用主要是让我们屏蔽网络编程的复杂性，实现远程调用像调用本地方法一样的效果 
  * RPC的实现有多种，可以基于HTTP1.1/HTTP2、Websocket、TCP、UDP任何一种 
  * RPC与HTTP没有可比性，包括任何技术而言都是在某些场景下是相对合适的 
  * JSON-RPC依赖JSON数据标准，兼容性高，大部分语言都支持且有相应的库。主打轻量级，简单，易用。 

##  最后

关注公众号「Goodme前端团队」，获取更多干货实践，欢迎交流分享~

  

预览时标签不可点

微信扫一扫  
关注该公众号





****



****



×  分析

  收藏

