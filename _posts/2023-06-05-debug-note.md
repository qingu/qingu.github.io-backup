---
layout: post
title: forrtl error 78
date: 2023-06-05
tags: fortran intel debug
---

## 问题描述

模式使用128个进程运行很快报错退出，测试其它进程配置也会得到相同的错误，多次运行出现相同的错误，排除了机器原因。

![](https://s1.vika.cn/space/2023/03/28/7352ea4787a543aab3722134fcdcbc9c)


## 调试平台和配置

### 平台环境

- Intel CPU, 32 cores/node, 192GB内存
- Linux /Intel Fortran Compiler 2018/Intel MPI


### 试验配置

模式网格：NX=3301, NY=3301, NZ=70

程序使用MPI并行，针对Y方向进行一维并行剖分，主要变量数据结构var(nx,nz,nb) ，其中nb=NY/nprocs。

## 排查经过

### 定位报错源代码

由于代码没有加调试选项编译，不能看到最后报错的调用栈对应的源代码信息。

使用intel fortran编译器选项`-g -traceback`重新编译代码运行，得到报错源码信息。

![](https://s1.vika.cn/space/2023/03/28/1032daaa2bc64b9987b95022efe444bc)

如上图所示，程序在调用`cmpclddrv`子程序（cmpclddrv.f第一行）时报错。

![](https://s1.vika.cn/space/2023/03/28/5dff6d34b0a74358a2574459ef4522d1)

从出错调用栈信息可以看出出错点在进入`cmpclddrv`程序时，还未执行该子程序内执行语句。

### 排查数组越界问题

当时考虑程序在传参时出现问题，比如数组越界。因此，加上数组越界编译选项`-check bounds`重新编译、运行，发现不是数组越界问题。

### 函数栈空间不够可能

重新查看报错信息，其中intel程序错误返回码引起了关注。

```bash
forrtl: error (78): process killed (SIGTERM
```

网上查阅**78**错误码代表的意义：

> In all cases it was due to some sort of "watchdog" process killing programs it thought were "runaway". Talk to your system/cluster administrator. It's an outside process that is killing your job.

猜测可能是内存耗尽导致程序被监控软件杀掉，程序出错点在子程序调用时，因此怀疑是自动数组过大引起的。

```fortran
  REAL :: tem1     (nx,nz,ny)
  REAL :: tem2     (nx,nz,ny)
  REAL :: tem3     (nx,nz,ny)
```

子程序cmpclddrv传入的哑元`nx,ny,nz`控制子程序自动数组在栈空间里分配空间大小。

测试以下解决方法是否有效：

- 作业脚本中增加`ulimit -s unlimited`
- 增加`-heap-arrays`编译选项，重新编译运行，使自动数组分配到heap堆上，防止栈空间不够。
- 测试更多MPI进程数并行（到1024进程），使nz这个哑元值更小。

以上方法仍未解决问题。

当MPI进程数为1024时，y方向并行剖分，一个进程分到的y方向格点最多是3301/1024 ~ 4。三维格点数最多是924280个，一个单精度浮点类型数组大概3.5MB。按理说占用的内存空间不大，不应该是自动数组导致的出错。

### 局部数组惹的祸

仔细查看该子程序的哑元及局部变量声明，发现几个可疑变量，类似于

```fortran
REAL :: var1(nx,nyy,nz)
REAL :: var2(nx,nz,nyy)
...
```

其中变量nyy不是通过子程序参数传递进来的，而是一个模块全局变量，其值等于Y方向总格点数，在这个case里是3301。

如果按照以上声明方式，存在两个问题使程序报错。

1. 由于nyy不是哑元变量，导致其参与声明的数组是局部变量，而不是自动数组，因此`-heap-arrays`对其无效，该数组在调用函数栈空间创建。
2. 由于nyy是未并行剖分的Y方向格点数，导致该数组声明是整个domain大小，每个进程中一个三维数组占用2.8GB左右，每个节点使用32个进程运行，占用栈内存巨大。

### 解决方法 

查看这些变量用途发现用于零号进程与其他进程之间数据的分发、收集。因此只需要零号进程声明整个domain大小即可。

```fortran
if (myid == 0) then !master process 
  allocate(var1(nx,nyy,nz))
else
  allocate(var1(1,1,1))
endif
```


## 总结

- 重视程序报错信息，了解错误返回码代表的含义和出错源代码行对程序调试非常重要。
- 对于MPI并行程序，寻找合适的调试工具可能会加快调试进程。


## 参考资料

1. https://community.intel.com/t5/Intel-Fortran-Compiler/forrtl-error-78-process-killed-SIGTERM/td-p/1007113
2. http://cali2.unilim.fr/intel-xe/compiler_f/GUID-44448B78-2B87-4998-9828-C8BAEB9F5C9A.htm
