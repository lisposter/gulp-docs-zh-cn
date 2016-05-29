# 分离任务到多个文件中

如果你的 `gulpfile.js` 开始变得很大，你可以通过使用下面的方法之一将任务分离到多个文件：

> 注意，这个方法 [考虑弃用][deprecated] 了
> 并且当迁移到 `gulp 4` 后会引发一些问题。

## 使用 `gulp-require-tasks`

你可以使用 [gulp-require-tasks][gulp-require-tasks]
模块来自动载入所有单独文件中的任务。

请查看 [模块的 README][gulp-require-tasks] 获取最新的说明。

## 使用 `require-dir`

你可以使用 [require-dir][require-dir] 模块来手动载入任务。

想象如下的文件结构：

```
gulpfile.js
tasks/
├── dev.js
├── release.js
└── test.js
```

安装 `require-dir`：

```sh
npm install --save-dev require-dir
```

在你的 `gulpfile.js` 文件中加入以下代码：

```js
var requireDir = require('require-dir');
var tasks = requireDir('./tasks');
```


  [gulp-require-tasks]: https://github.com/betsol/gulp-require-tasks
  [require-dir]:        https://github.com/aseemk/requireDir
  [deprecated]:         https://github.com/gulpjs/gulp/pull/1554#issuecomment-202614391
