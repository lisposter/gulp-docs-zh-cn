# Browserify + Uglify2 和 sourcemaps

[Browserify](http://github.com/substack/node-browserify) 现在已经成为了一个不可或缺的重要工具了，然后要让它能完美的和 gulp 一起协作，还得需要做一些封装处理。先面便是一个使用 Browserify 并且增加一个完整的 sourcemap 来对应到单独的每个源文件。

同时请看: [组合 Streams 来处理错误](https://github.com/gulpjs/gulp/blob/master/docs/recipes/combining-streams-to-handle-errors.md) 范例来查看如何处理你的 stream 中 browserify 或者 uglify 的错误。

``` javascript
'use strict';

var browserify = require('browserify');
var gulp = require('gulp');
var source = require('vinyl-source-stream');
var buffer = require('vinyl-buffer');
var uglify = require('gulp-uglify');
var sourcemaps = require('gulp-sourcemaps');
var gutil = require('gulp-util');

gulp.task('javascript', function () {
  // 在一个基础的 task 中创建一个 browserify 实例
  var b = browserify({
    entries: './entry.js',
    debug: true
  });

  return b.bundle()
    .pipe(source('app.js'))
    .pipe(buffer())
    .pipe(sourcemaps.init({loadMaps: true}))
        // 在这里将转换任务加入管道
        .pipe(uglify())
        .on('error', gutil.log)
    .pipe(sourcemaps.write('./'))
    .pipe(gulp.dest('./dist/js/'));
});
```
