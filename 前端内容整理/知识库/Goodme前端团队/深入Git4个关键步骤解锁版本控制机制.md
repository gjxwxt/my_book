![cover_image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicG9CszOyhWrA1pWoZbvuK3GYibJK2icRG6So7M82hiaiaKyTcNkxlEibZ8WyJDjS5bJY2HIysCh0lI7lBw/0?wx_fmt=jpeg)

#  深入Git：4个关键步骤解锁版本控制机制

原创  陆晨杰  [ Goodme前端团队 ](javascript:void\(0\);)

__ _ _ _ _

本篇文章主要面向对git的使用有一定了解的同学，通过对Git底层命令的介绍来理解git内部的工作机制，从而更好的学习并理解如何使用Git与为何是如此运作的。

##  基础知识

###  Git目录结构

当我们需要使用Git来进行版本控制时，第一步就是执行 ` git init ` 进行版本库的创建，此时Git会创建一个 ` .git `
的目录，这个目录包含的git存储的所有的信息。这个目录目录的部分目录结构如下：

  * **info** ：包含了一些用于存储和管理版本库元数据的文件，如exclude文件会配置不希望被追踪的文件或目录（类似.gitignore） 
  * **config** ：项目特有的配置信息，比如用户的姓名、邮件、远程地址等 
  * **object** ：包含了git中所有的对象，是Git用来存储项目历史的核心数据，我们后续会进行介绍 
  * **refs** ：存储着指向数据的提交对象的指针 
  * **HEAD** ：当前被检出的分支 我们在目录下还可能会发现如 ` description ` 、 ` hooks ` 等文件或目录，我们这次不讨论这些内容；Git的完整的目录结构与描述，可以阅读官方文档进行学习 

###  存储方式

首先，我们需要知道的是，Git的核心部分是一个键值对数据库，你可以通过向Git插入 **任意**
类型的内容获得一个唯一键，并且通过该唯一键来取回对应的内容。存储的数据将保存在上一段中我们提到的object的文件夹（即对象数据库）中。我们可以尝试新建一个版本库并执行
` git add ` 来演示效果

    
    
    $ git init test  
    $ cd test  
    $ ls .git/objects  
    info pack  
    $ echo "hello world" >> a.txt && git add .  
    $ ls -R .git/objects  
    3b info pack  
    .git/objects/3b:  
    18e512dba79e4c8300dd08aeb37f8e728b8dad  
    .git/objects/info:  
    .git/objects/pack:  
    

我们可以看出，objects多了一个在hash的前两位为文件夹名称（3b），其余38位作为文件名的文件来存储刚才我们添加的文件，而文件的内容则是将内容转化为二进制并压缩后生成的。

> GIt提供了 ` cat-file ` 的命令通过传递hash来读取对应二进制文件的内容，比如当前的文件我们可以执行 ` git cat-file -p
> 3b18e512dba79e4c8300dd08aeb37f8e728b8dad ` ，此时会即会答应出文件原本的内容：hello world

##  git add

上一段中，我们使用的 ` git add ` 来演示了将项目存储进版本库的效果，现在我们通过介绍一些底层命令的使用来拆解分析 ` git add `
的工作本质

###  保存内容

我们重新初始化一个版本库，新建相同的文件后，可以执行 ` git hash-object `
向数据库插入一条数据，此时可以看到，object文件夹中也有了一个相同的文件

    
    
    $ git init dismantle  
    $ cd dismantle  
    $ echo "hello world" >> a.txt && git hash-object -w ./a.txt  
    $ ls -R .git/objects  
    3b info pack  
    .git/objects/3b:  
    18e512dba79e4c8300dd08aeb37f8e728b8dad  
    .git/objects/info:  
    .git/objects/pack:  
    

> ` hash-object ` 默认仅会计算出对应的hash，我们通过添加 ` -w ` 参数来指明该命令不要只返回hash，还需要将内容写入数据库中

###  暂存区

那么问题来了，我们执行了两条不同的命令，同样都向数据库中插入了数据；那么这两条命令的区别在哪里呢？答案就是暂存（staged或index） 我们可以执行 `
git status ` 命令来查看当前文件的状态；可以看到 ` test ` 目录中，文件已经添加到了暂存区，而 ` dismantle ` 中并没有
我们可以执行 ` update-index ` 将文件添加至暂存区中；当一个文件还不在暂存区时，需要添加 ` --add ` 参数，同时通过 `
--cacheinfo ` 来指定需要添加到暂存区的文件的类型[^2]、hash、文件名

    
    
    ```shell  
    $ cd test && git status -s  
    A  a.txt  
    $ cd dismantle && git status -s  
    ??  a.txt  
      
    $ git update-index --add --cacheinfo 100644 3b18e512dba79e4c8300dd08aeb37f8e728b8dad a.txt  
    $ git status -s  
    A  a.txt  
    

当我们将文件添加至暂存区后，我们可以执行 ` status ` 、 ` diff ` 等命令查看返回的结果，我们可以看到两个目录下，返回的结果是相同的

> 对于暂存区或Git提交操作不太了解的同学，可以查阅Git的官方的小册内容

##  git commit

###  存储对象

在我们聊commit的过程之前，我们需要先了解Git的存储对象；Git一共有四种类型的存储对象：数据（blob）、树（tree）、提交（commit）、标签（tag)，我们本篇只讨论前三种。

我们先在 test工程中进行一次 ` commit ` 操作，此时，可以看到我们在objects下多了两个文件

    
    
    $ git commit -m 'feat: 2.5'  
    $ ls -R .git/objects  
    3b   eb   f4   info pack  
    .git/objects/3b:  
    18e512dba79e4c8300dd08aeb37f8e728b8dad  
    .git/objects/eb:  
    aa691b5554f29ac9d4f37811a1da6f24d376a1  
    .git/objects/f4:  
    100ba8b3119f593a2b89c7284cf66d4be739b3  
    .git/objects/info:  
    .git/objects/pack:  
    

如前半部分文章中我们通过 ` add ` 或 ` hash-object ` 生成文件的内容就是一个 **数据对象** ，我们当时通过 ` git cat-
file ` 可以查看对应的内容，可以看出其仅保存了文件的内容信息；这类型的对象我们称之为 **数据对象**
。但是我们在开发一个项目的过程中，仅知道代码的内容肯定是不够的，我们还需要通过文件名来检索代码、管理依赖等， **树对象**
就是来解决这个问题的。一个树对象包含了一条或多条树对象记录（tree entry），每条记录含有一个指向数据对象或者子树对象的
hash，以及相应的模式、类型、文件名信息。例如，当前这次提交的生成树对象为

    
    
    $ git cat-file -p ebaa691b5554f29ac9d4f37811a1da6f24d376a1  
    100644 blob 3b18e512dba79e4c8300dd08aeb37f8e728b8dad a.txt  
    

此时我们通过树对象和数据对象可以还原出某个时刻下项目工程中的所有文件，那么如何将所有的提交串起来呢？显而易见的， **提交对象**
就是来处理这个问题的。提交对象包含着一次提交的信息：当时树对象、父提交（如有）、作者信息、提交注释，一次提交的内容如下：

    
    
    $ git cat-file -p f4100ba8b3119f593a2b89c7284cf66d4be739b3  
    tree ebaa691b5554f29ac9d4f37811a1da6f24d376a1  
    author gugu <gugu@gmail.com> 1697353561 +0800  
    committer gugu <gugu@gmail.com> 1697353561 +0800  
    feat: 2.5  
    

我们可以通过提交对象来将所有的commit串起来，再通过对应的树对象和数据对象，检索出对应提交时所有的内容

> 我们在上文提到 ` cat-file ` 可以打印出对象的内容，其实此命令也可以打印对象的类型，只需要将 ` -p ` 替换为 ` -t `
> 即可，大家可以自己尝试，本文不再赘述

###  生成树对象和提交对象

与 ` add ` 相同，我们也可以调用GIt的底层命令来自己完成commit这个操作，首先我们可以通过 ` write-tree ` 来生成一个树对象

    
    
    $ cd dismantle && git write-tree  
    ebaa691b5554f29ac9d4f37811a1da6f24d376a1  
    $ git cat-file -p ebaa691b5554f29ac9d4f37811a1da6f24d376a1  
    100644 blob 3b18e512dba79e4c8300dd08aeb37f8e728b8dad a.txt  
    

可以看到，我们生成了一个与test项目中一模一样的树对象，然后我们再通过 ` commit-tree ` 来进行代码的提交，生成一个提交对象

    
    
    $ git commit-tree ebaa691b -m 'feat: 2.5'  
    f1ded58d3f850515daa3636efce0598bbe9a1180  
    $ git cat-file -p f1ded58d3f850515daa3636efce0598bbe9a1180  
    tree ebaa691b5554f29ac9d4f37811a1da6f24d376a1  
    author gugu <gugu@gmail.com> 1697354831 +0800  
    committer gugu <gugu@gmail.com> 1697354831 +0800  
      
    feat: 2.5  
    

提交对象的内容除了提交的时间，其余的内容都是与test项目中的相同（此时或许你会有疑问，为什么这次生成的hash都是不同的，没关系，我们最后再来说这个问题）

###  分支

那么我们的提交操作到此就结束了吗？给大家5秒钟的时间来思考这个问题 。。。。。。细心的小伙伴肯定发现了，不对，两者还有差异，我 ` git log `
怎么报错呢？

    
    
    $ git log master  
    fatal: ambiguous argument 'master': unknown revision or path not in the working tree.  
    Use '--' to separate paths from revisions, like this:  
    'git <command> [<revision>...] -- [<file>...]'  
    

从报错中可以得处，我们竟然还没有master分支？这不科学！大家是否有思考过，上文中我们可以通过存储对象来得到整个项目的结构、内容等，但是在切换分支的时候又是怎么做到同样的事情呢？项目中分支的信息又是存储在哪的呢？这里就得说一下，分支的本质即为指向一系列提交之首的一个引用，其信息会保存与
` refs/headers ` 下，以分支名为文件名，提交的 hash 为内容的文件 也就是说，我们距离生成一次提交，还剩下更新分支，将引用指向最新的提交

    
    
    $ git update-ref refs/heads/master f1ded58d3f850515daa3636efce0598bbe9a1180  
    $ ls .git/refs/heads  
    master  
    $ cat .git/refs/heads/master  
    f1ded58d3f850515daa3636efce0598bbe9a1180  
    

至此，完整的一次提交便结束咯

##  补充

###  对象文件的生成规则

还记得我们上文中的那个问题嘛？为什么两个项目中只有那个提交对象的文件名是不同的呢？我们来看一下Git是如何生成对象文件的hash和二进制内容的吧
Git会先生成一个以对象类型开头，随后加一个空格和内容的字节数，最后是一个空字节的头部信息；将头部信息和文件的内容拼接后进行 ` SHA-1 `
校验和得出的hash值即为对象文件的名称，通过 ` zlib ` 压缩得到的信息作为文件的内容 下方的node代码模拟的hash和内容的生成过程，并通过 `
Git ` 的命令进行验证，逻辑无误

    
    
    const { deflateSync } = require('zlib');  
    const crypto = require('crypto');  
    const fs = require('fs');  
    const addFile = (content) => {  
     const headers = `blob ${content.length}\0`;  
     const shasum = crypto.createHash('sha1')  
     shasum.update(headers + content)  
     const hash = shasum.digest('hex')  
     console.log('hash: %s', hash); // e0501eec17daa40898f8340ca52af1949852025e  
     const deflatedContent = deflateSync(headers + content);  
     const dirname = hash.slice(0, 2);  
     const fileName = hash.slice(2);  
     if (!fs.existsSync(`.git/objects/${dirname}`)) {  
      fs.mkdirSync(`.git/objects/${dirname}`, { recursive: true });  
     }  
     fs.writeFileSync(`.git/objects/${dirname}/${fileName}`, deflatedContent, { encoding: 'hex' });  
    }  
      
    addFile("this is a demo");  
      
    
    
    
    $ echo -n "this is a demo" | git hash-object --stdin  
    e0501eec17daa40898f8340ca52af1949852025e  
    git cat-file -p e0501eec17daa40898f8340ca52af1949852025e  
    this is a demo  
    

##  总结

我们先借用git book的一张图，来总结我们的整个数据库的结构
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicG9CszOyhWrA1pWoZbvuK3GxeFPG9AyN06vEPON1ohgkt0WGibU7d6T84iaic6jyhagqfUia6ryVmX1Qw/640?wx_fmt=png)

##  最后

📚 小茗文章推荐：

  * [ 5分钟回顾webpack的前世今生 ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247484423&idx=1&sn=c25ae2044d1a8c46f160ba5b75cf0725&chksm=cfe58300f8920a1650eae4220f430099430b6f6765aa0c11d270d3a3accf94876dea310b97a5&scene=21#wechat_redirect)
  * [ 遥遥领先！古茗门店菜单智能化的探索 ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247484400&idx=1&sn=133f12f491b9072b6b73283838b755d8&chksm=cfe584f7f8920de1b3b0fe32cf8b784b5173f7b82fc890269e4825cde79e218b31426e4f6500&scene=21#wechat_redirect)
  * [ 一个破RPC，又又又让我复习了一遍计算机网络 ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247484328&idx=1&sn=e51bfdcfa4fbd5de6906136fe6555d3b&chksm=cfe584aff8920db984069cf653b42108254a7cef4d97abef92c22e39a1cb1edd472534811205&scene=21#wechat_redirect)   

关注公众号「Goodme前端团队」，获取更多干货实践，欢迎交流分享~

预览时标签不可点

微信扫一扫  
关注该公众号





****



****



×  分析

  收藏

