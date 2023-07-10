---
layout: post
title: 新博客模板
date: 2023-05-25
tags: fortran
---

## 字符与数值类型转换

### 通过ASCII表转换

常用的字符与整数一一映射的编码表是ASCII表，规定了256个字符与0-255整数一一对应关系。

下表是Fortran用于两者互相转换的内置函数，其中ACHAR与CHAR、IACHAR与ICHAR区别在于一个指定使用ASCII表，另一个是处理器使用的编码表，不一定是ASCII表。但通常两者是等价的。
| 内置过程名      | 功能                                       |
| --------------- | ------------------------------------------ |
| ACHAR(I)        | 返回整数对应的ASCII码表中字符                |
| CHAR(I,[,KIND]) | 返回给定整数对应的处理器使用的编码序列字符 |
| IACHAR(C)       | 返回ASCII表中字符对应的十进制整数          |
| ICHAR(C)                | 返回处理器使用的编码表中字符对应的整数             |


ASCII码表中字符A-Z、a-b、0-9都是有序、连续排列的，利用这个特点我们可以对字符串进行大小比较、**大小写转换**等操作。

下面示例程序实现了一个转换大写字符串的功能。

```fortran
module string_utils
  implicit none
contains
  function to_upper(str)
    implicit none
    character(len=*),intent(in)  :: str
    character(len=len(str))      :: to_upper
    integer :: i,char2i

    do i = 1, len(str)
      char2i = iachar(str(i:i))
      if (char2i >= 97 .and. char2i <= 122) then
        to_upper(i:i) = achar(char2i - 32) !大小写字母相差ASCII十进制 32
      else
        to_upper(i:i) = str(i:i)
      endif
    enddo

  end function to_upper
end module string_utils

program test
  use string_utils
  implicit none
  character(len=24) :: str_in,str_out

  str_in = "ABCadc099DEF"
  str_out = to_upper(str_in)
  print*, "str_out = ",str_out
end
```

### Fortran与C字符串区别

C字符串有null字符’\0’作为字符串的结尾符，而Fortran不需要。因此，Fortran与C混编时，Fortran字符串传给C函数需要加上结尾符`string // char(0)`

### 字符串与数值类型转换

我们从文本文件中读入数值型数据及将数据写入文本文件都会有一个隐式的类型转换，在文本文件中都是字符形式存储，而在内存中可以是整型、实型数据，编译器调用内部函数进行了隐式类型转换。

如果我们需要自己显式类型转换，比如我想将关于日期时间的年、月、日等整型变量组成一个字符串变量用于文件名的一部分。我们可以使用Fortran中“Internal file”功能，想象有一个内部文本文件存放在内存中（而不是磁盘上），将数据写入这个文件就完成了数值类型到字符串的转换，反之从内部文件读入则可以将字符串转换为数值类型。

```fortran
  character*10  :: cdate
  integer       :: iyear,imon,iday
  
  iyear =  2022
  imon  =  7
  iday  =  21
  write(cdate, '(I4.4,A1,I2.2,A1,I2.2)') iyear,'-',imon,'-',iday

  print*, 'cdate=',cdate
```

---

## 删除字符串空格

一般我们声明字符串变量长度时不会完全与实际占有长度相同，多出来的空间一般会用空格占据，但处理字符串过程中往往需要删除这些空格，我们可以使用以下内置函数。

| 内置过程名      | 功能   |
| --------------- | ------ |
| ADJUSTL(STRING) | 字符串左对齐，即去除左端空格 |
| ADJUSTR(STRING) | 字符串右对齐，即去除右端空格 |
| TRIM(STRING)    | 去除字符串尾部空格       |

通过以下程序来查看三个函数的区别。

```fortran
  character*128 :: str3

  str3 = '       hello, fortran     '
  print*, 'BEG',str3,'END'
  print*, 'BEG',trim(str3),'END'
  print*, 'BEG',adjustl(str3),'END'
  print*, 'BEG',adjustr(str3),'END'
```

注意对齐函数虽然将其中一端的空格去除，但会在另一端补上空格。因此对于一个两端都有空格的路径字符串变量，需要配合trim函数去除两端空格。

```fortran
  filename = '     /home/username/a.txt     '
  open(99, file=trim(adjustl(filename)), ...)
```


## 计算字符长度 

计算字符串变量长度也是常见操作。

| 内置过程名                    | 功能                               |
| ----------------------------- | ---------------------------------- |
| LEN                         | 计算字符串长度                       |
| LEN_TRIM(STRING)              | 计算去掉尾部空格的字符长度             |

两个字符长度计算函数常用于对字符串变量遍历操作。

```fortran
character(len=128) :: str 

do i = 1, len(str)
  str(i:i) = char(i)
enddo
```

同时两个函数是在编译时执行(compile-time)，因此能够用于字符串变量声明，可以用于声明与传入字符串参数相同长度的字符变量。

```fortran
subroutine length(str1)
  implicit none
  character(len=*)              :: str1
  character(len=len(str1))      :: str2
  character(len=len_trim(str1)) :: str3

  print*, 'len(str1) = ', len(str1)
  print*, 'len(str2) = ', len(str2)
  print*, 'len(str3) = ', len(str3)
end subroutine length

program main
  implicit none
  character(len=16) :: str1

  str1 = '0123456789'

  call length(str1)
end
```

---

## 字符串匹配

| 内置过程名                    | 功能                               |
| ----------------------------- | ---------------------------------- |
| INDEX(string,substring)     | substring在string中第一个匹配的开始位置，不匹配返回0       |

可以增加可选参数BACK并设为.TRUE. ，返回最后一个匹配结果的开始位置。这个功能可以用于从一个包含路径的文件名变量中获取dirname或者basename，匹配最后一个'/’路径分隔符即可。

下面是使用index功能进行大小写字符转换另一种实现（摘自stdlib_ascii）

```fortran
character(len=*),parameter :: uppercase = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
character(len=*),parameter :: lowercase = "abcdefghijklmnopqrstuvwxyz"
character(len=1) :: c

k = index(uppercase, c)
if(k > 0) then
  t = lowercase(k:k)
else 
  t = c
endif
```

一个实际的例子是WRF后处理软件ARWpost中处理指定气象场。
变量plot_these_fields值通过namelist文件指定 

```
fields = 'height,geopt,theta,tc,tk,td,td2'
```

代码中用index函数确定某个气象场是否在这个字符串里来决定是否处理该气象场。

```fortran
IF (INDEX(plot_these_fields, ",pressure," /= 0))THEN
   ...
ENDIF

IF (INDEX(plot_these_fields, ",geopt," /= 0))THEN
   ...
ENDIF
```
