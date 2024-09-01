lmbench作为性能检测工具的一种，提供内存，网络，内核等多方面的测试工具。是benchmark众多功能测试软件中的一种。几天了解了下，记录于此。

# 参考链接

http://www.bitmover.com/lmbench/

http://elinux.org/Benchmark_Programs

http://processors.wiki.ti.com/index.php/Lmbench

http://blog.csdn.net/dianhuiren/article/details/7331777

http://kongll.github.io/2015/04/24/LMbench/


# 下载

https://sourceforge.net/projects/lmbench/

# 移植

根据需要更改CC和OS

make CC=$(TOOL_CHAIN_PREFIX)-gcc OS=arm-linux TARGET=linux

编译之后生成的可执行文件在

/bin/arm-linux

# 测试

root@Tony:~/lmbench/bin/arm-linux# ./bw_mem 1M rdwr

1.00 209.40

root@Tony:~/lmbench/bin/arm-linux# ./bw_mem 1M cp  

1.00 191.86

详情查看链接处的ti文档。