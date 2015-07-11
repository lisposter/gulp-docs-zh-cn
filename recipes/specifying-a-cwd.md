# 指定一个新的 cwd (当前工作目录)

在一个多层嵌套的项目中，这是非常有用的，比如：

```
/project
  /layer1
  /layer2
```

你可以使用 gulp 的 CLI 参数 `--cwd`.

在 `project/` 目中中：

```sh
gulp --cwd layer1
```

如果你需要对特定的匹配指定一个 cwd，你可以使用 [glob-stream](https://github.com/wearefractal/glob-stream) 的 `cwd` 选项：

```js
gulp.src('./some/dir/**/*.js', { cwd: 'public' });
```
