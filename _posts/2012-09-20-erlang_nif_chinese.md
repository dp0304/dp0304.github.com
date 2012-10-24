---
layout: post
title: "erlang nif 中文手册"
description: ""
category: erlang
tags: []
---
{% include JB/setup %}

## 前言
  
   这是翻译erlang官方文档中的 erts-5.9.2的erl_nif部分。尽量每日更新。（最后更新时间2012.10.24）

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
	 
	 
* __资源对象__  
	
	使用资源对象的一个比较安全的方式是调用NIF借口把指向该数据的指针返回，一个资源对象实际上只是一块通过 `enif_alloc_resource `分配的内存。一个处理方法是通过使用` enif_make_resource `把这块内存（指针）返回给erlang。`enif_make_resource`的返回值是完全不透明的，它可以在同节点内不同进程间传递和存储，唯一的正确用法是在最后把返回值返回给NIF，然后NIF可以调用`enif_get_resource`取得指针。直到最后一个`ERL_NIF_TERM`被虚拟机回收和资源已经被`enif_release_resource`释放的时候，资源对象才会被回收。   
	
	
	所有的资源对象都必须带有资源类型，这样可以使得资源在不同模块时候仍然可以区分出来。当库被加载时候调用`enif_get_resource`去创建资源类型。当对象带有资源类型后，可以通过`enif_get_resource`来校验是否是期待的类型。一个资源类型可以用户自定义的析构函数（借用C++）。资源类型是唯一定义的string名字和唯一的实现模块。
	
	
        ERL_NIF_TERM term;
        MyStruct* obj = enif_alloc_resource(my_resource_type, sizeof(MyStruct));
        /* initialize struct ... */
        term = enif_make_resource(env, obj);
        if (keep_a_reference_of_our_own) {
        	/* store 'obj' in static variable, private data or other resource object */
        }
        else {
       		 enif_release_resource(obj);
       		 /* resource now only owned by "Erlang" */
        }
        return term;
        
      
   
   
   	注意一旦`enif_make_resource`创建了term返回给Erlang之后，有两个选择，一是保存它的指针到分配的结构体里面以后再释放。二是在垃圾收集时候释放它。   
   	另外一种使用资源对象的方法的是使用定义的内存来创建binary terms，`enif_make_resource_binary`会创建binary terms并连接一个资源对象。当该binary terms 被回收时候后会调用资源对象的析构函数，同时该binary terms可以被释放。  
   	
   	资源类型支持运行时升级，通过运行允许一个被加载的库去接管已经存在的资源类型，并继承所有已经存在的对象，而且对象的类型都会变成那个接管的资源类型.新的库的析构函数从那以后就可以被继承的对象继承。这样的话，就的析构函数就可以被安全卸载。资源对象和模块被坑下后，必须被删除或者被新的NIF接管。卸载的库一直被阻塞直到所有已存在的资源对象被析构或是被新的NIF接管。   
   	
   	
* __多线程和并发__   
	一个NIF库是线程安全的，只要函数的动作只是单纯地读提供的数据，太是不需要直接声明任何同步锁。但是如果你对一个静态变量或者`enif_priv_data`进行写操作，你就要实现自己的同步锁操作。在进程独立环境里面的terms可以被多线程共享。资源对象需要锁。库在初始化时候的回调函数 `load` ,`reload` 和`upgrade`里就算是共享的静态数据也是线程安全的。__避免在NIF里面进行耗时的操作，这样会降低VM的相应速度。Erlang代码调用NIF时候是直接调同样调度线程去执行，调度会因此阻塞，直到返回。__    
   	
   	

	
---
by dp

  

  


