# 整合 streams 来处理错误

默认情况下，在 stream 中发生一个错误的话，它会被直接抛出，除非已经有一个时间监听器监听着 `error` 时间。 这在处理一个比较长的管道操作的时候会显得比较棘手。

通过使用 [stream-combiner2](https://github.com/substack/stream-combiner2)，你可以将一系列的 stream 合并成一个，这意味着，你只需要在你的代码中一个地方添加监听器监听 `error` 时间就可以了。

这里是一个在 gulpfile 中使用它的例子：

```js
var combiner = require('stream-combiner2');
var uglify = require('gulp-uglify');
var gulp = require('gulp');

gulp.task('test', function() {
  var combined = combiner.obj([
    gulp.src('bootstrap/js/*.js'),
    uglify(),
    gulp.dest('public/bootstrap')
  ]);

  // 任何在上面的 stream 中发生的错误，都不会抛出，
  // 而是会被监听器捕获
  combined.on('error', console.error.bind(console));

  return combined;
});
```
