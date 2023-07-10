---
layout: post
title: shell脚本计时方法
date: 2023-06-02
tags: shell bash
---

本文收集几种对shell脚本计时的方法。

测试脚本如下：

```bash
#!/bin/bash
# timing_case.sh
echo "start now"
sleep 10
echo "end now"
```

- 方法1：`time`命令

如果是对整个shell脚本计时，使用`time`命令即可。

```bash
$ time ./timing_case.sh
start now
end now

real    0m10.014s
user    0m0.007s
sys     0m0.000s
```

其中`real`时间就是我们关心的脚本运行时间。

- 方法2：`date`命令

如果需要知道shell脚本中某一段运行时间，可以使用`date`命令。

可以在shell代码段前后加上`date +%s`，然后相减就得到运行时间，如下所示

```bash
#!/bin/bash

start_time=`date +%s`
echo "start now"
sleep 10
echo "end now"
end_time=`date +%s`
runtime=$((end_time-start_time))
echo "elapsed time is " $runtime
```

`date +%s`是获得从19700101 00:00:00  UTC到当前时间的秒数，时间差就是运行时间。但以上计时精度为秒，如果需要更细粒度的计时，可以使用`date +%s.%N`，（`%N`是纳秒精度），使用`bc`命令可以进行浮点数相减。

```bash
#!/bin/bash

start_time=`date +%s.%N`
echo "start now"
sleep 10
echo "end now"
end_time=`date +%s.%N`
runtime=$( echo "$end_time - $start_time" | bc -l )
echo "elapsed time is " $runtime
```

脚本执行输出：

```
start now
end now
elapsed time is  10.003331300
```

- 方法3：特殊变量`SECONDS`

bash、ksh等使用特殊变量`SECONDS`。

```
#!/bin/bash

start_time=$SECONDS
echo "start now"
sleep 10
echo "end now"
end_time=$SECONDS
runtime=$((end_time-start_time))
echo "elapsed time is " $SECONDS
```

- 方法4：封装函数

将需要计时代码封装为函数，然后使用time命令计时。

```bash
func() {
   do something
}

time func "$@"   # $@传入脚本参数
```


## 参考资料

[bash - How to get execution time of a script effectively](https://unix.stackexchange.com/questions/52313/how-to-get-execution-time-of-a-script-effectively)
