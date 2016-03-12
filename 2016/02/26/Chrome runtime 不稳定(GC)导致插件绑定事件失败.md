# Chrome runtime 不稳定（GC）导致插件绑定事件失败

最近在重构插件，把之前和现在遇到的问题都记录一下吧，抛砖引玉，不对的地方，欢迎指正，: )

## 问题表现

插件在加载后（安装、插件页面重载、插件内部runtime重载），进行初始化时，概率性绑定事件遇到错误阻塞执行或者阻碍后续逻辑执行。

> TypeError: Cannot read property 'onBeforeSendHeaders' of undefined(…)

尤其是当你刷新调试页面时，提示报错页面由你的页面不停的变化成各种内部脚本等：

> extensions::guestViewEvents extensions::runtime

## 解决方案

在尝试简化代码、onload后执行，DOMLOADED后执行，onInstall后执行均无效后，使用try-catch容错方案对插件进行重载尝试，因插件是概率性出错，故可以通过插件快速自动重载来避免功能不可用。 示例代码

```js
function init () {
    try {
        Debug.log(debugModuleName, '你的初始化代码');

        chrome.webRequest.onBeforeSendHeaders...
        chrome.tabs.onUpdated.addListener...

    } catch (e) {
        Debug.error('插件加载出现错误，正在重载。', e);
        location.reload();
    }
}

document.addEventListener('DOMContentLoaded', init);
```


