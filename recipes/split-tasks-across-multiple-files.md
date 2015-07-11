# 分离任务到多个文件中

如果你的 `gulpfile.js` 开始变得很大，你可以通过使用 [require-dir](https://github.com/aseemk/requireDir) 模块将任务分离到多个文件

想象如下的文件结构：

```
gulpfile.js
tasks/
├── dev.js
├── release.js
└── test.js
```

安装 `require-dir` 模块：

```sh
npm install --save-dev require-dir
```

在 `gulpfile.js` 中增加如下几行代码：

```js
var requireDir = require('require-dir');
var dir = requireDir('./tasks');
```
