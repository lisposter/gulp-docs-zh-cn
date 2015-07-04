# 在使用通配符的时候维持目录结构

如果你打算从一个目录中读取一系列的文件或文件夹，并且想要维持原来的目录结构，那么你需要把 `{base: '.'}` 以第二个参数传递给 `gulp.src()`。


举个例子，如果你有一个像这样的目录结构

![Dev setup](https://cloud.githubusercontent.com/assets/2562992/3178498/bedf75b4-ec1a-11e3-8a71-a150ad94b450.png)

并且你想要读取像这样的文件

```js
[ 'index.html',
 'css/**',
 'js/**',
 'lib/**',
 'images/**',
 'plugin/**'
 ]
```

在这个例子中，gulp 将会读取所有（_所说的_）`css` 的子目录，然后将它们相对于你的根目录放置，并且它们将不在作为 `css` 的子目录存在。通配操作后，输出目录将会像是这样

![Zipped-Unzipped](https://cloud.githubusercontent.com/assets/2562992/3178614/27208c52-ec1c-11e3-852e-8bbb8e420c7f.png)

如果你想要维持这个目录结果，你需要把 `{base: '.'}` 传递给 `gulp.src()`。像这样

```js
gulp.task('task', function () {
   gulp.src(['index.html',
             'css/**',
             'js/**',
             'lib/**',
             'images/**',
             'plugin/**'
             ], {base: '.'})
       .pipe(operation1())
       .pipe(operation2());
});
```
然后，输入到 `operation1()` 的目录结构将会是这样子

![with-base](https://cloud.githubusercontent.com/assets/2562992/3178607/053d6722-ec1c-11e3-9ba8-7ce39e1a480e.png)

