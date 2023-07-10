---
layout: post
title: Fortran Interface更多功能
date: 2023-05-11
tags: fortran
---

## 过程重载

过程重载**类似C++中的函数重载功能**，使用相同的过程名称，但根据接口匹配情况编译器调用不同的过程。

编译器会根据实际调用过程的参数、返回值特征在接口块中匹配某个过程，如果匹配不上编译器会报错。

过程重载定义要求接口块中多个过程的接口要有区别，不能有歧义，否则编译器不能确定使用哪个过程。

一般会将多个功能相同、但传递参数区别的过程写在一个接口块中，实现过程重载功能。Fortran内置函数重度使用该功能，比如三角函数sin(x)可以对单精度、双精度、复数的参数x求sin，对应的函数是sin(x)、dsin(x)、csin(x)等。

**过程重载常用于函数库的编写**，提供统一过程名称供用户使用。比如我们读写NetCDF数据时经常要根据处理变量的类型（整型、单双精度实型）、不同维数数组传递不同的参数给NetCDF提供的库函数。我们可以利用过程重载功能将这些细节进行封装，根据传递参数的不同编写多个处理过程，然后封装成一个接口过程供用户使用，用户不用处理这些繁琐的细节。

以下是两个外部过程iadd,radd通过过程重载功能供主程序调用，我们既可以使用add这个接口名称，也可以直接使用特定的过程，比如对于整形参数直接调用iadd。

```fortran
! generic_procedure.f90
integer function iadd(a,b)
  implicit none
  integer,intent(in) :: a,b

  iadd = a + b
end function iadd

real function radd(a,b)
  implicit none
  real,intent(in) :: a,b

  radd = a + b
end function radd

program main
  implicit none
  interface add
    integer function iadd(a,b)
      integer,intent(in) :: a,b
    end function iadd
    real function radd(a,b)
      real,intent(in) :: a,b
    end function radd
  end interface add

  integer :: x =3, y = 4
  real    :: m =3., n = 4.

  print*,add(x,y)
  print*,iadd(x,y)
  print*,add(m,n)
  print*,radd(m,n)

 end
```

一般情况下我们不希望用户直接调用具体的某个过程，希望他们使用统一的接口。因此，我们利用模块封装的思想，将其写在模块中，利用public/private控制访问权限。

```fortran
!  module_generic_procedure.f90
module mod_add
  implicit none
  private
  public :: add   !只提供公开的过程接口
  interface add
    module procedure :: iadd
    module procedure :: radd
  end interface add
contains
  integer function iadd(a,b)
    implicit none
    integer :: a,b

    iadd = a + b
  end function iadd

  real function radd(a,b)
    implicit none
    real :: a,b

    radd = a + b
  end function radd
end module mod_add

program main
  use mod_add
  implicit none

  integer :: x =3, y = 4
  real    :: m =3., n = 4.

  print*,add(x,y)
  print*,add(m,n)

 end
```


## 操作符扩展

Fortran利用interface语法提供操作符扩展功能。

```fortran
interface operator(op)
  function foo()...
end interface 
```

其中操作符op可以是
 - 对已有的操作符的功能扩展，比如+ , -, >等
 - 定义新的操作符，比如(.MINUS. C)

*限制：接口块中过程只允许是函数*

**这个功能常用于面向对象编程中，针对类对象重载操作符。**

以下是字符串使用“+”操作符实现连接功能的示例。

```fortran
! generic_operator.f90
character(len=120) function conc(a,b)
  implicit none
  character(len=*),intent(in) :: a,b

  conc = trim(a)//trim(b)
end function conc

program main
  implicit none
  character(len=60) :: str1,str2
  interface operator(+)
    character(len=120) function conc(a,b)
      implicit none
      character(len=*),intent(in) :: a,b
    end function conc
  end interface

  str1 = "hello "
  str2 = "world!"
  print*,str1+str2

end
```


## 赋值操作操作 

**这个功能常用于面向对象编程中，针对类对象之间使用“=”进行赋值， obj1 = obj2。**

语法：

```fortran
interface assignment (=)
  subroutine foo() ...
end interface
```

**要求：接口块中过程只允许是子例程，且只有两个非可选参数，第一个参数intent属性为OUT或INOUT，第二个参数intent为IN**


## 过程指针与抽象接口

**过程指针类似C语言中的函数指针。** 过程指针可以关联（指向）外部过程、模块过程、内置过程和哑元过程。

Abstarct interface，定义一个抽象接口用于后面PROCEDURE(interface_name)语句中指定接口名称，之所以叫它为抽象接口，是因为定义时还未绑定实际的过程，只是描述了拥有相同的参数特征的一组过程。**推迟绑定**，直接后面过程指针指向某个过程或者哑元过程接收过程实参。

抽象接口与过程指针组合使用，过程指针可以动态的指向不同的函数，同一个接口实现不同的功能。

```fortran
module mod_op
  implicit none
  ! 定义抽象接口do_math，传入参数是integer,intent(in)的两个整型变量的一组函数
  abstract interface
    integer function do_math(a,b)
      integer,intent(in) :: a,b
    end function do_math
  end interface
contains
  integer function add(a,b)
    implicit none
    integer,intent(in) :: a,b

    add = a + b
  end function add
  integer function minus(a,b)
    implicit none
    integer,intent(in) :: a,b

    minus = a - b
  end function minus
end module mod_op

program main
  use mod_op
  implicit none
  ! 声明指向do_math接口的过程指针funcp
  procedure(do_math),pointer :: funcp => null()
  integer :: x =3, y = 4

  ! 过程指针指向add函数，后面就可以使用funcp代替add
  funcp => add
  print*,funcp(x,y)
  ! 过程指针重新指向minus函数
  funcp => minus
  print*,funcp(x,y)
 end
```

## 哑元过程 

将过程作为一个参数传给另一个过程调用，**类似C语言回调（callback）函数。**

```fortran
! dummy_is_procedure.f90
module mod_op
  implicit none
  abstract interface
    integer function do_math(a,b)
      integer,intent(in) :: a,b
    end function do_math
  end interface
contains
  integer function add(a,b)
    implicit none
    integer,intent(in) :: a,b

    add = a + b
  end function add
  integer function minus(a,b)
    implicit none
    integer,intent(in) :: a,b

    minus = a - b
  end function minus

  integer function do_something(a,b,op)
    implicit none
    integer,intent(in) :: a,b
    procedure(do_math) :: op

    do_something = op(a,b)
  end function do_something
end module mod_op

program main
  use mod_op
  implicit none

  print*,do_something(2,3,add)

  print*,do_something(2,3,minus)
end
```


可以和过程指针结合起来使用，不直接传入add/minus函数，而是通过过程指针指向它们，然后传入过程指针。

```fortran
program main
  use mod_op
  implicit none
  procedure(do_math),pointer :: funcp => null()

  ! 过程指针funcp指向add函数
  funcp => add
  print*,do_something(2,3,funcp)

  funcp => minus
  print*,do_something(2,3,funcp)

  nullify(funcp)
end
```
