# npm-updater & npm-check-updates
## npm-updater
> https://www.npmjs.com/package/npm-updater

提示升级
```
/**
 * check a package lastest version
 * @param {Object} options - query Object
 * @param {Object} options.name - package name, default get from parent package.
 * @param {Object} options.version - package current version, default get from parent package.
 * @param {Object} [options.package] - pass module's `package.json` object
 * @param {String} [options.registry] - publishConfig.registry || npm
 * @param {String} [options.tag] - compare with which tag, default to `latest`.
 * @param {String} [options.interval] - notify interval, default to 1d.通知间隔 1s
 * @param {Boolean} [options.abort] - If remote version changed, should we abort? default to `true`.
 * @param {String} [options.level] - abort level, default to `minor`.minor-不兼容
 * @param {String} [options.updateMessage] - appending update message.
 * @param {Function} [options.formatter] - custom format fn, with args { name, version, current, isAbort, options }.
 * @return {Object} - { name, version, current, type, pkg, options }, type: latest, major, minor, patch, prerelease, build, null
 */
 ```

 ## npm-check-updates
 > https://www.npmjs.com/package/npm-check-updates
升级 package.json 中的依赖到最新版本
```
ncu
ncu -u
--filter and --reject options
```

### Module Use
可以通过 require 引用
```
ncu.run({
  // 命令行的 options 可以在这里进行使用
});
```
