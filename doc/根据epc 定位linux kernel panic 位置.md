根据epc 定位linux kernel panic 位置

参考：

https://blog.csdn.net/han_dawei/article/details/41846055

https://blog.csdn.net/oqqyuji12345678/article/details/121090932

https://blog.csdn.net/jasonchen_gbd/article/details/45585133


* 1, 打开System.map, 找到epc之前的最近函数的地址.计算出epc距离该函数的偏移值. 

* 2, 使用objdump 找到该函数, 分析 epc 偏移处的汇编代码. 

* 3, 打开源代码, 根据分析汇编代码得到的信息进行定位. 