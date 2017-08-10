postid: 'note-for-deploy-gogs'
title: gogs部署小记
categories: [踩坑录]
tags: [gogs, 部署]
date: 2016-05-31 14:22:37
---

[Gogs](http://gogs.io)是一款使用golang编写的轻量的、开源的、自助式git托管服务。其功能与[github](http://github.com)及[gitlab](http://gitlab.com)比较相似，不过gogs的部署要比前者简单的多，而且其功能也比较轻量。

因为公司内部没有一个比较完善的代码托管系统，这两天在调研这块内容。本来备选的解决方案有gitlab和gogs，最后基于部署成本的考虑选择了gogs。其实还有另外一个原因，就是我个人比较喜欢go语言，也是我个人的业务爱好语言之一。

言归正传，本文将简要的记录如何在centos7上部署gogs服务。

# 安装过程

在部署gogs之前，我们需要做一些准备工作。笔者的本地环境是mac，服务器环境是centos7。

## 安装golang环境

首先我们要安装golang语言。由于一些不可抗的因素，golang的[官网](https://golang.org/)在国内访问十分不稳定。如果没有梯子，可以访问[golang中国](http://golangtc.com/)下载golang。同时golang中国也是go语言爱好者交流分享的一个好地方。

在笔者的本地环境中，可以通过brew来安装golang，

```bash
$ brew install go
```

若安装过程出现问题，不妨先`brew upgrade`试试。

安装完毕之后，我们可以通过如下代码来测试一下是否安装成功，

```bash
$ go version
```

其输出类似如下，

```bash
$ go version go1.6.2 darwin/amd64
```

可以看出安装的golang的版本及平台架构。

## 数据库的选择和安装

除了golang环境之外，gogs服务还需要数据库的支持。

gogs支持的数据库类型非常灵活，基本常见的主流数据库都支持。

你可以按照自己的习惯来选择mysql或者postgrsql。甚至你可以不安装关系型数据库，直接使用sqlite3或者TiDB。

笔者选择的是mysql，下面简要说一下在centos7上如何安装mysql。

在一个全新的centos7系统上，如果直接运行

```bash
yum install mysql
```

它会提示安装mariadb和mariadb-libs这两个包。

因为在centos7中，默认的yum源中并没有mysql，只有mysql的分支版本mariadb。如果你不介意的话，你当然可以直接谁用mariadb，因为mariadb跟mysql的功能基本一致。如果你一定要使用mysql，那我就给你指条明路。

首先需要安装mysql的源，

```bash
$ wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
```

然后安装rpm包，

```bash
$ sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
```

在安装这个rpm包后会得到两个yum的repo源，

- `/etc/yum.repos.d/mysql-community.repo`
- `/etc/yum.repos.d/mysql-community-source.repo`

最后，我们可以运行`yum install mysql-server`来安装mysql了。在安装之前，你会得到你需要安装一批软件包的提示，你只需要按下yes即可。

在安装完mysql之后，我们还需要创建一个数据库，将之取名为gogs，

```sql
CREATE DATABASE gogs CHARACTER SET utf8 COLLATE utf8_bin;
GRANT ALL PRIVILEGES ON gogs.* TO ‘root’@‘localhost’;
FLUSH PRIVILEGES;
```

## 安装gogs

gogs提供多重方式进行安装，比较常用的有两种。一个是二进制安装，另一个是通过源代码安装。

笔者这里选择的是通过二进制方式来安装。即下载与服务器架构匹配的二进制包，然后解压即可用。

然后切换到解压得到的目录之后，直接运行`./gogs web`即可在服务器上运行gogs服务了。

你可以直接运行`./gogs -h`得到更多的命令行参数说明。

笔者一般采用如下的方式运行gogs，

```bash
nohup gogs/gogs web > log/gogs_web.log 2>&1 &
```

## gogs的配置

启动gogs服务之后，首次运行会让你进行相关的配置。主要分为3个配置项，一是数据库的配置，包括数据库地址及密码；二是服务的应用配置，包括域名，路径等等；三是可选邮件服务和管理员配置。

具体的配置项含义可参考官方的[配置文件手册](https://gogs.io/docs/advanced/configuration_cheat_sheet)。

# 遇到的坑

gogs总体来说，配置还是比较简单的。没有过多的配置，就是几个基础环境的安装。

笔者在部署的过程中，遇到如下几个坑，

- centos7默认没有mysql的源
- nginx反向代理的坑


# 拓展

由于笔者是在自己的乞丐版阿里云上进行的部署，为了节省资源，使用了nginx做反向代理。

这里仅简单说下如何使用nginx给gogs服务做反向代理。

首先，添加nginx的yum源，

```bash
$ wget  http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
$ rpm -ivh nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

然后安装nginx，`yum install nginx`。

安装完毕之后，`nginx -t`测试一下nginx是否正确安装。得到实际的nginx配置文件地址。一般是`/etc/nginx/nginx.conf`。

nginx的反向代理其实是一个虚拟主机，我们需要在`/etc/nginx`目录下新建一个`vhosts`目录，在`vhosts`目录中创建一个`gogs.conf`文件，内容如下

```ini
server {
    listen 80;
    server_name gogs.gejiawen.com;
    access_log /root/log/gogs.log main;
    location / {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://0.0.0.0:3000;
    }
    error_page 500 502 405 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}
```

这个配置文件中，根据各自的情况，只需要改动`server_name`和`access_log`即可。

然后在默认的nginx配置文件（`/etc/nginx/nginx.conf`）中，加载虚拟主机的配置。如下，

```ini
...

http {
    ...
    # 包含所有虚拟主机的配置文件
    include /etc/nginx/vhosts/*.conf;

}
```

最后，重新加载nginx的配置即可。

```bash
$ nginx -s reload
```

体验地址：[gogs.gejiawen.com](http://gogs.gejiawen.com)


# 参考列表

- [官网](http://gogs.io)
- [README_ZH.md](https://github.com/gogits/gogs/blob/master/README_ZH.md)



