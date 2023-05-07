---
layout: post
title: Fortran现代化
date: 2023-05-07
tags: fortran
---

## 前言

Fortran作为一门非常古老的高级语言（1957年由IBM公司Backus等人设计实现），至今仍是科学工程计算领域主要编程语言之一。

但由于Fortran没有跟上互联网时代现代软件开发实践，Fortran受欢迎程度在降低，使用场景逐渐被C++、Python等语言取代。大学里开设Fortran编程课程的也不多，导致Fortran生力军缺乏。

下图是编程语言排行榜[TIOBE Index](https://www.tiobe.com/tiobe-index/fortran/)统计的Fortran语言二十多年来的热门程度。最好的排名是在2002年排到了第10名；最低位置在2020年，第50名；目前最新排名是26名。

![TIOBE Index of Fortran]({{ site.url }}/images/posts/2023-05-07/tiobe_index_of_fortran.png)

## Fortran优点

Fortran语言以下设计理念及特点仍使其作为追求高性能数值计算的首选语言。目前气象数值模式领域依然将其作为主要编程语言。

 - 简单易学
 
  正如Fortran名字由来“公式翻译器”，Fortran面向数值计算领域科学计算，语法简单易学，能够让科学家快速写出高性能的代码。Fortran强类型声明、指针的限制使用（虽然指针是C语言的最大特色）使程序不容易出错。

- 高性能 

Fortran与C/C++性能处于同一级别。作为静态强类型的编译型语言，编译器可以进行充分优化。同时语言本身支持的并行功能（如do concurrent）和外部并行APIs（MPI、OpenMP、OpenACC）使其能够充分利用集群硬件性能。

- 可移植性

ISO标准保证了Fortran跨平台可移植性。语言的向后兼容性保证了远古代码（比如大量的Fortran 77遗留代码）在现在仍然可以运行。

- 顽强的生命力

Fortran语言标准委员会仍在矜矜业业推动着Fortran现代化，继续向前发展（比如面向对象、与其它语言混编）。

## Fortran现代化

Fortran语言社区也意识到与现代编程语言在生态、工具方面的差距，比如缺少功能强大的标准库、没有简单易用的包构建分发工具、没有社区维护的编译器及Fortran语言官方网站。

因此Fortran语言社区启动以下几个项目使Fortran语言能够继续保持旺盛的生命力：
- Fortran Standard Library(stdlib),
- Fortran Package Manager(fpm),
- LFortran编译器
- Fortran官网及论坛


![From Curcic et al., 2020]({{ site.url }}/images/posts/2023-05-07/fortran_community.png)


### stdlib - Fortran标准库

学习C++语言，可能大部分时间是在学习C++的标准库；Python用户也有大量的标准库和第三方库使用。标准库和第三方库的使用可以让开发者避免重复造轮子，提高开发效率。而最新的Fortran 2018标准定义了168个内置程序和少量的常数、派生类型。这些内置程序不包括通用算法、容器等C++、Python语言标准库常见功能。此外，Fortran标准对字符串、I/O处理功能支持较弱。

Fortran社区开启stdlib项目，通过Github进行协作开发。stdlib处于密集开发状态，最新开发进展可以到Github项目仓库查看。

![From Shaffer et al., 2021]({{ site.url }}/images/posts/2023-05-07/stdlib.png)

项目地址：[fortran-lang/stdlib: Fortran Standard Library (github.com)](https://github.com/fortran-lang/stdlib)

API文档：[Fortran-lang/stdlib](https://stdlib.fortran-lang.org/)


### FPM - Fortran包管理器

Python用户可以使用pip、conda工具方便的安装、分发第三方库包，这些工具能自动解决依赖包安装依赖关系。而Fortran软件包安装常通过autoconf、make构建。

Fortran社区启动fpm项目创建专属于Fortran的构建系统和包管理器。

fpm具有Bootstrap特色，只需要使用Fortran编译器构建，最小化依赖其它库和软件。

![From Ehlert et al., 2021]({{ site.url }}/images/posts/2023-05-07/fpm.png)

项目地址：[fortran-lang/fpm: Fortran Package Manager (fpm) (github.com)](https://github.com/fortran-lang/fpm)

### LFortran - 社区维护编译器 

Fortran有许多开源（GCC）和商业编译器（Intel 、PGI等），编译器质量也非常高，为何Fortran社区要再开发一套编译器呢？

Fortran社区希望能够自己维护一套编译器用于为Fortran语言标准化提供前期原型验证，同时基于LLVM编译器框架开发现代化的Fortran编译器，支持在多种架构平台（CPU、GPU）编译及交互式编程（如Python、Julia）功能。

注：Intel one API编译器也正在基于LLVM框架开发。

LFortran目前还处于开发阶段，试用发现还不稳定，很多语法标准还不支持，但未来值得期待。

项目地址：[LFortran - LFortran](https://lfortran.org/)

开发仓库地址：[lfortran / lfortran · GitLab](https://gitlab.com/lfortran/lfortran)

### Fortran-lang社区网站 

网址：[Home - Fortran Programming Language (fortran-lang.org)](https://fortran-lang.org/)

论坛：https://fortran-lang.discourse.group/

## FortranCon国际会议

值得关注的Fortran语言的国际会议：FortranCon，目前已举办两届（2020，2021），由瑞士 Zurich大学主办。

- [FortranCon 2020 ](https://tcevents.chem.uzh.ch/event/12/)
- [FortranCon 2021](https://tcevents.chem.uzh.ch/event/14/)

## 结语

作为一名数值模式开发者，预计未来十年Fortran依然是我的主要开发编程语言。因此有必要对Fortran语言标准、社区发展动态保持关注。通过学习使用这些开源项目，有助于提高编程开发的效率，更好的理解Fortran语言。

## 参考资料

1. Curcic M, Čertík O, Richardson B, et al. Toward modern Fortran tooling and a thriving developer community[J]. arXiv preprint arXiv:2109.07382, 2021.
2. [The State of Fortran 2021 (uzh.ch)](https://tcevents.chem.uzh.ch/event/14/contributions/67/attachments/69/155/FortranCon2021_Fortran-Lang_State_of_Fortran.pdf)
3. [What's new in the Fortran standard library? (uzh.ch)](https://tcevents.chem.uzh.ch/event/14/contributions/65/attachments/74/160/stdlib-talk.pdf)
4. [Fortran package manager - Toward a rich ecosystem of Fortran packages (uzh.ch)](https://tcevents.chem.uzh.ch/event/14/contributions/68/attachments/68/153/ehlert210924.pdf)
