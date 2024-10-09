## 第一章 webgl概述

### 1-webgl是什么？和canvas有什么区别

webgl 是在网页上绘制和渲染三维图形的技术，可以让用户与其进行交互。
div+css、canvas 2d 都是专注于二维图形的。

#### 坐标系区别
canvas 2d 画布和webgl 画布使用的坐标系都是二维直角坐标系，只不过它们坐标原点、y 轴的坐标方向，坐标基底都不一样了。

canvas 2d 坐标系的原点在左上角。
canvas 2d 坐标系的y 轴方向是朝下的。
canvas 2d 坐标系的坐标基底有两个分量，分别是一个像素的宽和一个像素的高，即1个单位的宽便
是1个像素的宽，1个单位的高便是一个像素的高。
![[Pasted image 20241008135443.png]]

webgl坐标系的坐标原点在画布中心。
webgl坐标系的y 轴方向是朝上的。
朝向屏幕方向是z轴
webgl坐标基底中的两个分量分别是半个canvas的宽和canvas的高，即1个单位的宽便是半个个canvas的宽，1个单位的高便是半个canvas的高。

![[Pasted image 20241008135336.png]]

#### 语言不一样

浏览器有三大线程： js 引擎线程、GUI 渲染线程、浏览器事件触发线程。

其中GUI 渲染线程就是用于渲图的，在这个渲染线程里，有负责不同渲染工作的工人。比如有负责渲染HTML+css的工人，有负责渲染二维图形的工人，有负责渲染三维图形的工人。

渲染二维图形的工人和渲染三维图形的工人不是一个国家的，他们说的语言不一样。

==渲染二维图形的工人说的是js语言==。

==渲染三维图形的工人说的是GLSL ES 语言==。

而我们在做web项目时，业务逻辑、交互操作都是用js 写的。

我们在用js 绘制canvas 2d 图形的时候，渲染二维图形的工人认识js 语言，所以它可以正常渲图。

但我们在用js 绘制webgl图形时，渲染三维图形的工人就不认识这个js 语言了，因为它只认识GLSL ES 语言。

因此，这个时候我们就需要找人翻译翻译。

这个做翻译的人是谁呢，它就是我们之前提到过的手绘板，它在webgl 里叫==“程序对象”==。

接下来咱们从手绘板的绘图步骤中捋一下webgl 的绘图思路。

### 2-webgl 的绘图思路

1. 找一台电脑 - 浏览器里内置的webgl 渲染引擎，负责渲染webgl 图形，只认GLSL ES语言。
2. 找一块手绘板 - 程序对象，承载GLSL ES语言，翻译GLSL ES语言和js语言，使两者可以相互通信。
3. 找一支触控笔 - 通过canvas 获取的webgl 类型的上下文对象，可以向手绘板传递绘图命令，并接收手绘板的状态信息。
4. 开始画画 - 通过webgl 类型的上下文对象，用js 画画。


1.在html中建立canvas 画布

```js
<canvas id="canvas"></canvas>
```

2.在js中获取canvas画布

```js
const canvas=document.getElementById('canvas');
```

3.使用canvas 获取webgl 绘图上下文

```js
const gl=canvas.getContext('webgl');
```

4.在script中建立顶点着色器和片元着色器，glsl es

```html
//顶点着色器
<script id="vertexShader" type="x-shader/x-vertex">
    void main() {
        gl_Position = vec4(0.0, 0.0, 0.0, 1.0);
        gl_PointSize = 100.0;
    }
</script>
//片元着色器
<script id="fragmentShader" type="x-shader/x-fragment">
    void main() {
        gl_FragColor = vec4(1.0, 1.0, 0.0, 1.0);
    }
</script>
```

5.在js中获取顶点着色器和片元着色器的文本

```js
const vsSource = document.getElementById('vertexShader').innerText;
const fsSource = document.getElementById('fragmentShader').innerText;
```

6.初始化着色器

```js
initShaders(gl, vsSource, fsSource);


function loadShader(gl, type, source) {
	//根据着色类型，建立着色器对象
	const shader = gl.createShader(type);
	//将着色器源文件传入着色器对象中
	gl.shaderSource(shader, source);
	//编译着色器对象
	gl.compileShader(shader);
	//返回着色器对象
	return shader;
}

function initShaders(gl,vsSource,fsSource){
	//创建程序对象
	const program = gl.createProgram();
	//建立着色对象
	const vertexShader = loadShader(gl, gl.VERTEX_SHADER, vsSource);
	const fragmentShader = loadShader(gl, gl.FRAGMENT_SHADER, fsSource);
	//把顶点着色对象装进程序对象中
	gl.attachShader(program, vertexShader);
	//把片元着色对象装进程序对象中
	gl.attachShader(program, fragmentShader);
	//连接webgl上下文对象和程序对象
	gl.linkProgram(program);
	//启动程序对象
	gl.useProgram(program);
	//将程序对象挂到上下文对象上
	gl.program = program;
	return true;
}
```

7.指定将要用来清空绘图区的颜色

```js
gl.clearColor(0,0,0,1);
```

8.使用之前指定的颜色，清空绘图区

```js
gl.clear(gl.COLOR_BUFFER_BIT);
```

9.绘制顶点

```js
gl.drawArrays(gl.POINTS, 0, 1);
```




### 3-着色器的概念

一个着色器就是一个绘制东西到屏幕上的函数，着色器有顶点着色器和片段着色器

- 顶点着色器（Vertex shader）：描述顶点的特征，如位置、颜色等。处理每个顶点的属性，如位置、法线、纹理坐标等。它的主要任务是将顶点从对象坐标系转换到屏幕坐标系，并传递其他顶点属性给片段着色器。
- 片元着色器（Fragment shader）：进行逐片元处理，如光照。处理每个像素的颜色。它接收来自顶点着色器插值后的数据，并最终决定每个像素的颜色。

#### 3-1-着色器语言

webgl 的着色器语言是GLSL ES语言

- 顶点着色程序，要写在type=“x-shader/x-vertex” 的script中。

```js
<script id="vertexShader" type="x-shader/x-vertex">
    void main() {
        gl_Position = vec4(0.0, 0.0, 0.0, 1.0);
        gl_PointSize = 100.0;
    }
</script>
```

- 片元着色程序，要写在type=“x-shader/x-fragment” 的script中。

```js
<script id="fragmentShader" type="x-shader/x-fragment">
    void main() {
        gl_FragColor = vec4(1.0, 1.0, 0.0, 1.0);
    }
</script>
```

void main() {…… } 是主体函数。只有在main中使用的变量才能在后面拿到地址进行赋值。

在顶点着色器中，gl_Position 是顶点的位置，gl_PointSize 是顶点的尺寸，这种名称都是固定的，不能写成别的。

在片元着色器中，gl_FragColor 是片元的颜色。

vec4() 是一个4维矢量对象。

将vec4() 赋值给顶点点位gl_Position 的时候，其中的前三个参数是x、y、z，第4个参数默认1.0

将vec4() 赋值给片元颜色gl_FragColor 的时候，其中的参数是r,g,b,a。

至于GLSL ES语言的其它知识，咱们会在后面另开一篇详解，这里先以入门为主。

在第6步中，我们使用了一个自定义的方法initShaders() ，这是用于初始化着色器的，接下来咱们详细说一下。

####  3-2-着色器初始化

初始化着色器的步骤：

1. 建立程序对象，目前这只是一个手绘板的外壳。
    
    ```js
    const shaderProgram = gl.createProgram();
    ```
    
2. 建立顶点着色器对象和片元着色器对象，这是手绘板里用于接收触控笔信号的零部件，二者可以分工合作，把触控笔的压感（js信号）解析为计算机语言(GLSL ES)，然后让计算机(浏览器的webgl 渲染引擎)识别显示。
    
    ```js
    const vertexShader = loadShader(gl, gl.VERTEX_SHADER, vsSource);
    const fragmentShader = loadShader(gl, gl.FRAGMENT_SHADER, fsSource);
    ```
    
3. 将顶点着色器对象和片元着色器对象装进程序对象中，这就完成的手绘板的拼装。
    
    ```js
    gl.attachShader(shaderProgram, vertexShader);
    gl.attachShader(shaderProgram, fragmentShader);
    ```
    
4. 连接webgl 上下文对象和程序对象，就像连接触控笔和手绘板一样（触控笔里有传感器，可以向手绘板发送信号）。
    
    ```
    gl.linkProgram(shaderProgram);
    ```
    
5. 启动程序对象，就像按下了手绘板的启动按钮，使其开始工作。
    
    ```
    gl.useProgram(program);
    ```
    

上面第二步中的建立着色对象方法loadShader()，是一个自定义的方法，其参数是(webgl上下文对象，着色器类型，着色器源文件)，gl.VERTEX_SHADER 是顶点着色器类型，gl.FRAGMENT_SHADER是片元着色器类型。

```js
function loadShader(gl, type, source) {
    const shader = gl.createShader(type);
    gl.shaderSource(shader, source);
    gl.compileShader(shader);
    return shader;
}
```

- gl.createShader(type) ：根据着色器类型建立着色器对象的方法。
    
- gl.shaderSource(shader, source)：将着色器源文件传入着色器对象中，这里的着色器源文件就是我们之前在script 里用GLSL ES写的着色程序。
    
- gl.compileShader(shader)：编译着色器对象。
    

在以后的学习里，initShaders 会经常用到，所以我们可以将其模块化。

```js
function initShaders(gl,vsSource,fsSource){
    //创建程序对象
    const program = gl.createProgram();
    //建立着色对象
    const vertexShader = loadShader(gl, gl.VERTEX_SHADER, vsSource);
    const fragmentShader = loadShader(gl, gl.FRAGMENT_SHADER, fsSource);
    //把顶点着色对象装进程序对象中
    gl.attachShader(program, vertexShader);
    //把片元着色对象装进程序对象中
    gl.attachShader(program, fragmentShader);
    //连接webgl上下文对象和程序对象
    gl.linkProgram(program);
    //启动程序对象
    gl.useProgram(program);
    //将程序对象挂到上下文对象上
    gl.program = program;
    return true;
}
function loadShader(gl, type, source) {
    //根据着色类型，建立着色器对象
    const shader = gl.createShader(type);
    //将着色器源文件传入着色器对象中
    gl.shaderSource(shader, source);
    //编译着色器对象
    gl.compileShader(shader);
    //返回着色器对象
    return shader;
}
export {initShaders}
```

后面在需要的时候，import 引入即可。

```js
import {initShaders} from '../jsm/Utils.js';
```

#### 3-3-变量修饰符：

用于指定变量的作用域、生命周期和用途。不同的修饰符在顶点着色器和片段着色器中起到不同的作用。

+ attribute:用于顶点着色器，定义从顶点缓冲区传入的变量（仅在顶点着色器中使用）。
+ uniform:定义在整个渲染过程中保持不变的变量，常用于传递变换矩阵、光照参数等。
+ varying:用于在顶点着色器和片段着色器之间传递插值数据。

#### 3-4-数据类型

数据类型：基本数据类型： float, int, bool
向量类型： vec2, vec3, vec4(浮点型； ivec2 ivec3,ivec4(整数型)；bvec2,bvec3 bvec4(布尔型)矩阵类型： mat2, mat3, mat4
采样器类型： sampler2D samplerCube(用于纹理采样)

举例vec4：
`vec4` 由四个分量组成，通常记作 `(x, y, z, w)`，其中 `w` 通常用于表示齐次坐标或透明度。
`mat4` 表示一个 4x4 的矩阵。它常用于表示空间中的变换，如平移、旋转和缩放。`mat4` 的元素可以通过行和列的索引访问，例如 `mat4[0][0]` 访问第一行第一列的元素。常见的操作包括矩阵乘法，可以使用 `*` 运算符将矩阵与向量或另一个矩阵相乘。