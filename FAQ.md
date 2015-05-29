# FAQ

## 为什用 gulp 而不是 ____?

请先看 [gulp 介绍幻灯片] 来大致了解下 gulp 是怎么来的。

## 是 "gulp" 还是 "Gulp"？

gulp 一直都是小写的。除了在 gulp 的 logo 中是用大写的。

## 去哪里可以找到 gulp 插件的列表？

gulp 插件总是会包含 `gulpplugin` 关键字。在这[搜索 gulp 插件][search-gulp-plugins] 或者 在 npm [查看所有插件][npm plugin search]。

## 我想写一个 gulp 插件，我应该从哪里开始呢？

请查看 [编写插件] wiki 页面来阅读一些指导以及一些例子。

## 我的插件将做 ____, 它是不是做的太多了？

有可能。可以先自问下：

1. 我的插件是否做了一些其他插件可能需要做的事情？
  - 如果是，那么那一段功能应该作为一个独立的插件。[查看是否已经有相应的插件存在了][npm plugin search].
1. 我的插件是否做了两件事，两件根据配置的不同而截然不同的事情？
  - 如果是，那么为了社区的良好发展，最好是分开为两个插件发布
  - 如果两个任务是不同的，但是差别非常细微，那实际上是允许的

## 换行符在插件输出中应该如何表示？

请总是使用 `\n` 以避免不同的操作系统带来的兼容性问题。

## 我可以从哪里获取 gulp 的最新信息？

gulp 的更新信息可以通过关注以下的 twitter 来获取：

- [@wearefractal](https://twitter.com/wearefractal)
- [@eschoff](https://twitter.com/eschoff)
- [@gulpjs](https://twitter.com/gulpjs)

## gulp 是否有 IRC 频道？

有的，欢迎来 [Freenode] 上的 #gulpjs 来交流。

[编写插件]: writing-a-plugin/README.md
[gulp 介绍幻灯片]: http://slid.es/contra/gulp
[Freenode]: http://freenode.net/
[search-gulp-plugins]: http://gulpjs.com/plugins/
[npm plugin search]: https://npmjs.org/browse/keyword/gulpplugin
