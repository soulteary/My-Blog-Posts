# Mac,Win,Linux清除DNS缓存

<!-- more -->

Mac OS X：

```
dscacheutil -flushcache
```

Linux：

```
/etc/init.d/nscd restart
```

Windows:

```
ipconfig /flushdns
```

如果你想知道当前的DNS缓存

```
ipconfig /displaydns
```

