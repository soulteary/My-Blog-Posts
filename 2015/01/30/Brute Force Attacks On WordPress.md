# Brute Force Attacks On WordPress

Linux今天又爆出了glibc漏洞，早些时候顺手`apt-get update`了，但是发觉涉及一堆软件以及内核，只好`apt-get dist-upgrade`...但是让我感到惊讶的是，除了这个之外，有人在freebuf上发了一篇关于wordpress的漏洞利用，因为类似手段，在2010年的时候就沸沸扬扬了。 建议配合fail2ban的解决方案：

```nginx
location ~ /(wp-admin|wp-login\.php) {
   auth_basic "Hello World";
   auth_basic_user_file pass.conf;
   try_files $uri $uri/ /index.php?q=$uri&$args;
   location ~ \.php$ {
       fastcgi_keep_conn on;
       fastcgi_pass   127.0.0.1:9000;
       fastcgi_index  index.php;
       fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
       include        fastcgi_params;
   }
}
```

原文标题：[WordPress4.0及以下版本Dos攻击漏洞（CVE-2014-9034）的检测和利用](http://www.freebuf.com/vuls/57543.html) WordPress文档：[Brute Force Attacks](http://codex.wordpress.org/Brute_Force_Attacks)

