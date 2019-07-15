# Chokidar-文件监听
## Why
Node.js fs.watch

- OS X 系统环境不报告文件名变化
- OS X 系统中使用Sublime等编辑器时，不报告任何事件
- 经常会报告两次事件
- 多数事件通知为 `rename`
- 不能够简单地递归监控文件树

Node.js fs.watchFile

- 糟糕的事件处理
- 未提供递归监控

## How
Chokidar仍然依赖于Node.js核心fs模块，但是当使用fs.watch和fs.watchFile进行观看时，它会对收到的事件进行规范化，通常通过获取文件统计信息或目录内容来检查正确性。
在MacOS上，chokidar 默认使用native扩展暴露Darwin FSEvents API。与大多数 nix平台上提供的kqueue等实现相比，这提供了非常高效的递归观察。 Chokidar仍然需要做一些工作来规范化收到的事件。
在其他平台上，基于fs.watch的实现是默认的，这可以避免轮询并降低CPU使用率。请注意，chokidar将以指定路径范围内的所有内容递归地启动监听，因此请注意不要监听远远超过需要的文件来浪费系统资源。

## getting started

```javascript
// Initialize watcher.
const watcher = chokidar.watch('file, dir, glob, or array', {
  ignored: /(^|[\/\\])\../,
  persistent: true
});

// Add event listeners.
watcher
  .on('add', path => log(`File ${path} has been added`))
  .on('change', path => log(`File ${path} has been changed`))
  .on('unlink', path => log(`File ${path} has been removed`));

// More possible events.
watcher
  .on('addDir', path => log(`Directory ${path} has been added`))
  .on('unlinkDir', path => log(`Directory ${path} has been removed`))
  .on('error', error => log(`Watcher error: ${error}`))
  .on('ready', () => log('Initial scan complete. Ready for changes'))
  .on('raw', (event, path, details) => {
    log('Raw event info:', event, path, details);
  });

// 'add', 'addDir' and 'change' events also receive stat() results as second
// argument when available: http://nodejs.org/api/fs.html#fs_class_fs_stats
watcher.on('change', (path, stats) => {
  if (stats) console.log(`File ${path} changed size to ${stats.size}`);
});

// Watch new files.
watcher.add('new-file');
watcher.add(['new-file-2', 'new-file-3', '**/other-file*']);

// Get list of actual paths being watched on the filesystem
// 获取文件系统上正在被监听文件的实际路径列表 ？？
var watchedPaths = watcher.getWatched();

// Un-watch some files.
watcher.unwatch('new-file*');

// Stop watching.
watcher.close();
```

## API
chokidar.watch（path，[options]）
paths（字符串或字符串数组）:文件的路径，递归观察的dirs或glob模式。
options（object）选项对象，如下所示：

### Persistence

- persistent：指示正在监视文件，进程是否应继续运行。如果在使用 fsevents 进行监视时设置为 false，则准备就绪后不会再发出任何事件，即使进程继续运行也是如此。

### Path filtering

- ignore：定义要忽略的文件/路径。测试整个相对或绝对路径，而不仅仅是文件名。如果提供了具有两个参数的函数，则每个路径调用两次 - 一次使用单个参数（路径），第二次使用两个参数（路径和该路径的fs.Stats对象）
- ignoreInitial（默认值：false）。如果设置为false，则在chokidar发现这些文件路径（在ready事件之前）实例化观察时，还会为匹配路径发出add / addDir事件。
- followSymlinks（默认值：true）。如果为false，则仅监视符号链接本身的更改，而不是通过链接的路径跟踪链接引用和冒泡事件。
- cwd（没有默认值）。从中派生监视路径的基本目录。与事件一起发出的路径将与此相关。
- disableGlobbing（默认值：false）。如果设置为true，则传递给.watch（）和.add（）的字符串将被视为文字路径名，即使它们看起来像globs。(文件路径匹配模式_globs_匹配规则)

### Performance

- usePolling（默认值：false）。是否使用fs.watchFile（由轮询支持）或fs.watch。如果轮询导致CPU利用率过高，请考虑将其设置为false。通常需要将此设置为true才能成功通过网络查看文件，并且可能需要在其他非标准情况下成功查看文件。在MacOS上显式设置为true会覆盖useFsEvents默认值。您还可以将CHOKIDAR_USEPOLLING env变量设置为true（1）或false（0）以覆盖此选项。
- 轮询特定设置（usePolling：true时有效）
  - 间隔（默认值：100）。文件系统轮询的间隔。您还可以设置CHOKIDAR_INTERVAL env变量来覆盖此选项。
  - binaryInterval（默认值：300）。文件系统轮询二进制文件的间隔。
- useFsEvents（MacOS上的默认值：true）。是否使用fsevents监听接口（如果可用）。当显式设置为true并且fsevents可用时，这将取代usePolling设置。在MacOS上设置为false时，usePolling：true将成为默认值。
- alwaysStat（默认值：false）。如果依赖于add，addDir和change事件传递的fs.Stats对象，请将其设置为true以确保即使在基础监听callback事件尚未提供的情况下也返回stat。
- 深度（默认值：未定义）。如果设置，则限制将遍历多少级别的子目录。
- awaitWriteFinish（默认值：false）。默认情况下，在写入整个文件之前，当文件首次出现在磁盘上时，将触发add事件。此外，在某些情况下，在写入文件时会发出一些change事件。在某些情况下，尤其是在查看大文件时，需要等待写操作完成，然后才能响应文件创建或修改。将awaitWriteFinish设置为true（或truthy值）将轮询文件大小，挂起其add和change事件，直到大小在可配置的时间内没有更改。适当的持续时间设置在很大程度上取决于操作系统和硬件。为了准确检测，此参数应该相对较高，使文件观察响应性降低。谨慎使用。
  - options.awaitWriteFinish可以设置为一个对象，以便调整时间参数：
  - awaitWriteFinish.stabilityThreshold（默认值：2000）。文件大小在发出事件之前保持不变的时间阈值（以毫秒为单位）
  - awaitWriteFinish.pollInterval（默认值：100）。文件大小轮询间隔。

### Errors

- ignorePermissionErrors（默认值：false）。指示是否在可能的情况下观看没有读取权限的文件。如果由于EPERM或EACCES的设置为true而导致观看失败，则会以静默方式抑制错误。
- atomic（默认值：如果useFsEvents和usePolling为false，则为true）。自动过滤掉使用使用“原子写入”而不是直接写入源文件的编辑器时发生的文件。如果文件在被删除后的100毫秒内重新添加，Chokidar会发出更改事件而不是取消链接然后添加。如果默认值100 ms不适合您，则可以通过将atomic设置为自定义值（以毫秒为单位）来覆盖它。

## Methods & Events
`chokidar.watch()` 返回`FSWatcher实例。`  `FSWatcher实例方法：`
- .add（path / paths）：添加文件，目录或glob模式以进行跟踪。获取字符串数组或仅使用一个字符串。
- .on（event，callback）：侦听FS事件。可用事件：add，addDir，change，unlink，unlinkDir，ready，raw，error。此外，除了ready，raw和error之外的所有事件都可以使用基础事件和路径发出的所有事件。
- .unwatch（path / paths）：停止观看文件，目录或glob模式。获取字符串数组或仅使用一个字符串。
- .close（）：删除侦听器。
- .getWatched（）：返回一个对象，表示此FSWatcher实例正在监视的文件系统上的所有路径。对象的键是所有目录（使用绝对路径，除非使用了cwd选项），并且值是每个目录中包含的项的名称的数组。
