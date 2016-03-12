# jQuery焦点图

这个焦点图的亮点是index索引的使用. 来源懒人图库和QQ商城..

<!-- more -->

```
<!doctype>
<html>

    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <title>焦点图</title>
        <style type="text/css">
            * {
                margin:0;
                padding:0;
            }
            body {
                font-size:12px;
                color:#222;
                font-family:Verdana, Arial, Helvetica, sans-serif;
                background:#f0f0f0;
            }
            ul, li {
                list-style:none;
            }
            img {
                border:0;
            }
            .wrapper {
                width:500px;
                margin:0 auto;
                padding-bottom:50px;
            }
            h1 {
                height:50px;
                line-height:50px;
                font-size:22px;
                font-weight:normal;
                font-family:"Microsoft YaHei", SimHei;
            }
            #focus {
                width:500px;
                height:286px;
                overflow:hidden;
                position:relative;
            }
            #focus ul {
                height:386px;
                position:absolute;
            }
            #focus ul li {
                float:left;
                width:500px;
                height:286px;
                overflow:hidden;
                position:relative;
                background:#000;
            }
            #focus ul li div {
                position:absolute;
                overflow:hidden;
            }
            #focus .btnBg {
                position:absolute;
                width:500px;
                height:20px;
                left:0;
                bottom:0;
                background:#000;
            }
            #focus .btn {
                position:absolute;
                width:780px;
                height:10px;
                padding:5px 10px;
                right:0;
                bottom:0;
                text-align:right;
            }
            #focus .btn span {
                display:inline-block;
                _display:inline;
                _zoom:1;
                width:25px;
                height:10px;
                _font-size:0;
                margin-left:5px;
                cursor:pointer;
                background:#fff;
            }
            #focus .btn span.on {
                background:#fff;
            }
            #focus .preNext {
                width:45px;
                height:100px;
                position:absolute;
                top:90px;
                background:url(img/sprite.png) no-repeat 0 0;
                cursor:pointer;
            }
            #focus .pre {
                left:0;
            }
            #focus .next {
                right:0;
                background-position:right top;
            }
        </style>
        <script type="text/javascript" src="js/jquery.min.js"></script>
        <script type="text/javascript">
            $(function () {

                var sWidth = $("#focus").width(); //获取焦点图的宽度（显示面积）
                var len = $("#focus ul li").length; //获取焦点图个数
                var index = 0;

                var picTimer;



                //以下代码添加数字按钮和按钮后的半透明条，还有上一页、下一页两个按钮
                var btn = "<div class='btnBg'></div><div class='btn'>";

                for (var i = 0; i < len; i++) {

                    btn += "<span></span>";

                }

                btn += "</div><div class='preNext pre'></div><div class='preNext next'></div>";

                $("#focus").append(btn);

                $("#focus .btnBg").css("opacity", 0.5);



                //为小按钮添加鼠标滑入事件，以显示相应的内容
                $("#focus .btn span").css("opacity", 0.4).mouseenter(function () {

                    index = $("#focus .btn span").index(this);

                    showPics(index);

                }).eq(0).trigger("mouseenter");



                //上一页、下一页按钮透明度处理
                $("#focus .preNext").css("opacity", 0.2).hover(function () {

                    $(this).stop(true, false).animate({
                        "opacity": "0.5"
                    }, 300);

                }, function () {

                    $(this).stop(true, false).animate({
                        "opacity": "0.2"
                    }, 300);

                });



                //上一页按钮
                $("#focus .pre").click(function () {

                    index -= 1;

                    if (index == -1) {
                        index = len - 1;
                    }

                    showPics(index);

                });



                //下一页按钮
                $("#focus .next").click(function () {

                    index += 1;

                    if (index == len) {
                        index = 0;
                    }

                    showPics(index);

                });



                //本例为左右滚动，即所有li元素都是在同一排向左浮动，所以这里需要计算出外围ul元素的宽度
                $("#focus ul").css("width", sWidth * (len));



                //鼠标滑上焦点图时停止自动播放，滑出时开始自动播放
                $("#focus").hover(function () {

                    clearInterval(picTimer);

                }, function () {

                    picTimer = setInterval(function () {

                        showPics(index);

                        index++;

                        if (index == len) {
                            index = 0;
                        }

                    }, 4000); //此4000代表自动播放的间隔，单位：毫秒
                }).trigger("mouseleave");



                //显示图片函数，根据接收的index值显示相应的内容
                function showPics(index) { //普通切换
                    var nowLeft = -index * sWidth; //根据index值计算ul元素的left值
                    $("#focus ul").stop(true, false).animate({
                        "left": nowLeft
                    }, 300); //通过animate()调整ul元素滚动到计算出的position

                    //$("#focus .btn span").removeClass("on").eq(index).addClass("on"); //为当前的按钮切换到选中的效果
                    $("#focus .btn span").stop(true, false).animate({
                        "opacity": "0.4"
                    }, 300).eq(index).stop(true, false).animate({
                        "opacity": "1"
                    }, 300); //为当前的按钮切换到选中的效果
                }

            });
        </script>
    </head>

    <body>
        <div class="wrapper">
            <h1>jQuery2012QQ商城焦点图效果</h1>
            <div id="focus">
                <ul>
                    <li>
                        <a href="#" target="_blank"><img src="img/01.png" alt="QQ商城焦点图效果下载" /></a>
                    </li>
                    <li>
                        <a href="#" target="_blank"><img src="img/02.png" alt="QQ商城焦点图效果教程" /></a>
                    </li>
                    <li>
                        <a href="#" target="_blank"><img src="img/03.png" alt="jquery商城焦点图效果" /></a>
                    </li>
                    <li>
                        <a href="#" target="_blank"><img src="img/04.png" alt="jquery商城焦点图代码" /></a>
                    </li>
                    <li>
                        <a href="#" target="_blank"><img src="img/05.png" alt="jquery商城焦点图源码" /></a>
                    </li>
                </ul>
            </div>
        </div>
    </body>

</html>
```

演示地址: http://promiseforever.com/extra-docs/focus_1/

