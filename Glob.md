# Glob
> https://github.com/isaacs/node-glob#readme
glob允许使用规则，从而获取对应规则匹配的文件。这个glob工具基于javascript.它使用了 **minimatch** 库来进行匹配
```
var glob = require("glob")

// options is optional
glob("**/*.js", options, function (er, files) {
  // files is an array of filenames.
  // If the `nonull` option is set, and nothing
  // was found, then files is ["**/*.js"]
  // er is an error object or null.
})
```
## Glob 入门
在解析路径规则（pattern）前，大括号扩展部分转换成set。braced 由{}包裹，中间逗号分割，中间包括分割的字符
下面的字符用在路径中有特殊的意义：
* `*`匹配0或多个字符的搜索
* `？`匹配某部分的一个字符
* []
* (!|?|+)(pattern|pattern|pattern)
* @(pattern|pattern)匹配其中之一

## Dots
点符号具有特殊性，不被*所匹配
* a/.*/c -> a/.b/c
* 设置 dot:true 

## Basename Matching
* matchBase:true
会开启所有文件路径的搜索

## Empty Sets
没有匹配的文件会返回 []

## API
### glob.hasMagic(pattern, [options])
判定是否含有特殊字符
options会影响判断
* noext:true：不识别特殊字符
* nobrace:true：不识别大括号扩展

### glob(pattern, [options], cb)
异步文件搜索

### glob.sync(pattern, [options])
同步文件搜索

## Class: glob.Glob
通过实例化 `glob.Glob` 类创建一个 Glob 对象
```
var Glob = require("glob").Glob
var mg = new Glob(pattern, options, cb)
```
一个 EventEmitter 并立即开始在文件系统中进行匹配

### new glob.Glob(pattern, [options], [cb])
如果 options 设置了 sync ，匹配结果可以在 `g.found` 中获得

### Properties
？？ 这个东西用在哪里？？

## Events
* end:搜索结束时触发，返回匹配结果
* match:每次搜索成功触发
* error:
* abort:停止搜索

## Events
* pause
* resume
* abort

## Options
所有 可以传入 miniMatch 的 options 也可以传入 Glob
默认 false


