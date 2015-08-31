
# 删除文件和文件夹

你也许会想要在编译文件之前删除一些文件。由于删除文件和文件内容并没有太大关系，所以，我们没必要去用一个 gulp 插件。最好的一个选择就是使用一个原生的 node 模块。

因为 [`del`](https://github.com/sindresorhus/del) 模块支持多个文件以及 [globbing](https://github.com/sindresorhus/multimatch#globbing-patterns)，因此，在这个例子中，我们将使用它来删除文件：

```sh
$ npm install --save-dev gulp del
```

假想有如下的文件结构：

```
.
├── dist
│   ├── report.csv
│   ├── desktop
│   └── mobile
│       ├── app.js
│       ├── deploy.json
│       └── index.html
└── src
```

在 gulpfile 中，我们希望在运行我们的编译任务之前，将 `mobile` 文件的内容先清理掉：

```js
var gulp = require('gulp');
var del = require('del');

gulp.task('clean:mobile', function () {
  return del([
    'dist/report.csv',
    // 这里我们使用一个通配模式来匹配 `mobile` 文件夹中的所有东西
    'dist/mobile/**/*',
    // 我们不希望删掉这个文件，所以我们取反这个匹配模式
    '!dist/mobile/deploy.json'
  ]);
});

gulp.task('default', ['clean:mobile']);
```


## 在管道中删除文件

你可能需要在管道中将一些处理过的文件删除掉。

我们使用 [vinyl-paths](https://github.com/sindresorhus/vinyl-paths) 模块来简单地获取 stream 中每个文件的路径，然后传给 `del` 方法。

```sh
$ npm install --save-dev gulp del vinyl-paths
```

假想有如下的文件结构：

```
.
├── tmp
│   ├── rainbow.js
│   └── unicorn.js
└── dist
```

```js
var gulp = require('gulp');
var stripDebug = require('gulp-strip-debug'); // 仅用于本例做演示
var del = require('del');
var vinylPaths = require('vinyl-paths');

gulp.task('clean:tmp', function () {
  return gulp.src('tmp/*')
    .pipe(stripDebug())
    .pipe(gulp.dest('dist'))
    .pipe(vinylPaths(del));
});

gulp.task('default', ['clean:tmp']);
```

只有在已经使用了其他的插件之后才需要这样做，否则，请直接使用 `gulp.src` 来代替。
