## 1、准备好环境
### 1. master节点安装
#### 安装etcd服务2379端口
1. 2379是提供服务用的，往etcd中写数据用的就是2379端口，集群之间数据同步用的是2380。如果配置127.0.0.1:2379就是仅供本地访问，配置0.0.0.0:2380就是所有服务器都可以访问

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675923576631-f1f86175-db86-4a32-9a56-567390b2b1db.png)

#### 安装kubernetes master，其中apiserver8080端口
![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675924155533-e424be90-e8e6-45de-80d0-d6c54f795a1e.png)

安装完成之后，输入命令  ，可以查看相关服务的信息

### 3. node节点安装
1. 首先删除该服务器上的docker

```shell
删除方法：（依次执行下面的命令）

yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-selinux docker-engine-selinux docker-engine

rm -rf /etc/systemd/system/docker.service.d
rm -rf /var/lib/docker
rm -rf /var/run/docker

# 将此命令下输出的文件删除，例如
rpm -qa|grep docker
rpm -e docker-ce-cli-19.03.13-3.el7.x86_64(上面命令输出的文件)

```

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675927462636-06aa4f6c-93f8-4105-90b4-431749c0f757.png)

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675925149423-d6fcef86-8c5f-4df7-814d-72a88cf98460.png)

当node节点接入到集群中后，直接使用kubectl命令是使用不了的，但是可以远程操控master节点执行kubectl操作，例如

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676441411106-afdc0eec-dfc3-40ab-aa72-5973c0d5d882.png)

所以当该node节点安装jenkins后，可以通过-s远程操控master节点进行构建升级回滚操作

### 4. 所有节点安装flannel网络插件
![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675932629093-40c20ec8-3f90-4091-b7d7-90329e4d38ba.png)

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675992625946-18a0261d-ccb5-4f7d-a7f9-cfd521d19879.png)

## 2、k8s运行
### 1. pod资源
k8s启动一个pod，pod会控制docker至少生成两个容器，一个是业务容器例如nginx，另一个是pod容器，主要用于实现k8s的高级功能，例如自动修复等，业务容器的network类型是container，和pod容器共用一个ip地址，只是端口不同。一个pod可以启动多个业务容器，如果一个pod中多个服务都想被访问到，需要创建多个不同的service才行。

#### 创建运行一个pod资源
创建一个pod，首先书写yaml文件，然后kubectl create -f nginx.yaml即可创建pod，

```yaml
apiVersion: v1  
kind: Pod  
metadata:  
  name: nginx  #pod的名字
  labels:
    app: web
spec: 
  containers:
   - name: nginx
     image: nginx:1.13
     imagePullPolicy: IfNotPresent   # 如果本地docker镜像中不存在该镜像再去拉远程的
     commend: ["sleep","3600"]
     ports:
      - containerPort: 80
```

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675995776662-5aeb331e-d9db-48b6-aae6-1f85f323a8ab.png)

拉取镜像失败，需要修改一下配置文件vi /etc/kubernetes/kubelet

首先docker search pod-infrastructure，找到合适的地址，将该配置

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675996177329-f21078c3-19eb-432d-9822-5bf2d126fca7.png)

换成

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1675996065752-d89747c2-6715-4864-9da1-8e705741a5c1.png)

然后重启systemctl restart kubelet.service

#### 常用pod命令
通过kubectl get pod可以查看创建的pod，

kubectl get pod nginx -o wide就可查看具体pod的信息，包括安装在了哪台node节点上

kubectl get componentstatus查看k8s组件状态

kubectl describe pod nginx 查看该pod目前详细信息，用于排错

kubectl delete pod test1 --force --grace-period=0   强制删除pod

kubectl apply -f nginx.yaml  如果修改了yaml文件，执行更新pod

kubectl exec -it pod名 /bin/bash       进入pod中



### 2. rc副本控制器（replication controller）保证高可用
应用托管在Kubernetes.之后，Kubernetes需要保证应用能够持续运行，这是Rc的工作内容，它会确保任何时间Kubernetes中都有指定数量的Pod在运行。如果pod死了，rc就会控制k8s在别的node节点上起一个新的pod。 在此基础上，RC还提供了一些更高级的特性，比如滚动升级、升级回滚等。



一个rc会创建多个pod，如何判断是自己的pod呢，就根据pod的lables来进行管理的，如果一个之前没有关系的pod修改了标签和rc进行了关联，就会受rc的控制，多了就会杀死超过数量的pod，优先杀死年轻的pod

#### 基本使用
首先创建个文件夹rc，然后写nginx的yaml文件，

kubectl create -f nginx-rc.yaml 创建rc

kubectl get rc  查看rc

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myweb
spec:
  replicas: 1
  selector:
    app: myweb
  template:
    metadata:
      labels:
        app: myweb
    spec:
      containers:
      - name: myweb
        image: nginx:1.13
        ports:
        - containerPort: 80

```

```yaml
apiVersion: v1
# 副本控制器 RC
kind: ReplicationController
metadata:
  # RC 的名称，全局唯一
  name: mysql
spec:
  # 期待创建的 Pod 个数
  replicas: 1
  selector:
    # 选择符合拥有此标签的 Pod
    app: mysql
  # 根据模板下定义的信息创建 Pod
  template:
    metadata:
      labels:
        # Pod 拥有的标签，对应上边 RC 的 selector
        app: mysql
    # 定义 Pod 细则
    spec:
      # Pod 内容器的定义
      containers:
        # 容器名称
        - name: mysql
          # 使用的镜像
          image: mysql:5.6
          ports:
            # 暴露的端口号
            - containerPort: 3306
          # 注入到容器中的环境变量
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "123456"
```

#### 滚动升级与回滚
新建一个更新后的yaml文件，执行kubectl rolling-update 之前的rc名 -f 新的yaml文件 --update-period=30s(这是更新间隔，一个pod更新成功之后间隔多少秒更新下一个pod)

例如：kubectl rolling-update myweb -f nginx-rc2.yaml --update-period=30s

**回滚**就相当于又更新了一次，kubectl rolling-update myweb2 -f nginx-rc.yaml --update-period=1s；

如果从myweb-》myweb2升级的时候，**升级一半发现问题**，ctrl+c中断之后，可以通过kubectl rolling-update myweb myweb2 --rollback，重新回滚到myweb的时候



<font style="color:rgb(68, 68, 68);background-color:#FBDE28;">在新旧配置文件中需要注意以下两点。</font>

<font style="color:rgb(68, 68, 68);">◎ RC的名字（name）不能与旧RC的名字相同。</font>

<font style="color:rgb(68, 68, 68);">◎ 在selector中应至少有一个Label与旧RC的Label不同，以标识其为新RC。</font>

#### 常用命令
1. kubectl scale rc myweb --replicas=2   修改rc名为myweb的pod节点数量为2个

### 3. service资源，保证外界能访问到
在k8s文件夹下新建svc文件夹，创建nginx-svc.yaml文件，之后kubectl create -f nginx-svn.yaml，创建成功后，kubectl describe svc可以查看端口和ip

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myweb  # 创建的service的名字
spec:
  type: NodePort  # 类型时nodeport，还有clusterport,如果不选为nodeport，默认是clusterport
  ports:
    - port: 80  # vip的端口，也就是cluster的端口
      nodePort: 30000 # 宿主机的端口，访问宿主机端口映射vip端口再映射到pod端口，默认范围30000-32767
      targetPort: 80 # pod端口
  selector:
    app: myweb # 根据标签选择哪些pod纳入到一个service中，进行负载均衡
```

通过rc新加的pod，会自动加入service，被称为自动发现



**修改nodeport默认范围，**vim** **/etc/kubernetes/apiserver 中的最后一行，

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676021375267-4032af99-10b8-4b31-9584-2ae4df4c6063.png)

--service-node-port-range=30000-50000



通过echo写文件   echo 'node1' >index.html  将'node1'写入当前文件夹下的index.html文件

向pod中传文件：kubectl cp index.html  myweb-dl5vm:/usr/share/nginx/html/index.html

### 4. deployment
#### 基础操作
因为当rc进行滚动升级的时候，新旧配置文件要求给pod打不同的标签，才能筛选哪个是新的哪个是旧的，当rc更新完成后，生成的新pod和旧pod的lable不一样，导致service根据lable选择pod失败，这时候需要改service配置中的selector为新pod的lable才可以，所以为了解决该问题出现了deployment

##### 创建deployment
1. 在k8s文件夹创建deploy文件夹，新建文件nginx_deploy.yaml文件，执行kubectl create -f nginx_deploy.yaml  之后会创建deployment然后创建rs去生成pod 

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: myweb
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: myweb
    spec:
      containers:
      - name: myweb
        image: nginx:1.13
        ports:
        - containerPort: 80
```

##### 创建service
一条命令创建service，与deployment创建的pod关联起来：

kubectl expose deployment deployment的name --port=node节点要映射出来的端口 --type=NodePort

kubectl expose deployment myweb-deployment --port=80 --type=NodePort

于是创建了一个svc，对外端口是随即创建的31961，

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676081970409-c2a610f3-6736-40f3-91bb-31f99b9e53df.png)

##### 滚动升级
一条命令直接修改deployment

kubectl edit deployment myweb-deployment   然后直接修改保存，保存完后会直接进行创建和删除进行升级

升级完成之后service是可以直接管理的。新的修改会生成新的rs，旧的rs仍然存在，保存着相关的配置信息。我们可以直接进行**回滚操作**  kubectl rollout undo deployment myweb-deployment

查看回滚历史的具体信息  kubectl rollout history deployment myweb-deployment。 通过文件进行创建的deployment不会被记录在内，但是通过命令创建和修改的会被记录



#### 总结：通过命令创建和升级
1. **版本发布**

kubectl run 创建的deployment名字 --image=指定的镜像 --replicas=创建的pod数量 --record

例如：kubectl run myweb-deployment --image=nginx:1.13 --replicas=2 --record

2. **创建service**

kubectl expose deployment myweb-deployment --port=80 --type=NodePort

根据kubectl get all -o wide 可以找到创建的svc所映射出来的端口

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676084295959-044cc265-c0c5-4017-8e19-338782703c05.png)

3. **版本升级**

kubectl set image deploy deployment的名字 容器的名字=新镜像

例如：kubectl set image deploy myweb-deployment myweb=nginx:1.15

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676083066105-83015d86-a842-41d1-a155-c36faa65d4d5.png)

4. **版本回滚**

首先查看历史记录：kubectl rollout history deployment myweb-deployment，

不指定版本默认回滚到上一版本  kubectl rollout undo deployment myweb-deployment

回滚到上一版本 kubectl rollout undo deployment myweb-deployment

指定版本回滚 kubectl rollout undo deployment myweb-deployment --to-revision=1

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676083795783-5db014e8-7c90-42d4-b5e1-34871702fd20.png)

回滚成功后，历史记录变成了2和3

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676083964002-d088a966-dfac-44c7-b896-9353d7911182.png)



### 5. DNS服务
#### 产生背景（service中两种发现机制的不同）
Kubernetes中有一个很重要的特性，服务自发现。一旦一个service被创建，该servicel的service IP和service port等信息都可以被注入到pod中供它们使用。创建的pod越多，注入的环境变量越多

Kubernetes主要支持两种service发现机制：<font style="color:#DF2A3F;">环境变量和DNS</font>。没有dns服务的时候，kubernetes会采用环境变量的形式，一旦有很多个service,环境变量会变得很复杂，为了解决这个问题，我们使用DNS服务。

#### 基本安装使用
1. 在/root/k8s目录下，下载wget https://www.qstack.com.cn/skydns.zip，然后解压
2. 修改vim skydns-rc.yaml ，将![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676105276505-878b06a0-08ab-4639-bd03-a0ed863050f0.png)设置成master节点的地址，然后kubectl create -f skydns-rc.yaml，
3. 修改vim skydns-svc.yaml![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676105483079-092748e8-d498-4605-9536-6c3aa82280ca.png) ，设置dns设置的地址（只要不冲突就行），之后创建即可kubectl create -f skydns-svc.yaml 启动service，                                                                  **启动完成后，**执行kubectl get svc --namespace=kube-system，可以查看到启动的svc的vip地址

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676105808128-693ee4cf-2be5-4439-ac79-33c14140f65f.png)

4. 修改各node节点上的vim /etc/kubernetes/kubelet配置文件，增加如下行：

KUBELET_ARGS="--cluster_dns=10.254.230.254 --cluster_domain=cluster.local"

5. 重启各node节点kubelet:            systemctl restart kubelet
6. ![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676106637375-68b16bbe-d36a-4e43-a98b-6b9bfb6c31fe.png)



#### dns服务的作用
1. pod与pod之间的连接，可以不用每次都去找svc提供的vip地址，直接写svc的名字就可以了，会自动解析

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676188004152-de8acd5b-b070-4a72-8f2c-c5ca60f3a79e.png)

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676107623744-4e7c2109-ef07-4cac-8b0f-2ce0be0ef773.png)

#### 基本流程
![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676188088722-80855aa6-79cc-4e8a-9812-bd656f5eef74.png)

创建的每个pod有自己的ip地址，外界访问不到，而且一更新原有的ip地址就进不去了，需要一个管理者。rc创建pod保证高可用，svc根据pod的lable将一组pod统一管理，提供一个统一的ip地址供其他pod使用，

### 6. pod资源的健康检查
1. 探针种类：
    1. livenessProbe:健康状态检查，周期性 检查服务是否存活，检查结果失败，将**重启**容器
    2. readinessProbe:可用性检查，周期性检查服务是否可用，不可用将从service的endpoints中**移除**，就不会接受负载均衡的请求了
2. 探针的检测方法
    1. exec:   执行一段命令
    2. httpGet:   检测某个http请求的返回状态码，看是不是200-300之间的状态码
    3. tcpSocket:   测试某个端口是否能够连接
3. 演示

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exec
spec:
  containers:
  - name: nginx
    image: 192.168.187.101:5000/nginx:1.13
    ports:
    - containerPort: 80
    args:   # args会替换掉容器的初始命令
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600  #创建文件,休眠30s再删掉文件在休眠600秒
    livenessProbe:  
      exec:  # 探针执行命令，看这个文件有没有，没有就会重启容器
        command:
         - cat  
         - /tmp/healthy
      initialDelaySeconds: 5  # pod初始化之后间隔5s再开始检查
      periodSeconds: 5  # 检查周期5s
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpget # pod名字不能出现大写字母
spec:
  containers:
    - name: nginx
      image: 192.168.187.101:5000/nginx:1.13
      ports:
        - containerPort: 80
      livenessProbe:
        httpGet:
          path: /index.html  # 向该pod80端口发送http请求
          port: 80
        initialDelaySeconds: 3
        periodSeconds: 3
```

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: readiness
spec:
  replicas: 2
  selector:
    app: readiness
  template:
    metadata:
      labels:
      app: readiness
    spec:
      containers:
      - name: readiness
        image: 192.168.187.101:5000/nginx:1.13
        ports:
        -  containerPort: 80
        readinessProbe:
          httpGet:
            path: /index.html
            port: 80
          initialDelaySeconds: 3
          periodSeconds: 3
```

### 7. dashboard管理页面
#### 1. 安装，在k8s文件夹新建文件夹dashboard，新建两个文件如下
        1. vim dashboard-deploy.yaml 修改里面的镜像地址和api-service的地址

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
# Keep the name in sync with image version and
# gce/coreos/kube-manifests/addons/dashboard counterparts
  name: kubernetes-dashboard-latest
  namespace: kube-system
spec:
  replicas: 1
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
        version: latest
        kubernetes.io/cluster-service: "true"
    spec:
      containers:
      - name: kubernetes-dashboard
        image: 192.168.187.101:5000/kubernetes-dashboard-amd64:1.4.1
        resources:
          # keep request = limit to keep this container in guaranteed class
          limits:
            cpu: 100m
            memory: 50Mi
          requests:
            cpu: 100m
            memory: 50Mi
        ports:
        - containerPort: 9090
        args:
         -  --apiserver-host=http://192.168.187.101:8080
        livenessProbe:
          httpGet:
            path: /
            port: 9090
          initialDelaySeconds: 30
          timeoutSeconds: 30
```

        2. vim dashboard-svc.yaml 

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubernetes-dashboard
  namespace: kube-system
  labels:
    k8s-app: kubernetes-dashboard
    kubernetes.io/cluster-service: "true"
spec:
  selector:
    k8s-app: kubernetes-dashboard
  ports:
  - port: 80
    targetPort: 9090

```

        3. 下载镜像  docker pull registry.cn-hangzhou.aliyuncs.com/google-containers/kubernetes-dashboard-amd64:v1.4.1  ，拉取镜像后tag并push到私有仓库和之前的镜像地址对起来就行
            1. docker tag registry.cn-hangzhou.aliyuncs.com/google-containers/kubernetes-dashboard-amd64:v1.4.1 192.168.187.101:5000/kubernetes-dashboard-amd64:1.4.1
            2. docker push 192.168.187.101:5000/kubernetes-dashboard-amd64:1.4.1
    1. 最后执行创建
        1. kubectl create -f dashboard-deploy.yaml
        2. kubectl create -f dashboard-svc.yaml

#### 2. 进入
访问api-service的地址：192.168.187.101:8080，可以看到所有的可访问地址，进入ui页面，也就是192.168.187.101:8080/ui   就可以进入了







### 8. namespace
Namespace在很多情况下用于实现多租户的资源隔离。

cms项目用到数据库，商城项目也需要数据库。

同一个namespace下面不允许出现两个service叫mysql，可以将不同项目用namespace隔离起来，

首先创建namespace，然后创建pod的svc在metadata中加入namespace即可加入分组

kubectl create namespace test

kubectl get namespace  获取namespace

kubectl get svc  --all-namespaces   查到的svc根据namespace进行分组

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676266974392-e8ad6f16-d780-454b-accd-eeaf8fc8f7c0.png)

kubectl get pod/svc/all --all-namespace  查到的东西根据namespace进行分组

kubectl delete namespace test  删除该namespace中的所有资源，非常危险

### 9. heapster
在k8s目录下创建目录heapster，下载wget https://www.qstack.com.cn/heapster-influxdb.zip

解压unzip heapster-influxdb.zip

修改文件夹中的   vim heapster-controller.yaml，修改---source=kubernetes为api-service的地址

```yaml
containers:
name: heapster
image: kubernetes/heapster:canary
imagePul IPol icy:IfNotPresent
command
/heapster
---source=kubernetes:http://192.168.187.101:8080?inClusterConfig=false
---sink=influxdb:http://monitor ing-influxdb:8086
```

于是在dashboard中可以看到相关的cpu、内存占用率

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676269152720-a58ee419-e433-46f9-ab32-4c58331852bc.png)



### 10. 弹性伸缩
Horizontal Pod Autoscaler的操作对象是Replication Controller、ReplicaSet或DepIoyment。对应的Pod,根据观察到的**CPU使用量与用户的阈值**进行比对，做出**是否需要增减实例数量**的决策。controller目前使用heapSter来检测CPU使用量，检测周期默认是30秒。如果cpu使用率高，就创建pod，如果cpu使用率低，就缩减pod



新建一个文件夹/hpa，进去后创建nginx-hpa.yaml文件，创建rc创建两个pod

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myweb
spec:
  replicas: 1
  selector:
    app: myweb
  template:
    metadata:
      labels:
        app: myweb
    spec:
      containers:
      - name: myweb
        image: nginx:1.13
        ports:
        - containerPort: 80
        resources:
          limits:  # 对pod占用的资源进行限制
            cpu: 100m
            memory: 50Mi
          requests: # 容器初始大小的设置
            cpu: 100m
            memory: 50Mi
```

创建hpa ：<font style="color:rgb(38, 38, 38);">kubectl autoscale 资源类型(rc/deployment) 名称 --max=最多pod数量--min=最小pod数量 --cpu-percent=</font>指定目标CPU使用率

kubectl **autoscale** replicationcontroller myweb --max=8 --min=1 --cpu-percent=60

压力测试：

ab -n 500000 -c 100 http://172.16.68.2/index.html

### 11. pv和pvc（k8s持久化）nfs方式（后期扩容管理不方便）
PersistentVolume(一些简称<font style="color:#DF2A3F;">PV</font>):由管理员添加的的一个存储的描述，是一个<font style="color:#DF2A3F;">全局资源</font>，包含存储的类型，存储的大小和访问模式等。它的生命周期独立于Pod,例如当使用它的Pod销毁时对PV没有影响。

PersistentVolumeGlaim(一些简称<font style="color:#DF2A3F;">PVC</font>):是Namespace里的资源，描述对PV的一个请求。请求信息包含存储大小，访问模式等。

#### 创建pv前环境准备（nfs存储方式配置）
1. 所有node节点下载nfs-utils            yum install nfs-utils -y
2. 配置文件修改   vim /etc/exports  映射/data目录，允许ip段访问(读写，同步方式，不做root用户的uid映射，不做所有其他用户的uid映射)

```yaml
/data 192.168.187.0/24(rw,async,no_root_squash,no_all_squash)
```

3. 在master节点创建映射文件夹  mkdir /data/k8s -p，然后重启服务，systemctl restart rpcbind，systemctl restart nfs
4. 在其他node节点上看看能不能访问到，showmount -e 192.168.187.101![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676344148935-4ddbb1a2-b992-47f6-ac8e-1cc05ef82c82.png)

#### 创建pv
1. 在/root/k8s文件夹新建volume，进入之后，新建test-pv.yaml文件，kubectl create -f test-pv.yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: test # 不同pv的name需不同，label可以相同
  labels:
    type: test
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany  # 允许多个pod同时读写
  persistentVolumeReclaimPolicy: Recycle  # pvc回收策略是允许回收
  nfs:  # 创建共享存储方式nfs，需要先在下载nfs
    path: "/data/k8s"  # 文件夹需存在
    server: 192.168.187.101  # nfs服务器地址
    readOnly: false
```

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676344736658-a86b0b8b-51a7-467f-a038-d94e2e7f7940.png)于是pv创建好了

#### 创建pvc
vim test-pvc.yaml    然后kubectl create -f test-pvc.yaml 创建pvc，pvc会优先选择剩余容量小的pv进行创建

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nfs
spec:
  accessModes:
   - ReadWriteMany
  resources:
    requests:
     storage: 1Gi # 需求1g
```

#### 实战，首先创建一个pv用于mysql，
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql # 不同pv的name需不同，label可以相同
  labels:
    type: mysql
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany  # 允许多个pod同时读写
  persistentVolumeReclaimPolicy: Recycle  # pvc回收策略是允许回收
  nfs:  # 创建共享存储方式nfs，需要先在下载nfs
    path: "/data/mysql"  # 文件夹需存在
    server: 192.168.187.101  # nfs服务器地址
    readOnly: false
```

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mysql # pvc的名字，rc中不同container根据pvc的name挂载到不同的pvc中
spec:
  accessModes:
   - ReadWriteMany
  resources:
    requests:
     storage: 10Gi # 需求10g
```

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:5.7
          ports:
          - containerPort: 3306
          env:
          - name: MYSQL_ROOT_PASSWORD
            value: '123456'
          volumeMounts:
          - name: data  # 因为一个pod会起多个container，所以给一个container要挂载的卷起个名字
            mountPath: /var/lib/mysql # 挂载到容器内的什么位置
      volumes:
      - name: data # 和前面不同容器中的volumeMounts的name一致，用来个相应的container挂载具体的位置
        persistentVolumeClaim:
          claimName: mysql # 要挂载到哪个pvc上面，需和pvc一致
```

先创建pv，再创建pvc，然后创建pod关联pvc即可

### 12. 分布式文件系统glusterfs（生产上常用这种方式）
Glusterfs是一个开源分布式文件系统，具有强大的横向扩展能力，可支持数PB存储容量和数千客户端，通过网络互联成一个并行的网络文件系统。具有可扩展性、高性能、高可用性等特点。

####   · 首先安装环境
所有节点：

yum install centos-release-gluster -y            安装gluster的yum源

yum install install glusterfs-server -y             

systemctl start glusterd.service

systemctl enable glusterd.service

mkdir -p /gfs/test1

mkdir -p/gfs/test2

####   · 添加存储资源池
master节点：

gluster pool list  # 查看资源池列表

gluster peer probe k8s-node1  # 添加临时资源池

gluster peer probe k8s-node2

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676376319276-68a35b31-9d57-4e45-96ea-2f55c904ed3d.png)之前做过host解析

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676376422546-eccfac10-8970-4a5a-aa78-54618e6a69db.png)

####   ·  glusterfs卷管理（支持七种卷，分布式复制卷稳定）
1. 创建分布式复制卷（至少需要四个存储单元，一个目录挂载一个空硬盘，一个目录或一个硬盘就是一个存储单元）（replica 2代表相同文件复制两份，存储在不同的存储单元中）

gluster volume create gfs1 replica 2 k8s-master:/gfs/test1 k8s-master:/gfs/test2 k8s-node1:/gfs/test1 k8s-node1:/gfs/test2 force

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676386049300-6aaeb7f6-eed0-4050-93b7-1287f3e4c34b.png)

2. 启动卷

gluster volume start gfs1

3. 查看卷

gluster volume info gfs1

4. 挂载卷（在任何一个卷上，挂载任何一个节点ip地址）

mount -t glusterfs IP地址:/卷名 挂载到/mnt目录下

mount -t glusterfs 192.168.187.101:/gfs1 /mnt

####   · 分布式复制卷扩容
扩容前查看容量：

df -h

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676377647851-d8273ae9-f7c3-4a95-8dac-16b63678b4f8.png)

扩容命令：（将同一个资源池中的空硬盘扩容到已有的卷中，因为是复制卷，所以一次要加两个存储单元）

gluster volume add-brike 要扩容的卷 同一个资源池中节点:/文件目录 同一个资源池中节点:/文件目录

gluster volume add-brick gfs1 k8s-node2:/gfs/test1 k8s-node2:/gfs/test2 force

扩容后查看容量：

df -h

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676377908628-ebd0fb2a-700b-49a1-bfb8-c3a2861295af.png)

####   · glusterfs大体使用总结
之前将三个node节点注入到一个资源池中，然后将master节点中的两个存储单元和node1节点中的两个存储单元作为一个卷作为一个卷gfs1，挂载到了master节点的/mnt目录下，然后又将node2节点上的两个存储单元挂载了进去进行扩容。我们在master节点的/mnt目录下存储文件， 实际文件存储位置是分散在了三个不同的资源池中

master节点

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676386925281-18138f38-15c2-4ac0-9176-2fd81a1e0c26.png)

node1节点

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676386952910-dc6dcc5e-1b7d-4636-9dc9-7ec3045dc33f.png)

node2节点

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676386984092-cfcf5c31-4f27-4d19-a708-52877498f8f5.png)

文件根据hash值分配的，并不均衡。因为创建时副本数是2，所以文件拷贝了两份

####   · k8s使用glusterfs
1. 首先创建endpoint

在k8s目录下mkdir clusterfs，vim glusterfs-ep.yaml  ，create之后，可以通过kubectl get np获取详情

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: glusterfs
  namespace: default
subsets:
- addresses:  # 有多少个gluster的节点就增加几个ip
  - ip: 192.168.187.101
  - ip: 192.168.187.102
  - ip: 192.168.187.103
  ports:
  - port: 49152 # gluster服务默认端口
    protocol: TCP 
```

2. 创建service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: glusterfs  # 和ep通过name相连，要和ep的metadata下的name相同才能匹配起来
  namespace: default
spec:
  ports:
  - port: 49152
    protocol: TCP
    targetPort: 49152
  sessionAffinity: None
  type: ClusterIP 
```

![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676390592814-bcfd437b-4d48-480c-9782-0a347cad94bb.png)

3. 创建gluster类型pv

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: gluster
  labels:
    type: glusterfs
spec:
  capacity:
    storage: 5Gi 
  accessModes:
  - ReadWriteMany
  glusterfs:
    endpoints: "glusterfs"  # 要关联的ep的name
    path: "gfs1"  # 这就是创建的卷的名字，可以通过gluster volume list 查看卷列表
    readOnly: false 
```

4. 创建pvc

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: gluster
spec:
  accessModes:
   - ReadWriteMany
  resources:
    requests:
     storage: 5Gi # 需求5g
```

 ![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676391261919-56612c56-4e85-4c68-b1fd-c796eb900cba.png)可以看到，此时pvc绑定在了name为gluster的pv上

5. 在pod中使用gluster

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: web
spec:
  containers:
   - name: nginx
     image: nginx:1.13
     ports:
      - containerPort: 80
        hostPort: 2333 # 不经过svc直接通过部署在该节点上的端口访问
     volumeMounts:
      - name: nfs-vol2
        mountPath: /usr/share/nginx/html
  volumes:
  - name: nfs-vol2
    persistentVolumeClaim:
      claimName: gluster 
```

## 3、创建docker私服
#### 一、官方registry
首先在私服docker上下载镜像启动容器，然后别的服务器如果想push和pull首先要将私服的地址作为受信任的地址，修改配置文件，然后重启服务之后就可以push和pull了

    1. docker pull registry
    2. 修改配置文件
        1. vim /etc/sysconfig/docker，修改docker配置文件中的镜像地址和受信任的仓库地址

```javascript
OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false --registry-mirror=https://registry.docker-cn.com --insecure-registry=192.168.187.101:5000'
```

        2. systemctl daemon-reload
        3. systemctl restart docker.service
    3. docker run -d -p 5000:5000 --restart=always --name registry -v /opt/myregistry:/var/lib/registry registry   启动私有仓库，每次启动docker 就会启动该容器--restart=always.
    4. 传送镜像：
        1. **<font style="color:rgb(77, 77, 77);">docker tag</font>**<font style="color:rgb(77, 77, 77);"> docker.io/nginx:1.15 192.168.187.101:5000/nginx:1.15   </font>
            1. <font style="color:rgb(77, 77, 77);">解释：docker tag 本地已有的images 私有仓库地址/image名:版本，会在本地生成名为私有仓库地址/image名:版本的镜像</font>
        2. **<font style="color:rgb(77, 77, 77);">docker push </font>**<font style="color:rgb(77, 77, 77);">192.168.187.101:5000/nginx:1.15  （push 生成的tag名即可）</font>
            1. <font style="color:rgb(77, 77, 77);">如果push不上去，而且配置了文件之后。将私有仓库服务器上的</font>** selinux 关闭**就好了。可以在仓库服务器上执行setenforce 0，暂时禁止，如果成功push之后可以vim /etc/selinux/config，将里面的selinux 设置为disabled即可
        3. docker rmi 192.168.187.101:5000/nginx:1.15  push上去之后删除tag，因为tag会产生新的images。
    5. 拉镜像
        1. <font style="color:rgb(77, 77, 77);">docker pull 192.168.187.101:5000/nginx:1.15</font>
    6. 查看私有仓库有什么镜像
        1. <font style="color:rgb(77, 77, 77);">curl -XGET</font>**<font style="color:rgb(77, 77, 77);"> </font>**http://192.168.187.101:5000/v2/_catalog

#### 二、harbor私服
## 5、总结
![](https://cdn.nlark.com/yuque/0/2023/png/28792475/1676189341781-789fcc92-f590-4d56-9bf4-dae832ab292a.png)





















