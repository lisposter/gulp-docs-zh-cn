# 入门指南

#### 1. 全局安装 gulp：

__如果你之前有全局安装过一个版本的 gulp，请执行一下 `npm rm --global gulp` 来避免和 gulp-cli 冲突__

```sh
$ npm install --global gulp-cli
```

#### 2. 作为项目的开发依赖（devDependencies）安装：

```sh
$ npm install --save-dev gulp
```

#### 3. 在项目根目录下创建一个名为 `gulpfile.js` 的文件：

```js
var gulp = require('gulp');

gulp.task('default', function() {
  // 将你的默认的任务代码放在这
});
```

#### 4. 运行 gulp：

```sh
$ gulp
```

默认的名为 default 的任务（task）将会被运行，在这里，这个任务并未做任何事情。

想要单独执行特定的任务（task），请输入 `gulp <task> <othertask>`。

## 下一步做什么呢？

你已经安装了所有必要的东西，并且拥有了一个空的 gulpfile。那怎样才算是__真的__入门了呢？可以查看这些 [秘籍](recipes) 和这个 [文章列表](README.md#articles) 来学习更多的内容。

## .src, .watch, .dest, CLI 参数 - 我该怎么去用这些东西呢？

要了解 API 规范文档，请查看 [API 文档](API.md).

## 可用的插件

gulp 开发社区正在快速成长，每天都会有新的插件诞生。在 [主站](http://gulpjs.com/plugins/) 上可以查看完整的列表。

