#### 什么是framer
framer是网站原型设计工具，但是你设计好之后就会生成代码，而且有很多模板，并且会根据不同端生成响应式代码，一套代码，多端适配，2013年的时候看到了有ai一步生成的，目前没看到。

#### framer-motion
使用react开发的项目，我也想用framer中的各种动画交互咋办，引入framer-motion呗，让你获得一样的体验，

> npm install framer-motion
>
> import { motion } from "framer-motion"
>

##### 一、初步尝试
###### 基本结构，variants、initial、animate、exit
```jsx
<motion.xxx
  variants={
    {
      hidden: { y: 50, opacity: 0 },
      visible: { 
        y: 0, 
        opacity: 1, 
        transition: { 
          duration: 1, 
          delay: 0.2,
          ease: 'easeOut' 
        } 
      }
    }
  }
  initial="hidden"
  animate="visible"
  exit="exit"
>
</motion.xxx>
```

```tsx
import { motion } from "framer-motion"
// 原来结构
<div>
  <div></div>
</div>

// 给要加动画的结构改为motion.xxx
<motion.div>
  <div></div>
</motion.div>

// 添加props
const gridContainerVariants = {
  hidden: { opacity: 0 }, //key的名字可以随便起，只要和inital、animate、exit对上就行
  visible: {
    opacity: 1,
    transition: {
      staggerChildren: 0.25, //内部元素逐个显示，间隔0.25s
    }
  },
  exit: { opacity: 0 },
}
<motion.div
  variants={gridContainerVariants}
  initial="hidden"  // 对应variants中的初始状态的key
  animate="visible" // 对应动画主题
  exit="exit"
>
    <motion.div
      variants={{
        hidden: { y: 50, opacity: 0 },
        visible: { y: 0, opacity: 1, transition: { duration: 1, delay: 0.2, ease: 'easeOut' } },
      }}>
    </motion.div>
</motion.div>

// 有没有发现子元素中的initial、animate、exit没有写，因为继承了父元素的设置，方便
// 如果子元素单独设置了，就覆盖父元素透传过来的属性了
```

```jsx
<motion.div
  animate={{
    // 使用数组，来写分帧动画，例如动画时间平均分成四份，依次属性的值是多少
    scale: [1, 2, 2, 1],
    rotate: [0, 90, 90, 0],
    borderRadius: ["10%", "10%", "50%", "10%"],
  }}
  transition={{
    duration: 5,
    ease: "easeInOut",
    repeat: Infinity, // 一直循环
    repeatDelay: 1,
  }}
/>

// 对上面简单的过渡动画进行修改，让他循环上下动，最后一段动画回归初始位置
<motion.div
  animate={{
    y: [100, 0, 0, 100],
    opacity: [0, 1, 1, 0]
  }}
  transition={{
    duration: 5, 
    ease: "easeInOut", 
    repeat: Infinity, 
    delay: 0.2 
  }}
/>
```

这么一看和自己写animate差不多呀，就是换成js实现而已，那么继续深入把

###### 触发式动画，animate={isOpen ? "open" : "closed"}，不同情况使用不同动画
例如左侧菜单栏，点击展示icon的时候使用animate1，再次点击icon隐藏时使用animate2，如何优雅的写，css写成js方法，就是方便修改

initial={false}   在需要触发的动画来讲，不需要一上来就动

动态切换动画animate

```tsx
import { motion } from "framer-motion"
import { Toggle } from "./Toggle"

const variants = {
  open: { opacity: 1, x: 0 },
  closed: { opacity: 0, x: "-100%" },
}

export const MyComponent = () => {
  const [isOpen, setIsOpen] = useState(false)

  return (
    <motion.nav
      // 默认情况下isOpen为false，animate会初始展示closed动画，
      // 但是不需要，于是可以false，阻止一开始展示的动画
      initial={false}
      variants={variants}
      animate={isOpen ? "open" : "closed"}
    >
      <Toggle onClick={() => setIsOpen(isOpen => !isOpen)} />
      <Items />
    </motion.nav>
  )
}
```

motion是如何结合svg的path的

```tsx
import * as React from "react";
import { motion } from "framer-motion";

const Path = props => (
  <motion.path
    fill="transparent"
    strokeWidth="3"
    stroke="hsl(0, 0%, 18%)"
    strokeLinecap="round"
    {...props}
  />
);

export const MenuToggle = ({ toggle }) => (
  <button onClick={toggle}>
    <svg width="23" height="23" viewBox="0 0 23 23">
      {/* 这个Path需要动画，于是用motion.path写 */}
      <Path
        variants={{
          closed: { d: "M 2 2.5 L 20 2.5" },
          open: { d: "M 3 16.5 L 17 2.5" }
        }}
      />
      {/* 关闭的时候多出来的一条线 */}
      <Path
        d="M 2 9.423 L 20 9.423"
        variants={{
          closed: { opacity: 1 },
          open: { opacity: 0 }
        }}
        transition={{ duration: 0.1 }}
      />
      <Path
        variants={{
          closed: { d: "M 2 16.346 L 20 16.346" },
          open: { d: "M 3 2.5 L 17 16.346" }
        }}
      />
    </svg>
  </button>
);

```

transition: { staggerChildren: 0.07, delayChildren: 0.2 }

transition: { y: { stiffness: 1000, velocity: -100 }}

whileHover={{ scale: 1.1 }}

whileTap={{ scale: 0.95 }}

```tsx
import * as React from "react";
import { motion } from "framer-motion";

const getBoxVariants = {
  open: {
    transition: { staggerChildren: 0.07, delayChildren: 0.2 }
  },
  closed: {
    transition: { staggerChildren: 0.05, staggerDirection: -1 }
  }
};

const getItemVariants = {
  open: {
    y: 0,
    opacity: 1,
    transition: {
      y: { stiffness: 1000, velocity: -100 }
    }
  },
  closed: {
    y: 50,
    opacity: 0,
    transition: {
      y: { stiffness: 1000 }
    }
  }
};

const colors = ["#FF008C", "#D309E1", "#9C1AFF", "#7700FF", "#4400FF"];

export const Navigation = () => (
  const style = { border: `2px solid ${colors[i]}` };

  return <motion.ul variants={getBoxVariants}>
    {itemIds.map(i => (
      <motion.li
        i={i} 
        key={i}
        variants={getItemVariants}
        whileHover={{ scale: 1.1 }}
        whileTap={{ scale: 0.95 }}
      >
        <div className="icon-placeholder" style={style} />
        <div className="text-placeholder" style={style} />
      </motion.li>
    ))}
  </motion.ul>
);

const itemIds = [0, 1, 2, 3, 4];

```

> [https://codesandbox.io/p/sandbox/framer-motion-side-menu-forked-3hx7n6?file=%2Fsrc%2FExample.tsx%3A34%2C16](https://codesandbox.io/p/sandbox/framer-motion-side-menu-forked-3hx7n6?file=%2Fsrc%2FExample.tsx%3A34%2C16)
>

<details class="lake-collapse"><summary id="u4c79ac21"><span class="ne-text">初步总结</span></summary><ol class="ne-ol"><li id="ub73e067a" data-lake-index-type="0"><span class="ne-text">子组件通用的变量可以放在父组件，子组件使用自己的variants和transation例如</span></li></ol><ol class="ne-list-wrap"><ol ne-level="1" class="ne-ol"><li id="u63ad37f3" data-lake-index-type="0"><span class="ne-text">animate={isOpen ? &quot;open&quot; : &quot;closed&quot;}</span></li><li id="ue6ba1cdb" data-lake-index-type="0"><span class="ne-text">initial={false}</span></li><li id="uae1e2e93" data-lake-index-type="0"><span class="ne-text">custom给variants传的参，例如custom={height}</span></li></ol></ol><ol start="2" class="ne-ol"><li id="uab7e1e4f" data-lake-index-type="0"><span class="ne-text">transation分两种，一种是variants中的，一种是和variants同级的</span></li></ol><ol class="ne-list-wrap"><ol ne-level="1" class="ne-ol"><li id="uce063268" data-lake-index-type="0"><span class="ne-text">在variants中的：给单独设置任意多个样式属性的过渡变化数据</span></li><li id="u3db6a96a" data-lake-index-type="0"><span class="ne-text">&lt;motion.div  transation={xxx} /&gt; 这种所有的变化都适合该transation属性</span></li></ol></ol><ol start="3" class="ne-ol"><li id="u1b8819c2" data-lake-index-type="0"><span class="ne-text">transation中的属性</span></li></ol><ol class="ne-list-wrap"><ol ne-level="1" class="ne-ol"><li id="ubc5ef844" data-lake-index-type="0"><span class="ne-text">type一般不用设置，如果有duration默认是Tween</span></li></ol></ol><ol class="ne-list-wrap"><ol class="ne-list-wrap"><ol ne-level="2" class="ne-ol"><li id="ub4c35595" data-lake-index-type="0"><span class="ne-text">Tween的属性：ease，数组写关键帧，duration</span></li><li id="u2f453be5" data-lake-index-type="0"><span class="ne-text">spring是类似弹簧的效果，属性比较多，和运动相关的属性变化模式默认为spring，例如x，y，scale，rotate</span></li></ol></ol></ol><ol class="ne-list-wrap"><ol class="ne-list-wrap"><ol class="ne-list-wrap"><ol ne-level="3" class="ne-ol"><li id="u1464b4c8" data-lake-index-type="0"><span class="ne-text">bounce： 0-1  弹性up，默认0.25，和stiffness、damping或者mass属性有冲突</span></li><li id="u073f144a" data-lake-index-type="0"><span class="ne-text">mass：模拟物体质量大小，默认为1，越重弹性越小</span></li><li id="udbb8b547" data-lake-index-type="0"><span class="ne-text">stiffness：刚性，硬度，默认100，值越大越干脆</span></li><li id="u08667a6c" data-lake-index-type="0"><span class="ne-text">damping：反方向阻力，默认10，值越大阻力越大，停的越快</span></li></ol></ol></ol></ol><ol class="ne-list-wrap"><ol class="ne-list-wrap"><ol start="3" ne-level="2" class="ne-ol"><li id="u0644bba2" data-lake-index-type="0"><span class="ne-text">Inertia   模拟惯性衰减的减速效果，用于元素被拖拽后产生的惯性效果上。当给元素添加drag属性时，默认就改成inertia了，不需要单独设置transition，需要修改的属性写在dragTransition上</span></li></ol></ol></ol><ol class="ne-list-wrap"><ol class="ne-list-wrap"><ol class="ne-list-wrap"><ol ne-level="3" class="ne-ol"><li id="ua674a7ea" data-lake-index-type="0"><span class="ne-text">bounceStiffness：类似stiffness，默认500，决定在超出拖拽范围后的回弹，越大越干脆</span></li><li id="ua7e1b275" data-lake-index-type="0"><span class="ne-text">bounceDamping：类似damping，默认10，类似摩擦力，回弹时值越大越阻力越大</span></li></ol></ol></ol></ol><ol start="4" class="ne-ol"><li id="ub8e86ac7" data-lake-index-type="0"><span class="ne-text">父子联动</span></li></ol><ol class="ne-list-wrap"><ol ne-level="1" class="ne-ol"><li id="u24689f9e" data-lake-index-type="0"><span class="ne-text">默认情况下，父子动画是一起进行的，如何控制父子动画的变化呢，例如</span></li></ol></ol><ol class="ne-list-wrap"><ol class="ne-list-wrap"><ol ne-level="2" class="ne-ol"><li id="ua4676900" data-lake-index-type="0"><span class="ne-text">依次开始动画，间隔0.5s，只要父元素的animation中的transation中设置</span><span class="ne-text" style="color: rgb(204, 204, 221); background-color: rgb(22, 24, 32)">staggerChildren</span><span class="ne-text" style="color: rgb(255, 187, 102); background-color: rgb(22, 24, 32)">:</span><span class="ne-text" style="color: rgb(255, 153, 102); background-color: rgb(22, 24, 32)">0.5</span></li><li id="ua31d1760" data-lake-index-type="0"><span class="ne-text">父元素结束后0.5s进行子元素动画：父元素的animation中的transation中设置</span><span class="ne-text" style="background-color: #FBDE28">delayChildren:0.5</span></li><li id="uab901b56" data-lake-index-type="0"><span class="ne-text">父元素结束之后子元素再开始：</span><span class="ne-text" style="background-color: #FBDE28">when:beforeChildren</span></li><li id="u4a7b953d" data-lake-index-type="0"><span class="ne-text">子元素结束之后父元素再开始：when:afterChildren</span></li></ol></ol></ol><ol start="5" class="ne-ol"><li id="u631e0108" data-lake-index-type="0"><span class="ne-text"></span></li></ol></details>
###### 有人说layout这个属性老好使了，那么这个属性啥用呢
motion组件的layout本质上是经过计算变化的具体值，然后通过transform来实现各种变化过渡的效果。

1. 元素本身width，位置发生变化由定位相关的left导致的变化，例如just-content
2. 渲染的顺序，元素会有发生位置变化的动画效果
3. 元素本身的变化，导致父元素改变时，父元素加上layout也有过渡效果

<details class="lake-collapse"><summary id="u0d8f7378"><span class="ne-text">案例一、按钮切换</span></summary><pre data-language="jsx" id="CJTcX" class="ne-codeblock language-jsx"><code>// 一个按钮点击修改div的flex-direction属性从flex-start -&gt; flex-end
// 按理说该div的变化是没有动画的，设置了layout之后，默认就有了动画
// 并且可以设置transation属性进行修改具体的动画

const spring = {
  type: &quot;spring&quot;,
  stiffness: 700,
  damping: 30
};

&lt;div onClick={toggleSwitch}&gt;  // 修改div的flex-direction属性从flex-start -&gt; flex-end
  &lt;motion.div layout transition={spring} /&gt;
&lt;/div&gt;</code></pre></details>
<details class="lake-collapse"><summary id="uc2db0870"><span class="ne-text">案例二、tab栏切换</span></summary><p id="u9d753392" class="ne-p"><span class="ne-text">整体结构，tab栏点击切换，底部下划线过渡效果，main部分内容切入时上移淡入，切换走下移淡出</span></p><p id="u89dac86a" class="ne-p"><img src="https://cdn.nlark.com/yuque/0/2024/gif/28792475/1710837916030-45da5d54-a849-4b58-8471-4b1c7c7cb4c7.gif" width="360" id="u6a4f89e3" class="ne-image"></p><pre data-language="jsx" id="pqiwB" class="ne-codeblock language-jsx"><code>import &quot;./styles.css&quot;;
import { useState } from &quot;react&quot;;
import { motion, AnimatePresence } from &quot;framer-motion&quot;;

const tabs = [
  { icon: &quot;🍅&quot;, label: &quot;Tomato&quot; },
  { icon: &quot;🥬&quot;, label: &quot;Lettuce&quot; },
  { icon: &quot;🧀&quot;, label: &quot;Cheese&quot; },
  { icon: &quot;🥕&quot;, label: &quot;Carrot&quot; },
  { icon: &quot;🍌&quot;, label: &quot;Banana&quot; },
  { icon: &quot;🫐&quot;, label: &quot;Blueberries&quot; },
  { icon: &quot;🥂&quot;, label: &quot;Champers?&quot; },
];

export default function App() {
  const [selectedTab, setSelectedTab] = useState(tabs[0]);

  return (
    &lt;div className=&quot;window&quot;&gt;
      &lt;nav&gt;
        &lt;ul&gt;
          {tabs.map((item) =&gt; (
            &lt;li
              key={item.label}
              className={item === selectedTab ? &quot;selected&quot; : &quot;&quot;}
              onClick={() =&gt; setSelectedTab(item)}
            &gt;
              {`${item.icon} ${item.label}`}
              {/* 下划线，因为设置了layoutId，所以作为一个group进行计算的 */}
              {item === selectedTab ? (
                &lt;motion.div className=&quot;underline&quot; layoutId=&quot;underline&quot; /&gt;
              ) : null}
            &lt;/li&gt;
          ))}
        &lt;/ul&gt;
      &lt;/nav&gt;
      &lt;main&gt;
        &lt;AnimatePresence mode=&quot;wait&quot;&gt;
          &lt;motion.div
            key={selectedTab ? selectedTab.label : &quot;empty&quot;}
            initial={{ y: 10, opacity: 0 }}
            animate={{ y: 0, opacity: 1 }}
            exit={{ y: 10, opacity: 0 }}
            transition={{ duration: 0.2 }}
          &gt;
            {selectedTab ? selectedTab.icon : &quot;😋&quot;}
          &lt;/motion.div&gt;
        &lt;/AnimatePresence&gt;
      &lt;/main&gt;
    &lt;/div&gt;
  );
}</code></pre></details>
<details class="lake-collapse"><summary id="ue73d2251"><span class="ne-text">案例三、list结构排序动画</span></summary><p id="u93455448" class="ne-p"><span class="ne-text">关键是map出来的key，根据key来进行计算的，唯一不变才行，所以不能使用index</span></p><p id="u9bf39208" class="ne-p"><span class="ne-text">有时候，通过transform的变化可能导致元素的子元素有一些扭曲或者奇怪的变化，如果出现了，你可以给子元素设置一个layout属性，但一定要注意，这个子元素也需要是</span><span class="ne-text">👉</span><span class="ne-text">motion组件。</span></p><p id="ud573fbe4" class="ne-p"><img src="https://cdn.nlark.com/yuque/0/2024/gif/28792475/1710839864841-1ce15fe1-a62a-470e-a9a0-e5e1b3131a97.gif" width="360" id="u003de1eb" class="ne-image"></p><pre data-language="tsx" id="JTKQ0" class="ne-codeblock language-tsx"><code>const tabsList = [
  { icon: &quot;🍅&quot;, label: &quot;Tomato&quot; },
  { icon: &quot;🥬&quot;, label: &quot;Lettuce&quot; },
  { icon: &quot;🧀&quot;, label: &quot;Cheese&quot; },
];

&lt;motion.div variants={gridSquareVariants} className=&quot;bg-slate-800 aspect-square flex flex-col justify-center items-center gap-3&quot;&gt;
  {tabs.map(tab =&gt; (
    &lt;motion.div
      {/* key最关键，不能用index */}
      key={tab.label}
      layout
      className=&quot;flex items-center gap-y-4 p-3 bg-white rounded-lg&quot;
      whileHover={{ scale: 1.1 }}
      whileTap={{ scale: 0.9 }}
    &gt;
      &lt;motion.div layout transition={{ duration: 1 }}&gt;{tab.icon}&lt;/motion.div&gt;
    &lt;/motion.div&gt;
  ))}
  {/* 点击切换tab顺序 */}
  &lt;button className=&quot;p-3 bg-red-500 mt-4 text-cyan-400&quot; onClick={() =&gt; {
    const newTabs = [...tabs]
    newTabs.sort(() =&gt; Math.random() - 0.5)
    setTabs(newTabs)
  }}&gt;Shuffle&lt;/button&gt;
&lt;/motion.div&gt;</code></pre></details>
