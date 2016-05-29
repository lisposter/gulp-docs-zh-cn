# 编写插件

如果你打算自己写一个 Gulp 插件，为了节约你的时间，你可以先完整地阅读下这个文档。

* [导览](guidelines.md) (a MUST read)
* [使用 buffers](using-buffers.md)
* [使用 streams 来处理](dealing-with-streams.md)
* [测试](testing.md)

## 它要做什么？

### 流式处理文件对象（Streaming file objects）

gulp 插件总是返回一个 [object mode](http://nodejs.org/api/stream.html#stream_object_mode) 形式的 stream 来做这些事情：

1. 接收 [vinyl File 对象](http://github.com/gulpjs/vinyl)
2. 输出 [vinyl File 对象](http://github.com/gulpjs/vinyl) (通过 `transform.push()` 以及/或者插件的回调函数)

这通常被叫做 [transform streams](http://nodejs.org/api/stream.html#stream_class_stream_transform_1) (有时候也叫做 through streams)。
transform streams 是双工可读又可写的，它会对传给它的对象做一些转换的操作。

所有的插件本质上都可以归结成如此：

```js
var Transform = require('transform');

module.exports = function() {
  // 加 Monkey patch 或者创建你自己的子类
  // 实现 `_transform()` 和可选的 `_flush()`
  var transformStream = new Transform({objectMode: true});
  /**
   * @param {Buffer|string} 文件
   * @param {string=} 编码 - 如果文件包含 Buffer 则忽略
   * @param {function(Error, object)} 回调 - 当对传递进来的 chunk 处理完成后执行（可选的一个错误参数和数据）
   */
  transformStream._transform = function(file, encoding, callback) {
    var error = null,
        output = doSomethingWithTheFile(file);
    callback(error, output);
  };

  return transformStream;
};
```

很多插件使用 [through2](https://github.com/rvagg/through2/) 模块来简化代码：

```js
var through = require('through2');    // npm install --save through2

module.exports = function() {
  return through.obj(function(file, encoding, callback) {
    callback(null, doSomethingWithTheFile(file));
  });
};
```

`through()` 所返回的 stream (以及你 transform 函数中的 `this`) 都是 [Transform](https://github.com/iojs/readable-stream/blob/master/lib/_stream_transform.js) 类的实例，它继承自 [Duplex](https://github.com/iojs/readable-stream/blob/master/lib/_stream_duplex.js),
[Readable](https://github.com/iojs/readable-stream/blob/master/lib/_stream_readable.js)
(并寄生自 Writable) 最终继承 [Stream](https://nodejs.org/api/stream.html).
如果你需要处理额外的参数，你可以直接调用 `through()` 函数：

```js
  return through({objectMode: true /* other options... */}, function(file, encoding, callback) { ...
```

支持的参数包括:

* highWaterMark (defaults to 16)
* defaultEncoding (defaults to 'utf8')
* encoding - 'utf8', 'base64', 'utf16le', 'ucs2' etc.
    指定后, 一个 [StringDecoder](https://github.com/rvagg/string_decoder/blob/master/index.js) `decoder` 会被加到这个 stream。
* readable {boolean}
* writable {boolean}
* allowHalfOpen {boolean} 如果为 false，那么 stream 会在 sriteable 结束的时候自动结束 readable，反之亦然。

### 修改文内容

传递给 `through.obj()` 的参数是一个 [_transform](https://nodejs.org/api/stream.html#stream_transform_transform_chunk_encoding_callback)
函数，它将对输入的 `file` 做一些操作，你可能也将提供一个可选 [_flush](https://nodejs.org/api/stream.html#stream_transform_flush_callback)
函数，如果你需要在结束时候发送更多的数据的话。

从 transform 函数内部调用 `this.push(file)` 0 次或者多次来传递 transformed/cloned 的文件.
如果你把所有输出提供给 `callback()` 那么，你不需要调用 `this.push(file)`。

只在当前文件(stream/buffer) 被完全的消费后才调用 `callback` 函数
如果发生了一个错误，则将它以第一个参数的形式传递给回调函数，否则，将第一个参数设置为 null。
如果你已经将所有输出的内容传递给了 `this.push()` 那么，你可以省略回调函数的第二个参数。

通常，一个 gulp 插件会更新 `file.contents` 然后，选择以下一个：

 - call `callback(null, file)`
 _或_
 - 调用 `this.push(file)`

如果一个插件为一个输入的文件创建了多个输出文件，它会多次调用 `this.push()` - 如:

```js
module.exports = function() {
  /**
   * @this {Transform}
   */
  var transform = function(file, encoding, callback) {
    var files = splitFile(file);
    this.push(files[0]);
    this.push(files[1]);
    callback();
  };

  return through.obj(transform);
};
```

[gulp-unzip](https://github.com/suisho/gulp-unzip/blob/master/index.js) 插件提供了一个很好的例子，来演示如何多次调用 `push()`。它同时也使用了一个 chunk transform stream 和 _在_ Vinyl transform 函数中. `_flush()` 函数

Vinyl 文件可以通过三种不同形式来访问文件内容：

- [Streams](dealing-with-streams.md)
- [Buffers](using-buffers.md)
- 空 (null) - 对于删除, 清理, 等操作来说，会很有用，因为这时候内容是不需要处理的。

以下是一个简单的例子来展示如何检测和处理每一种形式，更多的细节请参考上面的链接。

```js
var PluginError = require('gulp-util').PluginError;

// 常量声明
var PLUGIN_NAME = 'gulp-example';

module.exports = function() {
    return through.obj(function(file, encoding, callback) {
        if (file.isNull()) {
            // 不做处理
            return callback(null, file);
        }

        if (file.isStream()) {
            // file.contents 是一个 Stream - https://nodejs.org/api/stream.html
            this.emit('error', new PluginError(PLUGIN_NAME, 'Streams not supported!'));

            // 或者, 你可以这样处理:
            //file.contents = file.contents.pipe(...
            //return callback(null, file);
        } else if (file.isBuffer()) {
            // file.contents 是 Buffer - https://nodejs.org/api/buffer.html
            this.emit('error', new PluginError(PLUGIN_NAME, 'Buffers not supported!'));

            // 或者, 你可以这样处理：
            //file.contents = ...
            //return callback(null, file);
        }
    });
};
```

注意：当你查看其他的插件的代码时候（以及上面的例子），你会注意到， transform 函数会返回 callback：

```js
return callback(null, file);
```

...不要被困扰 - gulp 会忽略所有 transform 函数返回的结果，上面的例子只是以下代码的简写方式：

```js
if (someCondition) {
  callback(null, file);
  return;
}
// 进一步执行...
```

## 有用的资源

* [File object](https://github.com/gulpjs/gulp-util/#new-fileobj)
* [PluginError](https://github.com/gulpjs/gulp-util#new-pluginerrorpluginname-message-options)
* [event-stream](https://github.com/dominictarr/event-stream)
* [BufferStream](https://github.com/nfroidure/BufferStream)
* [gulp-util](https://github.com/gulpjs/gulp-util)


## 插件范例

* [sindresorhus' gulp plugins](https://github.com/search?q=%40sindresorhus+gulp-)
* [contra's gulp plugins](https://github.com/search?q=%40contra+gulp-)
* [gulp-replace](https://github.com/lazd/gulp-replace)


## 关于 streams

如果你不熟悉 stream，你可以阅读这些来

* https://github.com/substack/stream-handbook (必读)
* http://nodejs.org/api/stream.html

其他的一些为 gulp 创建的和使用的，但又并非通过 stream 去处理的库，在 npm 上都会被打上 [gulpfriendly](https://npmjs.org/browse/keyword/gulpfriendly) 标签。
