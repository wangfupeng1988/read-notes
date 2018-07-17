# 《docker入门》学习笔记

视频地址 https://www.imooc.com/learn/867 ，这个视频讲解的并不是那么入门，我又搜索了一下资料，配合看。例如 https://blog.csdn.net/S_gy_Zetrov/article/details/78161154 这篇博客就不错。

## 什么是 docker

docker 一般称之为容器技术，可以简单的理解为更加灵活、轻量、可移植的虚拟机。但是它不是虚拟机，它的实现原理和虚拟机不一样，具体看视频中的第二章。

可以对比之前玩虚拟机的过程来了解 docker 。首先需要安装 vmware 软件，这里也得安装 docker 。然后，虚拟机需要下载各种 iso 镜像文件，docker 也得去 search 或者 pull 远程的镜像。最后，虚拟机需要通过镜像安装为系统，docker 也可以通过影响 run commit 生成一个容器。还有人将 docker 的镜像比作是面向对象的 class ，容器比作是 instance ，这样也很形象。

虚拟机是 PC 时代的产物，而 docker 是互联网、大数据时代的产物，有时代特点。docker 直接可以从自己的镜像库 pull 镜像，还可以 push 自己的镜像到镜像库，跟 github 很像。例如，你在本地开发机搞了一个镜像，push 上去，然后登录测试机 pull 下这个镜像，即可直接运行。

## mac os 安装

安装 homebrew 并且更新镜像源，运行`brew cask install docker`即可安装。我在安装过程中遇到过无法下载安装文件（被墙）问题，开启科学上网就可以了。

安装完成之后，命令行提示已经将`.app`文件拷贝到`Applications`文件夹，然后就可以像其他 App 那样去启动了。启动成功之后，运行`docker --version`可以看到版本。

接下来你还需要注册账号，来 https://cloud.docker.com/ 这里注册、登录即可。不过根据我的操作，这一步依赖于科学上网。注册、登录成功之后，点击菜单栏 docker 小图标，登录，就 OK 了。

最后，你还要选择国内的镜像源，默认国外的下载太慢了。来 https://www.daocloud.io/ 这里注册然后登录，进入个人页面之后，页面右上部分有一个加速器的 icon 。点击进入加速引导页面，跟着指示操作即可。

## 镜像操作

- 搜索镜像`docker search mysql`，结果中带有`/`是用户自己提交的，不带`/`是经过 docker 官网认证的
- 下载镜像`docker pull mysql`，配置了国内镜像源之后下载速度会很快。这样，你就不用再在本地亲自安装 mysql 了
- 显示本地镜像`docker images`
- 发布镜像`docker push image_name`
- 移除镜像``

## 容器操作

待补充……

## 总结

待补充……

