---
layout: post
title: 新博客模板
date: 2023-07-10
tags: NWP HPC 
---

NWP领域最近有两件很重要的事，一件是发表在Nature上的华为盘古大模型研究成果《Accurate medium-range global weather forecasting with 3D neural networks》，另一件事是ECMWF在6月27日将IFS升级到版本48r1，将集合预报系统分辨率升级到全球9公里，与确定性模式分辨率一致。这两件事分别代表了新兴的人工智能方法和传统数值模式对中期全球天气预报的最新研究进展。 

本文主要关注ECMWF IFS升级，以及计算相关内容。 

## IFS升级内容

IFS这次升级是将中期集合预报（ENS）的水平分辨率从原先18公里升级到9公里，使得集合预报成员的分辨率与高分辨率预报（HRES）分辨率一致。垂直分辨率仍保持137层不变。集合预报成员数保持51个不变。

另外一个主要升级是延伸期集合预报（ENS extended）配置。过去是作为中期预报的扩展，每周启动两次，每次预报15天。升级后变成完全独立系统，每天00UTC运行一次，使用101个成员预报46天。但其分辨率仍保持不变：水平36公里、垂直137层。

升级后，48r1 将提供两组后报（hindcast），一组用于中期预报，另一组用于延伸期预报。

下表是IFS Cycle 48r1系统分辨率配置。IFS是一个耦合系统，包括大气、海洋、海浪分量。

![来自ECMWF](https://s1.vika.cn/space/2023/07/09/987459d6e89f486d8e802b8a8c042de8)

大气模式参数配置如下表所示。 

![来自ECMWF](https://s1.vika.cn/space/2023/07/09/b11633404c114622a439a0e53dafd7ab)

注：48r1 GRIB2产品使用CCSDS压缩方法，建议使用ecCOdes 2.30.0或更新版本处理。

## 所需计算资源 

在本次的升级描述中未提到需要的计算资源使用情况，只能从其它信息源推测。

在WGNE（Working Group on Numerical Experimentation）官网有一份2022年的各大业务中心模式计算资源情况。其中ECMWF的IFS确定性预报TCo 1279(9公里)/137层，在博洛尼亚的Atos Bull Sequana高性能计算平台上使用16384核预报5天需要18分钟。

博洛尼亚新超算具体情况见之前的文章介绍。

按照该数据进行简单推测： 

- 升级后的IFS ENS单个成员预报15天，需要**54分钟**； 
- 每个ENS成员需要16384核，51个成员需要835,584核，再加上HRES预报，总共需要**851,968核**。

但根据ECMWF 2022年度报告描述，新超算核数大概是100多万核，其中25%计算能力提供给成员国，多达10%保留给特殊项目，即使剩下的计算资源全部用于业务的话，是不能满足升级后的ENS运行的。

**不负责任猜测：**IFS进行了计算优化，使得业务运行所需核数降低或者通过降低计算核数使得业务运行时间变长，但仍能满足业务时效性。

## Remark 

ECMWF将集合预报分辨率提高到了全球9公里，进一步巩固了其NWP领先地位。高分辨率ENS业务运行证明了ECMWF计算优化团队的优秀。

## 参考资料 

1. [Accurate medium-range global weather forecasting with 3D neural networks | Nature](https://www.nature.com/articles/s41586-023-06185-3)

2. [Implementation of IFS Cycle 48r1 - Forecast User - ECMWF Confluence Wiki](https://confluence.ecmwf.int/display/FCST/Implementation+of+IFS+Cycle+48r1)

3. https://www.ecmwf.int/sites/default/files/elibrary/2023/81359-annual-report-2022.pdf

4. [wgne.net/nwp-systems-wgne-table/wgne-table/](https://wgne.net/nwp-systems-wgne-table/wgne-table/)
