### 安装 
[Install Tailwind CSS with Vite - TailwindCSS中文文档 | TailwindCSS中文网](https://www.tailwindcss.cn/docs/guides/vite#react)

### 主要概念
宽高  w-4 h-4   代表宽和高1rem也就是默认16px， w-20就代表  5rem和默认80px

#### 一般flex布局设置
父元素：flex flex-col items-center justify-center flex-wrap  gap-10 overflow-x-hidden

子元素：base-4 flex-auto flex-none flex-1

###### 关于响应式媒体查询
<font style="color:rgb(13, 13, 13);">在 Tailwind CSS 中，屏幕尺寸的前缀包括：</font>

+ **sm**<font style="color:rgb(13, 13, 13);">: 手机端 (>= 640px)</font>
+ **md**<font style="color:rgb(13, 13, 13);">: 平板电脑端 (>= 768px)</font>
+ **lg**<font style="color:rgb(13, 13, 13);">: 笔记本电脑端 (>= 1024px)</font>
+ **xl**<font style="color:rgb(13, 13, 13);">: 大型显示屏端 (>= 1280px)</font>
+ **2xl**<font style="color:rgb(13, 13, 13);">: 非常大型显示屏端 (>= 1536px)  
</font>

<font style="color:#DF2A3F;">flex-none md:flex-1</font> 代表默认是flex-none不进行shrink或者grow，当屏幕>=768px后flex-1进行shrink或者grow

### 对于flex和grid的整理
grid关键就是对网格的创建和放置

+ 父元素创建网格

```css
display: grid;
grid-template-columns: 6fr 1fr 1fr 3fr 5fr 1fr; 
grid-template-rows: 40px repeat(6, auto);


row-gap: 20px;
column-gap: 20px;
// 简洁 gap: row-gap colomn-gap;


// 给每个列起个名
grid-template-columns: [c1] 100px [c2] 100px [c3] auto [c4]
// 给每个行起个名
grid-template-rows: [r1] 100px [r2] 100px [r3] auto [r4];
// 可视化起名
grid-template-areas: 'a a a'
                     'b b b'
                     'c c c';  不需要起名的地方，则使用"."表示例如 'a . c'

// 如果子元素不说具体放哪，会按照先行后列按顺序排入
grid-auto-flow: column;  // 此时会先列后行

// 子元素在单元格中的位置
justify-items: start | end | center | stretch;  // 列方向
align-items: start | end | center | stretch;  // 行方向
// 简化 place-items: <align-items> <justify-items>;
```

+ 子元素放置位置

```css
grid-column: 1 / 2;   // 使用span代表跨几个格，  等于 grid-column: 1 / span 1;
grid-row: 3 / 4;

// 结合一下
grid-area: <row-start> / <column-start> / <row-end> / <column-end>;  相当于 3 / 1 / 4 / 2

// 用名字来坐位置
grid-area: a;

// 我们子元素有自己的想法，不想受父元素justify-items和align-items的影响，我们在块内想在哪就在哪
  justify-self: start | end | center | stretch;
  align-self: start | end | center | stretch;
  // 结合一下
  place-self: <align-self> <justify-self>;
```

![](https://cdn.nlark.com/yuque/0/2024/png/694718/1706239548015-7b127599-6e10-43bd-ae6c-27ac6c20fc74.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_102%2Ctext_55Sxemhhb2J15Y6f5Yib5oiW6ICF6L2s6L295L-u5pS55o6S54mI%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fformat%2Cwebp%2Fresize%2Cw_1191%2Climit_0)

##### 
