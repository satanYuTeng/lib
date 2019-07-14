# cross-env
> [https://www.npmjs.com/package/cross-env](https://www.npmjs.com/package/cross-env)

## The problem
当您使用 NODE_ENV = production 设置环境变量时，大多数Windows命令提示都会阻塞。 （例外是Windows上的Bash，它使用本机Bash。）同样，Windows和POSIX命令如何利用环境变量也有所不同。使用POSIX，您可以使用：$ ENV_VAR，在Windows上使用％ENV_VAR％。

## This solution
cross-env使得您可以拥有一个命令，而无需担心为平台正确设置或使用环境变量。只需设置它就像在POSIX系统上运行一样，cross-env将负责正确设置它。

## Installation
```
npm install --save-dev cross-env
```

## Usage
使用示例
```javascript
{
  "scripts": {
    "build": "cross-env NODE_ENV=production webpack --config build/webpack.config.js"
  }
}
```
最终，执行的命令（使用[`cross-spawn`](https://www.npmjs.com/package/cross-spawn)）是：
```javascript
webpack --config build/webpack.config.js
```
NODE_ENV环境变量会被cross-env设置

您还可以将一条命令拆分为多个命令，或将环境变量声明与实际命令执行分开。你可以这样做：
```javascript
{
  "scripts": {
    "parentScript": "cross-env GREET=\"Joe\" npm run childScript",
    "childScript": "cross-env-shell \"echo Hello $GREET\""
  }
}
```
其中 childScript包含要执行的实际命令，而parentScript设置要使用的环境变量。然后运行父代，而不是运行childScript。这对于使用不同的 env 变量启动相同的命令或者当环境变量太长而无法在一行中包含所有内容时非常有用。这也意味着即使在Windows上也可以使用$ GREET env var语法，通常要求它为％GREET％。

如果您在 $ 符号前面加上奇数个反斜杠，那么表达式语句将不会被替换。请注意，这意味着在JSON字符串转义后的反斜杠数量。 “FOO = \\$ BAR”将不会被替换。 “FOO = \\\\ $ BAR”将被替换。

最后，如果你需要传递 JSON 字符串（比如使用ts-loader），可以这么处理：
```javascript
{
  "scripts": {
    "test": "cross-env TS_NODE_COMPILER_OPTIONS={\\\"module\\\":\\\"commonjs\\\"} node some_file.test.ts"
  }
}
```
要特别注意双引号（“）之前的三个反斜杠（\\\）和没有单引号（'）。为了在Windows和UNIX上都能工作，必须满足这两个条件。

## `cross-env` vs `cross-env-shell`
cross-env模块公开了两个bin：cross-env和cross-env-shell。第一个使用cross-spawn执行命令，而第二个使用Node的spawn中的shell选项。？？
cross-env-shell的主要使用场景是当你需要在整个内部shell脚本中设置环境变量时，而不是只需要一个命令。
例如，如果要将环境变量应用于串行的多个命令，则需要将它们包含在引号中并使用cross-env-shell而不是cross-env。
```javascript
{
  "scripts": {
    "greet": "cross-env-shell GREETING=Hi NAME=Joe \"echo $GREETING && echo $NAME\""
  }
}
```
经验法则是：如果要传递给包含要解释（使用）的特殊shell字符的命令，请使用cross-env-shell。否则使用`cross-env`。
在Windows上，如果要处理程序内部的 [signal events](https://nodejs.org/api/process.html#process_signal_events) ，则需要使用cross-env-shell。一种常见的情况是，您希望通过在命令行界面上按Ctrl + C来捕获调用的 SIGINT EVENT。 ??

## Other Solutions
[`env-cmd`](https://github.com/toddbluhm/env-cmd) - 从文件中读取环境变量
