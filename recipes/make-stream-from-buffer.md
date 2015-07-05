# 将 buffer 变为 stream (内存中的内容)

有时候，你会需要这样一个 stream，它们的内容保存在一个变量中，而不是在一个实际的文件中。换言之，怎么不使用 `gulp.src()` 而创建一个 'gulp' stream。

我们来举一个例子，我们拥有一个包含 js 库文件的目录，以及一个包含一些模块的不同版本文件的目录。编译的目标是为每个版本创建一个 js 文件，其中包含所有库文件以及相应版本的模块文件拼接后的结果。

逻辑上我们将把这个拆分为如下步骤：

* 载入库文件
* 拼接库文件的内容
* 载入不同版本的文件
* 对于每个版本的文件，将其和库文件的内容拼接
* 对于每个版本的文件，将结果输出到一个文件

想象如下的文件结构：

```sh
├── libs
│   ├── lib1.js
│   └── lib2.js
└── versions
    ├── version.1.js
    └── version.2.js
```

你应该要得到这样的结果：

```sh
└── output
    ├── version.1.complete.js # lib1.js + lib2.js + version.1.js
    └── version.2.complete.js # lib1.js + lib2.js + version.2.js
```

一个简单的模块化处理方式将会像下面这样：

```js
var gulp = require('gulp');
var runSequence = require('run-sequence');
var source = require('vinyl-source-stream');
var vinylBuffer = require('vinyl-buffer');
var tap = require('gulp-tap');
var concat = require('gulp-concat');
var size = require('gulp-size');
var path = require('path');
var es = require('event-stream');

var memory = {}; // 我们会将 assets 保存到内存中

// 载入内存中文件内容的任务
gulp.task('load-lib-files', function() {
  // 从磁盘中读取库文件
  return gulp.src('src/libs/*.js')
  // 将所有库文件拼接到一起
  .pipe(concat('libs.concat.js'))
  // 接入 stream 来获取每个文件的数据
  .pipe(tap(function(file) {
    // 保存文件的内容到内存
    memory[path.basename(file.path)] = file.contents.toString();
  }));
});

gulp.task('load-versions', function() {
  memory.versions = {};
  // 从磁盘中读取文件
  return gulp.src('src/versions/version.*.js')
  // 接入 stream 来获取每个文件的数据
  .pipe( tap(function(file) {
    // 在 assets 中保存文件的内容
    memory.versions[path.basename(file.path)] = file.contents.toString();
  }));
});

gulp.task('write-versions', function() {
  // 我们将不容版本的文件的名字保存到一个数组中
  var availableVersions = Object.keys(memory.versions);
  // 我们创建一个数组来保存所有的 stream 的 promise
  var streams = [];

  availableVersions.forEach(function(v) {
    // 以一个假文件名创建一个新的 stream
    var stream = source('final.' + v);
    // 从拼接后的文件中读取数据
    var fileContents = memory['libs.concat.js'] +
      // 增加版本文件的数据
      '\n' + memory.versions[v];

    streams.push(stream);

    // 将文件的内容写入 stream
    stream.write(fileContents);

    process.nextTick(function() {
      // 在下一次处理循环中结束 stream
      stream.end();
    });

    stream
    // 转换原始数据到 stream 中去，到一个 vinyl 对象/文件
    .pipe(vinylBuffer())
    //.pipe(tap(function(file) { /* 这里可以做一些对文件内容的处理操作 */ }))
    .pipe(gulp.dest('output'));
  });

  return es.merge.apply(this, streams);
});

//============================================ 我们的主任务
gulp.task('default', function(taskDone) {
  runSequence(
    ['load-lib-files', 'load-versions'],  // 并行载入文件
    'write-versions',  // 一旦所有资源进入内存便可以做写入操作了
    taskDone           // 完成
  );
});

//============================================ 我们的监控任务
// 只在运行完 'default' 任务后运行，
// 这样所有的资源都已经在内存中了
gulp.task('watch', ['default'], function() {
  gulp.watch('./src/libs/*.js', function() {
    runSequence(
      'load-lib-files',  // 我们只需要载入更改过的文件
      'write-versions'
    );
  });

  gulp.watch('./src/versions/*.js', function() {
    runSequence(
      'load-versions',  // 我们只需要载入更改过的文件
      'write-versions'
    );
  });
});
```
