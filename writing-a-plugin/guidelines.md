# 指导

> 这个指导实际上不是必须的，但是我们强烈建议每一个人来遵守。因为没有人会喜欢用一个不好的插件。这个指导能在某种意义上确保你的插件能很好的适应 gulp，以此来让你的生活变得更将轻松。

[编写插件](README.md) > 指导

1. 你的插件不应该去做一些现有 node 模块已经能很容易做到的事情
  - 比如：删除一个文件夹并不需要做成一个 gulp 插件，在 task 里使用一个类似 [del](https://github.com/sindresorhus/del) 这样的插件即可。
  - 只是为了封装而封装一些的东西进去，这只会增加很多低质量的插件到生态中，这不符合 gulp 的期望。
  - gulp 插件都是以文件为基础操作的，如果你发现你正在把一些很复杂的操作塞进 stream 中去，那么，请直接写一个 node 模块就好。
  - 一个好的 gulp 插件例子像是 gulp-coffee，coffee-script 模块并不能直接和 vinyl 做很好的适配，因此，才去封装它来使用相应的功能，并且将一些比较痛苦的操作抽象出来，做成更简单的 gulp 插件来使用。
1. 你的插件应该只做**一件事**，并且做好。
  - 避免使用配置选项，使得你的插件能胜任不同场合的任务。
  - 比如：一个 JS 压缩插件不应该有一个加头部的选项
1. 你的插件不能去做一些其他插件做的事：
  - 不应该去拼接，用 [gulp-concat](https://github.com/wearefractal/gulp-concat) 去做
  - 不应该去增加头部，用 [gulp-header](https://github.com/godaddy/gulp-header) 去做
  - 不应该去增加尾部，用 [gulp-footer](https://github.com/godaddy/gulp-footer) 去做
  - 如果是一个常用的可选的操作，那么，请在文档中注明你的插件通常和其他某个插件一起使用
  - 在你的插件中使用其他的插件，这能大大减少你的代码量，并保证生态系统的稳定。
1. 你的插件必须被**测试过**
  - 测试一个插件很简单，你甚至不需要 gulp 就能测试
  - 参考其他的插件是怎么做的
1.  在 `package.json` 中增加一个名为 `gulpplugin` 的关键字，这可以让它能在我们的搜索中出现
1. 不要再 stream 里面抛出错误
  - 你应该以触发 **error** 事件来代替
  - 如果你在 stream 外面遇到错误，比如在创建 stream 时候发现错误的配置选项等，那么你应该抛出它。
1. 错误需要加上以你插件名字作为前缀
  - 比如： `gulp-replace: Cannot do regexp replace on a stream`
  - 使用 gulp-util 的 [PluginError](https://github.com/gulpjs/gulp-util#new-pluginerrorpluginname-message-options) 类来完成它
1. `file.contents` 的类型需要总是在输入输出中保持一致
  - 如果 file.contents 为空 (不可读) 请将他忽略，并传过去
  - 如果 file.contents 是一个 stream，但是你不支持，那么请触发一个错误
    - 不要把 stream 硬转成 buffer 来使你的插件支持 stream，这会引发很严重的问题。
1. 在你处理完成之前，不要将 `file` 传到下游去
1. 使用 [`file.clone()`](https://github.com/wearefractal/vinyl#clone) 来复制一个文件或者创建另一个以此为基础的文件
1. 使用我们 [模块推荐页](recommended-modules.md) 上列举的模块来让你的开发更加轻松
1. 不要把 `gulp` 作为一个依赖
  - 使用 gulp 来测试你的插件的工作流这的确很酷，但请务必确保你将它放到 devDependency 中
  - 在你的插件中依赖 gulp，这意味着安装你的插件的用户将会重新安装一遍 gulp 以及所有它所依赖的东西。
  - 没有任何理由说明你需要将 gulp 写到你的插件代码中去，如果你发现你必须这么做，那么请开一个 issue，我们会帮你解决。

## 为什么这些指导这么严格？

gulp 的目标是为了让用户觉得简单，通过提供一些严格的指导，我们就能提供一致并且高质量的生态系统给大家。不过，这确实给插件作者增加了一些需要考虑的东西，但是也确保了后面的问题会更少。

### 如果我不遵守这些，会发生什么？

npm 对每个人来说是免费的，你可以开发任何你想要开发的东西出来，并且不需要遵守这个规定。我们承诺测试机制将会很快建立起来，并且加入我们的插件搜索中。如果你坚持不遵守插件导览，那么这会反应在我们的打分/排名系统上，人们都会更加喜欢去使用一个 "更加 gulp" 的插件。

### 一个插件大概会是怎么样的？

```js
// through2 是一个对 node 的 transform streams 简单封装
var through = require('through2');
var gutil = require('gulp-util');
var PluginError = gutil.PluginError;

// 常量
const PLUGIN_NAME = 'gulp-prefixer';

function prefixStream(prefixText) {
  var stream = through();
  stream.write(prefixText);
  return stream;
}

// 插件级别函数 (处理文件)
function gulpPrefixer(prefixText) {

  if (!prefixText) {
    throw new PluginError(PLUGIN_NAME, 'Missing prefix text!');
  }
  prefixText = new Buffer(prefixText); // 预先分配

  // 创建一个让每个文件通过的 stream 通道
  return through.obj(function(file, enc, cb) {
    if (file.isNull()) {
      // 返回空文件
      cb(null, file);
    }
    if (file.isBuffer()) {
      file.contents = Buffer.concat([prefixText, file.contents]);
    }
    if (file.isStream()) {
      file.contents = file.contents.pipe(prefixStream(prefixText));
    }

    cb(null, file);

  });

};

// 暴露（export）插件主函数
module.exports = gulpPrefixer;
```
