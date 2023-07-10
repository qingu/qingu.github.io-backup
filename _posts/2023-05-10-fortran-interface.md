---
layout: post
title: Fortran过程接口
date: 2023-05-10
tags: fortran
---

## 前言

为了避免英文术语翻译成中文带来的歧义，首先解释下本文涉及的Fortran术语。

- procedure 

过程，简单理解就是指Fortran的函数（function）和子例程（subroutine）

- function 

函数，与C函数相同，在表达式中被调用评估（evalute）返回函数值

- subroutine 

子例程，CALL语句调用，没有返回值，类似C语言中返回类型是void的函数

- procedure interface 

过程接口，类似C语言中函数声明 

- dummy argument 

哑元，也可以称为虚参，过程定义时参数

- actual argument 

实参，过程调用时传入的参数

---

## 过程接口含义及分类

每个过程都有一个接口说明：包括过程名称和特征，每个哑元的名称、特征及可能存在的过程被引用的通用generic标识符。

上述过程特征包括：
- 函数还是子例程
- 如果是函数，返回值特征
- 哑元特征 
- 是否是pure/impure/elemental
- 是否有BIND属性


哑元特征包括（也就是类型、属性登）：
- 类型和类型参数（有的话）
- shape/intent
- 是否是optional
- 是否是allocatable
- 是否是pointer或target
- 是否polymorphic
- 是否是assumed rank或assumed type
- ...

**过程接口分为显式（explicit） 和隐式（ implicit）。**

- 如果调用程序知道被调用过程的特征，过程接口就是显式的，否则就是隐式的（为保证链接成功编译器会从过程引用和声明中推导得到）。

过程接口概念很*类似C语言中的函数声明*。如果过程定义和调用不在同一个编译单元中，没有显式过程接口的话编译器无法检查传递参数的正确性，即使成功编译链接生成可执行文件，执行时可以有莫名其妙的错误。

我们来看下面这个示例： 

```fortran
!!! add.f90
integer function add(a,b)
  implicit none
  integer,intent(in) :: a,b

  add = a + b
end function add


!!! main.f90
program main
  implicit none
  integer,external :: add

  print*,add(2.0,3.0)
end
```

构建运行命令如下：
```bash
$ gfortran -c add.f90
$ gfortran -o main.exe main.f90 add.o
$ ./main.exe
 -2143289344
```

以上主程序只是简单的调用了一个外部加法函数，但该函数定义要求哑元是整型，但实际调用时实参却传入了实型，预期结果是5.0，但却得到了一个奇怪的数-2143289344。

原因在于浮点数实参传给哑元被当做了整型看待。由于过程定义和调用不在同一个编译单元内，编译器无法检查参数传递的正确性。

解决方法就是添加显式接口说明帮助编译器检查。

- 方法一：将add函数定义也放到main.f90中使其在同一个编译单元内。

- 方法二：在调用处添加显式接口说明

```fortran
!!! main_new.f90
program main
  implicit none
  interface
    integer function add(a,b)
      implicit none
      integer,intent(in) :: a,b
      end function add
  end interface

  print*,add(2.0,3.0)
end
```

和上面一样的构建命令编译发现编译器报错，信息如下：

```bash
main_interface.f90:10:9:

   10 |   print*,add(2.0,3.0)
      |         1
Error: Type mismatch in argument ‘a’ at (1); passed REAL(4) to INTEGER(4)
main_interface.f90:10:9:

   10 |   print*,add(2.0,3.0)
      |         1
Error: Type mismatch in argument ‘b’ at (1); passed REAL(4) to INTEGER(4)
```

编译器检查出实参与哑元类型不一致，报错。

- 方法三：将add函数放到一个module文件中，编译器会自动添加interface说明。

## Fortran各种过程类型 

以下是Fortran各种过程其接口类型的统计。

| 过程类型                        | 接口类型 |
| ------------------------------- | -------- |
| 外部过程（external procedure）  | implicit |
| 模块过程（module procedure）    | explicit |
| 内部过程（internal procedure）  | explicit |
| 内置过程（intrinsic procedure） | explicit |
| 哑元过程 （dummy procedure）    | implicit |
| 语句函数（statement function）  | implicit |

对于默认为隐式（implicit）接口类型的过程我们可以通过interface语句显式指定。

```fortran
interface
  [interface_body]
end interface 
```

关于interface更多功能以后再讨论。

---

附：以上6种过程类型可以通过下面这个示例程序理解：

```fortran
subroutine external_proc()
  implicit none

  print*, "external_proc is a external procedure"
end subroutine external_proc

integer function add_int(x,y)
  implicit none
  integer,intent(in) :: x,y

  add_int = x + y
end function add_int

module mod_a
  implicit none
  abstract interface
    integer function simple_math(x,y)
      implicit none
      integer,intent(in) :: x,y
    end function simple_math
  end interface
contains
  subroutine mod_proc()
    print*, "mod_proc a module procedure"
  end subroutine mod_proc
  integer function do_math(a,b,op)
    implicit none
    integer,intent(in) :: a,b
    procedure(simple_math),pointer  :: op

    do_math = op(a,b)
  end function do_math
end module mod_a

program main
  use mod_a
  implicit none
  real :: a,b,c
  real :: minus,m,n
  minus(m,n) = (m-n)
  interface
    integer function add_int(x,y)
      implicit none
      integer,intent(in) :: x,y
    end function add_int
  end interface
  procedure(add_int),pointer :: funcp => null()


  a = 2.0

  ! internal procedure by host assoication using contains
  b = square(a)

  ! sqrt is a intrinsic procedure from Fortran Standard
  c = sqrt(b)

  ! external procedure
  call external_proc()

  ! module procedure
  call mod_proc()

  ! statement function, only one statement
  write(0,*) "statement function minus result",minus(3.0,2.0)

  ! dummy procedure
  funcp => add_int
  c = do_math(1, 2, funcp)

  print*,'c=',c
contains
  real function square(x)
    implicit none
    real,intent(in) :: x

    print*, "square is a internal procedure"
    square = x * x
  end function square
end
```

