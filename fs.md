## 前言

fs模块是nodejs内置的文件系统模块，负责系统的文件读写，增删等操作，是必须掌握的模块。

几乎所有方法都同时提供了异步和同步的版本，这是因为IO操作本身就是一个异步。



## 常用功能列表

先主要列举了👇的一些常用操作

`fs.readFile`读取文件

`fs.readdir`读取目录

`fs.writeFile`创建写入文件

`fs.appendFile`追加文件

`fs.stat`检测是文件还是目录

`fs.mkdir`创建目录

`fs.rename`重命名

`fs.rmdir`删除目录

`fs.unlink`删除文件 



## 代码实践

下面会具体使用代码演示一些常用的基本操作

### 读取文件

#### 同步版本

```javascript
const fs = require('fs');

let data;
try {
  data = fs.readFileSync('./test.txt', 'utf8');
  console.log(`文件内容: ${data}`);
} catch (err) {
  console.log(`读取文件出错: ${err.message}`);
}
```

控制台输出结果

![image-20190727180208135](https://ws4.sinaimg.cn/large/006tNc79ly1g5ervr348uj309t01taa2.jpg)

#### 异步版本

```javascript
const fs = require('fs');

fs.readFile('./test.txt', 'utf8', (err, data) => {
  if (err) {
    return console.log(`读取文件出错: ${err.message}`);
  }
  console.log(`文件内容: ${data}`);
})
```

输出结果和上面的一致，返回的结果写入到回调的第二个参数data。当文件不存在的时候，会报错。



### 文件流读取文件

这边简单说一下stream这个对象，这是一种特殊的数据结构，叫做流。我们可以把它想象成水流，它可以从源头流动到目的地。在linux中也有类似这种流的概念，比如标准输入流stdin，它的含义的是键盘打字输入到命令行或者应用程序，标准输出流stdout则是一个相反的操作。我们常用的`cat`方法就是从标准输入读入文本流，并输出到标准输出。我们还可以通过pipe管道把标准输入流连接到标准输出流。

在读取文件的时候，nodejs也提供了这种方法`createReadStream`。创建出来的对象继承于nodejs的模块`EventEmitter`，所以它可以通过on这个方法来监听各种各种事件，比如data事件

```javascript
const fs = require('fs');
// 创建一个读取流
const readStream = fs.createReadStream('./test.txt', 'utf8');
readStream
  .on('data', chunk => {
    console.log(`文件流读取数据:${chunk}`);
  })
  .on('error', err => {
    console.log(`文件流读取出错:${err.message}`);
  })
  .on('end', () => {  // 没有数据了
    console.log('没有数据了');
  })
  .on('close', () => {  // 已经关闭，不会再有事件抛出
    console.log('已经关闭');
  });
```

控制台输出结果



![](https://ws4.sinaimg.cn/large/006tNc79ly1g5epxihsv8j309302cmx9.jpg)

需要注意的是这个data事件，在文件内容比较大的时候，不止一次被触发。这个也好理解，文件流比较大，而流在内存中开辟的内存区域有限，所以不能一次性全部返回所有内容，只能分片返回，正如这个参数的名字叫chunk一样，它只是一个分片。下面我们实验验证一下。

```javascript
let count = 0;
const readStream = fs.createReadStream('./test.txt', 'utf8');
readStream
  .on('data', chunk => {
    count++;
  })
  .on('error', err => {
    console.log(`文件流读取出错:${err.message}`);
  })
  .on('end', () => {  // 没有数据了
    console.log('没有数据了');
  })
  .on('close', () => {  // 已经关闭，不会再有事件抛出
    console.log('已经关闭');
    console.log(`读取了${count}次`)
  });
```

控制台输出结果

![](https://ws4.sinaimg.cn/large/006tNc79ly1g5eqes0dvnj307x02gq2y.jpg)

我们把text.txt文件的内容增加一下，可以看到一共触发了五次data事件。

#### pipe

上面提到了文件的读取流，与之对应的还有一个写入流`createWriteStream`。我们用`pipe`把一个文件流和另一个文件流串起来，这样源文件的所有内容就写入到目标文件里了，这样就实现了一个类似文复制的操作。简单高效。默认情况下，当`Readable`流的数据读取完毕，`end`事件触发后，将自动关闭`Writable`流。

```javascript
const readStream = fs.createReadStream('test.txt');
const writeStream = fs.createWriteStream('testCopied.txt');
try {
  readStream.pipe(writeStream);
} catch (err) {
  console.log(`文件流出错:${err.message}`);
}
```



### 读取目录

#### 同步版本

```javascript
const fs = require('fs');

let data;
try {
  data = fs.readdirSync('./第一级目录');
  console.log('读取目录成功');
  console.log(data);
} catch(err) {
  console.log(`读取目录出错:${err.message}` );
}
console.log(data);
```

控制台输出结果

![](https://ws3.sinaimg.cn/large/006tNc79ly1g5epmhwxjuj30a701r74a.jpg)

需要注意的是`fs.readdirSync()`只会读一层，所以需要判断读取的文件类型是否为目录，如果是，则进行递归遍历。

#### 异步版本

```javascript
const fs = require('fs');

fs.readdir('./第一级目录', (err, data) => {
  if (err) {
    return console.log(err);
  }
  console.log('读取目录成功');
  console.log(data);
})
```



### 写入文件

#### 同步版本

```javascript
const fs = require('fs');

fs.writeFile('test2.txt', '这是写入的数据', err => {
  if (err) {
     return console.log(`写入失败：${err.message}`);
  }
  console.log('写入成功');
})
```

需要注意的几点。第一，写入的时候是覆盖原文件中的所有内容，如果想要追加内容，可以使用`appendFile`。第二，函数第二个参数是写入的内容，可以是字符串或者buffer数据，第三个参数默认是utf-8，当传入的是一个buffer的时候，该值忽略。第三，当文件不存在的时候，会自动创建一个文件。

#### 异步版本

```javascript
const fs = require('fs');

try{
  fs.writeFileSync('test2.txt');
  console.log('写入成功');
} catch(err) {
  console.log(`写入失败：${err.message}`);
}
```



## 检查读取的文件类型

#### 同步版本

```javascript
const fs = require('fs');

fs.stat('package.json', (error, stats) => {
  if (error) {
    return console.log(error);
  }
  console.log(stats);
  console.log(`文件：${stats.isFile()}`);
  console.log(`目录：${stats.isDirectory()}`);
})
```

控制台输出结果

![](https://ws1.sinaimg.cn/large/006tNc79ly1g5ek2q742kj30bd0dgabk.jpg)

我们把stats对象输出，发现了一堆不知道什么含义的字段，上网查了一下这些字段的含义，找到了部分字段的解释。

* size：文件大小

- atime：访问时间
- mtime：文件内容修改时间
- ctime： 文件状态修改时间，比如修改文件所有者、修改权限、重命名等
- birthtime：创建时间

#### 异步版本

```javascript
const fs = require('fs');

let stats = fs.statSync('package.json');
```



### 创建目录

上面列举的同步和异步版本都很相似，所以下面不再列举。

```javascript
const fs = require('fs');
//创建文件
fs.mkdir('pro', (err) => {
  if(err) {
    return console.log(err);
  }
  console.log("创建目录成功");
})
```

控制台输出结果

![image-20190727181104300](https://ws1.sinaimg.cn/large/006tNc79ly1g5ervgs22oj30bu03p3yv.jpg)

可以看到首次执行成功，当前目录下会多了一个pro文件夹，再次执行同样的代码，会报错，提示文件已经存在。对应的删除使用`rmdir`，这里不再赘述了。



### 文件是否存在

```javascript
const fs = require('fs');

fs.access('./test.txt', err => {
  if (err) {
    return console.log('test.txt文件不存在');
  };
  console.log('test.txt存在');
});
```

控制台输出结果

![](https://ws2.sinaimg.cn/large/006tNc79ly1g5epdq8vxqj308m018dfp.jpg)



### 创建目录

```javascript
const fs = require('fs');

fs.mkdir('./demotest', function(err){
  if(err) return console.log('目录创建成功失败');
  console.log('目录创建成功');
});
```

控制台输出结果

![](https://ws2.sinaimg.cn/large/006tNc79ly1g5epfizxqdj307p019mx2.jpg)



### 重命名

```javascript
fs.rename('test.txt', 'new-test.txt' , function(err){
  if(err) return console.log(`重命名失败：${err.message}`);
  console.log('重命名成功');
});
```

控制台输出结果

![](https://ws2.sinaimg.cn/large/006tNc79ly1g5erbexvu5j308a01b746.jpg)

这个方法还有一个作用就是剪切，看下面这个例子

```javascript
fs.rename('test.txt', './第一级目录/test.txt' , function(err){
  if(err) return console.log(`重命名失败：${err.message}`);
  console.log('重命名成功');
});
```

![](https://ws3.sinaimg.cn/large/006tNc79ly1g5ereiq3wsj305l064t8r.jpg)



### 检测文件的改动

```javascript
const fs = require('fs');

const options = {
  persistent: true,  // 默认就是true，表示持续监视，不退出程序
  interval: 2000  //  单位毫秒，表示每隔多少毫秒监视一次文件
};

fs.watchFile('./test.txt', options, function(curr, prev){
  console.log(curr);
  console.log(prev);
});
```

控制台输出结果

![](https://ws2.sinaimg.cn/large/006tNc79ly1g5eruxcsfxj30br0dv75v.jpg)

可以看到curr和prev都是stats对象，所以我们可以得到文件的修改时间。通过这个方法，我们也可以做一些修改代码文件以后，服务重启之类的操作。