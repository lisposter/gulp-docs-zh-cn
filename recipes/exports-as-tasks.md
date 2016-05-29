# 使用 ES2015 的 export 来写 tasks

使用 ES2015 模块语法，你可以使用 export 来写 task。

```js
import gulp from 'gulp';
import babel from 'gulp-babel';

// 命名的 task
export function build() {
  return gulp.src('src/*.js')
    .pipe(babel())
    .pipe(gulp.dest('lib'));
}

// 默认 task
export default function dev() {
  gulp.watch('src/*.js', ['build']);
}
```

这个写法 **无法** 在 gulp 3.x 所打包的 gulp-cli 上工作。你必须使用最新版来尝试这个功能。
