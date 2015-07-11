# 通过 stream 工厂来共享 stream

如果你在多个任务中使用了相同的插件，你可能发现你很想把这些东西以 DRY 的原则去处理。这个方法可以创建一些工厂来把你经常使用的 stream 链分离出来。

我们将使用 [lazypipe](https://github.com/OverZealous/lazypipe) 来完成这件事。

这是我们的例子：

```js
var gulp = require('gulp');
var uglify = require('gulp-uglify');
var coffee = require('gulp-coffee');
var jshint = require('gulp-jshint');
var stylish = require('jshint-stylish');

gulp.task('bootstrap', function() {
  return gulp.src('bootstrap/js/*.js')
    .pipe(jshint())
    .pipe(jshint.reporter(stylish))
    .pipe(uglify())
    .pipe(gulp.dest('public/bootstrap'));
});

gulp.task('coffee', function() {
  return gulp.src('lib/js/*.coffee')
    .pipe(coffee())
    .pipe(jshint())
    .pipe(jshint.reporter(stylish))
    .pipe(uglify())
    .pipe(gulp.dest('public/js'));
});
```

然后，使用了 lazypipe 之后，将会是这样：

```js
var gulp = require('gulp');
var uglify = require('gulp-uglify');
var coffee = require('gulp-coffee');
var jshint = require('gulp-jshint');
var stylish = require('jshint-stylish');
var lazypipe = require('lazypipe');

// 赋给 lazypipe
var jsTransform = lazypipe()
  .pipe(jshint)
  .pipe(jshint.reporter, stylish)
  .pipe(uglify);

gulp.task('bootstrap', function() {
  return gulp.src('bootstrap/js/*.js')
    .pipe(jsTransform())
    .pipe(gulp.dest('public/bootstrap'));
});

gulp.task('coffee', function() {
  return gulp.src('lib/js/*.coffee')
    .pipe(coffee())
    .pipe(jsTransform())
    .pipe(gulp.dest('public/js'));
});
```

你可以看到，我们把多个任务中都在使用的 JavaScript 管道（JSHint + Uglify）分离到了一个工厂。工厂可以在任意多的任务中重用。你也可以嵌套这些工厂，或者把它们连接起来，已达到更好的效果。分离出每个共享的管道，也可以让你能够集中地管理，当你的工作流程更改后，你只需要修改一个地方即可。
