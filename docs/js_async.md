- [1. JS单线程的原因](#1-JS单线程的原因)
- [2. JS同步模式和异步模式](#2-JS同步模式和异步模式)
- [3. 常见异步任务](#3-常见异步任务)
   - [3.1 SetTimeOut&setInterval](#3-1-SetTimeOut-and-setInterval)
   - [3.2 IO](#3-2-IO)
- [4. 线程|事件循环|任务队列](#4-线程_事件循环_任务队列)
   - [4.1 JS的线程](#4-1-JS的线程)
   - [4.2 任务队列 task queue](#4-2-任务队列task_queue)
   - [4.3 事件循环 event loop](#4-3-事件循环event_loop)
- [5. 一段话概括JS异步](#getting-started-with-markdown)
- [6. 异步方案](#getting-started-with-markdown)
  - [Promise](#getting-started-with-markdown)
    - [Promise错误处理](#getting-started-with-markdown) 
    - [Promise.reject](#getting-started-with-markdown)
    - [Promise的执行时序](#getting-started-with-markdown)
    - [宏任务(macro)task](#getting-started-with-markdown)
    - [微任务(micro)task](#getting-started-with-markdown)
    - [Generator 异步方案、Async/Await 语法糖](#getting-started-with-markdown)
    - [Async/Await 语法糖](#getting-started-with-markdown)

----------------------------------



# 1-JS单线程的原因
JavaScript 之所以采用**单线程**，而不是多线程，跟历史有关系。JavaScript 从诞生起就是单线程，原因是不想让浏览器变得太复杂，因为多线程需要共享资源、且有可能修改彼此的运行结果，对于一种网页脚本语言来说，这就太复杂了。如果 JavaScript 同时有两个线程，一个线程在网页 DOM 节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？是不是还要有锁机制？所以，为了避免复杂性，JavaScript 一开始就是单线程，这已经成了这门语言的核心特征，将来也不会改变。
这种模式的好处是实现起来比较简单，执行环境相对单纯；坏处是只要有一个任务耗时很长，后面的任务都必须排队等着，会拖延整个程序的执行。常见的浏览器无响应（假死），往往就是因为某一段 JavaScript 代码长时间运行（比如死循环），导致整个页面卡在这个地方，其他任务无法执行。JavaScript 语言本身并不慢，慢的是读写外部数据，比如等待 Ajax 请求返回结果。这个时候，如果对方服务器迟迟没有响应，或者网络不通畅，就会导致脚本的长时间停滞。
如果排队是因为计算量大，CPU 忙不过来，倒也算了，但是很多时候 CPU 是闲着的，因为 IO 操作（输入输出）很慢（比如 Ajax 操作从网络读取数据），不得不等着结果出来，再往下执行。JavaScript 语言的设计者意识到，这时 CPU 完全可以不管 IO 操作，挂起处于等待中的任务，先运行排在后面的任务。等到 IO 操作返回了结果，再回过头，把挂起的任务继续执行下去。这种机制就是 JavaScript 内部采用的“事件循环”机制（Event Loop）。单线程模型虽然对 JavaScript 构成了很大的限制，但也因此使它具备了其他语言不具备的优势。如果用得好，JavaScript 程序是不会出现堵塞的，这就是为什么 Node 可以用很少的资源，应付大流量访问的原因。
为了利用多核 CPU 的计算能力，HTML5 提出 Web Worker 标准，允许 JavaScript 脚本创建多个线程，但是子线程完全受主线程控制，且不得操作 DOM。所以，这个新标准并没有改变 JavaScript 单线程的本质。

# 2-JS同步模式和异步模式
要想理解JS的异步原理，先要理解JS的同步和异步。在单线程中，同步任务是那些在主线程上排队执行的任务。只有前一个任务执行完毕，才能执行后一个任务。如果任务总是按照这样的顺序执行，那么后续任务的执行可以假设所有前面任务都没有错误地执行，并且确保它们的所有结果都可以使用——这在逻辑上是某种简化。
**异步任务**指的是，不进入主线程、而进入"任务队列"（task queue）的任务，**这个队列的所有任务都是不进入主线程执行**，**而是被浏览提供的线程执行**，**当执行完毕后就会产生一个回调函数，并且通知主线程**，在主线程执行完当前所执行的任务后，就会调取最早通知自己的回调函数，使其进入主线程中执行。
大多数现代操作系统都有event notification。比如，我们有一个read function，它负责读取服务器的response。那么这个function就是个异步任务，它会被阻塞(block) 直到服务器返回response。
在这个过程中，我们的应用可以让系统监视服务器端口并把event notification存入队列中。应用可以在处理完一些scripting之后在来查看events。这个过程就是异步，因为应用在一个时间点表现出对异步任务的兴趣，但是在另一个时间点才用到它。

# 3-常见异步任务
## 3-1 SetTimeOut and setInterval
最基础的异步是setTimeout和setInterval函数，很常见，但是很少人有人知道其实这就是异步，因为它们可以控制js的执行顺序。

``` js
        console.log( "1" );
        setTimeout(function() {
            console.log( "2" )
        }, 0 );
        setTimeout(function() {
            console.log( "3" )
        }, 0 );
        setTimeout(function() {
            console.log( "4" )
        }, 0 );
        console.log( "5" );

// ouput
// 1
// 5
// 2
// 3
// 4
```

from the output 可见，尽管我们设置了setTimeout（function，time）中的等待时间为0，结果其中的function还是后执行。<br>
火狐浏览器的api文档有这样一句话：Because even though setTimeout was called with a delay of zero, it's placed on a queue and scheduled to run at the next opportunity, not immediately. Currently executing code must complete before functions on the queue are executed, the resulting execution order may not be as expected.
这里说到了一个“队列”（即任务队列 task queue），该队列放的是什么呢，放的就是setTimeout中的callback，这些callback依次加入该队列，即该队列中所有callback中的程序将会在该队列以外的所有代码执行完毕之后再以此执行，这是为什么呢？因为在执行程序的时候，浏览器默认setTimeout以及ajax请求这一类的方法都是异步任务，将其加入一个队列中。我们后面会详细解释.

## 3-2 IO
前面说过了，I/O操作(input/output), 属于异步任务。比如ajax请求，当我们向server发出请求时，我们不需要等，晚点再来处理ajax的response。

# 4-线程_事件循环_任务队列
## 4-1 JS的线程

**其实同步和异步，无论如何，做事情的时候都是只有一条流水线（单线程），同步和异步的差别就在于这条流水线上各个流程的执行顺序不同。**

这里有人就会问到，JavaScript明明单线程语言，但又同时可进行异步操作，这两者不是完全相反的概念吗？没错，JavaScript是单线程，它的宿主，**浏览器**可**不是单线程**的。

Javascript是单线程的，但是却能执行异步任务，这主要是因为 JS 中存在**`事件循环`（Event Loop）和`任务队列`（Task Queue）**。


## 4-2 任务队列task_queue

异步操作会将相关异步任务加到任务队列中。任务队列是用来存储各种events的地方。主线程中的同步任务全部清空之后，主线程才会去查看任务队列。

而不同的异步操作添加到任务队列的时机也不同，如**`onclick`, `setTimeout`,`ajax`** 处理的方式都不同，这些异步操作是由浏览器内核的**`webcore`**来执行的，`webcore`包含下图中的3种 webAPI，分别是**`DOM Binding`、`network`、`timer`模块。**

- **DOM Binding** 模块处理一些DOM绑定事件，如`onclick`事件触发时，回调函数会立即被`webcore`添加到任务队列中。
- **network** 模块处理`Ajax`请求，在网络请求返回时，才会将对应的回调函数添加到任务队列中。
- **timer** 模块会对`setTimeout`等计时器进行**延时处理**，当时间到达的时候，才会将回调函数添加到任务队列中。

## 4-3 事件循环event_loop
![image](https://user-images.githubusercontent.com/44330488/182200584-40db560f-1025-4025-9a8f-13251c7a4bd0.png)
![image](https://user-images.githubusercontent.com/44330488/182200653-4a77ec2c-97fa-4889-a8e0-9d88c49e9dfe.png)

一个事件循环就是它听起来的样子，有一个event的队列（所有发生的event都存储在这里——上图中称为“任务队列”）还有一个loop，它不断地将这些event从队列中取出，并调用事件中的回调（call stack会执行所有的回调）。API是用于处理异步函数的API，比如说处理等待来自客户端或server的响应。

在此流程中，所有function call首先进入call stack，然后通过**API**执行异步任务。当异步任务完成后，callback进入**任务队列**，然后再次进入call stack。当任务执行完之后，event loop会再次去task queue重复上面的流程。

[**check this video for more detail about event loop**](https://www.youtube.com/watch?v=8aGhZQkoFbQ)

