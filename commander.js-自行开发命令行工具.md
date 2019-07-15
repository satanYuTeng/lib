# commander.js-自行开发命令行工具
> 翻译自 [https://github.com/tj/commander.js](https://github.com/tj/commander.js)
> 用于自行开发命令行工具来初始化自定义项目

## Declaring program variable
Commander导出一个方便快速编程的全局对象。README中的进行简单示例。
```javascript
const program = require('commander');
program.version('0.0.1');
```
对于可能以多种方式使用commander的大型程序，包括单元测试，最好创建一个本地Command对象来使用。
```javascript
const commander = require('commander');
const program = new commander.Command();
program.version('0.0.1');
```

## Options
Options 使用 `.option()` 方法定义，也用作 option 的文档。每个 option 都可以有一个简短标识（单个字符）和一个长标识，用逗号或空格分隔。

可以在 Command 对象上作为属性访问这些选项。诸如`--template-engine`之类的多单词选项将转换成驼峰式，变成了program.templateEngine。多个短标志可以组合成一个arg，例如-abc相当于-a -b -c。

## Common option types, boolean and value
两个最常用的 option 类型是布尔类型或取值类型。除非在命令行中指定，否则两者的结果都是未定义。
```javascript
const program = require('commander');

program
  .option('-d, --debug', 'output extra debugging')
  .option('-s, --small', 'small pizza size')
  .option('-p, --pizza-type <type>', 'flavour of pizza');

program.parse(process.argv);

if (program.debug) console.log(program.opts());
console.log('pizza details:');
if (program.small) console.log('- small pizza size');
if (program.pizzaType) console.log(`- ${program.pizzaType}`);

输出：
$ pizza-options -d
{ debug: true, small: undefined, pizzaType: undefined }
pizza details:
$ pizza-options -p
error: option `-p, --pizza-type <type>' argument missing
$ pizza-options -ds -p vegetarian
{ debug: true, small: true, pizzaType: 'vegetarian' }
pizza details:
- small pizza size
- vegetarian
$ pizza-options --pizza-type=cheese
pizza details:
- cheese
```
program.parse（arguments）处理参数，将未使用的 args 作为 program[args]。

## Default option value
您可以为option中的取值指定默认值。
```javascript
const program = require('commander');

program
  .option('-c, --cheese <type>', 'add the specified type of cheese', 'blue');

program.parse(process.argv);

console.log(`cheese: ${program.cheese}`);

输出：
$ pizza-options
cheese: blue
$ pizza-options --cheese stilton
cheese: stilton
```

## Other option types, negatable boolean and flag|value
您可以指定一个带有前导 no- 的布尔值长名称，默认情况下使其为true，并且可以取消。
```javascript
const program = require('commander');

program
  .option('-n, --no-sauce', 'Remove sauce')
  .parse(process.argv);

if (program.sauce) console.log('you ordered a pizza with sauce');
else console.log('you ordered a pizza without sauce');

输出：
$ pizza-options
you ordered a pizza with sauce
$ pizza-options --sauce
error: unknown option `--sauce'
$ pizza-options --no-sauce
you ordered a pizza without sauce
```

您可以指定一个可选的 option，同时它也可以使用取值（使用方括号声明）。
```javascript
const program = require('commander');

program
  .option('-c, --cheese [type]', 'Add cheese with optional type');

program.parse(process.argv);

if (program.cheese === undefined) console.log('no cheese');
else if (program.cheese === true) console.log('add cheese');
else console.log(`add cheese type ${program.cheese}`);

输出：
$ pizza-options
no cheese
$ pizza-options --cheese
add cheese
$ pizza-options --cheese mozzarella
add cheese type mozzarella
```

## Custom option processing
您可以指定一个函数来自定义处理options的值。回调函数接收两个参数，用户指定的值和选项的旧值。它返回选项的新值。这允许您将选项值强制转换为所需类型或累积计算值或完全自定义处理。您可以选择在自定义处理函数后指定选项的默认/起始值。

```javascript
const program = require('commander');

function myParseInt(value, dummyPrevious) {
  // parseInt takes a string and an optional radix
  return parseInt(value);
}

function increaseVerbosity(dummyValue, previous) {
  return previous + 1;
}

function collect(value, previous) {
  return previous.concat([value]);
}

function commaSeparatedList(value, dummyPrevious) {
  return value.split(',');
}

program
  .option('-i, --integer <number>', 'integer argument', myParseInt)
  .option('-v, --verbose', 'verbosity that can be increased', increaseVerbosity, 0)
  .option('-c, --collect <value>', 'repeatable value', collect, [])
  .option('-l, --list <items>', 'comma separated list', commaSeparatedList)
;

program.parse(process.argv);

if (program.integer !== undefined) console.log(`integer: ${program.integer}`);
if (program.verbose > 0) console.log(`verbosity: ${program.verbose}`);
if (program.collect.length > 0) console.log(program.collect);
if (program.list !== undefined) console.log(program.list);

输出：
$ custom --integer 2
integer: 2
$ custom -v -v -v
verbose: 3
$ custom -c a -c b -c c
[ 'a', 'b', 'c' ]
$ custom --list x,y,z
[ 'x', 'y', 'z' ]
```

## Version option
可选的 version 方法添加了显示命令版本的处理。默认 option 标识是-V和--version，如果存在，该命令将打印版本号并退出。
```javascript
program.version('0.0.1');

$ ./examples/pizza -V
    0.0.1
```
您可以通过使用与option方法相同的语法将其他参数传递给version方法来指定自定义标识。version标志可以命名为任何名称，但需要长名称。

## Command-specific options
您可以将option附加到command上。
```javascript
#!/usr/bin/env node

var program = require('commander');

program
  .command('rm <dir>')
  .option('-r, --recursive', 'Remove recursively')
  .action(function (dir, cmd) {
    console.log('remove ' + dir + (cmd.recursive ? ' recursively' : ''))
  })

program.parse(process.argv)
```
使用该命令时，将验证命令的 option。任何未知选项都将报告为错误。但是，如果基于操作的命令未定义操作，则不验证选项。

## Regular Expression
## Variadic arguments
命令的最后一个参数可以是variadic（可变的），但只能是最后一个参数。要使参数变量可变，您必须将 ... 附加到参数名称后。这是一个例子：

```javascript
#!/usr/bin/env node

/**
 * Module dependencies.
 */

var program = require('commander');

program
  .version('0.1.0')
  .command('rmdir <dir> [otherDirs...]')
  .action(function (dir, otherDirs) {
    console.log('rmdir %s', dir);
    if (otherDirs) {
      otherDirs.forEach(function (oDir) {
        console.log('rmdir %s', oDir);
      });
    }
  });

program.parse(process.argv);
```
将数组形式用于可变参数的值。这适用于 program.args 以及传递给您的操作的参数，如上所示。

## Specify the argument syntax
```javascript
#!/usr/bin/env node

var program = require('commander');

program
  .version('0.1.0')
  .arguments('<cmd> [env]')
  .action(function (cmd, env) {
     cmdValue = cmd;
     envValue = env;
  });

program.parse(process.argv);

if (typeof cmdValue === 'undefined') {
   console.error('no command given!');
   process.exit(1);
}
console.log('command:', cmdValue);
console.log('environment:', envValue || "no environment given");
```
尖括号（例如<cmd>）表示必需的输入。方括号（例如[env]）表示可选输入。

## Git-style sub-commands
```javascript
// file: ./examples/pm
var program = require('commander');

program
  .version('0.1.0')
  .command('install [name]', 'install one or more packages')
  .command('search [query]', 'search with optional query')
  .command('list', 'list packages installed', {isDefault: true})
  .parse(process.argv);
```
当调用.command()同时使用描述参数时，不应调用.action（回调）来处理子命令，否则会出错。这告诉 commander 你将为子命令使用单独的可执行文件，就像git(1)和其他流行的工具一样。<br />commander 将尝试使用 program-command 搜索脚本目录中的可执行文件（如./examples/pm），如pm-install，pm-search。<br />可以通过调用.command()传递选项。为 opts.noHelp 指定 true 将从生成的帮助输出中删除子命令。如果未指定其他子命令，则为opts.isDefault指定true的话将运行子命令。

### `--harmony`
可以通过两种方式使用--harmony选项

- 使用 `#! /usr/bin/env node --harmony` 在子命令脚本中。
- 调用命令时使用--harmony选项，如node --harmony examples / pm publish。产生子命令进程时将保留--harmony选项。

## Automated --help
帮助信息是根据 commander 已知的有关程序的信息自动生成的，因此以下--help信息for free：
```javascript
$ ./examples/pizza --help
Usage: pizza [options]

An application for pizzas ordering

Options:
  -h, --help           output usage information
  -V, --version        output the version number
  -p, --peppers        Add peppers
  -P, --pineapple      Add pineapple
  -b, --bbq            Add bbq sauce
  -c, --cheese <type>  Add the specified type of cheese [marble]
  -C, --no-cheese      You do not want any cheese
```

## Custom help
您可以通过监听“--help”来显示任意-h，-help信息。一旦完成，Commander将自动退出，以便程序的其余部分不会执行从而导致意外行为，例如，在使用--help时，以下可执行文件“stuff”将不会输出
```javascript
#!/usr/bin/env node

/**
 * Module dependencies.
 */

var program = require('commander');

program
  .version('0.1.0')
  .option('-f, --foo', 'enable some foo')
  .option('-b, --bar', 'enable some bar')
  .option('-B, --baz', 'enable some baz');

// must be before .parse() since
// node's emit() is immediate

program.on('--help', function(){
  console.log('')
  console.log('Examples:');
  console.log('  $ custom-help --help');
  console.log('  $ custom-help -h');
});

program.parse(process.argv);

console.log('stuff');
```
运行 `node script-name.js -h` 或者 `node script-name.js --help`
```javascript
Usage: custom-help [options]

Options:
  -h, --help     output usage information
  -V, --version  output the version number
  -f, --foo      enable some foo
  -b, --bar      enable some bar
  -B, --baz      enable some baz

Examples:
  $ custom-help --help
  $ custom-help -h
```

## .outputHelp(cb)(与.help(cb)相反)
输出帮助信息而不退出。可选的回调 cb 允许在显示帮助文本之前对其进行后处理。如果要在默认情况下显示帮助（例如，如果未提供命令），则可以使用以下内容：
```javascript
var program = require('commander');
var colors = require('colors');

program
  .version('0.1.0')
  .command('getstream [url]', 'get stream URL')
  .parse(process.argv);

if (!process.argv.slice(2).length) {
  program.outputHelp(make_red);
}

function make_red(txt) {
  return colors.red(txt); //display the help text in red on the console
}
```

## Custom event listeners
您可以通过监听命令和选项事件来执行自定义操作。
```javascript
program.on('option:verbose', function () {
  process.env.VERBOSE = this.verbose;
});

// error on unknown commands
program.on('command:*', function () {
  console.error('Invalid command: %s\nSee --help for a list of available commands.', program.args.join(' '));
  process.exit(1);
});
```

# API
> [http://tj.github.io/commander.js/](http://tj.github.io/commander.js/)

## option()
```javascript
function Option(flags, description) {
  this.flags = flags;
  this.required = ~flags.indexOf('<'); // 必填参数
  this.optional = ~flags.indexOf('['); // 可选参数
  this.bool = !~flags.indexOf('-no-'); // -no-xx 默认true
  flags = flags.split(/[ ,|]+/);
  if (flags.length > 1 && !/^[[<]/.test(flags[1])) this.short = flags.shift(); // 短命名
  this.long = flags.shift(); // 长命名
  this.description = description || ''; // 描述
}
```

### Command(name)

#### Command#command()
.action 回调在command name 通过 argv 命名时调用，其余的参数会直接应用于函数中。当 name 为 '*'（通配符）时，一个un-matched命令会被作为第一个参数进行传输。
```javascript
Command.prototype.command = function(name, desc, opts) {
  opts = opts || {};
  var args = name.split(/ +/);
  var cmd = new Command(args.shift());

  if (desc) {
    cmd.description(desc);
    this.executables = true;
    this._execs[cmd._name] = true;
    if (opts.isDefault) this.defaultExecutable = cmd._name;
  }

  cmd._noHelp = !!opts.noHelp;
  this.commands.push(cmd);
  cmd.parseExpectedArgs(args);
  cmd.parent = this;

  if (desc) return this;
  return cmd;
};
```

#### Command#arguments()
为顶层command配置arguments
#### Command#parseExpectedArgs()
格式化参数
#### Command#action()
给命令注册callback
#### Command#option()
通过flags，description和可选的协定 fn 定义option。flags 字符串应该同时包括长短两种标识，通过逗号，分割或者空格进行区分。下面三种方式都是合理的，在使用--help的时候可以按格式输出：<br />"-p, --pepper"<br />"-p|--pepper"<br />"-p --pepper"
#### Command#allowUnknownOption()
所有命令行重的未知options
#### Command#parse()
转换 argv，设置options 并执行相应命令。
#### Command#description()
设置 description
#### Command#usage(str)
设置/获得命令使用的str
#### Command#name()
返回command的name
#### Command#outputHelp()
输出command的help信息
#### Command#help()
输出help信息并退出
