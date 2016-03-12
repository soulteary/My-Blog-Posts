# marquee实现黑客帝国矩阵效果

逛旧帖子，发现“高人”用marquee实现的一个效果,蛮巧妙的..

<!-- more -->

```html
<!doctype>
<html>
    
    <head>
        <title>marquee matrix</title>
    </head>
    
    <body>
        <table border="0" height="20" cellspacing="0" cellpadding="0" style="bgcolor:black;color:blue;width:400px;">
            <tbody>
                <tr style="bgcolor:black">
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="3">1
                            <br />A
                            <br />B
                            <br />C
                            <br />D
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />6
                            <br />7
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="5">1
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />2
                            <br />3
                            <br />A
                            <br />B
                            <br />C
                            <br />D
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="6">1
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />7
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="2">1
                            <br />2
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="9">1
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />A
                            <br />B
                            <br />C
                            <br />D
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="-1">1
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />7
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="5">1
                            <br />A
                            <br />B
                            <br />C
                            <br />D
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />8
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="7">1
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="4">1
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />A
                            <br />B
                            <br />C
                            <br />D
                            <br />7
                            <br />8
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="1">1
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />1
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="8">1
                            <br />2
                            <br />3
                            <br />A
                            <br />B
                            <br />C
                            <br />D
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="3">1
                            <br />2
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="5">1
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />8
                            <br />9
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="6">1
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="2">G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />1
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="9">1
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />8
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="-1">A
                            <br />B
                            <br />C
                            <br />D
                            <br />E
                            <br />F
                            <br />1
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                </tr>
            </tbody>
        </table>
    </body>

</html>
</pre>
<runcode>
<!doctype>
<html>
    
    <head>
        <title>marquee matrix</title>
    </head>
    
    <body>
        <table border="0" height="20" cellspacing="0" cellpadding="0" style="bgcolor:black;color:blue;width:400px;">
            <tbody>
                <tr style="bgcolor:black">
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="3">1
                            <br />A
                            <br />B
                            <br />C
                            <br />D
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />6
                            <br />7
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="5">1
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />2
                            <br />3
                            <br />A
                            <br />B
                            <br />C
                            <br />D
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="6">1
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />7
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="2">1
                            <br />2
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="9">1
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />A
                            <br />B
                            <br />C
                            <br />D
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="-1">1
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />7
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="5">1
                            <br />A
                            <br />B
                            <br />C
                            <br />D
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />8
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="7">1
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="4">1
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />A
                            <br />B
                            <br />C
                            <br />D
                            <br />7
                            <br />8
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="1">1
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />1
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="8">1
                            <br />2
                            <br />3
                            <br />A
                            <br />B
                            <br />C
                            <br />D
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="3">1
                            <br />2
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="5">1
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />8
                            <br />9
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="6">1
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="2">G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />1
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="9">1
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />8
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />9
                            <br />0</marquee>
                    </td>
                    <td width="10" bgcolor="black">
                        <marquee direction="down" scrollamount="-1">A
                            <br />B
                            <br />C
                            <br />D
                            <br />E
                            <br />F
                            <br />1
                            <br />2
                            <br />3
                            <br />4
                            <br />5
                            <br />6
                            <br />7
                            <br />G
                            <br />H
                            <br />I
                            <br />J
                            <br />K
                            <br />L
                            <br />M
                            <br />N
                            <br />8
                            <br />9
                            <br />0</marquee>
                    </td>
                </tr>
            </tbody>
        </table>
    </body>

</html>
```

