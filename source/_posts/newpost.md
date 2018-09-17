---
title: i春秋61挑战 wp
date: 2018-06-05 00:51:34
tags:
    编程技术
---
在i春秋 [61挑战](https://bbs.ichunqiu.com/thread-41125-1-1.html)出了道简单的签到题
题目是一个损坏的pyc文件


## wp
首先生成一个正常pyc文件  `python -m py_compile $filename ` 对比如下
![winhex](/images/winhex_pyc.png)

可以发现文件头被破坏 对照正常pyc文件修复. 然后使用工具对pyc进行反编译 比如 https://tool.lu/pyc/ 

得到反编译代码
```
_ = (lambda .0: continue[ chr(i ^ 51) for i in .0 ])((85, 95, 82, 84, 72, 67, 74, 80, 74, 86, 64, 91, 90, 67, 74, 78))
```

打印 `_` 得到flag `flag{pycyeshipy}` 


题目很简单 但是不知道为什么只有两位表哥做出来
