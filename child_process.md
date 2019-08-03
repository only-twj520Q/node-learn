

## 前言

### 进程

我们都知道计算机的核心是CPU，它承担了所有的计算任务，而操作系统是计算机的管理者，它负责任务的调度，资源的分配和管理，统领整个计算机硬件；应用程序是具有某种功能的程序，程序是运行于操作系统之上的。

摘自网络上的一段对进程的定义

> 进程是一个具有一定独立功能的程序在一个数据集上的一次动态执行的过程，是操作系统进行资源分配和调度的一个独立单位，是应用程序运行的载体。进程是一种抽象的概念，进程一般由程序，数据集合和进程控制块三部分组成。程序用于描述进程要完成的功能，是控制进程执行的指令集；数据集合是程序在执行时所需要的数据和工作区；程序控制块包含进程的描述信息和控制信息，是进程存在的唯一标志。

我们只需要记住，进程就是一个应用程序的实例，同一个应用程序可以起多个实例（进程）。另外，进程是资源分配的最小单位，线程是CPU调度的最小单位。

在linux中，我们通常通过`ps`命令来查看进程的信息。

### node环境

关于子进程

exec：启动一个子进程来执行命令，调用bash来解释命令，所以如果有命令有外部参数，则需要注意被注入的情况

spwan：更安全的启动一个子进程来执行命令，使用option传入各种参数来设置子进程的stdin、stdout等。通过内置的管道来与子进程建立IPC通信。

fork：spwan的特殊情况，专门用来产生worker或者worker池。返回值是childProcess对象，可以方便的与子进程交互。

IPC：进程间通信（Inter-process communication, IPC）其实是个很简单的概念，只要你将这个进程的数据传递到另外一个进程就是IPC了，要实现这个数据的传递方法。

Node.js 在启动子进程的时候，主进程先建立 IPC 频道，然后将 IPC 频道的 fd (文件描述符) 通过 **process.env** **环境变量**（NODE_CHANNEL_FD）的方式传递给子进程，然后子进程通过 fd 连上 IPC 与父进程建立连接。

#### nodejs真的是单线程吗

nodejs基于V8引擎，遵循的是单进程单线程的模式，但是nodejs的单线程是指js的引擎只有一个实例，且在nodejs的主线程中执行，同时node以事件驱动的方式处理IO等异步操作。

>在nodejs中，异步IO和事件驱动相关的线程通过libuv来实现内部的线程池和线程调度。libv中存在了一个Event Loop，通过Event Loop来切换实现类似于多线程的效果。简单的来讲Event Loop就是维持一个执行栈和一个事件队列，当前执行栈中的如果发现异步IO以及定时器等函数，就会把这些异步回调函数放入到事件队列中。当前执行栈执行完成后，从事件队列中，按照一定的顺序执行事件队列中的异步回调函数。也就是说node中的单线程是指js引擎只在唯一的主线程上运行，其他的异步操作，也是有独立的线程去执行，通过libv的Event Loop实现了类似于多线程的上下文切换以及线程池调度。线程是最小的进程，因此node也是单进程的。

下面列举一下nodejs的几个其它线程（部分）。

- 主线程：编译、执行代码。
- 编译/优化线程：在主线程执行的时候，可以优化代码。
- 分析器线程：记录分析代码运行时间，为 Crankshaft 优化代码执行提供依据。
- 垃圾回收线程
- 定时器线程(setTimeout, setInterval)

一句话总结，nodejs的单线程指的是js的执行是单线程的，但js的宿主环境无论是node还是浏览器都是多线程的，这种只维持一个主线程的模式大大减少了线程间切换的开销。

#### 单线程的缺陷

缺点也显而易见，nodejs的单线程使得在主线程不能进行CPU密集型操作，否则会阻塞主线程。对于CPU密集型操作，在nodejs中通过child_process可以创建独立的子进程，父子进程通过IPC通信，子进程可以是外部应用也可以是nodejs子程序，子进程执行后可以将结果返回给父进程。

### process

在nodejs中，process对象代表nodejs进程的一些信息，它是一个全局对象。可以通过它来获取nodejs进程的用户、环境等各种信息的属性、方法和事件。

**重要的属性**

* stdin：标准输入可读流
* stdout：标准输出可写流
* stderr：标准错误输出流
* argv：终端输入参数数组
* env：操作系统环境信息
* pid：应用程序进程id

**重要的方法**

* `process.memoryUsage()` ： 查看内存使用信息

* `process.nextTick()`： 当前eventloop执行完毕执行回调函数，属于微任务，优先级高于promise

* `process.chdir()`： 方法用于修改nodejs应用程序中使用的当前工作目录

* `process.cwd()`： 进程当前工作目录

* `process.kill()`： 杀死进程

* `process.uncaughtException()`： 当应用程序抛出一个未被捕获的异常时触发进程对象的uncaughtException事件

我们在前端脚手架开发的时候必须很熟悉process的常见属性和方法，下面简单列举两个demo。

#### demo1

```javascript
process.stdin.on('data', chunk => {
  process.stdout.write(`进程接收到数据${chunk}` )
})
```

控制台运行结果

![](https://ws2.sinaimg.cn/large/006tNc79ly1g5at48gj03j30a404lt8p.jpg)

#### demo2

```javascript
// index.js代码
console.log(process.argv)
```

![](https://ws1.sinaimg.cn/large/006tNc79ly1g5at5shq51j30d60330sv.jpg)

可以看到`process.argv`是一个数组，其中第一个元素是node的执行路径，第二个元素是当前执行文件的路径，从第三个元素开始，是执行时带入的参数。



## child_process

Nodejs提供了child_process模块来实现子进程，从而实现一个广义上的多进程的模式。通过child_process模块，可以实现1个主进程，多个子进程的模式，主进程称为master进程，子进程又称工作进程。在子进程中不仅可以调用其他node程序，也可以执行非node程序以及shell命令等等，执行完子进程后，以流或者回调的形式返回。熟悉shell脚本的知道，可以用它来做很多事情，比如文件压缩、增量部署。

### 创建子进程的方式

child_process提供了几种创建子进程的方式。

- 异步方式：spawn、exec、execFile、fork
- 同步方式：spawnSync、execSync、execFileSync

其中后缀加Sync是对应的同步版本，比如spawnSync是spawn的同步方法，所以我们只简单介绍spawn、exec、execFile、fork这四种。

#### exec

概念：创建一个shell，然后在shell里面执行命令。命令完成后，将stdout，stderr作为参数传入回调方法。

使用方法

```javascript
child_process.exec(command[, options][, callback])
```

参数说明

- command：要执行的命令
- options：配置项
  - cwd：当前工作路径
  - env：环境变量
  - encoding：编码，默认是utf-8
  - shell：用来执行命令的shell，unix上默认是/bin/shell
  - Timeout：进程运行时间上限，如果超过该时间，就会给进程发送killSingal信号。默认是0
  - killSignal：默认是SIGTERM
  - maxBuffer：标准输出、错误输出最大允许的数据流
  - uid：执行进程的uid
  - gid：执行进程的gid

实际使用

```javascript
const { exec } = require('child_process');
exec('ls -l', (error, stdout, stderr) => {
  if(error) {
    console.error('error:' + error);
  }
  console.log('stdout:' + stdout);
  console.log('stderr:' + stderr);
});
```

#### execFile

概念：和exec类似，不同的地方在于没有创建一个新的shell。

使用方法

```javascript
child_process.execFile(file[, args][, options][, callback])
```

参数说明

* file：可执行文件的名字或者路径
* args：参数
* options：配置

实际使用

```javascript
const { execFile } = require('child_process');
execFile('echo', ['hi'], (error, stdout, stderr) => {
	if(error) {
    console.error('error:' + error);
  }
  console.log('stdout:' + stdout);
  console.log('stderr:' + stderr);
});
execFile('ls -al', {shell: '/bin/bash'}, (error, stdout, stderr) => {
	if(error) {
    console.error('error:' + error);
  }
  console.log('stdout:' + stdout);
  console.log('stderr:' + stderr);
});
```

解释：execFile会在环境变量的路径中，比如/usr/local/bin，寻找是否存在echo这个程序，传入hi这个参数，执行后返回。

本质上，exec最后调用的就是execFile方法。唯一的区别是exec中调用的方法会将shell配置属性设置为true。而在execFile方法中，最终调用的是spawn方法。exec方法会将spawn的输出流转化成string，默认使用UTF-8的编码，传给回调函数。需要注意的是，buffer是有最大缓存区的，如果超出会直接被kill调，可以通过maxBuffer属性进行配置。

#### spawn

概念：用于执行非node应用，且不能直接执行shell。spawn执行后的结果并不是一次性输出的，而且以流的形式输出的。

使用方法

```javascript
child_process.spawn(command[, args][, options])
```

* command：要执行的命令
* args：传递给命令的参数
* options：配置项

实际使用

```javascript
const { spawn } = require('child_process');
const child = spawn('ls', ['-al']);
```

控制台并没有任何的输出，因为子进程有自己的stdio流（stdin,stdout,stderr），控制台的输出是与当前主进程的stdio绑定的。因此如果希望看到输出信息，可以通过在子进程的stdout与当前进程的stdout之间建立管道实现。

```javascript
child.stdout.pipe(process.stdout);
```

 或者通过监听事件的方式

```javascript
child.stdout.on('data', data => {
	process.stdout.write(data)
})
```

在nodejs中使用的`console.log`其实底层依赖的就是`process.stdout`。通过spawn创建的子进程，继承自EventEmitter，所以可以在上面进行事件（`discount`，`error`，`close`，`message`）的监听。同时子进程具有三个输入输出流：stdin、stdout、stderr，通过这三个流，可以实时获取子进程的输入输出和错误信息。

来看另一个例子，如果有多个shell命令需要执行。比如读取一个文件，然后统计文件中的文字行数，最后用流的形式输出.

```javascript
const { spawn } = require('child_process');
let cat = spawn('cat', ['./test.txt']);
let grep = spawn('wc', ['-l']);

cat.stdout.pipe(grep.stdin);
grep.stdout.pipe(process.stdout);
```

#### fork

概念：衍生一个新的nodejs进程，并通过建立一个IPC通讯通道来调用一个指定的模块，该通道允许父进程与子进程之间互相发送消息。在js层，我们可以使用`process.send(message)`和`process.on('message', msg => {})`进行通信。

使用方法

```javascript
child_process.fork(modulePath[, args],[, options])
```

参数说明

* modulePath：子进程的路径或模块
* args：参数
* options：配置
  * execPath：用来创建子进程的可执行文件，默认是/usr/local/bin/node。也是说，可以通过execPath来指定具体的node可执行文件路径。（比如多个node版本）
  * exexArgv：传给可执行文件的参数列表。默认是process.execArgv，跟父进程保持一致。
  * slient：默认是false，即子进程的stdio从父进程继承。如果是true，则直接pipe向子进程的child.stdin、child.stdout等。子进程的stdout属性可以通过监听data事件，获取输出结果。
  * stdio：如果设置了stdio，则会覆盖slient选项的设置。

实际使用

```javascript
// 父子通信
// parent.js
const { fork } = require('child_process');
let child = fork('./child.js');
child.on('message', msg => {
	console.log('message from child', msg);
});
child.send({ msg: 'hello child' });

// child.js
process.on('message', msg => {
  console.log('message from parent', msg)
});
process.send({ msg: 'hello parent' });
```

执行结果如下图

![](https://ws3.sinaimg.cn/large/006tNc79ly1g5b68iuhb9j30bc01qjrh.jpg)



### 四种方式的对比

#### 不同点



![](https://ws1.sinaimg.cn/large/006tNc79ly1g5b1zy02r3j30h70endgh.jpg)





**exec**：子进程执行的是非node程序，传入一串shell命令，执行后结果以回调的形式返回，与execFile 不同的是exec可以直接执行一串shell命令。

**execFile**：子进程中执行的是非node程序，提供一组参数后，执行的结果以回调的形式返回。它不会创建子shell环境

**fork**：子进程执行的是node程序，提供一组参数后，执行的结果以流的形式返回，与spawn不同，fork生成的子进程只能执行node应用。fork方式创建的子进程与父进程是完全独立的，它拥有单独的内存，单独的V8实例，因此不推荐创建很多的nodejs子进程。它也是spawn方法的一个特例。

**spawn** ： 子进程中执行的是非node程序，提供一组参数后，执行的结果以流的形式返回。

#### 安全性

exec的执行等级很高，执行后会出现安全性的问题，比如下面的代码。通过exec是可以直接执行的，rm -rf会删除当前目录下的文件。

```javascript
const { exec } = require('child_process');
exec('rm -rf test.js', (error, stdout, stderr) => {})
```

而execFile在传入参数的同时，会检测传入实参执行的安全性，如果存在安全性问题，会抛出异常。除了execFile外，spawn和fork也都不能直接执行shell，因此安全性较高。

#### 相同点

exec、execFile和fork方法，归根到底都是通过spawn方法实现的，而spawn方法底层调用了libuv进程的进程管理。内部调用关系如下。

> exec() -> execFile() -> spawn()

![](https://ws1.sinaimg.cn/large/006tNc79ly1g5cbglgi6dj30hi0dodg8.jpg)



### 事件

上述方式创建出的子进程都继承了EventEmitter类。所以子进程可以通过监听事件来实现通信。

常见的事件有close，exit，error等。

#### close

当子进程的stdio流关闭的时候会触发，注意的是并不是子进程exit的时候close事件就一定会触发，因为多个子进程可以共用相同的stdio。

####exit

exit事件的回调函数有两个参数code和signal，code是代码子进程最终的退出码，如果子进程是由于接收到signal信号终止的话，signal会记录子进程接受的signal值。如果子进程是自己退出的，那么`code`就是退出码，否则为null；如果子进程是通过信号结束的，那么，`signal`就是结束进程的信号，否则为null。这两者中，肯定不会都是null。

SIGINT：interrupt，程序终止信号，通常在用户按下CTRL+C时发出，用来通知前台进程终止进程。 SIGTERM：terminate，程序结束信号，该信号可以被阻塞和处理，通常用来要求程序自己正常退出。

```javascript
const { exec } = require('child_process');
const child = exec('ls');
child.on('exit', function(code, signal) {
  console.log(code);
  console.log(signal);
});
```

输出结果

![](https://ws2.sinaimg.cn/large/006tNc79ly1g5c874wm0yj307z01tdfo.jpg)

#### error

error事件的触发条件有下面几条

- 无法创建进程
- 无法结束进程
- 给进程发送消息失败

*注意当代码执行出错的时候，error事件并不会触发，exit事件会触发，code为非0的异常退出码*

```javascript
// parent.js
const { exec } = require('child_process');
const child = exec('node test.js', {
  timeout: 3000
});

child.on('error', function(code, signal) {
  console.log('error');
  console.log(code);
  console.log(signal);
});
child.on('exit', function(code, signal) {
  console.log('exit');
  console.log(code);
  console.log(signal);
});

// test.js
throw new Error('我错了')
```

![](https://ws4.sinaimg.cn/large/006tNc79ly1g5mv8isimej308g02ejr9.jpg)

```javascript
const { exec } = require('child_process');
const child = exec('node test.js', {
  timeout: 1
});

child.on('error', function(code, signal) {
  console.log('error');
  console.log(code);
  console.log(signal);
});
child.on('exit', function(code, signal) {
  console.log('exit');
  console.log(code);
  console.log(signal);
});
```

![](https://ws4.sinaimg.cn/large/006tNc79ly1g5mv97v0cqj308m02p0sn.jpg)

#### message

当采用`process.send()`来发送消息时触发。 参数：`message`，为json对象。message事件适用于父子进程之间建立IPC通信管道的时候信息传递。



## 负载均衡

### cluster

除了用nginx做反向代理，node本身也提供了一个`cluster`模块，用于多核CPU环境下多进程的负载均衡。cluster模块创建子进程本质上是通过child_procee.fork，利用该模块可以很容易的创建共享同一端口的子进程服务器。下面是一个官方实例。具体细节下次再讲。

```javascript
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) { // 判断是否为主进程
  console.log(`主进程 ${process.pid} 正在运行`);

  // 衍生工作进程。
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`工作进程 ${worker.process.pid} 已退出`);
  });
} else { // 子进程进行服务器创建
  // 工作进程可以共享任何 TCP 连接。
  // 在本例子中，共享的是一个 HTTP 服务器。
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('hello world\n');
  }).listen(8000);

  console.log(`工作进程 ${process.pid} 已启动`);
}
```

