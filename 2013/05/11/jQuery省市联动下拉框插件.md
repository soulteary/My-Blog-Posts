# jQuery省市联动下拉框插件

最近通宵严重，整理一下碎片化的代码，否则真担心一段时间后忘记了什么。

> Github代码已经更新，如使用插件，请参考最新代码。

把之前写的省市联动的JQUERY插件重写了一下，记录如下： [![jquery-select-box](https://attachment.soulteary.com/2013/05/11/jquery-select-box.png "jquery-select-box")](https://attachment.soulteary.com/2013/05/11/jquery-select-box.png)

完成地址:[http://thecdn.sinaapp.com/page/demo/jq-select/](http://thecdn.sinaapp.com/page/demo/jq-select/) 

GitHub地址:[https://github.com/soulteary/jquery-city-select](https://github.com/soulteary/jquery-city-select)


首先建立基础HTML结构：

```html
html:5>div#warp>((select#province>option[value="载入中"])+(select#city>option[value="载入中"]))
```

预览地址：[http://thecdn.sinaapp.com/page/demo/jq-select/step-1.html](http://thecdn.sinaapp.com/page/demo/jq-select/step-1.html)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>省市下拉联动插件 @soulteary</title>
</head>
<body>
<div id="warp">
    <select id="province">
        <option value="载入中">载入中</option>
    </select>
    <select id="city">
        <option value="载入中">载入中</option>
    </select>
</div>
</body>
</html>
```

添加简单的样式。

地址:[http://thecdn.sinaapp.com/page/demo/jq-select/step-2.html](http://thecdn.sinaapp.com/page/demo/jq-select/step-2.html)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>省市下拉联动插件 @soulteary</title>

    <style>
        #warp {
            width: 230px;
            height: 80px;
            position: absolute;
            top: 50%;
            left: 50%;
            margin-left: -200px;
            margin-top: -130px;
            box-shadow: 1px 1px #3A393A;
            border: 1px solid #222;
            font-size: 13px;
            line-height: 13px;
        }

        #province {
            margin: 30px 0 0 30px;
            float: left;
        }

        #city {
            float: right;
            margin: 30px 30px 0 0;
        }
    </style>

</head>
<body>
<div id="warp">
    <select id="province">
        <option value="载入中">载入中</option>
    </select>
    <select id="city">
        <option value="载入中">载入中</option>
    </select>
</div>
</body>
</html>
```

接着来写JS实现。 首先我们要进行数据定义，就是这个省市联动的数据是如何的关系。 有许多的省份，那么省份是包含在一个数组中的，然后每个省份中的城市包含在这些省份中，所以获得包含关系。 定义结构如下：(为了简单的开发，我随意写的数据，真实上线用自己定义的数据来替换即可)

```js
    [
        {'name':'北京市', id:100000, children:[
            {'name':'海淀区', id:100003},
            {'name':'西城区', id:100002},
            {'name':'东城区', id:100001}
        ]},
        {'name':'天津市', id:300000, children:[
            {'name':'北辰区', id:300003},
            {'name':'红桥区', id:300002},
            {'name':'河西区', id:300001}
        ]}
    ];
```

简单的思考后，插件接受的参数为不定长，1~2个，为什么这么说呢。 场景A:数据要输出到一个下拉框中，即省份和城市输出一起。 场景B:数据分别初始化到两个下拉列表中。 那么设计插件框架如下：

```js
(function ($) {
    $.fn.extend({
        "citylist": function (params) {
            params = $.extend({
                param:'this is a param'
            }, params);
            var target = $(this);
            if(!target.length){
                return this;
            }else if(target.length==1){

            }else{

            }
            console.log('PLUGN LOADED!:',target,params);

            return this;
        }
    });
})(jQuery, 'SOULTEARY.COM');

//USEAGE:
$('#province').citylist();
$('#province , #city').citylist();
```

接着来写实现代码: 地址:[http://thecdn.sinaapp.com/page/demo/jq-select/step-3.html](http://thecdn.sinaapp.com/page/demo/jq-select/step-3.html) 首先是将省市数据初始化到两个不同的列表中的功能的实现，即：

`$('#province , #city').citylist();`

```js
     //插件基础代码
    ;
    (function ($) {
        $.fn.extend({
            "citylist": function (params) {
                params = $.extend({
                    param: 'this is a param'
                }, params);
                var target = $(this);
                if (!target.length) {
                    return this;
                } else if (target.length == 1) {

                } else {
                    var province = target.eq(0);
                    var city = target.eq(1);

                    var html = [], oItem;
                    for(var item in params.data){
                        oItem = params.data[item];
                        console.log(oItem)
                        html.push('<option data-extra="'+oItem['id']+'">'+oItem['name']+'</option>');
                    }
                    html = html.join('');
                    province.find('option').remove();
                    province.append(html);

                    var provinces = province.find('option');
                    province.on('change',function(){
                        var curSelect = $(this).val();
                        provinces.each(function(k,v){
                            if ($(v).val() == curSelect){
                                return (function(v){
                                    var extra = $(v).attr('data-extra');
                                    var html = [], oItem;
                                    for(var item in params.data){
                                        oItem = params.data[item];
                                        if(oItem['id'] == extra && oItem.children){
                                            oItem = oItem.children;
                                            for(var sItem in oItem){
                                                html.push('<option data-extra="'+oItem[sItem]['id']+'">'+oItem[sItem]['name']+'</option>');
                                            }
                                            break;
                                        }
                                    }
                                    html = html.join('');
                                    city.find('option').remove();
                                    city.append(html);
                                }(v));
                            }
                        })

                    }).trigger('change');

                }
                return this;
            }
        });
    })(jQuery, 'SOULTEARY.COM');

    //插件调用代码
    ;
    $(function () {

        var data = [
            {'name': '北京市', id: 100000, children: [
                {'name': '海淀区', id: 100003},
                {'name': '西城区', id: 100002},
                {'name': '东城区', id: 100001}
            ]},
            {'name': '天津市', id: 300000, children: [
                {'name': '北辰区', id: 100003},
                {'name': '红桥区', id: 100002},
                {'name': '河西区', id: 100001}
            ]}
        ];

        $('#province, #city').citylist({data:data});
    }); 
```

接着我们继续写单独SELECT BOX的功能。 首先修改HTML结构，添加一个元素作为合并输出数据的容器。可以这么做：

```html

<div id="warp2">
    <select id="all">
        <option value="载入中">载入中</option>
    </select>
</div>

```

当然，为了有较好的心情去写实现代码，我们需要把CSS也写一下:

```css
#warp2 {
    width: 230px;
    height: 80px;
    position: absolute;
    top: 50%;
    left: 50%;
    margin-left: -200px;
    margin-top: 0;
    box-shadow: 1px 1px #3A393A;
    border: 1px solid #222;
    font-size: 13px;
    line-height: 13px;
}

#all {
    margin: 30px;
}
```

接着开始写实现代码：

```js
var all = target;
var data = params.data;
var html = [];

for(var oo in data){
    html.push(''+data[oo]['name']+'');
    for(var xx in data[oo].children){
        html.push(''+data[oo].children[xx]['name']+'');
    }
}
html = html.join('');
all.find('option').remove();
all.append(html);
```

完整实现代码: 地址:`http://thecdn.sinaapp.com/page/demo/jq-select/step-4.html`

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>省市下拉联动插件 @soulteary</title>
    <style>
        #warp {
            width: 230px;
            height: 80px;
            position: absolute;
            top: 50%;
            left: 50%;
            margin-left: -200px;
            margin-top: -130px;
            box-shadow: 1px 1px #3A393A;
            border: 1px solid #222;
            font-size: 13px;
            line-height: 13px;
        }

        #province {
            margin: 30px 0 0 30px;
            float: left;
        }

        #city {
            float: right;
            margin: 30px 30px 0 0;
        }

        #warp2 {
            width: 230px;
            height: 80px;
            position: absolute;
            top: 50%;
            left: 50%;
            margin-left: -200px;
            margin-top: 0;
            box-shadow: 1px 1px #3A393A;
            border: 1px solid #222;
            font-size: 13px;
            line-height: 13px;
        }

        #all {
            margin: 30px;
        }
    </style>
    <script src="extra/jquery.js"></script>
</head>
<body>
<div id="warp">
    <select id="province">
        <option value="载入中">载入中</option>
    </select>
    <select id="city">
        <option value="载入中">载入中</option>
    </select>
</div>

<div id="warp2">
    <select id="all">
        <option value="载入中">载入中</option>
    </select>
</div>

<script type="text/javascript">
  //插件基础代码
  ;(function($) {
    $.fn.extend({
      'citylist': function(params) {
        params = $.extend({
          param: 'this is a param',
        }, params);
        var target = $(this);
        if (!target.length) {
          return this;
        } else if (target.length == 1) {
          var all = target;
          var data = params.data;
          var html = [];

          for (var oo in data) {
            html.push('<option data-extra="' + data[oo]['id'] + '">' + data[oo]['name'] + '</option>');
            for (var xx in data[oo].children) {
              html.push('<option data-extra="' + data[oo].children[xx]['id'] + '">' + data[oo].children[xx]['name'] + '</option>');
            }
          }
          html = html.join('');
          all.find('option').remove();
          all.append(html);

        } else if (target.length == 2) {
          var province = target.eq(0);
          var city = target.eq(1);

          var html = [], oItem;
          for (var item in params.data) {
            oItem = params.data[item];
            html.push('<option data-extra="' + oItem['id'] + '">' + oItem['name'] + '</option>');
          }
          html = html.join('');
          province.find('option').remove();
          province.append(html);

          var provinces = province.find('option');
          province.on('change', function() {
            var curSelect = $(this).val();
            provinces.each(function(k, v) {
              if ($(v).val() == curSelect) {
                return (function(v) {
                  var extra = $(v).attr('data-extra');
                  var html = [], oItem;
                  for (var item in params.data) {
                    oItem = params.data[item];
                    if (oItem['id'] == extra && oItem.children) {
                      oItem = oItem.children;
                      for (var sItem in oItem) {
                        html.push('<option data-extra="' + oItem[sItem]['id'] + '">' + oItem[sItem]['name'] + '</option>');
                      }
                      break;
                    }
                  }
                  html = html.join('');
                  city.find('option').remove();
                  city.append(html);
                }(v));
              }
            });

          }).trigger('change');

        }
        return this;
      },
    });
  })(jQuery, 'SOULTEARY.COM');

  //插件调用代码
  ;$(function() {

    var data = [
      {
        'name': '北京市', id: 100000, children: [
          {'name': '海淀区', id: 100003},
          {'name': '西城区', id: 100002},
          {'name': '东城区', id: 100001},
        ],
      },
      {
        'name': '天津市', id: 300000, children: [
          {'name': '北辰区', id: 100003},
          {'name': '红桥区', id: 100002},
          {'name': '河西区', id: 100001},
        ],
      },
    ];

    $('#province, #city').citylist({data: data});

    $('#all').citylist({data: data});
  });
</script>
</body>
</html>
```

最后，把CSS,JS独立出页面即可。

当然，这里多添加了一个小功能，如果使用者定义的数据不是id,name,children这样的键，或者不想使用DATA-EXTRA来保存数据，那么一样可以使用这个小插件。

- 完成地址:[http://thecdn.sinaapp.com/page/demo/jq-select/](http://thecdn.sinaapp.com/page/demo/jq-select/) 
- GitHub地址:[https://github.com/soulteary/jquery-city-select](https://github.com/soulteary/jquery-city-select)


