postid: "nodejs-series"
title: 漫游NodeJS系列
date: 2015-03-17 17:16:40
categories: [系列]
tags: [目录]

---

# 为什么会有这个系列

[漫游NodeJS系列](http://blog.gejiawen.com/2015/03/17/系列/漫游NodeJS系列/)文章，将会记录作者一路学习[NodeJS](http://nodejs.org/)（[iojs](https://iojs.org/en/index.html)）的点点滴滴。

时至今日，NodeJS已经不再像是两年前那么新的技术要素了，相对来说成熟了不少，不管是各种框架类库的生态系统还是被各界的认可情况。就本人目前的经验来说，NodeJS现阶段在好几个领域仍然有探索和发展的潜力。

第一是Web开发方面。NodeJS依托JavaScript语言，而web开发中js绝对是不可或缺的一部分。再者现在各种基于nodejs的web开发框架层出不穷，比较主流的有[express](https://www.npmjs.com/package/express)，[koa](https://www.npmjs.com/package/koa)等等。随着web端技术或者说浏览器标准的推进，基于nodejs的web开发框架将会越来越先进。

第二是基于NodeJS的各种工具（包括一些纯命令行的CLI）。工具这个范畴就大了，各种各样功能的工具，数不胜数。如果要强行对各种工具类的package分类的话，那么我觉得一种是非常底层工具package，比如[npm](https://www.npmjs.com/packages/npm)，[async](https://www.npmjs.com/packages/async)，[commander](https://www.npmjs.com/packages/commander)之类的，因为它们做的事情比较“底层”，而且package本身的质量也非常高，然后逐渐被认可，被越来越多的其他package依赖。另一种就依赖各种“底层”package，搭建出一套用于解决特定问题的解决方案。比如[gulp](https://www.npmjs.com/packages/gulp)，[Karma](https://www.npmjs.com/packages/karma)等等。显然，垂直于解决某一特定问题的package仍然会非常多，这里就不展开说了。

第三的话，我感觉应该是一些研究之类的方向。比如[NW.js](http://nwjs.io/)（就是以前的node-webkit），基于nodejs提出的各种web前后端分离的思想以及一些跨语言方向的研究，比如[Nodejs与R跨平台通信](http://blog.fens.me/r-rserve-nodejs/)、[tty.js打通浏览器与Linux的通道](http://blog.fens.me/nodejs-linux-sh-tty/)等等。

在NodeJS的官方包管理器[NPM](https://www.npmjs.com/)上，我们可以看到基于nodejs的各种package非常的多，已经达到了13W+。当然这其中会包含一些质量不太高的package，但是不管怎么说，整个NodeJS社区都是一片欣欣向荣的景象。

值得一提的是，前一段时间NodeJS社区更是发生了一件大事。简要来说，就是NodeJS的几位核心开发者不满NodeJS维护公司Joyent对NodeJS开发的缓慢进度，从[joyent/node](https://github.com/joyent/node)fork了一份源代码并重命名为[io.js](https://github.com/iojs/io.js)，其目的就是为了加速推动node的开发。更多详情可参阅[iojs.org](https://iojs.org/en/index.html)。

# 内容简要

本系列的文章，将会按照内容归结为几大类。（TODO）

# 目录

## 基础入门

[（1）NodeJS模块全面指南](http://blog.gejiawen.com/2015/03/17/nodejs-module-guide/)
[（2）如何导出NodeJS模块](http://blog.gejiawen.com/2014/10/16/how-to-export-nodejs-module/)
[（3）NodeJS最佳实践](http://blog.gejiawen.com/2014/09/19/nodejs-best-practices/)

## 进阶研究

TODO

## 工具探索

[（1）gulp vs. grunt](http://blog.gejiawen.com/2015/01/09/gulp-vs-grunt/)
[（2）通读SuperAgent文档](http://blog.gejiawen.com/2015/08/14/read-superagent-doc/)
[（3）通读Cheerio文档](http://blog.gejiawen.com/2015/08/17/read-cheerio-doc/)
[（4）gulp备忘录（一）：API指南](http://blog.gejiawen.com/2015/12/05/gulp-reminder-1-api/)
[（5）gulp备忘录（二）：常用插件](http://blog.gejiawen.com/2015/12/05/gulp-reminder-2-plguins)
[（6）gulp备忘录（三）：使用技巧](http://blog.gejiawen.com/2015/12/05/gulp-reminder-3-recipes)
[（7）gulp备忘录（四）：编写插件](http://blog.gejiawen.com/2015/12/05/gulp-reminder-4-write-plugin)
[（8）使用commander.js做一个Nodejs命令行程序](http://blog.gejiawen.com/2016/09/21/make-a-node-cli-program-by-commander-js)


## Web开发

[（1）Express4入门指南](http://blog.gejiawen.com/2015/08/13/first-to-express4-guide/)

# 总结

TODO



