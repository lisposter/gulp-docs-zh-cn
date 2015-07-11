# 串行方式运行任务，亦即，任务依赖

默认情况下，任务会以最大的并发数同时运行 -- 也就是说，它会不做任何等待地将所有的任务同时开起来。如果你希望创建一个有特定顺序的串行的任务链，你需要做两件事：

- 给它一个提示，用以告知任务在什么时候完成，
- 而后，再给一个提示，用以告知某任务需要依赖另一个任务的完成。

举个例子，我们假设你有两个任务，"one" 和 "two"，并且你明确的希望他们就以这样的顺序运行：

1. 在任务 "one" 中，你添加的一个提示，来告知何时它会完成。你可以传入一个回调函数，然后在完成后执行回调函数，也可以通过返回一个 promise 或者 stream 来让引擎等待它们分别地被解决掉。

2. 在任务 "two" 中，你添加一个提示，来告知引擎它需要依赖第一个任务的完成。

因此，这个例子将会是像这样：

```js
var gulp = require('gulp');

// 传入一个回调函数，因此引擎可以知道何时它会被完成
gulp.task('one', function(cb) {
    // 做一些事 -- 异步的或者其他任何的事
    cb(err); // 如果 err 不是 null 和 undefined，流程会被结束掉，'two' 不会被执行
});

// 标注一个依赖，依赖的任务必须在这个任务开始之前被完成
gulp.task('two', ['one'], function() {
    // 现在任务 'one' 已经完成了
});

gulp.task('default', ['one', 'two']);
// 也可以这么写：gulp.task('default', ['two']);
```

另一个例子，通过返回一个 stream 来取代使用回调函数的方法：

```js
var gulp = require('gulp');
var del = require('del'); // rm -rf

gulp.task('clean', function(cb) {
    del(['output'], cb);
});

gulp.task('templates', ['clean'], function() {
    var stream = gulp.src(['src/templates/*.hbs'])
        // 执行拼接，压缩，等。
        .pipe(gulp.dest('output/templates/'));
    return stream; // 返回一个 stream 来表示它已经被完成

});

gulp.task('styles', ['clean'], function() {
    var stream = gulp.src(['src/styles/app.less'])
        // 执行一些代码检查，压缩，等
        .pipe(gulp.dest('output/css/app.css'));
    return stream;
});

gulp.task('build', ['templates', 'styles']);

// templates 和 styles 将会并行处理
// clean 将会保证在任一个任务开始之前完成
// clean 并不会被执行两次，尽管它被作为依赖调用了两次

gulp.task('default', ['build']);
```
