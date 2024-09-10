1. echart已封装图表库

[数据可视化技术分享-echarts热门组件 - Powered by Discuz!](http://192.144.199.210/forum.php?mod=forumdisplay&fid=2)

[PPChart - 让图表更简单](http://ppchart.com/#/)

[EChartsDemo集](https://www.isqqw.com/#/homepage)

其他的可视化库：

[Highcharts 演示 | Highcharts](https://www.highcharts.com.cn/demo/highcharts)

[http://analysis.datains.cn/finance-admin/index.html#/chartLib/all](http://analysis.datains.cn/finance-admin/index.html#/chartLib/all)

2. echarts快速入手

[Examples - Apache ECharts](https://echarts.apache.org/examples/zh/index.html)

[Documentation - Apache ECharts](https://echarts.apache.org/zh/option.html#title)

3. 常见属性对应图

![](https://cdn.nlark.com/yuque/0/2022/png/28792475/1661391550905-621a9240-5121-4723-8125-f2cdb62a6bf1.png)

4. 坐标轴的type
+ type:category    ![](https://cdn.nlark.com/yuque/0/2022/png/28792475/1661393028742-ef718778-74d5-4d48-8b36-4ca4404cbbec.png)与相匹配的data进行渲染

    	+ 当设置boundaryGap: false,时，坐标轴变为![](https://cdn.nlark.com/yuque/0/2022/png/28792475/1661393117426-c5175b50-810b-4d79-994d-05df12c85f6e.png)

        + axisTick.alignWithLabel: true，<font style="color:rgb(51, 51, 51);">在 </font><font style="color:rgb(199, 37, 78);background-color:rgb(244, 244, 244);">boundaryGap</font><font style="color:rgb(51, 51, 51);"> 为 </font><font style="color:rgb(199, 37, 78);background-color:rgb(244, 244, 244);">true</font><font style="color:rgb(51, 51, 51);"> 的时候有效，可以保证刻度线和标签对齐</font>![](https://cdn.nlark.com/yuque/0/2022/png/28792475/1661397168222-182bc050-6f69-4ebd-a749-e9eff836b994.png)

+ type:value   ![](https://cdn.nlark.com/yuque/0/2022/png/28792475/1661392114317-47d92248-fb4b-414c-ac82-eb631eab2bb8.png)连续的数值型 

```javascript
  xAxis: {
    type: 'category',
    boundaryGap: false,
    data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
  },
  yAxis: {
    type: 'value'
  },
```

5. series的type
+ line   =>折线图
+ bar   =>柱状图
+ pie   =>饼图

![](https://cdn.nlark.com/yuque/0/2022/png/28792475/1661394735578-99b9cb05-80dc-4e21-8c26-a922b661873e.png)

6. series中有多少项就有对象，每一项都是单独的一个完整的图所需的数据
+ 有一项时：![](https://cdn.nlark.com/yuque/0/2022/png/28792475/1661395517976-8993b262-9d09-40da-8c07-fca731773fab.png)
+ 有两项时：![](https://cdn.nlark.com/yuque/0/2022/png/28792475/1661395552811-142b1686-82d0-4924-b4e3-93faf846ae3a.png)
7. x或者y轴哪个轴有data，数据就沿着哪个轴延展（注意修改type）

```javascript
  xAxis: {
    type: 'category',
    boundaryGap: [0, 0.01],
    data: ['Brazil', 'Indonesia', 'USA', 'India', 'China', 'World']
  },
  yAxis: {
    type: 'value',
  },
```

```javascript
  xAxis: {
    type: 'value',
    boundaryGap: [0, 0.01]
  },
  yAxis: {
    type: 'category',
    data: ['Brazil', 'Indonesia', 'USA', 'India', 'China', 'World']
  },
```

#### 常见案例1（折线与柱状图）
```javascript
option = {
  title: {
    text: 'Stacked Line',
    subtext:"子标题",
		//子标题带的链接，点击进入
		sublink:'http://zh.wikipedia.org/wiki/%E9%A6%99%E6%B8%AF%E8%A1%8C%E6%94%BF%E5%8D%80%E5%8A%83#cite_note-12',
		left: '50',
    top: 0,
    textStyle: {
      color: '#ccc'
    }
  },
  tooltip: {
    trigger: 'item'
  },
  legend: {
    data: ['Email', 'Union Ads']
  },
  grid: {
    left: '3%',
    right: '4%',
    bottom: '3%',
    containLabel: true
  },
  toolbox: {
    show: true,
    feature: {
      dataZoom: {
        yAxisIndex: 'none'
      },
      dataView: { readOnly: false },
      magicType: { type: ['line', 'bar'] },//切换成折线图或者饼图
      restore: {},//变完之后还原
      saveAsImage: {}//保存成图片
    }
  },
  xAxis: {
    type: 'category',
    // boundaryGap: false,
    data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'],
    axisTick: {
      alignWithLabel: true//在 boundaryGap 为 true 的时候有效，可以保证刻度线和标签对齐
    },
		axisLabel: {   //修改该轴线上的label
        rotate: 30//旋转30°
    },
  },
  yAxis: {
    type: 'value',
    axisLabel: {//对y轴的label进行格式修改
      formatter: '{value} °C'
    }
  },
  series: [
    {
      name: 'Email',//放置和legend顺序、名称一致
      type: 'bar',
      // smooth: true，让此数据折线图平滑
      // stack: 'Total',//让此数据数据堆叠
      emphasis: {
        focus: 'item'
      },
      showBackground: true,//柱状图每个柱本身后面的颜色
      backgroundStyle: {
        color: 'rgba(180, 180, 180, 0.2)'
      },
      data: [120, {//对单个数据进行样式设定
          value: 200,
          itemStyle: {
            color: '#a90000'
          }
        }, 101, 134, 90, 230, 210],
      markPoint: {
        data: [
          { type: 'max', name: 'Max' },//对该email中所有数据中最大的进行标记
          { type: 'min', name: 'Min' }//对最小的进行标记
        ]
      },
      markLine: {
        data: [{ type: 'average', name: 'Avg' }]//该email中平均数据进行标记
      }
    },
    {
      name: 'Union Ads',
      type: 'bar',
      // stack: 'Total',
      emphasis: {
        focus: 'series'
      },
      showBackground: true,
      backgroundStyle: {
        color: 'rgba(180, 180, 180, 0.2)'
      },
      data: [220, 182, 191, 234, 290, 330, 310]
    },
  ]
};
```

```javascript
option = {
  xAxis: {
    type: 'category',
    data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'],
    boundaryGap: false,//category类型的轴，让刻度底下就是数据名
  },
  tooltip: {
    trigger: 'axis',
		//鼠标悬停tooltip显示内容的位置自适应
		position: function(point, params, dom, rect, size) {
              // 鼠标坐标和提示框位置的参考坐标系是：以外层div的左上角那一点为原点，x轴向右，y轴向下
              // 提示框位置
              var x = 0; // x坐标位置
              var y = 0; // y坐标位置

              // 当前鼠标位置
              var pointX = point[0];
              var pointY = point[1];
              // 提示框大小
              var boxWidth = size.contentSize[0];
              var boxHeight = size.contentSize[1];

              // boxWidth > pointX 说明鼠标左边放不下提示框
              if (boxWidth > pointX) {
                x = 5;
              } else { // 左边放的下
                x = pointX - boxWidth;
              }

              // boxHeight > pointY 说明鼠标上边放不下提示框
              if (boxHeight > pointY) {
                y = 5;
              } else { // 上边放得下
                y = pointY - boxHeight;
              }
              return [x, y];
            },
  },
  yAxis: {
    type: 'value'
  },
  series: [
    {
      data: [820, 932, 901, 934, 1290, 1330, 1320],
      type: 'line',
      stack: 'Total',//堆叠型
      // areaStyle: {},
       areaStyle: {
        color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
          {
            offset: 0,
            color: 'rgba(58,77,233,0.8)'
          },
          {
            offset: 1,
            color: 'rgba(58,77,233,0.3)'
          }
        ])
      },
      label: {
        show: true,
        position: 'top'
      },
      emphasis: {
        focus: 'series'
      },
      smooth: true
    },
      {
      data: [620, 532, 301, 904, 120, 130, 132],
      type: 'line',
      stack: 'Total',
      // areaStyle: {},//什么都不写也会显示面积颜色
      areaStyle: {//自定义颜色，渐变色
        color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
          {
            offset: 0,
            color: 'rgba(213,153,210,0.8)'//一般渐变色只修改透明度就可以达到
          },
          {
            offset: 1,
            color: 'rgba(213,153,210,0.5)'
          }
        ])
      },
      label: {
        show: true,
        position: 'top'
      },
      emphasis: {
        focus: 'series'
      },
      smooth: true
    }
  ]
};
```

实现样式：

![](https://cdn.nlark.com/yuque/0/2022/png/28792475/1661411008071-6038f0c9-590b-4ea2-ab7d-85cf1cb79125.png)

![](https://cdn.nlark.com/yuque/0/2022/png/28792475/1662359025731-d3143b79-fc28-49f9-baac-94371f7596bd.png)

#### 常见案例2（饼状图）
```javascript
option = {
  // backgroundColor: '#2c343c',//整个图标背景颜色
  title: {
    text: 'Customized Pie',
    left: 'center',
    top: 20,
    textStyle: {
      color: '#ccc'
    }
  },
   tooltip: {
    trigger: 'item',
    formatter: '{a} <br/>{b} : {c} ({d}%)',   //a饼图name，b块name，c块value，d块百分比
		showDelay: 0,  //显现出来的延迟时间
		transitionDuration: 0.5  //过渡到另一块之后移动过去的延迟时间
  },
  legend:{
    left:0,
    top:20,
    orient: 'vertical',//竖向排列，不设置默认是横向
  },
  visualMap: {
    show: false,//虽然看不见角落小图，但是会对数据块进行颜色标注
    min: 80,   
    max: 600,
		text: ['High', 'Low'],//分别是小块上的字和下面的字
    inRange: {
      colorLightness: [0, 1],//颜色的明暗度，适用于给整个图所有块设置同一个颜色之后根据数据不同用明暗度区分，数据越大越亮
      symbolSize:[30, 100]//小方块底部到顶部的大小，呈过渡状
      // color: ['#121122', 'rgba(3,4,5,0.4)', 'red'],
    }
    //outOfRange:与inRange相反，排除某些颜色
  },
  series: [//饼状图只有一个对象
    {
      name: 'Access From',
      type: 'pie',
      radius: '55%',//饼图的半径，（容器高宽中较小一项）的 55% 长度
      // radius:[50,250],圆环,第一个内圆半径,第二个外圆半径
      center: ['50%', '50%'],//设置饼图在图表中的位置,50%代表正中心
      hoverAnimation:false, //关闭饼图hover在扇区上的放大动画效果。适用于圆环
      data: [
        { value: 335, name: 'Direct' },
        { value: 310, name: 'Email' },
        { value: 274, name: 'Union Ads' },
        { value: 235, name: 'Video Ads' },
        { value: 400, name: 'Search Engine' }
      ].sort(function (a, b) {
        return a.value - b.value;
      }),
      emphasis: {//当聚焦在某个块上时的样式设定
        itemStyle: {
          shadowBlur: 10,
          shadowOffsetX: 0,
          shadowColor: 'rgba(0, 0, 0, 0.5)'
        }
      },
      // roseType: 'area',//item的值作为扇形的面积，角度每块都一样，但大小不一样
      roseType:'redius',//item的值作为扇形的半径
      // label: {//描述饼块的文字
      //   color: 'rgba(255, 255, 255, 0.3)'
      // },
      // labelLine: {//字和饼块连接的线
      //   lineStyle: {
      //     color: 'rgba(255, 255, 255, 0.3)'
      //   },
      //   smooth: 0.2,
      //   length: 10,//第一段距离长度
      //   length2: 20//第二段距离长度
      // },
      itemStyle: {
        color: '#c23531',
        shadowBlur: 200,//整个饼图的阴影模糊状态
        shadowColor: 'rgba(0, 0, 0, 0.5)',
        borderRadius:8//每个角设置圆角
      },
      animationType: 'scale',
      animationEasing: 'elasticOut',
      animationDelay: function (idx) {
        return Math.random() * 200;
      }
    }
  ]
};
```

![](https://cdn.nlark.com/yuque/0/2022/png/28792475/1661406992629-de95fa6a-c16d-4a07-9551-f744806ec8cd.png)

#### 常见案例3（地图）
```javascript
//获取地图信息之后，注册地图并且添加配置项
//三步走：registerMap()注册地图，option.series.data放置数据以及完善配置项，setOption(option)添加配置项,
getMapData().then(geoJson=>{
	//registerMap的参数有三个，第三个可以省略，
	//分别为注册的地图名称(mapName)，和地图json(geoJSON)，最后为对地图json特殊地区的修改(specialAreas)
	//第三个参数例子：
		 // {
		 //        澳门: {
		 //          left: 113,// 左上角经度
		 //          top: 20.5,// 左上角纬度
		 //          width: 0.7,//横跨的经度
		 //        },
		 //        香港: {
		 //          left: 115,
		 //          top: 21.3,
		 //          width: 2,
		 //        },
		 //      }
	echarts.registerMap('HK', geoJson);
  myChart.setOption(
		(option = {
      title: {
        text: 'Population Density of Hong Kong （2011）',
      },
      tooltip:{},
      toolbox:{},
      visualMap: {
        min: 800,
        max: 50000,
        text: ['High', 'Low'],
        realtime: false,
        // calculable: true,
        inRange: {
          color: ['lightskyblue', 'yellow', 'orangered']
        }
      },
      series: [
        {
          name: '香港18区人口密度',
          type: 'map',
          map: 'HK',//同 registerMap 方法的第一个参数一致
          label: {
						color：#fff
            show: true//每块的名字都会显示在所属的块上
          },
          roam: true,//能拖拽放大
          itemStyle: {
            normal: {
                borderWidth: .5, //区域边框宽度
                borderColor: '#009fe8', //区域边框颜色
                areaColor: "#1a364b", //区域颜色，如果已经设置了visualMap就不生效
            },
            emphasis: {
                borderWidth: .5,
                borderColor: '#4b0082',
                areaColor: "#ffdead",
            }
        	},
          //地图内部有各地区的名称，name和地区名一致时自动匹配，如果给的数据name和地图内部默认的name不一致
          //需要自定义名称映射nameMap{},key为地图默认name，value为我们自己数据的name
          data: [ 
            { name: '中西区', value: 20057.34 },
            { name: '湾仔', value: 15477.48 },
            { name: '东区', value: 31686.1 },
            { name: '南区', value: 6992.6 },
            { name: '油尖旺', value: 44045.49 },
            { name: '深水埗', value: 40689.64 },
            { name: '九龙城', value: 37659.78 },
            { name: '黄大仙', value: 45180.97 },
            { name: '观塘', value: 55204.26 },
            { name: '葵青', value: 21900.9 },
            { name: '荃湾', value: 4918.26 },
            { name: '屯门', value: 5881.84 },
            { name: '元朗', value: 4178.01 },
            { name: '北区', value: 2227.92 },
            { name: '大埔', value: 2180.98 },
            { name: '沙田', value: 9172.94 },
            { name: '西贡', value: 3368 },
            { name: '离岛', value: 806.98 }
          ],
          // 自定义名称映射，用于地图里面的name和数据name不一致的情况
          nameMap: {
            'Central and  Western': '中西区',
            Eastern: '东区',
            Islands: '离岛',
            'Kowloon City': '九龙城',
            'Kwai Tsing': '葵青',
            'Kwun Tong': '观塘',
            North: '北区',
            'Sai Kung': '西贡',
            'Sha Tin': '沙田',
            'Sham Shui Po': '深水埗',
            Southern: '南区',
            'Tai Po': '大埔',
            'Tsuen Wan': '荃湾',
            'Tuen Mun': '屯门',
            'Wan Chai': '湾仔',
            'Wong Tai Sin': '黄大仙',
            'Yau Tsim Mong': '油尖旺',
            'Yuen Long': '元朗'
          }
        }
      ]
    })
  );
}

```

![](https://cdn.nlark.com/yuque/0/2022/png/28792475/1661411956754-d8ba51a5-a16b-47cb-88b7-f7f877487809.png)

#### 常见案例4（地图下钻与回退）
```javascript
<template>
  <div style="position:relative;width: 500px;height: 500px;">
    <div class="hello" style="width:500px;height:500px;background-color: black;position: absolute;z-index: 1;"></div>
    <div id="back" style="position:absolute;z-index:2;color:red;bottom: 25px;right: 25px;" v-show="show">返回</div>
  </div>
</template>

<script>
  import mapOption from '@/components/options/mapOption'
  import {getMapJson,getMapData} from '@/api/getmap.js' 
  import ChineseHelper from '@/utils/ConvertPinyin.js' 
  import relation from '@/utils/codeToArea'
  let option = mapOption;
  //用于上行，保留父级的code和data（series.data）
  let areaCode='370000';//'100000'
  let dataPool=[];
  let areaLevel='china';
  let areaName='china';//进来默认是china，点击之后会修改，在dataPool中记录原来的值，点击返回时传入原来的areaName值进行渲染
export default {
  name: 'HelloWorld',
  props: {
    code:String,//拿着从主页传过来的code去找地图的json，但是还要知道是省级还是市级，之后
    token:String
  },
  data() {
    return {
      chart:null,
      show:false,
    }
  },
  created(){
    // areaCode=this.code;
  },
  mounted(){
    this.changeCode(areaCode);
    this.init();
    this.render(areaCode,areaLevel,areaName,false);
    this.bindEvent();
  },
  methods:{
     /**
     * @description: 用于解决一开始只传入行政编码code，来查找相关的地区
     * @param {String} areaCode 
     */
    changeCode(areaCode){
      if(areaCode=='10000'){
        //  areaLevel='china';
        //  areaName='china';
      }else{
        let codeStr=areaCode.slice(2);
       let area = relation(areaCode);
       areaName = ChineseHelper.ConvertPinyin(area);
        if(codeStr=='0000'){
           areaLevel='province';
        }else{
          areaLevel='city'
        }
      }
      console.log(areaName)
    },

    init(){
      const chart=document.querySelector('.hello');
      this.chart=this.$echarts.init(chart);
    },
    bindEvent(){
      let that=this;
      const back = document.querySelector('#back');
      this.chart.on('click',that.areaClick );
      back.addEventListener('click',that.back)
    },

    async changeOption(){
      this.chart.showLoading();
      if( areaCode == '10000') {
         this.show=false;
         option.series[0].map=areaName;
         await  getMapData(areaCode).then(res=>{  
          option.series[0].data=res.data.data;
        })
      }
      else{
        this.show=dataPool.length!==0?true:false;
        areaName = ChineseHelper.ConvertPinyin(areaName);
        option.series[0].map=areaName;
        await  getMapData(areaCode).then(res=>{
          option.series[0].data=res.data.data;
        })
      }
    },

    async render(areaCode,areaLevel){
      let that = this;
      await this.changeOption();//修改series.data和series.name
      getMapJson(areaCode,areaLevel).then(res=>{
        that.$echarts.registerMap(areaName,res.data);
      }).then(()=>{
        that.chart.setOption(option,true);
        that.chart.hideLoading();
      })
    },



    //点击拿着该地区的name换code之后，拿着code获取data修改option并且拿着code获取地图的json文件去注册
    async areaClick(params){   
        if(areaLevel!='city'){
          let arr={};
           arr.areaCode = areaCode;
           arr.areaName = areaName;
           arr.areaLevel=areaLevel;
           dataPool.push(arr)
        }
        //入栈code和series.data，只记录一二级,
        let name= params.name;//用name去获取code，用code再去找地图json
        areaName=name;
         //获取点击地区的code
        await getMapJson(areaCode,areaLevel).then(res=>{
             let item = res.data.features.find(item=>item.properties.name==name);
             areaCode=item.properties.adcode;
             areaLevel=item.properties.level;
         });
        this.render(areaCode,areaLevel);
    },

    //点击返回按钮，拿dataPool中的code和level获取地图信息和已存的data进行渲染
    back(){
      let data=dataPool.pop();
      areaName=data.areaName;
      areaCode=data.areaCode;
      areaLevel=data.areaLevel;
      this.render(areaCode,areaLevel);
    }
  }
}
</script>

```

# <font style="color:rgb(34, 34, 38);">echarts中setOption第二个参数的作用:</font>
配置项改变，想覆盖之前的配置而不是与之前的配置合并，第二个参数就必须设为true，false为默认合并

如果不设置容易出问题：第二个配置项比之前的配置项少了一项，修改后不会导致图标的刷新，因为默认合并了

```javascript
import echarts from 'echarts'   //5.x以下
import * as echarts from 'echarts'  //5.x以上

Vue.prototype.$echarts = echarts
```

### 原生图表的生成
#### 1. 饼图
```javascript
<template>
  <div>
    <canvas id="HolkWhIjURlZmXWtHwpgmYjNMSqkEQqs" class="charts" @click="tap"/>
  </div>
</template>

<script>
import uCharts from '@/js_sdk/u-charts/u-charts.js';
var uChartsInstance = {};
export default {
  data() {
    return {
      cWidth: 750,
      cHeight: 500
    };
  },
  onReady() {
    this.getServerData();
  },
  methods: {
    getServerData() {
      //模拟从服务器获取数据时的延时
      setTimeout(() => {
        //模拟服务器返回数据，如果数据格式和标准格式不同，需自行按下面的格式拼接
        let res = {
            series: [
              {
                data: [{"name":"一班","value":50},{"name":"二班","value":30},{"name":"三班","value":20},{"name":"四班","value":18},{"name":"五班","value":8}]
              }
            ]
          };
        this.drawCharts('HolkWhIjURlZmXWtHwpgmYjNMSqkEQqs', res);
      }, 500);
    },
    drawCharts(id,data){
      const ctx = document.getElementById('HolkWhIjURlZmXWtHwpgmYjNMSqkEQqs')
				.getContext("2d");
      uChartsInstance[id] = new uCharts({
        type: "pie",
        context: ctx,
        width: this.cWidth,
        height: this.cHeight,
        series: data.series,
        animation: true,
        timing: "easeOut",
        duration: 1000,
        rotate: false,
        rotateLock: false,
        background: "#FFFFFF",
        color: ["#1890FF","#91CB74","#FAC858","#EE6666","#73C0DE","#3CA272","#FC8452","#9A60B4","#ea7ccc"],
        padding: [5,5,5,5],
        fontSize: 13,
        fontColor: "#666666",
        dataLabel: true,
        dataPointShape: true,
        dataPointShapeType: "solid",
        touchMoveLimit: 60,
        enableScroll: false,
        enableMarkLine: false,
        legend: {
          show: true,
          position: "bottom",
          float: "center",
          padding: 5,
          margin: 5,
          backgroundColor: "rgba(0,0,0,0)",
          borderColor: "rgba(0,0,0,0)",
          borderWidth: 0,
          fontSize: 13,
          fontColor: "#666666",
          lineHeight: 11,
          hiddenColor: "#CECECE",
          itemGap: 10
        },
        extra: {
          pie: {
            activeOpacity: 0.5,
            activeRadius: 10,
            offsetAngle: 0,
            labelWidth: 15,
            border: true,
            borderWidth: 3,
            borderColor: "#FFFFFF",
            customRadius: 0,
            linearType: "none"
          },
          tooltip: {
            showBox: true,
            showArrow: true,
            showCategory: false,
            borderWidth: 0,
            borderRadius: 0,
            borderColor: "#000000",
            borderOpacity: 0.7,
            bgColor: "#000000",
            bgOpacity: 0.7,
            gridType: "solid",
            dashLength: 4,
            gridColor: "#CCCCCC",
            fontColor: "#FFFFFF",
            splitLine: true,
            horizentalLine: false,
            xAxisLabel: false,
            yAxisLabel: false,
            labelBgColor: "#FFFFFF",
            labelBgOpacity: 0.7,
            labelFontColor: "#666666"
          }
        }
      });
    },
    tap(e){
      uChartsInstance[e.target.id].touchLegend(e);
      uChartsInstance[e.target.id].showToolTip(e);
    }
  }
};
</script>

<style scoped>
  .charts{
    width: 750px;
    height: 500px;
  }
</style>
```

