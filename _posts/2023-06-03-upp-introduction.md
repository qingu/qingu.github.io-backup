---
layout: post
title: 模式后处理系统UPP介绍（1）
date: 2023-06-03
tags: NWP upp
---

Unified Post Processor (UPP)是美国业务数值模式使用的统一后处理系统，负责将模式输出数据转换生成业务数据产品。

- [项目主页](https://dtcenter.org/community-code/unified-post-processor-upp)
- [UPP Github主页](https://github.com/NOAA-EMC/UPP)
- [UPP最新用户文档](https://upp.readthedocs.io/en/latest/)

最新版本：v11.0.0 （截止：20230102）

## UPP支持模式

按照文档说明，UPP基本上支持美国所有的业务模式，包括全球、区域模式。

- Global Forecast System (GFS)
- GFS Ensemble Forecast System (GEFS)
- North American Mesoscale (NAM)
- Rapid Refresh (RAP)
- High Resolution Rapid Refresh (HRRR)
- Short Range Ensemble Forecast (SREF)
- Hurricane WRF (HWRF)
- Unified Forecasting System (UFS)，包括RRFS、HAFS、MRW、SRW

## UPP功能

- 读取模式输出数据
- 根据模式提供的气象场计算诊断量
- 垂直插值功能（比如模式面插值到等压面）
- 输出grib2格式的业务数据产品

**注：水平插值功能由外部程序wgrib2完成。**

## UPP实现

UPP是多进程MPI并行程序，并可支持OpenMP多线程并行。

UPP系统可以生成独立的可执行程序来处理模式输出数据，也可以编译成第三方库形式供模式的后处理部分在线调用。

美国GFS最新业务版本将UPP作为库方式在线调用，在运行模式时直接生成下发的grib2业务数据产品。

UPP系统编译安装方式已经开始基于CMake进行构建。

未完待续...


