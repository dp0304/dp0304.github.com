---
layout: post
title: "unity3d scene和prefab合并工具"
description: ""
category: unity3d
tags: []
---
{% include JB/setup %}

unity3d的scene和prefab都是二进制，虽然也可以改成文本模式，但是在冲突时候，确实不好解决。只能回退，一个一个轮流加上去。   
这也是大部分团队采用的方式，沟通好，避免冲突。   
	
但是冲突总会出现，怎么合并呢？
推荐一个插件[UniMerge](https://www.assetstore.unity3d.com/#/content/9733)   。[介绍文档](http://wiki.unity3d.com/index.php/Unity_Merge)

可以合并scene和prefab。它把每一条属性都形象地列举出来，比较容易merge。    



$15算是一般吧，亲测 win下 unity4.3可用，git绑定可用。   
	
截图：   

![2](/image/unity_scene_merge/b.png)    

![1](/image/unity_scene_merge/a.png)    
 

