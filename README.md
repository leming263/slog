# slog
A simple log library
一个简单的程序本地日志库(4个API.一个源文件,一个头文件)

支持功能:
1.日志分级.目前分为verbose,debug,info,err,fatal等五个等级，打印日志时设定。
2.日志过滤.在打开日志结构时设置过滤级别.
3.日志滚动.支持日志的自动滚动，可以设置滚动下标上限和单日志文件大小
4.时间粒度.支持设置日志的不同粒度，默认粒度到秒，可以到毫秒，微秒及纳秒
5.动态修正.在进程过程中可以动态修改打开日志时的一些基本属性
使用方法：将slog.h slog.c加入源码编译即可。
注意：该日志库目前非线程安全，多线程类型程序需斟酌使用.


API:
1）int slog_open(char *log_name , SLOG_LEVEL filt_level , SLOG_DEGREE log_degree , int log_size , int rotate , char *err);
用于程序中打开一个日志控制结构。成功则返回控制结构描述符，失败返回-1并填充错误信息到err
log_name:用作日志的文件名.如果为空则打印到标准输出
filt_level:设定日志过滤级别，低于该级别的日志则不会输出。具体宏请参见slog.h
log_degree:日志记录粒度。默认为秒。具体请参见宏slog.h
log_size:单个日志文件大小.如果填0则默认为10M.
rotate:日志滚动下标上限。如果填0则使用默认值9
err:错误时返回错误信息。

说明：相同文件名(包括路径)只能打开一次。另外的不同日志文件可以在程序里打开，返回不同的描述符


2）int slog_close(int sld);
关闭一个已经打开的日志控制描述符

3）int slog_log(int sld , SLOG_LEVEL log_level , char *fmt , ...);
打印一条日志
sld:通过slog_open调用成功之后返回的描述符
log_level:该条日志的等级
fmt及...：日志的模式及内容 形如printf

4）int slog_chg_attr(int sld , int filt_level , int degree , int size , int rotate);
动态更新已打开描述符的属性
sld:已打开的描述符
filt_level:新的过滤等级.-1则忽略
degree:新的日志粒度. -1则忽略
size:新的单个日志文件大小 -1则忽略
rotate:滚动下标上限 -1则忽略

备注：
以上所有API调用过程中产生的错误，以及调试信息都被打印在主进程的执行目录下slog.log.*中

附简单压测：
gcc -g use_slog.c slog.c -lm [-lrt] -o use_slog
环境：
CPU：Intel(R) Xeon(R) CPU E5-2682 v4 @ 2.50GHz 单核
MEM：2G
OS：CentOS 2.6.18-308.el5

数据：
循环打印100万条（1\*1024\*1024）日志数据,单条数据长度210Byte左右
单条日志长度10M.滚动上限9

1)调整日志粒度为秒级:
测试结果：成功率100%
CPU USAGE:99%
MEM COST:2720K
usr_time:1.86s
sys_time:0.25s
total_time:2.11s

2)调整日志粒度为纳秒级:
测试结果：成功率100%
CPU USAGE:99%
MEM COST:2736K
usr_time:2.19s
sys_time:0.32s
total_time:2.52s

3)调整日志数量为1000万条（10\*1024\*1024） 其余测试条件不变
测试结果：成功率100%
CPU USAGE:90%
MEM COST:2752K
usr_time:18.89s
sys_time:3.41s
total_time:24.77s

