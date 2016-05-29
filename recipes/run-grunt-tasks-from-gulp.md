# 在 Gulp 中执行 Grunt 任务

在 Gulp 中运行 Grunt 任务或者 Grunt 插件是可能的。这在从 Grunt 迁移到 Gulp 阶段或者你对某个 Grunt 插件有需求的时候会非常有用。按照如下描述的方法，不需要引入 Grunt 命令行工具或者 Gruntfile。

**这个方法需要 Grunt 版本 >=1.0.0**

非常简单的 `gulpfile.js` 示例：

```js
// npm install gulp grunt grunt-contrib-copy --save-dev

var gulp = require('gulp');
var grunt = require('grunt');

grunt.initConfig({
    copy: {
        main: {
            src: 'src/*',
            dest: 'dest/'
        }
    }
});
grunt.loadNpmTasks('grunt-contrib-copy');

gulp.task('copy', function (done) {
    grunt.tasks(
        ['copy:main'],    // 你可以在这个数组中加入更多的 Grunt 任务
        {gruntfile: false}, // 不查找 Gruntfile - 因为并没有. :-)
        function () {done();}
    );
});

```

现在开始执行任务：
`gulp copy`

通过上述方式，grunt 任务会被注册到 gulp 的任务系统中。**需要注意的是，grunt 任务通常是阻塞的（不像 gulp），因此在 grunt 任务执行完成之前，没有任何其他的任务可以执行（甚至 gulp 任务也不可以）**。


### A few words on alternatives

有一个 *gulp友好* 的 node 模块 `gulp-grunt` [可以使用](https://www.npmjs.org/package/gulp-grunt) ，它使用了一个不同的方案。它会创建一些子进程来执行 grunt 任务，这意味着它有一些限制：

* 到现在为止，使用 `gulp-grunt` 还不能把类似命令行参数或者其他的选项参数传递到 grunt 任务中
* 所有的 grunt 任务都需要被定义到一个独立的 Gruntfile
* 你需要安装 Grunt 命令行
* 一些 grunt 任务的输出可能会有奇怪的格式(.i.e. color coding).
