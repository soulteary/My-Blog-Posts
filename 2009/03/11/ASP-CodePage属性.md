# [ASP]CodePage属性

关于CodePage的属性。在页面编码时需要注意。 codepage属性：指出网页的代码页 如果制作的网页脚本与WEB服务端的默认代码页不同，则必须指明代码页：

```text
codepage=936 简体中文GBK
codepage=950 繁体中文BIG5
codepage=437 美国/加拿大英语
codepage=932 日文
codepage=949 韩文
codepage=866 俄文
codepage=65001 unicode UFT-8
```

常见的问题

```asp
%@LANGUAGE="JAVASCRIPT" CODEPAGE="CP_ACP"?>
```

Active Server Pages, ASP 0203 (0x80004005) 指定的代码页特性无效。 CodePage 查询表

```text
codepage 437 -- 美国/加拿大英语

codepage 737 -- 希腊语

codepage 775 -- 波罗的海语

codepage 850 -- 包括西欧语种(德语,西班牙语,意大利语)中的一些字符

codepage 852 -- Latin 2 包括中东欧语种(阿尔巴尼亚语,克罗地亚语,捷克语,英语,芬兰语,匈牙利语,爱尔兰语,德语,波兰语,罗马利亚语,塞尔维亚语,斯洛伐克语,斯洛文尼亚语,Sorbian语)

codepage 855 -- 斯拉夫语

codepage 857 -- 土耳其语

codepage 860 -- 葡萄牙语

codepage 861 -- 冰岛语

codepage 862 -- 希伯来语

codepage 863 -- 加拿大语

codepage 864 -- 阿拉伯语

codepage 865 -- 日尔曼语系

codepage 866 -- 斯拉夫语/俄语

codepage 869 -- 希腊语(2)

codepage 874 -- 泰语

codepage 936 -- 简体中文GBK

codepage 950 -- 繁体中文Big5

iso8859-1 -- 西欧语系(阿尔巴尼亚语,西班牙加泰罗尼亚语,丹麦语,荷兰语,英语,Faeroese语,芬兰语,法语,德语,加里西亚语,爱尔兰语,冰岛语,意大利语,挪威语,葡萄牙语,瑞士语.)这同时适用于美国英语.

iso8859-2 -- Latin 2 字符集,斯拉夫/中欧语系(捷克语,德语,匈牙利语,波兰语,罗马尼亚语,克罗地亚语,斯洛伐克语,斯洛文尼亚语)

iso8859-3 -- Latin 3 字符集, (世界语,加里西亚语,马耳他语,土耳其语)

iso8859-4 -- Latin 4 字符集, (爱莎尼亚语,拉脱维亚语,立陶宛语),是Latin 6 字符集的前序标准

iso8859-5 -- 斯拉夫语系(保加利亚语,Byelorussian语,马其顿语,俄语,塞尔维亚语,乌克兰语) 一般推荐使用 KOI8-R codepage

iso8859-6 -- 阿拉伯语.

iso8859-7 -- 现代希腊语

iso8859-8 -- 希伯来语

iso8859-9 -- Latin 5 字符集, (去掉了 Latin 1中不经常使用的一些冰岛语字符而代以土耳其语字符)

iso8859-10 -- Latin 6 字符集, (因纽特(格陵兰)语,萨摩斯岛语等Latin 4 中没有包括的北欧语种)

iso8859-15 -- Latin 9 字符集, 是Latin 1字符集的更新版本,去掉一些不常用的字符,增加了对爱莎尼亚语的支持,修正了法语和芬兰语部份,增加了欧元字符)

koi8-r -- 俄语的缺省支持
```

