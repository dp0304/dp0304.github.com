---
layout: post
title: "erlang socket reuseaddr选项说明"
description: ""
category: erlang
tags: []
---
{% include JB/setup %}

erlang socket option里面有一个参数 {reuseaddr,Boolean} ，某人很纠结，硬是试了很多，查了很多。我就开篇说说清晰点。   
  
一切的开始从socket的一个状态说起， 当双方建立socket之后，其中一方发起断开时候，会进入“四次断开”的流程，这个流程中，端口状态会进入TIME_WAIT。   
 
最常见的情况是，kill掉你的server。  server被kill掉，socket没有走完“四次断开”流程，端口会进入TIME_WAIT状态，（linux 内核回收这些端口需要几分钟吧，没细挖这个，片面的曾经见过的经验），如果这时候重启了server，重新监听该端口时候，会出现端口占用的错误。

{reuseaddr,true}便是处理这一个情况，它机制是：当linux 内核返回TIME_WAIT状态时候，便可以复用监听。   

  

这里需要说明 erlang 中 reuseaddr 和linux的 SO_REUSEADDR 的区别。   

linux 的  SO_REUSEADDR 有四种效果：   
1） 当端口处在TIME_WAIT时候，可以复用监听   
2） 可以允许多个实例（进程）监听同一端口，但是必须不同ip   
3） 允许单进程监听同一端口，但是必须不同ip  
4） 使用udp时候，可以允许多个实例或者但进程同时监听同个端口同个ip   

 

erlang socket gen_tcp option中，有一个{ip,{0,0,0,0}}选项，可以允许监听不同ip的同个端口，如上面的2）和3）

而erlang 的reuseaddr 则对于以上的 1）   


--EOF