# 从命令行传递参数

```js
// npm install --save-dev gulp gulp-if gulp-uglify minimist

var gulp = require('gulp');
var gulpif = require('gulp-if');
var uglify = require('gulp-uglify');

var minimist = require('minimist');

var knownOptions = {
  string: 'env',
  default: { env: process.env.NODE_ENV || 'production' }
};

var options = minimist(process.argv.slice(2), knownOptions);

gulp.task('scripts', function() {
  return gulp.src('**/*.js')
    .pipe(gulpif(options.env === 'production', uglify())) // 仅在生产环境时候进行压缩
    .pipe(gulp.dest('dist'));
});
```

然后，通过如下命令运行 gulp：

```sh
$ gulp scripts --env development
```
