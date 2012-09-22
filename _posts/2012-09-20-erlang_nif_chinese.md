---
layout: post
title: "erlang nif 中文手册"
description: ""
category: erlang
tags: []
---
{% include JB/setup %}

## 前言
  
   这是翻译erlang官方文档中的 erts-5.9.2的erl_nif部分。尽量每日更新。（最后更新时间2012.09.22）

## erlang nif 中文手册
---
* [概括][]
* [功能][]


[概括]: #sum
[功能]: #functionality

-----


#概括   <a name="sum"></a>   
NIF库包含了erlang模块的一些方法的原生实现。这些NIF方法的调用方式跟其他普通方法的调用一样，但是每个NIF函数都要用erlang对应的实现，如果NIF库成功载入，在调用NIF函数之前，会先调用对应的erlang实现的函数。


一个典型的用法是：根实现方法用作抛出异常。有时候当果系统没有实现任何NIF库，它也可以用作一个备份。

下面是一个最小调用NIF库的例子：


c部分：

	/* niftest.c */
	#include "erl_nif.h"
	static ERL_NIF_TERM hello(ErlNifEnv* env, int argc, const ERL_NIF_TERM argv[])
	{
		return enif_make_string(env, "Hello world!", ERL_NIF_LATIN1);
	}
	static ErlNifFunc nif_funcs[] =
	{
		{"hello", 0, hello}
	};
	ERL_NIF_INIT(niftest,nif_funcs,NULL,NULL,NULL,NULL)
	


erlang部分：

	-module(niftest).
	-export([init/0, hello/0]).
	init() ->
		erlang:load_nif("./niftest", 0).
	hello() ->
		"NIF library not loaded".


在linux上的操作如下：

	$> gcc -fPIC -shared -o niftest.so niftest.c -I $ERL_ROOT/usr/include/
	$> erl
	1> c(niftest).
	{ok,niftest}
	2> niftest:hello().
	"NIF library not loaded"
	3> niftest:init().
	ok
	4> niftest:hello().
	"Hello world!"


对于一个真正使用的模块，一个更好的实现办法是利用``on_load``  > >  ```注：这里的on_load是erlang的一个描述符，跟-export一样，通常用法是: -on_load(init/0) ```,在erlang模块被load的时候，会调用指定方法把c模块也load进。

**注意：
NIF库不一定要被exproted的，可以作为erlang模块私有。  
同时注意一些没有用的函数会被优化时候处理掉，导致调用时候出错。
**


一个NIF库被load时候是绑定erlang模块的version。如果erlang模块更新了，新模块要去加载多一次NIF作为自己的（或选择不去加载，这样的话，新的erlang模块并没有NIF）。如果一个NIF被多次load和调用的话，意味着NIF里面的static data同样会被共享。为了避免这样的无心的共享static data，每个erlang模块代码应该要保存自己的私有数据，而在NIF里面，可以通过调用``void *enif_priv_data(ErlNifEnv* env)``来获得私有数据。

没有方法去声明unload一个NIF库，当erlang模块被卸载时候，NIF会被自动unload。







#功能   <a name="functionality"></a>
NIF库所有函数有以下几种功能：  

* __读写erlang的terms__  

	所有的erlang terms都可以通过作为函数的参数或者返回值来进行erlang和NIF库的传递。erlang的terms对应的是一个叫`ERL_NIF_TERM`的c结构体，而且只能通过调用特定的API才能读写erlang terms。大部分读terms的API都是`enif_get_`前缀的。而大部分写terms的API都是`enif_make_`前缀，而通常会把写的那个`ERL_NIF_TERM`作为返回值返回。还有一些API是用来监狱terms的，例如`enif_is_atom`,`enif_is_identical`,`enif_compare`。   
	
	
	所有`ERL_NIF_TERM`都属于一个代表NIF环境的类型：`ErlNifEnv`。所有terms的生命周期都由该`ErlNifEnv`来控制。所有读写terms的函数的第一个变量都是这个`ErlNifEnv`。
  	
  		
  		
		
  		
* __二进制__  
  
	二进制类型的terms可以通过一个结构体`ErlNifBinary`来操作。这个结构体包含一个指向原始二进制数据的指针和这个数据在字节长度。数据和长度都是只读的类型而且只能通过调用API函数去写（`注：调用某些方法后，会使他们变成可读`）。然而通常会实例一个`ErlNifBinary`然后分配给用户（通常是本地变量）。		
	指针指向的原始数据只有通过调用`enif_alloc_bianry`或者`enif_realloc_bianry`之后才能变得可改变。其他所以操作binary的函数都是只读。一个可以改变的binary在最后必须要调用`enif_release_binary`来释放或者调用`enif_make_binary`转换成只读的erlang terms。只读的binary是不需要被释放。在同样的NIF call里面可以使用`enif_make_new_binary`分配和返回一个binary。binaries会被序列化全部字节。bitstrings 还不支持任意bit长度。
	
	
---
by dp

  

  


