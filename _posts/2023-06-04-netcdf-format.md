---
layout: post
title: netCDF数据格式介绍
date: 2023-06-04
tags: netcdf 
---

netCDF经过多年的发展，支持多种数据格式，其格式发展如下图所示。

- Classic format: CDF-1、CDF-2、CDF-5
- netCDF-4 format: 底层二进制格式是HDF5 （需要HDF5 ≥ 1.8.0）

![netCDF数据格式的发展历程](https://s1.vika.cn/space/2023/03/26/5eaf4a90fa864730ac4a8e9c3d9082ec)

## netCDF格式安装要求

### netCDF-4支持

netCDF库构建时配置参数加上`--enable-netcdf-4`

### netCDF-4 parallel I/O

为了netCDF-4库支持并行I/O，依赖库HDF5必须使用`--enable-parallel`选择配置，使用MPI编译器编译。

netCDF编译使用与HDF5相同的MPI编译器。

### Classic netCDF parallel I/O

使用PnetCDF库并行读写classic格式，包括CDF-1,CDF-2,CDF-5。

netCDF-4版本如使用PnetCDF支持并行I/O，需配置`--enable-pnetcdf`选项。

## 创建指定netCDF格式方法

使用库API `n*_create()`创建netCDF文件，通过指定参数cmod设置。

- nc_create()  （C接口）
- nf_create() （fortran 77接口）
- nf90_create() （Fortran 90接口）

| netCDF格式                    | C接口           | Fortran90接口     | Fortran77接口   |
| ----------------------------- | --------------- | ----------------- | --------------- |
| CDF-2  (64-bit Offset format) | NC_64BIT_OFFSET | NF90_64BIT_OFFSET | NF_64BIT_OFFSET |
| CDF-5  (64-bit Data format)   | NC_64BIT_DATA   | NF90_64BIT_DATA   | NF_64BIT_DATA   |
| netCDF-4                      | NC_NETCDF4      | NF90_NETCDF4      | NF_NETCDF4      |

## 参考资料

1. [NetCDF Users Guide: An Introduction to NetCDF (ucar.edu)](https://docs.unidata.ucar.edu/nug/current/netcdf_introduction.html#classic_format)
2. [PnetCDF (parallel-netcdf.github.io)](https://parallel-netcdf.github.io/)
