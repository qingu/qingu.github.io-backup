---
layout: post
title: NetCDF文件压缩功能
date: 2023-06-01
tags: netcdf fortran
---

## 无损压缩zlib

netCDF4格式支持**无损**压缩算法DEFLATE压缩文件节省文件存储空间。DEFLATE算法使用Lempel-Ziv编码和Huffman编码组合。

无损压缩是相对于有损压缩说的，指压缩完的数据能够完全还原恢复。

netCDF4/HDF5使用zlib库提供的压缩算法，这也是netcdf4安装需要zlib库原因。gzip压缩工具也是用的zlib压缩算法。

压缩等级（即deflation level）从无压缩等级0到最大压缩等级9，数值越大，压缩率越高，但所需压缩时间也会越长。

按照NCO文档说法，从最低等级（dfl_lvl=1）到最高等级（dfl_lvl=9），文件大小差异一般小于10%。对于一般的气候数据集deflation算法能够压缩30-60%。

常用的两个nc4压缩命令：

- nccopy （netcdf4库自带命令）

```bash
$ nccopy -4 -d N in.nc out.nc  # N-压缩等级,0-9
```

- ncks （来自NCO工具包）

```bash
$ ncks -4 -L N in.nc out.nc   # N-压缩等级，0-9
```



## 数据测试

### 试验准备

以一个模式初始场数据为例，原始数据格式是netcdf 64bit-offset。

- 通过nccopy转换为nc4无压缩格式

```bash
$ du -k test_64bit.nc 
33322660     test_64bit.nc #大概32GB大小
$ nccopy -4 test_64bit.nc test4.nc 
33322702     test4.nc     #nc4格式文件稍微大一些
```

- 测试各等级压缩率和压缩时间

测试脚本：

```bash
#!/bin/bash

for lvl in `seq 1 9`
do
    echo $lvl
    time ncks -4 -L $lvl test4.nc test4_$lvl.nc
done
```

### 试验结果 

| 压缩等级 | 压缩后文件大小/KB | 压缩率% | 压缩时间/s |
| -------- | ----------------- | ------ | -------- |
| 0        | 33322702          |       |          |
| 1        | 16324542          |  48.99      |   804       |
| 2        | 16203218          | 48.63       |   781       |
| 3        | 16101553          | 48.32       |   844       |
| 4        | 16083338          | 48.27       |   862       |
| 5        | 16013387          | 48.06       |   934       |
| 6        | 15953534          | 47.88       |   1097       |
| 7        | 15928392          | 47.80       |   1243       |
| 8        | 15895986          | 47.70       |   2080       |
| 9        | 15884182          |  47.67      |   3887       |

注：压缩率公式$$compress\_rate = \frac{压缩后文件大小}{压缩前文件大小}$$

### 结果验证

使用`nccmp`工具验证压缩前后结果是否相同，是否是无损压缩。

## 有损压缩 

### szip技术

使用szip库压缩netCDF4/HDF5文件。

以下结果来自Yepes-Arbós and Acosta(HPCM 2021)测试结果。SZ压缩率比gzip高很多，但压缩时间没有太大变化。

![](https://s1.vika.cn/space/2022/10/29/ef3c1b06dd484555adab1ae01cafa0ef)

### Packing技术

有损压缩技术Packing能够很容易达到50%压缩率。Packed数据再用deflation压缩能压缩到80%。

```bash
$ ncpdq in.nc out.nc    #标准packing ，~50%压缩率
$ ncpdq -4 -L 9 in.nc out.nc  #Deflated packing， ~80%压缩
```
