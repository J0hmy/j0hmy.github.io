---
title: a mac bug about udp port triggered by srs
date: 2021-06-28 17:31:04
tags:
---

在srs上,有一个bug困扰我很长时间了, 最近才分析清楚其原因.

### 1 问题概述

在srs中开启rtc,打开浏览器多次publish 和 play 后, 出现ice协商失败的情况; 
调试发现udp阻塞所致;
更换 rtc 端口可暂时解决问题.

### 2 问题分析
在srs中阻塞函数是来自st的st_recvfrom, 可以从`srs/trunk/research/st` 入手, 设计如下测试 

问题组
```
server: g++ udp-server.cpp ../../objs/st/libst.a -g -O0 -o udp-server && ./udp-server 127.0.0.1 8000 3
client: g++ udp-client.cpp ../../objs/st/libst.a -g -O0 -o udp-client && ./udp-client 127.0.0.1 8000 3
```

对照组
```
server: g++ udp-server.cpp ../../objs/st/libst.a -g -O0 -o udp-server && ./udp-server 127.0.0.1 18000 3
client: g++ udp-client.cpp ../../objs/st/libst.a -g -O0 -o udp-client && ./udp-client 127.0.0.1 18000 3
```

测试发现问题组server会卡住, 与srs情况一致

### 3 更换server的监听ip

将 server 端 ip 换成 `0.0.0.0`(即 `udp-server 0.0.0.0 8000 3` 和 `udp-server 0.0.0.0 3`)

测试结果没有变化;

初步结论是: bug 与 st 有关

### 4 调整client

将 client换成 nc, 具体如下

问题组
```
server: g++ udp-server.cpp ../../objs/st/libst.a -g -O0 -o udp-server && ./udp-server 127.0.0.1 8000 3
client: echo "Hello" | nc -4u $CANDIDATE 8000
```
对照组
```
server: g++ udp-server.cpp ../../objs/st/libst.a -g -O0 -o udp-server && ./udp-server 127.0.0.1 18000 3
client: echo "Hello" | nc -4u $CANDIDATE 18000
```

测试结果与之前一致.

### 5 调整server

再进一步调整测试

问题组

```
server: nc -u -l 8000
client: echo "Hello" | nc -4u $CANDIDATE 8000
```
对照组

```
server: nc -u -l 8000
client: echo "Hello" | nc -4u $CANDIDATE 18000
```

测试结果与之前依然一致. 不过,可以排除st的嫌疑了;

结论是: bug与st无关

那么,只能是与mac有关了

### 6 结论

综合上述各次测试对比, 这个bug与st无关,与操作系统mac有关. 

能力有限, 这个问题就不再往下分析了,就此记录一下.

我的系统是`macOS Big Sur 11.4`, 通过`uname -a`查看, 具体为 `Darwin Kernel Version 20.5.0: Sat May  8 05:10:33 PDT 2021; root:xnu-7195.121.3~9/RELEASE_X86_64 x86_64`

不过在升级系统前(macOS 10.15)也同样存在这个问题.

PS: 
1. CANDIDATE=$(ifconfig en0 inet| grep inet|awk '{print $2}')
2. 查看端口占用 lsof -i:8000
