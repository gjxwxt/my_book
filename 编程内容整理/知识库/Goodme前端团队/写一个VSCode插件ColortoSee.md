![cover_image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicES2HSTC6iboU6ribN9Mia8Z9IpR7FG9EEXleAJ5Zm3OO3HyZHOCD1ucOyGpiaN28TyyClkJvNibGdhKUw/0?wx_fmt=jpeg)

#  写一个VS Code 插件：Color to See

原创  兰燕平  [ Goodme前端团队 ](javascript:void\(0\);)

__ _ _ _ _

##  背景

在进行项目开发的时候，可能会遇到“想找某个色值”的场景，因为颜色值一般是数字类型，没有语义，不好全局搜索。所以就希望有一个工具，能够快速展示工作目录下的所有颜色，方便取值。

由于是在编写代码阶段，所以这个工具最方便的其实是编辑器的插件。我使用VSCode插件，所以先来试试开发一个VSCode插件。

在正式开发之前，先来给这个插件取个好名字，color, see, 一想到这两个单词，我脑海里直接蹦出了“给你点颜色看看”的中文式表达”give you
color to see”，看起来Color to See 是个好名字。

##  需求分析

在开发之前，先来想想我们的”产品“长什么样子，要实现什么功能。

插件要实现的目标是能够快速找到某个色值，所以我们要将颜色可视化，色值是最终我们希望能拿到的，所以需要将项目中所有的颜色收集起来，统一展示（包含可视化、色值）。

具体功能清单如下：

  * 收集工作区下的所有颜色 

    * 文件更新（删除、新建、文本更新）了能够获取到最新的颜色 
  * 可视化所有颜色块 

    * 以网格状的形式展示颜色色块 
  * 色值展示和获取 

    * 点击颜色展示对应的色值，支持Hex和RGB格式 
    * 点击色值一键复制 

UI设计如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicES2HSTC6iboU6ribN9Mia8Z9IaiawZy7aWaVtvQmVvOdmFuibcmc394nLuDoFFXJNutv2MkSiaabezVwqw/640?wx_fmt=png&from=appmsg)

##  功能实现

###  VSCode WebView

` WebView ` 可以用来创建自定义的视图，可以看成是VSCode内部的 ` Iframe ` 容器，可以渲染任何HTML,
通过消息传递实现通信，所以具备非常强大的页面渲染和交互能力。

VSCode内部提供了三个API可以创建 ` WebView ` :

####  ` window.createWebviewPanel `

创建一个编辑器面板。即时的，关闭编辑器就销毁了，主要用于那些不需要持久化状态（在关闭后不需要恢复）的场景，示例代码如下：

    
    
    const panel = vscode.window.createWebviewPanel(  
        'webviewId', // Webview 的标识符  
        'My Webview', // 面板标题  
        vscode.ViewColumn.One, // 面板显示在哪个编辑器列中  
        { enableScripts: true } // 额外的 Webview 选项  
    );  
      
    panel.webview.html = "<html><body><h1>Hello, Webview!</h1></body></html>";  
    

####  ` window.registerWebviewPanelSerializer `

定义如何序列化和反序列化 WebView 面板的状态，这样即使在 VSCode 重启后，这个 WebView
面板的状态也可以被恢复。所以这种WebView，对于那些需要保留用户状态或信息的面板来说非常有用。

    
    
    class MyWebviewPanelSerializer implements vscode.WebviewPanelSerializer {  
        async deserializeWebviewPanel(webviewPanel: vscode.WebviewPanel, state: any) {  
            // 重新设置 Webview 的内容等  
            webviewPanel.webview.html = "<html><body><h1>Restored Webview</h1></body></html>";  
        }  
    }  
      
    context.subscriptions.push(vscode.window.registerWebviewPanelSerializer('webviewId', new MyWebviewPanelSerializer()));  
    

####  ` window.registerWebviewViewProvider `

创建一个编辑器面板。持久的视图，常驻在侧边栏（Sidebar）和面板(Panel)上

    
    
    class MyWebviewProvider implements vscode.WebviewViewProvider {  
        resolveWebviewView(webviewView: vscode.WebviewView) {  
            webviewView.webview.html = "<html><body><h1>Hello, Sidebar Webview!</h1></body></html>";  
        }  
    }  
      
    const provider = new MyWebviewProvider();  
    context.subscriptions.push(vscode.window.registerWebviewViewProvider('myWebview', provider));  
    

考虑我们插件的使用场景一般是没有UI稿的后台开发，使用次数不会很频繁，所以插件的视图不需要持久，所以可以选择用 ` createWebviewPanel `
创建一个编辑器面板来承载界面。

###  项目初始化

通过官方文档给的脚手架命令，开始初始化我们的插件项目。目录结构如下：

    
    
    ├─ .eslintrc.json  
    ├─ .gitignore  
    ├─ .vscode  
    │  ├─ extensions.json  
    │  ├─ launch.json  
    │  ├─ settings.json  
    │  └─ tasks.json  
    ├─ .vscode-test.mjs  
    ├─ .vscodeignore  
    ├─ .yarnrc  
    ├─ CHANGELOG.md  
    ├─ README.md  
    ├─ package.json  
    ├─ src  
    │  ├─ extension.ts  
    │  ├─ test  
    │  │  └─ extension.test.ts  
    ├─ tsconfig.json  
    ├─ vsc-extension-quickstart.md  
    ├─ webpack.config.js  
    ├─ yarn-error.log  
    └─ yarn.lock  
    

在 ` package.json ` 文件中，可以看到 ` main ` 配置是 ` dist ` 文件夹下的 ` extension.js `
文件，这是我们项目的入口文件，是Webpack把 ` src/extentions.ts ` 作为入口文件编译过来的

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicES2HSTC6iboU6ribN9Mia8Z9IBbp6QVYaJgEuIFEFGSbBnVSwPHkWOF3YL1fPGUjt4fzFXvIEAh6oKA/640?wx_fmt=png&from=appmsg)

看 ` package.json ` 的 ` script ` 配置，发现可以运行 ` yarn watch `
命令进行热更新，也就是实时把改动的代码编译成可执行的JS文件。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicES2HSTC6iboU6ribN9Mia8Z9IZosHiaYeb2zuhfTbD2SE8A6Zof467NxP2uiaTzMFD8ownoFUDibV61xhQ/640?wx_fmt=png&from=appmsg)

运行完 ` yarn watch ` 后，通过按“ ` F5 `
”或者“运行->启动调试”，运行我们的插件，这时候VSCode会打开一个新的窗口，快捷键ctrl + shift + p输入hello world

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicES2HSTC6iboU6ribN9Mia8Z9IJXN6z3F9ibfomvN0XYn9O0J2Cm9K0Kn4AibBOmNjPia7IyqNMvXblQibnQ/640?wx_fmt=png&from=appmsg)

最后，点击执行这个命令，可以看到右下角弹出了欢迎标语。如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicES2HSTC6iboU6ribN9Mia8Z9IsHeKE98IwchQ9Ax5iaiaiawfpju94SMKWgXhicDQK5icGKOQDTLw7Z90yvg/640?wx_fmt=jpeg)

如果修改了代码，可以F5刷新debug窗口，也可以在debug窗口上使用快捷键command + R刷新，和在浏览器刷新网页一样。

以上就是 VSCode 插件开发起步的过程。

###  接入WebView

####  新增 ` colorToSee ` 命令

在 ` package.json ` 文件的 ` commands ` 属性上新增 ` extension.colorToSee `
，这个命令是使用我们插件的起点，所以务必写个好 ` title ` ，为了让插件国际化，我的 ` title ` 设置为 ` "ColorToSee:
Show colors of the working directory in a webview panel" `

    
    
     "commands": [  
        {  
          "command": "extension.colorToSee",  
          "title": "ColorToSee: Show colors of the working directory in a webview panel"  
        }  
      ],  
    

这个命令的执行回调函数在 ` extension.ts ` 文件上，重点代码如下：

    
    
     vscode.commands.registerCommand(COMMAND_NAME, () => {  
      registerWebviewViewProvider(context);  
    })  
    

在这串代码中，我们把具体的实现逻辑封装在 ` registerWebviewViewProvider ` 方法上，接下来我们可以只关注这个方法的实现。

####  创建WebView来显示自定义UI

使用WebView API ` createWebviewPanel ` 来创建一个自定义的HTML页面，基本的代码块如下所示：

    
    
    const panel = vscode.window.createWebviewPanel(  
      CatCodiconsPanel.viewType,  
      "Cat Codicons",  
      column || vscode.ViewColumn.One  
     );  
      
    panel.webview.html = _getHtmlForWebview(panel.webview, extensionUri);  
      
    function _getHtmlForWebview(webview: vscode.Webview) {  
        // Use a nonce to only allow specific scripts to be run  
        const nonce = getNonce();  
      
        return `<!DOCTYPE html>  
                <html lang="en">  
                <head>  
                    <meta charset="UTF-8">  
                    <!-- Use a content security policy to only allow loading images from https or from our extension directory, and only allow scripts that have a specific nonce. -->  
                    <meta http-equiv="Content-Security-Policy" content="default-src 'none'; style-src ${webview.cspSource}; script-src 'nonce-${nonce}';">  
                    <meta name="viewport" content="width=device-width, initial-scale=1.0">  
                    <title>Your View</title>  
                </head>  
                <body>  
                    <h1>Hello from Your View!</h1>  
                </body>  
                </html>`;  
    }  
      
    function getNonce() {  
        let text = '';  
        const possible = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';  
        for (let i = 0; i < 32; i++) {  
            text += possible.charAt(Math.floor(Math.random() * possible.length));  
        }  
        return text;  
    }  
    

在 ` registerWebviewViewProvider ` 方法中，我们将基于上述代码做些改造，大致的框架如下：

    
    
    const registerWebviewViewProvider = (context: vscode.ExtensionContext) => {  
      const provider = new ViewProvider(context.extensionUri, config);  
      
      const panel = vscode.window.createWebviewPanel(  
        ViewProvider.viewType, // Webview 的标识符  
        PANEL_TITLE, // 面板标题  
        vscode.ViewColumn.One, // 面板显示在哪个编辑器列中  
        {  
          enableScripts: true  
        } // 额外的 Webview 选项  
      );  
      
      provider.resolveWebviewView(panel as unknown as vscode.WebviewView);  
      
      context.subscriptions.push(panel);  
    };  
    

在这段代码中，我们把页面信息维护在 ` ViewProvider ` 这个类上。在这个类上，我们要实现页面的渲染和更新。

‍
![](https://mmbiz.qpic.cn/mmbiz_svg/SCug0ESSOHic3xBs1kGVXrKviallSkffLEJmK3DHJtknPqMd09HWYz2BDKGBG7WenbnWf2MuVcqWAql6FbvAqenqnEIfBgMLZib/640?wx_fmt=svg&from=appmsg)
‍

###  页面渲染

页面的渲染机制可以借用React 框架的 render(state)
思想，这个思想基于函数式编程的原则，其核心是将UI视为状态的函数，UI的每一次更新可以看作是一个状态转换的结果。

本插件的页面构成比较简单，为了方便，可以直接使用原生JS和HTML开发。页面的模块主要分为以下4个部分：

  * 刷新按钮 
  * 颜色网格 
  * 颜色值展示 
  * toast弹窗提醒（消息提醒） 

整个数据驱动式的页面渲染如下所示：

    
    
    <body>  
      ${generateMainDiv(this.colorInfos)}  
    </body>  
    

####  定义状态

首先，我们定义 ` colorInfos ` 状态，这个变量存储了当前工作区所有颜色信息，包括该颜色的所在文件的位置和色值，TS定义如下所示：

    
    
    export type ColorItem = {  
      /** 颜色值起始位置 */  
      start: number;  
      /** 颜色值结束位置 */  
      end: number;  
      /** 颜色值 */  
      color: string;  
      /** 颜色值所在的文件路径 */  
      file: string;  
    };  
    

####  渲染函数

”颜色网格“是我们整个页面主要的功能模块，所以我们重点介绍这个功能的渲染函数，根据 ` colorInfos `
，一个能根据这个状态生成HTML的函数可以简化成这样：

    
    
    function generateMainDiv(colors) {  
      return colors.map(info => `<div style="color: ${info.color}" data-colorItem="${encodeURIComponent(  
        JSON.stringify(item)  
      )}">${info.color}</div>`).join('');  
    }  
    

其中自定义属性 ` data-colorItem ` 存储了整个颜色块的所有信息。

####  页面挂载

把 ` html ` 元素赋值给 ` WebView ` 的 ` html ` 属性，就可以实现页面的挂载。

    
    
    this._view.webview.html = this._getHtmlForWebview(this._view.webview);  
      
    function _getHtmlForWebview(webview: vscode.Webview) {  
        // Use a nonce to only allow specific scripts to be run  
        const nonce = getNonce();  
      
        return `<!DOCTYPE html>  
                <html lang="en">  
                <head>  
                    <meta charset="UTF-8">  
                    <!-- Use a content security policy to only allow loading images from https or from our extension directory, and only allow scripts that have a specific nonce. -->  
                    <meta http-equiv="Content-Security-Policy" content="default-src 'none'; style-src ${webview.cspSource}; script-src 'nonce-${nonce}';">  
                    <meta name="viewport" content="width=device-width, initial-scale=1.0">  
                    <title>Your View</title>  
                </head>  
                <body>  
                    ${generateMainDiv(this.colorInfos)}  
                </body>  
                </html>`;  
    }  
    

###  如何构造状态 ` colorInfos ` ?

上面描述的数据驱动式框架比较简单，还没有涉及到本项目的难点实现，如果构造 ` colorInfos ` 数据，是本项目的难点之一，本节主要讲这块内容。

####  颜色格式分析

在项目开发中，比较常见的颜色格式分为RGB, Hex, HSL和颜色关键字（Named Colors）这几种。下面我们来分析这几种颜色在CSS中的语法。

从MDN官网发现 ` rgb() ` 的写法分为绝对和相对格式，我们平时熟悉的写法是这样的： ` rgb(x, y, z) `
，其中x、y、z是从0到255的整数，属于绝对格式的一种。

为了让项目能尽快先落地，我们此处只分析“绝对格式”的场景。绝对格式的写法如下：

    
    
    rgb(R G B[ / A])  
    

其中 ` R ` ` G ` ` B `
的值可以为0-255的整数，或者0%-100%的百分比；A的值为0到1（或者0%-100%），表示颜色的透明度信息。如果有A值时，需要使用 ` / ` 。

我们经常见到的一种写法是 ` R ` ` G ` ` B ` 通过逗号隔开，这种写法其实是一种过时的写法，但是考虑到大部分人都在用，匹配CSS
RGB颜色的时候也应该考虑到。 ` rgba() ` 语法也是一种过时的写法，同样本次也会考虑这种颜色格式的匹配。

CSS ` hsl() ` 的语法和 ` rgb() ` 是一致的，这里不在赘述，需要请移至MDN官网查看。

16进制Hex的色值写法分为以下几种：

    
    
    #RGB        // The three-value syntax  
    #RGBA       // The four-value syntax  
    #RRGGBB     // The six-value syntax  
    #RRGGBBAA   // The eight-value syntax  
    

其中 ` R ` ` G ` ` B ` 的值为0到ff。

在CSS中，一些常用颜色可以使用预定义的关键字来表示，例： ` red ` 、 ` blue ` 、 ` green `
等，这些就是颜色关键词。完整的标准关键词可以在MDN官网找到。

####  颜色匹配

了解了颜色在项目中的多种写法，下面考虑如何匹配代码中的颜色。常见的方案分为两种：

  * 基于文本模式的正则匹配 
  * 基于代码结构的抽象语法树（AST)分析 

前者实现比较简单且通用性大，因此本插件选择通过正则语法，针对不同的颜色格式，定制不同的匹配策略。

编写一个基本的正则表达式来匹配颜色代码其实不难，但是如果想提高要求，确保写出来的正则表达式既准确又具有良好的防御性，能够考虑到各种边缘情况以及性能优化，这并不容易。

为了实现这一点，此处我借鉴了另外一个插件Color
Highlight的实现。这个插件是我目前在使用的比较好用的插件，在开发自己的插件之前，我从没想过它是如何实现的，但是我熟悉它的功能：在编写代码的时候，可以高亮显示当前当前编辑器中的颜色格式。要实现的功能和我的有相似之处，所以了解这个插件的实现应该对于完成自己的插件很有帮助。

事实确实是这样的，Color Highlight插件给出了一系列针对不同颜色格式的匹配策略，如下图所示。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicES2HSTC6iboU6ribN9Mia8Z9I9yOHETjCdWCIoWicu6UP6jhUElcibcGxTWWJLZSscJ76RHv4RSjpSlOg/640?wx_fmt=png&from=appmsg)

基于它的实现，下面分析了针对不同颜色格式的正则表达式解析。

#####  RGB和HSL

    
    
    const colorRegex = /((rgb|hsl)a?(\s*[\d]*.?[\d]+%?\s*(?<commaOrSpace>\s|,)\s*[\d]*.?[\d]+%?\s*\k<commaOrSpace>\s*[\d]*.?[\d]+%?(\s*(\k<commaOrSpace>|/)\s*[\d]*.?[\d]+%?)?\s*))/gi;  
    

整个正则表达式是一个全局不区分大小写的匹配（由结尾的 ` gi ` 标志指定），用于在文本中查找所有匹配的颜色值。主要分为两部分： ` 格式名称(值) `
，其中格式名称分为三种情况，如下图所示，分别为 ` rgb ` , ` hsl ` , ` rgba ` , ` hsla `
。后半部分主要是匹配它们的不同书写方式，包含数字值、逗号或空格分割。具体解析如下：

  1. ` ((rgb|hsl)a? ` : 这部分使用捕获组来匹配 ` rgb ` 或 ` hsl ` 字符串，后面跟着一个可选的 ` a ` 字符，这样可以匹配 ` rgb ` 、 ` rgba ` 、 ` hsl ` 和 ` hsla ` 。 

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicES2HSTC6iboU6ribN9Mia8Z9IGpxHhdFOBScuJOQ8ykhsjySGkMYEw3cYVS83p0PfhNadOvZjULlhMw/640?wx_fmt=png&from=appmsg)

  2. ` ( ` ：匹配左括号 
  3. ` **\s*[\d]*.?[\d]+%?\s*** ` : 用来匹配颜色值的第一个参数，（如红色值 R、色相值 H ），首先是匹配零个或多个空格符，然后是一个数字（可能是整数或小数），该数字可能后跟一个百分号 ` % ` 。 

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicES2HSTC6iboU6ribN9Mia8Z9IcM7dCnUA2HMz6A3PNK68fyHMYG3Zf1l0aoib4No6p8S9YThshhyFcpA/640?wx_fmt=png&from=appmsg)

  3. ` (?<commaOrSpace>\s|,) ` : 定义了一个命名捕获组 ` commaOrSpace ` ，用来匹配空格或逗号。这是颜色值参数之间的分隔符。 
  4. ` **\s*[\d]*.?[\d]+%?\s*** ` : 用来匹配颜色值的第二个参数，（如红色值 R、色相值 H ），首先是匹配零个或多个空格符，然后是一个数字（可能是整数或小数），该数字可能后跟一个百分号 ` % ` 。 

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicES2HSTC6iboU6ribN9Mia8Z9IG0ebwLOestO1Hb8pljEaQSlqzicGY9lI570Cr8Tm2DJ5PYW74TrVUjg/640?wx_fmt=png&from=appmsg)

  5. ` \k<commaOrSpace> ` : 对前面定义的 ` commaOrSpace ` 捕获组的引用，确保分隔符的一致性。 
  6. ` \s*[\d]*.?[\d]+%? ` : 再次匹配一个数字（整数或小数），可能后跟一个百分号 ` % ` 。这部分用于匹配颜色值的第三个参数。 

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicES2HSTC6iboU6ribN9Mia8Z9IJexgtIs8GYEXfzWdSMON4S4SbeicaibvwEBc5Ta1kBkUwRaksJiazFhIw/640?wx_fmt=png&from=appmsg)

  7. ` **\s*(\k<commaOrSpace>|/)\s*[\d]*.?[\d]+%?)?\s* ` : 这是一个可选的捕获组，用于匹配颜色透明度值（alpha 值）。它首先匹配空格，然后是前面定义的分隔符（ ` commaOrSpace ` ）或斜杠 ` / ` ，后面跟着空格和数字（整数或小数），数字后面可能有一个百分号 ` % ` **。 ![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicES2HSTC6iboU6ribN9Mia8Z9I14JwbX1rMbWyTicqvROSUqOApwNwibOWJgpAZ1PEX5DX56dM1s7ZibYew/640?wx_fmt=png&from=appmsg)
  8. ` ) ` ：最后匹配右括号 ` ) ` 。 

#####  颜色关键词

收集MDN官网给出的颜色关键字，写一个简单的正则，下面是列举了部分色值的正则匹配：

    
    
    (aqua|black|blue|fuchsia|gray|green|lime|maroon|navy|olive|purple|red|silver|teal|white|yellow)  
    

#####  16进制Hex

    
    
    const colorHex =  
      /.?((?:#|\b0x)([a-f0-9]{6}([a-f0-9]{2})?|[a-f0-9]{3}([a-f0-9]{1})?))\b/gi;  
    

这个正则表达式用来匹配 CSS
中的十六进制颜色值，同时也匹配十六进制颜色值的简写形式，并且考虑到可能的透明度值（RGBA格式）。下面逐部分解释这个正则表达式：

  1. ` .? ` : 匹配任意单个字符，但不捕获它。 
  2. ` ((?:#|\b0x) ` : 这是一个非捕获组（由 ` ?: ` 开头），用于匹配 ` # ` 或 ` 0x ` 。这里的 ` \b ` 是一个单词边界，确保 ` 0x ` 前面是一个单词边界，避免中间匹配到不正确的字符串。 

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicES2HSTC6iboU6ribN9Mia8Z9In7vCeEpViaUYH6tD1gsRGChT9WRofyrprbAu9amqYuk7AHoicJ84iat7w/640?wx_fmt=png&from=appmsg)

  3. ` ([a-f0-9]{6}([a-f0-9]{2})?|[a-f0-9]{3}([a-f0-9]{1})?) ` : 这部分有两大选项，分别通过 ` | ` 分隔： 

     * ` [a-f0-9]{6}([a-f0-9]{2})? ` : 匹配6个十六进制字符，后面可选地跟随2个十六进制字符（通常用于表示透明度）。 
     * ` [a-f0-9]{3}([a-f0-9]{1})? ` : 匹配3个十六进制字符，后面可选地跟随1个十六进制字符。这是十六进制颜色的简写形式 
  4. ` \b ` : 单词边界，确保匹配的字符串在一个单词的边界结束，防止部分匹配到更长的字符串中。 

####  颜色获取

了解了不同颜色格式的匹配规则后，颜色的获取思路其实很简单：遍历每个文件，获取每个文件的文本字符串，通过对应的正则匹配策略去匹配。基本的代码如下所示：

#####  策略维护

从产品的使用场景看来，本插件所关注的颜色格式比较简单：一些基本的rgb, hex,
hsl，单词的颜色可以不考虑，因为用不上，能用上我都可以直接匹配了，因此我们只关注没有语义、且常用的颜色格式。

最终，针对本插件，可以总结出下面三种策略：

  * ` findColorFunctionsInText ` : 查找rgb(a), hsl(a)基本颜色 
  * ` findHexRGBA ` ：查找hex颜色 

    
    
    this.strategies = [findColorFunctionsInText,findHexRGBA];  
    

#####  文件遍历

    
    
    for (const file of files) {  
      try {  
        const document = await vscode.workspace.openTextDocument(file);  
        const instance = await this.findOrCreateInstance(document);  
      
        colorsInfos.push(await instance.getColorInfo());  
      } catch {  
        continue;  
      }  
    }  
    

#####  颜色获取 ` getColorInfo `

    
    
     getColorInfo(document = this.document) {  
        const text = this.document.getText();  
        const version = this.document.version.toString();  
      
        const file = this.document.uri.fsPath; // file path  
        const result = await Promise.all(this.strategies.map((fn) => fn(text)));  
       return resolveResult(result); // 颜色解析，根据colorInfos的数据格式进行数据解析  
    }  
    

###  页面更新

颜色的更新场景主要有以下几种情况：

  * 编辑已有文件的颜色 
  * 新增有颜色的文件 
  * 删除了有颜色的文件 

针对这三种情况，在产品设计上，可以设置一个“刷新按钮“按钮，点击该按钮，拉取最新的颜色信息。本章节主要从消息通信和更新机制两个角度讨论本插件的实现。

####  消息通信

在页面上点击按钮，然后执行对应的事件，这点的实现涉及到VSCode插件中 ` Extension ` 和 ` Webview ` 的通信，主要是通过 `
postMessage ` 和 ` onDidReceiveMessage ` 实现消息的发送与接收。

#####  在 ` Webview ` 发送消息

` Extension ` 里通过 ` vscode.postMessage ` 发送消息:

    
    
    // 刷新  
    refreshBtn.addEventListener('click', () => {  
      // 如果已经在Loading, 无需发送message  
      if (refreshBtn.classList.contains('btn--loading')) {  
        return;  
      }  
      
      refreshBtn.classList.add('btn--loading');  
      vscode.postMessage({ command: 'refresh' });  
    });  
    

#####  在 ` Extension ` 接收消息

    
    
    webviewView.webview.onDidReceiveMessage((message) => {  
      switch (message.command) {  
        case 'refresh':  
          const prom = () => this.doUpdateWebView()  
      
          prom().finally(() => {  
            webviewView.webview.postMessage({  
              command: 'refreshEnd'  
            });  
          });  
      
          break;  
      }  
    });  
    

####  更新机制

在 ` Extension ` 接受到需要更新的消息后，需要执行对应的更新机制，这个实现主要封装在 ` doUpdateWebView `
方法中。更新机制主要实现的就是更新 ` colorInfos ` ，并重新执行渲染函数以更新UI。

为了让插件的第一个版本快速上线，文件新增、删除场景的更新我们可以重新扫描一下所有文件；文件编辑的场景容易定位具体文档，所以可以按需更新。

下面给出了本项目的更新实现，其中整个了页面初始化的渲染，因为这个场景的逻辑和文件新增、删除重复。

    
    
    private async doUpdateWebView() {  
      try {  
        if (  
          this.type === 'init' ||  
          this.type === 'add' ||  
          this.type === 'delete'  
        ) {  
          await this.initDataView();  
      
          return Promise.resolve();  
        }  
      
        // 颜色变更：text change  
      
        // 收集变更的document，局部更新颜色视图  
        for (let index = 0; index < this.instanceMap.length; index++) {  
          const instance = this.instanceMap[index];  
      
          // 如果页面更改了  
          if (instance.changed) {  
            const colorDocumentItem = await instance.getColorInfo();  
            this.colorMapArray[index] = colorDocumentItem;  
            // 恢复  
            instance.changed = false;  
          }  
        }  
      
        // 更新颜色信息  
        this.colorInfos = updateColorInfosByMap(this.colorMapArray);  
      
        // 更新视图  
        this._view.webview.html = this._getHtmlForWebview(this._view.webview);  
      
        return Promise.resolve();  
      } catch {  
        return Promise.reject();  
      }  
    }  
    

代码示意图如下所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicES2HSTC6iboU6ribN9Mia8Z9IV43cchD36L60IYPtMnzZ1loRe6ZMYYcOeicCDgmTj9k7PbAMY5dtIlw/640?wx_fmt=png&from=appmsg)

###  插件配置

并不是项目中所有文件都有颜色值，所以为了让插件能够有效地找到有用的色值，我们需要给插件增加文件类型配置项，用来定义哪些文件应该在文件扫描过程中包含或排除。

` include ` 和 ` exclude ` 这两个配置项在很多前端开发相关的工具和配置文件中非常常见，如 ` webpack.config `
文件、 ` tsconfig ` 文件、 ` babel.config ` 文件。为了降低插件使用者的学习曲线，本插件也选择使用 ` include ` 和
` exclude ` 来指定扫描的文件类型。

下面是本插件默认的配置属性。

    
    
    "color-to-see.findFilesRules": {  
      "default": {  
        "include": [  
          "**/*.js",  
          "**/*.jsx",  
          "**/*.tsx",  
          "**/*.css",  
          "**/*.less",  
          "**/*.sass",  
          "**/*.html",  
          "**/*.vue"  
        ],  
        "exclude": [  
          "**/node_modules/**",  
          "**/dist/**",  
          ".git"  
        ]  
      },  
    

扫描的文件类型主要关注实际开发中包含色值定义的文件类型，比如样式文件、HTML文件或者JS文件。排除不需要扫描或处理的文件夹，如 `
node_modules ` ， ` dist ` ， ` .git ` 文件等。 ` include ` 和 ` exclude ` 的值使用
**Glob Pattern** **语法来指定哪些文件被包括或排除。**

在Extension中，这套配置的使用原理如下：

    
    
    config = vscode.workspace.getConfiguration(EXTENSION_NAME);  
      
    const findFilesUsingConfig = async (config: Config) => {  
      const { include, exclude } = config.findFilesRules;  
      const includePattern = `{${include.join(',')}}`;  
      const excludePattern = `{${exclude.join(',')}}`;  
      
      try {  
        const files = await vscode.workspace.findFiles(  
          includePattern,  
          excludePattern  
        );  
        return files;  
      } catch {  
        return [];  
      }  
    };  
    

##  成品

###  Demo Snapshop

  * 有颜色 

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicES2HSTC6iboU6ribN9Mia8Z9IHBYLAIwwyFXJsicsiaxiaSVTIwoyoHW21iaotgY7Unyv84dVicJs3C6MZ0w/640?wx_fmt=png&from=appmsg)

  * 无颜色 

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicES2HSTC6iboU6ribN9Mia8Z9IxyDdQjGgiaKfIvjIEBgqOxs7hd3cw4kBLRVnfoxQZdbiciakeeia0l5UkQ/640?wx_fmt=png&from=appmsg)

###  插件市场链接

👉👉👉：Color to See - Visual Studio Marketplace

##  局限性分析和优化点

尽管插件的功能大致已经实现了，但是本插件还是有一些局限性，以及后续可以优化的地方

###  文件扫描效率

####  局限性

插件需要扫描整个工作区的所有文件来查找颜色值，尽管增加了文件类型限制少扫描一些文件，但是剩下的文件扫描还是会消耗大量的计算资源。

####  优化

  * 编译打包：项目代码打包到一个文件（如app.js）中以减少颜色扫描的文件数量的方法 
  * 缓存+增量更新：通过缓存机制保留未更改的文件，而只对更改的文件进行扫描，而不是整个工作区。 

###  用户体验

####  局限性

颜色展示必须等待文件全部扫描完，长时间的扫描和等待会影响用户体验

####  优化

实现异步扫描机制，一旦扫描到新的颜色值，就立即在UI中展示，而不是等待所有扫描完成。

###  颜色值跳转功能配置化

####  局限性

颜色值跳转功能的实用性其实比较低，而且在本插件中，如果一个颜色在多个地方出现，我只会记录第一个位置。因此，本插件的使用过程中，用户应该更加关注于颜色值本身，而不是其在代码中的具体位置。

除此之外，颜色跳转到具体某个文件后，当前视图的位置没有滚动到对应的高度。

####  优化

增加一个配置项让用户可以自行决定是否启用颜色值跳转功能。默认关闭。

##  总结

本文基于一个场景问题，介绍了VSCode插件的开发过程，总的来说，这个项目算是一次很有意思的独立开发体验。在这个过程中，可以总结出以下几个比较重要的点：

  * 善于发现问题，并尝试解决它 
  * 积累经验，关注细节，说不定何时能派上用场 
  * Always have fun 

##  最后

关注公众号

获取更多干货实践，欢迎交流分享。

##  小茗推荐

  * [ 门店：“电脑又双叒叕中病毒了” ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247485622&idx=1&sn=e92967722a840eeafa3cbd35597e2b37&chksm=cfe58fb1f89206a7d4961b145834b5531b497932e32ebbd5d955f25c42ee847062ddbc2ce507&scene=21#wechat_redirect)
  * [ 古茗 Mars 预编译技术方案探索 ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247485675&idx=1&sn=0d53b5441b29bbced27c0cf0544dfaab&chksm=cfe58fecf89206fa1ea0ba6af43f7bcb78c6119ae92d6105b1664f8bf6597612f5abd3f2ca29&scene=21#wechat_redirect)
  * [ 探究前端包管理工具：npm、yarn 和pnpm ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247485637&idx=1&sn=ba3ee4db0190d6458b0fcc05a600759c&chksm=cfe58fc2f89206d48344d136d82afef51ff149af722bf7e6f49f6755461bc957961ca974bdde&scene=21#wechat_redirect) [ ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247485637&idx=1&sn=ba3ee4db0190d6458b0fcc05a600759c&chksm=cfe58fc2f89206d48344d136d82afef51ff149af722bf7e6f49f6755461bc957961ca974bdde&scene=21#wechat_redirect)   

预览时标签不可点

微信扫一扫  
关注该公众号





****



****



×  分析

  收藏

