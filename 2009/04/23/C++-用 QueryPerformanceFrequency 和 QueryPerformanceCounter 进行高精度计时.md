# [C++]用 QueryPerformanceFrequency 和 QueryPerformanceCounter 进行高精度计时

用 QueryPerformanceFrequency 和 QueryPerformanceCounter 进行高精度计时 [http://www.cppblog.com/bidepan2023/archive/2008/01/22/41627.html](http://www.cppblog.com/bidepan2023/archive/2008/01/22/41627.html)

```c
void main() {       
    LARGE_INTEGER lv;   

    // 获取每秒多少CPU Performance Tick   
    QueryPerformanceFrequency( &lv );   

    // 转换为每个Tick多少秒   
    double secondsPerTick = 1.0 / lv.QuadPart;   

    for ( size_t i = 0; i < 100; ++i ) {   
        // 获取CPU运行到现在的Tick数   
        QueryPerformanceCounter( &lv );   

        // 计算CPU运行到现在的时间   
        // 比GetTickCount和timeGetTime更加精确   
        double timeElapsedTotal = secondsPerTick * lv.QuadPart;   

        cout.precision( 6 );   
        cout << fixed << showpoint << timeElapsedTotal << endl;   
        //printf( "%lf
", timeElapsedTotal ) ;   
    }   
}
```

