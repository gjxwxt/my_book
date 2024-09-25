##### 1. 选中第三个到第六个元素（3.4.5.6）
```javascript
element:nth-child(-n+6):nth-child(n+3){
 //连在一起即可，注意前后顺序，先匹配前六个在匹配后四个
}

1. element:nth-child(odd)===element:nth-child(2n+1)
2. element:nth-child(even)===element:nth-child(2n)
3. element:nth-child(-n+3)  //匹配前三个元素1.2.3
4. element:nth-child(n+3)   //匹配第三个元素及之后的元素3.4....
5. element:nth-child(-n+6):nth-child(n+3)//即前六个中的第三个及往后的元素3.4.5.6
```

##### <font style="color:#E8323C;">2. mask-image</font><font style="color:rgb(81, 87, 103);">属性</font>
<font style="color:rgb(81, 87, 103);">b站视频中点播视频是ai生成蒙版下发下来的，直播是实时蒙版，实现弹幕不遮挡人物</font>

##### <font style="color:rgb(81, 87, 103);">3. 对dom的class属性进行增删查（add、remove、contain）</font>
<font style="color:#E8323C;">element.classList</font><font style="color:rgb(81, 87, 103);">.contain('xxx')，可以查看当前dom元素是否包含该class属性</font>

```javascript
<div id="dom1" class="abc"></div>

const dom1=document.querySelector("#dom1");

// className返回的是字符串，有出错的可能，classList创建contain方法会记录不同的class，而不是直接合并成一个字符串
dom1.onclick=function(){
	// dom1.classList += "def";
	dom1.classList.add('def');// 添加
	// dom1.classList.remove('def')// 删除
	console.log(dom1.classList)// DOMTokenList
	console.log(dom1.classList.contain("def")) //true
	console.log(dom1.classList.contain("de")) //false
	console.log(dom1.className) //'abcdef'
	console.log(dom1.className.includes('de')) // true
}
```

##### 4. 关于三倍屏
         window. devicePixelRatio （设备像素比）= 设备的水平物理像素/设备 PPI（<font style="color:rgb(51, 51, 51);">Pixels Per Inch</font>）。假设同ppi的设备，如果长宽物理像素数量翻倍，设备像素比也会翻倍，也就是之前1px对应1个像素， 现在1px对应2个像素，图片被放大了，为了保持清晰度，可以针对不同 <font style="color:rgb(38, 38, 38);">devicePixelRatio 显示不同图片</font>

```css
#my-image { background: (low.png); }
@media only screen and (min-device-pixel-ratio: 1.5) {
	#my-image { background: (high.png); }
}
```

##### 5. transform和absolute的区别
1. <font style="color:rgb(56, 56, 56);">改变transform或opacity不会触发浏览器重排（reflow）或重绘（repaint），只会触发（compositions）复合，⽽改变绝对定位会触发重新布局，进⽽触发重绘和复合。</font>
2. <font style="color:rgb(56, 56, 56);">transform使浏览器为元素创建⼀个 GPU 图层，触发硬件加速。 因此translate()更⾼效，可以缩短平滑动画的绘制时间。 </font>
3. <font style="color:rgb(56, 56, 56);">translate改变位置时，元素依然会占据其原始空间，绝对定位不占。</font>

##### 6. 单行溢出与多行溢出
```css
/* 单行溢出 */
overflow: hidden;            // 溢出隐藏
text-overflow: ellipsis;      // 溢出用省略号显示
white-space: nowrap;         // 规定段落中的文本不进行换行

/* 多行溢出 */
overflow: hidden;            // 溢出隐藏
text-overflow: ellipsis;     // 溢出用省略号显示
display:-webkit-box;         // 作为弹性伸缩盒子模型显示。
-webkit-box-orient:vertical; // 设置伸缩盒子的子元素排列方式：从上到下垂直排列
-webkit-line-clamp:3;        // 显示的行数
```

##### 7. 常见布局
```css
.outer {
  height: 100px;
}
.left {
  float: left;
  width: 200px;
  background: tomato;
}
.right {
  margin-left: 200px;
  width: auto;
  background: gold;
}
```

```css
/* 垂直居中 */
父级设置有三类，
		内联line-height，
		块级inline-block+line-height，
		多子容器flex+justify-content: center;
自身设置有三类，
		脱离文档选定位，定高: margin-top 负值，不定高: translate，
				最佳包裹:position: absolute;top: 0;bottom: 0;margin: auto 0;
/* 水平居中 */
父级设置有三类，
	内联:text-align:center;
	多子容器display: flex+align-items: center；
自身设置有 1+3，
	块级定宽：margin-left:auto;margin-right:auto;
	脱离文档选定位，定宽: margin 负值，不定宽: translate，
				最佳包裹配置  position: absolute;left: 0;right: 0;margin: 0 auto;

```

##### 8、 css中var()使用字符串
```jsx
// 字符串
--bar: 'hello';   
--foo: var(--bar)' world';

// 变量值是数值
foo { 
  --gap: 20; 
  margin-top: var(--gap)px;  /* 无效 */  
}

// 上面代码中，数值与单位直接写在一起，这是无效的。必须使用calc() 函数，将它们连接，
foo { 
  --gap: 20; 
  margin-top: calc(var(--gap) * 1px); 
}

// 变量值带有单位
.foo {
  --foo: '20px'; 
  font-size: var(--foo);  /* 无效 */ 
} 

.foo {
  --foo: 20px;
  font-size: var(--foo);  /* 有效 */ 
}
```

##### 9、 块状盒子宽度自适应：width: fit-content
##### 10、让元素保持宽高比：aspect-ratio： 宽/高； 例如16/9代表保持16/9的比例
##### 注意事项：
1. 子组件即使用了scope，在最外层也会添加上父组件的scopeid，所以有两个scopeid，用一个div包裹起来就好了
2. img的title是滑到图片上显示的文字，alt是图片无法显示展示文字，seo会重点分析
3. 渐进增强：先针对低版本浏览器构建页面，再对高版本进行优化。优雅降级：针对高版本进行开发，再向下兼容
4. inline元素不能设置宽高，但是可以设置水平方向的margin和padding，垂直方向不行。
5. display属性不会继承。visibility属性继承，可实现父隐子现。
6. link引入css样式中也可以使用媒体查询

```css
<!-- link元素中的CSS媒体查询 --> 
<link rel="stylesheet" media="(max-width: 800px)" href="example.css" /> 
<!-- 样式表中的CSS媒体查询 --> 
<style> 
@media (max-width: 600px) { 
  .facet_sidebar { 
    display: none; 
  } 
}
</style>
```



