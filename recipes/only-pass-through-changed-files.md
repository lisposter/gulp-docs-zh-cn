# 仅仅传递更改过的文件

默认情况下，每次运行时候所有的文件都会传递并通过整个管道。通过使用 [gulp-changed](https://github.com/sindresorhus/gulp-changed) 可以只让更改过的文件传递过管道。这可以大大加快连续多次的运行。


```js
// npm install --save-dev gulp gulp-changed gulp-jscs gulp-uglify

var gulp = require('gulp');
var changed = require('gulp-changed');
var jscs = require('gulp-jscs');
var uglify = require('gulp-uglify');

// 我们在这里定义一些常量以供使用
var SRC = 'src/*.js';
var DEST = 'dist';

gulp.task('default', function() {
	return gulp.src(SRC)
		// `changed` 任务需要提前知道目标目录位置
		// 才能找出哪些文件是被修改过的
		.pipe(changed(DEST))
		// 只有被更改过的文件才会通过这里
		.pipe(jscs())
		.pipe(uglify())
		.pipe(gulp.dest(DEST));
});
```
