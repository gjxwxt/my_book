sso需要一个独立的**<font style="color:#E8323C;">认证中心</font>**，所有子系统都通过认证中心的登录入口进行登录，登录时带上自己的地址，子系统只接受认证中心的授权，授权通过令牌（token）实现，<font style="background-color:#FADB14;">sso认证中心验证用户的用户名密码正确，创建全局会话和token，token作为参数发送给子系统，子系统拿到token，即得到了授权</font>，可以借此创建局部会话，局部会话登录方式与单系统的登录方式相同。

<font style="color:rgb(77, 77, 77);">用</font>**<font style="color:rgb(77, 77, 77);">cookie作为媒介存放用户凭证</font>**<font style="color:rgb(77, 77, 77);">。 用户登录系统之后，会返回一个加密的cookie，当用户访问子应用的时候会带上这个cookie，授权以解密cookie并进行校验，校验通过后即可登录当前用户。</font>

1. 访问子系统1受保护资源-->系统1发现未登录将自己的地址作为参数跳转到<font style="color:rgba(0, 0, 0, 0.75);">sso认证中心-->sso认证中心发现没有登录-->将登录页面返回给客户端-->（用户名、密码、系统1的地址传入sso认证中心）登陆之后生成全局令牌-->sso认证中心带着全局令牌跳转到系统1并将cookie写入本地-->系统1带着令牌去sso认证中心再验证一遍成功后认证中心注册系统1返回有效,于是系统1返回被保护资源并且再系统1中写入登录态。</font>

![](https://cdn.nlark.com/yuque/0/2022/png/28792475/1661010932967-780b90b8-401e-444a-b76f-703bdac934b0.png)

![](https://cdn.nlark.com/yuque/0/2022/png/28792475/1661011233412-aea43913-8009-45c2-b023-1d99c247b22c.png)

2. <font style="color:rgba(0, 0, 0, 0.75);">访问系统2受保护资源--></font>系统2发现未登录将自己的地址作为参数跳转到<font style="color:rgba(0, 0, 0, 0.75);">sso认证中心-->sso认证中心发现已经登录返回全局令牌-->系统2带着令牌去sso认证中心再验证一遍成功后认证中心注册系统2返回有效于是系统2返回被保护资源</font>

![](https://cdn.nlark.com/yuque/0/2022/png/28792475/1661011152888-5e4c9fdc-4d0f-4ec2-b344-5f138e3be2d5.png)

---

> [https://blog.csdn.net/XH_jing/article/details/114063823](https://blog.csdn.net/XH_jing/article/details/114063823)
>



