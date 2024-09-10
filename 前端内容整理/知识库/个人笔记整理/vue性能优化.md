#### 1. 函数式组件加快渲染速度
```javascript
//函数式组件渲染开销低，因为函数式组件只是函数，入参是渲染上下文（render context），返回值是渲染好的HTML
// Stateless(无状态)：组件自身是没有状态的
// Instanceless(无实例)：组件自身没有实例，也就没有this

//因为函数式组件没有状态，所以不需要像vue的响应式系统一样需要经过额外的初始化。
//函数式组件仍然会做出响应式变化，比如新传入props，但是在组件本身中，它无法知道数据什么时候发生变化，因为它不维护自身状态。

// 适用场景：

// 一个简单的展示组件，也就是所谓的dumb组件，例如button组件，pills，tags，cards，甚至整个页面都是静态文本，比如about页面
// “高阶组件”用于接收一个组件作为参数，返回一个被包装过的组件
// v-for循环中每项通常都是很好的候选项。
```

```javascript
// 子组件（函数式组件，用于展示数据，不产生实例没有this、对props有响应式）
<template functional>
    <div class="list">
        <div class="item" v-for="item in props.list" :key="item.id" @click="props.itemClick(item)">
            <p>{{item.title}}</p>
            <p>{{item.content}}</p>
        </div>
    </div>
</template>

// 父组件（传值、监听点击事件）
<template>
    <div>
        <List :list="list" :itemClick="item => (currentItem = item)" />
    </div>
</template>
import List from '@/components/List.vue'
export default {
    components: {
        List
    },
    data() {
        return {
            list: [{
                title: 'title',
                content: 'content'
            }],
            currentItem: ''
        }
    }
}
```

#### 2. 少修改this，采用局部变量的形式（直接解构this），因为多次this.xxx执行多次getter，如果只需要触发一次getter，就可以将getter返回的值储存在局部变量中，便不会多次触发了
```javascript
// 要比直接用this.n,this.m快
data(){
	return{
		n:5,
		h:255
	}
},
computed(){
	m(){
		return h^3
	}
}
mounted(){
	console.time('AAA')
	this.subtotal(this)
	console.timeEnd('AAA')
},
methods:{
	subtotal({ n , m }){
			let result = n;
			for(let i=0 ; i<999999 ; i++){
				result += Math.sqrt(Math.cos(Math.sin(m)))；
				return result;
			}
	}
}

// computed使用局部变量
props: ['start'],
computed: {
	base () {
		return 42
	},
	result ({ base, start }) {
		let result = start
		for (let i = 0; i < 1000; i++) {
			result += Math.sqrt(Math.cos(Math.sin(base))) + base * base + base + base * 2 + base * 3
		}
		return result
	},
},
```

#### 3. 长列表的性能优化vue-virtual-scroll-list实现虚拟列表
+ 安装并作为局部组件引入、设置总体样式、传入数据、创建items组件、设置样式即可

```javascript
npm install vue-virtual-scroll-list --save
	
 <template>
  <div>
	 // 样式（整个虚拟列表的展示框）、唯一的key值、总数据、展示某一项的小组件
    <virtual-list
      style="height: 360px; overflow-y: auto; width: 500px" 
      :data-key="'id'"
      :data-sources="items"  //传入的数据列表
      :data-component="itemComponent"  // item组件
			 keeps:"20" // 真实DOM的个数，默认为30
			 estimate-size："50" // 每个item的高，可以更精准计算总高度
			 extra-props:extraData// 用于传入的一些其他props数据 
    >
    </virtual-list>
  </div>
</template>
   
<script>
		import virtuallist from "vue-virtual-scroll-list"; 
		import Items from "./Items";// 内部小组件
		export default {
		  name: "List",
		  components: [virtuallist],
		  props: {
		    msg: String,
		  },
		  data() {
		    return {
		      itemComponent: Items,// 虚拟DOM
		      items: [],// 唯一标识+数据  用来传给内部组件
		    };
		  },
		  methods: {
				// 创造数据
		    dataSource() {
		      for (let i = 0; i < 9999; i++) {
		        this.items.push({
		          id: i,
		          name: "模拟字段" + i,
		        });
		      }
		    },
		  },
		  created() {
		    this.dataSource();
		  },
		};
</script>
```

```javascript
<template functional>
  <div>{{ props.index }}---{{ props.source.name }}</div>
</template>
     
<script>
	export default {
	  name: "Items",
	};
</script>
```

#### 4. 返回大量数据给data里时如果只是展示，不需要数据劫持，可以将数据Object.freeze()
#### 5. 图片懒加载  
```javascript
methods:{
	//拿到新数据，添加到list中=》生成dom元素进行监听=》出现在视图加载真实src
		async query(){
			let result = await this.$api.trending();
			result = result.map((item)=>{
				return object.freeze(item.object.data);//因为数据只是展示用，不会进行修改
			}):
			this.list.push(...result);
			this.visible = true;
			//获取img dom元素并且进行监听
			this.SnextTick(()=>{
				this.$refs["pics"].forEach((item)=> this.ob.observe(item)):
			}):
		}
}
mounted(){
		this.ob = new IntersectionObserver(
			(changes)=>{
				changes.forEach(({isIntersecting , target })=>{
					if (!isIntersecting) return;
		      //出现在视口中：加载真实图片
					const img = target.querySelector("img");
					if (!img)return;
					img.src = img.getAttribute("data-src");
					img.onload = ()=> img.style.opacity = 1;
					this.ob.unobserve(target);
			}):
		},{ threshold:[1] }):
		//滚动到底部加载更多数据
		this.obLoading = new IntersectionObserver(([item])=>{
				if (item.isIntersecting){
					//滚动到底部了
				  this.query();
					this.ob.unobserve(item);
				}
		)
		this.obLoading.observe(this.$refs ["loading"]);
}
```

#### 6. 域名发散：主页面和静态资源要置于不同的域名下（可能是相同的服务器也可以不是）
    1. 浏览器对同一域名下<font style="color:#DF2A3F;">请求的并发数量</font>有所限制，若超出最大限制数，超出的请求会被阻塞。浏览器会等待前面的资源下载完毕，然后复用前面连接的 tcp 发起后续的请求。减少主域名下的请求，达到减少网页白屏时间或者DCL、L时间的效果
    2. Cookie 是紧跟域名的。<font style="color:#DF2A3F;">同一个域名下的请求都会携带 Cookie</font>。可想而知，如果我们所有资源都在一个域名下，那么加载那些静态资源也会带上cookie，很明显加大了请求成本。
    3. 静态资源独立部署，为全局产品服务。<font style="color:#DF2A3F;">方便复用</font>，放在一个服务器上的文件可以共其他服务器上的产品使用。 比如taobao.com和tmll.com都会用到tbcdn.cn上的静态资源，这些资源不必从属于某个产品。而且如果taobao.com下载了某个js资源文件，访问tmll.com如果再次利用到该js文件，直接利用缓存即可。
    4. 将静态内容和动态请求存放在不同服务器上，更加<font style="color:#DF2A3F;">方便进行 CDN 请求</font>。但是也不能域名太多，因为dns解析也需要时间，一般主站和cdn两个域名比较合理

#### 7. 一次性渲染很多重组件导致的白屏
1. 用v-if来控制组件的渲染，将一帧渲染的元素分成多帧渲染。

```javascript
import { onUnmounted, ref } from 'vue';
export function useDefer(max = 100){
  let frameCount = ref(0);
  function updateFrameCount(){
    let rafId = requestAnimationFrame(()=>{
      frameCount.value++;
      if(frameCount.value >= max){
        return;
      }
      updateFrameCount();
    })
  }
  updateFrameCount();
  onUnmounted(()=>{
    cancelAnimationFrame(rafId);
  })
  return function defer(n){
    return frameCount.value >= n;
  }
}
```

#### 其他
1. umy-ui
2. 封装一个自己的虚拟列表滚动组件

