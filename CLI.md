## gulp 命令行（CLI）文档

### 参数标记

gulp 只有你需要熟知的参数标记，其他所有的参数标记只在一些任务需要的时候使用。

- `-v` 或 `--version` 会显示全局和项目本地所安装的 gulp 版本号
- `--require <module path>` 将会在执行之前 reqiure 一个模块。这对于一些语言编译器或者需要其他应用的情况来说来说很有用。你可以使用多个`--require`
- `--gulpfile <gulpfile path>` 手动指定一个 gulpfile 的路径，这在你有很多个 gulpfile 的时候很有用。这也会将 CWD 设置到该 gulpfile 所在目录
- `--cwd <dir path>` 手动指定 CWD。定义 gulpfile 查找的位置，此外，所有的相应的依赖（require）会从这里开始计算相对路径
- `-T` 或 `--tasks` 会显示所指定 gulpfile 的 task 依赖树
- `--tasks-simple` 会以纯文本的方式显示所载入的 gulpfile 中的 task 列表
- `--color` 强制 gulp 和 gulp 插件显示颜色，即便没有颜色支持
- `--no-color` 强制不显示颜色，即便检测到有颜色支持
- `--silent` 禁止所有的 gulp 日志

命令行会在 process.env.INIT_CW 中记录它是从哪里被运行的。

#### Task 特定的参数标记

请参考 [StackOverflow](http://stackoverflow.com/questions/23023650/is-it-possible-to-pass-a-flag-to-gulp-to-have-it-run-tasks-in-different-ways) 了解如何增加任务特定的参数标记。

### Tasks

Task 可以通过 `gulp <task> <othertask>` 方式来执行。如果只运行 `gulp` 命令，则会执行所注册的名为 `default` 的 task，如果没有这个 task，那么 gulp 会报错。

### 编译器

你可以在 [interpret](https://github.com/tkellen/node-interpret#jsvariants) 找到所支持的语言列表。如果你想要增加一个语言的支持，请在这里提交一个 pull request 或者 issue。
