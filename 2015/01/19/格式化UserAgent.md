# 格式化UserAgent

数据统计和降级实现基础之一，格式化UserAgent。 

代码源自KISSY v1.4.x - seed.js 文件中的[UA模块](https://github.com/kissyteam/kissy/blob/1.4.x/src/seed/src/ua.js)，并稍加修改。

```js
var UA = (function (undefined) {
    /*global process*/

    var win = window,
        doc = win.document,
        navigator = win.navigator,
        ua = navigator && navigator.userAgent || '';

    function numberify(s) {
        var c = 0;
        // convert '1.2.3.4' to 1.234
        return parseFloat(s.replace(/\./g, function () {
            return (c++ === 0) ? '.' : '';
        }));
    }

    function setTridentVersion(ua, UA) {
        var core, m;
        UA[core = 'trident'] = 0.1; // Trident detected, look for revision

        // Get the Trident's accurate version
        if ((m = ua.match(/Trident\/([\d.]*)/)) && m[1]) {
            UA[core] = numberify(m[1]);
        }

        UA.core = core;
    }

    function getIEVersion(ua) {
        var m, v;
        if ((m = ua.match(/MSIE ([^;]*)|Trident.*; rv(?:\s|:)?([0-9.]+)/)) &&
                (v = (m[1] || m[2]))) {
            return numberify(v);
        }
        return 0;
    }

    function getDescriptorFromUserAgent(ua) {
        var EMPTY = '',
            os,
            core = EMPTY,
            shell = EMPTY, m,
            IE_DETECT_RANGE = [6, 9],
            ieVersion,
            v,
            end,
            VERSION_PLACEHOLDER = '{{version}}',
            IE_DETECT_TPL = '<!--[if IE ' + VERSION_PLACEHOLDER + ']><' + 's></s><![endif]-->',
            div = doc && doc.createElement('div'),
            s = [];
        /**
         * KISSY UA
         * @class KISSY.UA
         * @singleton
         */
        var UA = {
            /**
             * webkit version
             * @type undefined|Number
             * @member KISSY.UA
             */
            webkit : undefined,
            /**
             * trident version
             * @type undefined|Number
             * @member KISSY.UA
             */
            trident: undefined,
            /**
             * gecko version
             * @type undefined|Number
             * @member KISSY.UA
             */
            gecko  : undefined,
            /**
             * presto version
             * @type undefined|Number
             * @member KISSY.UA
             */
            presto : undefined,
            /**
             * chrome version
             * @type undefined|Number
             * @member KISSY.UA
             */
            chrome : undefined,
            /**
             * safari version
             * @type undefined|Number
             * @member KISSY.UA
             */
            safari : undefined,
            /**
             * firefox version
             * @type undefined|Number
             * @member KISSY.UA
             */
            firefox: undefined,
            /**
             * ie version
             * @type undefined|Number
             * @member KISSY.UA
             */
            ie     : undefined,
            /**
             * ie document mode
             * @type undefined|Number
             * @member KISSY.UA
             */
            ieMode : undefined,
            /**
             * opera version
             * @type undefined|Number
             * @member KISSY.UA
             */
            opera  : undefined,
            /**
             * mobile browser. apple, android.
             * @type String
             * @member KISSY.UA
             */
            mobile : undefined,
            /**
             * browser render engine name. webkit, trident
             * @type String
             * @member KISSY.UA
             */
            core   : undefined,
            /**
             * browser shell name. ie, chrome, firefox
             * @type String
             * @member KISSY.UA
             */
            shell  : undefined,

            /**
             * PhantomJS version number
             * @type undefined|Number
             * @member KISSY.UA
             */
            phantomjs: undefined,

            /**
             * operating system. android, ios, linux, windows
             * @type string
             * @member KISSY.UA
             */
            os: undefined,

            /**
             * ipad ios version
             * @type Number
             * @member KISSY.UA
             */
            ipad  : undefined,
            /**
             * iphone ios version
             * @type Number
             * @member KISSY.UA
             */
            iphone: undefined,
            /**
             * ipod ios
             * @type Number
             * @member KISSY.UA
             */
            ipod  : undefined,
            /**
             * ios version
             * @type Number
             * @member KISSY.UA
             */
            ios   : undefined,

            /**
             * android version
             * @type Number
             * @member KISSY.UA
             */
            android: undefined,

            /**
             * nodejs version
             * @type Number
             * @member KISSY.UA
             */
            nodejs: undefined
        };

        // ejecta
        if (div && div.getElementsByTagName) {
            // try to use IE-Conditional-Comment detect IE more accurately
            // IE10 doesn't support this method, @ref: http://blogs.msdn.com/b/ie/archive/2011/07/06/html5-parsing-in-ie10.aspx
            div.innerHTML = IE_DETECT_TPL.replace(VERSION_PLACEHOLDER, '');
            s = div.getElementsByTagName('s');
        }

        if (s.length > 0) {

            setTridentVersion(ua, UA);

            // Detect the accurate version
            // 注意：
            //  UA.shell = ie, 表示外壳是 ie
            //  但 UA.ie = 7, 并不代表外壳是 ie7, 还有可能是 ie8 的兼容模式
            //  对于 ie8 的兼容模式，还要通过 documentMode 去判断。但此处不能让 UA.ie = 8, 否则
            //  很多脚本判断会失误。因为 ie8 的兼容模式表现行为和 ie7 相同，而不是和 ie8 相同
            for (v = IE_DETECT_RANGE[0], end = IE_DETECT_RANGE[1]; v <= end; v++) {
                div.innerHTML = IE_DETECT_TPL.replace(VERSION_PLACEHOLDER, v);
                if (s.length > 0) {
                    UA[shell = 'ie'] = v;
                    break;
                }
            }

            // https://github.com/kissyteam/kissy/issues/321
            // win8 embed app
            if (!UA.ie && (ieVersion = getIEVersion(ua))) {
                UA[shell = 'ie'] = ieVersion;
            }

        } else {
            // WebKit
            if ((m = ua.match(/AppleWebKit\/([\d.]*)/)) && m[1]) {
                UA[core = 'webkit'] = numberify(m[1]);

                if ((m = ua.match(/OPR\/(\d+\.\d+)/)) && m[1]) {
                    UA[shell = 'opera'] = numberify(m[1]);
                }
                // Chrome
                else if ((m = ua.match(/Chrome\/([\d.]*)/)) && m[1]) {
                    UA[shell = 'chrome'] = numberify(m[1]);
                }
                // Safari
                else if ((m = ua.match(/\/([\d.]*) Safari/)) && m[1]) {
                    UA[shell = 'safari'] = numberify(m[1]);
                }

                // Apple Mobile
                if (/ Mobile\//.test(ua) && ua.match(/iPad|iPod|iPhone/)) {
                    UA.mobile = 'apple'; // iPad, iPhone or iPod Touch

                    m = ua.match(/OS ([^\s]*)/);
                    if (m && m[1]) {
                        UA.ios = numberify(m[1].replace('_', '.'));
                    }
                    os = 'ios';
                    m = ua.match(/iPad|iPod|iPhone/);
                    if (m && m[0]) {
                        UA[m[0].toLowerCase()] = UA.ios;
                    }
                } else if (/ Android/i.test(ua)) {
                    if (/Mobile/.test(ua)) {
                        os = UA.mobile = 'android';
                    }
                    m = ua.match(/Android ([^\s]*);/);
                    if (m && m[1]) {
                        UA.android = numberify(m[1]);
                    }
                }
                // Other WebKit Mobile Browsers
                else if ((m = ua.match(/NokiaN[^\/]*|Android \d\.\d|webOS\/\d\.\d/))) {
                    UA.mobile = m[0].toLowerCase(); // Nokia N-series, Android, webOS, ex: NokiaN95
                }

                if ((m = ua.match(/PhantomJS\/([^\s]*)/)) && m[1]) {
                    UA.phantomjs = numberify(m[1]);
                }
            }
            // NOT WebKit
            else {
                // Presto
                // ref: http://www.useragentstring.com/pages/useragentstring.php
                if ((m = ua.match(/Presto\/([\d.]*)/)) && m[1]) {
                    UA[core = 'presto'] = numberify(m[1]);

                    // Opera
                    if ((m = ua.match(/Opera\/([\d.]*)/)) && m[1]) {
                        UA[shell = 'opera'] = numberify(m[1]); // Opera detected, look for revision

                        if ((m = ua.match(/Opera\/.* Version\/([\d.]*)/)) && m[1]) {
                            UA[shell] = numberify(m[1]);
                        }

                        // Opera Mini
                        if ((m = ua.match(/Opera Mini[^;]*/)) && m) {
                            UA.mobile = m[0].toLowerCase(); // ex: Opera Mini/2.0.4509/1316
                        }
                        // Opera Mobile
                        // ex: Opera/9.80 (Windows NT 6.1; Opera Mobi/49; U; en) Presto/2.4.18 Version/10.00
                        // issue: 由于 Opera Mobile 有 Version/ 字段，可能会与 Opera 混淆，同时对于 Opera Mobile 的版本号也比较混乱
                        else if ((m = ua.match(/Opera Mobi[^;]*/)) && m) {
                            UA.mobile = m[0];
                        }
                    }

                    // NOT WebKit or Presto
                } else {
                    // MSIE
                    // 由于最开始已经使用了 IE 条件注释判断，因此落到这里的唯一可能性只有 IE10+
                    // and analysis tools in nodejs
                    if ((ieVersion = getIEVersion(ua))) {
                        UA[shell = 'ie'] = ieVersion;
                        setTridentVersion(ua, UA);
                        // NOT WebKit, Presto or IE
                    } else {
                        // Gecko
                        if ((m = ua.match(/Gecko/))) {
                            UA[core = 'gecko'] = 0.1; // Gecko detected, look for revision
                            if ((m = ua.match(/rv:([\d.]*)/)) && m[1]) {
                                UA[core] = numberify(m[1]);
                                if (/Mobile|Tablet/.test(ua)) {
                                    UA.mobile = 'firefox';
                                }
                            }
                            // Firefox
                            if ((m = ua.match(/Firefox\/([\d.]*)/)) && m[1]) {
                                UA[shell = 'firefox'] = numberify(m[1]);
                            }
                        }
                    }
                }
            }
        }

        if (!os) {
            if ((/windows|win32/i).test(ua)) {
                os = 'windows';
            } else if ((/macintosh|mac_powerpc/i).test(ua)) {
                os = 'macintosh';
            } else if ((/linux/i).test(ua)) {
                os = 'linux';
            } else if ((/rhino/i).test(ua)) {
                os = 'rhino';
            }
        }

        UA.os = os;
        UA.core = UA.core || core;
        UA.shell = shell;
        UA.ieMode = UA.ie && doc.documentMode || UA.ie;

        return UA;
    }

    var UA = getDescriptorFromUserAgent(ua);

    // nodejs
    if (typeof process === 'object') {
        var versions, nodeVersion;

        if ((versions = process.versions) && (nodeVersion = versions.node)) {
            UA.os = process.platform;
            UA.nodejs = numberify(nodeVersion);
        }
    }

    // use by analysis tools in nodejs
    UA.getDescriptorFromUserAgent = getDescriptorFromUserAgent;

    var browsers = [
            // browser core type
            'webkit',
            'trident',
            'gecko',
            'presto',
            // browser type
            'chrome',
            'safari',
            'firefox',
            'ie',
            'opera'
        ],
        documentElement = doc && doc.documentElement,
        className = '';

    var RE_TRIM = /^[\s\xa0]+|[\s\xa0]+$/g,
        EMPTY = '';
    /**
     * Removes the whitespace from the beginning and end of a string.
     * @method
     * @member KISSY
     */
    if (String.prototype.trim) {
        function trim(str) {
            return str == null ? EMPTY : String.prototype.trim.call(str);
        }
    } else {
        function trim(str) {
            return str == null ? EMPTY : (str + '').replace(RE_TRIM, EMPTY);
        }
    }

    if (documentElement) {
        for (var key = 0, j = browsers.length; key < j; key++) {
            var v = UA[key];
            if (v) {
                className += ' ks-' + key + (parseInt(v) + '');
                className += ' ks-' + key;
            }
        }
        if (trim(className)) {
            documentElement.className = trim(documentElement.className + className);
        }
    }
    for (var prop in UA) {
        if (!UA[prop]) {
            delete UA[prop];
        }
    }
    delete UA['getDescriptorFromUserAgent'];
    return UA;
})();

/*
 NOTES:
 2013.07.08 yiminghe@gmail.com
 - support ie11 and opera(using blink)

 2013.01.17 yiminghe@gmail.com
 - expose getDescriptorFromUserAgent for analysis tool in nodejs

 2012.11.27 yiminghe@gmail.com
 - moved to seed for conditional loading and better code share

 2012.11.21 yiminghe@gmail.com
 - touch and os support

 2011.11.08 gonghaocn@gmail.com
 - ie < 10 使用条件注释判断内核，更精确

 2010.03
 - jQuery, YUI 等类库都推荐用特性探测替代浏览器嗅探。特性探测的好处是能自动适应未来设备和未知设备，比如
 if(document.addEventListener) 假设 IE9 支持标准事件，则代码不用修改，就自适应了“未来浏览器”。
 对于未知浏览器也是如此。但是，这并不意味着浏览器嗅探就得彻底抛弃。当代码很明确就是针对已知特定浏览器的，
 同时并非是某个特性探测可以解决时，用浏览器嗅探反而能带来代码的简洁，同时也也不会有什么后患。总之，一切
 皆权衡。
 - UA.ie && UA.ie < 8 并不意味着浏览器就不是 IE8, 有可能是 IE8 的兼容模式。进一步的判断需要使用 documentMode.
 */
```

附送压缩版本。

```js
var UA=function(d){function f(b){var d=0;return parseFloat(b.replace(/\./g,function(){return 0===d++?".":""}))}function q(b,d){var e;d.trident=.1;(e=b.match(/Trident\/([\d.]*)/))&&e[1]&&(d.trident=f(e[1]));d.core="trident"}function r(b){var d,e;return(d=b.match(/MSIE ([^;]*)|Trident.*; rv(?:\s|:)?([0-9.]+)/))&&(e=d[1]||d[2])?f(e):0}function t(b){var e,g="",k="",a,h=[6,9],l,p=n&&n.createElement("div"),m=[],c={webkit:d,trident:d,gecko:d,presto:d,chrome:d,safari:d,firefox:d,ie:d,ieMode:d,opera:d,mobile:d,
core:d,shell:d,phantomjs:d,os:d,ipad:d,iphone:d,ipod:d,ios:d,android:d,nodejs:d};p&&p.getElementsByTagName&&(p.innerHTML="\x3c!--[if IE {{version}}]><s></s><![endif]--\x3e".replace("{{version}}",""),m=p.getElementsByTagName("s"));if(0<m.length){q(b,c);a=h[0];for(h=h[1];a<=h;a++)if(p.innerHTML="\x3c!--[if IE {{version}}]><s></s><![endif]--\x3e".replace("{{version}}",a),0<m.length){c[k="ie"]=a;break}!c.ie&&(l=r(b))&&(c[k="ie"]=l)}else if((a=b.match(/AppleWebKit\/([\d.]*)/))&&a[1]){c[g="webkit"]=f(a[1]);
(a=b.match(/OPR\/(\d+\.\d+)/))&&a[1]?c[k="opera"]=f(a[1]):(a=b.match(/Chrome\/([\d.]*)/))&&a[1]?c[k="chrome"]=f(a[1]):(a=b.match(/\/([\d.]*) Safari/))&&a[1]&&(c[k="safari"]=f(a[1]));if(/ Mobile\//.test(b)&&b.match(/iPad|iPod|iPhone/))c.mobile="apple",(a=b.match(/OS ([^\s]*)/))&&a[1]&&(c.ios=f(a[1].replace("_","."))),e="ios",(a=b.match(/iPad|iPod|iPhone/))&&a[0]&&(c[a[0].toLowerCase()]=c.ios);else if(/ Android/i.test(b))/Mobile/.test(b)&&(e=c.mobile="android"),(a=b.match(/Android ([^\s]*);/))&&a[1]&&
(c.android=f(a[1]));else if(a=b.match(/NokiaN[^\/]*|Android \d\.\d|webOS\/\d\.\d/))c.mobile=a[0].toLowerCase();(a=b.match(/PhantomJS\/([^\s]*)/))&&a[1]&&(c.phantomjs=f(a[1]))}else if((a=b.match(/Presto\/([\d.]*)/))&&a[1])c[g="presto"]=f(a[1]),(a=b.match(/Opera\/([\d.]*)/))&&a[1]&&(c[k="opera"]=f(a[1]),(a=b.match(/Opera\/.* Version\/([\d.]*)/))&&a[1]&&(c[k]=f(a[1])),(a=b.match(/Opera Mini[^;]*/))&&a?c.mobile=a[0].toLowerCase():(a=b.match(/Opera Mobi[^;]*/))&&a&&(c.mobile=a[0]));else if(l=r(b))c[k=
"ie"]=l,q(b,c);else if(a=b.match(/Gecko/))c[g="gecko"]=.1,(a=b.match(/rv:([\d.]*)/))&&a[1]&&(c[g]=f(a[1]),/Mobile|Tablet/.test(b)&&(c.mobile="firefox")),(a=b.match(/Firefox\/([\d.]*)/))&&a[1]&&(c[k="firefox"]=f(a[1]));e||(/windows|win32/i.test(b)?e="windows":/macintosh|mac_powerpc/i.test(b)?e="macintosh":/linux/i.test(b)?e="linux":/rhino/i.test(b)&&(e="rhino"));c.os=e;c.core=c.core||g;c.shell=k;c.ieMode=c.ie&&n.documentMode||c.ie;return c}var e=window,n=e.document,e=e.navigator,e=t(e&&e.userAgent||
"");if("object"===typeof process){var h,g;(h=process.versions)&&(g=h.node)&&(e.os=process.platform,e.nodejs=f(g))}e.getDescriptorFromUserAgent=t;var m="webkit trident gecko presto chrome safari firefox ie opera".split(" ");h=n&&n.documentElement;g="";var x=/^[\s\xa0]+|[\s\xa0]+$/g,u=String.prototype.trim?function(b){return null==b?"":String.prototype.trim.call(b)}:function(b){return null==b?"":(b+"").replace(x,"")};if(h){for(var l=0,m=m.length;l<m;l++){var v=e[l];v&&(g+=" ks-"+l+(parseInt(v)+""),
g+=" ks-"+l)}u(g)&&(h.className=u(h.className+g))}for(var w in e)e[w]||delete e[w];delete e.getDescriptorFromUserAgent;return e}();
```

如果想要一个基本够用的，不妨使用下面的。

```js
var UA=function(d){function f(b){var d=0;return parseFloat(b.replace(/\./g,function(){return 0===d++?".":""}))}function p(b,d){var e;d.trident=.1;(e=b.match(/Trident\/([\d.]*)/))&&e[1]&&(d.trident=f(e[1]));d.core="trident"}function q(b){var d,e;return(d=b.match(/MSIE ([^;]*)|Trident.*; rv(?:\s|:)?([0-9.]+)/))&&(e=d[1]||d[2])?f(e):0}var e=window,r=e.document,e=e.navigator,e=function(b){var e,g="",h="",a,l=[6,9],m,k=r&&r.createElement("div"),n=[],c={webkit:d,trident:d,gecko:d,presto:d,chrome:d,safari:d,
firefox:d,ie:d,opera:d,mobile:d,core:d,shell:d,phantomjs:d,os:d,ipad:d,iphone:d,ipod:d,ios:d,android:d,nodejs:d};k&&k.getElementsByTagName&&(k.innerHTML="\x3c!--[if IE {{version}}]><s></s><![endif]--\x3e".replace("{{version}}",""),n=k.getElementsByTagName("s"));if(0<n.length){p(b,c);a=l[0];for(l=l[1];a<=l;a++)if(k.innerHTML="\x3c!--[if IE {{version}}]><s></s><![endif]--\x3e".replace("{{version}}",a),0<n.length){c[h="ie"]=a;break}!c.ie&&(m=q(b))&&(c[h="ie"]=m)}else if((a=b.match(/AppleWebKit\/([\d.]*)/))&&
a[1]){c[g="webkit"]=f(a[1]);(a=b.match(/OPR\/(\d+\.\d+)/))&&a[1]?c[h="opera"]=f(a[1]):(a=b.match(/Chrome\/([\d.]*)/))&&a[1]?c[h="chrome"]=f(a[1]):(a=b.match(/\/([\d.]*) Safari/))&&a[1]&&(c[h="safari"]=f(a[1]));if(/ Mobile\//.test(b)&&b.match(/iPad|iPod|iPhone/))c.mobile="apple",(a=b.match(/OS ([^\s]*)/))&&a[1]&&(c.ios=f(a[1].replace("_","."))),e="ios",(a=b.match(/iPad|iPod|iPhone/))&&a[0]&&(c[a[0].toLowerCase()]=c.ios);else if(/ Android/i.test(b)){if(/Mobile/.test(b)&&(e=c.mobile="android"),(a=b.match(/Android ([^\s]*);/))&&
a[1])c.android=f(a[1])}else if(a=b.match(/NokiaN[^\/]*|Android \d\.\d|webOS\/\d\.\d/))c.mobile=a[0].toLowerCase();(a=b.match(/PhantomJS\/([^\s]*)/))&&a[1]&&(c.phantomjs=f(a[1]))}else if((a=b.match(/Presto\/([\d.]*)/))&&a[1]){if(c[g="presto"]=f(a[1]),(a=b.match(/Opera\/([\d.]*)/))&&a[1])c[h="opera"]=f(a[1]),(a=b.match(/Opera\/.* Version\/([\d.]*)/))&&a[1]&&(c[h]=f(a[1])),(a=b.match(/Opera Mini[^;]*/))&&a?c.mobile=a[0].toLowerCase():(a=b.match(/Opera Mobi[^;]*/))&&a&&(c.mobile=a[0])}else if(m=q(b))c[h=
"ie"]=m,p(b,c);else if(a=b.match(/Gecko/))c[g="gecko"]=.1,(a=b.match(/rv:([\d.]*)/))&&a[1]&&(c[g]=f(a[1]),/Mobile|Tablet/.test(b)&&(c.mobile="firefox")),(a=b.match(/Firefox\/([\d.]*)/))&&a[1]&&(c[h="firefox"]=f(a[1]));e||(/windows|win32/i.test(b)?e="windows":/macintosh|mac_powerpc/i.test(b)?e="macintosh":/linux/i.test(b)?e="linux":/rhino/i.test(b)&&(e="rhino"));c.os=e;c.core=c.core||g;c.shell=h;return c}(e&&e.userAgent||"");if("object"===typeof process){var g,t;(g=process.versions)&&(t=g.node)&&(e.os=
process.platform,e.nodejs=f(t))}for(g in e)e[g]||delete e[g];return e}();console.log(UA);
```

关于UA，还有几篇老文，或许你会感兴趣。

> *   [混乱的User-Agent](http://www.soulteary.com/2009/03/31/story-about-user-agent.html)
> *   [走进User-Agent – 第一节(认知)](http://www.soulteary.com/2009/05/20/close-to-user-agent-a.html)
> *   [走进User-Agent – 第二节(识别)](http://www.soulteary.com/2009/05/20/close-to-user-agent-b.html)
> *   [走进User-Agent – 第三节(伪造)](http://www.soulteary.com/2009/05/20/%E6%95%99%E7%A8%8B%E8%B5%B0%E8%BF%9Buser-agent-%E7%AC%AC%E4%B8%89%E8%8A%82%E4%BC%AA%E9%80%A0.html)

