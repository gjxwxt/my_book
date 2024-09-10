##  工作流程
1. 由`开发管理员`负责在Gitlab上创建空白的仓库，并clone到本地，在sourcetree的git flow菜单中选择初始化仓库，并push到远端。
2.  在Gitlab上设置保护分支，把master、develop分支保护起来，只有指定人可push。
3. `功能开发者`clone代码到本地，先在sourcetree的git flow菜单中选择初始化仓库。
4. 然后再开始新建功能分支，进行开发工作。
5.  新功能开发全部完成或部分完成后，`功能开发者`把最新代码push到远端同样的新功能分支里，并在Gitflow发起pull request给`开发管理员`
6. `开发管理员`review代码，选择合并代码到develop，并可选择删除已经合并的新功能分支
7. 当`开发管理员`处理完合并请求后，开发者,切换到develop分支，直接删除自己的本地分支及远程分支，不要点击完成(Finish Feature)，此时开发者pull远端develop分支最新代码即可，可忽视本地的push提醒。
8.  release、hotfix分支和feature分支操作类似。
9. 不可点击完成新功能、完成发布版本、完成修复补丁，因为这样会导致自动合并代码到master或develop分支

#### git flow 和客户端sourcetree的使用
[开发文档_05git开发规范(草稿).md](https://www.yuque.com/attachments/yuque/0/2022/md/28792475/1661494982485-5e3e8a01-9e52-49c7-939f-27ca614c33f8.md)

[一文读懂Git工作流](https://zhuanlan.zhihu.com/p/266916800)

