postid: 'working-with-scoped-packages'
title: 使用scoped npm package
categories: [踩坑录]
tags: [npm]
date: 2017-08-10 12:05:25

---

前两天在使用[verdaccio](https://github.com/verdaccio/verdaccio)搭建私有npm仓库时，遇到了一个问题，刷新了我对npm私有包的认识。原先我一直认为npmjs.com上的那些形如`@scope/project-name`都是私有的npm包，是要付费托管的。然而事实情况并不是这样。

## 发布scoped package

从npm@2.7.0开始，支持scoped package。一个scoped package的组成方式是：**`@scope/project`**。

如果我们想要publish一个scoped package，第一步需要在项目的`package.json`的`name`属性中添加scope相关的申明，比如，

```json
{
    "name": "@gejiawen/scoped-package"
}
```

我们可以在`npm init`时带有`--scope`参数，也能初始化一个带有scope的`package.json`。下面是一个示例，

```bash
npm init --scope=gejiawen
```

第二步，我们需要登录到[npmjs.com](https://www.npmjs.com/)，

```bash
npm login
```

在执行登录时，确定你的`~/.npmrc`中没有添加任何的私有npm registry配置，否则将会登录失败。

或者在登录时带有官方的registry参数也是可以的，

```bash
npm login --registry=https://registry.npmjs.org/
```

第三步，我们执行`npm publish`。不出意外，这步会出问题。npm会提示你，

```bash
npm ERR! You need a paid account to perform this action. For more info, visit: https://www.npmjs.com/private-modules : @gejiawen/scoped-package
```

查阅相关文档后，可以知道，npm规定scoped package默认是私有包，而在npm上托管私有包是需要付费的，费用是$7/月。这里所谓的私有包的含义是允许包所有者控制包的访问权限。我们可以在publish时添加`--access public`参数告知npm这不是一个私有包。

```bash
npm publish --access=public
```

这样我们就可以成功在npm上部署一个scoped package了。

所以，我们可以看出，**scoped package不一定就是私有包，但是私有包一定是一个scoped package**。一个普通的npm用户也是在npm上部署scoped package的。

在npm上除了可以使用自己的username作为scope之外，还可以创建organization scope。在以前npm不支持scope packge的时候，大家可能会遇到一个非常无奈的情况，就是在你绞尽脑汁给项目想出了一个犀利的名字，但是在publish的时候，发现这个名字已经被别人占用了。此时你只能忍痛给你的项目换一个名字了。

但是有了scoped package，这个问题就迎刃而解了，再也不用担心好名字都被别人抢占了。

## 使用scoped package

当我们需要使用一个scoped package时，我们需要在`package.json`中写全其scope和project name，比如

```json
{
    "dependencies": {
        "@gejiawen/scoped-package": "^0.0.1"
    }
}
```

在引用时，也需要写全包的路径，包括其scope和project name，比如

```js
let pkg = require('@gejiawen/scoped-package')
```


## 题外话

其实所谓的scoped package就是在原来的包管理上新增了命名空间的概念。原先npm官方就npm是否需要支持个性化的namespace有过[讨论](https://github.com/npm/npm/issues/5239)，这个讨论发生在2014年，由当时的npm的掌门人[isaacs](https://github.com/isaacs)发起。可能因为NodeJS社区的一种“仁慈的独裁”，经过讨论最终并未支持scoped package。isaacs当时给出的原因大概意思是，npm上的package面向社区的，希望能够集社区的力量让一个package不断更新，而不是同一个功能有着非常多的不同的包实现。

现在看来，isaacs这个理由未能站得住脚，因为现在npm上充斥着大量的**僵尸**包。其实我个人也是不太认同这种观点的。事实也证明命名空间还是需要的。不管怎么说，不用担心好的项目名被抢占了。

另一点，在做企业开发时往往需要一个企业级的私有npm仓库。根据我个人这段时间的实践，给publish在私有仓库上的package都冠以统一的scope（比如公司名简称），绝对是一个很棒的想法。


## 参考

- [npm-scope](https://docs.npmjs.com/misc/scope#publishing-public-scoped-packages-to-the-public-npm-registry)
- [Working with scoped packages](https://docs.npmjs.com/getting-started/scoped-packages)

