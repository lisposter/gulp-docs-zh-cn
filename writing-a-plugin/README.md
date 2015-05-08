# 编写插件

如果你打算自己写一个 Gulp 插件，为了节约你的时间，你可以先完整地阅读下这个文档。

* [导览](guidelines.md) (a MUST read)
* [使用 buffers](using-buffers.md)
* [使用 streams 来处理](dealing-with-streams.md)
* [测试](testing.md)

## 它要做什么？

### 流式处理文件对象（Streaming file objects）

gulp 插件总是返回一个 [object mode](http://nodejs.org/api/stream.html#stream_object_mode) 形式的 stream 来做这些事情：

1. 接收 [vinyl File 对象](http://github.com/wearefractal/vinyl)
2. 输出 [vinyl File 对象](http://github.com/wearefractal/vinyl)

这通常被叫做 [transform streams](http://nodejs.org/api/stream.html#stream_class_stream_transform_1) (有时候也叫做 through streams)。transform streams 是可读又可写的，它会对传给它的对象做一些转换的操作。

### 修改文内容

Vinyl 文件可以通过三种不同形式来访问文件内容：

- [Streams](dealing-with-streams.md)
- [Buffers](using-buffers.md)
- 空 (null) - 对于删除, 清理, 等操作来说，会很有用，因为这时候内容是不需要处理的。

## 有用的资源

* [File object](https://github.com/wearefractal/gulp-util/#new-fileobj)
* [PluginError](https://github.com/gulpjs/gulp-util#new-pluginerrorpluginname-message-options)
* [event-stream](https://github.com/dominictarr/event-stream)
* [BufferStream](https://github.com/nfroidure/BufferStream)
* [gulp-util](https://github.com/wearefractal/gulp-util)


## 插件范例

* [sindresorhus' gulp plugins](https://github.com/search?q=%40sindresorhus+gulp-)
* [Fractal's gulp plugins](https://github.com/search?q=%40wearefractal+gulp-)
* [gulp-replace](https://github.com/lazd/gulp-replace)


## 关于 streams

如果你不熟悉 stream，你可以阅读这些来

* https://github.com/substack/stream-handbook (必读)
* http://nodejs.org/api/stream.html

其他的一些为 gulp 创建的和使用的，但又并非通过 stream 去处理的库，在 npm 上都会被打上 [gulpfriendly](https://npmjs.org/browse/keyword/gulpfriendly) 标签。
