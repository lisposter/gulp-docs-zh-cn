# 在监控时处理文件删除这个事件

你可以通过监听来自 `gulp.watch` 的 `'change'` 事件来触发相应的监视器函数。

每一个 change 事件都有一个 `type` 事件。如果 `type` 是 `'deleted'`，你可以同时把你的目标文件夹中的对应文件也删除掉，可以是使用如下方法：

```js
'use strict';

var del = require('del');
var path = require('path');
var gulp = require('gulp');
var header = require('gulp-header');
var footer = require('gulp-footer');

gulp.task('scripts', function() {
  return gulp.src('src/**/*.js', {base: 'src'})
    .pipe(header('(function () {\r\n\t\'use strict\'\r\n'))
    .pipe(footer('\r\n})();'))
    .pipe(gulp.dest('build'));
});

gulp.task('watch', function () {
  var watcher = gulp.watch('src/**/*.js', ['scripts']);

  watcher.on('change', function (event) {
    if (event.type === 'deleted') {
      // 模拟在 scripts 任务中 gulp.src 所用的 {base: 'src'}
      var filePathFromSrc = path.relative(path.resolve('src'), event.path);

      // 拼接在 scripts 任务重 gulp.dest 所用的 'build' 绝对路径
      var destFilePath = path.resolve('build', filePathFromSrc);

      del.sync(destFilePath);
    }
  });
});
```
