# PHP计算一年内所有星期开始和结束日期

PHP计算一年内所有星期开始以及结束日期，PHP如何计算一年内所有星期的范围呢。

52星期这个其实不用再做判断了，直接给数值52即可。

```
<?php
function get_weeks_whole_year($year)
{
    $year_start = $year . "-01-01";
    $year_end   = $year . "-12-31";
    $startday   = strtotime($year_start);
    if (intval(date('N', $startday)) != '1') {
        $startday = strtotime("next Monday", strtotime($year_start));
    }
    $year_mondy = date("Y-m-d", $startday); 
    $endday     = strtotime($year_end);

//必然52个星期,所以这句其实不用加的
//    if (intval(date('W', $endday)) == '7') {
        $endday = strtotime("last Sunday", strtotime($year_end));
//    }
    $num = intval(date('W', $endday));


    for ($i = 1; $i <= $num; $i++) {
        $j              = $i - 1;
        $start_date     = date("Y-m-d", strtotime("$year_mondy $j week "));
        $end_day        = date("Y-m-d", strtotime("$start_date +6 day"));
        $week_array[$i] = array(
            str_replace("-", ".", $start_date),
            str_replace("-", ".", $end_day)
        );
    }
    return $week_array;
}

var_dump(get_weeks_whole_year(2012));

?>
```


