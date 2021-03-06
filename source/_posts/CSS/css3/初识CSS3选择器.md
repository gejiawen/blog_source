postid: "first-to-css3-selectors"
title: "初识CSS3选择器"
date: 2015-04-09 01:26:15
categories: [CSS]
tags: [CSS拾遗系列, css3]

---

**注意**：由于CSS3中的部分内容尚未完全定稿，所以本文的部分可能会随时更新。

现在谈起CSS3的相关内容，其实CSS3已经不算是一个新东西了，毕竟出来有一段时间了。不过我却没有什么系统的经验，上次被人问到是否了解CSS3新增的选择器，各种尴尬，因为我压根就不知道哪些是CSS3新增的选择器，即使可能我之前有使用过。

为了补上这块知识的缺陷，恶补了一下CSS选择器的相关内容，并实验了每种选择器的效果。本文参考资源主要来自[W3C文档](http://www.w3.org/TR/css3-selectors/)

首先我们来看一张图，

![](//images0.gejiawen.com/posts/first-to-css3-selectors/001.png)

图中展示目前CSS Level1~Level3支持的所有的选择器。这张图是从W3C上抄下来的，绝对权威。

本篇文章将会说一说CSS Level3新增的选择器（如上图中飘黄的加重部分）。并附带一些浏览器兼容性说明。


# 属性选择器

新增的**属性选择器**如下图所示，

![](//images0.gejiawen.com/posts/first-to-css3-selectors/002.png)

这几个其实比较容易理解，使用起来应该没有什么障碍，这里就不多解释了。

其浏览器兼容性如下图，

![](//images0.gejiawen.com/posts/first-to-css3-selectors/003.png)

结论：除了IE6，基本上可以放心的使用了。

# 结构伪类选择器

新增的**结构伪类选择器**如下图所示，

![](//images0.gejiawen.com/posts/first-to-css3-selectors/004.png)

说实话，新增的这几个结构伪类选择器很容易弄混淆，特别是`*-child(n)`跟`*-of-type(n)`。

这里，我们就以`nth-child(n)`与`nth-of-type(n)`为例，通过一个[demo](http://runjs.cn/code/g1rt37di)来说明一下他们之间的区别。

通过demo，我们可以看出，

- `nth-child(n)`表示，选择满足以下条件的元素：第一是一个p元素；第二是父元素的第n个子元素
- `nth-of-type(n)`表示，选择父元素的第n个子元素p

如果还没弄明白，这里有一篇[文章](https://css-tricks.com/the-difference-between-nth-child-and-nth-of-type/)可供参考。

其浏览器兼容性如下图，

![](//images0.gejiawen.com/posts/first-to-css3-selectors/005.png)

# UI元素状态伪类选择器

新增的**UI元素状态伪类选择器**如下图所示，

![](//images0.gejiawen.com/posts/first-to-css3-selectors/006.png)

其浏览器兼容性如下图，

![](//images0.gejiawen.com/posts/first-to-css3-selectors/007.png)

# 目标伪类选择器

新增的**目标伪类选择器**如下图所示，

![](//images0.gejiawen.com/posts/first-to-css3-selectors/008.png)

这个新增的选择器有点不太好理解，它是啥意思呢？例如页面上有个`id=test`的div，然后在URI中有#test的话，就会给id为test的div加上定义的样式。

其浏览器兼容性如下，

![](//images0.gejiawen.com/posts/first-to-css3-selectors/009.png)

# 否定伪类选择器

新增的**否定伪类选择器**如下图所示，

![](//images0.gejiawen.com/posts/first-to-css3-selectors/010.png)

个人觉得这个选择器可能会使用的比较频繁，它的作用是在匹配的一些列元素剔除选择符为`s`的元素。相当于做了个筛选。

其浏览器兼容性如下，

![](//images0.gejiawen.com/posts/first-to-css3-selectors/011.png)


# 通用兄弟元素选择器

新增的**通用兄弟元素选择器**如下图所示，

![](//images0.gejiawen.com/posts/first-to-css3-selectors/012.png)

这个选择器也应该蛮有用的，稍微注意一下与`E + F`这种通用兄弟选择器的区别就好了。

其浏览器兼容性如下，

![](//images0.gejiawen.com/posts/first-to-css3-selectors/013.png)




