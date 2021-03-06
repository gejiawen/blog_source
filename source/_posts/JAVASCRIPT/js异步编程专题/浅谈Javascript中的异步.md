postid: 'think-about-async-in-javascript'
title: 浅谈Javascript中的异步
categories: [JAVASCRIPT]
tags: [js, Javascript异步编程专题]
date: 2015-10-12 15:12:51

---

最近心血来潮，准备整理一下Javascript中有关异步编程方面的东西，写几篇文章。当然同时也是为了沉淀一下自己所学。

暂时将这个专题命名为**Javascript异步编程专题**吧，目前包含以下几篇文章，

- [浅谈Javascript中的异步](http://blog.gejiawen.com/2015/10/12/think-about-async-in-javascript/)
- [Javascript中常见的异步编程模型](http://blog.gejiawen.com/2015/10/12/some-javascript-async-pattern/)
- Generator函数处理异步调用

------

本篇文章是本专题的第一篇文章。

正文开始。

Javascript这门语言出身“低贱”，但是随着近年来社区的活跃，反而给人以耀眼的活跃。同时带来的也是各种各样的对这门原先被当作玩具语言的解析。本篇文章将会带了Javascript中关于异步编程以及其周边内容的介绍。

# Javascript的单线程

*Javascript是单线程的*。javascript的程序员一定要时刻谨记这句话。

可以说单线程是Javascript语言最本质的特性之一。言外之意，Javascript引擎在运行js代码的时候，同一个时间只能执行单个任务。

这是为何呢？

其实这跟Javascript的历史有关系。Javascript最初被设计成一种浏览器脚本语言，决策者们仅仅是需要一门轻量的浏览器脚本语言来做一些简单的用户交互及DOM操作。在这样一个背景下，据说Javascript的设计者仅仅花了10天就完成了这门语言的设计。

相比多线程，单线程不需要考虑创建线程、销毁线程、线程间通信等等诸如此类的复杂问题。所以，为了避免复杂性，Javascript从诞生伊始就是单线程，而且日后也应该不会改变这一特性。


# Javascript中的异步原理

在C#、Java等语言中，异步操作往往是创建一个线程来执行异步任务，异步任务执行完毕之后再将执行结果通知给主线程。这其中往往会伴随着，多线程、线程池、并发等相关术语。但是在Javascript中不存在这些。

如前文所述，Javascript是单线程的，那么Javascript的异步是怎么回事呢？Javascript又是如何在单线程上给出所谓的异步编程呢？

## 两组概念

在解释这个问题之前，我们应该首先明确两组概念。就是**同步**和**异步**以及**阻塞**和**非阻塞**这两组概念。

首先这是两组不同的概念。同步、异步一般指的是消息的通信机制；而阻塞、非阻塞一般强调的是程序在等待（消息）时的状态。

他们的概念如下，

| 名称 | 解释 | 备注 |
| :--- | :--- | :--- |
| 同步 | 发出一个功能调用（执行函数）时，在没有得到调用结果之前，该调用就不会返回 | - |
| 异步 | 发出一个调用后，不关心结果返回，而是立即返回。结果出来后，将会通过一些额外手段通知调用者 | *注1* |
| 阻塞 | 在调用结果返回之前，当前线程会被挂起等待。直到得到结果之后函数调用才会返回。 | - |
| 非阻塞 | 在不能立刻得到结果之前，该函数不会阻塞当前线程，而会立刻返回。 | - |

*注1*：这里所说的额外手段，往往是指异步调用的一些具体实现方式，比如事件监听、轮询等等。

可见，同步、异步和阻塞、非阻塞是两组概念，我们不应该混淆他们。同时，这两组概念其实是可以共存，可以混合搭配最多四种模式。

后面我们会说到其实Javascript中的异步并非真正意思上的异步，它最多能做到非阻塞的地步。所以Node.js官网在介绍自己时仅仅是说“基于事件驱动”，“非阻塞IO模型”，并没有提到异步的字眼。

好了，关于这两组概念虽然还有许多可以说的内容，不过这篇文章的重点并不是介绍这两组概念，有兴趣的可自行查阅相关资料。

## Javascript中的异步

让我们言归正传。在Javascript中，什么样的调用可以算是异步调用呢？

我个人认为，Javascript中异步的应用场景主要有两个，其一是基于浏览器的各种异步IO和事件监听，其二是Javascript代码中的关于定时器（`setTimeout`、`setInterval`）的应用。

对于异步IO，比如ajax请求（当然，ajax分为同步ajax和异步ajax，这里我们仅用异步ajax来作示例）。当js代码发出一个ajax请求时，浏览器仍然会接着往下执行js代码。我们将处理ajax返回的逻辑绑定在ajax回调函数上，当ajax请求完毕成功返回时会触发回调函数，然后接着执行我们绑定的处理逻辑。如下图，

![](//images0.gejiawen.com/posts/think-about-async-in-javascript/001.png)

这种场景下的异步其实涉及到了两个线程，一个是javascript线程，一个是浏览器为了发出ajax请求而开辟的线程（它属于浏览器进程的子线程）。

对于定时器的异步场景，我们可以用一段示例代码来说明，

```javascript
function foo(callback) {
    console.log('foo start');
    setTimeout(function() {
        callback();
    }, 2000);
    console.log('foo end');
}

function foo2() {
    console.log('Hello!');
}

foo(foo2);
```

这段代码的执行过程如下图，

![](//images0.gejiawen.com/posts/think-about-async-in-javascript/002.png)

从图中我们可以看出执行`foo(foo2)`使用了50ms，其中在1010ms时开始执行`setTimeout`语句。由于我们使用`setTimeout`设置了一个延时，所以在执行`foo(foo2)`时并没有真正的执行`foo2()`，而是将其时机推迟了。直到3010ms左右时才去真正的执行`foo2()`的代码。

（**注**，为了方便叙述，上述时间都是假设的，不必纠结。下同。）

从上图中可以看出，整个执行过程中只有一个javascript线程，异步调用的部分的代码，最终也是在javascript线程上执行的，只不过被**延迟**了。

从上面的两个示例中，我们足以管中窥豹。笼统的说，**单线程的Javascript中其实没办法实现真正意义上的异步，Javascript中所谓的异步可以理解成延迟执行**。

# 事件和任务队列

往往在Javascript运行环境中，除了Javascript线程之外，还有会有一个专门维护任务队列的线程。说到这里，有人会问，不是说好的Javascript是单线程吗？怎么这里会有两个线程？

好问题！注意这里我并没有说Javascript语言，而是**Javascript运行环境**（Javascript Runtime）。常见的运行环境有各种浏览器（浏览器中的Javascript解释引擎），Nodejs环境等等。

这里我们就以浏览器环境作为示例来进行相关说明。首先试想一下我们现在有这样的一个场景。我们在一段js代码中发出了一个ajax请求，同时给click和press注册了事件监听。那么当我们执行这段代码后，其效果图如下，

![](//images0.gejiawen.com/posts/think-about-async-in-javascript/003.png)

因为Javascript是单线程的，就意味着Javascript同一时间只能处理一个任务。如果一旦任务产生的速度过快，那么后续的任务就得排队。每一个任务往往与一个事件相对应。

如上图所示，当浏览器的用于ajax的线程得到ajax返回结果后会产生一个事件，将其放入任务队列。（其实这里还涉及到Ajax线程如何通知任务队列等等问题，这里我们就不讨论了）

除此之外，可能用户还在页面进行了鼠标点击操作和按下键盘输入。这两个动作都将会产生一个事件，将产生的事件压入任务队列。

## EventLoop的解释

上图中还有一个叫做**Event Loop**的东西，这个东西是什么东西呢？

> Event Loop是一个程序结构，用于等待和发送消息和事件。（a programming construct that waits for and dispatches events or messages in a program.）

简单来说，EventLoop也是一个程序，它会不停的轮询任务队列中的事件。如果Javascript线程空了，则取出任务队列的第一个事件，然后找到此事件的处理函数并执行处理函数。当前一个事件执行完了（此时Javascript又会空下来），那么继续取出任务队列中的下一个事件。如此往复。

回到我们上面的例子中，当从任务队伍队列中取出第一个事件时（Ajax返回事件），Javascript线程将会去执行函数`foo1`。其实从图中可以看出，先前绑定的三个回调函数最终都是在Javascript线程上执行的，只不过由于任务队列的存在，导致了他们的执行时机是不一样的。


# Javascript中的定时器

Javascript中内置了两个方法，用于设置定时任务。分别是`setTimeout`和`setInterval`。它们的具体用法我这里不展开说了。

前文有提到过，在Javascript中常见的有两种异步场景，一种是异步IO，另一种就是定时任务。

JQuery的作者John Resig有一篇[文章](http://ejohn.org/blog/how-javascript-timers-work/)，就是探讨Javascript中的定时器是如何工作的。我觉得大神文章中的示例对`setTimeout`和`setInterval`的运行原理，以及它们在Javascript中是如何与异步任务协调工作的，说明的都非常好。我这里就直接拿过来用了。


首先我们来看一张场景图，

![](//images0.gejiawen.com/posts/think-about-async-in-javascript/004.png)

乍一看这张图一坨方块加一坨英文，看不出头绪。其实Javascript定时器的秘密就隐藏在其中，让我们一步一步来解析。

图中深蓝色的方块可以理解成Javascript线程的执行过程，左侧是时序。右侧是对每一个Javascript代码发生了什么事作的说明。

先看第一个Javascript代码块，它在整个时序中占用了不到20ms（大概18ms的样子），它依次发生哪些事情呢？

- 初始化了一个10ms的`setTimeout`定时器
- 鼠标被点击了一次
- 初始化了一个10ms的`setInterval`定时器
- 触发10ms的**Timer**定时器

我们发现

- 发生了一次鼠标点击，将产生一个鼠标click事件，此事件将会压入任务队列，等待执行。
- **Timer**计时器的回调实际上会在第一个代码块执行完毕前被触发。但是这里注意的是，它不会立即执行（单线程上不能这样做，因为当前的Javascript线程还在处理第一个代码块呢）。实际上，触发的回调将被排成一个队列，等待下一个可执行时间。


直到Javascript执行完第一块js代码，我们的任务队列中已经有两个待执行的任务了。如下，

![](//images0.gejiawen.com/posts/think-about-async-in-javascript/005.png)

现在Javascript线程已经处于空闲状态了，此时任务队列已经有2个任务在等待执行了，按照入队顺序，我们先处理click事件。队列后面的任务继续排队等待。


回到场景图中，我们现在开始处理click事件，即执行click事件绑定的回调函数。这其中触发了一次10ms的**Interval**定时器。但是此时Javascript线程上正在执行click事件的回调，所以即使**Interval**触发了，但是并不能执行。怎么办呢？只能对它说不好意思了，让它去排队吧。

此时，任务队列的情况如下，

![](//images0.gejiawen.com/posts/think-about-async-in-javascript/006.png)

可以看到，之前的click事件已经出队了，但是又新增了一个**Interval**定时器回调。

随着时间的推移，在完成click事件任务完成之后，继续从任务队列中取出任务。从上图可知，下一个任务是**Timer**定时器的回调。

我们在执行**Timer**定时器任务时，发现又有一个**Interval**定时器触发了。不过此时Javascript线程还在执行**Timer**定时器的任务呢，所以刚触发的**Interval**定时器任务只能去排队了。

此时，任务队列如下，

![](//images0.gejiawen.com/posts/think-about-async-in-javascript/007.png)

现在任务队列里面已经有两个一样的**Interval**定时器任务在等待执行了。

------

**注**：John Resig的原文中说，在**Timer**定时器执行期间触发的**Interval**定时器会被*Dropped*。起先我百思不得其解。按道理应该将其入队才对啊。后又仔细研读了原文，外加查阅了相关资料，终于知道是为何了。

在John Resig的原文中，他给出的两个定时器的初始化如下，

```javascript
var id = setTimeout(fn, delay);
var id = setInterval(fn, delay);
```

关键的地方来了，这里`setTimeout`和`setInterval`使用的回调函数都是同一个函数`fn`。在这种先提条件下，在前面的分析中，当Javascript正在执行**Timer**定时器的回调时，虽然又一次触发了**Interval**回调，但是由于两个定时器采用回调都是同一个函数。所以导致此时被触发的**Interval**定时器由于绑定的回调函数正在被执行，所以此次触发就被抛弃了。

------

让我们继续回到分析之中来。

当**Timer**定时器的回调执行完毕之后，我们与之前一样，从任务队列中取出下一个任务。这次是**Interval**定时器的回调。

因为John Resig给的示例中，**Interval**定时器的回调执行将在10ms之内完成，所以按照上述的分析如此往复之后，最后将会出现Javascript线程空等的情况。

在40-50ms这段时间之内，我们将积累的两个**Interval**定时器任务执行完毕了，发现任务队列中没有等待执行的任务了。那么接下来Javascript线程就会处于“空等”状态。

在50ms的时候，有一个**Interval**定时器触发了。我们同样将其入队。不过由于之前任务队列本来就是空的，所以这个触发就立马执行了。

## 定时器的时间精度问题

根据前文所述，其实不管是`setTimeout`还是`setInterval`定时器，因为Javascript的单线程缘故，他们当在任务量较多的时候，都会涉及到一个排队等待执行的问题。所以定时器的对于时间间隔的把握其实不是那么精确的。

就拿前文的例子来说，如果click事件的回调要消耗1000ms，那么后面的**Timer**和**Interval**定时器的回调都得等着，直接click事件的回调执行完毕。所以真正等到定时器回调执行时，可能早就过了原先设定的10ms了。

这点对于`setInterval`的影响更为严重。比如，

```javascript
setInterval(function(){
  console.log(2);
},1000);

(function (){
  sleeping(3000);
})();
```

上面的第一行语句要求每隔1000毫秒，就输出一个2。但是，第二行语句需要3000毫秒才能完成，请问会发生什么结果？

结果就是等到第二行语句运行完成以后，立刻连续输出三个2，然后开始每隔1000毫秒，输出一个2。也就是说，`setIntervel`具有累积效应，如果某个操作特别耗时，超过了`setInterval`的时间间隔，排在后面的操作会被累积起来，然后在很短的时间内连续触发，这可能或造成性能问题（比如集中发出Ajax请求）。

## setTimeout(fn, 0)的理解

记得以前有被人问过这句代码的含义😂😂😂。

按照常规理解，定时器的延时参数为0，表示是立即执行么？显然不是。

不管如何，只要经过`setTimeout`操作，那么`fn`将会被放到任务队列中排队。我们说一旦进行了任务队列，何时能执行那就由不得你了。得看排在你前面的任务是不是耗时任务。

所以这里延时参数的含义只有一个。那就是让`fn`早点去排队！如果任务队列中没有其他待执行任务，那么此时将会直接执行`fn`，其实这跟直接执行`fn`的区别不大。如果队列前面有待执行任务，那不好意思，你等着吧。

## 定时器的区别

其实`setTimeout`和`setInterval`这两个定时器是有区别的。当然我想说的区别并不是指它们功能上的区别。

**Timer**定时器在触发时，会将其回调任务压入任务队列进行排队。而**Interval**定时器在触发时，它不管当前Javascript线程的状态，它会无差别的往任务队列中压任务。如果某一个任务执行时占用了较多的Javascript线程时间，可能会导致**Interval**定时器连续压入多个回调。导致本来应该存在时间间隔的**Interval**定时器回调连续执行。

此外，当**Interval**定时器回调正在被当前Javascript线程运行时，此次**Interval**定时器触发将会被抛弃。因为此次触发的**Interval**定时器回调不会被入队。《Javascript高级程序设计》一书中也提到了这一点，

> 当使用`setInterval()`时，仅当没有该定时器的任何其他代码实例时，才将定时器代码添加到队列中。

如果你对**Interval**定时器的时间间隔要求比较严格的话，可以尝试使用循环加`setTimeout`的方式来模拟`setInterval`的功能。





