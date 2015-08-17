# 自动化发布工作流

如果你的项目遵循语义化版本规则，那么将发布版本需要做的一些事情自动化处理，将会是个很不错的主意。
下面有一个简单的秘籍，包含了更新版本好，提交（commit）更改到 git 以及创建一个 tag。

``` javascript

var gulp = require('gulp');
var runSequence = require('run-sequence');
var conventionalChangelog = require('conventional-changelog');
var conventionalGithubReleaser = require('conventional-github-releaser');
var bump = require('gulp-bump');
var gutil = require('gulp-util');
var git = require('gulp-git');
var fs = require('fs');

gulp.task('changelog', function () {
  return gulp.src('CHANGELOG.md', {
    buffer: false
  })
    .pipe(conventionalChangelog({
      preset: 'angular' // 或者其他任何你在使用的 commit 信息规范
    }))
    .pipe(gulp.dest('./'));
});

gulp.task('github-release', function(done) {
  conventionalGithubReleaser({
    type: "oauth",
    token: '0126af95c0e2d9b0a7c78738c4c00a860b04acc8' // 将这个更改至你自己的 GitHub token，或者使用一个环境变量
  }, {
    preset: 'angular' // 或者其他任何你在使用的 commit 信息规范
  }, done);
});

gulp.task('bump-version', function () {
// 这里我们硬编码了版本更新类型为 'patch'，但是更好的方式是
// 使用 (https://www.npmjs.com/package/minimist) 模块，定义一个
// 命令行参数来确定你是需要 'major'，'minor'，还是 'patch' 类型的更新
  return gulp.src(['./bower.json', './package.json'])
    .pipe(bump({type: "patch"}).on('error', gutil.log))
    .pipe(gulp.dest('./'));
});

gulp.task('commit-changes', function () {
  return gulp.src('.')
    .pipe(git.commit('[Prerelease] Bumped version number'));
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
    // 我们自己解析这个 json 文件，因为使用 require 会有缓存
    // 在多次调用的时候，版本号并不会改变
    return JSON.parse(fs.readFileSync('./package.json', 'utf8')).version;
  };
});

gulp.task('release', function (callback) {
  runSequence(
    'bump-version',
    'changelog',
    'commit-changes',
    'push-changes',
    'create-new-tag',
    'github-release',
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
