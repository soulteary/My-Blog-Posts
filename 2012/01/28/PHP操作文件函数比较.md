# PHP操作文件函数比较

[这篇](http://promiseforever.com/redirect?url=http%3A%2F%2Fwww.raditha.com%2Fwiki%2FReadfile_vs_include&key=bfc41556fd458dee9180d40b4ffde61e)真心不是我要的,但是作为资料,备份下来吧. 

前面的文字性质的描述就直接省略了,有兴趣的直接点击文章开头的链接,访问原文. 从手册上摘录下列的方法，大致是6种，但是印象中还有使用服务器组件进行操作的方法。

```php
string file_get_contents ( string $filename [, bool $use_include_path = false [, resource $context [, int $offset = -1 [, int $maxlen ]]]] )
```

file_get_contents() is the preferred way to read the contents of a file into a string. It will use memory mapping techniques if supported by your OS to enhance performance.

```php
int fpassthru ( resource $handle )
```

Reads to EOF on the given file pointer from the current position and writes the results to the output buffer. You may need to call rewind() to reset the file pointer to the beginning of the file if you have already written data to the file. If you just want to dump the contents of a file to the output buffer, without first modifying it or seeking to a particular offset, you may want to use the readfile(), which saves you the fopen() call.

```php
string fgets ( resource $handle [, int $length ] )
```

Gets a line from file pointer.

```php
array file ( string $filename [, int $flags = 0 [, resource $context ]] )
```

Reads an entire file into an array.

```php
require(string filename)
include(string filename)
require_once(string filename)
include_once(string filename)
```

includes and evaluates the specific file.

```php
int readfile ( string $filename [, bool $use_include_path = false [, resource $context ]] )
```

Reads a file and writes it to the output buffer. 下面将老外用1MB文件来进行测试的结果也贴一下,有兴趣的朋友补充测试.

| Function | Sample Usage | Time (s) | Memory (b) |
| --- | --- | --- | --- |
| file_get_contents | `echo file_get_contents($filename);` | 0.00564 | 1067856 |
| fpassthru | `fpassthru($fp);` | **0.00184** | 20032 |
| fgets | `$fp = fopen($filename,"rb"); while(!feof($fp)) { echo fgets($fp); }` | 0.07190 | 30768 |
| file | `echo join("",file($filename));` | 0.06464 | 2185624 |
| require_once | `require_once($filename);` | 0.08065 | 2067696 |
| include | `include($filename);` | 0.08202 | 2067696 |
| readfile | `readfile($filename);` | 0.00191 | **19208** |

接着是32KB的小文件处理上

| Function | Time (s) | Memory (b) |
| --- | --- | --- |
 32Kb File | 1Mb File | 32Kb File | 1Mb File |
| file_get_contents | 0.00152 | 0.00564 | 52480 | 1067856 |
| fpassthru | **0.00117** | 0.00184 | 20016 | 20032 |
| fgets | 0.00195 | 0.07190 | 30760 | 30768 |
| file | 0.00157 | 0.06464 | 87344 | 2185624 |
| require_once | 0.00225 | 0.08065 | 67992 | 2067696 |
| include | 0.00222 | 0.08202 | 67928 | 2067624 |
| readfile | **0.00117** | 0.00191 | **19192** | 19208 |

用上面的结果的话,readfile 和fpassthru 是我们最佳的选择.

