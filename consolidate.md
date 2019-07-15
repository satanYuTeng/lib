# consolidate-模板引擎整合库
> 模板引擎整合库。

## Supported template engines
- [atpl](https://github.com/soywiz/atpl.js)
- [bracket](https://github.com/danlevan/bracket-template)
- [doT.js](https://github.com/olado/doT) [(website)](http://olado.github.io/doT/)
- [dust (unmaintained)](https://github.com/akdubya/dustjs) [(website)](http://akdubya.github.com/dustjs/)
- [dustjs-linkedin (maintained fork of dust)](https://github.com/linkedin/dustjs) [(website)](http://linkedin.github.io/dustjs/)
- [eco](https://github.com/sstephenson/eco)
- [ect](https://github.com/baryshev/ect) [(website)](http://ectjs.com/)
- [ejs](https://github.com/mde/ejs) [(website)](http://ejs.co/)
- [haml](https://github.com/visionmedia/haml.js)
- [haml-coffee](https://github.com/9elements/haml-coffee)
- [hamlet](https://github.com/gregwebs/hamlet.js)
- [handlebars](https://github.com/wycats/handlebars.js/) [(website)](http://handlebarsjs.com/)
- [hogan](https://github.com/twitter/hogan.js) [(website)](http://twitter.github.com/hogan.js/)
- [htmling](https://github.com/codemix/htmling)
- [jade](https://github.com/visionmedia/jade) [(website)](http://jade-lang.com/)
- [jazz](https://github.com/shinetech/jazz)
- [jqtpl](https://github.com/kof/jqtpl)
- [JUST](https://github.com/baryshev/just)
- [liquid](https://github.com/leizongmin/tinyliquid) [(website)](http://liquidmarkup.org/)
- [liquor](https://github.com/chjj/liquor)
- [lodash](https://github.com/bestiejs/lodash) [(website)](http://lodash.com/)
- [marko](https://github.com/marko-js/marko) [(website)](http://markojs.com/)
- [mote](https://github.com/satchmorun/mote) [(website)](http://satchmorun.github.io/mote/)
- [mustache](https://github.com/janl/mustache.js)
- [nunjucks](https://github.com/mozilla/nunjucks) [(website)](https://mozilla.github.io/nunjucks)
- [plates](https://github.com/flatiron/plates)
- [pug (formerly jade)](https://github.com/pugjs/pug) [(website)](http://jade-lang.com/)
- [QEJS](https://github.com/jepso/QEJS)
- [ractive](https://github.com/Rich-Harris/Ractive)
- [react](https://github.com/facebook/react)
- [slm](https://github.com/slm-lang/slm)
- [swig (maintained fork)](https://github.com/node-swig/swig-templates)
- [swig (unmaintained)](https://github.com/paularmstrong/swig)
- [teacup](https://github.com/goodeggs/teacup)
- [templayed](http://archan937.github.com/templayed.js/)
- [toffee](https://github.com/malgorithms/toffee)
- [twig](https://github.com/justjohn/twig.js)
- [underscore](https://github.com/documentcloud/underscore) [(website)](http://underscorejs.org/#template)
- [vash](https://github.com/kirbysayshi/vash)
- [walrus](https://github.com/jeremyruppel/walrus) [(website)](http://documentup.com/jeremyruppel/walrus/)
- [whiskers](https://github.com/gsf/whiskers.js)

注意：您仍然必须安装要使用的模版引擎，将它们添加到package.json依赖项中。

## API
此库支持的所有模板都可以使用函数签名(path[,locals],callback)进行渲染，如下所示，这恰好是Express 3.x支持的签名，因此任何这些引擎都可以在Express中使用。

注意：所有此示例代码都使用 cons.swig 作为模板引擎。可以用正在使用的任何模板替换swig。例如，对于hogan.js使用cons.hogan，对jade使用cons.jade等，使用console.log（cons）获得完整引擎列表。

```javascript
var cons = require('consolidate');
cons.swig('views/page.html', { user: 'tobi' }, function(err, html){
  if (err) throw err;
  console.log(html);
});
```
要动态传递引擎，只需使用下标运算符和变量：
```javascript
var cons = require('consolidate')
  , name = 'swig';

cons[name]('views/page.html', { user: 'tobi' }, function(err, html){
  if (err) throw err;
  console.log(html);
});
```

### Promises
此外，如果未提供回调函数，则所有模板都可选择返回promise。 promise表示模板函数的最终结果，该函数将解析为字符串，从模板编译或被 reject。 Promise公开一个then方法，它注册回调以接收promise的最终值，以及一个catch方法，去捕获错误。 

```javascript
var cons = require('consolidate');

cons.swig('views/page.html', { user: 'tobi' })
  .then(function (html) {
    console.log(html);
  })
  .catch(function (err) {
    throw err;
  });
```

## Caching
要启用缓存，只需传递{cache：true}。引擎可以使用此选项来缓存读取文件内容，编译函数等的内容。不支持此功能的引擎可能会忽略它。 所有consolidate支持的I\O引擎都将缓存文件内容，非常适合生产环境。直接使用合并时：
```cons.swig（'views / page.html'，{user：'tobi'，cache：true}，callback;```

## Express 3.x example
## Template Engine Instances
模板引擎通过cons.requires对象公开，但在您调用`cons[engine].render()`方法之前，它们不会被实例化。如果要添加过滤器，全局变量，混合或其他引擎功能，可以事先手动实例化它们

```javascript
var cons = require('consolidate'),
  nunjucks = require('nunjucks');

// add nunjucks to requires so filters can be
// added and the same instance will be used inside the render method
cons.requires.nunjucks = nunjucks.configure();

cons.requires.nunjucks.addFilter('foo', function () {
  return 'bar';
});
```

## Notes
- 如果您正在使用Nunjucks，请查看lib.consolidate.js中的exports.nunjucks.render函数。您可以通过options.nunjucksEnv传递您自己的引擎/环境，或者如果您想支持Express，您可以传递options.settings.views，或者如果您有其他用例，请传递options.nunjucks（请参阅代码以获取更多信息）。
- 您可以使用options.partials传递部分内容
- 对于使用nunjucks的模板继承，您可以使用options.loader传递一个加载器。
- 要使用带有tinyliquid的过滤器，请使用options.filters并指定一个属性数组，每个属性都是一个命名过滤器函数。过滤器函数将字符串作为参数并返回其修改版本。
- 要使用带有tinyliquid的自定义标记，请使用options.customTags指定遵循tinyliquid自定义标记定义的标记函数数组。
- 与tinyliquid包含标记一起使用的默认目录是当前工作目录。要覆盖它，请使用options.includeDir。
- React要将内容呈现到html基本模板（例如，React应用程序的index.html），请使用options.base传递模板的路径。
