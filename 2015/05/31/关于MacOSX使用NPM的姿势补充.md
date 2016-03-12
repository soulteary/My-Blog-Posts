# 关于MacOSX使用NPM的姿势补充

经常看到有人说『为啥npm install 的时候报错，显示EACCESS错误...』，之前大家都是`sudo`大法解决问题，也没太在意。

至于这个问题是brew安装工具的时候造成的，还是系统修改磁盘权限造成的，还是安装各种小工具的时候造成的不得而知...（这个实在懒得追究了）

## 问题背景

最近在搞generator的时候，如果不想把一些文件包含在generator中，那么会调用`npm install`，所以会遇到报错。

## 解决方案

NPM 维护者的解决方案是： https://github.com/npm/npm/issues/5922

```bash
sudo chown -R `whoami` /usr/local
```

不过似乎忘记清除缓存了，个人建议把组权限也修改掉，当然，如果有洁癖，可以干掉`/Users/$(whoami)`下的.npm缓存目录。

```bash
sudo chown -R `whoami`:staff /usr/local
sudo chown -R `whoami`:staff /Users/`whoami`/.npm
```

至于项目中的node_modules，建议直接rm掉重新安装。 至此，愉悦的使用npm/cnpm吧。 补充（如果你身在大中国，npm偶尔速度不佳的话，可以使用大淘宝业界良心的NPM仓库镜像）：

```bash
npm install -g cnpm --registry=http://registry.npm.taobao.org
```

--EOF--

