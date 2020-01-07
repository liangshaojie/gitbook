[深入理解Node.js中的进程与线程(原文)](http://www.inode.club/node/processAndThread.html#%E6%96%87%E7%AB%A0%E5%AF%BC%E8%A7%88)

## 面试会问

* Node.js是单线程吗？
* Node.js 做耗时的计算时候，如何避免阻塞？
* Node.js如何实现多进程的开启和关闭？
* Node.js可以创建线程吗？
* 你们开发过程中如何实现进程守护的？
* 除了使用第三方模块，你们自己是否封装过一个多进程架构?

[Node.js可以创建线程吗？](https://segmentfault.com/a/1190000014926921)

##  Node.js 的结构大致分为三个层次：

* 1、 Node.js 标准库，这部分是由 Javascript 编写的，即我们使用过程中直接能调用的 API。在源码中的 lib 目录下可以看到。

* 2、 Node bindings，这一层是 Javascript 与底层 C/C++ 能够沟通的关键，前者通过 bindings 调用后者，相互交换数据。

* 3、这一层是支撑 Node.js 运行的关键，由 C/C++ 实现。

V8：Google 推出的 Javascript VM，也是 Node.js 为什么使用的是 Javascript 的关键，它为 Javascript 提供了在非浏览器端运行的环境，它的高效是 Node.js 之所以高效的原因之一。
Libuv：它为 Node.js 提供了跨平台，线程池，事件池，异步 I/O 等能力，是 Node.js 如此强大的关键。
C-ares：提供了异步处理 DNS 相关的能力。
http_parser、OpenSSL、zlib 等：提供包括 http 解析、SSL、数据压缩等其他的能力。

## Node.js多进程架构模型

### 主进程 

master.js 主要处理以下逻辑：

* 创建一个 server 并监听 3000 端口。
* 根据系统 cpus 开启多个子进程
* 通过子进程对象的 send 方法发送消息到子进程进行通信
* 在主进程中监听了子进程的变化，如果是自杀信号重新启动一个工作进程。
* 主进程在监听到退出消息的时候，先退出子进程在退出主进程

``` javascript
// master.js
const fork = require('child_process').fork;
const cpus = require('os').cpus();

const server = require('net').createServer();
server.listen(3000);
process.title = 'node-master'

const workers = {};
const createWorker = () => {
    const worker = fork('worker.js')
    worker.on('message', function(message) {
        if (message.act === 'suicide') {
            createWorker();
        }
    })
    worker.on('exit', function(code, signal) {
        console.log('worker process exited, code: %s signal: %s', code, signal);
        delete workers[worker.pid];
    });
    worker.send('server', server);
    workers[worker.pid] = worker;
    console.log('worker process created, pid: %s ppid: %s', worker.pid, process.pid);
}

for (let i = 0; i < cpus.length; i++) {
    createWorker();
}

process.once('SIGINT', close.bind(this, 'SIGINT')); // kill(2) Ctrl-C
process.once('SIGQUIT', close.bind(this, 'SIGQUIT')); // kill(3) Ctrl-\
process.once('SIGTERM', close.bind(this, 'SIGTERM')); // kill(15) default
process.once('exit', close.bind(this));

function close(code) {
    console.log('进程退出！', code);

    if (code !== 0) {
        for (let pid in workers) {
            console.log('master process exited, kill worker pid: ', pid);
            workers[pid].kill('SIGINT');
        }
    }

    process.exit(0);
}
```

### 工作进程

worker.js 子进程处理逻辑如下：

* 创建一个 server 对象，注意这里最开始并没有监听 3000 端口
* 通过 message 事件接收主进程 send 方法发送的消息
* 监听 uncaughtException 事件，捕获未处理的异常，发送自杀信息由主进程重建进程，子进程在链接关闭之后退出

``` javascript
// worker.js
const http = require('http');
const server = http.createServer((req, res) => {
    res.writeHead(200, {
        'Content-Type': 'text/plan'
    });
    res.end('I am worker, pid: ' + process.pid + ', ppid: ' + process.ppid);
    throw new Error('worker process exception!'); // 测试异常进程退出、重启
});

let worker;
process.title = 'node-worker'
process.on('message', function(message, sendHandle) {
    if (message === 'server') {
        worker = sendHandle;
        worker.on('connection', function(socket) {
            server.emit('connection', socket);
        });
    }
});

process.on('uncaughtException', function(err) {
    console.log(err);
    process.send({
        act: 'suicide'
    });
    worker.close(function() {
        process.exit(1);
    })
})
```

