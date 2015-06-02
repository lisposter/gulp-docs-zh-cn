# 改变版本号以及创建一个 git tag

如果你的项目遵循语义化版本，那么，把那些发布新版本的时候需要做的事情通过自动化的手段去完成将会是个很不错的主意。
下面有一个简单的范例展示了如何改变项目的版本号，将更新提交到 git，以及创建一个 tag。

``` javascript

var gulp = require('gulp');
var runSequence = require('run-sequence');
var bump = require('gulp-bump');
var gutil = require('gulp-util');
var git = require('gulp-git');
var fs = require('fs');

gulp.task('bump-version', function () {
// 注意：这里我硬编码了更新类型为 'patch'，但是更好的做法是用
//      minimist (https://www.npmjs.com/package/minimist) 通过检测一个命令行参数来判断你正在做的更新是
//      一个 'major'， 'minor' 还是一个 'patch'。
  return gulp.src(['./bower.json', './package.json'])
    .pipe(bump({type: "patch"}).on('error', gutil.log))
    .pipe(gulp.dest('./'));
});

gulp.task('commit-changes', function () {
  return gulp.src('.')
    .pipe(git.commit('[Prerelease] Bumped version number', {args: '-a'}));
});

gulp.task('push-changes', function (cb) {
  git.push('origin', 'master', cb);
});

gulp.task('create-new-tag', function (cb) {
  var version = getPackageJsonVersion();
  git.tag(version, 'Created Tag for version: ' + version, function (error) {
    if (error) {
      return cb(error);
    }
    git.push('origin', 'master', {args: '--tags'}, cb);
  });

  function getPackageJsonVersion () {
    // 这里我们直接解析 json 文件而不是使用 require，这是因为 require 会缓存多次调用，这会导致版本号不会被更新掉
    return JSON.parse(fs.readFileSync('./package.json', 'utf8')).version;
  };
});

gulp.task('release', function (callback) {
  runSequence(
    'bump-version',
    'commit-changes',
    'push-changes',
    'create-new-tag',
    function (error) {
      if (error) {
        console.log(error.message);
      } else {
        console.log('RELEASE FINISHED SUCCESSFULLY');
      }
      callback(error);
    });
});

```
