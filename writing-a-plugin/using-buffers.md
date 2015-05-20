# 使用 buffer

> 这里有些关于如何创建一个使用 buffer 来处理的插件的有用信息。

[编写插件](README.md) > 使用 buffer

## 使用 buffer

如果你的插件依赖着一个基于 buffer 处理的库，你可能会选择让你的插件以 buffer 的形式来处理 file.contents。让我们来实现一个在文件头部插入额外文本的插件：

```js
var through = require('through2');
var gutil = require('gulp-util');
var PluginError = gutil.PluginError;

// 常量
const PLUGIN_NAME = 'gulp-prefixer';

// 插件级别的函数（处理文件）
function gulpPrefixer(prefixText) {
  if (!prefixText) {
    throw new PluginError(PLUGIN_NAME, 'Missing prefix text!');
  }

  prefixText = new Buffer(prefixText); // 提前分配

  // 创建一个 stream 通道，以让每个文件通过
  var stream = through.obj(function(file, enc, cb) {
    if (file.isStream()) {
      this.emit('error', new PluginError(PLUGIN_NAME, 'Streams are not supported!'));
      return cb();
    }

    if (file.isBuffer()) {
      file.contents = Buffer.concat([prefixText, file.contents]);
    }

    // 确保文件进入下一个 gulp 插件
    this.push(file);

    // 告诉 stream 引擎，我们已经处理完了这个文件
    cb();
  });

  // 返回文件 stream
  return stream;
};

// 导出插件主函数
module.exports = gulpPrefixer;
```

上述的插件可以这样使用：

```js
var gulp = require('gulp');
var gulpPrefixer = require('gulp-prefixer');

gulp.src('files/**/*.js')
  .pipe(gulpPrefixer('prepended string'))
  .pipe(gulp.dest('modified-files'));
```

## 处理 stream

不幸的是，当 gulp.src 如果是以 stream 的形式，而不是 buffer，那么，上面的插件就会报错。如果可以，你也应该让他支持 stream 形式。请查看[使用 Stream 处理](dealing-with-streams.md) 获取更多信息。

## 一些基于 buffer 的插件

* [gulp-coffee](https://github.com/wearefractal/gulp-coffee)
* [gulp-svgmin](https://github.com/ben-eb/gulp-svgmin)
* [gulp-marked](https://github.com/lmtm/gulp-marked)
* [gulp-svg2ttf](https://github.com/nfroidure/gulp-svg2ttf)
