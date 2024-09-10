![cover_image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicGUMjCqrOAnTicAP1aBc2LZUaicC6OnaRkG22DBaFxyD39P7MwBy50sJtrKwDicib5x8yO8BNNNJOfSYg/0?wx_fmt=jpeg)

#  WebSocket协议分析

原创  陈杰  [ Goodme前端团队 ](javascript:void\(0\);)

__ _ _ _ _

> 温馨Tips：阅读本文大约需要 10 分钟。

##  基本介绍

###  维基百科定义

**WebSocket** 是一种网络传输协议， **可在单个 TCP 连接上进行全双工通信** ，位于 OSI 模型的应用层。WebSocket 协议在
2011 年由 IETF 标准化为 **RFC 6455** ，后由 RFC 7936 补充规范。Web IDL 中的 WebSocket API 由
W3C 标准化。

WebSocket 使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在 WebSocket API
中，浏览器和服务器只需要完成一次握手，两者之间就可以创建持久性的连接，并进行双向数据传输。

**名词解释** ：

  * **IETF** : 全称 **I** nternet **E** ngineering **T** ask **F** orce（互联网工程任务组），是一个开放的标准组织，负责开发和推广自愿互联网标准，特别是构成 TCP/IP 协议族的标准。 
  * **Web IDL** : 是一种接口描述语言格式，用于描述要在 Web 浏览器中实现的应用程序编程接口。 

###  浏览器支持

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicHXCTPlO44W3tdU4ia2micax9RmiarpWm5cFZCt9PjBcctGiaA9z1xnxssMQ6iah3oo0dP4OgcH6Jhm7bg/640?wx_fmt=png)

###  使用场景

凡是对 “消息” 推送有较高及时性的场景，都可以使用 WebSocket，例如：客服服务、邮件通知等服务。

###  历史

在浏览器未实现 WebSocket 的时候，想要实现服务端推送，通常会使用 **轮询** 或 **长轮询** 技术实现。

####  轮询

无脑定时发送请求去判断有没有新的“消息”。

####  长轮询

**流程** ：

  1. 浏览器发送请求发送到服务器。 
  2. 服务器在有“消息”之前不会关闭连接。 
  3. 当有新“消息”时 或 挂起超时，服务器将对其请求作出响应。 
  4. 浏览器立即发出一个新的请求。 

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicHXCTPlO44W3tdU4ia2micax9eDvNOS8dGviaT1weNFPIiccIv9QQEQOnrVqHFiaPtl99iafiag2R36wlw3g/640?wx_fmt=png)

**下面是 QQ 邮箱网页版长轮询请求截图** ：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicHXCTPlO44W3tdU4ia2micax9hz7m6iamx72RHV6pCjHMxibnAhvKtZN72Obibsl17T4u3QWAKmiacgicZ0w/640?wx_fmt=png)

###  WebSocket 优点

  * 较少的控制开销 
  * 更强的实时性 
  * 保持连接状态（无需基于类似 cookie 机制去维持会话状态） 
  * 更好的二进制支持 
  * 更好的压缩效果 

##  知识普及

###  二进制

####  基本概念

  * **bit** : 位，二进制的最小单位，1 位表示 1 个 ` 0 ` 或 ` 1 `
  * **byte** : 字节，每 8 位叫做 1 字节（Node buffer 类型： ` Uint8Array ` ，即 buffer 中每个元素代表一个 8 位无符号数字，范围 0~255） 
  * **KB** : 1024 个字节 
  * **MB** : 1024 kb 
  * **有符号** ：二进制的第一位用作符号位（通常是 0 代表正数，1 代表负数），这样实际可表示的数值范围绝对值会减少一半（ ` Int8Array ` 表示 8 位有符号二进制数组，每个元素范围为：-128~127） 
  * **无符号** ：将有符号的第一位也用作数值表示， ` Uint8Array ` 就是一个无符号二进制数组 

####  运算

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicHXCTPlO44W3tdU4ia2micax9M9ZGicpLAmEnXhdIW4xf6aXvicmY0fvxeHXpHqib8f9TjHmkiaXqXPun8g/640?wx_fmt=png)

####  示例

    
    
       1 1 0 0 1 1 0 0  
     & 1 0 0 1 1 0 1 1  
    ————————————————————  
       1 0 0 0 1 0 0 0  
    
    
    
       1 1 0 0 1 1 0 0  
     | 1 0 0 1 1 0 1 1  
    ————————————————————  
       1 1 0 1 1 1 1 1  
    
    
    
       1 1 0 0 1 1 0 0  
     ^ 1 0 0 1 1 0 1 1  
    ————————————————————  
       0 1 0 1 0 1 1 1  
    

**Note** ：用相同的数异或二次等于本身： ` 0b1010 == (0b1010 ^ 0b1100 ^ 0b1100) == 10 `

    
    
     ~ 1 1 0 0 1 1 0 0  
    ————————————————————  
       0 0 1 1 0 0 1 1  
    
    
    
       1 1 0 0 1 1 0 0 << 1  
    —————————————————————————  
       1 0 0 1 1 0 0 0  
    

**Note** ：全部位向左移动一位，高位（左侧）的一个 1 被丢弃，低位（右侧）补充一个 0

    
    
       1 1 0 0 1 1 0 0 >> 1  
    —————————————————————————  
       0 1 1 0 0 1 1 0  
    

####  大小端

  * 大端小端是不同的字节顺序存储方式，统称为 **字节序** ； 
  * **大端模式** ，高位在前，低位在后（和我们的阅读习惯一致） 
  * **小端模式，** 与大端模式相反 

###  Stream

流提供了两个主要优点：

  * **内存效率** : 无需加载大量的数据到内存中即可进行处理。 
  * **时间效率** : 当获得数据之后即可立即开始处理数据，这样所需的时间更少，而不必等到整个数据有效负载可用才开始。 

####  与 buffer 处理文件的内存占用对比

分别使用 buffer 及 stream 两种方式，对一个约为 61M 的文件做 md5 hash 摘要计算，来对比内存占用情况：

#####  Buffer

    
    
    const crypto = require('crypto');  
    const fs = require('fs');  
    const path = require('path');  
    const { performance } = require('perf_hooks');  
    const v8 = require('v8');  
      
    function bufferImpl() {  
      v8.writeHeapSnapshot('buffer.1.heapsnapshot');  
      
      const start = performance.now();  
      const filename = path.join(__dirname, 'test.zip');  
      const buffer = fs.readFileSync(filename);  
      const hash = crypto.createHash('md5').update(buffer).digest('hex');  
      
      v8.writeHeapSnapshot('buffer.2.heapsnapshot');  
      
      console.log('Buffer Impl:');  
      
      console.log('  Hash:', hash);  
      console.log('  Cost:', performance.now() - start);  
    }  
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicHXCTPlO44W3tdU4ia2micax98k2JBNHLcibk7s8SHPagMY1X0sFcxVFYFwRKWjoNwF4kASlaFrvaQQg/640?wx_fmt=png)

#####  Stream

    
    
    const crypto = require('crypto');  
    const fs = require('fs');  
    const path = require('path');  
    const { performance } = require('perf_hooks');  
    const v8 = require('v8');  
      
    function streamImpl() {  
      v8.writeHeapSnapshot('stream.1.heapsnapshot');  
      
      const start = performance.now();  
      const filename = path.join(__dirname, 'test.zip');  
      const stream = fs.createReadStream(filename);  
      const md5 = crypto.createHash('md5').setEncoding('hex');  
      const hash = stream.pipe(md5);  
      
      v8.writeHeapSnapshot('stream.2.heapsnapshot');  
      
      console.log('Stream Impl:');  
      
      process.stdout.write('  Hash: ');  
      hash.pipe(process.stdout);  
      hash.on('end', () => {  
        console.log('\n  Cost:', performance.now() - start);  
      });  
    }  
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicHXCTPlO44W3tdU4ia2micax9LgER8jALxZibkCiaQ3AxImVy2aAqUcWNEnicRqALoTsLzsOJ7BcUU9aWw/640?wx_fmt=png)

####  Node Stream API

######  ` stream.Writable `

**常用方法** ：

  * ` .write(chunk) ` 写入数据片段 

#####  ` stream.Readable `

**常用事件** :

  * ` data ` 接收到数据片段 
  * ` error ` 错误 
  * ` end ` 读取结束 

**常用方法** ：

  * ` .pipe(writeable) ` 通俗来讲，就是把 **“读” 管道** 和 **“写” 管道** 头尾相接，这样数据就可以从一个管道直接 “流” 到另一个管道，方便操作。 

###  最大传输单元（MTU）

泛指通讯协议中的最大传输单元。一般用来说明 TCP/IP 四层协议中数据链路层的最大传输单元，不同类型的网络 MTU 也会不同，我们普遍使用的以太网的
MTU 是 1500，即最大只能传输 1500 字节的数据帧。可以通过 ` ifconfig ` 命令查看电脑各个网卡的 MTU。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicHXCTPlO44W3tdU4ia2micax9zLhG4Pwj9yVG54RnswpMJ1wThhKhQRno2INjQ7GLrTxKzrWOKV6aQQ/640?wx_fmt=png)

###  最大分片大小（MSS）

指 TCP 建立连接后双方约定的可传输的最大 TCP 报文长度，是 TCP 用来限制应用层可发送的最大字节数。如果底层的MTU是1500byte，则 MSS
= 1500 - 20(IP Header) - 20 (TCP Header) = 1460 byte。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicHXCTPlO44W3tdU4ia2micax9b1XYmgJjK5x7SOibPux8yTG2Zgssw3NCBSqRdUYM72FKibOI03zocp9A/640?wx_fmt=png)

图片来源：互联网协议介绍 · 应用层

###  Nagle 算法

**作用** ：减少网络拥塞

**规则** ：

  1. 如果包长度达到 MSS，则允许发送 
  2. 如果包含 FIN，则允许发送 
  3. 如果设置了TCP_NODELAY，则允许发送 
  4. 未设置 TCP_CORK 选项时，若所有发出去的小数据包（包长度小于 MSS ）均被确认，则允许发送 
  5. 上述条件都未满足，但发生了超时（一般为200ms），则立即发送。 

大概意思就是：数据包不是立马发送的，而且要在一定延迟下（200ms）等待下一个包，如果加起来小于 MSS，这将这些包一起发送出去。

###  TCP 粘包和拆包

受 MSS 及 Nagle 算法等因素影响，TCP 在发送时会遇到粘包和拆包的情况：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicHXCTPlO44W3tdU4ia2micax9ibOBIaGhNibVtmhXOVtJ4BhfxQZyPLLTic5dnstibFgr7HQOo7aibjaYx4g/640?wx_fmt=png)

##  协议分析

###  1\. 通过一次 HTTP 握手，完成协议升级 (rfc6455#section-1.2)

首先客户端发送如下 Headers：

    
    
    GET /chat HTTP/1.1  
    Host: server.example.com  
    Origin: http://example.com  
      
    Upgrade: websocket  
    Connection: Upgrade  
    Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==  
    Sec-WebSocket-Protocol: chat, superchat  
    Sec-WebSocket-Version: 13  
    

服务端响应如下 Headers:

    
    
    HTTP/1.1 101 Switching Protocols  
    Upgrade: websocket  
    Connection: Upgrade  
    Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=  
    Sec-WebSocket-Protocol: chat  
    

其中，服务端响应的 ` Sec-WebSocket-Accept ` 字段的值生成算法伪代码如下：

  1. ` let str = Sec-WebSocket-Key + "258EAFA5-E914-47DA-95CA-C5AB0DC85B11" `
  2. ` base64(sha1( str )) `

**备注** : 字符串 ` "258EAFA5-E914-47DA-95CA-C5AB0DC85B11" ` 是协议指定的唯一 GUID

当完成这次握手动作，客户端与服务端就建立了一个长连接，后续两端的消息收发等操作靠解析数据帧来完成。

###  2\. 数据帧解析 (rfc6455#section-5.2)

    
    
       
      0                   1                   2                   3  
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1  
      +-+-+-+-+-------+-+-------------+-------------------------------+  
      |F|R|R|R| opcode|M| Payload len |    Extended payload length    |  
      |I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |  
      |N|V|V|V|       |S|             |   (if payload len==126/127)   |  
      | |1|2|3|       |K|             |                               |  
      +-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +  
      |     Extended payload length continued, if payload len == 127  |  
      + - - - - - - - - - - - - - - - +-------------------------------+  
      |                               |Masking-key, if MASK set to 1  |  
      +-------------------------------+-------------------------------+  
      | Masking-key (continued)       |          Payload Data         |  
      +-------------------------------- - - - - - - - - - - - - - - - +  
      :                     Payload Data continued ...                :  
      + - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +  
      |                     Payload Data continued ...                |  
      +---------------------------------------------------------------+  
    

  * **FIN** : 占 1 位，表示这是消息的最后一个片段，（如果为 1，表示当前已经结束了，如果为0，表示消息分片还没结束） 
  * **RSV1, RSV2, RSV3** : 各占1位，表示扩展协议 (extensions) 
  * **opcode** : 占4位，表示操作码，各操作码含义如下： 

    
    
    -   %x0 : 表示消息帧的延续（一般来说配合 FIN = 0 使用）  
    -   %x1 : 表示传输的内容是文本  
    -   %x2 : 表示传输的内容是二进制  
    -   %x3-7 : 给未来保留的非控制帧的操作吗  
    -   %x8 : 表示连接关闭  
    -   %x9 : 表示一个 ping  
    -   %xA : 表示一个 pong  
    -   %xB-F : 给未来保留的控制帧的操作码  
    

  * **MASK** : 占1位，表示是否使用掩码 
  * **Payload len** : 占7位：表示数据的字节数，该长度值的含义如下： 

    
    
    -   如果值的范围是：0~125，则数据的真实长度就是该值  
    -   如果值为：126，则以无符号整数的方式读取接下来的 16 位，读取结果为数据的真实长度  
    -   如果值为：127，则以无符号整数的方式读取接下来的 64 位，读取结果为数据的真实长度  
    

**备注** ：Payload len 实际占有的位数可能有： **7** (0~125)、 **16** (126)、 **64** (127)

  * **Masking-key** : 如果使用了掩码（ MASK = 1）, 则占 32 位，否则，不占空间 
  * **Payload Data** ：数据，包含2部分： 

    
    
    -   扩展数据（握手时协商了扩展的时候才有，并要明确给出扩展数据的字节数）  
    -   应用数据  
    

###  3\. 一些细节

####  3.1 关于掩码（rfc6455#section-5.3）

掩码用于加密或解密载荷数据，加密和解码的计算方式都如下所示 (nodejs 代码)：

    
    
    /**  
     * 处理控制帧  
     * @param data 数据  
     * @param maskingKey 掩码key（4个字节）  
     */  
    function mask(data: Buffer, maskingKey: Buffer) {  
      for (let i = 0; i < data.length; i++) {  
        data[i] = data[i] ^ maskingKey[i % 4]  
      }  
    }  
    

**协议规定** ：客户端向服务端发送数据是，必须 **使用** 掩码；而服务端向客户端发送数据时，则 **不能使用**
掩码(rfc6455#section-5.1)

####  3.2 关于 ping 和 pong 操作 (rfc6455#section-5.5.2)

ping 和 pong 通常用于检测两端是否处于连接正常的状态（心跳检测）

**协议规定** ：一端向向一端发送 ping 操作帧时，另一端必须发送一个 pong 操作帧作为响应

###  4\. 实现 demo 示例

请参阅：https://github.com/peakchen90/tiny-websocket

##  一些思考

###  HTTP 握手时为啥要使用 ` \r\n ` 作为换行符？

参考：https://www.rfc-editor.org/rfc/rfc2616.html#section-2.2

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicHXCTPlO44W3tdU4ia2micax9NALKK86QKGm9mLLJpBmLqTSibyvCU1wpWmk85XWkGc4uITDoiciaKQnBA/640?wx_fmt=png)

###  发送二进制文件怎么携带发送者的用户信息？

可以实现一个自定义二进制协议。如下所示，二进制的第一个 8 位用来存放用户昵称的字节长度 L（最大支持 256 个字节，使用 UTF-8 编码字符大约能放
64 个中文），其后仅接着占用 L 个字节存放真实昵称使用 UTF-8 编码后的二进制序列，最后部分为真实的二进制文件内容。

    
    
    // 0 0 0 0 0 0 0 0 : 昵称长度  
    // ...             : 昵称位置  
    // ...             : 二进制数据位置  
    const nickBuffer = Buffer.from(sender['nickname']);  
    const headerBuffer = Buffer.allocUnsafe(1 + nickBuffer.length);  
    headerBuffer.writeUInt8(nickBuffer.length, 0);  
    headerBuffer.set(nickBuffer, 1);  
      
    wss.broadcast([headerBuffer, message], true);  
    

###  WebSocket 如何处理大文件？

大文件的内存占用对于服务端来说是致命的。客户端可以使用 Blob, 服务端只做消息转发的功能，不要在服务端组合分片内容完成后再下发。

###  IP 协议职责是？TCP 协议的职责是？

**IP** 用于找到主机， **TCP** 用于找到主机的程序（根据 port）

##  参考链接

  * **中英文双语对照版** ：RFC 6455: The WebSocket Protocol 中文翻译 
  * **英文版协议** : RFC 6455: The WebSocket Protocol 
  * **WebSocket：5分钟从入门到精通** ：https://juejin.cn/post/6844903544978407431 
  * **ws: a Node.js WebSocket library** : https://github.com/websockets/ws 
  * **本文 demo** : https://github.com/peakchen90/tiny-websocket 

##  最后

关注公众号「Goodme前端团队」，获取更多干货实践，欢迎交流分享~

  

预览时标签不可点

微信扫一扫  
关注该公众号





****



****



×  分析

  收藏

