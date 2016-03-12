# 更新Dash文档的小技巧

Dash估计蛮多童鞋都在使用，可能是用户数量实在众多，也可能是程序下载功能写的蛮烂的，所以很多时候，想拖个新文档下来，尤其是哪种百M以上的文件，诸如drupal、Android、WordPress、JavaScript之类（PHP文档打包后才60多M，实在是小意思啦，什么，你PHP也更新不下来啊，那你更需要好好看看本文了）。

本文介绍一种取巧的方法，可以帮助你愉快的将文档进行更新！

## 问题背景

作者为我们提供了若干台服务器，有旧金山的、日本的、悉尼的、伦敦的、纽约的，但是实际上除了日本的持续连接速度尚可，其他的都是坑爹的，尤其是长时间下载的时候，下着下着就没速度了，然后程序就卡死了（一定是写的时候木有处理异常啊）。

## 解决方案

既然知道是网络问题了，那么解决方法无非是：
 
1. 使用各种网络加速手段，加速程序下载过程。 
2. 使用下载工具从速度快的服务器把文档搞下来，然后本地导入进去。 

方案一实施之后发觉还是太慢了，于是采取方案二。

首先要可以获取文档地址，怎么获取，还记得下载错误的时候会提示一个错误信息么（当然，抓包或者防火墙的话，就更简单了）。

```text
Error Installing Docset

An error occured while installing the docset. The error message is: 

Could not connect to the server.

The feed URL is http://kapeli.com/feeds/{{docs—name}}.xml.

Check your Internet connection and try again. Please contact the developer if you cannot recover from this error.
```

想要知道这个报错怎么出来么，当然是人为干涉，不让他正常下载成功啊。修改你本地的hosts：

```text
## Dash Docs Download
127.0.0.1 newyork3.kapeli.com newyork2.kapeli.com newyork.kapeli.com
127.0.0.1 london3.kapeli.com london2.kapeli.com london.kapeli.com
127.0.0.1 sanfrancisco3.kapeli.com sanfrancisco2.kapeli.com sanfrancisco.kapeli.com
127.0.0.1 sydney3.kapeli.com sydney2.kapeli.com sydney.kapeli.com
127.0.0.1 tokyo3.kapeli.com tokyo2.kapeli.com tokyo.kapeli.com
```

等报错后，直接打开这个xml地址，你会看到这个xml中包含了好多可选的下载地址，这里我一律选择的是linode tokyo2节点。（其实你把xml后缀换成tgz就好了...）

然后使用下载工具下载，接着本地使用apache/nodejs/nginx做代理，并再次修改任意hosts到你的本地，并点击dash程序中的download就大功告成了。

PS:记得保持和线上一样的目录结构（feeds目录下）/如非必要，暂时关闭自动更新文档。

```text
➜  tokyo.kapeli.com  tree -L 3
.
├── conf
│   └── nginx.conf
├── logs
├── public
│   └── feeds
│       ├── ActionScript.tgz
│       ├── Android.tgz
│       ├── Appcelerator_Titanium.tgz
│       ├── Bash.tgz
│       ├── C++.tgz
│       ├── C.tgz
│       ├── Cocos2D-x.tgz
│       ├── Cocos2D.tgz
│       ├── Cocos3D.tgz
│       ├── CoffeeScript.tgz
```

