# python_performance
《python高性能编程》相关学习代码

---

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
> 原生timeit工具，运行单个函数

       5 loops, best of 5: 33.4 usec per loop

4. /usr/bin/time -p python julia.py

       real         0.43
       user         0.06
       sys          0.06

5. kernprof -l -v julia_lineprofiler.py
> * 需要修改源代码，在需要测试的函数上加装饰器 @profile （无需import任何模块）
> * 用于检查被测函数中每一条语句的执行时间
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

