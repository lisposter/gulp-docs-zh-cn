# 使用 Stream 处理

> 极力推荐让缩写的插件支持 stream。这里有一些关于让插件支持 stream 的一些有用信息。

> 请确保使用处理错误的最佳实践，并且加入一行代码，使得 gulp 能在转换内容的期间在捕获到第一个错误时候正确报出错误。

[编写插件](README.md) > 编写以 stream 为基础的插件

## 使用 stream 处理

让我们来实现一个用于在文件头部插入一些文本的插件，这个插件支持 file.contents 所有可能的形式。

```js
var through = require('through2');
var gutil = require('gulp-util');
var PluginError = gutil.PluginError;

// 常量
const PLUGIN_NAME = 'gulp-prefixer';

function prefixStream(prefixText) {
  var stream = through();
  stream.write(prefixText);
  return stream;
}

// 插件级别函数 (处理文件)
function gulpPrefixer(prefixText) {
  if (!prefixText) {
    throw new PluginError(PLUGIN_NAME, 'Missing prefix text!');
  }

  prefixText = new Buffer(prefixText); // 预先分配

  // 创建一个让每个文件通过的 stream 通道
  var stream = through.obj(function(file, enc, cb) {
    if (file.isBuffer()) {
      this.emit('error', new PluginError(PLUGIN_NAME, 'Buffers not supported!'));
      return cb();
    }

    if (file.isStream()) {
      // 定义转换内容的 streamer
      var streamer = prefixStream(prefixText);
      // 从 streamer 中捕获错误，并发出一个 gulp的错误
      streamer.on('error', this.emit.bind(this, 'error'));
      // 开始转换
      file.contents = file.contents.pipe(streamer);
    }

    // 确保文件进去下一个插件
    this.push(file);
    // 告诉 stream 转换工作完成
    cb();
  });

  // 返回文件 stream
  return stream;
}

// 暴露（export）插件的主函数
module.exports = gulpPrefixer;
```

上面的插件可以像这样使用：

```js
var gulp = require('gulp');
var gulpPrefixer = require('gulp-prefixer');

gulp.src('files/**/*.js', { buffer: false })
  .pipe(gulpPrefixer('prepended string'))
  .pipe(gulp.dest('modified-files'));
```

## 一些使用 stream 的插件

* [gulp-svgicons2svgfont](https://github.com/nfroidure/gulp-svgiconstosvgfont)

