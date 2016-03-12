# 使用 Hugo 生成 API 接口

随着 `SSG` 静态生成工具的蓬勃发展，市面上能看到越来越多的静态站点，一般的使用方法是通过静态展点生成工具生成静态页面，然后进行发布。

我个人使用了[一年多](https://soulteary.com/about-site/)的 `Hugo` ，不论是稳定性还是易用性方面，都无愧于开源社区里关注度第一。

但是这类静态站点生成器就只能做一些静态站点使用了么？当然不是。

`Hugo` 的自定义输出功能，搭配模板生成，可以轻松输出一些静态 `API` 接口，而内容可以使用 `Markdown` 来进行编写，还允许使用目录树的方式进行管理。

不论是搭配静态生成站点使用、还是简单提供给外部其他的 `SPA` 应用都是很方便的。

下面就来介绍如何使用 `Hugo` 输出 `API` 接口。

## 声明配置

首先需要在 `config.toml` 中指定各种页面的输出类型为 `json`。

```toml
[outputs]
    page = ["json"]
    home = ["json"]
    teams = ["json"]
    section = ["json"]
    taxonomy = ["json"]
```

## 定义模板

接着

接着需要在  `layouts/_default/` 中创建 `single.json`、 `item.json`、 `list.json` 并使用变量和模板函数编写即可，比如:

```json
{
    "title": "{{ .Title }}",
    "date": "{{ .Date }}",
    "type": "{{ .Type }}",
    "permalink" : "{{ .Permalink }}",
    "summary" : "{{ .Summary }}"
}
``` 

即可将某个 `Markdown` 文件转换为 下面的内容:

```json
{
  "data": {
    "title": "About",
    "date": "2018-02-09 11:47:06 -0500 -0500",
    "type": "page",
    "permalink": "/page/about/index.json",
    "summary": "An API about our school athletes!"
  }
}
```

列表文件、和独立的页面文件同理，可以参考我提供的示例项目代码。

## 调整输出

`Hugo` 输出 `API` 依赖模板，使用的是拼凑的方法，所以输出结果难免会像下面一样，掺杂大量无用的空格字符，甚至有可能包含错误结果。

```json
{
	"data" : 
	{
    "name": "Frank J. Robinson",
    "contact" : "+1 (555) 555 5555",
    "permalink" : "/players/frank-j-robinson/index.json",
    "year" : "junior"
    
    	
    		,"practices" : "Monday, Thursday"
    	
    
    	
    		,"sports" : "soccer, baseball"
    	
    
   	
}

}
```

为了最后产物的可用性（性能、预发正确），需要使用脚本对产物进行额外处理，这里我使用 `Node.js` ：

```js
'use strict';

const {readdirSync, statSync, readFileSync, writeFileSync} = require('fs');
const {join} = require('path');

function getAllFiles(dirPath, ext) {
  function flatten(arr) {
    return arr.reduce((flat, toFlatten) => flat.concat(Array.isArray(toFlatten) ? flatten(toFlatten) : toFlatten), []);
  }

  function scanDir(dirPath, ext) {
    const result = readdirSync(dirPath);
    if (!result.length) return [];
    return result.map((dirName) => {
      const filePath = join(dirPath, dirName);
      if (statSync(filePath).isDirectory()) {
        return scanDir(join(dirPath, dirName), ext);
      } else {
        if (!ext) return filePath;
        if (filePath.lastIndexOf(ext) === filePath.indexOf(ext) && filePath.indexOf(ext) > -1) return filePath;
        return '';
      }
    });
  }

  return flatten(scanDir(dirPath, ext)).filter((file) => file);
}

const allMarkdownFiles = getAllFiles('./public', '.json');
allMarkdownFiles.forEach((item) => {
  try {
    const jsonData = JSON.stringify(JSON.parse(readFileSync(item, 'utf8')));
    writeFileSync(item, jsonData);
  } catch (e) {
    console.error(`${item} content error.`);
    writeFileSync(item, JSON.stringify({code: 500, desc: 'Unexpected token'}));
  }
});
```

将脚本保存为 `checker.js` 执行 `node checker` 即可批量对 `Hugo` 的产物进行正确性校验和结果压缩。

比如上面的产物在执行完毕处理脚本后会变成：

```json
{"data":{"name":"Frank J. Robinson","contact":"+1 (555) 555 5555","permalink":"/players/frank-j-robinson/index.json","year":"junior","practices":"Monday, Thursday","sports":"soccer, baseball"}}
```

把这个脚本配置到 CI 中，即可完成修改 `Markdown` 文件，就能够自动生成高性能可用的静态 `API` 了。

## 其他

相关示例代码，可以访问：

- [Hugo API Maker](https://github.com/soulteary/hugo-api-maker)

Hugo 还有一堆其他的玩法，后面有机会再聊。

-EOF