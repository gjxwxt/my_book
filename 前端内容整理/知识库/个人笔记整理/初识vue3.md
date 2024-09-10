### 一、创建项目
1. pnpm create vite 或者 npm create vite 创建vue3项目

### 二、基础语法
#### 初步差别对比：components引入，props，ref和reactive，emit，computed，watch（监听响应式对象和值区别）
##### ref和reactive区别和使用
```javascript
<script setup>
  // ref 用于将基本类型的数据和引用数据类型（对象）转换为响应式数据，通过 .value 访问和修改。
  // reactive 用于将对象转换为响应式数据，可以直接访问和修改属性，适用于复杂的嵌套对象和数组。
  // ref重新分配一个新对象不会失去响应
  // 目前官网推荐使用 ref() 函数来声明响应式状态
import { ref, reactive } from 'vue'
// 定义
const count = ref(0)
const obj = ref({
  nested: { count: 0 },
  arr: ['foo', 'bar']
})
// 修改
function mutateDeeply() {
  // 以下都会按照期望工作
  count.value = 2
  obj.value.nested.count++
  obj.value.arr.push('baz')
}

const state = reactive({ num: 0 })
state.num = 3
</script>

<template>
  <!-- 模板使用，ref自动脱value -->
  <div @click = "count++">{{ count }}</div>

  <button @click="state.num++">
    {{ state.num }}
  </button>
</template>
```

##### props传值
```vue
<script setup>
// 一个组件需要显式声明它所接受的 props，这样 Vue 才能知道外部传入的哪些是 props，
// 哪些是透传 attribute，
  // 拿透传属性  const attrs = useAttrs()

// 数组方式
defineProps(['foo'])
const props = defineProps(['foo'])
// 对象方式
defineProps({
  title: String,
  likes: Number
})
// default和required
defineProps({
  propA: Number,
  // 多种可能的类型
  propB: [String, Number],
  // 必传，且为 String 类型
  propC: {
    type: String,
    required: true
  },
  // Number 类型的默认值
  propD: {
    type: Number,
    default: 100
  },
  // 对象或数组的默认值
  propE: {
    type: Object,
    // 必须从一个工厂函数返回。
    // 该函数接收组件所接收到的原始 prop 作为参数。
    default(rawProps) {
      return { message: 'hello' }
    }
  },
  // 函数类型
  propF: {
    type: Function,
    // default就是默认的函数
    default() {
      return 'Default function'
    }
  }
})
// ts类型写法
defineProps<{ msg: string }> 
//会被编译为 
defineProps({ 
  msg: { 
    type: String, 
    required: true 
  }
})

// 配合withDefault提高代码简洁程度
const props = withDefaults(defineProps<{ msg: string }>(), {
  msg: '测试'
})
  
// 例如一个表格组件
const props = withDefaults(defineProps<TableProps>(), {
	columns: () => [],
	pagination: true,
	initParam: {},
	border: true,
	toolButton: true,
	selectId: "id",
	searchCol: () => ({ xs: 1, sm: 2, md: 2, lg: 3, xl: 4 })
});
</script>
<template>
    <div>{{ foo }}</div>
</template>

```

##### emit
```vue
<script setup>
  // 数组形式
  const emit = defineEmits(['inFocus', 'submit'])

  // 添加ts
  const emit = defineEmits<{
    (e: 'change', id: number): void
    (e: 'update', value: string): void
  }>()

  function buttonClick() {
    emit('change', 112300225)
  }
</script>
```

##### 组件的v-model，简化props传值和emit事件
```vue
//
<child :modelValue="searchText" @update:modelValue="(val) => searchText = val">
<child v-model="searchText">

<child :searchText="searchText" @update:searchText="(val) => searchText = val">
<child v-model:searchText="searchText">
```

##### computed
```javascript
// 计算属性会自动追踪响应式依赖，
// 返回值为一个计算属性 ref，所以需要.value获取值
const publishedBooksMessage = computed(() => {
  return author.books.length > 0 ? 'Yes' : 'No'
})

// 经常和props使用结合，用作对props的二次加工
```

##### watch
###### source类型![](https://cdn.nlark.com/yuque/0/2024/png/28792475/1709103703874-64973912-2cad-4f82-a081-1311d2671f30.png)
```javascript
// 监听ref
const x = ref(0)
watch(
  x, 
  // () => x.value, //或者getter函数形式
  (newX) => {
    console.log(`x is ${newX}`)
  }
)

// 监听响应式对象的某个值
const obj = reactive({ count: 0 })
const obj = ref({ count: 0 })
watch(
  () => obj.count,// reactive
  // () => obj.value.count, // ref
  (count) => {
    console.log(`count is: ${count}`)
  }
)

// 监听整个响应式对象，默认开启deep
watch(
  obj, // reactive
  //obj.value, // ref
  (newValue, oldValue) => {
  // 嵌套的属性变更时触发
  // 注意：`newValue` 此处和 `oldValue` 是相等的
  // 因为它们是同一个对象！
})

// 监听多个参数
watch([fooRef, barRef], ([foo, bar], [prevFoo, prevBar]) => {
  /* ... */
})


// 第三个参数常见配置
watch(
  source,
  (newValue, oldValue) => {
    // 立即执行，且当 `source` 改变时再次执行
  },
  { 
    immediate: true,
    once: true,
    flush: 'post' | 'pre' | 'sync'
  }
)
● "pre"：（默认值）组件 渲染/更新前 执行
● 'post'：组件 渲染/更新后 执行；能拿到修改后的dom
● 'sync'：强制效果始终 同步 触发（响应式依赖发生改变时立即触发侦听器），官网提示如果有多个属性同时更新，这将导致一些性能和数据一致性的问题，要谨慎使用；
```

##### 常用生命周期：setup，onMounted，onUpdated，onUnmounted
```vue
// vue3
<script setup>     
import { onMounted } from 'vue'
 
onMounted(() => {
  ...
})
// 可将不同的逻辑拆开成多个onMounted，依然按顺序执行，不被覆盖
onMounted(() => {
  ...
})
</script>
 
// vue2
<script>     
   export default {         
      mounted() {             
        ...         
      },           
   }
</script> 
 
```

##### 传值
+ 父传子：props
+ 子传父：emits
+ 跨组件
    - 通过ref拿到<font style="color:#DF2A3F;">组件暴露</font>出来的方法和内容

```plain
// dialog
defineExpose({
  acceptParams
});

// use
const dialogRef = ref(null)
dialogRef.value.acceptParams(params);
```

    - provide和inject：提供一个值，可以<font style="color:#DF2A3F;">被后代组件注入</font>。

```javascript
<script setup>
import { ref, provide } from 'vue'

// 提供静态值
provide('path', '/project/')

// 提供响应式的值
const count = ref(0)
provide('count', count)
</script>



// 后代组件 inject
<script setup>
import { inject } from 'vue'

const path = inject('path')

const count = inject('count')

// 注入一个值，若为空则使用提供的默认值
const bar = inject('path', '/default-path')
</script>


// 也可以全局注入
app.provide()
```

    - attrs属性透传

```vue
<child class="but" value="123" @click="onClick">
<child v-bind="obj" />

const obj = {
  class:"but",
  value="123",
  onClick="onClick"
}
// child  单个根，自动添加到根元素上
<div>
  123  
</div>
defineProps({
  value: string
})
// 编译成，因为value已经被props拿走了
<div class="but" @click="onClick">
  123  
</div>

// 当child 多个根，只能手动选择
<p :class="$attrs.class">
  {{ $attrs.value }}
</p>
<footer>...</footer>

// js使用attrs属性
<script setup>
import { useAttrs } from 'vue'

const attrs = useAttrs()
</script>

// 如果是单根节点，而且没有被子节点props、emit消费，会透传到下级节点
// 如果是多跟节点，也可以手动选择需要透传的值，接着传下去

<sunzi v-bind="$attrs">
<sunzi :class="$attrs.class">
```

    - 全局方法

> app.config.globalProperties.$msg = xxx 
>

```javascript
import { ComponentInternalInstance, getCurrentInstance } from "vue";
const { appContext } = getCurrentInstance() as ComponentInternalInstance;

appContext.config.globalProperties.$msg;
```

    - pinia

##### 响应式本质
+ 响应式是<font style="background-color:#FBDE28;">函数与数据</font>的关系，数据一变函数重新执行，哪些函数和哪些数据之间的关系呢？
+ 被监控的函数：被 vue2的watcher  、vue3的effect  包裹起来的函数，例如watch回调、computed回调、render、watchEffect的回调，内部响应式数据一变，函数重新执行
+ 响应式数据：ref和reactive

```javascript
// 当props.count变了，谁也会变
const doubleCount = ref(props.count * 2)

watchEffect(()=>{
  doubleCount.value = props.count * 2
})

const doubleCount = computed(()=>{
  return props.count * 2
})

function useDouble(count){ 
  const doublecount = ref(count * 2); 
  watchEffect(()= {
    doubleCount.value = count * 2;
  }) 
  return doublecount; 
}
const doublecount = useDouble(props.count);
```

###### 响应式
#### 其他差异
###### 多根节点
Vue3 支持了多根节点组件

```javascript
<template>
  <header>...</header>
  <main>...</main>
  <footer>...</footer>
</template>
```

###### 异步组件
Vue3 提供 Suspense组件，允许程序在等待异步组件时渲染兜底的内容，如 loading ，使用户体验更平滑。使用它，需在模板中声明，并包括两个命名插槽：default和fallback。Suspense确保加载完异步内容时显示默认插槽，并将fallback插槽用作加载状态。

```vue
<tempalte>
   <suspense>
		<template #default>
			<component :is="LayoutComponents[themeConfig.layout]" />
		</template>
		<template #fallback>
			<Loading />
		</template>
	</suspense>
</template>

<script setup lang="ts">
import { computed, defineAsyncComponent, type Component } from "vue";
import { GlobalStore } from "@/stores";

const LayoutComponents: { [key: string]: Component } = {
	vertical: defineAsyncComponent(() => import("./LayoutVertical/index.vue")),
	columns: defineAsyncComponent(() => import("./LayoutColumns/index.vue"))
};

const globalStore = GlobalStore();
const themeConfig = computed(() => globalStore.themeConfig);
</script>
```

###### Teleport
<Teleport> 接收一个 to  来指定传送的目标。to 的值可以是一个 <font style="background-color:#FBDE28;">CSS 选择器</font>字符串，也可以是一个<font style="background-color:#FBDE28;"> DOM 元素对象</font>。

```vue
<template>
  <teleport
    to="body"
  >
    <transition
      name="dialog-fade"
      @after-enter="afterEnter"
      @after-leave="afterLeave"
      @before-leave="beforeLeave"
    >
      ....
    </transition>
  </teleport>
</template>
```

###### diff优化、打包优化、ts支持等等
### 三、路由
#### route定义
```javascript
import { createRouter, createWebHashHistory } from "vue-router";
const staticRouter = [
    {
        path: "/",
        name: "Home",
        component: () => import("@/views/home/index.vue"),
        meta: {
            title: "首页",
            keepAlive: true
        }
    }.....
];

const router = createRouter({
    history: createWebHashHistory(),
    routes: [...staticRouter],
    strict: false,
    // scrollBehavior是用于控制路由切换时页面滚动行为的函数
    scrollBehavior: () => ({ left: 0, top: 0 })
});

/**
 * @description 路由拦截 beforeEach
 * */
router.beforeEach(async (_to, _from, next) => {
    next();
});
....
export default router;
```

```javascript
import router from "@/routers/index";
app.use(router)
```

#### 路由跳转和路由传参
```javascript
// vue-router 关键是useRouter和useRoute
// useRoute返回当前路由的信息对象，包含当前路由的参数、查询、和 hash。
// useRouter返回一个路由实例对象，可以用来导航，beforeEach和afterEach等。
import { useRouter } from 'vue-router';
const router = useRouter();

const toxxx = () => {
  router.push('/person');
  // params形式需要匹配定义路由的path
  // 例如：path:'/person/:keyword?'
  // 跳转后地址栏变为：/#/person/2
  router.push({
    name:'person',
    params:{
      keyword:2
    }
  })
  // query
  // 地址：/#/student?name=张三&age=18
  router.push({
    path:'/student',
    query:{
      age:18,
      name:'张三'
    }
  })
};


// 获取方拿参数
import { useRoute } from 'vue-router';
const route = useRouter();
// route.query
// route.params
```

### 四、pinia
    1. 特点：可以用defineStore定义不同的store，需要哪个引入哪个并实例化，整体结构更像setup，getter和action直接this拿到state以及其他getter，使用方便，实例化之后可以直接拿到定义的方法

```javascript
import { createApp } from "vue";
import App from "./App.vue";
import { createPinia } from "pinia";
const pinia = createPinia();


const app = createApp(App);
app.use(pinia);
app.mount("#app");
```

```javascript
import { defineStore } from "pinia";

// 第一个参数是应用程序中 store 的唯一 id，调用getter不传参不加(),调用actions要加()
export const useUsersStore = defineStore("users", {
  state: () => {
    return {
      name: "小猪",
      age: 25,
      sex: "男",
    };
  },
  getters: {
		 // 普通的getter使用
		getAddAge:(state)=>{
        return state.age+5
      },
		 // 给getter传参时，返回值采用函数的形式，因为一般使用的时候直接store.getAddAge就执行了该函数
    getNumAddAge: (state) => {
      return (num) => state.age + num;
    },
		 // 调用其他getter方法时因为要用this，所以不能用箭头函数的形式
    getNameAndAge() {
      return this.name + this.getAddAge; 
    },
  },
  actions: {
		// 将actions当做methods，主要是this指向当前的store
    change(name) {
      this.name = name;
    },
  },
});
```

```javascript
import { defineStore } from "pinia";
import {ref,computed} from 'vue'
// 这种形式不好使用插件实现本地化存储，可以自己手动实现
export const useUsersStore = defineStore("users", {
	const name = ref('小猪');
	const age = ref(25);
	const sex = ref("男");
	const getAddAge=computed(()=> age.value+5);
	// 用computed的形式不能给getters传参
	const getNameAndAge=computed(()=> name.value+getAddAge.value)
	const change(name){
		name.value=name
	}
return { name,age,sex,getAddAge,getNameAndAge,change }
})
```

```javascript
<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <p>姓名：{{ name }}</p>
	<p>姓名：{{ store.name }}</p>
  <p>年龄：{{ age }}</p>
  <p>性别：{{ sex }}</p>
	<p>使用普通getter：{{ store.getAddAge }}</p>
  <p>给getter传参：{{ store.getNumAddAge(1100) }}</p>
  <p>调用其它getter：{{ store.getNameAndAge }}</p>
  <button @click="changeName">更改姓名</button>
  <button @click="reset">重置store</button> // 重置store为最初的状态
  <button @click="patchStore">批量修改数据</button> 
  <button @click="saveName">调用aciton</button>
	
  <!-- 子组件 -->
  <child></child>
</template>
<script setup lang="ts">
import child from "./child.vue";
import { storeToRefs } from "pinia";
import { useUsersStore } from "../src/store/user";
const store = useUsersStore();
// 为了让使用到的值能随着store中的值变化而变化，采用storeToRefs(),将store中的数据ref包裹
const { name, age, sex } = storeToRefs(store); // 直接展示修改store并非响应式，

const changeName = () => {
  store.name = "张三";
};
// 重置store
const reset = () => {
  store.$reset();
};
// 批量修改数据,建议采用action改变state，
//一是难维护，在组件繁多情况下，一处隐蔽state更改，整个开发组帮你排查；
//二是破坏store封装，难以移植到其他地方。
const patchStore = store.$patch((state)=>{
      state.hasChanged = true 
      state.age=26
})
// 调用actions方法
const saveName = () => {
  store.change("我是小猪");
};
</script>
```

    2. 实现pinia数据持久化，利用pinia-plugin-persistedstate插件实现，也是存在本地缓存

```javascript
// pnpm i pinia-plugin-persistedstate

import { createApp } from "vue";
import App from "./App.vue";
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

const pinia = createPinia();
pinia.use(piniaPluginPersistedstate);
createApp(App).use(pinia);
```

```javascript
import { defineStore } from "pinia";

// 第一个参数是应用程序中 store 的唯一 id，调用getter不传参不加(),调用actions要加()
export const useUsersStore = defineStore("users", {
  state: () => {
    return {
      name: "小猪",
      age: 25,
      sex: "男",
    };
  },
  getters: {
		 // 普通的getter使用
		getAddAge:(state)=>{
        return state.age+5
      },
		 // 给getter传参时，返回值采用函数的形式，因为一般使用的时候直接store.getAddAge就执行了该函数
    getNumAddAge: (state) => {
      return (num) => state.age + num;
    },
		 // 调用其他getter方法时因为要用this，所以不能用箭头函数的形式
    getNameAndAge() {
      return this.name + this.getAddAge; 
    },
  },
  actions: {
		// 将actions当做methods，主要是this指向当前的store
    change(name) {
      this.name = name;
    },
  },
	// 所有数据持久化
  // persist: true,
  // 持久化存储插件其他配置
  persist: {
    // 修改存储中使用的键名称，默认为当前 Store的 id
    key: 'storekey',
    // 修改为 sessionStorage，默认为 localStorage
    storage: sessionStorage,
    // 部分持久化状态的点符号路径数组，[]意味着没有状态被持久化(默认为undefined，持久化整个状态)
    paths: ['name','age'],
  },
});
```

### 五、命令式组件
    3. 主要目的是简化组件使用形式
    4. 例如封装一个消息弹窗组件，使用时直接

```vue
// 传入弹窗的message, 第二个参数返回close函数，当弹窗关闭时执行，该函数
showMsg('这是一个弹窗',(close)=>{
  // 执行操作，执行完毕后调用close方法关闭弹窗
  console.log('点击了确定')
  close()
})

// 普通的组件，每次引入都要做这些事
  先引入
  后定义是否显示
  emit时的触发函数
```

    5. 也就是使用时调用函数

```javascript
import ShowModel from "@/components/model.vue";
import { createVNode, render } from "vue";

export function showMsg(msg, closeHandler) {
    // createApp 方法返回一个应用实例，它提供了 mount 方法，接受一个 DOM 元素作为参数，将应用挂载到该元素上。
    // 参数分别是：组件，组件的 props
  const model = createApp(ShowModel, {
        msg,
        onClose: () => {
            closeHandler?.(() => {
                div.remove()
            })
        }
    })
    const div = document.createElement('div')
    document.body.appendChild(div)
    model.mount(div)
}

```

```javascript
import ShowModel from "@/components/model.vue";
import { createVNode, render } from "vue";

export function showMsg(msg, closeHandler) {
    const model = createVNode(
        ShowModel,
        {
            msg,
            onClose: () => {
                closeHandler?.(() => {
                    div.remove()
                })
            }
        }
    )
    const div = document.createElement('div')
    render(model, div)
    document.body.appendChild(div)
}

```

能不能不用单独用一个文件去单独写组件，直接在一个文件里

```javascript
import { createVNode, h, render } from "vue";

const styles = {
    "model": {
      position: "fixed",
      top: "0",
      left: "0",
      width: "100%",
      height: "100%",
      backgroundColor: "rgba(47, 53, 66, 0.5)",
      display: "flex",
      justifyContent: "center",
      alignItems: "center",
      zIndex: "999"
    },
    "model-item": {
      width: "300px",
      height: "200px",
      backgroundColor: "#fff",
      borderRadius: "10px",
      position: "relative"
    },
    "model-btn": {
      position: "absolute",
      bottom: "0",
      left: "0",
      width: "100%",
      height: "50px",
      display: "flex",
      justifyContent: "space-around",
      alignItems: "center",
      backgroundColor: "#fff",
      borderRadius: "10px"
    },
    "model-btn-item": {
      width: "100px",
      height: "30px",
      lineHeight: "30px",
      textAlign: "center",
      border: "1px solid #000",
      borderRadius: "5px"
    },
    "txt": {
      width: "100%",
      height: "150px",
      lineHeight: "150px",
      textAlign: "center"
    }
  };

const ShowModel = {
  props: {
    msg: {
      type: String,
      required: true
    }
  },
  setup(props, { emit }) {
    return () => {
      return h('div', {  style: styles['model'] }, [
        h('div', {  style: styles['model-item'] }, [
          h('div', { style: styles['txt'] }, props.msg),
          h('div', {  style: styles['model-btn'] }, [
            h('div', {  style: styles['model-btn-item'], onClick: () => emit('close') }, '确定')
          ])
        ])
      ]);
    };
  }
};


export function showMsg(msg, closeHandler) {
    const model = createVNode(
        ShowModel,
        {
            msg,
            onClose: () => {
                closeHandler?.(() => {
                    div.remove()
                })
            }
        }
    )
    const div = document.createElement('div')
    render(model, div)
    document.body.appendChild(div)
}

```

h组件的阅读性太差了，能不能就像react一样直接写render函数一样

1. 使用jsx
    1. npm i @vitejs/plugin-vue-jsx -d
    2. 配置vite.config.ts

```javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueJsx from '@vitejs/plugin-vue-jsx'

export default defineConfig({
  plugins: [vue(), vueJsx()]
})
```

    3. 配置tsconfig.json

```javascript
{
  ...,
  "jsx": "preserve",
  "include": ["src/**/*.jsx"],
}
```

    4. 编写jsx文件，例如之前的showMsg组件

```jsx
const ShowModel = {
  props: {
    msg: {
      type: String,
      required: true
    }
  },
  setup($props, $emit) {
    return () => (
      <div style={styles['model']}>
        <div style={styles['model-item']}>
          <div style={styles['txt']}>{$props.msg}</div>
          <div style={styles['model-btn']}>
            <div style={styles['model-btn-item']} onClick={() => $emit('close')}>确定</div>
          </div>
        </div>
      </div>
    );
  }
};
```

    5. [Vue 3中JSX/TSX最佳实践 - 掘金](https://juejin.cn/post/7291953947974139942)

### 六、vite配置
```typescript
import vue from '@vitejs/plugin-vue';
import vueJsx from "@vitejs/plugin-vue-jsx";
import { resolve } from "path";
import { ConfigEnv, UserConfig, defineConfig, loadEnv } from 'vite';
import compressPlugin from "vite-plugin-compression";


const gz = {
  // 生成的压缩包后缀
  ext: ".gz",
  // 体积大于threshold才会被压缩
  threshold: 0,
  // 默认压缩.js|mjs|json|css|html后缀文件，设置成true，压缩全部文件
  filter: () => true,
  // 压缩后是否删除原始文件
  deleteOriginFile: false
};
// const br = {
//   ext: ".br",
//   algorithm: "brotliCompress",
//   threshold: 0,
//   filter: () => true,
//   deleteOriginFile: false
// };

// https://vitejs.dev/config/
export default defineConfig(({ mode }: ConfigEnv): UserConfig => {
  // process.cwd() 返回Node.js 进程的绝对路径
  // loadEnv 用于加载目录下的.env.[mode]的配置文件，返回一个对象
  // 设置第三个参数为 '' 来加载所有环境变量，而不管是否有 `VITE_` 前缀。
  const viteEnv = loadEnv(mode, process.cwd(), '');
  return {
    base: "./",
    plugins: [vue(), vueJsx(), viteEnv.VITE_COMPRESSION === "true" ? compressPlugin(gz) : null],
    resolve: {
      alias: {
        '@': resolve(process.cwd(), 'src'), 
      },
    },
    css: {
			preprocessorOptions: {
				scss: {
					additionalData: `@import "@/styles/var.scss";`
				}
			}
		},
    server: {
      proxy: {
        '/api': {
          target: 'http://localhost:3000',
          changeOrigin: true,
          rewrite: (path) => path.replace(/^\/api/, '')
        }
      },
    },
    // 开发环境时采用esbuild进行预编译（转为es规范、缓存到deps文件下、统一集成多包）、转换TypeScript、转换Jsx、Tsx
    esbuild: {
      pure: viteEnv.VITE_DROP_CONSOLE === "true" ? ["console.log", "debugger"] : []
    },
    // 生产打包时采用rollup进行打包，
    build: {
      outDir: "dist",
      target: "es2015",
      sourcemap: false, // 默认是false，单独生成sourcemap文件
      reportCompressedSize: false, // gzip 压缩大小报告
      // minify: "esbuild", // 默认是esbuild，如果使用terser，需要安装terser npm i terser -D
      // esbuild 打包更快，但是不能去除 console.log，terser打包慢，但能去除 console.log
      minify: "terser",
      terserOptions: {
        compress: {
          drop_console: viteEnv.VITE_DROP_CONSOLE === 'true' ? true : false,
          drop_debugger: true
        }
      },
      chunkSizeWarningLimit: 1500,
      rollupOptions: {
        treeshake: true,
        output: {
          // Static resource classification and packaging
          chunkFileNames: "assets/js/[name]-[hash].js",
          entryFileNames: "assets/js/[name]-[hash].js",
          assetFileNames: "assets/[ext]/[name]-[hash].[ext]"
        }
      }
    },
    // vite 配置
    // 在文件中使用vite全局变量
    // console.log('__APP_INFO__', __APP_INFO__) // vite.config.ts中define的变量，直接使用
    // console.log('import.meta.env', import.meta.env) // .env相关文件不同mode下的变量
    define: {
      __APP_INFO__: JSON.stringify({
        pkg: {
          name: viteEnv.VITE_APP_NAME,
          version: viteEnv.VITE_APP_VERSION,
          engines: {
            node: viteEnv.VITE_NODE_VERSION,
            pnpm: viteEnv.VITE_PNPM_VERSION
          },
          dependencies: {},
          devDependencies: {}
        },
        lastBuildTime: new Date().toISOString()
      }),
    },
    optimizeDeps:{
      include:['lodash-es']
      // exclude: ['lodash-es']
    }
  }

})
```

```typescript
// vite默认将env文件中VIRE开头的变量注入到import.meta.env对象中

VITE_DROP_CONSOLE = false

VITE_APP_NAME = "tuoray"

VITE_APP_VERSION = "1.0.0"

VITE_NODE_VERSION = "16.15.0"

VITE_PNPM_VERSION = "8.15.4"

__APP_INFO__ = "测试"
```

```javascript
VITE_COMPRESSION = false
```

```javascript
VITE_COMPRESSION = true
```

