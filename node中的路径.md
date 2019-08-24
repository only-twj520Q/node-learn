## 前言

在做webpack配置或者做node脚手架开发的时候，我们经常需要操作和访问文件，避免不了遇到很多路径的问题。这边简单整理一下常见的一些用法。



## 路径变量

node中常见的路径变量和相关的方法有dirname，filename，process.cwd，nodejs原生的path模块。

###区别

#### 实战一

新建一个项目node-demo，执行npm init，然后在项目根目录下新建目录src，在src目录下新建文件index.js。

```javascript
const path = require('path');
console.log(__dirname);
console.log(__filename);
console.log(process.cwd());
console.log(path.resolve('./'));
```

在项目根目录下执行 `node src/index`，输出结果如下

```javascript
/* 输出结果
/Users/apple/Desktop/node-demo/src
/Users/apple/Desktop/node-demo/src/index.js
/Users/apple/Desktop/node-demo
/Users/apple/Desktop/node-demo
*/
```

如果我们改变node的执行路径，`cd src`，切换到src目录下，然后执行`node index`，输出结果如下

```javascript
/* 输出结果
/Users/apple/Desktop/node-demo/src
/Users/apple/Desktop/node-demo/src/index.js
/Users/apple/Desktop/node-demo/src
/Users/apple/Desktop/node-demo/src
*/
```

如果我们在src目录下，执行`npm start`也就是执行package.json中的脚本`node src/index`，输出结果如下

```javascript
/* 输出结果
/Users/apple/Desktop/node-demo/src
/Users/apple/Desktop/node-demo/src/index.js
/Users/apple/Desktop/node-demo
/Users/apple/Desktop/node-demo
*/
```

对比上面的输出结果，第三组的输出结果和第一组是完全一样的，因为我们执行`npm start`的时候执行路径其实已经切换到根目录下了。

如果我们使用使用babel cli工具将源码打包转化成新的代码到dist/static文件夹下

比如，我们在`package.json`中新增build命令，然后执行`npm run build`，再执行`node dist/static `

```javascript
"build": "babel src -d dist/static"
```

```javascript
/* 输出结果
/Users/apple/Desktop/node-demo/dist/static
/Users/apple/Desktop/node-demo/dist/static/index.js
/Users/apple/Desktop/node-demo
/Users/apple/Desktop/node-demo
*/
```

#### 结论

dirname：返回被执行的js所在文件夹的绝对路径

filenname：返回被执行的js的绝对路径

process.cwd：返回运行启动脚本或者node命令所在的文件夹的绝对路径

./：返回运行启动脚本或者node命令所在的文件夹的绝对路径

#### 实战二

现在我们在src的目录下增加一个文件demo.js，它和index.js属于同级目录。

```javascript
// demo.js
const obj = {
  number: 10
}
module.exports = obj

// index.js
fs.readFile('./demo.js', 'encoding:utf8', function (err, data) {
  if (err) {
    console.log('err', err);
    return;
  }
  console.log('data', data);
})
```

如果我们在项目根目录下执行`node src/index`或者`npm start`会提示找不到目标文件。而如果在src目录下执行`node index`是可以正常执行的。

还有一种情况是我们使用require方法，引入模块，使用相对路径也是可以正常执行的。

```javascript
const demo = require('./demo.js');
console.log(demo)
// 输出结果 { number: 10 }
```

所以关于./路径的使用，在require中的使用是相当于dirname的作用，和启动脚本的目录位置没有关系，而在`fs.readFile`的情况下，我们必须使用绝对路径，保证文件可以正常访问到。



## path模块

在nodejs中，path是个使用频率很高的原生的nodejs模块，如果想入门nodejs，这是个必须掌握的模块，下面介绍一些path中常用的方法。

#### dirname

用法：`path.dirname(path)`

作用：返回path的目录名

```javascript
const path = require('path');

const testPath = '/project/src/index.js';
console.log(path.dirname(testPath));
// 输出结果 project/src
```

如果 `path` 不是字符串，则抛出 [`TypeError`](http://nodejs.cn/s/Z7Lqyj)。

#### basename

用法：`path.basename(path[, ext])`

作用：返回文件名

```javascript
const path = require('path');

const testPath = '/project/src/index.js';
console.log(path.basename(testPath));
// 输出结果 index.js

const testPath = '/project/src/';
console.log(path.basename(testPath));
// 输出结果 src

// 如果只想获取文件名，可以用上第二个参数。
const testPath = '/project/src/index.js';
console.log(path.basename(testPath, '.js'));
// 输出结果 index
```

#### normalize

用法：`path.normalize(path)`

作用：规范路径

```javascript
const path = require('path');

const testPath = '/desktop///ab//';
console.log(path.normalize(testPath));
// 输出结果 /desktop/ab
```

规范化路径，把不规范的路径规范化。

#### parse

用法：`path.parse(path)`

作用：解析路径

```javascript
const testPath = '/project/src/index.js';
console.log(path.parse(testPath));
// 输出结果 
{ root: '/', dir: '/project/src', base: 'index.js', ext: '.js', name: 'index' }
const testPath = 'project/src/';
console.log(path.parse(testPath));
// 输出结果 
{ root: '', dir: 'project', base: 'src', ext: '', name: 'src' }
```

返回结果是一个对象，每个字段的含义如下。

* root：代表根目录
* dir：代表文件所在的文件夹
* base：代表文件
* name：代表文件名
* ext：代表文件的后缀名

#### join

用法：`path.join([...paths])`

作用：拼接路径

```javascript
console.log(path.join('a', 'b/', '/c'));
// 输出结果 a/b/c
console.log(path.join('/a', 'b/', '/c'));
// 输出结果 /a/b/c
console.log(path.join('/a', 'b/', '../c'));
// 输出结果 /a/c
console.log(path.join(''));
// 输出结果 .
console.log(path.join('a', 'b', 'index.js/'));
// 输出结果 a/b/index.js/
path.join('foo', {}, 'bar');
```

方法返回的是一个拼接好的路径，但是根据平台的不同，他会对路径进行不同的规范化，举个例子，`Unix`系统是`/`，`Windows`系统是`\`，那么你在两个系统下看到的返回结果就不一样。如果任何路径片段不是字符串，则抛出 [`TypeError`](http://nodejs.cn/s/Z7Lqyj)。

#### resolve

用法：`path.resolve([...paths])`

作用：拼接路径

```javascript
console.log(path.resolve('/a', 'b/', 'c'));
// 输出结果 /a/b/c
console.log(path.resolve('a', 'b/', 'c'));
// 输出结果 /Users/apple/Desktop/node-demo/a/b/c
console.log(path.resolve('a', 'b/', 'c/', '../'));
// 输出结果 /Users/apple/Desktop/node-demo/a/b
console.log(path.resolve('a', 'b/', 'c/', '/d'));
// 输出结果 /d
console.log(path.resolve('./a', 'b'));
// 输出结果 /Users/apple/Desktop/node-demo/a/b
console.log(path.resolve(''));
// 输出结果 /Users/apple/Desktop/node-demo
```

方法其实就是相当于shell中的cd操作，返回的是执行cd操作以后的绝对路径。