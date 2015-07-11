# 拥有实时刷新（live-reloading）和 CSS 注入的服务器

使用 [BrowserSync](http://browsersync.io) 和 gulp，你可以轻松地创建一个开发服务器，然后同一个 WiFi 中的任何设备都可以方便地访问到。BrowserSync 同时集成了 live-reload 所以不需要另外做配置了。

首先安装模块：

```sh
$ npm install --save-dev browser-sync
```

然后，考虑拥有如下的目录结构...

```
gulpfile.js
app/
  styles/
    main.css
  scripts/
    main.js
  index.html
```

... 通过如下的 `gulpfile.js`，你可以轻松地将 `app` 目录中的文件加到服务器中，并且所有的浏览器都会在文件发生改变之后自动刷新：

```js
var gulp = require('gulp');
var browserSync = require('browser-sync');
var reload = browserSync.reload;

// 监视文件改动并重新载入
gulp.task('serve', function() {
  browserSync({
    server: {
      baseDir: 'app'
    }
  });

  gulp.watch(['*.html', 'styles/**/*.css', 'scripts/**/*.js'], {cwd: 'app'}, reload);
});

```

在 `index.html` 中引入 CSS：

```html
<html>
  <head>
    ...
    <link rel="stylesheet" href="styles/main.css">
    ...

```

通过如下命令启动服务，并且打开一个浏览器，访问默认的 URL (http://localhost:3000)：

```bash
gulp serve
```


## + CSS 预处理器

一个常见的使用案例是当 CSS 文件文件预处理之后重载它们。以 sass 为例，这便是你如何指示浏览器无需刷新整个页面而只是重载 CSS。

考虑有如下的文件目录结构...

```
gulpfile.js
app/
  scss/
    main.scss
  scripts/
    main.js
  index.html
```
... 通过如下的 `gulpfile.js`，你可以轻松地监视 `scss` 目录中的文件，并且所有的浏览器都会在文件发生改变之后自动刷新：

```js
var gulp = require('gulp');
var sass = require('gulp-ruby-sass');
var browserSync = require('browser-sync');
var reload = browserSync.reload;

gulp.task('sass', function() {
  return sass('scss/styles.scss')
    .pipe(gulp.dest('app/css'))
    .pipe(reload({ stream:true }));
});

// 监视 Sass 文件的改动，如果发生变更，运行 'sass' 任务，并且重载文件
gulp.task('serve', ['sass'], function() {
  browserSync({
    server: {
      baseDir: 'app'
    }
  });

  gulp.watch('app/scss/*.scss', ['sass']);
});
```

在 `index.html` 文件中引入预处理后的 CSS 文件：

```html
<html>
  <head>
    ...
    <link rel="stylesheet" href="css/main.css">
    ...

```

通过如下命令启动服务，并且打开一个浏览器，访问默认的 URL (http://localhost:3000)：

```bash
gulp serve
```

## 附注：

- 实时重载（Live reload），CSS 注入以及同步滚动可以在 [BrowserStack](http://www.browserstack.com/) 虚拟机里无缝执行。
- 设置 `tunnel: true` 来使用一个公开的 URL 来访问你本地的站点 (支持所有 BrowserSync 功能)。
