### 一、基本配置
1. worker_processes worker_processes 1; 默认为1，表示开启一个业务进程
2. worker_connections worker_connections 1024; 单个业务进程可接受连接数
3. include mime.types; 引入http mime类型,响应头的类型用于告诉浏览器如何解析该文件，配置文件中是映射表关系，例如css结尾的文件对应 text/css，不可能全部概括
4. default_type application/octet-stream; 如果mime类型没匹配上，默认使用二进制流的方式传输。
5. sendfile on; sendfile on; 使用linux的 sendfile(socket, file, len) 高效网络传输，也就是数据拷贝。一般而言：nginx启动之后会向操作系统注册某个端口只要有这个端口的请求就转发给nginx，当浏览器向服务器发送请求，nginx根据配置文件寻找并read读取请求的该文件到nginx应用程序中，再发送给操作系统的网络接口，过程中会被层层缓存再发送出去，如果开启sendfile，nginx不会去加载缓存该文件，直接让网络接口去缓存并发送该文件。减少了向nginx中数据拷贝的过程
6. keepalive_timeout 65; 保持长连接的时间，动态时间，在时间内有请求就会重置keepalive保持时间
7. server配置虚拟主机

```nginx
server {

	# server_name+listen+location就是一个站点
	# server_name用于匹配域名，即使多个域名对应一个ip地址，nginx会根据域名返回相关内容
	# server_name后可加多个域名，空格隔开
	# 域名匹配优先精确匹配之后是头通配之后是尾通配之后是正则，自上而下，匹配上直接返回不继续往下走
	# 如果所有的server_name都没有匹配上域名，则会返回第一个站点
	# 站点匹配支持正则，~开头代表开启正则  例如server_name ~^[0~9]+\.gao\.cn$;

	listen 80; # 监听端口号
	server_name localhost; # 域名、主机名，localhost也就是127.168.0.0.1
	# http://abc.com/
	location / { # 匹配域名后面的部分如果是/，就命中当前

		root html; # 去nginx config文件夹同级目录下html文件夹下寻找
		index index.html index.htm; #如果文件夹下没有命中的文件则默认返回这些文件
	}
	# 当服务器错误时访问 http://abc.com/50x.html
	error_page 500 502 503 504 /50x.html; # 报错编码对应页面，访问地址
	location = /50x.html {  
		root html; # 去html文件夹下寻找50x.html
	}
}
```

### 二、本机hosts文件作用
1. 是ip与域名的映射表，浏览器输入域名后首先在本机hosts文件中查找有无对应，再去dns服务器进行域名解析，如果有对应直接访问相应ip地址，省去了dns解析的过程
2. 局域网中要记很多IP地址，不好记的话可以在hosts文件中加个对应的好记得域名
3. 要屏蔽一些网站时，将要屏蔽的域名对应上其他的ip地址即可

### 三、负载均衡策略
1. 轮询

```nginx
upstream test {
	# 最后的;不要忘写，后面可跟配重weight=1，用于轮询次数的权重分配，端口号要写，因为多端口容易混
	# down 表示停止访问该主机 backup 表示该主机处于备用状态，当其他所有服务器都挂掉的时候返回该备用服务器
	server 192.168.154.130:80; 
	server 192.168.154.129:80;
}
server {
	listen       80;
	server_name  www.gaojiaxiang.cn;

	location / {
		# proxy_pass 主要是配置代理服务器ip或服务器组的名字,和下面两个配置互斥
		proxy_pass http://test;	# 当访问该站点时返回的站点，用于反向代理
		# root   /www/80;
		#  index  index.html index.htm;
	}
	
	# 浏览器类型代理。适用于根据你的浏览器类型去跳转，例如安卓就下载安卓版，win就下载win版
	location /down {
		# 如果是ie浏览器、chrome浏览器就访问xxx
	    if ($http_user_agent ~ "MSIE") {
	        proxy_pass...
	    }
	    if ($http_user_agent ~ "Chrome") {
	        proxy_pass...
	    }
		#如果是手机端访问，就访问xxx页面
			if($http_user_agent ~* "(Android|iPhone|Windows Phone|UC|Kindle)"){
				proxy_pass...
			}
	}
	error_page   500 502 503 504  /50x.html;
	location = /50x.html {
		root   html;
	}

}
```

2. ip_hash：根据客户端的ip地址转发同一台服务器，可以保持回话，但是用户换ip就无法保持会话了，或者类似学校、网吧这种局域网中用户很多，ip地址是一个，都打到一个服务器中。容易造成流量倾斜，但方便快，或者后端服务器宕机了，用户会话也就消失了。初期中小型快速部署可以使用，

```nginx
upstream test {
	ip_hash    # 采用ip_hash的方式进行分配
	server 192.168.154.130:80; 
	server 192.168.154.129:80;
}
```

3. uri_hash：根据用户访问的uri+hash定向转发请求，同个uri+hash进入同一个服务器，适用于服务器之间存储着不同的资源，nginx拿到uri进行hash算出所在的服务器，在一开始存储到哪台服务器的时候就是利用hash算法算出要存在哪台服务器。或者像移动端无法发送cookie，可以将cookie放在uri上以参数的形式传给nginx，同一个uri连接同一台服务器来实现保存会话

```nginx
upstream test {
	hash $request_uri    # 采用uri_hash的方式进行分配,$代表用到了nginx的变量
	server 192.168.154.130:80; 
	server 192.168.154.129:80;
}
```

4. cookie_hash：根据用户登陆后存储在本地的cookie来保证同一个用户一直访问同一台服务器，保证会话状态，可以解决ip_hash中局域网中大量访问的问题，因为每个用户都不同

```nginx
upstream test {
	# jsessionid代表对cookie中的JSESSIONID进行hash，一般是tomcat下java程序产生
	hash $cookie_jsessionid     # 可以是别的cookie的key值，但是要保证用户的唯一性
	server 192.168.154.130:80; 
	server 192.168.154.129:80;
}
```

5. fair：根据后端服务器响应时间转发请求。这几个都不常用，因为无法实现动态服务器上下
6. 动静分离：中小型企业、传统项目，系统加速，将静态资源css、js、image文件放到nginx服务器上，把tomcat服务器上的css、js、image删除，本来是要将这些请求发送到tomcate服务器上，为了减轻服务器和网络负担，直接让nginx返回静态资源即可，关键在于location 匹配到这些请求，nginx服务器和tomcat服务器处理静态性能差不多，但是tomcat首次建立会话会验证session，之后会保持keep alive，复用之前的链接·

```nginx
# 将css文件夹放到/usr/local/nginx/static下，或其他文件夹只要配置能找到就行
# 只要是请求css、js、image文件夹下的内容就捕获进行返回
location /css {
	root /usr/local/nginx/static;
	index index.html index.htm;
}
# 将js文件夹放到/usr/local/nginx/static下
location /js {
	root /usr/local/nginx/static;
	index index.html index.htm;
}
# 将image文件夹放到/usr/local/nginx/static下
location /images {
	root /usr/local/nginx/static;
	index index.html index.htm;
}
# 上面需要配置三次，可以用正则实现一个配置
# 正则匹配，开启正则之前匹配高于/，正则使用之后低于/优先级
location ~*/(css|img|js) {
	root /usr/local/nginx/static;
	index index.html index.htm;
}
```

### 四、url_rewrite
1. 目的：减少url长度，避免暴露文件目录或者传参形式
2. 实现：nginx捕捉到在转到真正的地址

```javascript
flag标记说明：
  last #本条规则匹配完成后，继续向下匹配新的location URI规则
  break #本条规则匹配完成即终止，不再匹配后面的任何规则
  redirect #返回302临时重定向，浏览器地址会显示跳转后的URL地址，不会完全转，站点仍是代理前的
  permanent #返回301永久重定向，浏览器地址栏会显示跳转后的URL地址
```

```nginx
location / {
	# 此时访问本http://192.168.154.100/1.html,
	# 真实访问应该为http://192.168.154.101/index.jsp?pageNum=1
	rewrite ^/([0-9]+).html$ /index.jsp?pageNum=$1 break;
	proxy_pass http://192.168.154.101;	# 当访问该站点时返回的站点，用于反向代理
}
```

### 五、防盗链
1. 原理：当请求返回的页面中引用了某些资源，便会发起二次请求，浏览器会自动给这些请求中加上referer:该站点，防盗链用于让服务器认为我这是合法的请求，是你返回页面里的请求，不是非法引用
2. 不一定非要配置，更多站点引用代表着更多流量，除非是资源比较稀缺

```javascript
valid_referers none | blocked | server_names | strings ....;
  none， #检测Referer头域不存在的情况。
  blocked，#检测Referer头域的值被防火墙或者代理服务器删除或伪装的情况。这种情况该头域的值不以
    #"http://”或“https://"开头
  server_names ，#设置一个或多个URL.用于检测Referer头域的值是否是这些URL中的某一个
```

```nginx
location ~*/(css|img|js) {
	# 如访问192.168.154.101/css/index.css
	# 只允许浏览器加的referer为该站点才能访问，如果右键打开某资源也不允许访问，
	# 因为直接访问该资源不会带上referer
	valid_referers www.gaojiaxiang.cn; 

	# 没有refer或者referer为该站点才能访问，也就代表着可以直接访问该资源，
	# 但是别的网站的二次请求会带上referer也就不能请求了
	# valid_referers none www.gaojiaxiang.cn;

	if ($invalid_referer) {
		return 403;
	}

	root /usr/local/nginx/static;
	index index.html index.htm;
}
```

3. curl用于上传下载文件，可以用此来检测是否能访问到某资源

```nginx
# 下载插件
yum install -y curl
# 不会将资源展示出来，只会展示返回头信息
curl -I http://192.168.44.101/img/logo.png
#  带上referer去访问该资源
curl -e "http://baidu.com" -I http://192.168.44.101/img/logo.png
```

4. 一个电脑可以配多个网卡，一个网卡能配置多个ip，可以一个网卡连内网一个网卡连外网

### 六、keepalived实现ip漂移
1. 目的：一台nginx代理后台多台服务器，当nginx服务器挂掉之后所有的外网就都访问不到后端服务器了，如何实现多台nginx同时代理呢，可以使用虚拟ip，假如两台nginx作为一组，两个利用keepalived使用同一个虚拟ip，对外暴露虚拟ip，当一个nginx服务器挂掉之后另一台nginx服务器接受该虚拟ip，实现外网访问ip不变，但是nginx服务器实现切换，但是keepalived是检测服务器上keepalived进程是否挂掉，如果服务器没宕机，只是nginx服务器down掉了，keepalived不会漂移走，他以为这台nginx服务器上没挂，不代表nginx程序没挂，所以可以通过脚本检测nginx进程，如果nginx程序挂了，可以将keepalived进程挂掉，实现nginx进程挂ip飘走
2. 安装yum install keepalived // 配置文件在   /etc/keepalived/keepalived.conf 
3. 配置

```nginx
! Configuration File for keepalived
global_defs {
	router_id lb111
}
vrrp_instance abc {     # 同一组nginx服务器使用同一个实例，abc是实例名称
		state MASTER              # 每台nginx服务器之间state不同
		interface ens33           # ip addr 查看nginx服务器的网卡类型
		virtual_router_id 51       
		priority 100              # 竞选权重，即同一组服务器都在线时谁接手虚拟ip
		advert_int 1              # 间隔检测的时间
		authentication {          # 内网中要组成一组该配置要一致
		auth_type PASS
			auth_pass 1111
		}
	virtual_ipaddress {
		192.168.44.200        # 对外暴露的虚拟ip地址，可以填多个
	}
}
```

```nginx
! Configuration File for keepalived
global_defs {
    router_id lb110
}
vrrp_instance abc {            # 同一个实例
  state BACKUP                 # 更改名称
  interface ens33
  virtual_router_id 51
  priority 50                  # 优先级与第一台不同
  advert_int 1
  authentication {             # 与第一台机器一致
    auth_type PASS
    auth_pass 1111
  }
  virtual_ipaddress {
    192.168.44.200
  }
}
```

4. 使用

systemctl start keepalived   // 启动keepalived 

systemctl status keepalived  // 查看keepalived运行状态 

ip addr  可以查看到是哪台机器在使用着虚拟ip 

### 七、https协议
#### 非对称加密：
公钥加密私钥解密，私钥签名公钥验签，公钥加密公钥解不了密。

签名就是私钥加密摘要算法形成的摘要，形成数字签名。

验签就是用公钥解密出摘要再恢复成明文与已有的明文进行对比

#### 对称加密：
数据传输过程中，加密解密采用同一个密匙，运算快

#### https采用非对称加密传输密钥，再用对称加密解密速度快安全性高
**过程**：浏览器拿到服务器公钥a，随机生成一个用于对称加密的密钥x，公钥加密密钥x传给服务器端，服务器用私钥解密出对称加密的密钥x，之后双方所有数据都通过密钥X加密解密即可。

**问题**：如果拦截了服务器传来的公钥a，换成坏人的公钥b，浏览器用坏人的公钥b加密密钥x也被坏人拦截了，坏人用私钥b解密出密钥x，再用公钥a加密之后传给服务器端，此时坏人也知道了密钥x，信息也就暴露了

**解决**：我要明确知道给我的公钥就是请求的服务器端给的公钥a，不是你坏人的公钥b。服务端使用https协议前会向CA申请数字证书，由两部分组成：明文和数字签名。明文中其中包含证书持有者信息、公钥信息、网站域名等。数字签名的生成就是采用非对称加密产生，浏览器本身会保存信任的CA结构的公钥来解密数字签名验证明文。

数字签名加密过程：

+ CA机构对证书明文数据T进行hash即通过摘要算法简化内容。（证书信息较长hash之后缩小加密的内容，毕竟非对称加密比较慢）
+ CA机构对hash后的值用私钥加密，得到数字签名S。

浏览器拿到数字证书后解密验证：

+ 浏览器得到明文T，签名S。
+ 浏览器用CA机构的公钥对S解密，得到S’。
+ 用证书里指明的hash算法对明文T进行hash得到T’。
+ T’应当等于S‘，除非明文或签名被篡改。所以此时比较S’是否等于T’，证书可信于是拿到公钥a并安心传输

**解决拦截问题：**如果服务器返回的数字证书被拦截了，坏人发了一个自己的网站的数字证书，浏览器解密后验证明文和证书一致了，会去访问坏人的网站吗，不会的，因为证书中会包含域名信息，浏览器一看我请求的服务器地址和你证书上的不一致也不会跳转的。

**信任问题**：CA签发的证书分DV、OV、EV三种，区别在于可信程度。DV是最低的，只是域名级别的可信，背后是谁不知道。EV是最高的，经过了法律和审计的严格核查，可以证明网站拥有者的身份（在浏览器地址栏会显示出公司的名字，例如Apple、GitHub的网站），如何保证这些CA是可信的呢？**信任链**的问题。

+ 小一点的CA可以让大CA签名认证，但链条的最后，也就是Root CA，就只能自己证明自己了，这个就叫“自签名证书”（Self-Signed Certificate）或者“根证书”（Root Certificate）。你必须相信，否则整个证书信任链就走不下去了。上网的时候只要服务器发过来它的证书，就可以验证证书里的签名，顺着证书链（Certificate Chain）一层层地验证，直到找到根证书，就能够确定证书是可信的，从而里面的公钥也是可信的。
+ 存在问题隐患：
    - 如果CA失误或者被欺骗，签发了错误的证书，虽然证书是真的，可它代表的网站却是假的。
    - CA被黑客攻陷，或者CA有恶意，因为它（即根证书）是信任的源头，整个信任链里的所有证书也就都不可信了。

[https://www.cnblogs.com/xrxc/p/15210997.html](https://www.cnblogs.com/xrxc/p/15210997.html)

[https://www.suibibk.com/topic/693503017139306496/](https://www.suibibk.com/topic/693503017139306496/)

### 八、 证书安装
第一步需要看nginx是否安装了ssl的模块，如果没有，需要配置上单独添加上 --with-http_stub_status_module --with-http_ssl_module  编译即可

申请完证书之后，将下载文件放在/nginx/conf文件夹下

```nginx
server{
  listen  443 ssl; # https协议默认进入443接口，采用ssl协议
  server_name gaojiaxiang.cn;   
  ssl_certificate  xxx.pem;   # 填刚放进来的文件中的.pem文件
  ss1_certificate_key   xxx.key;  # 填刚放进来的文件中的.key文件
}
```



### 九、实现跨域
#### 1. 跨http链接
```nginx
# 文件中请求链接是 /api/xxx
location ^~/api {
	  rewrite  ^.+api/(.*)$  /$1 break;  # 将链接中的api去掉
		proxy_pass http://www.baidu.com;   # 代理到需要代理的网址
}
```

### 十、细节注意事项：
#### 1. location之间的匹配顺序
```basic
精确匹配：以=开头，只有完全匹配才能生效，例子location = /uri

非正则匹配：以^~开头，^表示非、~表示正则，例子location ^~ /uri，abc/20/js可匹配上abc/ /js

正则匹配：
以~开头，表示区分大小写的正则匹配，例子location ~ pattern
以!~开头，表示区分大小写不匹配的正则，例子location !~ pattern
以~*开头，表示不区分大小写的正则匹配，例子location ~* pattern
以!~*开头，表示不区分大小写不匹配的正则，例子location !~* pattern
普通匹配：不带任何修饰符，例子location /uri、location /
把普通匹配和非正则匹配看作前缀匹配，也就是介于精准匹配和正则匹配的一种匹配
```

先**精准匹配**，匹配成功就结束，匹配失败开始**所有的前缀匹配**，遍历完之后记录下最匹配的，如果最匹配的是非正则匹配，匹配成功结束，如果前缀匹配没有匹配的或者最匹配的那个不是非正则匹配就开始**顺序正则匹配**，正则匹配成功就成功，正则匹配失败就采用之前记录的最匹配的，如果没有就匹配失败。不会匹配失败，因为普通匹配中至少/是匹配的。

图示：

![](https://cdn.nlark.com/yuque/0/2022/png/28792475/1666235940888-580632fa-7be0-4732-8f1d-64cd1c8bbe31.png)

#### 2. proxy_pass 端口号后面的/
当proxy_pass的端口号后面没有"/"时，用户访问到的资源为IP地址: 

+ 端口号/location匹配中的部分/ location未匹配中的部分

当proxy_pass的端口号后面有"/"时，用户访问到的资源为IP地址: 

+ 端口号/ location未匹配中的部分

```nginx
# 浏览器访问地址为 http://127.0.0.1:80/css/index.css
# 应代理到的地址为 http://www.baidu.com/index.css

local /css {  # 如果关闭，要将已匹配的部分剥离出去例如
	proxy_pass http://www.baidu.com; # 未关闭，实际代理地址为http://www.baidu.com/css/index.css
# proxy_pass http://www.baidu.com/;# 已关闭，实际代理地址为http://www.baidu.com/index.css
}

# 浏览器访问地址为 http://nginx-study.nrsc.com:9002/yoyo1/nginx-study
# 应代理到的地址为 					http://172.17.0.3:8080/nrsc/nginx-study
local /yoyo1 {
	proxy_pass http://172.17.0.3:8080/nrsc  #已关闭，地址为http://172.17.0.3:8080/nrsc/nginx-study
	proxy_pass http://172.17.0.3:8080/nrsc/ #已关闭，地址为http://172.17.0.3:8080/nrsc//nginx-study
}
# 所以关键闭合看的是端口号后面是否有/，如果关闭了就把未匹配的部分添加到后面
```

