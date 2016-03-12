# jQuery 事件trigger变更背后的八卦

昨天浏览segmentfault的时候，发现了某个网友的问题回复略失妥当。

于是注册回复了一下，但是仅仅指出了问题的原因是代码变更，没有探究细节。

因为JQ现在已经是工作中常用的工具了，所以比较好奇为何官方没有在released log上进行这次变更的记录，翻着代码看了一圈，发现这后面有几个小故事，记录下来，蛮好玩的。

解决方案也附在文末，欢迎讨论更佳的方法，当然，个人建议不要使用未公开的api进行程序编写，风险你懂的。

有人疑惑"在《锋利的JQuery》第二版中的一段示例代码，在1.7.2的JQ中正常，在最新的1.10.2中失效"，而触发失效的原因为`$("div").trigger("click!");`不能执行

示例代码是这样的:

```js
$("div").bind("click", function(){  
    alert("click only");  
});  
$("div").bind("click.plugin", function(){  
    alert("click plugin");  
});  
$("#btn").click(function(){  
    $("div").trigger("click!");//在click后添加叹号,则只触发不在命名空间内的事件  
});
```

那我们来看一看1.7.2中的代码实现吧：

```js
// JQ 1.7.2 trigger实现
trigger: function( event, data, elem, onlyHandlers ) {
        // 省略代码若干...
        if ( type.indexOf( "!" ) >= 0 ) {
            // Exclusive events trigger only for the exact event (no namespaces)
            type = type.slice(0, -1);
            exclusive = true;
        }

        if ( type.indexOf( "." ) >= 0 ) {
            // Namespaced trigger; create a regexp to match event type in handle()
            namespaces = type.split(".");
            type = namespaces.shift();
            namespaces.sort();
        }
        // 省略代码若干...

        // Caller can pass in an Event, Object, or just an event type string
        event = typeof event === "object" ?
            // jQuery.Event object
            event[ jQuery.expando ] ? event :
            // Object literal
            new jQuery.Event( type, event ) :
            // Just the event type (string)
            new jQuery.Event( type );

        event.type = type;
        event.isTrigger = true;
        event.exclusive = exclusive;
        event.namespace = namespaces.join( "." );
        event.namespace_re = event.namespace? new RegExp("(^|\\.)" + namespaces.join("\\.(?:.*\\.)?") + "(\\.|$)") : null;
        ontype = type.indexOf( ":" ) < 0 ? "on" + type : "";
        // 省略代码若干...
    },
```

而1.9.1版本中的代码却发生了变化（准确的说是在jQuery Core 1.9.0-b1后就发生了变化）：

```js
trigger: function( event, data, elem, onlyHandlers ) {
        // 省略代码若干...
        if ( type.indexOf(".") >= 0 ) {
            // Namespaced trigger; create a regexp to match event type in handle()
            namespaces = type.split(".");
            type = namespaces.shift();
            namespaces.sort();
        }
        ontype = type.indexOf(":") < 0 && "on" + type;

        // Caller can pass in a jQuery.Event object, Object, or just an event type string
        event = event[ jQuery.expando ] ?
            event :
            new jQuery.Event( type, typeof event === "object" && event );

        event.isTrigger = true;
        event.namespace = namespaces.join(".");
        event.namespace_re = event.namespace ?
            new RegExp( "(^|\\.)" + namespaces.join("\\.(?:.*\\.|)") + "(\\.|$)" ) :
            null;
        // 省略代码若干...
    },
```

改变一目了然，对于带有!的字符串的匹配和处理那段在新版本中被干掉了，变更版本为1.8到1.9之间。既然发生变化的大版本确定了，那么我们就不必git到本地然后全局搜索这条修改了，直接查找1.9b的event.js的修改记录，很容易就找到了修改的那次提交[395f1da76ba9faeb2f72548c28da228474a2434c]。

[GitHub上的围观地址](https://github.com/jquery/jquery/commit/395f1da76ba9faeb2f72548c28da228474a2434c) 作者卖萌的写到（估计内心十万个草泥马奔腾不愿删除）：

> Fix #12827\. Remove exclusive event semantics from .trigger(). No unit tests were removed in the undoing of this feature.![Image](https://attachment.soulteary.com/2014/01/17/sob.png "Image")

另外，这里有了ticket的ID，那么去官方的ticket系统里看一下吧，地址：http://bugs.jquery.com/ticket/12827

> Now that #11101 deprecated and we have removed data events via #10544 we can remove exclusive events in 1.9\. No docs changes required since this was never documented.

#11101这个ticket是一个细心的网友jzaefferer提交的标题为**DEPRECATE "EXCLUSIVE" EVENTS OPTION FROM TRIGGER METHOD**的ticket：

> Apparently trigger allows you to specify to trigger only non-namespaced events, via .trigger("click!"). Here's the implementation:[https://github.com/jquery/jquery/blob/4534db196bf9475c79f74d6b62ebc866c27d06d9/src/event.js#L250-254](https://github.com/jquery/jquery/blob/4534db196bf9475c79f74d6b62ebc866c27d06d9/src/event.js#L250-254) (line 250-254) I don't see any mention of that in the documentation: [ http://api.jquery.com/trigger/](http://api.jquery.com/trigger/) So either it should either be removed or documented.

看到这个ticket，作为主力之一的Dave Methvin童鞋只有囧囧的回复下面的内容了：

> We use exclusive events internally, in data events. However we've talked about deprecating those and when we remove them we can also remove exclusive events. That would be my preference. These have never been in the formal documentation but have been mentioned in a few places:
> 
> *   http://www.learningjquery.com/2007/09/namespace-your-events
> *   http://longgoldenears.blogspot.com/2010/02/jquery-namespaced-events-exclusive.html

另外，Dave Methvin在1.9版本中将这个内部实现的属性去掉了。

> Changed 19 months ago by dmethvin Status changed from assigned to closed Resolution set to fixed I will mark this as done since there are no docs changes. This ticket will serve as notice. Removal will be done as a separate ticket in 1.9.

在#10544中（这个是一个bug，大坑啊），有一个热心网友Darsain不仅提出来了，还给了作者一个在线demo，以及解决方案。

> $.fn.data() doesn't behave the same way how $.data() does when using a key namespacing. Example: if you have a data key "name.key" and it doesn't exist, the $.fn.data() will return the "name" key instead, while $.data() returns undefined, which is what I'd expect to get Test: http://jsfiddle.net/pNCP8/3/ (test is for 1.6.4, but 1.7b2 has the same issue)
> 
> <pre>@@ -1943,9 +1943,7 @@
>                 data = dataAttr( this[0], key, data );
>             }
> 
> -           return data === undefined && parts[1] ?
> -               this.data( parts[0] ) :
> -               data;
> +           return data;
> 
>         } else {
>             return this.each(function() {</pre>
> 
> Now, since I see that it is deliberately coded that way, I don't know if this is a bug, or a "feature". I've asked on #jquery-dev, but after a few hours of no reply I'm just posting it here. In case of a "feature" decision, I'd like to know the resoning behind it :)

这位童鞋的代码是这样：

```js
jQuery(function($){

    var div = $("div");

    // $.fn.data() case
    div.data("name", "name data");
    div.data("name.subdata", "subdata"); 
    div.data("name.removeme", "removeme"); 

    div.removeData("name.removeme");

    div.append("$.fn.data():"); 

    div.append("<br>I want this subdata   -> " + div.data("name.subdata"));
    div.append("<br>I want this undefined -> " + div.data("name.removeme"));
    div.append("<br>I want this undefined -> " + div.data("name.whatevs")); 

    // $.data() case
    $.data(div, "name", "name data"); 
    $.data(div, "name.subdata", "subdata");  
    $.data(div, "name.removeme", "removeme");  

    $.removeData(div, "name.removeme");

    div.append("<br><br>$.data():");

    div.append("<br>I want this subdata   -> " + $.data(div, "name.subdata"));
    div.append("<br>I want this undefined -> " + $.data(div, "name.removeme"));
    div.append("<br>I want this undefined -> " + $.data(div, "name.whatevs"));

});

//下面是输出内容
$.fn.data():
I want this subdata -> subdata
I want this undefined -> name data
I want this undefined -> name data

$.data():
I want this subdata -> subdata
I want this undefined -> undefined
I want this undefined -> undefined
```

这个充满童心的童鞋让作者addyosmani也瞬间懵了，一看demo，真的貌似是错误诶！于是回复到：

> Our docs don't appear to state whether or not namespaced keys are officially supported, but let me find out and I'll post back on this ticket (unless someone is able to answer before that).

不久，又回复到：

> Confirmed with @gnarf that this is a bug. CC'ing DaveMethvin for further input.

但是不久，大家发现，这货不是错误，而是一个单元测试函数....具体的大家可以浏览这里:http://bugs.jquery.com/ticket/10544 这个时候，gnarf觉得jquery支持带有命名空间的数据不太好啊，于是用IRC和timmy以及dave童鞋碰了下头，觉得，即使这种实现被很多东西依赖了（早在1.2.6中就有使用），但是始终不太好啊。 于是大家就吵啊吵啊（激烈讨论），gnarf告诉这位童鞋，呃，.data不是正常吗，你先将就一下，我们1.8版本会干掉这货的。 接着addyosmani童鞋说经过IRC上的讨论，大家很大可能（着重号下的maybe，又卖萌？）在1.7就干掉这货，因为1.7和1.7以下，带命名空间的家伙们或许会引发一些奇奇怪怪的事情，还提到了JohnResig 有一点新的点子将会在版本发布的时候添加，给了大家一个美好的盼头。 dmethvin童鞋在这个事情过后的一个月后说，我们将决定在1.8到1.9版本之间，让这个bug安乐死（其实想想就知道依赖有很多了...）。

时光飞逝，7个月后，dmethvin将这个bug解决了：https://github.com/jquery/jquery/commit/e8cf41a051a62bf1f19beab1a5c1d643f121e28e。 闲话扯了这么多，我们回到主题，因为上面的“data风波”解决掉了，于是在ticket#12827中，作者们废弃了专属事件的触发方式：

> Now that #11101 deprecated and we have removed data events via #10544 we can remove exclusive events in 1.9\. No docs changes required since this was never documented.

所以呢，我们找到了为何无法在官方released log里发现这个改变的原因。 上面的八卦我们看完之后，并没有解决问题，接着我们看看这个被去掉的属性在何处有被使用吧。 data的setter和getter和主题关联不大以及估计篇幅又要涉及蛮多，先不管之。 下面是1.8版本中的相关代码：

```js
dispatch: function( event ) {
        // 省略代码...变量声明为了篇幅写法略微修改
        var run_all = !event.exclusive && !event.namespace;
        // 省略代码...变量声明为了篇幅写法略微修改

        // Run delegates first; they may want to stop propagation beneath us
        for ( i = 0; i < handlerQueue.length && !event.isPropagationStopped(); i++ ) {
            matched = handlerQueue[ i ];
            event.currentTarget = matched.elem;

            for ( j = 0; j < matched.matches.length && !event.isImmediatePropagationStopped(); j++ ) {
                handleObj = matched.matches[ j ];

                // Triggered event must either 1) be non-exclusive and have no namespace, or
                // 2) have namespace(s) a subset or equal to those in the bound event (both can have no namespace).
                if ( run_all || (!event.namespace && !handleObj.namespace) || event.namespace_re && event.namespace_re.test( handleObj.namespace ) ) {

                    event.data = handleObj.data;
                    event.handleObj = handleObj;

                    ret = ( (jQuery.event.special[ handleObj.origType ] || {}).handle || handleObj.handler )
                            .apply( matched.elem, args );

                    if ( ret !== undefined ) {
                        event.result = ret;
                        if ( ret === false ) {
                            event.preventDefault();
                            event.stopPropagation();
                        }
                    }
                }
            }
        }
        // 省略代码...
    },
```

下面是1.9中的相关代码：

```js
dispatch: function( event ) {
    // 省略代码...
    // Run delegates first; they may want to stop propagation beneath us
    i = 0;
    while ( (matched = handlerQueue[ i++ ]) && !event.isPropagationStopped() ) {
        event.currentTarget = matched.elem;

        j = 0;
        while ( (handleObj = matched.handlers[ j++ ]) && !event.isImmediatePropagationStopped() ) {

            // Triggered event must either 1) have no namespace, or
            // 2) have namespace(s) a subset or equal to those in the bound event (both can have no namespace).
            if ( !event.namespace_re || event.namespace_re.test( handleObj.namespace ) ) {

                event.handleObj = handleObj;
                event.data = handleObj.data;

                ret = ( (jQuery.event.special[ handleObj.origType ] || {}).handle || handleObj.handler )
                        .apply( matched.elem, args );

                if ( ret !== undefined ) {
                    if ( (event.result = ret) === false ) {
                        event.preventDefault();
                        event.stopPropagation();
                    }
                }
            }
        }
    }
    // 省略代码...
},
```

dispatch这个函数中，关键差异为run_all这个在循环中的变量，直观差异如下，我写了一个demo：http://jsfiddle.net/UK89W/

```js
// 1.7.2
jQuery(function($){
    var board = $("#board");
    var btn = $('#btn');
    var echo = function(text, elem){
        var msg = text;
        if(elem && elem.id){
            msg+=' '+elem.id;
            msg = '====>'+msg;
        }
        board.append('<p>'+msg+"</p>");
    };
    btn.on('click',function(){
        echo('click 1st',this);
    }).on('click',function(){
        echo('click 2rd',this);
    }).on('click.new',function(){
        echo('click.new 1st',this);
    }).on('click.new',function(){
        echo('click.new 2rd',this);
    }).on('click.old',function(){
        echo('click.old 1st',this);
    }).on('click.old',function(){
        echo('click.old 2rd',this);
    });

    echo ('trigger click.new:');  
    btn.trigger('click.new');
    echo ('trigger click:');   
    btn.trigger('click');
    echo ('trigger click!:');   
    btn.trigger('click!');
});
```

在1.7.2以及1.8.3中输出为：

```text
trigger click.new:
====>click.new 1st btn
====>click.new 2rd btn
trigger click:
====>click 1st btn
====>click 2rd btn
====>click.new 1st btn
====>click.new 2rd btn
====>click.old 1st btn
====>click.old 2rd btn
trigger click!:
====>click 1st btn
====>click 2rd btn
```

但是在1.9以上输出为：

```text
trigger click.new:
====>click.new 1st btn
====>click.new 2rd btn
trigger click:
====>click 1st btn
====>click 2rd btn
====>click.new 1st btn
====>click.new 2rd btn
====>click.old 1st btn
====>click.old 2rd btn
trigger click!:
```

两者相比，少了没有命名空间的函数的执行，如果你依旧想实现之前的方法，或者区别执行无命名空间的事件以及全部的函数的话，或许可以用下面的方法：

```js
jQuery(function($){
    $('body').append('<div id="btn">BTN</div><div id="board"></div>');
    var board = $("#board");
    var btn = $('#btn').css({'border': '1px solid #A7A7A7','width': '80px','height': '30px','background-color': '#DBDBDB','text-align': 'center','color': '#000','margin': '10px','line-height':'30px','cursor': 'pointer'});
    var echo = function(text, elem){
        var msg = text;
        if(elem && elem.id){
            msg+=' '+elem.id;
            msg = '====>'+msg;
        }
        board.append('<p>'+msg+"</p>");
    };
    btn.
        on('click',function(){echo('click 1st',this)}).
        on('click',function(){echo('click 2rd',this)}).
        on('click.new',function(e, exec){if(!exec)echo('click.new 1st',this)}).
        on('click.new',function(e, exec){if(!exec)echo('click.new 2rd',this)}).
        on('click.old',function(e, exec){if(!exec)echo('click.old 1st',this)}).
        on('click.old',function(e, exec){if(!exec)echo('click.old 2rd',this)});
    echo ('trigger click.new:');  
    btn.trigger('click.new');
    echo ('trigger click:');   
    btn.trigger('click');
    echo ('trigger click!:');   
    btn.trigger('click','only no namespaces');
});
```

上面的代码实现比较简单，在新版本的jq中，trigger通过正则去运行所有匹配到的函数，所以如果要排除的话，只有在具有自定义命名空间的函数入口加判断，让其不执行。 demo地址：http://jsfiddle.net/6K6bq/ 输出同jq 1.7.2

```js
trigger click.new:

====>click.new 1st btn

====>click.new 2rd btn

trigger click:

====>click 1st btn

====>click 2rd btn

====>click.new 1st btn

====>click.new 2rd btn

====>click.old 1st btn

====>click.old 2rd btn

trigger click!:

====>click 1st btn

====>click 2rd btn
```

如果你有更好的想法，欢迎留言讨论，或者微博上at我。
 
--EOF--

