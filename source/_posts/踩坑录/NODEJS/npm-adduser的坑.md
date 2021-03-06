postid: "npm-adduser-refuse"
title: "npm adduser的坑"
date: 2015-08-05 20:10:32
categories: [踩坑录]
tags: [nodejs, npm]

---

# 场景回放

今天在折腾[nproxy2](https://github.com/gejiawen/nproxy2)时，无意中发现了一个诡异的问题。

问题总结起来是这样的，

使用正确的[npmjs.org](http://www.npmjs.org)用户名和密码进行`npm adduser`操作，但是给出的结果都是不停报错。

而且报错的信息还很诡异，如下

![](//images0.gejiawen.com/posts/npm-adduser-refuse/001.png)

提示我说用户名或者密码错误？！！我就惊了个呆了，我都已经登陆上[npmjs.org](http://www.npmjs.org)了，怎么还说我用户名或者密码错误呢？


# 追根究底

一般人的，当提示你用户名或者密码错误时，肯定是首先怀疑自己输错了。

当然我的第一反应也是这样的。然后我连续试了好几次，确定我肯定没有手误输错。那么问题出在哪里呢？

于是开始在网上各种找答案。网上说啥的都有，不过就是没有专门说`npm adduser`出错的问题的。大部分的nodejs入门级教程中，对`npm adduser`都是一笔带过，让读者知道通过这个命令可以将自己的包发布上npm上。所以我并没有得到一些额外的帮助信息。

折腾了半天，仍然没有解决。最后我使出了大招，换了台机器试试看。

在Windows上一试便通过了，但是在Mac上却一直报错，难道不支持Mac OS？！说不支持Mac OS我自己都不信！那么为什么一台机器上可以，另一台机器却报错呢？

难道跟npm的配置文件有关系？那我就来对应下两个环节的`.npmrc`文件.

不对比不知道，一对比我就惊醒了。

Mac OS中的`.npmrc`中多了一行配置，

```bash
registry=http://registry.npm.taobao.org
```

妈了个蛋的，终于找到了罪魁祸首了。就是因为多了这一行配置。

因为重定向了npm库的源，所以`npm adduser`时会将用户名和密码提交到`http://registry.npm.taobao.org`去验证，那当然一直报用户名密码不正确啦！

# 闭宗结案

这次这个坑，其实并不是什么深奥的问题，但是却没有在第一时间想到配置文件这一层面上，导致走了很多弯路。

出了问题了，首先想到的是去网络上找答案，一般情况下找到答案的速度与你是否能够熟练使用搜索引擎有很大关系。但是最悲剧的莫过于你找到的那条记录提到的问题跟你一样，更关键的是也没解决！

所以，出了问题，所以还是应该自己思考一番啊！

嗯。




