---
title: 第一篇博客
date: 2021-10-26 12:31:44 

tags:
- 日常
index_img: /img/sunrise.jpg
---





Hexo多台电脑写博客：https://www.dazhuanlan.com/frank87/topics/1040043

<!--more-->

##### 2021.10.22

上传项目到github,有几个测试数据文件太大,超过了100m,push到github报错,直接删除了那几个文件

```
git add .
git status
git commit -m '删除大文件'
```

之后执行 `git push -u origin master `还是报错,那几个文件还是存在了本地, `.git`文件夹也是很大,保存了删除掉的文件.

重复了几次上面步骤问题依旧

解决办法:删除 `.git` 文件夹,重新

```
git init
git remote add origin git@github.com:th66778899/Algorithms4th.git
```

之后再进行 `git add . git push -m '' git push -u origin master `没有问题,成功上传项目到github

具体原因之后还需要再学习git使用

ssh默认不支持rsa了 https://silenwang.github.io/2021/10/11/ssh默认不支持rsa了/

配置多个git账号 : https://blog.csdn.net/q13554515812/article/details/83506172

gradle使用

git基本使用 https://zhuanlan.zhihu.com/p/30044692

ssh-keygen -t rsa -C "[2391415158@qq.com](mailto:2391415158@qq.com)"

ssh-keygen -t rsa -f ~/.ssh/id_rsa_github -C "[2391415158@qq.com](mailto:2391415158@qq.com)"

ssh -T [git@github.com](mailto:git@github.com) -i ~/.ssh/id_rsa_github

dev分支测试

建立新分支,需要 `git push -u origin dev` 将本地dev分支 与 远程分支建立流联系,需要在远程仓库创建对应的分支

$ git push fatal: The current branch dev has no upstream branch. To push the current branch and set the remote as upstream, use

```
git push --set-upstream origin dev
```

=======

dev 和 master分支 `git checkout master | git checkout dev`

会导致对应分支下的文件发生覆盖问题

master分支和 dev 分支 merge 问题

同一个文件 两分支内容不一致,先后更改 666 git分支的使用 https://zhuanlan.zhihu.com/p/137855358



[About - Nyima's Blog (gitee.io)](https://nyimac.gitee.io/about/)

https://nyimac.gitee.io/about/