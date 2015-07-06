# 同时输出一个压缩过和一个未压缩版本的文件

同时输出压缩过的和未压缩版本的文件可以通过使用 `gulp-rename` 然后 pipe 到 `dest` 两次来实现 (一次是压缩之前的，一次是压缩后的)：

```js
'use strict';

var gulp = require('gulp');
var rename = require('gulp-rename');
var uglify = require('gulp-uglify');

var DEST = 'build/';

gulp.task('default', function() {
  return gulp.src('foo.js')
    // 这会输出一个未压缩过的版本
    .pipe(gulp.dest(DEST))
    // 这会输出一个压缩过的并且重命名未 foo.min.js 的文件
    .pipe(uglify())
    .pipe(rename({ extname: '.min.js' }))
    .pipe(gulp.dest(DEST));
});

```
