### <font >一、使用sticky模块完成对Nginx的负载均衡</font>
#### 下载与安装以及实现nginx的平滑升级
1. 下载：[https://bitbucket.org/nginx-goodies/nginx-sticky-module-ng/src/master/](https://bitbucket.org/nginx-goodies/nginx-sticky-module-ng/src/master/)
2. 安装到nginx服务器上：
+ 首先将.zip文件放到root根目录下，`unzip 文件`解压.zip文件
+ 修改解压后文件夹中的ngx_http_sticky_misc.c文件，在include下添加这两行，相当于引用文件

```nginx
#include <openssl/sha.h>
#include <openssl/md5.h>
```

+ 下载所需的依赖  yum install -y openssl-devel
+ 最后重新编译nginx
    - 找到之前解压缩的nginx文件夹，进去后执行

```shell
// 执行下一行代码，首先因为是重新编译，所以之前安装时做的配置项要重新做一遍，最后将sticky模块添加上即可
./configure --prefix=/usr/local/nginx --add-module=/root/nginx-goodies-nginx-sticky-module-ng-c78b7dd79d0d
```

    - 之后make   不要make  install  ，因为install之后会直接覆盖之前的nginx配置
3. 完成nginx的升级：
+ 首先将旧的nginx执行文件修改名称，备份一下

`  mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.old  `     

+ 把编译好的Nginx程序替换到原来的目录里，这个objs文件夹就是之前解压nginx后的文件夹中的objs，一般执行文件就在objs文件夹中

`  cp objs/nginx /usr/local/nginx/sbin/  `

+ 此时就实现了nginx的更新，systemctl start nginx 执行一下就可以看出来了

#### 使用
```nginx
upstream test {
# name和expires是常见的可配置项，name就是nginx下发的cookie的key
# 注意 因为上游服务器为了保持会话会默认返回cookie，例如tomcat返回cookie的key为JSESSIONID
# sticky下发的cookie的name不能和后端服务器返回的冲突
# 默认下发的cookie的key就为route
sticky name=route expires=6h;  

server 192.168.44.102;
server 192.168.44.103;
}
```

#### 优点：
+ 利用cookie实现保持会话，不光动态服务器，连静态服务器也可以实现，因为静态服务器不会下发cookie，nginx利用sticky给客户端下发cookie，以此来辨识你这个用户去哪个服务器，创建多少个新标签也是去那个服务器

### 二、keepalive长连接
目的：nginx向上游服务器链接的时候也用keepalive，减少握手次数，提高效率，

当使用nginx作为反向代理时，为了支持长连接，需要做到两点：

1. 从client到nginx的连接是长连接
2. 从nginx到server的连接是长连接

#### 1、保持和client的长连接：
默认情况下，nginx已经自动开启了对client连接的keep alive支持（同时client发送的HTTP请求要求keep alive）。一般场景可以直接使用，但是对于一些比较特殊的场景，还是有必要调整个别参数

+ 参数详解：
    - keepalive disable   none | browser   # 设置针对某些浏览器禁用keepalive，默认值为msie6，一般不设
    - keepalive_timeout   75; # 活动链接之间的间隔时间，默认75s，第二个参数可选，一般不设置。设置为0时代表keepalive关闭
    - keepalive_requests   100；# <font>一个keep-alive连接上可以请求的最大数量，超过就关闭，默认100,这是nginx调优的一部分，如果QPS(秒请求量)过大的会不断重新建立tcp链接，可以设大一点</font>
    - keepalive_time  1h; # 限制keepalive保持连接的最大时间
    - send_timeout   60; # client两次成功的写入操作，但是服务器在该时间内没有返回数据的情况下关闭tcp链接，注意时间较长的系统操作该时间要够长，默认是60

#### <font>2、保持和server的长连接：</font>
+ <font >使用http1.1协议。以便建立更高效的传输，默认使用http1.0，在http1.0中需要配置header才能</font>
+ <font>在Upstream中所配置的上游服务器默认都是用短连接，即每次请求都会在完成之后断开</font>

```nginx
upstream test {
	keepalive 100; # 向上游服务器的保留连接数
	keepalive_requests 1000; # 一个tcp复用中 可以并发接收的请求个数
	sticky;
	server 192.168.154.130:80;
	server 192.168.154.129:80;
}
server {
		listen       80;
		server_name  www.gaojiaxiang.cn;

		location / {
			proxy_http_version 1.1;  # 配置http版本号
			proxy_set_header Connection "";  # nginx建立链接之后会把connection变为close，
			proxy_pass http://test;	
		}

		error_page   500 502 503 504  /50x.html;
		location = /50x.html {
				root   html;
		}

}
```



#### 3、总结：关于keepalive的常用配置
```nginx
keepalive_timeout 65 65;#超过这个时间设有活动。会让keepalive失效
keepalive_time 1h;#个tcp连接总时长，超过之后强制失效
send_timeout 60;#默认60s此处有坑！！系统中若有耗时银作，超过send_timeout强制斯开连接，注意：准备过程中，不是传输i过程
keepalive_request 1000; #一个tcp复用中可以并发接收的请求个数

upstream test {
	sticky;
	keepalive 100; # 向上游服务器的保留连接数
	keepalive_timeout 65;
	keepalive_requests 1000; # 一个tcp复用中 可以并发接收的请求个数
	server 192.168.154.130:80;
	server 192.168.154.129:80;
}
server {
		listen       80;
		server_name  www.gaojiaxiang.cn;

		location / {
			proxy_pass http://test;
			proxy_http_version 1.1;  # 配置http版本号
			proxy_set_header Connection "";
		}
	

		error_page   500 502 503 504  /50x.html;
		location = /50x.html {
				root   html;
		}
	
		location ~*/(css|img|js){
				root /usr/local/nginx/html:
		}
}
```

#### 4、ab压测对比
1. <font >yum install httpd-tools下载，软件下载在与nginx和上游服务器不同的linux虚拟机中去请求。</font>
2. ab -n 100000 -c 30 [http://vod.gaojiaxiang.cn:80/](http://vod.gaojiaxiang.cn:80/)      意思是100000次请求，30次并发
3. 参数说明：
+ -n  即requests，用于指定压力测试总共的执行次数。
+ -c  即concurrency，用于指定的并发数。
+ -t  即timelimit，等待响应的最大时间(单位：秒)。
+ -b  即windowsize，TCP发送/接收的缓冲大小(单位：字节)。
+ -p  即postfile，发送POST请求时需要上传的文件，此外还必须设置-T参数。
+ -u  即putfile，发送PUT请求时需要上传的文件，此外还必须设置-T参数。
+ -T  即content-type，用于设置Content-Type请求头信息，例如：application/x-www-form-urlencoded，默认值为text/plain。
+ -v  即verbosity，指定打印帮助信息的冗余级别。
+ -w  以HTML表格形式打印结果。
+ -i  使用HEAD请求代替GET请求。
+ -x  插入字符串作为table标签的属性。
+ -y  插入字符串作为tr标签的属性。
+ -z  插入字符串作为td标签的属性。
+ -C  添加cookie信息，例如："Apache=1234"(可以重复该参数选项以添加多个)。
+ -H  添加任意的请求头，例如："Accept-Encoding: gzip"，请求头将会添加在现有的多个请求头之后(可以重复该参数选项以添加多个)。
+ -A  添加一个基本的网络认证信息，用户名和密码之间用英文冒号隔开。
+ -P  添加一个基本的代理认证信息，用户名和密码之间用英文冒号隔开。
+ -X  指定使用的和端口号，例如:"126.10.10.3:88"。
+ -V  打印版本号并退出。
+ -k  使用HTTP的KeepAlive特性。
+ -d  不显示百分比。
+ -S  不显示预估和警告信息。
+ -g  输出结果信息到gnuplot格式的文件中。
+ -e  输出结果信息到CSV格式的文件中。
+ -r  指定接收到错误信息时不退出程序。
+ -h  显示用法信息，其实就是ab -help。

### 三、反向代理的缓冲区设置
背景： nginx作为中间人，客户端发送过来的请求中的数据，例如请求体中的数据如何读取并转发，缓冲区大小以及如何缓冲。服务器返回数据的时候，nginx同样该如何配置

#### 1. 与客户端相关的缓冲区设置
+ client_body_buffer_size     # <font >对客户端请求中的body缓冲区大小，</font>缓冲区不够大就写入临时文件。
    - <font >默认32位8k 64位16k，一般get请求8k就够了，一般表单提交16k也够了</font>
+ client_header_buffer_size  # <font >对客户端请求中的header缓冲区大小，</font>
+ client_body_temp_path     # 临时文件<font >在磁盘上客户端的body临时缓冲区位置</font>
+ client_max_body_size         # body最大是多少，通过http协议的协议头上的content-length看出请求大小
+ client_body_timeout           # 如果客户端在指定时间内没有发送任何内容，Nginx 返回 HTTP 408
+  client_header_timeout       #如果客户端在指定时间内没有发送一个完整的 request header，返回408
+ client_body_in_file_only      # 对请求发过来的body保存在磁盘中不删除，即使没有body也会存一个o字节的文件，线上环境不能配，容易内存溢出
+ client_body_in_single_buffer  #<font >尽量缓冲body的时候在内存中使用连续单一缓冲区，在二次开发时使用</font><font style="color:rgb(51, 51, 51);background-color:rgb(243, 244, 244);">$request_body</font><font >读取数据时性能会有所提高</font>

#### 2.与服务器相关设置
+ proxy_set_header      		# nginx转发的时候添加上某些浏览器需要的header

```nginx
#将客户端真实ip在请求头传过去，nginx代理时后端只会拿到nginx服务器的ip地址
location / {
		proxy_pass http://192.168.100.30:80/;
		proxy_set_header Connection "";
		proxy_set_header X-Forwarded-For $remote_addr;
}
# 多级代理的时候，只要第一次代理添加ip地址即可，后面如果重复设置会覆盖并且ip是上一级nginx地址
```

+ proxy_connect_timeout   # <font >与上游服务器连接超时时间、快速失败</font>
+ proxy_send_timeout		# nginx向后端服务发送请求的间隔时间(不是耗时)。默认60秒，超过这个时间会关闭连接
+ proxy_read_timeout		# <font >后端服务给nginx响应的时间，规定时间内后端服务没有给nginx响应，连接会被关闭，nginx返回504 Gateway Time-out。默认60秒</font>
+ proxy_requset_buffering on; # <font >是否完全读到请求体之后再向上游服务器发送请求</font>
+ proxy_buffering	on;			# 是否缓冲上游服务器数据
+ proxy_buffers 32 64k; 		# <font >缓冲区大小 ：32个 64k大小内存缓冲块</font>
+ proxy_buffer_size  64k;		# <font >header缓冲区大小</font>
+ proxy_temp_file_write_size  8k;		# 一次限制写入临时文件的数据的大小
+ proxy_max_temp_file_size  1024m;	# <font >临时文件最大值</font>
+ proxy_temp_path      /spool/nginx/proxy_temp 1 2;          # 临时文件的放置路径  





### 四、前端打包gzip + nginx开启静态gzip
目的：作为资源托管的nginx服务器，传输资源的时候如果传输的是压缩过的文件，会缩短传输时间。动态gzip是服务器动态压缩要返回的数据，如果并发量大会十分消耗服务器性能，可以提前将要压缩的文件提前打包时压缩好，判断浏览器发来的请求头中是否支持该压缩格式，然会返回压缩文件或者源文件。图片视频音频都是经过压缩过的格式，不需要gzip进行再次压缩，适合压缩html、js、css文件

#### 动态gzip完整配置：开启gzip时，sendfile就失效了，因为肯定要拷贝一份进行压缩
```nginx
gzip on;      #开启
gzip_buffers 16 8k;    #压缩过程中缓冲，32内核32 4k，64位内核16 8k，代表16个8k大小的缓冲块
gzip_comp_level 6;     #压缩等级 1-9，越大压缩比越高，对cpu消耗越大
gzip_http_version 1.1;  #支持最低的http版本号
gzip_min_length 256;  #小于256字节的内容不进行压缩，页面字节数从header头中的Content-Length中进行获取
gzip_proxied any;   #开启代理的时候，根据上游服务器返回的头信息进行判断是否进行压缩，off代表不管是啥压缩都不压缩，any代表不管是啥都压缩
gzip_vary on;   #返回头添加一项：Vary: Accept-Encoding，为了适应老的浏览器
gzip_types text/plain application/x-javascript text/css application/xml;   # 对于哪些mime类型的配置进行压缩，一般的文本文档
gzip_types
	text/xml application/xml application/atom+xml application/rss+xml application/xhtml+xml image/svg+xml
	text/javascript application/javascript application/x-javascript
	text/x-json application/json application/x-web-app-manifest+json
	text/css text/plain text/x-component
	font/opentype application/x-font-ttf application/vnd.ms-fontobject
	image/x-icon;   #较为全的mime类型，不配就不压缩了，icon很小，一般也不压缩
# gzip_disable "MSIE [1-6]\.(?!.*SV1)";  #针对哪些浏览器就不压缩了，一般不配置，因为正则匹配会消耗cpu
```

```markdown
HTTP/1.1 200 OK
Server: nginx/1.21.6
Date: Mon, 24 Oct 2022 06:03:33 GMT
Content-Type: text/html
Content-Length: 450             没压缩的时候来表示内容大小
Last-Modified: Thu, 20 Oct 2022 01:25:23 GMT
Connection: keep-alive
Keep-Alive: timeout=65
ETag: "6350a383-1c2"
Accept-Ranges: bytes

HTTP/1.1 200 OK
Server: nginx/1.21.6
Date: Mon, 24 Oct 2022 06:05:35 GMT
Content-Type: text/html
Last-Modified: Thu, 20 Oct 2022 01:25:23 GMT
Transfer-Encoding: chunked  一个一个数据包传输，只要包大小为0代表内容读完了，和content-length对应
Connection: keep-alive
Keep-Alive: timeout=65  
Vary: Accept-Encoding     这一项时支持老的浏览器
ETag: W/"6350a383-1c2"
Content-Encoding: gzip     
```

#### 静态压缩：
作为资源服务器时，源文件和压缩文件存在服务器上，此时，content-length和content-encoding都存在响应头中，支持sendfile，如果只有压缩文件没有源文件可以搭配gunzip实现动态编译成源文件。

1. 安装：
+ 首先复制之前nginx编译时的参数，<font > /usr/local/nginx/sbin/nginx -V</font>
+ <font >重新编译nginx，在之前参数的基础上添加  </font><font style="color:rgb(0, 0, 0);">--with-http_gzip_static_module，实现nginx平滑升级</font>
2. 使用
+ 添加配置项  gzip_static  on; 开启后检查客户端是否支持gzip，如果支持会先寻找以.gz结尾的文件，找到直接返回，如果找不到gz文件但是有源文件则进行压缩。如果客户端不支持gzip，就返回没压缩的源文件。如果只有压缩包，而且碰上不支持gzip文件压缩的客户端，会直接返回404页面，因为没文件可以返回。此时可以通过安装gunzip将压缩文件解压编译成支持的文件发送过去。
    - 1. 安装：<font style="color:rgb(0, 0, 0);">--with-http_gunzip_module重新编译nginx</font>
    - <font style="color:rgb(0, 0, 0);">2. 使用：</font>gunzip on;开启同时开启<font style="color:rgb(38, 38, 38);">gzip_static always;always代表不管客户端支不支持gzip，都返回gzip，此时在通过gunzip将gzip文件解压缩返回，可以实现只有gzip但是可以返回源文件</font>

```nginx
location /{
	gunzip on;
	gzip_static always;
	gzip on;
	gzip_buffers 16 8k;
  gzip_comp_level 6;
  gzip_http_version 1.1;
  gzip_min_length 256;
  gzip_proxied any;
  gzip_vary on;
  gzip_types text/plain application/x-javascript text/css application/xml;
  gzip_types
    text/xml application/xml application/atom+xml application/rss+xml application/xhtml+xml image/svg+xml
    text/javascript application/javascript application/x-javascript
    text/x-json application/json application/x-web-app-manifest+json
    text/css text/plain text/x-component
    font/opentype application/x-font-ttf application/vnd.ms-fontobject
    image/x-icon;
  gzip_disable "MSIE [1-6]\.(?!.*SV1)";
}
```

+ 通过webpack利用compression-webpack-plugin将文件进行压缩

```javascript
// npm install compression-webpack-plugin --save-dev
const CompressionWebpackPlugin = require('compression-webpack-plugin');
const productionGzipExtensions = ['js', 'css'];
configureWebpack: config => {
	if (process.env.NODE_ENV === 'production') {
		// 打包生产.gz压缩包
		config.plugins.push(new CompressionWebpackPlugin({
			algorithm: 'gzip',
			test: new RegExp('\\.(' + productionGzipExtensions.join('|') + ')$'),
			threshold: 10240,
			minRatio: 0.8
		}))
	}
}
```

### 五、让nginx支持br压缩方法
##### 1. br压缩简介：
谷歌chrome内核支持，只针对https协议下的请求头才支持带：accept-encoding:br;

##### 2. 安装：
1. 下载并放置在root目录下

 [https://github.com/google/ngx_brotli](https://github.com/google/ngx_brotli`)

[https://codeload.github.com/google/brotli/tar.gz/refs/tags/v1.0.9](https://codeload.github.com/google/brotli/tar.gz/refs/tags/v1.0.9`)

2. 解压：
+  tar -xzvf ngx_brotli-1.0.0rc.tar.gz 
+  tar -xzvf brotli-1.0.9.tar.gz  （算法包，解压完进入该文件）
+  mv ./* /root/ngx_brotli-1.0.0rc/deps/brotli/     （将算法包下所有内容放在程序下的deps/brotli目录下）
3. 编译nginx
+ <font >/usr/local/nginx/sbin/nginx -V   查看之前的所有配置</font>
+ ./configure --with-compat --add-dynamic-module=/root/ngx_brotli-1.0.0rc   添加上动态添加模块命令
+ make  编译
+ mkdir /usr/local/nginx/modules   创建一个新文件夹将压缩好的文件放在单独文件夹中，作为模块
    - cp ngx_http_brotli_filter_module.so /usr/local/nginx/modules/
    - cp ngx_http_brotli_static_module.so /usr/local/nginx/modules/
+ 将之前的nginx主程序备份并放在该文件夹中
    - mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.old345
    - cp nginx /usr/local/nginx/sbin/            （这是objs文件夹下的nginx）

##### 3.配置
1. 因为是动态模块，所以先load才能使用命令。在config文件头部添加
+ load_module "/usr/local/nginx/modules/ngx_http_brotli_filter_module.so";
+ load_module "/usr/local/nginx/modules/ngx_http_brotli_static_module.so";
2. 配置

```nginx
	brotli on;
  brotli_static on;
	brotli_comp_level 6;
	brotli_buffers 16 8k;
	brotli_min_length 20;
	brotli_types text/plain text/css text/javascript application/javascript text/xml application/xml application/xml+rss application/json image/jpeg image/gif image/png;
```

##### 4.测试
因为只有https的请求头才会支持br的压缩格式，可以用nginx上的curl去修改请求头就可以了

curl -H 'accept-encoding:br' -I http://192.168.187.101

### 六、利用concat模块合并请求
1. 模块简介：开发中分割模块利于维护和更新，但是会产生多个css文件和js文件，会产生多次请求。concat模块可以根据请求中??后面的部分将一次请求返回多个css或者js文件，淘宝构想的模块，目前仍在使用
2. 安装
+ [https://github.com/alibaba/nginx-http-concat](https://github.com/alibaba/nginx-http-concat)   下载zip文件放在虚拟机root目录下
+ unzip xxx 解压该文件，如果没有unzip 可以先下载yum install unzip
+ 重新编译nginx，添加上配置   --add-module=/root/nginx-http-concat-master，后面仍是之前一套流程
3. 使用

```nginx
# html中原本
<link rel="stylesheet" href="index2.css">
<link rel="stylesheet" href="index1.css">
# 变成
<link rel="stylesheet" href="??index1.css,index2.css">

# config的配置
 location / {
	  root test_concat;
	  concat on;
   	concat_max_files 30;
    index  index.html index.htm;
}
```

