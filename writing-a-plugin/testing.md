# 测试

> 测试是保证你的插件质量的唯一途径。这能使你的用户有信心去使用，且能让你更加轻松。

[编写插件](README.md) > Testing


## 工具

大多数的插件使用 [mocha](https://github.com/visionmedia/mocha)，[should](https://github.com/visionmedia/should.js) 以及 [event-stream](https://github.com/dominictarr/event-stream) 来做测试。下面的例子也将会使用这些工具。


## 测试插件的流处理（streaming）模式

```js
var assert = require('assert');
var es = require('event-stream');
var File = require('vinyl');
var prefixer = require('../');

describe('gulp-prefixer', function() {
  describe('in streaming mode', function() {

    it('should prepend text', function(done) {

      // 创建伪文件
      var fakeFile = new File({
        contents: es.readArray(['stream', 'with', 'those', 'contents'])
      });

      // 创建一个 prefixer 流（stream）
      var myPrefixer = prefixer('prependthis');

      // 将伪文件写入
      myPrefixer.write(fakeFile);

      // 等文件重新出来
      myPrefixer.once('data', function(file) {
        // 确保它以相同的方式出来
        assert(file.isStream());

        // 缓存内容来确保它已经被处理过（加前缀内容）
        file.contents.pipe(es.wait(function(err, data) {
          // 检查内容
          assert.equal(data, 'prependthisstreamwiththosecontents');
          done();
        }));
      });

    });

  });
});
```


## 测试插件的 buffer 模式

```js
var assert = require('assert');
var es = require('event-stream');
var File = require('vinyl');
var prefixer = require('../');

describe('gulp-prefixer', function() {
  describe('in buffer mode', function() {

    it('should prepend text', function(done) {

      // 创建伪文件
      var fakeFile = new File({
        contents: new Buffer('abufferwiththiscontent')
      });

      // 创建一个 prefixer 流（stream）
      var myPrefixer = prefixer('prependthis');

      // 将伪文件写入
      myPrefixer.write(fakeFile);

      // 等文件重新出来
      myPrefixer.once('data', function(file) {
        // 确保它以相同的方式出来
        assert(file.isBuffer());

        // 检查内容
        assert.equal(file.contents.toString('utf8'), 'prependthisabufferwiththiscontent');
        done();
      });

    });

  });
});
```


## 一些拥有高质量的测试用例的插件

* [gulp-cat](https://github.com/ben-eb/gulp-cat/blob/master/test.js)
* [gulp-concat](https://github.com/contra/gulp-concat/blob/master/test/main.js)
