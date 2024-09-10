```javascript
//校验数据类型
export const typeOf = function(obj) {
  return Object.prototype.toString.call(obj).slice(8, -1).toLowerCase()
}

//防抖，执行一次先清除定时器再设一个定时器
export const debounce = (() => {
  let timer = null
  return (callback, wait = 800) => {
    timer&&clearTimeout(timer)
    timer = setTimeout(callback, wait)
	}
})()

//节流，方式一、函数一开始执行设定定时器，定时器结束之后才能再次触发该函数
//方式二、函数一开始用一个变量接着执行开始时间，没执行一次函数会和上次执行的时间进行对比，如果过了设置的间隙，执行并且改变变量为当前时间
export const throttle = (() => {
	let last=0;
  return (callback, wait = 800) => {
    let now = +new Date()
    if (now - last > wait) {
      callback()
      last = now
  }
})()

//手机号脱敏
export const hideMobile = (mobile) => {
  return mobile.replace(/^(\d{3})\d{4}(\d{4})$/, "$1****$2")
}  //使用hideMobile('12345678978')=>'123****8978'

//开启全屏
export const launchFullscreen = (element) => {
  if (element.requestFullscreen) {
    element.requestFullscreen()// 正常浏览器 
  } else if (element.mozRequestFullScreen) {
    element.mozRequestFullScreen()//早期火狐浏览器
  } else if (element.msRequestFullscreen) {
    element.msRequestFullscreen()//早期IE浏览器
  } else if (element.webkitRequestFullscreen) {
    element.webkitRequestFullScreen()// webkit 
  }
}//使用xxx.onclick=launchFullscreen(document.querySelector('#container'))

//关闭全屏
export const exitFullscreen = () => {
  if (document.exitFullscreen) {
    document.exitFullscreen()
  } else if (document.msExitFullscreen) {
    document.msExitFullscreen()
  } else if (document.mozCancelFullScreen) {
    document.mozCancelFullScreen()
  } else if (document.webkitExitFullscreen) {
    document.webkitExitFullscreen()
  }
}

//大小写转换，type=1为全部大写，type=2为全部小写，type=3首字母大写，注意传入的是字符串
export const turnCase = (str, type) => {
  switch (type) {
    case 1:
      return str.toUpperCase()
    case 2:
      return str.toLowerCase()
    case 3:
      return str[0].toUpperCase() + str.substring(1).toLowerCase()
    default:
      return str
  }
}

//解析当前页面的url地址
export const getSearchParams = () => {
	//window.location.search获取url链接中包含？在内的以后得所有字符串
  const searchPar = new URLSearchParams(window.location.search)
  const paramsObj = {}
  for (const [key, value] of searchPar.entries()) {
    paramsObj[key] = value
  }
  return paramsObj
}//假设目前位于 https://****com/index?id=154513&age=18;
//getSearchParams(); // {id: "154513", age: "18"}

//判断手机是安卓还是ios
export const getOSType=() => {
  let u = navigator.userAgent, app = navigator.appVersion;
  let isAndroid = u.indexOf('Android') > -1 || u.indexOf('Linux') > -1;
  let isIOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/);
  if (isIOS) {
    return 1;
  }
  if (isAndroid) {
    return 2;
  }
  return 3;
}

//判断当前是pc/moblie
export const detectDeviceType = () => {
	return /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent) ? 'Mobile' : 'Desktop'; 
};


//滚动到页面顶部，会有过渡时间
export const scrollToTop = () => {
  const height = document.documentElement.scrollTop || document.body.scrollTop;
  if (height > 0) {
    window.requestAnimationFrame(scrollToTop);
    window.scrollTo(0, height - height / 8);
  }
}//scrollToTop()


// 滚动到页面中某个元素的位置
export const smoothScroll = element =>{
    document.querySelector(element).scrollIntoView({
        behavior: 'smooth'
    });
};//smoothScroll('#target')=>于是页面平滑滚动到id为target元素的位置

// 生成从min到max的随机整数
export function randomNumInteger(min, max) {
    return parseInt(Math.random() * (max - min + 1) + min, 10);
}

//校验是否为整数
export function validateInt(num){
	return num % 1 == 0 ? true : false
}

//校验是否为姓名（四位中文）
export function validatename(name) {
    var regName = /^[\u4e00-\u9fa5]{2,4}$/;
		return regName.test(name) ? true : false
}

//校验邮箱格式
export function validateEmail(email) {
    const re = /^(([^<>()\\[\]\\.,;:\s@"]+(\.[^<>()\\[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
    return re.test(email)
}

//获取前七天的日期(不包含今天) ['2022-09-06', '2022-09-07', '2022-09-08', '2022-09-09', '2022-09-10', '2022-09-11', '2022-09-12']
export function get7days() {
      const myDate = new Date(); //获取今天日期
      myDate.setDate(myDate.getDate() - 7);
      const flag = 1;
			let arr=[];
      for (var i = 0; i < 7; i++) {
        let m = "";
        let d = "";
        if (myDate.getMonth() + 1 < 10) {
          m = "0" + (myDate.getMonth() + 1); //补齐
        } else {
          m = myDate.getMonth() + 1;
        }
        if (myDate.getDate() < 10) {
          d = "0" + myDate.getDate(); //补齐
        } else {
          d = myDate.getDate();
        }
        let dateTemp = myDate.getFullYear() + "-" + m + "-" + d;
        arr.push(dateTemp);
        myDate.setDate(myDate.getDate() + flag);
      }
			return arr
}
```

```javascript
				// 完备的深拷贝，推荐使用 lodash 中的 cloneDeep 方法
						// const _ = require('lodash');
						// const obj1 = {
						//     a: 1,
						//     b: { f: { g: 1 } },
						//     c: [1, 2, 3]
						// };
						// const obj2 = _.cloneDeep(obj1);
export const clone = parent => {
  // 判断类型
  const isType = (obj, type) => {
    if (typeof obj !== "object") return false;
    const typeString = Object.prototype.toString.call(obj);
    let flag;
    switch (type) {
      case "Array":
        flag = typeString === "[object Array]";
        break;
      case "Date":
        flag = typeString === "[object Date]";
        break;
      case "RegExp":
        flag = typeString === "[object RegExp]";
        break;
      default:
        flag = false;
    }
    return flag;
  };

  // 处理正则
  const getRegExp = re => {
    var flags = "";
    if (re.global) flags += "g";
    if (re.ignoreCase) flags += "i";
    if (re.multiline) flags += "m";
    return flags;
  };
  // 维护两个储存循环引用的数组
  const parents = [];
  const children = [];

  const _clone = parent => {
    if (parent === null) return null;
    if (typeof parent !== "object") return parent;

    let child, proto;

    if (isType(parent, "Array")) {
      // 对数组做特殊处理
      child = [];
    } else if (isType(parent, "RegExp")) {
      // 对正则对象做特殊处理
      child = new RegExp(parent.source, getRegExp(parent));
      if (parent.lastIndex) child.lastIndex = parent.lastIndex;
    } else if (isType(parent, "Date")) {
      // 对Date对象做特殊处理
      child = new Date(parent.getTime());
    } else {
      // 处理对象原型
      proto = Object.getPrototypeOf(parent);
      // 利用Object.create切断原型链
      child = Object.create(proto);
    }

    // 处理循环引用
    const index = parents.indexOf(parent);

    if (index != -1) {
      // 如果父数组存在本对象,说明之前已经被引用过,直接返回此对象
      return children[index];
    }
    parents.push(parent);
    children.push(child);

    for (let i in parent) {
      // 递归
      child[i] = _clone(parent[i]);
    }

    return child;
  };
  return _clone(parent);
};
```

```javascript
// 第一个参数就是需要四舍五入的值，第二个参数就是要保留小数点以后多少位
function toFixed(num, digits = 0) {
  let zeroStrNum = num.toString()
  
  // 处理科学计算情况
  if(zeroStrNum.includes("e")){
     const m = zeroStrNum.match(/\d(?:\.(\d*))?e([+-]\d+)/);
    zeroStrNum = num.toFixed(Math.max(0, (m[1] || '').length - m[2]))
  }
  
  let isNegativeNum = false
  // 判断是否为负数
  if(zeroStrNum.startsWith('-')){
    isNegativeNum = true
    zeroStrNum = zeroStrNum.slice(1)
  }
  // 获取小数点位置
  const dotIndex = zeroStrNum.indexOf('.')
  // 如果是整数/保留小数位数等于超过当前小数长度，则直接用toFixed返回
  if(dotIndex === -1 || (zeroStrNum.length - (dotIndex + 1) <= digits)){
    return num.toFixed(digits)
  }

  // 找到需要进行四舍五入的部分
  let numArr = zeroStrNum.match(/\d/g) || [];
  numArr = numArr.slice(0, dotIndex + digits + 1)
 
  // 核心处理逻辑
  if (parseInt(numArr[numArr.length - 1], 10) > 4) {
    // 如果最后一位大于4，则往前遍历+1
    for (let i = numArr.length - 2; i >= 0; i--) {
      numArr[i] = String(parseInt(numArr[i], 10) + 1);
      // 判断这位数字 +1 后会不会是 10
      if (numArr[i] === '10') {
        // 10的话处理一下变成 0，再次for循环，相当于给前面一个 +1
        numArr[i] = '0';
      } else {
        // 小于10的话，就打断循环，进位成功
        break;
      }
    }
  }
  // 将小数点加入数据
  numArr.splice(dotIndex, 0, ".")
  
  // 处理多余位数
  numArr.pop()
  
  // 如果事负数，添加负号
  if(isNegativeNum){
    numArr.unshift("-")
  }
  
  return Number(numArr.join('')).toFixed(digits)  
}

// 返回1.26
toFixed(1.255, 2)
```

```javascript
function randomNum(n){ 
	    let t=''; 
	    for(let i=0;i<n;i++){ 
	        t+=Math.floor(Math.random()*10); 
	    } 
	    return t; 
} 
```

时间字符串的处理=》[安装 | Day.js中文网](https://dayjs.fenxianglu.cn/category/)

```javascript
// 金额的千分位转换，obj不传即默认千分位，传'en'前带美元符号，传'ch'前带中文符号
export const thousandFormat = (num, obj) => {
            let param = '';
            if (!obj) {
                //格式化千分位输出
                param = num.toLocaleString();
            } else if (obj && obj.lang === 'en') {
                //格式化为千分位带$符号输出
                param = num.toLocaleString('en', { style: 'currency', currency: 'USD' });
            } else if (obj && obj.lang === 'cn') {
                //格式化为带￥符号输出
                param = num.toLocaleString('cn', { style: 'currency', currency: 'CNY' });
            } else {
                //格式化千分位输出
                param = num.toLocaleString();
            }
            return param;
  }//使用 thousandFormat(2500,{lang:'en'})=》$2,500.00
```

