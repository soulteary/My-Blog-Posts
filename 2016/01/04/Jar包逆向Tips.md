# Jar包逆向Tips

最近做Java程序逆向，简单记个笔记。

## 笔记正文

使用`jar`命令来处理jarball比使用zip软件靠谱，虽然本质就是压缩包。

诸如:

```bash
// 提取文件
jar -xvf filename.jar com/path/any.class
// 更新文件
jar -uvf filename.jar com/path/any.class
```

逆向后的java文件，需要重新编译，建议使用`META-INF/MANIFEST.MF`文件的`Build-Jdk`。

编译可以使用保守的1.6版本的SDK，当然版本选择不要高于Build-Jdk（也可选择Build-Jdk），如：

```bash
javac -source 1.6 -target 1.6 -d . any.java
```

最后，逆向的文件，如果不搞keygen之类的东西，直接剔除private函数，处理public导出函数，酌情处理会被子类继承的protect函数就好（如果不存在集成，仅保留public即可）。

