# 如果不用 Node.js 写业务

最近整理博客，发现很久没有介绍语言相关的小用法了，正巧休息的时候把代码仓库归置了一遍，用几个简短的例子，聊聊 `Node.js` 除了写业务脚本、做构建运行时，它还能做些什么有趣的事情吧。

## 持续集成中的粘合剂

在做持续集成中，偶尔会遇到需要解析 API 结果，或者读取文件，获取文件指定数据的需求，当使用常规的 `shell` 难以完成需求的时候，相比较 Python 来说使用 Node 做为粘合剂来获取数据不失为一个好的方案，因为写出的代码将会更简单明了。

### 示例

比如你希望获取到某些 API 中的特殊字段，然后再次进行参数拼装，请求其他的 API ，完成某种程度的完全自动化操作。

使用 `Node.js` 的 `fs.readFileSync` 方法读取 `/dev/stdin` 即可，比如：

```bash
curl SOME_API_WITH_ARGS | node -e 'console.log(JSON.parse(require("fs").readFileSync("/dev/stdin").toString()).data);'
```

当然，你也可以把这个脚本保存为文件，然后进行调用，结果是一样的。

```js
const fs = require("fs");console.log(JSON.parse(fs.readFileSync("/dev/stdin").toString()).iid);
```

如果将上面的代码保存为 `pipe.js`，那么接下来执行下面的代码，将会得到相同的结果。

```bash
echo data | node pipe.js
```

## 辅助进行数据清理

一提到数据清理，大家会不自觉地想到 `SQL`、`Python`、`R`，但是有的时候，并不需要进行大量编码，使用系统内置的 `Shell` 配合 `Node.js` 在实现和效率上都会变的更快。

### 示例

程序构建完毕，想看看到底有多少内容是冗余的，那么便要进行文件去重操作。通常我们并不能够简单依赖文件名称进行去重，这个时候就要使用文件内容签名来做了。

使用静态编译的 shell 工具可以快速计算出文件的签名：

```text
find . | xargs -I {} shasum -a 256 {}

shasum: ./dist: 
shasum: ./dist/some: 
b0a635b4d6be1a6f14f25c64e3ff6fa2ccacc4e55be93d6f542184a386f748c8  ./dist/some/aea0d6c054cf3fadf5bb.js
9498cb1451fc6d02612610f109ebb29dc94957e2a2741b701443182573837bfc  ./dist/some/
```

这个输出结果会有许多看起来无意义的行，这个时候配合上一小节的方法，将输出结果当参数传递给 Node.js 程序只需要几行代码就能将文件分析完毕。


```js
const fs = require('fs');

const shasumList = fs.readFileSync('/dev/stdin').toString().split('\n').filter(l => l).filter(l => l.match(/^\w{64}\s+\S+$/));

let reduceResult = {};

shasumList.map(item => {
  const [shasum, name] = item.match(/(\w+)\s+(\S+)/);
  return {shasum, name};
}).forEach(item => {
  reduceResult[item.shasum] = reduceResult[item.shasum] || [];
  reduceResult[item.shasum].push(item.name);
});

console.log(reduceResult);
```

当然，如果你愿意的话，写个简单的目录扫描，然后使用 `Node.js` 的 `crypto` 模块直接计算签名也是可以的。

```js
const fs = require('fs');
const crypto = require('crypto');
const secret = '';

const hash = crypto.createHmac('sha256', secret).update(fs.readFileSync(`${item.name}-${item.id}.html`).toString()).digest('hex');
```

## 柯里化外部操作

标题看起来是不是很熟悉，一般提到柯里化，我们会想到使用更少的参数去封装后面的高阶函数。使用 `Node.js` 可以提供类似的功能，只不过这里的 `curry` 发生的级别不是代码级别。

### 示例

比如我找到了一些零几年古老的代码片段，它们原本不存在于 CVS 版本控制系统中，倘若直接添加，提交时间就是当下提交的时刻。但是这些文件本身提供了创建时间，可以作为原始信息一并入库。

这个时候，使用 `git commit --date` 可以修改提交时间戳，但是实际使用起来却很麻烦，因为它要保持一个特定的格式：`ddd, DD MMM YYYY HH:MM:SS +8000`。

如果这里有一个小脚本可以将输入的时间自动转换成这个格式，那么处理这些文件就方便多了。

```js
#!/usr/bin/env node

var moment = require('moment');

var argvs = process.argv.slice(2);

console.log('input:', argvs[0]);

if (!argvs.length) {
  console.log('useage: tool timestamp');
  process.exit(1);
}

console.log('output:\n');

if (argvs.length === 2) {
  console.log('git add .');
  console.log('gc -m "' + argvs[0] + '" --date="' + moment(new Date(argvs[1])).format('ddd, DD MMM YYYY HH:MM:SS +8000') + '"');
  console.log('git push');
} else {
  console.log('gc --amend -m "' + moment(new Date(argvs[0])).format('ddd, DD MMM YYYY HH:MM:SS +8000') + '" --date="' + moment(new Date(argvs[0])).format('ddd, DD MMM YYYY HH:MM:SS +8000') + '"');
  console.log('git push -f');
}
```

当然，如果你将这个脚本保存到一个全局可以找到的路径，然后赋予执行权限，还可以在脚本中添加 `exec` 操作，让代码能够自动提交。

## 辅助文件管理

除了作为数据管理的工具外，`Node.js` 提供的文件操作模块，还能辅助我们进行文件整理。

### 示例

在购买了 `XX音乐包`、`XX会员` 之后，发现歌曲音质确实好了不少，但是音质的代价是不得不将大量文件缓存在移动设备中，即使用着 `256G` 的移动设备，也不堪长年累月的缓存之重。于是我把设备中的音乐导出、也从其他地方下载了一些音乐，计划将它们同一归置到一台专门的设备中进行管理存放，但是几千首随机生成的文件名十分不利于归档，好在多数文件中包含了音乐信息 `ID3`，配合解析软件包，让程序自动将他们重新命名和进程存放吧。

```js
const {readdirSync, renameSync, statSync, unlinkSync, existsSync, mkdirSync} = require('fs');
const {join} = require('path');
const {inspect} = require('util');
const {parseFile} = require('music-metadata');

const {rootDir} = require('./config');

const files = readdirSync(rootDir).
    filter((file) => file.endsWith('.m4a') || file.endsWith('.flac') || file.endsWith('.mp3'));

files.forEach((file) => {
  const fullPath = join(rootDir, file);

  parseFile(fullPath, {native: true}).then((metadata) => {
    const name = `${metadata.common.title.trim()} - ${metadata.common.artist.trim()}.${file.split('.').pop()}`;

    if (name !== file) {
      renameSync(fullPath, join(rootDir, name));
    }

  }).catch((err) => {
    if (statSync(fullPath).size < 10) {
      unlinkSync(fullPath);
    } else {
      console.log('parse error', statSync(fullPath).size);
      console.error(err.message);
    }
  });
});

readdirSync(rootDir).filter((file) => {
  if (file.includes(' - ')) {
    let lastPart = file.split(' - ')[1];
    const name = lastPart.replace(/\..+/, '');

    const target = join(rootDir, name);

    if (!existsSync(target)) mkdirSync(target);
    renameSync(join(rootDir, file), join(target, file));
  }
});
```

## 其他

像上面使用 `Node` 只写十几行代码就能完成任务的事情还有很多，比如之前：

- 给公司做过智能设备网关，监听公司大门是否忘记关闭，使用企业微信接口快速通知负责人。
- 编写简单 UDP 脚本控制家里的各种设备自动休眠和唤醒。
- 一些游戏存档修改器 XD
- 快速爬取任何浏览过的网站。




