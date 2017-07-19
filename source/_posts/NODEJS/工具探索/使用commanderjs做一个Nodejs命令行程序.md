postid: 'make-a-node-cli-program-by-commander-js'
title: 使用commander.js做一个Nodejs命令行程序
categories: [NODEJS]
tags: [nodejs, commanderjs, 漫游NodeJS系列]
date: 2016-09-21 15:45:01

---

# 前言

在当下，作为一名前端码农，不知道Nodejs是不可原谅的。可以说，除了一些特别要求的业务范畴，常见的后端业务Nodejs都能handle住。

Nodejs的另一个常用场景是，造出一些实用工具，而这些工具大部分都是一些命令行程序。今天我们就来介绍如何写出一个Nodejs的命令行应用程序。

# 土著做法

当一个Nodejs程序运行时，会有许多存在内存中的全局变量，其中有一个叫做`process`，意为进程对象。`process`对象中有一个叫做`argv`的属性。命令行程序的第一个重头戏就是解析这个`process.argv`属性。

我们先随便写一个node程序，把`process.argv`打印出来看看，

```bash
$ node test1.js --name gk
[
    '/usr/local/Cellar/node/6.6.0/bin/node',
    '/Users/gejiawen/code/20160921/test1.js',
    '--name',
    'gk'
]
```

看起来`process.argv`好像是一个数组，其中第一个元素是node的执行路径，第二个元素是当前执行文件的路径，从第三个元素开始，是执行时带入的参数。

所以，规律很简单。我们在写命令行程序时，只需要对`process.argv`这个数组的第三个元素及其之后的参数进行解析即可。

如果不嫌麻烦，完全可以写出很多判断分支来做。但是现在我们有更好的方法。

# 使用commander.js

[commander.js](https://github.com/tj/commander.js)是[TJ](https://github.com/tj)所写的一个工具包，其作用是让node命令行程序的制作更加简单。

## 安装及使用

安装很简单，

```bash
$ npm install commander
```

注意包名是**commander**而不是**commander.js**。

然后我们在新建一个js文件，叫做index.js，内容如下

```js
var program = require('commander')

program
    .version('0.0.1')
    .description('a test cli program')
    .option('-n, --name <name>', 'your name', 'GK')
    .option('-a, --age <age>', 'your age', '22')
    .option('-e, --enjoy [enjoy]')

program.parse(process.argv)
```

此时，一个简单的命令行程序就完成了。我们通过如下的命令来执行它，

```bash
$ node index.js -h
```

结果如下，

```bash
$ ./test -h

  Usage: test [options]

  a test cli program

  Options:

    -h, --help           output usage information
    -V, --version        output the version number
    -n, --name <name>    your name
    -a, --age <age>      your age
    -e, --enjoy [enjoy]
```

commander.js第一个优势就是提供了简介的api对可选项、参数进行解析。第二个优势就是自动生成帮助的文本信息。

## 常用api

commander.js中命令行有两种可变性，一个叫做`option`，意为选项。一个叫做`command`，意为命令。

看两个例子，

```js
program
   .version('0.0.1')
   .option('-C, --chdir <path>', 'change the working directory')
   .option('-c, --config <path>', 'set config path. defaults to ./deploy.conf')
   .option('-T, --no-tests', 'ignore test hook')

 program
   .command('setup')
   .description('run remote setup commands')
   .action(function() {
     console.log('setup');
   });

 program
   .command('exec <cmd>')
   .description('run the given remote command')
   .action(function(cmd) {
     console.log('exec "%s"', cmd);
   });

 program
   .command('teardown <dir> [otherDirs...]')
   .description('run teardown commands')
   .action(function(dir, otherDirs) {
     console.log('dir "%s"', dir);
     if (otherDirs) {
       otherDirs.forEach(function (oDir) {
         console.log('dir "%s"', oDir);
       });
     }
   });

 program
   .command('*')
   .description('deploy the given env')
   .action(function(env) {
     console.log('deploying "%s"', env);
   });

 program.parse(process.argv);
```

- 通过`option`设置的选项可以通过`program.chdir`或者`program.noTests`来访问。
- 通过`command`设置的命令通常在`action`回调中处理逻辑。



### `version`

用法： `.version('x.y.z')`

用于设置命令程序的版本号，

### `option`

用户：`.option('-n, --name <name>', 'your name', 'GK')`

- 第一个参数是选项定义，分为短定义和长定义。用`|`，`,`，` `连接。
    - 参数可以用`<>`或者`[]`修饰，前者意为必须参数，后者意为可选参数。
- 第二个参数为选项描述
- 第三个参数为选项参数默认值，可选。

### `command`

用法：`.command('init <path>', 'description')`

- `command`的用法稍微复杂，原则上他可以接受三个参数，第一个为命令定义，第二个命令描述，第三个为命令辅助修饰对象。
- 第一个参数中可以使用`<>`或者`[]`修饰命令参数
- 第二个参数可选。
    - 当没有第二个参数时，commander.js将返回`Command`对象，若有第二个参数，将返回原型对象。
    - 当带有第二个参数，并且没有显示调用`action(fn)`时，则将会使用子命令模式。
    - 所谓子命令模式即，`./pm`，`./pm-install`，`./pm-search`等。这些子命令跟主命令在不同的文件中。
- 第三个参数一般不用，它可以设置是否显示的使用子命令模式。

### `description`

用法：`.description('command description')`

用于设置命令的描述

### `action`

用法：`.action(fn)`

用于设置命令执行的相关回调。`fn`可以接受命令的参数为函数形参，顺序与`command()`中定义的顺序一致。

### `parse`

用法：`program.parse(process.argv)`

此api一般是最后调用，用于解析`process.argv`。

### `outputHelp`

用法：`program.outputHelp()`

一般用于未录入参数时自动打印帮助信息。

eg:

```js
if (!process.argv.slice(2).length) {
    program.outputHelp(make_red);
}

function make_red(txt) {
    return colors.red(txt); //display the help text in red on the console
}
```


# 实践

下面我们来做一个小工具，实践一下commander.js的强大之处。

这个工具叫做[npmrc-local](https://github.com/gejiawen/npmrc-local)，作用是在执行目录下生成一个`.npmrc`文件，用使用默认的registry、disturl、loglevel配置（指向的是npm.taobao.org）。


首先我们得创建一个项目，

```bash
$ mkdir npmrc-local
$ git init
$ npm init
$ touch .gitignore
$ touch bin/npmrc.js
$ touch lib/index.js
```


修改`package.json`文件，添加`bin`字段，

```json
{
    "bin": {
        "npmrc": "./bin/npm.js"
    }
}
```

修改`bin/npmrc.js`,

```js
#!/usr/bin/env node

require('../lib/index')
```

修改`lib/index.js`,

```js
#!/usr/bin/env node

var fs = require('fs')
var path = require('path')
var readline = require('readline')
var program = require('commander')
var rc = require('../rc')

var exit_bak = process.exit

program
    .version('0.0.1')
    .allowUnknownOption()
    .option('-r, --registry <registry>', 'use custom registry', rc.registry)
    .option('-d, --dist-url <disturl>', 'use custom dist url', rc.disturl)
    .option('-l, --log-level <loglevel>', 'use custom log level', rc.loglevel)

program.parse(process.argv)

program.registry && (rc.registry = program.registry)
program.distUrl && (rc.disturl = program.distUrl)
program.logLevel && (rc.loglevel = program.logLevel)

if (!_exit.exited) {
    _main()
}

// Graceful exit for async STDIO
function _exit(code) {
    var draining = 0
    var streams = [process.stdout, process.stderr]

    function done() {
        if (!(draining--)) {
            exit_bak(code)
        }
    }

    _exit.exited = true

    streams.forEach(function (stream) {
        draining += 1
        stream.write('', done)
    })

    done()
}

function _confirm(msg, cb) {
    var rl = readline.createInterface({
        input: process.stdin,
        output: process.stdout
    })

    rl.question(msg, function (input) {
        rl.close()
        cb(/^y|yes|ok|true$/i.test(input))
    })
}

function _write(path, content, mode) {
    fs.writeFileSync(path, content, {
        mode: mode || 0o666
    })

    console.log('success!!!')
}

function _generateFile(filePath) {
    var content = 'registry={registry}\ndisturl={disturl}\nloglevel={loglevel}\n'

    content = content.replace(/\{(\w+)\}/gi, function (a, b) {
        return rc[b]
    })
    _write(filePath, content)
}

function _overwrite(filePath) {
    _generateFile(filePath)
}


function _existNpmRC(filePath) {
    fs.exists(filePath, function (exists) {
        if (exists) {
            _confirm('ATTENTION: .npmrc is exist, over write? [y/N] ', function (ans) {
                ans ? _overwrite(filePath) : console.log('bye!')
            })
        } else {
            _generateFile(filePath)
        }
    })
}

function _main() {
    var filePath = path.resolve(process.cwd(), '.npmrc')
    console.log('writing path: ' + filePath)
    _existNpmRC(filePath)
}
```

所有的代码工作完毕之后，修改`package.json`中version字段，然后执行`npm adduser`&`npm publish`，将这个工具包推送npmjs.org供所有人使用了。

因为我已经很长时间没更新博客，没有往npm仓库上推送package了，这次又踩了一次之前遇到的坑，泪崩。详见[npm adduser的坑](http://blog.gejiawen.com/2015/08/05/npm-adduser-refuse/)。


如果对如何上传自己的工具包到npm还不是很清楚，可以参考[这篇文章](http://www.cnblogs.com/pingfan1990/p/4824658.html)。

[这篇文章](https://aotu.io/notes/2015/12/23/building-command-line-tools-with-node-js/)介绍除了介绍commander.js之外，还介绍了[chalk](https://www.npmjs.com/package/chalk)和[process](https://www.npmjs.com/package/progress)等在创建命令行工具时的常用工具，值得一读。









