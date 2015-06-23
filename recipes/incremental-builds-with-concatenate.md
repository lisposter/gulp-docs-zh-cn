# 增量编译打包，包括处理整所涉及的所有文件

在做增量编译打包的时候，有一个比较麻烦的事情，那就是你常常希望操作的是 _所有_ 处理过的文件，而不仅仅是单个的文件。举个例子，你想要只对更改的文件做代码 lint 操作，以及一些模块封装的操作，然后将他们与其他已经 lint 过的，以及已经进行过模块封装的文件合并到一起。如果不用到临时文件的话，这将会非常困难。

使用 [gulp-cached](https://github.com/wearefractal/gulp-cached) 以及 [gulp-remember](https://github.com/ahaurw01/gulp-remember) 来解决这个问题。

```js
var gulp = require('gulp');
var header = require('gulp-header');
var footer = require('gulp-footer');
var concat = require('gulp-concat');
var jshint = require('gulp-jshint');
var cached = require('gulp-cached');
var remember = require('gulp-remember');

var scriptsGlob = 'src/**/*.js';

gulp.task('scripts', function() {
  return gulp.src(scriptsGlob)
      .pipe(cached('scripts'))        // 只传递更改过的文件
      .pipe(jshint())                 // 对这些更改过的文件做一些特殊的处理...
      .pipe(header('(function () {')) // 比如 jshinting ^^^
      .pipe(footer('})();'))          // 增加一些类似模块封装的东西
      .pipe(remember('scripts'))      // 把所有的文件放回 stream
      .pipe(concat('app.js'))         // 做一些需要所有文件的操作
      .pipe(gulp.dest('public/'));
});

gulp.task('watch', function () {
  var watcher = gulp.watch(scriptsGlob, ['scripts']); // 监视与 scripts 任务中同样的文件
  watcher.on('change', function (event) {
    if (event.type === 'deleted') {                   // 如果一个文件被删除了，则将其忘记
      delete cached.caches.scripts[event.path];       // gulp-cached 的删除 api
      remember.forget('scripts', event.path);         // gulp-remember 的删除 api
    }
  });
});
```
