##### vue3setup中如何给组件添加name
1. 写两个script标签，一个setup一个没有setup
2. 使用插件   <font style="color:rgb(18, 18, 18);">vite-plugin-vue-setup-extend</font>

<font style="color:rgb(18, 18, 18);">1、安装</font>

<font style="color:rgb(18, 18, 18);">npm i vite-plugin-vue-setup-extend -D</font>

<font style="color:rgb(18, 18, 18);">2、配置 ( vite.config.ts )</font>

```javascript
import { defineConfig } from 'vite' 
import VueSetupExtend from 'vite-plugin-vue-setup-extend' 
export default defineConfig({ plugins: [VueSetupExtend()]}) 
```

<font style="color:rgb(18, 18, 18);">3、使用</font>

<font style="color:rgb(18, 18, 18);"><script lang="ts" setup name="demo"> </script></font>

##### vue中的provide和inject，如果传的值是this等可监听的对象，使用inject的组件可以修改provide组件中的值
```javascript
// provide传值
export default{
	provide(){
		return{
			comp:this
		}
	},
	data(){
		return{
			show:true
		}
	}
}
// 修改传来的值，并且provide组件中的该值同时改变
export default{
	inject:['comp'],
	mounted(){
		this.comp.show=false
	}
}
```

##### 在父、子组件中同时使用了window.onresize，子组件中window.onresize无效。
此时需要把window.onresize改为window.addEventListeners

##### 爷孙插槽的使用
```vue
<!-- 使用 -->
<father>
	<template #default>
		<button>使用</button>
	</template>
</father>

// father使用child组件
<child></child>

// child组件，用$parent和$scopedSlots拿到father页面的插槽信息，然后render成组件即可
<template>
	<option-content></option-content>	
</template>
<script>
export default{
  name:'child',
  componentName:'child',
  components:{
    OptionContent: {
      render(h) {
        const getParent = vm => {
          if (vm.$options.componentName === 'child') {
            return vm;
          } else if (vm.$parent) {
            return getParent(vm.$parent);
          } else {
            return vm;
          }
        };
        const child = getParent(this);
        const father = child.$parent || child;
        return father.$scopedSlots.default() # default是爷页面使用插槽的名字
      }
    }
  },
</script>
```

```vue
<!-- 作用域插槽，使用孙页面数据 -->
<father>
	<template #abc="{ option }"> # option是孙页面的数据
		<button>{{option.label}}</button>
	</template>
</father>

// father使用child组件
<child></child>

<template>
	<option-content :option="{label:'第一'}"></option-content>	
</template>
<script>
export default{
  name:'child',
  componentName:'child',
  components:{
    OptionContent: {
			props: {
        option: Object
      },
      render(h) {
        const getParent = vm => {
          if (vm.$options.componentName === 'child') {
            return vm;
          } else if (vm.$parent) {
            return getParent(vm.$parent);
          } else {
            return vm;
          }
        };
        const child = getParent(this);
        const father = child.$parent || child;
        return father.$scopedSlots.abc({option:this.option})
      }
    }
  },
</script>
```

##### **<font style="color:rgb(38, 38, 38);">vue中静态资源的引入机制</font>**
1. 以相对路径和导入方式引入时，webpack会进行处理
+ <font style="color:rgb(38, 38, 38);">诸如 <img src="..."> 、 background: url(...) 和 CSS @import 的资源</font>
+ <font style="color:rgb(38, 38, 38);">例如， url(./image.png) 会被翻译为 require('./image.png')</font>
2. <font style="color:rgb(38, 38, 38);">引用public 目录下或以绝对路径方式。不会经过 webpack 的处理，</font>

```vue
<img src="./assets/images/01.jpg" alt=""> // √
// 编译后:  <img src="/img/01.f0cfc21d.jpg" alt="">

<img :src="'./assets/images/02.jpg'" alt=""> // ×
// 编译后:  <img src="./assets/images/02.jpg" alt="">

<img :src="require('./assets/images/03.jpg')" alt=""> // √
<img :src="require('./assets/images/'+ this.imgName +'.jpg')" alt=""> // √
// 编译后:  <img src="/img/03.ea62525c.jpg" alt="">

//用绝对路径引入时，路径读取的是public文件夹中的资源，
//任何放置在 public 文件夹的静态资源都会被简单的复制到编译后的目录中，而不经过 webpack特殊处理。
// 此时需要注意的是，要和部署时的地址相匹配
<img :src="this.publicPath + 'images/05.jpg'" alt=""> // √
// 编译后:  <img src="/foo/images/05.jpg" alt="">


// 图片懒加载，只有在出现在界面的时候才将url赋值给src,一般这个url是个字符串，所以在打包的时候
// 需要将url进行webpack处理才能正确引用图片，就可以采用require('../assets/img/1.png')
<img v-lazy="require('../assets/gt2.png')"  />
// 编译后：  <img src="/img/gt2.bc0c2c89.png">
```

##### v-bind传props和直接传props的区别
```javascript
<child num="3" />  ==>props:{num:{type:String}}
<child :num="3" /> ==>props:{num:{type:Number}}
```

##### 动态style和class
```javascript
:style="{color:state == true ? 'red' : 'black'}"
:style="{fontSize:`${size}px`}"

:class="['name', state == true ? 'success' : '']"

注意类名只能是单个单词或_连接
success-text 不行
success_text 可以
```

##### 伪元素中的content不能直接与var变量结合，需结合定时器
```javascript
// template
<div :style="{'--content':`${percentage}`}"><b></b></div>  // --content代表的值需要是字符串

// css
b:after {
  counter-reset: box_content var(--content);  // 设置定时器，默认值是变量--content
  content: counter(box_content)"%";  // 使用定时器中的值，并且后面加了个%
}

// 以为可以直接使用,但除了content都可以使用
b:after{
	content: var(--content);   // 失败
}
.progress-65:before {
  transform: rotate(var(--imgDeg3)) translate(0, -72.5px); // 这些方式可以使用
}
```

##### vue3中使用jsx语法
1. create-vue 和 Vue CLI 都有预置的 JSX 语法支持，vite创建的项目可以手动配置

```javascript
// 首先安装 pnpm i @vitejs/plugin-vue-jsx -D
// 在vite.config.js文件中配置
import vue from "@vitejs/plugin-vue";
import vueJsx from "@vitejs/plugin-vue-jsx";

export default defineConfig{
  plugins:[vue(),vueJsx()]  // 添加上vueJsx插件
}
```

2. 使用
    1. 第一种是直接创建jsx文件，例如demo.jsx

```javascript
import { defineComponent,ref } from 'vue'
import MyComponent from './MyComponent.jsx'

export default defineComponent({
  let flag = ref(false);
	let num = ref(0);

  setup(){
    return ()=>{

      <div onClick={()=>flag.value = !flag.value}>{flag.value ? <div>yes</div> : <span>no</span>}</div>
      
      <input
        onClickCapture={() => {}}
        onKeyupOnce={() => {}}
        onMouseoverOnceCapture={() => {}}
      />
      
      <span style={{color: "red"}}></span>

    	// 默认插槽
      <MyComponent>{() => 'hello'}</MyComponent>
      
      // 具名插槽,子组件内部通过调用函数来引用插槽，执行时传入参数就是函数传参
      <MyComponent num={num.value} onUpdate:num={data=>num.value = data}>{{
        default: () => 'default slot',
        foo: props => <div>{props}</div>,
        bar: props => [<span>{props}</span>, <span>two</span>]
      }}</MyComponent>
  }
})
```

```javascript
import { defineComponent,ref } from 'vue'

// 函数式组件   直接返回jsx语法
function component(props, { slots, emit, attrs }){
  return {
    <div>msg:{props.msg}</div>
    <div onClick={()=>props.fun()}>按钮</div>
  }
}

// jsx组件   返回函数
export default defineComponent({
  props:['num'],
  emits:['update:num'],
  setup(props,{slots,emit}){
      return ()=>{
          <div>
        			<div onClick={()=>emit('update:num',30)}>{props.num}</div>
              <div>{slots.default()}</div>
              <div>{slots.foo('传参2')}</div>
              <div>{slots.bar('传参3')}</div>
              <component msg="abc" fun={()=>console.log(1)} />
        	</div>
      }
  }
})
```

    2. 第二种是在vue文件中使用jsx

```vue
<template>
	<div>
    <Child>
  </div>
</template>
<script lang="jsx" setup>
  let arr = [{
    status: 1,
    name: '张三'
  },{
    status: 1,
    name: '李四'
  }];
  function Child(){
    return <>
      {
        arr.map((item)=>{
          if(item.status ==l || item.name='王五'){
          		return
          }else{
            return 
              <tr>
                <td>{item.name}</td>
              </tr>
          })
      }
    <>
  }
</script>
```

vue

d

s

12123c  

ds













##### vue2种components中使用render函数生成组件
```vue
<template>
  <div :style="{ opacity: number / 300 }">
    <ChildComp/>
  </div>
</template>

<script>
export default {
  components: {
    ChildComp: {
      methods: {
        heavy () {
          const n = 100000
          let result = 0
          for (let i = 0; i < n; i++) {
            result += Math.sqrt(Math.cos(Math.sin(42)))
          }
          return result
        },
      },
      render (h) {
        return h('div', this.heavy())
      }
    }
  },
  props: ['number']
}
</script>
```

<font style="color:rgb(0, 0, 0);">Vue 的更新是组件粒度的，如果子组件中没有父组件更新引起的响应式变化，就不会去更新子组件。如果不封装成组件，那么每次渲染都会将未封装成组件的部分进行更新</font>

##### 文件放置在 assets 和 static 的区别
<font style="color:rgb(36, 41, 46);">npm run build时会将</font><font style="color:#DF2A3F;">assets</font><font style="color:rgb(36, 41, 46);">中放置的静态资源文件进行</font><font style="color:#DF2A3F;">打包压缩</font><font style="color:rgb(36, 41, 46);">最终也都会放置在static文件中，而放置在static中的文件不会被压缩，所以将项目中template需要的</font><font style="color:#DF2A3F;">样式文件js文件</font><font style="color:rgb(36, 41, 46);">等都可以放置在assets中，走打包这一流程。减少体积。而项目中引入的</font><font style="color:#DF2A3F;">第三方的资源文件</font><font style="color:rgb(36, 41, 46);">如iconfoont.css等文件可以放置在static中，因为这些引入的第三方文件已经经过处理，我们不再需要处理</font>

##### scope实现<font style="color:rgb(53, 53, 53);">样式隔离</font>
通过css的属性选择器，当使用scoped时会给组件增加上唯一的属性data-v-xxx

```vue
<style scoped>
  
.a{ /*...*/}  ==>  .a[data-v-xxx]

.a >>> .b { /* ... */ }   ==> .a[data-v-f3f3eg9] .b { /* ... */ }
  
</style>


// less或者sass中解析不了>>>，可以使用::v-deep，或者 /deep/
<style lang="less" scoped>
.demo {      ==> .demo a[data-v-219e4e87]
  a {
    color: red;
  } 
}

.demo {      ==> .demo[data-v-ca3944e4] a
  /deep/ a {
    color: red;
  } 
}
</style>
```

##### ts中type和interface异同点
相同：

1. 定义对象、函数、类的类型

```typescript
interface Iperson{
    name:string;
    getAge(age:number):void;
}

type Tperson = {
    name:string;
    getAge(age:number):void;


  class N implements Iperson{
    name='s'
    getAge(age:number){

    }
}

class NA implements Tperson{
    name='sp'
    getAge(age:number){

    }
}
```

2. 支持泛型、可拓展

```typescript
interface Iperson<name>{
    name:name;
    getAge(age:number):void;
}

type Tperson<name> = {
    name:name;
    getAge(age:number):void;
}

interface Su extends Iperson<string>{
    age:string
}

type Pan = Tperson<string> & {
    age:string
}
```

不同

1. type可定义普通类型，泛型不行 

```typescript
type MyBoolean = boolean;
type StringOrBoolean = string | MyBoolean;
type Collect = StringOrBoolean[];
```

2. type不可同名，interface同名自动继承

```typescript
interface Person {
  name: string;
}

interface Person {
  age: number;
}

let user: Person = {
    name:'sp';
    age:28
}
```

##### vueUse中常用composition API
1. useDebounceFn和useThrottleFn    实现防抖、节流

```javascript
import { useDebounceFn } from '@vueuse/core'

const debouncedFn = useDebounceFn(() => {
  // do something
}, 1000)

window.addEventListener('resize', debouncedFn)
```

2. useInfiniteScroll  无限滚动

```javascript
<script setup lang="ts">
import { ref } from 'vue'
import { vInfiniteScroll } from '@vueuse/components'

const data = ref([1, 2, 3, 4, 5, 6])

function onLoadMore() {
  // 给data中push值
}
</script>

<template>
  <div v-infinite-scroll="onLoadMore">
    <div v-for="item in data" :key="item">
      {{ item }}
    </div>
  </div>

  <!-- with options -->
  <div v-infinite-scroll="[onLoadMore, { 'distance' : 10 }]">
    <div v-for="item in data" :key="item">
      {{ item }}
    </div>
  </div>
</template>
```

3. useVirtualList   虚拟列表



