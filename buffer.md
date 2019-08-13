## 前言

原生js语言中的数据类型中没有二进制数据类型，所以也没有操作二进制数据流的机制。而在服务器端，我们通常需要处理文件流的读写、网络请求数据，tcp流等等，这时候必须用到二进制数据。所以在nodejs中，定义了一个Buffer类，该类用来创建一个专门存放二进制数据的缓存区。

#### 作用

在nodejs中，Buffer存在于全局对象上，无需引入模块即可使用。Buffer是在内存中开辟了一块用于存放二进制数据的区域。它为nodejs带来了一种存储原始数据的方法。原始数据存储在Buffer类的实例中。一个Buffer实例类似于一个数组，对应内存堆之外的一块原始内存。

#### 应用场景

* 数据流，比如IO操作或者完成网络响应。流是数据的集合（与数据、字符串类似），但是流的数据不能一次性获取到，数据也不会全部load到内存中，因此流非常适合大数据处理以及断断续续返回chunk的外部源。流的生产者与消费者之间的速度通常是不一致的，因此需要Buffer来暂存一些数据。Buffer可以在stream中与二进制数据进行交互和操作。当你有一个很大的数据需要传输、搬运时，你不需要等待所有数据都传输完成才开始下一步工作。

* 存储需要占用大量内存的数据。Buffer对象占用的内存空间是不计算在nodejs进程内存空间限制上的，所以可以用来存储大对象，但是对象的大小还是有限制的。



## Buffer的使用

### 创建Buffer的方式

分为手动和自动创建。在使用了stream流的相关操作的时候会自动隐式地创建缓存区。下面介绍一下手动的方法。

#### 1. new方式

`new Buffer()`

使用数组元素为utf8编码的字符串创建一个Buffer实例。现在标记为废弃状态，已经不推荐使用了。

```js
const buf = new Buffer([0x62, 0x63, 0x64);
// 输出：<Buffer 62 63 64>
```

####2. 通过指定长度创建

`Buffer.alloc(length,[, fill[, encoding]])`和`Buffer.alloceUnsafe(length,[, fill[, encoding])`。

方法返回一个Buffer实例，如果没有设置fill，默认为0。

区别：alloc的特点是安全，但是速度慢，alloceUnsafe的特点是速度快，但是不安全，可能会包含敏感数据。

```javascript
// 创建一个长度为5且用0填充的 Buffer。
const buf1 = Buffer.alloc(5);
// 输出：<Buffer 00 00 00 00 00>

// 创建一个长度为5且用a填充的 Buffer。
const buf2 = Buffer.alloc(10, 'a');
// 输出：<Buffer 61 61 61 61 61>
```

#### 3. 根据内容直接创建

`Buffer.from(array)`

`Buffer.from(arrayBuffer[, byteOffset [, length]])`

`Buffer.from(buffer)`

`Buffer.from(string[, encoding])`

方法返回一个Buffer实例。

```javascript
const buf = Buffer.from([0x62, 0x75, 0x66, 0x66, 0x65, 0x72]);
console.log(buf.toString());
// 输出：buffer

// 默认编码方式是utf8
const buf = Buffer.from('buffer');
console.log(buf.toString());
// 输出：buffer
console.log(buf.toString('hex'));
// 输出：627566666572

// 创建新的Buffer实例，并将buffer的数据拷贝到新的实例
const buf1 = Buffer.from('buffer');
const buf2 = Buffer.from(buff1);
console.log(buf1.toString());
// 输出：buffer
console.log(buf2.toString());
// 输出：buffer
```

### 缓存区读取数据

#### Buffer实例转字符串

`buf.toString([encoding[, start[, end]]])`

方法返回字符串

参数

* encoding：使用的编码。
* start：指定开始读取的索引位置，默认为0
* end：结束位置，默认为结束位置

```javascript
const buf = Buffer.from([65, 66, 67, 68, 69]);
console.log(buf.toString('ascii', 0, 1));
console.log(buf.toString('utf8'));
```

#### Buffer实例转JSON对象

`buf.toJSON()`

方法返回JSON对象

```javascript
const buf = Buffer.from([65, 66, 67, 68, 69]);
console.log(buf.toJSON());    
// 输出：{ type: 'Buffer', data: [ 101, 102, 103, 104, 105 ] }
```

### 缓存区写入数据

`buf.write(string[, offset[, length]][, encoding])`

```javascript
const buf = Buffer.alloc(4);
buf.write('a');
console.log(buf.toString());
// 输出：a
buf.write('bc');
console.log(buf.toString());
// 输出：bc
buf.write('defgh');
console.log(buf.toString());
// 输出：defg
```

### 操作缓存区

#### 拷贝缓存区

`buf.copy(targetBuffer[, targetStart[, sourceStart[, sourceEnd]]])`

```javascript
const buf1 = Buffer.from('abc');
const buf2 = Buffer.from('xyz');
buf2.copy(buf1, 1);
console.log(buf1.toString());
// 输出：axy

const buf1 = Buffer.from('abcdef');
const buf2 = Buffer.from('xyz');
buf2.copy(buf1, 1);
console.log(buf1.toString());
// 输出：axyzef
```

#### 比较缓存区

`buf.compare(otherBuf)`

方法返回一个数字。有三种结果。

* 小于0。buf在otherBuf之前。
* 等于0。buf在otherBuf相同。
* 大于0。buf在otherBuf之后。

```javascript
const buf1 = Buffer.from('abcdef');
const buf2 = Buffer.from('abc');
console.log(buf1.compare(buf2))
// 输出：1
```

`buf.equals(otherBuf)`

方法判断两个Buffer实例存储的数据是否相等。如果是，返回true，否则返回false。

```javascript
const buf1 = Buffer.from('abc');
const buf2 = Buffer.from('abc');

console.log(buf1.equals(buf2));
// 输出：1
console.log(buf1.compare(buf2));
// 输出：1
```

#### 裁剪缓存区

`buf.slice([start[, end]])`

方法返回一个新的缓存区，新的buffer与原buffer指向同一块内存。

```javascript
const buf1 = Buffer.from('abcdef');
const buf2 = buf1.slice(0, 2);
console.log(buf2.toString());
// 输出：ab

const buf1 = Buffer.from('abcdef');
const buf2 = buf1.slice(0);
console.log(buf1.compare(buf2));
// 输出：0
```

其它操作

`Buffer.concat(list[, totalLength])`：buffer连接

`buf.includes(value[, byteOffset][, encoding])`：查找，返回是true还是false

`buf.indexOf(value[, byteOffset][, encoding])`：查找，返回索引值

`buf.values() ` `buf.keys()` `buf.entries()`：遍历



字符集就是定义数字所代表的字符的一个规则表，同样定义了怎样用二进制存储和表示。那么用多少位来表示一个数字，这个就叫字符编码。有一种字符编码叫做UTF-8。它规定了，字符应该以字节为单位来表示。一个字节是8位。所以8个1和0组成的序列，应该用二进制来存储和表示任意一个字符。