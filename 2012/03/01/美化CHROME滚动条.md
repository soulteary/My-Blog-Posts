# 美化CHROME滚动条

原文地址:http://www.douban.com/group/topic/27738925/

<!-- more -->

找到你的CHROME安装路径 在 Google\Chrome\User Data\Default\User StyleSheets\Custom.css 里输入以下CSS即可. 

如果实在找不到,且使用WINXP可以考虑找这个位置 **"C:\Documents and Settings\你的用户名\Local Settings\Application Data\Google\Chrome\User Data\Default\User StyleSheets"**

```css
::-webkit-scrollbar-track-piece{ 
background-color:#fff; 
} 
::-webkit-scrollbar{ 
width:11px; 
height:11px; 
} 
::-webkit-scrollbar-thumb{ 
height:50px; 
background-color:#cbcbcb; 
outline:0px solid #fff; 
} 
::-webkit-scrollbar-thumb:hover{ 
height:50px; 
background-color:#989898; 
}
```

