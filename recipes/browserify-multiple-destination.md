# Browserify + Globs (多个目标路径)

这个例子用来展示如何设置一个使用 browserify 打包多个输入并且有多个输出目的地的任务。

下边的 `js` 任务会打包所有在 `src/` 目录下所有的 `.js` 文件，然后输出每一个对应的文件到 `dest/` 目录。


```js
var gulp = require('gulp');
var browserify = require('browserify');
var gutil = require('gulp-util');
var tap = require('gulp-tap');
var buffer = require('gulp-buffer');
var sourcemaps = require('gulp-sourcemaps');
var uglify = require('gulp-uglify');

gulp.task('js', function () {

  return gulp.src('src/**/*.js', {read: false}) // 不需要读取文件内容，browserify 会处理这个问题

    // 使用 gulp-tap 转换文件内容
    .pipe(tap(function (file) {

      gutil.log('bundling ' + file.path);

      // 使用 browserify 的打包 stream 来替换文件
      file.contents = browserify(file.path, {debug: true}).bundle();

    }))

    // 转换 stram 内容为 buff 内容（因为 gulp-sourcemaps 不支持 stream 形式的内容）
    .pipe(buffer())

    // 载入并初始化 sourcemaps
    .pipe(sourcemaps.init({loadMaps: true}))

    .pipe(uglify())

    // 写入 sourcemaps
    .pipe(sourcemaps.write('./'))

    .pipe(gulp.dest('dest'));

});
```
