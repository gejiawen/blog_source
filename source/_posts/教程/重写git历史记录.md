postid: 'rewrite-git-log-history'
title: 重写git历史记录
categories: [教程]
tags: [git]
date: 2017-08-31 19:39:22
---

大家在使用git时做版本管理时，有时候会遇到下面的场景，

- 不小心将一个很大的文件提交到仓库中了，导致仓库臃肿，上传下载都非常消耗网络，追悔莫及！
- 随着项目的推进，突然不想用git来管理某个文件了，将其放入`.gitignore`文件中，发现git依然能够探测到这个文件的改动，一脸懵逼！


本文将会就以上两个问题给你指条明路。

`git filter-branch`命令就是我们的主角。[git filter-branch](https://git-scm.com/docs/git-filter-branch)命令在git中号称大杀器命令，基本上可以用这个命令能够到达任何操作要求，注意，是**任何**！

关于这个命令的文档我已经在上面的链接中给出了，一句话概括这个命令的作用：**操作所有的git对象数据以重写历史记录**。有兴趣掌握其原理的，可自行找虐。下面，我们将会用一个实际的案例来说明“如何重写git历史记录”。

## 案例

在前端开发中，`npm`是必不可少的包管理工具。在`npm@5.0.0`之后，npm引入了`package-lock.json`文件。关于这个`package-lock.json`是干什么用的，这个官方的[说明文档](https://docs.npmjs.com/files/package-lock.json)。

简单来说，这个`package-lock.json`文件是npm用来“锁版本”的。这里所谓的锁版本是指当在项目拥有package-lock.json文件时，npm会根据其自动解析出包依赖，不会出现在不同的环境场景下，安装了不同的包版本。

在开始使用npm@5.0.0的时候，执行完`npm install`后，有一个提示，

> npm notice created a lockfile as package-lock.json. You should commit this file.

然后我就按照这句提示来做了。后来发现这玩意坑很多。所以我们现在想将这个`package-lock.json`文件放到`.gitignore`中，希望git仓库不再追踪这个文件。

## 方案

想做的事情已经很明确了，接下来我们来动手了。

首先明确一点，只要一个文件被git管理中（即被添加到git仓库中），那么无论是删除这个文件，还是将其添加到.gitignore文件中，都是没办法组织git继续对齐进行管理和追踪的。

所以，我们改写git仓库的历史记录。

**step 1**，

```bash
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch package-lock.json' --prune-empty --tag-name-filter cat -- --all
```

我们使用了`git filter-branch`命令，对所有的分支上的commit执行额外的命令操作。这个操作就是`git rm --cached --ignore-unmatch package-lock.json`，忽略对`package-lock.json`文件的追踪，从git仓库中将其移除，并同时重写每一条记录。

**step 2**，

```bash
rm -rf .git/refs/original
git reflog expire --expire=now --all
git gc --prune=now
git gc --aggressive --prune=now
```

现在历史记录中已经不包含对那个文件的引用了。不过`reflog`以及运行`filter-branch`时，git往`.git/refs/original`添加的一些`refs`中仍有对它的引用，因此需要将这些引用删除并对仓库进行repack操作。在进行repack前需要将所有对这些commits的引用去除。

**Step 3**，

```bash
git push origin master --force
```

将本地重写的git仓库强制推送到远程origin。

## 额外的问题

如果之前签出了远程分支，比如在很早的时候，通过`git checkout --track origin/a`签出了分支`a`到本地。那么在重写历史之后，可能本地`a`分支会丢失对远程分支`remote/a`的追踪。造成这种现象的原因是在重写记录时，将`origin/a`分支上的记录重写了，导致本地`a`分支与远程`remote/a`分支不再匹配。

此时，我们应该删除本地游离的分支`a`，然后从`remote/a`上重新签出分支`a`到本地。


## 参考链接

- [Removing sensitive data from a repository](https://help.github.com/articles/removing-sensitive-data-from-a-repository/)
- [git-filter-branch - Rewrite branches](https://git-scm.com/docs/git-filter-branch)



