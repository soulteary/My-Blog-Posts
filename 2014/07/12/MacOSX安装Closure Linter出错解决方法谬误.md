# MacOSX安装Closure Linter出错解决方法谬误

最近开始对权限敏感起来，或许是因为NPM使用了内部私有NPM包管理工具的缘故吧，直接执行[官方说明](https://developers.google.com/closure/utilities/docs/linter_howto)，会报错。 场景重现：

```bash
➜  ~  easy_install XXX
error: can't create or remove files in install directory

The following error occurred while trying to add or remove files in the
installation directory:

    [Errno 13] Permission denied: '/Library/Python/2.7/site-packages/test-easy-install-7472.write-test'

The installation directory you specified (via --install-dir, --prefix, or
the distutils default setting) was:

    /Library/Python/2.7/site-packages/

Perhaps your account does not have write access to this directory?  If the
installation directory is a system-owned directory, you may need to sign in
as the administrator or "root" account.  If you do not have administrative
access to this machine, you may wish to choose a different installation
directory, preferably one that is listed in your PYTHONPATH environment
variable.

For information on other options, you may wish to consult the
documentation at:

  http://peak.telecommunity.com/EasyInstall.html

Please make the appropriate changes for your system and try again.
```

网上的解决方案是：

```bash
chmod +a 'user:YOUR_USER_NAME allow add_subdirectory,add_file,delete_child,directory_inherit' /Library/Python/2.7/site-packages
```

这个YOUR_USER_NAME，使用whoami可以轻易得到，如果你确实不知道的话。 但是发现：

```bash
chmod +a 'user:soulteary allow add_subdirectory,add_file,delete_child,directory_inherit' /Library/Python/2.7/site-packages
chmod: Failed to set ACL on file '/Library/Python/2.7/site-packages': Operation not permitted
```

sudo执行上面这条语句，似乎的可以安装了，但是却发现：

```bash
error: /Library/Python/2.7/site-packages/easy-install.pth: Permission denied
```

所以还是乖乖的：

```bash
sudo easy_install http://closure-linter.googlecode.com/files/closure_linter-latest.tar.gz
```

安装结果： `Installed /Library/Python/2.7/site-packages/python_gflags-2.0-py2.7.egg Finished processing dependencies for closure-linter==2.3.13`

