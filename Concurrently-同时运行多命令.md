# Concurrently-同时运行多命令
> 翻译自 [https://github.com/kimmobrunfeldt/concurrently](https://github.com/kimmobrunfeldt/concurrently)

我喜欢npm的任务自动化，但同时运行多个命令的常用方法是 npm run watch-js＆npm run watch-css。这很好，但很难追踪不同的输出。此外，如果一个进程失败，其他进程仍然继续运行，您甚至不会注意到差异。
另一种选择是在不同的终端中运行所有命令。我厌倦了打开终端去做并行。
特征：

跨平台（包括Windows）

- 输出很容易通过前缀区分
- 使用--kill-others开关，如果一个命令失败，所有命令都将被终止
- 使用spawn-command生成命令
  - spawn-command：类似 child_process.exec

## 使用
请记住使用引号括起单独的命令

```javascript
concurrently "command1 arg" "command2 arg"
```

否则 **concurrently **会同时运行4个独立的命令：command1 ，arg，command2 ，arg
package.json中需要转译引号

```javascript
"start": "concurrently \"command1 arg\" \"command2 arg\""
```

可以缩短NPM运行命令

```javascript
concurrently "npm:watch-js" "npm:watch-css" "npm:watch-node"

# Equivalent to:
concurrently -n watch-js,watch-css,watch-node "npm run watch-js" "npm run watch-css" "npm run watch-node"
```
NPM缩短命令也支持通配符。给出package.json中的以下脚本：

```javascript
{
    //...
    "scripts": {
        // ...
        "watch-js": "...",
        "watch-css": "...",
        "watch-node": "...",
        // ...
    },
    // ...
}
  
concurrently "npm:watch-*"

# Equivalent to:
concurrently -n js,css,node "npm run watch-js" "npm run watch-css" "npm run watch-node"

# Any name provided for the wildcard command will be used as a prefix to the wildcard
# part of the script name:
concurrently -n w: npm:watch-*

# Equivalent to:
concurrently -n w:js,w:css,w:node "npm run watch-js" "npm run watch-css" "npm run watch-node"
```

## 程序化使用
concurrently可以通过 API 在程序中调用

## FAQ
进程退出时代码为null？
子进程结束后将发出此事件。如果进程正常终止，则代码是进程的最终退出代码，否则为null。如果由于接收到信号而终止该过程，则signal是信号的字符串名称，否则为null


