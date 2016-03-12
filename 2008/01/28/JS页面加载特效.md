# JS页面加载特效

2017了，IE9以下的浏览器份额越来越低，直接使用CSS3实现即可。

```html
<!DOCTYPE html>
<html>
<head>
    <style>
        .progress {
            background: #8ac6f7;
            border: 1px solid #c1c1ff;
            width: auto;
            height: 10px;
            -webkit-animation-name: example;
            -webkit-animation-duration: 2s;
            animation-name: loading;
            animation-duration: 2s;
            animation-iteration-count: infinite;
        }

        /* Safari 4.0 - 8.0 */
        @-webkit-keyframes loading {
            0% {
                width: 0;
            }
            10% {
                width: 10%;
            }
            20% {
                width: 20%;
            }
            30% {
                width: 30%;
            }
            40% {
                width: 40%;
            }
            50% {
                width: 50%;
            }
            60% {
                width: 60%;
            }
            70% {
                width: 70%;
            }
            80% {
                width: 80%;
            }
            90% {
                width: 90%;
            }
            100% {
                width: 100%;
            }

        }

        /* Standard syntax */
        @keyframes loading {
            0% {
                width: 0;
            }
            10% {
                width: 10%;
            }
            20% {
                width: 20%;
            }
            30% {
                width: 30%;
            }
            40% {
                width: 40%;
            }
            50% {
                width: 50%;
            }
            60% {
                width: 60%;
            }
            70% {
                width: 70%;
            }
            80% {
                width: 80%;
            }
            90% {
                width: 90%;
            }
            100% {
                width: 100%;
            }
        }

        .loader {
            width: 120px;
            border: 1px solid #b1b1ff;
            height: 50px;
            margin: 0 auto;
            text-align: center;
            font-size: 13px;
            line-height: 2;
            background: #d7edff;
        }
    </style>
</head>
<body>

<div class="loader">
    <span>加载中...</span>
    <div class="progress"></div>
</div>

</body>
</html>
```


