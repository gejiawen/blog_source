postid: 'all-things-you-need-to-know-about-nginx'
title: nginx必知必会
categories: [CODER]
tags: [web开发, nginx]
date: 2018-10-27 13:11:24

---

[Nginx](http://nginx.org/)是一款俄罗斯人写的开源web服务器，当然它除了是一个web服务器之外，还有许多其他的使用姿势。

本文将会着重介绍在平时进行web开发时会使用到的功能。本文不会教如何进行nginx配置文件编写、不会介绍任何第三方的module。

那么我们在进行web开发时，一般会在什么场景下需要用到nginx呢？

我总结了如下几点，我个人认为这几点也是nginx最最常用的使用姿势，应该是每一个从事web开发同学应该了解的内容。

1. web服务器
2. 反向代理
3. 正向代理
4. 负载均衡

下面我会进行详细说明。

## Web服务器

nginx可以被用作一个静态文件托管服务器。nginx做这个非常简单，其配置如下。

```
server {
    listen      80;
    server_name localhost;

    location / {
        root    ~/wwwroot;
        index   index.html;
    }
}
```

这时候，我们访问`http://localhost`，就会默认访问`~/wwwroot/index.html`文件。

当nginx被用作一个Web服务器时，其核心的配置location下的`root`和`index`配置子项，上面配置中的`location /`表示所有的请求都会按照请求路径去尝试访问`~/wwwroot`目录下的文件。

举个例子，我访问`http://localhost/my.html`时候，nginx会试图读取`~/wwwroot/my.html`文件，然后将文件内容返回给请求者。若nginx找不到这个文件，则会触发默认的404配置。

其实，实际情况中比上述的过程要复杂许多，比如各种复杂的location规则、rewrite规则、响应头设置、文件内存缓存、浏览器渲染等等，这些问题不在本文的讨论范围内。

## 代理

nginx可以简单的实现代理服务。通常代理又分为正向代理和反向代理。下面分别描述。

### 正向代理

所谓的正向代理，其实就是在客户端和服务器之间多架设了一个代理服务器proxy。当客户端需要访问服务器资源时，并不是直接从客户端发送一个请求到达服务器，而是从客户端发送一个请求到达proxy，然后再由proxy向真正想要访问的服务器资源发出请求。当proxy拿到服务器的返回时，再将此响应数据通过proxy返回给客户端。

这就是正向代理的流程。可以参考下图。

![](http://images0.gejiawen.com/posts/all-things-you-need-to-know-about-nginx/001.png)

大家常用的VPN代理就是正向代理的一个经典场景。

使用nginx就可以搭建一个简单的正向代理服务器，其配置代码如下：

```
resolver 114.114.114.114 8.8.8.8;

server {
    
    resolver_timeout 5s;
 
    listen 81;
 
    access_log  ~\wwwroot\proxy.access.log;
    error_log   ~\wwwroot\proxy.error.log;
 
    location / {
        proxy_pass http://$host$request_uri;
    }
}
```

其中`resolver`就是正向代理的DNS服务器地址，`listen`就是正向代理的监听端口。然后我们就可以将这台运行nginx的机器作为正向代理服务器了。

### 反向代理

什么是反向代理？下面是百度得到的解释，

反向代理（Reverse Proxy）方式是指以代理服务器来接受互联网上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给互联网上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。简单来说就是真实的服务器不能直接被外部网络访问，所以需要一台代理服务器，而代理服务器能被外部网络访问的同时又跟真实服务器在同一个网络环境。当然也可能是同一台服务器，端口不同而已。

相对正向代理，反向代理大家可能听的更多一些，使用的场景可能也更多一些。下图是反向代理的示意图，

![](http://images0.gejiawen.com/posts/all-things-you-need-to-know-about-nginx/002.png)

一般来说，反向代理有两个明显的作用，

- 保证内网的安全。比如可以使用反向代理提供WAF功能，阻止web攻击。比如现在所有的大型站点，对公网暴露的都是反向代理服务器，真实服务器可能不是直接对公网暴露的。
- 负载均衡。反向代理另一个经常被所用的场景就是通过反向代理服务器来优化网站的负载，即使所谓的负载均衡。

使用nginx可以简单搭建一个反向代理服务器，其配置如下，

```
server {
    listen       80;                                                         
    server_name  localhost;                                               
    client_max_body_size 1024M;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host:$server_port;
    }
}
```

上述的配置会将所有来自`http://localhost:80`的请求，都会转发到`http://localhost:8080`上。

### 两种代理的区别

大家可能会有一个疑问，为什么会叫正向、反向，这种看起来挺奇怪的一个名字？这个其实跟历史有一些关系。

首先，给大家先说明，这两种代理方式的区别。网上有一句非常精辟的话概括了正向、反向代理的本质。

“所谓正向代理，就是代理了客户端，对服务端而言，它响应的请求都是来自正向代理服务器。”
“所谓反向代理，就是代理了服务端，对客户端而言，都是反向代理服务器来响应它的请求的。”

下面有一张盗来的手绘图，将正向代理解释的非常形象，

![](http://images0.gejiawen.com/posts/all-things-you-need-to-know-about-nginx/003.jpg)

历史上，**正向代理**这种方式是先出现的，然后人们将后来出现的这种可用于负载均衡的代理方式就称之为**反向代理**了。

## 负载均衡

反向代理经常运用在负载均衡上。

所谓的负载均衡，指的是，服务端提供集群式服务，当客户端发出一个请求后，这个请求会最终落在一个**最闲**的服务器上进行处理。这里所谓的**最闲的服务器**，是一种话术，其实它是某种负载均衡策略选出来一台服务器来处理这个客户端请求。

常见的负载均衡示意图如下，

![](http://images0.gejiawen.com/posts/all-things-you-need-to-know-about-nginx/004.png)

如上图所示，我们服务端总共有三台服务器提供服务，客户端的请求通过反向代理服务器最终被转发到其中一台服务器上。

nginx可以快速搭建一个负载均衡服务，其配合如下，

```
upstream test {
    server 172.16.1.1:8080;
    server 172.16.1.2:8080;
}

server {
    listen       81;                                                         
    server_name  localhost;

    location / {
        proxy_pass http://test;
        proxy_set_header Host $host:$server_port;
    }
}
```

与普通的反向代理相比，负载均衡的nginx配置中，我们需要配置upstream，在upstream中申明服务端集群。

上面的nginx配置表示会将所有来自`http://localhost:81`的请求转发到一个名叫`test`的upstream中。这个test的upstream中有两个服务，分别是172.16.1.1:8080和172.16.1.2:8080。

当一个请求达到此负载均衡时，会由具体的负载均衡策略来决定将请求转发到哪个服务上。

常见的负载均衡策略有如下几种，

### RR（默认方式）

这是nginx的默认负载均衡策略，意为按照顺序进行负载。如果后端服务下线，能够自动剔除。

其典型配置如下，

```
upstream test {
    server 172.16.1.1:8080;
    server 172.16.1.2:8080;
}

server {
    listen       81;                                                         
    server_name  localhost;

    location / {
        proxy_pass http://test;
        proxy_set_header Host $host:$server_port;
    }
}
```

### weight

按照权重进行负载。即我们指定upstream中各个服务的命中比例。

其典型配置如下，

```
upstream test {
    server 172.12.1.1:8080 weight=9;
    server 172.12.1.2:8080 weight=1;
}
```

上述配置会导致10次请求，有9次请求到达172.12.1.1:8080，有一次到达172.12.1.2:8080。

这种负载策略非常适合后台机器配置有高低之别，配置高性能好的机器处理的请求多一些，反之，配置低性能差的机器处理请求就少一些。

### ip_hash

按照请求ip进行负载。

有的时候我们希望，来自同一个ip的请求总是落到同一个机器上进行处理。那么此时，我们就可以使用这种方式了。

其配置如下，

```
upstream test {
    ip_hash;
    server 172.12.1.1:8080;
    server 172.12.1.2:8080;
}
```

### fair

按后端服务器的响应时间来分配请求，响应时间短的优先分配。

```
upstream backend { 
    fair; 
    server localhost:8080;
    server localhost:8081;
} 
```

### url_hash

按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。

有的时候，我们希望同一个url请求能够落到同一个机器上进行处理。一般情况下，当后台是根据url计算的缓存，是非常有效的。

```
upstream backend { 
    hash $request_uri; 
    hash_method crc32; 
    server localhost:8080;
    server localhost:8081;
}
```

其中`hash_method`是url hash的方法。


以上几种负载均衡策略都有其合适的场景。除了本文介绍的这些内容之外，nginx还支持大量的第三方模块，可定制的功能非常多。



