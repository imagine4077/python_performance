# python_performance
《python高性能编程》相关学习代码


---

# CPU优化工具

1. python -O -m cProfile -s cumulative julia.py
> 原生cProfile工具，运行后根据各个函数累计耗时，将各函数排序。（-O 【欧】用于关闭assert）

       328513570 function calls in 143.591 seconds
          
       Ordered by: cumulative time
       
          ncalls  tottime  percall  cumtime  percall filename:lineno(function)
               1    0.000    0.000  143.591  143.591 {built-in method builtins.exec}
               1    0.000    0.000  143.591  143.591 julia.py:1(<module>)
               1    0.355    0.355  143.591  143.591 julia.py:19(calc_pure_python)
           10000    0.137    0.000  142.985    0.014 julia.py:5(measure_time)
           10000  107.569    0.011  142.600    0.014 julia.py:55(calculate_z_serial_purepython)
       328353350   35.027    0.000   35.027    0.000 {built-in method builtins.abs}
           40000    0.468    0.000    0.468    0.000 {built-in method builtins.print}
           40000    0.019    0.000    0.019    0.000 {built-in method time.time}
           40000    0.010    0.000    0.010    0.000 {built-in method builtins.len}
           20200    0.005    0.000    0.005    0.000 {method 'append' of 'list' objects}
               1    0.000    0.000    0.000    0.000 julia.py:4(timefn)
               1    0.000    0.000    0.000    0.000 functools.py:34(update_wrapper)
               7    0.000    0.000    0.000    0.000 {built-in method builtins.getattr}
               1    0.000    0.000    0.000    0.000 functools.py:64(wraps)
               5    0.000    0.000    0.000    0.000 {built-in method builtins.setattr}
               1    0.000    0.000    0.000    0.000 {method 'update' of 'dict' objects}
               1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}


2. python -O -m cProfile -o profile.stats julia.py
> 详细记录运行时长，并保存在profile.stats文件中

       >>> import pstats
       >>> p = pstats.Stats("profile.stats")
       >>> p.sort_stats("cumulative")
       <pstats.Stats object at 0x7f9bfaa8a250>
       >>> p.print_stats()
       
       328513570 function calls in 143.591 seconds
          
       Ordered by: cumulative time
       
          ncalls  tottime  percall  cumtime  percall filename:lineno(function)
               1    0.000    0.000  143.591  143.591 {built-in method builtins.exec}
               1    0.000    0.000  143.591  143.591 julia.py:1(<module>)
               1    0.355    0.355  143.591  143.591 julia.py:19(calc_pure_python)
           10000    0.137    0.000  142.985    0.014 julia.py:5(measure_time)
           10000  107.569    0.011  142.600    0.014 julia.py:55(calculate_z_serial_purepython)
       328353350   35.027    0.000   35.027    0.000 {built-in method builtins.abs}
           40000    0.468    0.000    0.468    0.000 {built-in method builtins.print}
           40000    0.019    0.000    0.019    0.000 {built-in method time.time}
           40000    0.010    0.000    0.010    0.000 {built-in method builtins.len}
           20200    0.005    0.000    0.005    0.000 {method 'append' of 'list' objects}
               1    0.000    0.000    0.000    0.000 julia.py:4(timefn)
               1    0.000    0.000    0.000    0.000 functools.py:34(update_wrapper)
               7    0.000    0.000    0.000    0.000 {built-in method builtins.getattr}
               1    0.000    0.000    0.000    0.000 functools.py:64(wraps)
               5    0.000    0.000    0.000    0.000 {built-in method builtins.setattr}
               1    0.000    0.000    0.000    0.000 {method 'update' of 'dict' objects}
               1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}

> * 如果某函数被多个地方调用，可以通过print_callers()定位到最耗时的父调用

       >>> p.print_callers()
          Ordered by: cumulative time
       
              Function                                                                         was called by...
                                                                                            ncalls  tottime  cumtime
       {built-in method builtins.exec}                                                  <- 
       julia.py:1(<module>)                                                             <-       1    0.000  154.878  {built-in method builtins.exec}
       julia.py:19(calc_pure_python)                                                    <-       1    0.351  154.878  julia.py:1(<module>)
       julia.py:5(measure_time)                                                         <-   10000    0.133  154.305  julia.py:19(calc_pure_python)
       julia.py:55(calculate_z_serial_purepython)                                       <-   10000  117.240  153.931  julia.py:5(measure_time)
       {built-in method builtins.abs}                                                   <- 328353350   36.688   36.688  julia.py:55(calculate_z_serial_purepython)
       {built-in method builtins.print}                                                 <-   10000    0.228    0.228         julia.py:5(measure_time)
                                                                                      30000    0.203    0.203  julia.py:19(calc_pure_python)
       {built-in method time.time}                                                      <-   20000    0.012    0.012  julia.py:5(measure_time)
                                                                                             20000    0.007    0.007  julia.py:19(calc_pure_python)
       {built-in method builtins.len}                                                   <-   20000    0.006    0.006  julia.py:19(calc_pure_python)
                                                                                             20000    0.004    0.004  julia.py:55(calculate_z_serial_purepython)
       {method 'append' of 'list' objects}                                              <-   20200    0.005    0.005  julia.py:19(calc_pure_python)
       julia.py:4(timefn)                                                               <-       1    0.000    0.000  julia.py:1(<module>)
       /Users/albert/anaconda3/anaconda3/lib/python3.8/functools.py:34(update_wrapper)  <-       1    0.000    0.000  julia.py:4(timefn)
       {built-in method builtins.getattr}                                               <-       7    0.000    0.000  /Users/albert/anaconda3/anaconda3/lib/python3.8/functools.py:34(update_wrapper)
       /Users/albert/anaconda3/anaconda3/lib/python3.8/functools.py:64(wraps)           <-       1    0.000    0.000  julia.py:4(timefn)
       {built-in method builtins.setattr}                                               <-       5    0.000    0.000  /Users/albert/anaconda3/anaconda3/lib/python3.8/functools.py:34(update_wrapper)
       {method 'update' of 'dict' objects}                                              <-       1    0.000    0.000  /Users/albert/anaconda3/anaconda3/lib/python3.8/functools.py:34(update_wrapper)
       {method 'disable' of '_lsprof.Profiler' objects}                                 <- 
       
       
       <pstats.Stats object at 0x7f9bfaa8a250>

> * 如果函数被多个地方调用，print_callees()得到的列表能帮助我们定位到最耗时的父函数
       
       >>> p.print_callees()
          Ordered by: cumulative time
       
       Function                                                                         called...
                                                                                     ncalls  tottime  cumtime
       {built-in method builtins.exec}                                                  ->       1    0.000  154.878  julia.py:1(<module>)
       julia.py:1(<module>)                                                             ->       1    0.000    0.000  julia.py:4(timefn)
                                                                                                 1    0.351  154.878  julia.py:19(calc_pure_python)
       julia.py:19(calc_pure_python)                                                    ->   10000    0.133  154.305  julia.py:5(measure_time)
                                                                                             20000    0.006    0.006  {built-in method builtins.len}
                                                                                             30000    0.203    0.203  {built-in method builtins.print}
                                                                                             20000    0.007    0.007  {built-in method time.time}
                                                                                             20200    0.005    0.005  {method 'append' of 'list' objects}
       julia.py:5(measure_time)                                                         ->   10000  117.240  153.931  julia.py:55(calculate_z_serial_purepython)
                                                                                             10000    0.228    0.228  {built-in method builtins.print}
                                                                                             20000    0.012    0.012  {built-in method time.time}
       julia.py:55(calculate_z_serial_purepython)                                       -> 328353350   36.688   36.688  {built-in method builtins.abs}
                                                                                             20000    0.004    0.004  {built-in method builtins.len}
       {built-in method builtins.abs}                                                   -> 
       {built-in method builtins.print}                                                 -> 
       {built-in method time.time}                                                      -> 
       {built-in method builtins.len}                                                   -> 
       {method 'append' of 'list' objects}                                              -> 
       julia.py:4(timefn)                                                               ->       1    0.000    0.000  /Users/albert/anaconda3/anaconda3/lib/python3.8/functools.py:34(update_wrapper)
                                                                                                 1    0.000    0.000  /Users/albert/anaconda3/anaconda3/lib/python3.8/functools.py:64(wraps)
       /Users/albert/anaconda3/anaconda3/lib/python3.8/functools.py:34(update_wrapper)  ->       7    0.000    0.000  {built-in method builtins.getattr}
                                                                                                 5    0.000    0.000  {built-in method builtins.setattr}
                                                                                                 1    0.000    0.000  {method 'update' of 'dict' objects}
       {built-in method builtins.getattr}                                               -> 
       /Users/albert/anaconda3/anaconda3/lib/python3.8/functools.py:64(wraps)           -> 
       {built-in method builtins.setattr}                                               -> 
       {method 'update' of 'dict' objects}                                              -> 
       {method 'disable' of '_lsprof.Profiler' objects}                                 -> 
       
       
       <pstats.Stats object at 0x7f9bfaa8a250>



3. python -m timeit -n 5 -r 5 -s "import julia" "julia.test_run(6)"

       5 loops, best of 5: 33.4 usec per loop
> * 原生timeit工具，运行单个函数
> * 也可以在Ipython内执行和使用，执行单个语句：
>> 
>>        In [1]: %timeit abs(-3) < 2
>>        76 ns ± 0.896 ns per loop (mean ± std. dev. of 7 runs, 10000000 loops each)


4. /usr/bin/time -p python julia.py

       real         0.43
       user         0.06
       sys          0.06

> * real 记录了整体的耗时。
> * user 记录了 CPU 花在任务上的时间，但不包括内核函数耗费的时间。 
> * sys 记录了内核函数耗费的时间。
> 
> 对 user 和 sys 相加就得到了 CPU 总共花费的时间。而这个时间和 real 的差则 有可能是花费在等待 IO 上，也可能是你的系统正在忙着运行其他任务因此影响了你的测量。

5. kernprof -l -v julia_lineprofiler.py
> * 需要修改源代码，在需要测试的函数上加装饰器 @profile （无需import任何模块）,具体可见julia_lineprofiler.py
> * 用于检查被测函数中每一条语句的执行时间
> * % Time指当前语句耗时占总体耗时的比例

       Wrote profile results to julia_lineprofiler.py.lprof
       Timer unit: 1e-06 s
       
       Total time: 629.313 s
       File: julia_lineprofiler.py
       Function: calculate_z_serial_purepython at line 57
       
       Line #      Hits         Time  Per Hit   % Time  Line Contents
       ==============================================================
           57                                           @timefn
           58                                           @profile
           59                                           def calculate_z_serial_purepython(maxiter, zs, cs):
           60                                               """Calculate output list using Julia update rule"""
           61     10000      57741.0      5.8      0.0      output = [0] * len(zs)
           62  50015000   23108122.0      0.5      3.7      for i in range(len(zs)):
           63  50005000   22033007.0      0.4      3.5          n = 0
           64  50005000   25636079.0      0.5      4.1          z = zs[i]
           65  50005000   24792545.0      0.5      3.9          c = cs[i]
           66 328353350  213632138.0      0.7     33.9          while abs(z) < 2 and n < maxiter:
           67 278348350  154265234.0      0.6     24.5              z = z * z + c
           68 278348350  140518954.0      0.5     22.3              n += 1
           69  50005000   25262461.0      0.5      4.0          output[i] = n
           70     10000       6633.0      0.7      0.0      return output

### 上述几种方法，涉及的分析文件

* .stats (cProfile产生，可在ipython或python命令行中通过代码，调用pstats模块，解析之)
* .lprof (kernprof产生)

### 遗留问题
1. runsnakerun的使用，遇到了问题，待解决，待试用
2. .lprof 的使用，书中未说明

---

# 内存优化工具

1. python -m memory_profiler julia_lineprofiler.py

        Filename: julia_lineprofiler.py
        
        Line #    Mem usage    Increment  Occurences   Line Contents
        ============================================================
            57   38.281 MiB -251.266 MiB         225   @timefn
            58                                         @profile
            59                                         def calculate_z_serial_purepython(maxiter, zs, cs):
            60                                             """Calculate output list using Julia update rule"""
            61   38.281 MiB -289.527 MiB         225       output = [0] * len(zs)
            62   38.281 MiB -56175.379 MiB       25650       for i in range(len(zs)):
            63   38.281 MiB -55881.410 MiB       25425           n = 0
            64   38.281 MiB -55881.410 MiB       25425           z = zs[i]
            65   38.281 MiB -55881.410 MiB       25425           c = cs[i]
            66   38.281 MiB -441406.484 MiB      169605           while abs(z) < 2 and n < maxiter:
            67   38.281 MiB -385520.926 MiB      144180               z = z * z + c
            68   38.281 MiB -385524.625 MiB      144180               n += 1
            69   38.281 MiB -55885.844 MiB       25425           output[i] = n
            70   38.281 MiB -293.969 MiB         225       return output
        
2. mmemory_profiler中的一个功能叫mprof，可以用于对内存使用情况进行采用和画图
> * 具体怎么用，只说需要在代码里加入 `with profile.timestamp("create_xxx")`，添加标签。之后怎么出图没有说

3. heapy可以调查堆上的对象(他是guppy项目中的一个工具)
> * pip install guppy
> * 需要修改代码：
> 
>         from guppy import phy
>         hp = hpy()
>         ...
>         hp.setrelheap() # 可选项，用来创建一个内存断点，当后续调用hp.heap()时就会产生一个跟这个断电的差额。这样你就可以略过断点前由python内部操作导致的内存分配
>         ...
>         h = hp.heap()

4. 用dowser实时画出变量的实例（对于大公司的OBP版本不实用，存在一些违规风险，存在对外暴露监听的问题）
> * 对于自娱自乐的web服务器，可以用dowser和dozer，将对象行为实时可视化

### 遗留问题
1. memory_profiler中的mprof，要搞清楚用法，怎么出图





