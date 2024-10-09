#### 三元运算
```javascript
错误形式: val ? return 1 ? return 0;

正确形式: return val ? 1 : 0;
```

```javascript
//正常普通从左到右形
false?'true':true?'t':'f'   a>b?0:(c>d?1:2)


//呈洋葱形（true?(false?'a':'b'):'c'）
true?false?'a':'b':'c'//b   
true?true?'a':'b':'c'//a
false?true?'a':'b':'c'//c
false?false?'a':'b':'c'//c
//第一次判断分成两部分a:b和c，如果第一个为false不管第二个是啥结果都为c
```

#### 协商缓存和强缓存
目的：<font>  
</font><font>○ 减少http请求次数  
</font><font>○ 节省带宽，避免网络资源的浪费，减轻服务器的压力  
</font><font>○ 提升网页的加载速度，优化用户体验</font>

##### 1、强缓存：
<font>当服务端响应头</font>**<font> Cache-Control：max-age=1200</font>**<font>（即缓存有效时间为1200s）。这会让浏览器</font>**<font>自动</font>**<font>将该请求的资源缓存到本地。下一次请求该资源时，浏览器先看本地缓存的资源有没有过期，没过期的话直接使用该资源，</font>**<font>不发送请求</font>**<font>，且返回 Status Code:200 OK，但是会添加上（</font>**<font>from memory cache</font>**<font> 或 </font>**<font>from disk memory</font>**<font>）的标识，表示该文件是从缓存中拿到的，没有向服务端发送请求。</font>

<font>关于浏览器缓存机制，首先由三个参数，早期expires设置缓存时间，pragma唯一值no-cache，带上之后即使设置expires也不会缓存，cache-control一般就用max-age，用于设置缓存有效期。</font>

##### 2、协商缓存
**协商缓存是在强制缓存失效之后，需要重新对比缓存，由服务器决定是否失效的一种机制。**

1、向服务器发送请求，服务器会根据请求头的标识判断是否命中协商缓存

2、如果命中，就会返回304状态码，通知浏览器从缓存中读取数据

**3、异同：**

共同点：都是从浏览器读取资源

不同点：1、强缓存不发请求给服务器；2、协商缓存发送请求给服务器，根据服务器返回的信息决定是否使用缓存。

##### <font style="color:#DF2A3F;">3、整体流程：</font>
两组四个字段进行判断，<font style="background-color:#FBDE28;">If-None-Match</font>和<font style="background-color:#FBDE28;">ETag</font>（一组）， <font style="background-color:#FBDE28;">If-Modified-Since</font> 字段与 <font style="background-color:#FBDE28;">Last-Modified</font> 字段（一组），前者优先级比后者高，同时出现会出现“短路效应”。

<font>如果第一次请求服务器设置了Last-Modified响应头，第二次请求的时候，客户端会自动带上If-Modified-Since请求头，</font>

<font>如果第一次请求服务器设置了ETag响应头，第二次请求的时候，客户端会自动携带If-None-Match请求头，</font>

**<font style="color:#DF2A3F;">协商缓存过程1：(文件hash值，服务端返回Etag)    </font>**

1. 浏览器请求资源，服务器返回报文中加入Etag值，资源更新则Etag值也会更新。
2. 浏览器再次请求资源，此时请求报文会加入If-None-Match，值为上一次响应报文的Etag值。
3. 服务器比对报文的If-None-Match和当前的Etag是否一致，不一致则更新Etag并且返回，如果一致表示资源没有更新，状态码返回304，浏览器从本地缓存获取，此时响应头会同时返回Etag值。（虽然没有变化）

**<font style="color:#DF2A3F;">协商缓存过程2：(最后修改时间，服务端返回Last-Modified)</font>**

1. 首次发送请求时，服务端返回<font style="background-color:#FBDE28;">Last-Modified</font> 字段，表示资源最后一次修改时间。
2. 等浏览器再次发送请求时，会在请求头中加入<font style="background-color:#FBDE28;">If-Modified-Since</font> 字段，就是上次返回的<font style="background-color:#FBDE28;">Last-Modified</font> 字段值
3. 服务器判断如果相等，代表资源没变，返回304，

##### <font>f5刷新会暂时禁止强缓存，还会有协商缓存，ctrl+f5会禁止所有缓存策略。</font>
#### 空值合并运算符
let c=a ?? b  当a为null/undefined时c=b

let c=a || b    当a为false/0/""/null/undefined时c=b

综上所述，？？判断第一个值是不是空值，||判断第一个参数是不是假值，而且运算优先程度很低，需要加上括号

#### 阴历转农历
```javascript
new Date().toLocaleString('zh-u-ca-chinese', {
		weekday: "long",
		year: "numeric",
		month: "numeric",
		day: "2-digit",
		hour: "numeric",
		minute: "numeric",
		second: "numeric",
		hour12: false,
})
2022.9.2  15：54:30
=> 2022壬寅年八月07，星期五 15:54:30
```

#### toLocaleString
```javascript
Array.prototype.toLocaleString([locales[,options]])  
Number.prototype.toLocaleString([locales[,options]])
Date.prototype.toLocaleString([locales[,options]]) 

// 例如
new Date().toLocaleString()   //'2022/9/2 15:59:30'
123456789.toLocaleString()    //123,456,789 默认千分位分割

```

 locales

+ 参数必须是一个 BCP 47 语言标记的字符串，或者是一个包括多个语言标记的数组。如果 locales 参数未提供或者是 undefined，便会使用运行时默认的 locale。[Intl - JavaScript | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Intl)

options

| <font>style字段值</font> | <font>说明</font> |
| :--- | :--- |
| **<font>decimal</font>** | <font>用于纯数字格式（默认）</font> |
| **<font>currency</font>** | <font>用于货币格式</font> |
| **<font>percent</font>** | <font>用于百分比格式</font> |
| **<font>unit</font>** | <font>用于单位格式</font> |


| <font>currency字段值</font> | <font>说明</font> |
| :--- | :--- |
| **<font>USD</font>** | <font>使用美元格式（默认）</font> |
| **<font>EUR</font>** | <font>使用欧元格式</font> |
| **<font>CNY</font>** | <font>使用人民币格式</font> |


#### json.stringify（）实现深拷贝的几个坑
```javascript
const obj = {
    nan:NaN, //变为null
    infinityMax:1.7976931348623157E+10308,//变为null
    infinityMin:-1.7976931348623157E+10308,//变为null
    undef: undefined,//丢失
    fun: () => { console.log('叽里呱啦，阿巴阿巴') },//丢失
    date:new Date,//转为字符串类型
}
```

+ <font style="color:rgb(89, 89, 89);">对象中有时间类型的时候，序列化之后会变成字符串类型。</font>
+ <font style="color:rgb(89, 89, 89);">对象中有undefined和Function、</font>symbol值<font style="color:rgb(89, 89, 89);">类型数据的时候，序列化之后会直接丢失。</font>
+ <font style="color:rgb(89, 89, 89);">对象中有NaN、Infinity和-Infinity的时候，序列化之后会显示null。</font>
+ <font style="color:rgb(89, 89, 89);">对象循环引用的时候，会直接报错。</font>

#### 跳出foreach循环
```javascript
try{
	arr.foreach(item=>{
		xxx
		if(xxx) throw new error('break')   //关键一手，抛出错误
	})
}catch(error){
	  if(err.message === "break")	
        console.log("break success!") //接住错误
    else 
        console.error(err)
}


```

#### 获取当前选中的文字
```javascript
window.getSelection().toString()
```

#### 多维数组平铺成一维数组
```javascript
let arr=[1,2,3,[4,5,[6,7,[8,9,10]]]];
arr.flat(Infinity)  //=>[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

#### CHROME浏览器发送HTTP最大请求并发数限制
1. 同一get请求：同一域名下并发数为1，最大等待时间是20s，超过20s允许并发
2. post请求/不同GET请求：同一域名下最大并发数为6，前一批6个请求完毕后执行下一组

#### git flow


#### canvas、图形学、svg
#### 浏览器同源策略的原因
1. <font>dom同源策略</font>

<font>如果没有 DOM 同源策略，也就是说</font><font style="color:#E8323C;">不同域的 iframe 之间可以相互访问</font><font>，那么黑客可以这样进行攻击： 做一个假网站，里面用 iframe 嵌套一个银行网站 </font>mybank.com。<font> 把 iframe 宽高啥的调整到页面全部，这样用户进来除了域名，别的部分和银行的网站没有任何差别。 这时如果用户输入账号密码，我们的主网站可以跨域访问到 </font>mybank.com<font> 的 dom 节点，就可以拿到用户的账户密码了。 </font>

2. <font>XMLHttpRequest 同源策略</font>

<font>如果没有 XMLHttpRequest 同源策略，那么黑客可以进行 CSRF（跨站请求伪造） 攻击： 用户登录了自己的银行页面 </font>mybank.com，http://mybank.com<font> 向用户的 cookie 中添加用户标识。 用户浏览了恶意页面 </font>evil.com<font>, 执行了页面中的恶意 AJAX 请求代码。</font><font style="color:#E8323C;"> </font><font style="color:#E8323C;">evil.com</font><font style="color:#E8323C;"> 向 </font><font style="color:#E8323C;">mybank.com</font><font style="color:#E8323C;"> 发起 AJAX HTTP 请求</font><font>，请求会默认把 </font>mybank.com<font> 对应 cookie 也同时发送过去。 银行页面从发送的 cookie 中提取用户标识，验证用户无误，response 中返回请求数据。此时数据就泄露了。 而且由于 Ajax 在后台执行，用户无法感知这一过程。 </font>

#### <font>js是单线程的原因</font>
1. <font>作为浏览器脚本语言，JavaScript的主要用途是与用户互动，以及操作DOM。这决定了它只能是单程，否则会带来很复杂的同步问题。比如，假定JavaScript同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？所以，为了避免复杂性，从一诞生，JavaScript就是单线程，</font>
2. <font style="color:rgb(18, 18, 18);">简单的比喻：进程=火车，线程=车厢</font>
+ <font style="color:rgb(18, 18, 18);">线程在进程下行进（单纯的车厢无法运行）</font>
+ <font style="color:rgb(18, 18, 18);">一个进程可以包含多个线程（一辆火车可以有多个车厢）</font>
+ <font style="color:rgb(18, 18, 18);">不同进程间数据很难共享（一辆火车上的乘客很难换到另外一辆火车，比如站点换乘）</font>
+ <font style="color:rgb(18, 18, 18);">同一进程下不同线程间数据很易共享（A车厢换到B车厢很容易）</font>
+ <font style="color:rgb(18, 18, 18);">进程要比线程消耗更多的计算机资源（采用多列火车相比多个车厢更耗资源）</font>
+ <font style="color:rgb(18, 18, 18);">进程间不会相互影响，一个线程挂掉将导致整个进程挂掉（一列火车不会影响到另外一列火车，但是如果一列火车上中间的一节车厢着火了，将影响到所有车厢）</font>
+ <font style="color:rgb(18, 18, 18);">进程可以拓展到多机，进程最多适合多核（不同火车可以开在多个轨道上，同一火车的车厢不能在行进的不同的轨道上）</font>
+ <font style="color:rgb(18, 18, 18);">进程使用的内存地址可以上锁，即一个线程使用某些共享内存时，其他线程必须等它结束，才能使用这一块内存。（比如火车上的洗手间）－"互斥锁"</font>
+ <font style="color:rgb(18, 18, 18);">进程使用的内存地址可以限定使用量（比如火车上的餐厅，最多只允许多少人进入，如果满了需要在门口等，等有人出来了才能进去）－“信号量”</font>

#### <font>Dom常见操作回顾  --》</font><font>创建节点，查询节点，更新节点，添加节点，删除节点</font>
#### 对BOM的理解
1. <font> (Browser Object Model)，浏览器对象模型，提供了独立于内容与浏览器窗口进行交互的对象</font>  
<font>其作用就是</font><font style="color:#E8323C;">跟浏览器做一些交互</font><font>效果,比如如何进行页面的后退，前进，刷新，浏览器的窗口发生变化，滚动条的滚动，以及获取客的一些信息如：浏览器品牌版本，屏幕分辨率</font>  
<font>		浏览器的全部内容可以看成</font><font style="color:rgb(255, 80, 44);background-color:rgb(255, 245, 245);">DOM</font><font>，整个浏览器可以看成</font><font style="color:rgb(255, 80, 44);background-color:rgb(255, 245, 245);">BOM</font><font>。</font>
2. <font>核心对象是</font><font style="color:rgb(255, 80, 44);background-color:rgb(255, 245, 245);">window</font><font>，它表示浏览器的一个实例</font>  
<font>在浏览器中，</font><font style="color:rgb(255, 80, 44);background-color:rgb(255, 245, 245);">window</font><font>对象有双重角色，即是浏览器窗口的一个接口，又是全局对象</font>
3. [说说你对BOM的理解，以及常见的BOM对象有哪些？- 题目详情 - 前端面试题宝典](https://fe.ecool.fun/topic/483e37bb-b761-4231-82e8-4f6345a6df78?orderBy=updateTime&order=desc&titleKey=bom)

#### throttle(油门、节流阀)和debounce(防抖)
bounce弹起，debounce就是一直不让它弹起，例如resize这个事件，一直在变化难道要一直触发fun吗，肯定是等最后不动了之后才去执行fun，所以防抖应运而生。还有异步搜索

```javascript
function debounce(func, duration) {
  let timeout

  return function (...args) {
    const effect = () => {
      timeout = null
      return func.apply(this, args)
    }

    clearTimeout(timeout)
    timeout = setTimeout(effect, duration)
  }
}

button.addEventListener(
  'click',
  debounce(function () {
    throwBall()
  }, 500)
)
```

throttle节流阀、油门，就像用油门驱动弹簧弹球一样，在弹出去之后需要一定时间收缩，在这期间不能执行扔球的操作。例如一般的button，点击就能执行fun，但是执行如果有时间跨度，就必须让这时间段中的点击不再去执行，要不然得执行多少个fun啊

```javascript
function throttle(func, duration) {
  let shouldWait = false

  return function (...args) {
    if (!shouldWait) {
      func.apply(this, args)
      shouldWait = true

      setTimeout(function () {
        shouldWait = false
      }, duration)
    }
  }
}

button.addEventListener(
  'click',
  throttle(function () {
    throwBall()
  }, 500)
)
```

vueUse使用

```vue
<script>
	import { useDebounceFn } from "@vueuse/core";
  
  const screenWidth = ref(0);
  const listeningWindow = useDebounceFn(() => {
    // 获取浏览器宽度
    screenWidth.value = document.body.clientWidth;
    // 设置是否折叠，如果小于1200并且没折叠，就折叠起来
    if (!isCollapse.value && screenWidth.value < 1200) globalStore.setGlobalState("isCollapse", true);
    if (isCollapse.value && screenWidth.value > 1200) globalStore.setGlobalState("isCollapse", false);
  }, 100);
  window.addEventListener("resize", listeningWindow, false);
  onBeforeUnmount(() => {
    window.removeEventListener("resize", listeningWindow);
  });
</script>
```

#### 小细节	
1. 事件委托中的e.target是触发事件的元素，e.currentTarget是绑定事件的元素
2. == 机制：<font style="color:rgb(255, 80, 44);background-color:rgb(255, 245, 245);">undefined == null</font><font>，结果是true。且它俩与所有其他值比较的结果都是false。 </font><font style="color:rgb(255, 80, 44);background-color:rgb(255, 245, 245);">String == Boolean</font><font>，需要两个操作数同时转为Number。 </font><font style="color:rgb(255, 80, 44);background-color:rgb(255, 245, 245);">String/Boolean == Number</font><font>，需要String/Boolean转为Number。 </font><font style="color:rgb(255, 80, 44);background-color:rgb(255, 245, 245);">Object == Primitive</font><font>，需要Object转为Primitive(具体通过valueOf和toString方法)。</font>
3. <font>声明未赋值为undefined，未声明为ReferenceError</font>
4. <font style="color:#E8323C;">Cookie过期时间设置为0</font>，表示跟随系统默认，其销毁与Session销毁时间相同，即都在浏览器关闭后的特定时间删除。
5. onmouseenter、onmouseleave没有事件冒泡，也就是本身触发完之后不会触发父组件，onmouseover、onmouseout有事件冒泡，也就是本身触发完之后会触发父组件的该事件
6. qs.parse()是将URL解析成对象的形式     qs.stringify()是将对象序列化成URL的形式，以&进行拼接
7. type of null  =>Object。在 JS 的最初版本中使用的是 32 位系统，为了性能考虑使用低位存储变量的类型信息，000 开头代表是对象然而 null 表示为全零，所以将它错误的判断为 object 。
8. <font> 标记用户与跟踪用户行为的情况推荐使用cookie 。适合长期保存在本地的数据（令牌）、用来跨页面传递参数推荐使用localStorage。 敏感账号一次性登录、保存一些临时的数据，防止用户刷新页面之后丢失了一些参数推荐使用sessionStorage 。存储大量数据的情况、在线文档（富文本编辑器）保存编辑历史的情况，推荐使用indexedDB</font>
9. <font>slice:不改变原数组、两个参数分别是开始index和结束index，第二个参数不写默认直接到最后一项。</font>splice:改变原数组、三个参数分别是startIndex、length、insertData，如果length为0，第三个参数插入原数组中，所以可以用slice浅克隆一个数组let brr = arr.slice(0)
10. 通过修改<font style="color:#E8323C;">url中的hash</font>值自动定位到id相符元素的具体位置

```javascript
 setTimeout(() => {
    window.location.hash = '#abc' // 修改url中的hash值
  }, 2000)

window.addEventListener('hashchange', function (e) {
        console.log(e.oldURL); // http://127.0.0.1:5500/index.html
        console.log(e.newURL)  // http://127.0.0.1:5500/index.html#abc
}, false);
```

11. 给对象的原型上添加了一个属性，会被for in 遍历出来，for in 会遍历数组，但是会改变顺序
12. sort 排序字符串按照的是编码顺序，并非是首字母的先后顺序，如何修改成按照字典顺序呢

```javascript
const name=['北京','上海','广西'];
name.sort((a,b)=>a.localeCompare(b)) //['北京', '广西', '上海']
```

13. async await 的使用中搭配try catch使用，便于错误排查，await后面可以支持promise，也可以支持普通function，但是promise避免使用.then().catch，而是使用trycatch，

```javascript
async function getData() {
  try {
    const response = await fetch('https://example.com/data'); // fetch是promise
    const data = await response.json();
    return data;
  } catch (error) {
    console.error(error);
  }
}
```

14. **<font style="color:#DF2A3F;">计算1px用多少物理像素进行展示：</font>**

电脑上的设置屏幕分辨率（实际是改变逻辑分辨率，因为屏幕分辨率是设备本身长宽像素点的多少，无法被修改）就是改变<font style="color:#DF2A3F;">DPR</font>（<font style="color:rgb(77, 77, 77);">devicePixelRatio</font>），即1csspx用多少物理像素来代表，比如电脑本身的屏幕分辨率是3072*1920，设置显示器屏幕分辨率为1792*1120，那么DPR就是3072/1792，即1.71。那么1px就用1.71个像素进行显示。此时只要知道屏幕中1个像素点的大小，就知道1px的大小了。获取屏幕DPR：安卓、windows（<font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">window.devicePixelRatio</font>）

15. **<font style="color:#DF2A3F;">计算1个像素点的大小（cm）</font>**

因为没有直接给出，一般厂商会给出设备尺寸和屏幕分辨率，尺寸代表着设备对角线的长度，屏幕分辨率就是长宽像素点的多少，那么1英寸长度包含的像素点的多少就是对角线像素数/对角线长度，就算出来1个像素点有多大了，例如屏幕分辨率为3072*1920，尺寸为16英寸，![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1681440423779-cbaa8e07-a2e8-40b3-ac74-dfdd039dbc93.png)，于是1英寸长度大约有226.4个像素点。这就是<font style="color:#DF2A3F;">PPI</font>（**<font style="color:rgb(77, 77, 77);">pixels per inch</font>**）1英寸=2.54厘米，于是1个像素点大小=2.54/226.4≈0.011cm

16. splice(0,1)表示删除数组的第一项，如果splice中第一个参数是变量，传了一个undefined值，那么就会变成<font style="color:#DF2A3F;">splice(undefined,1)</font>，结果就是默认删除第一项
17. 127.0.0.1和0.0.0.0的区别：
    1. 127.0.0.1代表着回环地址，表示访问本机
    2. 0.0.0.0是一种通配符，表示接收本机所有网卡的请求，例如tomcat启动时配置地址为192.168.1.103，作为当前电脑的一个网卡，如果请求打到本机的其他网卡上，接收不到。如果使用的是0.0.0.0，那么只要是发送到本机的请求，都会被接收到。

#### 一图弄懂scrollTop、scrollHeight、clientHeight、offsetHeight等
<font style="color:rgb(0, 0, 0);"></font>

![](https://cdn.nlark.com/yuque/0/2022/png/28792475/1670378677241-72b1813c-3c48-4c1f-8c0f-179049d57763.png)

<font>clientHeight 包含padding，</font>

<font>offsetHeight</font>

#### 监听复制事件
```javascript
document.addEventListener('copy',(e)=>{
		e.preventDefault(); //禁止复制
		e.clipboardData.setData('text/plain','不准复制') //向剪切板中添加字符，也就是默认复制的内容
})
```

