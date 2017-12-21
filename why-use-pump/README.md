# 为什么使用Pump?

当我们使用Node.js文件流提供的 `pipe` 时，错误不会随着 `pipe` 传递，所以原文件流不会随目的流关闭而关闭。
[`pump`][pump] 解决了这个问题并将错误在回调函数中传递给你。

## 一个常见的gulpfile例子

在通常的gulpfile中，我们只是简单的将Node.js的文件流返回，然后期待gulp会帮我们处理错误。

```javascript
// example of a common gulpfile
var gulp = require('gulp');
var uglify = require('gulp-uglify');

gulp.task('compress', function () {
  // returns a Node.js stream, but no handling of error messages
  return gulp.src('lib/*.js')
    .pipe(uglify())
    .pipe(gulp.dest('dist'));
});
```

![pipe error](pipe-error.png)

上述的JavaScript文件中有一个错误，但是抛出的错误信息显然不能提供太多帮助。
我们想要知道的是文件中的哪一行导致出错，所以为什么会显示这种无用的错误信息呢？

当有一个错误出现在流中，Node.js会触发 `error` 事件，但是如果没有对应的事件处理函数，Node.js并不会调用
[uncaught exception][uncaughtException] 来处理事件。默认的处理行为正如文档中所提到的:

> 默认的Node.js会将错误的堆栈信息打印出来然后终止程序的执行


## 处理错误

Since allowing the errors to make it to the uncaught exception handler isn’t
useful, we should handle the exceptions properly. Let’s give that a quick shot.

```javascript
var gulp = require('gulp');
var uglify = require('gulp-uglify');

gulp.task('compress', function () {
  return gulp.src('lib/*.js')
    .pipe(uglify())
    .pipe(gulp.dest('dist'))
    .on('error', function(err) {
      console.error('Error in compress task', err.toString());
    });
});
```

Unfortunately, Node.js stream’s `pipe` function doesn’t forward errors through
the chain, so this error handler only handles the errors given by
`gulp.dest`. Instead we need to handle errors for each stream.

```javascript
var gulp = require('gulp');
var uglify = require('gulp-uglify');

gulp.task('compress', function () {
  function createErrorHandler(name) {
    return function (err) {
      console.error('Error from ' + name + ' in compress task', err.toString());
    };
  }

  return gulp.src('lib/*.js')
    .on('error', createErrorHandler('gulp.src'))
    .pipe(uglify())
    .on('error', createErrorHandler('uglify'))
    .pipe(gulp.dest('dist'))
    .on('error', createErrorHandler('gulp.dest'));
});
```

This is a lot of complexity to add in each of your gulp tasks, and it’s easy to
forget to do it. In addition, it’s still not perfect, as it doesn’t properly
signal to gulp’s task system that the task has failed. We can fix this, and we
can handle the other pesky issues with error propogations with streams, but it’s
even more work!

## Using pump

The [`pump`][pump] module is a cheat code of sorts. It’s a wrapper around the
`pipe` functionality that handles these cases for you, so you can stop hacking
on your gulpfiles, and get back to hacking new features into your app.

```javascript
var gulp = require('gulp');
var uglify = require('gulp-uglify');
var pump = require('pump');

gulp.task('compress', function (cb) {
  pump([
      gulp.src('lib/*.js'),
      uglify(),
      gulp.dest('dist')
    ],
    cb
  );
});
```

The gulp task system provides a gulp task with a callback, which can signal
successful task completion (being called with no arguments), or a task failure
(being called with an Error argument). Fortunately, this is the exact same
format `pump` uses!

![pump error](pump-error.png)

现在我们终于能很清楚的看到错误是哪个插件产生的，这个错误到底是什么，并且从哪个文件中的哪一行而来。

[pump]: https://github.com/mafintosh/pump
[uncaughtException]: https://nodejs.org/api/process.html#process_event_uncaughtexception
