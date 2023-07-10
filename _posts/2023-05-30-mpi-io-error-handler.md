---
layout: post
title: MPI IO错误处理
date: 2023-05-30
tags: fortran mpi io
---

本周遇到一个MPI IO的问题让我对MPI标准在遇到错误时如何处理有了更深入的了解，在此记录一下。

## 问题

>问题是这样的：有一个MPI并行程序，IO部分是通过MPI IO函数并行读入(`MPI_FILE_OPEN()`)一个文件，但程序执行时在没有找到该文件情况下仍然继续执行。由于我没对MPI_FILE_OPEN函数返回码进行检查，程序正常运行，但读入数组实际上值是0，根本不是期望的值。出问题后还不容易查找。

这与我对文件IO出错处理理解不一样，比如对于Fortran open语句默认找不到文件程序会报错终止。

## 原因

通过查阅MPI标准说明，发现MPI标准规定了MPI库运行时一旦侦测到错误时会终止程序运行，但文件操作例外。

**MPI IO 产生错误不会像通信错误一样被认为是致命的（MPI_ERRORS_ARE_FATAL），仅仅是捕捉这些错误（函数返回码）并继续执行程序。**

MPI标准预定义了两个错误handlers: 

- MPI_ERRORS_ARE_FATAL  致命错误，一旦侦测到，调用MPI_ABORT退出
- MPI_ERRORS_RETURN  仅仅返回错误码给用户，其它没有影响


MPI IO文件处理的缺省预定义错误handler是MPI_ERRORS_RETURN，因此在遇到如“文件找不到”错误时不会终止程序。

MPI标准提供了`MPI_FILE_SET_ERRHANDLER`函数设置文件错误handler。

```fortran
MPI_FILE_SET_ERRHANDLER(FILE, ERRHANDLER, IERROR)
  INTEGER FILE, ERRHANDLER, IERROR
```

该函数第一个参数需要文件句柄FILE，但只有MPI_FILE_OPEN函数成功执行后才能获得文件句柄。好在MPI标准提供了一个预定义的常量`MPI_FILE_NULL`用于还没有有效文件句柄时。


## 解决方法

在读该文件前设置错误handle为MPI_ERRORS_ARE_FATAL

```fortran
call MPI_FILE_SET_ERRHANDLER(MPI_FILE_NULL, MPI_ERRORS_ARE_FATAL, ierr)

call MPI_FILE_OPEN(...)
```

再次执行程序时，侦测文件不存在，程序报错并输出错误信息如下：

>I/O error: File does not exist, error stack:
  ADIOI_NFS_OPEN(69): File foo.dat does not exist

另一种方法是对MPI_FILE_OPEN函数返回码进行判断，函数未成功执行则终止程序程序 

```fortran
 if (ierr /= MPI_SUCCESS) then
   call MPI_ABORT()
 endif
```

---

## 补充

MPI标准还提供了查询当前默认文件错误句柄函数：

通过`MPI_FILE_GET_ERRHANDLER`函数，其中第一个文件句柄参数可以指定为`MPI_FILE_NULL`。

```fortran
MPI_FILE_GET_ERRHANDLER(FILE, ERRHANDLER, IERROR)
  INTEGER FILE, ERRHANDLER, IERROR
```

此外，MPI标准还支持自定义错误处理方法，但我还没发现有多大用处，感兴趣的可以查阅MPI标准说明了解。
