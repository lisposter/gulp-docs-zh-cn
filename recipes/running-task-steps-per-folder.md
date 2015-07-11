# 每个文件夹生成单独一个文件

如果你有一整套的文件目录，并且希望执行相应的一套任务，比如...

```
/scripts
/scripts/jquery/*.js
/scripts/angularjs/*.js
```

...然后希望完成如下的结果h...

```
/scripts
/scripts/jquery.min.js
/scripts/angularjs.min.js
```

...你将会需要像下面所示的东西...

``` javascript
var fs = require('fs');
var path = require('path');
var merge = require('merge-stream');
var gulp = require('gulp');
var concat = require('gulp-concat');
var rename = require('gulp-rename');
var uglify = require('gulp-uglify');

var scriptsPath = 'src/scripts';

function getFolders(dir) {
    return fs.readdirSync(dir)
      .filter(function(file) {
        return fs.statSync(path.join(dir, file)).isDirectory();
      });
}

gulp.task('scripts', function() {
   var folders = getFolders(scriptsPath);

   var tasks = folders.map(function(folder) {
      // 拼接成 foldername.js
      // 写入输出
      // 压缩
      // 重命名为 folder.min.js
      // 再一次写入输出
      return gulp.src(path.join(scriptsPath, folder, '/*.js'))
        .pipe(concat(folder + '.js'))
        .pipe(gulp.dest(scriptsPath))
        .pipe(uglify())
        .pipe(rename(folder + '.min.js'))
        .pipe(gulp.dest(scriptsPath));
   });

   return merge(tasks);
});
```

注：

- `folders.map` - 在每一个文件夹中分别执行一次函数，并且返回异步 stream
- `merge` - 汇总 stream，并且在所有的 stream 都完成后完成
