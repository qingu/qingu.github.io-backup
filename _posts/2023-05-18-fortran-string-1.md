---
layout: post
title: 新博客模板
date: 2023-05-18
tags: fortran
---

Fortran中使用character声明单个字符或多个字符组成的字符（串）变量。Fortran标准也提供了一些内置函数用于字符串处理，但总体来说Fortran字符串处理能力和C语言一样都不是很强。

**说明：可以将character声明的变量理解成字符串变量。**

## 字符串变量声明与赋值

### 声明

```fortran
!字符常量
character(len=5),parameter :: constr = 'Hello'
character :: ch    !单个字符变量
character(len=10) :: str  !长度为10的字符（串）变量
character*10 :: str1  !使用*length 方式声明
character*10 :: str_arr(10) !长度为10的字符数组变量，数组有10个元素
```

推荐使用`character(len=length)`声明字符串变字符串。

### 赋值

```fortran
str = 'nice to'
str1 = ' meet you'
str_arr(1) = str   !字符串变量之间赋值
```

**将一个字符串变量值赋给另一个字符串变量，不要求两个字符串变量长度相同**，如果短字符串赋给长字符串，长字符串后面用空格占据；如果长字符串赋值给短字符串，字符串截断赋值给短字符串。

```fortran
character(len=128) :: str2
character(len=5)   :: str3

str2 = str   ! 'nice to_____________________'
str3 = str   ! 'nice '
```

### 子串substring

提取字符串中一部分内容，语法很像子数组语法（如果将字符串看成由字符组成的数组的话），但有些区别。

```fortran
str(1:2)  !str前两个字符子串
str(3:3)  !str第3个字符
str(3:)   !str第3个到最后一个字符子串
```

但以下操作是不允许的(毕竟它不是数组)：

```fortran
str(3)
str(1:5:2)
```

### 假定长度字符类型

假定长度字符类型(assumed-length character type )使用`*` 表示假定长度类型

```fortran
character(len=*) ...
```

字符长度来自实际参数(作为哑元情况)或者常量表达式(parameter)(编译器在编译时确定)

```fortran
module str_mod
  implicit none
contains
  function join(str_a, str_b)
    implicit none
    !assumed-length character type dummy
    character(len=*),intent(in) :: str_a,str_b
    character(len=len(str_a)+len(str_b)) :: join

    join = str_a // str_b
  end function join
end module str_mod

program main
  use str_mod
  implicit none
  !assumed-length character parameter
  character(len=*),parameter :: a = "hello world!"
  character(len=12) :: b

  b = " I am a coder"
  print*,"new string size is ",len(join(a,b))
  print*,"new string is ",join(a,b)
end
```

### 推迟长度字符类型

推迟长度字符类型(deferred-length character type)使用`:` 表示推迟长度类型

```fortran
character(len=:) ...
```

Fortran 2003引入，**动态长度字符类型，字符长度运行时可变**，因此必须拥有pointer或者 allocatable属性。

```fortran
character(len=:),allocatable :: alloc_name
character(len=:),pointer :: ptr_name 

!注意分配推迟长度字符类型语法
allocate(character(len=5) :: alloc_name, ptr_name)
alloc_name = 'Name'
ptr_name => another_name
```

和其他allocatable或pointer变量使用没什么区别，只是在allocate时语法有点特别。

将上面假定长度字符程序稍微修改一下，函数返回类型改为推迟长度字符类型。

```fortran
module str_mod
  implicit none
contains
  function join(str_a, str_b)
    implicit none
    !assumed-length character type dummy
    character(len=*),intent(in) :: str_a,str_b
    !deferred-length allocatable function result
    character(len=:),allocatable :: join

    join = str_a // str_b
  end function join
end module str_mod

program main
  use str_mod
  implicit none
  !assumed-length character parameter
  character(len=*),parameter :: a = "hello world!"
  character(len=12) :: b

  b = " I am a coder"
  print*,"new string size is ",len(join(a,b))
  print*,"new string is ",join(a,b)
end
```

**动态类型变量join没有显式分配，而是利用alloctable变量在内置赋值语句中的自动分配功能。**


## 字符串连接符//

使用`//`可以将两个字符串拼接成一个字符串，常用于文件路径和文件名的拼接操作。

```fortran
str // str2
dirname // filename
```

需要注意的是：

- 连接后的字符串长度是原来两个字符串长度之和。
- 连接操作不会自动删除字符串中的空格字符，在路径连接操作中一般使用内置函数trim配合删除路径空格，`trim(trim(dirname)//filename)`

下一篇再介绍关于字符串的内置函数功能及日常工作中经常用到的处理文件路径等操作。
