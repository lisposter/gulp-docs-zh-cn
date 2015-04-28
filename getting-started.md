# 入门指南

#### 1. 全局安装 gulp：

```sh
$ npm install --global gulp
```

#### 2. 作为项目的开发环境依赖（devDependencies）安装：

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

这个默认的任务将会被运行，并且其实啥也没干。

想要运行单独的任务，请这样执行 `gulp <task> <othertask>`.

## 现在从哪开始？

你已经有了一个空的 gulpfile 并且所需要的都已经安装了。那怎样才算是__真的__入门了呢？查看这些 [recipes](recipes) 和这个 [文章列表](README.md#articles) 来获取更多的信息吧。

## .src, .watch, .dest, CLI args - 我怎么去用这些东西？

要了解 API 规范文档，请查看 [API 文档](API.md).

## 可用的插件

gulp 开发社区正在快速成长，每天都会有新的插件诞生。在 [主站](http://gulpjs.com/plugins/) 上可以查看完整的列表。

