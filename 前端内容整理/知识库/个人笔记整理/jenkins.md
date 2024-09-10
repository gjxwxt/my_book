![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1674996970270-417ca99f-d215-4c0e-a110-12e3f90e8a47.png)

### docker安装gitlab
```nginx
yum install -y yum-utils device-mapper-persistent-data lvm2

yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

yum list docker-ce --showduplicates | sort -r
# 安装列表中的版本，后面是版本号
yum install docker-ce-20.10.9-3.el7

systemctl start docker
systemctl enable docker


docker run --detach --hostname 192.168.187.103 --publish 443:443 --publish 80:80 --name gitlab --restart always --volume $GITLAB_HOME/config:/etc/gitlab:Z --volume $GITLAB_HOME/1ogs:/var/log/gitlab:Z  --volume $GITLAB_HOME/data:/var/opt/gitlab:Z --shm-size 256m registry.gitlab.cn/omnibus/gitlab-jh:latest

docker start gitlab

# 进入docker容器之中的/bin/bash目录下
docker exec -it gitlab /bin/bash
		# 查看该容器占内存和剩余内存
		free -m 
		# 查看root密码
		cat /etc/gitlab/initial_root_password

# docker 容器创建之后，下次进入可以直接启动容器
docker ps -a # 查看全部容器
docker start <id号> # 启动id为此的容器
```

```nginx
# https://cr.console.aliyun.com/#/accelerator 获取专属Docker加速器地址

mkdir -p /etc/docker

tee /etc/docker/daemon.json <<-'EOF'

{
	"registry-mirrors": ["https://limktnz1.mirror.aliyuncs.com"]
}
EOF

sudo systemctl daemon-reload
sudo systemctl restart docker
```

    1. ![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1674697264979-60c709f4-99c7-465d-bf62-3822a607be26.png)
    2. ![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1674724083100-4234dec9-81b3-40e8-ad21-e976d62f9c84.png)

### 安装jenkins
```nginx
yum search java|grep jdk

yum install -y java-11-openjdk
# 安装jdk
yum install -y java-devel
```

#### docker安装jenkins
```javascript
docker pull jenkins/jenkins

//创建目录
mkdir -p /var/jenkins_home
//授权权限
chmod 777 /var/jenkins_home


	-d 后台运行镜像
	-p 10240:8080 将镜像的8080端口映射到服务器的10240端口。
	-p 10241:50000 将镜像的50000端口映射到服务器的10241端口
	-v /var/jenkins_mount:/var/jenkins_home 将本地文件地址和容器地址链接起来
	-v /etc/localtime:/etc/localtime让容器使用和服务器同样的时间设置。
	–name myjenkins 给容器起一个别名
	最后是要启动的镜像REPOSITORY:TAG   也就是用于在docker images中启动指定的镜像

docker run -d -p 10240:8080 -p 10241:50000 -v /var/jenkins_home:/var/jenkins_home -v /etc/localtime:/etc/localtime --name myjenkins jenkins/jenkins:lts-jdk11

// 检查是否启动
docker ps

// 此时浏览器就可以访问了
http://192.168.XX.XX:10240

// 获取管理员密码
vim /var/jenkins_home/secrets/initialAdminPassword
```

#### 安装maven
[https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi) 下载，传输到服务器之后，解压缩，将文件夹移到/usr/local/maven目录中，运行即可

```nginx
# 解压缩
tar zxvf xxx
# 移动文件
mv apache-maven-3.8.7 /usr/local/maven
# 运行maven
/usr/local/maven/bin/mvn
```



![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1674809261426-11cf1beb-9d6d-4e7c-aac8-6f97f451b3e6.png)

### jenkins使用
#### 1. 部署前端项目
```dockerfile
# 基础镜像
FROM nginx
# author
MAINTAINER ruoyi

# 挂载目录，所做的需要将
# dist文件传到/home/ruoyi/projects/ruoyi-ui/html文件夹下
# dockerfile文件传到/home/ruoyi/projects/ruoyi-ui/dockerfile文件夹下
# nginx.conf文件传到/home/ruoyi/projects/ruoyi-ui/conf文件夹下
VOLUME /home/ruoyi/projects/ruoyi-ui
# 创建目录
RUN mkdir -p /home/ruoyi/projects/ruoyi-ui
# 指定路径
WORKDIR /home/ruoyi/projects/ruoyi-ui
# 复制conf文件到路径
COPY ./conf/nginx.conf /etc/nginx/nginx.conf
# 复制html文件到路径
COPY ./html/dist /home/ruoyi/projects/ruoyi-ui

```

#### 2. 部署后端项目
1. 打包
2. 传输
    1. 下载插件 publish over ssh
    2. 系统配置中增加ssh server，也就是要部署的服务器地址，添加用户名和密码

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675005970571-fcdc2e97-b81a-4daf-be8f-bda822006e5d.png)

    3. 在流程设置中，在构建中选择Send files or execute commands over SSH，选择之前增加的项，并且配置**<font style="color:rgb(51, 51, 51);background-color:rgb(249, 249, 249);">Source files，代表要传输过去的文件，根目录是workspace，例如</font>**ruoyi-ui/dist/**，代表workspack中的ruoyi-ui/dist/**，**/demo1*_._jar，代表不管前面有多少文件夹，只要是以demo开头的jar包就传送过去_，_**而且传过去的remote directory以root为根目录，/root/xxoo代表传到/root/root/xxoo文件夹下，**remove prefix是将因为传过去的文件路径会包含前缀，例如以下文件传到/root/xxoo上是ruoyi-ui/dist/xxxx，如果只想是/root/xxoo/dist的话就可以用remove prefix将ruoyi-ui这个层级删掉
    4. ![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675005345175-a5ba82c5-72cb-434b-8e27-26c6b05ea950.png)
    5. 如果上传上jar包之后，想要自动运行，可以在exec command中设置，nohup java  xx &代表后台运行![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675006145802-bc94e767-ce1f-4e81-988a-f04a03a58e0f.png) 

```shell
## 后台进程执行jar包，并将标准输出和标准错误输出都输出到mylog.log文件中，这样即使失败也不会卡住了
nohup java -jar /root/xxoo/demo*.jar >mylog.log 2>&1 &
# 或者同等效果写法
# nohup java -jar /root/xxoo/demo*.jar &>mylog.log &
```

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675042782333-91158a74-984c-488e-9d0e-712110a6a7b1.png)

    6.                  jsp命令监视java进程，可以用来查看是否启动，
    7. 如果再次上传，首先需要杀死之前的进程，并且为了防止启动shell脚本执行失败卡住，设置输出日志 

```powershell
## 杀死进程需要在构建打包之前pre steps,可以通过执行服务器shell脚本的方式获取进程id并杀死进程
# 如果jar包是以demo开头，用正则匹配相关的全部全格式进程信息
# 得到两行数据，第一行是具体的进程，第二行是查看进程信息的进程
ps -ef | grep demo  
# 继续筛选，第二行进程名称包含grep，所以可以通过正则筛选除掉包含grep的进程
ps -ef | grep demo | grep -v grep
	#  或者通过java -jar筛选第一行数据,因为-是特殊字符，所以用单引号引起来
	#  ps -ef | grep demo | grep 'java -jar'
# 通过字符串处理工具拿到筛选出来结果的进程id,awk后可以跟通配符以及正则表达式，
ps -ef | grep demo | grep 'java -jar' | awk '{printf $2}'
# 赋给变量pid,需要用``引起来作为一个整体才行，$2代表筛选出来的第二个字段（进程id）
pid=`ps -ef | grep demo | grep 'java -jar' | awk '{printf $2}'`
# 输出一下pid尝试一下
echo $pid
# 最后杀死这个进程
kill -9 $pid
```

```shell
# 构建的jar包名称有时候不是固定的，可以将名字作为变量传给shell脚本
# 例如./x.sh demo ，在x.sh文件中$1,就是拿到的变量值
pid=`ps -ef | grep $1 | grep 'java -jar' | awk '{printf $2}'`
```

```shell
if [ -z $pid ];
		then echo "$1 not started"
		else 
				 kill -9 $pid
				 echo "$1 are stoping..."
fi
```

```shell
#!/bin/bash
#删除历史数据，/root文件夹下的xxoo文件夹
rm -rf xxoo

#获取传入的参数
echo "arg:$1"
appname=$1

# 赋给变量pid,需要用``引起来作为一个整体才行，$2代表筛选出来的第二个字段（进程id）
pid=`ps -ef | grep demo | grep 'java -jar' | awk '{printf $2}'`

# 如果pid为空，提醒一下，不为空杀死进程
if [ -z $pid ];
		then echo "$appname not started"
		else 
				 kill -9 $pid
				 echo "$appname are stoping..."
fi

# 检测是否杀死进程成功
check=`ps -ef | grep -w $pid | grep java`
if [-z $check ];
		then
			echo "$appname pid:$pid is stop"
		else
			echo "$appname stop failed"
fi
```

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675047348423-17ab004d-df40-42eb-ad6b-2fcce9ae46dc.png)

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675047366090-9aa649a6-4ed0-46b9-9ca1-bad4dec547b5.png)



![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675042489295-db583d9c-2ebf-495a-bc01-2283ac31cba6.png)

#### 3. 自动开始构建(github类的钩子)
1. 安装插件[Build Authorization Token Root](https://plugins.jenkins.io/build-token-root)
2. 在具体项目中的触发构建器中设置token，访问时http://192.168.187.102:10240/job/ruoyi-ui/build?token=123456。但是如果不是同一浏览器不能触发，安装完插件后，访问地址为                                         <font style="color:#DF2A3F;">http://192.168.187.102:10240</font><font style="color:#000000;">/buildByToken/build?job=</font><font style="color:#DF2A3F;">ruoyi-ui</font><font style="color:#000000;">&token=</font><font style="color:#DF2A3F;">123456</font>，只要在项目的构建触发器中设置了token，那么就可以通过<font style="color:rgb(51, 51, 51);background-color:rgb(249, 249, 249);">JENKINS_URL</font><font style="color:rgb(51, 51, 51);background-color:rgb(249, 249, 249);">/</font>buildByToken/build?job=NAME&token=SECRET去触发该项目，**<font style="color:rgb(51, 51, 51);background-color:rgb(249, 249, 249);">JENKINS_URL</font>**<font style="color:rgb(51, 51, 51);background-color:rgb(249, 249, 249);">时jenkins服务器地址，</font>**NAME**是触发的构建项目名称，**SECRET**就是相应设置的身份验证令牌token

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675048200933-00639ae3-ff5b-4dc5-b754-fa97074fae42.png)

3. 在git服务器中的项目里找到webhooks，设置url以及钩子触发时间

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675048925195-a9de1590-193f-4d62-83d8-572634d3ce7b.png)

4. 如果本地的gitlab不允许向本地服务器发送请求，可以从设置的网络中，更改出站请求，将允许向本地服务器发送请求勾上就可以了
5. 在触发钩子中的合并请求，实际上是在开始合并的时候就触发了一次，合并完成之后还会触发一次

#### 4. 定时构建
##### 几种常见构建方式
1. 快照依赖构建/Build whenever a SNAPSHOT dependency is built
    - 当依赖的快照被构建时执行本job
2. 触发远程构建（例如，使用脚本）
    - 远程调用本job的restapi时执行本job
3. job依赖构建/Build after other projects are built
    - 当依赖的job被构建时执行本job
4. **定时构建**/Build periodically
    - 使用cron表达达时构建本job
5. 向GitHub提交代码时触发jenkins自动构建/GitHub hook trigger for GITScm polling
    - Github-WebHook出发时构建本job
6. 定期检查代码变更/Poll SCM，jenkins主动发起请求去检查
    - 使用cron表达式定时检查代码变更，变更后构建本job

##### 定时构建的使用
###### 1. cron语句
效果查看[https://crontab.guru/](https://crontab.guru/)，分钟，小时，第几日，第几月，周几，/代表步长，*代表任意

+ 5 * * * * 代表每个小时的第五分钟开始构建一次
+ */5 * * * * 代表每个小时的每五分钟开始构建一次，/5代表步进为5
+ */5 2 * * * 代表每天的两点每五分钟开始构建一次
+ */5 */2 * * * 代表每天的每隔两个小时中该小时中每五分钟构建一次
+ */5 2 1 1-6 * 代表1-6月份中的1号当天的两点每隔五分钟执行一次
+ */5 2 1 1-6,10-12 * 代表1-6和10-12月份中的1号当天的两点每隔五分钟执行一次
+ 0 2 5 1-6 1  代表1-6月中的每周一以及每个月的5号的两点开始构建一次
2. **构建触发器中的H，是对job名取hash值，每个表达式位上都可以使用，H/5,代表每隔5分钟，但开始时间是H，**为什么用hash值呢，因为如果服务器间隔时间相同的有很多任务会造成服务器的拥挤，随机时间可以有效避免堆在一块
    1. H * * * * 代表每小时的该hash分钟进行构建
    2.  * * * * * 代表每分钟构建一次
    3. H(10-30) * * * * 代表取的hash从10-30中取，每小时的该hash分钟进行构建
    4. H/5 * * * * 代表开始时间用hash确定，每隔5分钟开始构建，例如8点33分，8点38分
    5. */5  * * * * 代表开始时间是零点触发，每隔五分钟开始构建，例如8点5分，8点10分
    6. H(0-30) * * * 1-6 代表周一到周六的每小时的第hash（0-30之内的值）分钟构建一次
    7. H 0-6/2 * * 1-6  代表周一到周六的0-6点中每隔两小时中的hash分钟构建一次，例0:33,2:33,4:33,6:33



#### 5. 邮箱通知
1. 发送信息的邮箱开通SMTP服务，记住授权登录密码
2. 配置jenkins location，jenkins url是要发送过去的内容，系统管理员邮件地址是要发送邮箱的地址，与后面配置中的用户名要匹配才能发送![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675063473296-b7018659-3a22-4c3b-90b2-328cdde44ccc.png)
3. 在jenkins的全局配置中找到邮箱服务，添加凭证中，username是gaojiaxiang2018,password是保存的授权登录密码，![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675062569769-0aad60f2-61fe-48e3-b693-b7fe5974e581.png)
4. 在jenkins的全局配置中找到邮箱通知，配置并测试，密码也是登录凭证，![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675063598671-79652a67-d78c-418c-82b3-ca6b02fc2baf.png)![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675063635357-ebbb5a4c-4dd1-4662-9071-3e099c85514c.png)
5. 在项目设置中找到构建后操作，添加上Editable Email Notification，在高级设置中，设置触发钩子和接收人，例如设置了build user就会向设置好的构建job的用户邮箱发送email

### 如何将打包文件部署到docker容器中
#### 一、外挂目录(不需要重新构建镜像)
1. 准备一台测试服务器docker环境
2. 准备支持jdk的镜像
    1. 把jar包打包到容器内/root/jarfile下，目录随意，只要能copy到容器内就行	
    2. 拉取jdk镜像	docker pull openjdk:11
    3. 运行镜像生成容器                                                           docker run -d -p 8080:8080 --name demo -v /root/jarfile/demo*.jar:/app.jar openjdk:11 java -jar app.jar
    4. 此时，容器运行成功。在jenkins再次构建之前，需要先**将docker停掉**，**将文件删除**之后进行构建，构建完成后**传输到docker关联的文件夹**，然后**启动docker容器**即可

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675082810838-3742fd6f-a4f9-4fea-84cd-2ea7591fbabd.png)

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675082567630-fc90ad8d-faed-43de-8151-e2ad7daa3f59.png)

#### 二、生成dockerfile文件，与jar包同时发送到服务器，生成新的镜像，生成容器
##### （1）：理论基础
1. 准备一台测试服务器docker环境
2. 准备支持jdk的镜像
    1. 把jar包打包到容器内/root/jarfile下，目录随意，只要能copy到容器内就行

```dockerfile
# 拉取镜像
FROM openjdk:11  
EXPOSE 8080
# work目录
WORKDIR /root
# 将work目录下的也就是/root/jarfile/demo*.jar文件添加到docker容器目录下，并改名字
# 注意，copy和add后面文件的路径是基于当前执行dockerfile路径能找到的前提下才行
# 因为将dockerfile文件传到root目录下，所以执行的时候能找到jarfile文件夹，
ADD jarfile/demo*.jar /root/app.jar
# 执行容器内的jar包，命令空格用逗号隔开
ENTRYPOINT ["java","-jar","/root/app.jar"]
```

    2. 生成镜像：在dockerfile存放的目录下，执行docker build -t demo .
        1. docker build -t  镜像名:版本名 dockerfile所在目录，. 代表当前目录
    3. 运行容器：docker run -d --name demo -p 8080:8080 demo
        1. docker run -d --name 生成的容器名 -p 映射本地端口:容器端口 运行的镜像名 可跟构建完成后要执行的命令
3. 了解了大体思路后，通过jenkins构建需要在构建前，将文件删除（jar包和dockerfile），停止容器（docker  stop demo）删除容器（docker rm demo）删除镜像（docker rmi demo-images），构建完成后，传到服务器指定文件夹，根据dockerfile构建镜像并启动容器即可![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675087073205-ec9e2dac-324a-442c-ba4d-edce2160f3b6.png)![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675087736414-ee00c326-ea49-421b-8d69-e0c52be16f5c.png)![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675088308981-1fe19afd-498a-4ffe-b15c-b9b451db408c.png)

##### （2）：实战练习(若依前端项目为例)
###### 1. 拉取git代码
![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675144632916-6c8a6317-d8a1-4e1a-a64b-bc24be9f1d81.png)

###### 2. 搭建构建环境并编译
![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675144692256-8f6d71b3-b5ba-42c9-bfac-71167cc015b1.png)

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675144709202-0659b63a-9c47-4fb2-ba56-754ce2222666.png)

###### 3. 发送dist文件和dockerfile以及nginx.conf文件
![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675144916453-2247e589-890f-4918-9386-903c2dea82fb.png)

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675144938500-ba7c38f4-da26-4da4-b2e0-f1a68f90757d.png)

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675147306143-32d0390a-c73b-43a8-acbd-710ce0bcd1f2.png)

#### 三、生成dockerfile文件，与jar包同时发送到docker私服，例如Harbor，然后k8s拉取镜像生成多个容器同时运行
### 如果代码的commit没变，就不去进行构建
防止误操作立即构建

```shell
#!/bin/bash
if [ $GIT_PREVIOUS_SUCCESSFUL_COMMIT == $GIT_COMMIT ];then
  echo "no change, skip build"
  exit 0
else
  xxx构建
fi
```

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676440521353-34c19d1e-b5c7-4799-bd8d-d73002005459.png)

### 举例：ruoyi项目配置截图
#### 项目配置：
![](https://cdn.nlark.com/yuque/0/2023/jpeg/28792475/1684547852113-70c07dd1-351b-4add-b3f6-f33532749ca2.jpeg)

#### 系统配置
![](https://cdn.nlark.com/yuque/0/2023/jpeg/28792475/1684547988137-7898557d-f57b-42d7-b4a9-438117449a22.jpeg)

#### 
