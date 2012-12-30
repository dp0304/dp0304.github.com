---
layout: post
title: "erlang nif 中文手册"
description: ""
category: erlang
tags: []
---
{% include JB/setup %}

## 前言
  
   这是翻译erlang官方文档中的 erts-5.9.2的erl_nif部分。尽量每日更新。（最后更新时间2012.12.31）

## erlang nif 中文手册
---
* [概括][]
* [功能][]
* [初始化][]
* [数据类型][]
* [接口-资源分配类][]
* [接口-线程操作类][]
* [接口-类型操作类][] 

[概括]: #sum
[功能]: #functionality
[初始化]: #initialization
[数据类型]: #datatype
[接口-资源分配类]: #exports-alloc
[接口-线程操作类]: #exports-thread
[接口-类型操作类]: #exports-type

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

	所有的erlang terms都可以通过作为函数的参数或者返回值来进行erlang和NIF库的传递。erlang的terms对应的是一个叫`ERL_NIF_TERM`的c结构体，而且只能通过调用特定的API才能读写erlang terms。大部分读terms的API都是`enif_get_`前缀的。而大部分写terms的API都是`enif_make_`前缀，而通常会把写的那个`ERL_NIF_TERM`作为返回值返回。还有一些API是用来判断terms的，例如`enif_is_atom`,`enif_is_identical`,`enif_compare`。   
	
	
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
   	 
   	 
   	 
   	 
#初始化   <a name="initialization"></a>     

* __ERL_NIF_INIT(MODULE, ErlNifFunc funcs[], load, reload,upgrade, unload)__   

	这是一个初始化NIF库的宏技巧，它应该赋值给一个静态变量。   		
	
	 `MODULE`是erlang的模块名，是用一串无空格的字符串作为标识，它由一个宏去字符串化。  
	 
	`funcs` 是一个静态的数组 ，用来存放描述NIF库中实现的所有的方法名。  
	
	`load`,`reload`,`upgrade`,`unload`是函数的指针，在NIF库初始化或者卸载，更新时候就会自动调用对应的方法。（下面有介绍）  
	
	
* __int (*load)(ErlNifEnv* env, void** priv_data, ERL_NIF_TERM load_info)__	
	该方法是当NIF库第一次载入时候到会调用。	
	
	`*priv_data` 是一个指向保存自由数据的指针，该区域可以在多次NIF中保持静态，当该方法调用时候会被初始化为`NULL`。`enif_priv_data`方法可以返回该指针。  
	
	`load_info`是`erlang:load_nif/2` 的第二个参数。  
	
	当库加载失败时候会返回非0，如果没有指定加载方法就把`ERL_NIF_INIT`的`load`设为`NULL`。   
	
	
* __int (*upgrade)(ErlNifEnv* env, void** priv_data, void** old_priv_data, ERL_NIF_TERM load_info)__  
		
	当已经载入旧库的库，而且要更新时候使用。  
	
	`*old_priv_data`已经包含了之前调用`load`和`reload`的值，而`*priv_data`会被初始化为`NULL`.  `*old_priv_data` 和 `*priv_data`都是可写的。	
	
	当更新失败时候返回非0，如果没有指定更新方法就把`ERL_NIF_INIT`的`upgrade`设为`NULL`。  
	
	
* __void (*unload)(ErlNifEnv* env, void* priv_data)__  
	
	当库属于过旧要清理的情况下使用，注意这里并不是替换库操作。  
	
	
	
* __int (*reload)(ErlNifEnv* env, void** priv_data, ERL_NIF_TERM load_info)__  

	__这个方法的实现机制是不可靠的，这只是一个开发者特性。不要使用该方法，实际环境中请把`reload`设为`NULL`__
	

	
	
#数据类型   <a name="datatype"></a>    
	
	
* __ERL_NIF_TERM__  
	
	`ERL_NIF_TERM`类型的变量可以引用任何erlang term，这是个不透明类型，而且它的值只可以用作api函数的参数或者返回值。所有的`ERL_NIF_TERM`都属于`ErlNifEnv`。一个term是不可以单独摧毁的，它会一直有效，直到它的环境被摧毁。  
	
	
	
* __ErlNifEnv__  
	
	`ErlNifEnv`被描述为erlang terms的宿主（生存环境），环境中所有的terms的生命周期都和该环境一样。`ErlNifEnv`是一个不透明类型，指向它的指针只可以在api函数中传递，这里有两种环境，进程绑定和进程独立。  
	
	一个进程绑定的环境，会在所有的NIF函数里面作为第一个参数传递，所有传递给	
	
	
* __ErlNifFunc__  
       typedef struct {
       const char* ;
       unsigned ;
       ERL_NIF_TERM (*)(ErlNifEnv* env, int argc, const RL_NIF_TERM argv[]);
       } ErlNifFunc;  
	
	通过它的名字，参数数量和实现去定义一个NIF，`fptr` 是一个指向NIF实现函数的指针。NIF的参数`argv`包含了erlang传递给NIF的全部参数，`argc`是参数数组的长度。例如：如果参数`argv[N-1]` 就表明是第N个参数。注意：这个`argc`可以使一个C的函数被多个erlang函数的调用，这些erlang调用时候可以用不同的参数长度。   
	
	
* __ErlNifBinary__  
       typedef struct {
         unsigned ;
         unsigned char* ;
       } ErlNifBinary;  
	
	
	`ErlNifBianry`包含一个检查二进制term的短暂信息，`data`是一个指向长度为`size`,存放二进制原生内容的buffer。  
	
	注意：`ErlNifBinary`是一个半透明的类型，只允许读字段`size`和`data`。  
	
	
	
* __ErlNifPid__   
	
	`ErlNifPid`是一个进程的身份，对比pid terms（`ERL_NIF_TERM`的实例），`ErlNifPid`是自身包含自己且为绑定任何 environment。是一个不透明类型。  
	
	
* __ErlNifResourceType__  
	
	每个`ErlNifResourceType`代表着一种内存管理资源对象，该可以可以被垃圾回收。每个资源类型都有一个唯一的名字和一个析构函数。对象销毁时候会自动调用析构函数。  
	
	
* __ErlNifResourceDtor__  
       typedef void ErlNifResourceDtor(ErlNifEnv* env, void* obj);  
	
	该函数原型包含一个资源的析构函数，这个资源析构函数不允许调用任何制造term的函数。  
	
	
* __ErlNifCharEncoding__  
       typedef enum {
         ERL_NIF_LATIN1
       }ErlNifCharEncoding;  
	
	这个字符转义用于string和atoms的转换，当前唯一支持的是`ERL_NIF_LATINL`(iso-latin-l,8-bit ascii)。  
	
	
* __ErlNifSysInfo__  
	用于`enif_system_info`去返回运行时系统的信息。包含当前精确的信息，跟`ErlDrvSysInfo`一样精确。  
	
	
* __ErlNifSInt64__  
	一个原生有符号的64bit长度的整形类型。  
	
	
* __ErlNifUInt64__  
	一个原生的无符号64bit长度的整形类型。  
	
	
	
#接口-资源分配类   <a name="exports-alloc"></a> 

* __viod * enif_alloc(size_t size)__  
	分配`size`个字节大小的内存，如果分配失败就返回`NULL`。  
	

* __void enif_free(void* ptr)__  
	释放通过`enif_alloc`分配的内存。  
	
		
	

	
* __ErlNifEnv *enif_alloc_env()__  
	分配一个进程独立了的环境，该环境必须用来存放没有绑定任何进程的terms。可以在稍后再通过`enif_make_copy`复制到一个进程环境，或者通过`enif_send`作为一个信息发送到一个进程上。  
	返回一个指向新空间的指针。  
	
* __void enif_clear_env(ErlNifEnv* env)__  
	清除环境中所有的term，使环境可以复用，注意这个环境必须是通过`enif_alloc_env`分配的。  
	
	
* __void enif_free_env(ErlNifEnv* env)__  
	析构环境，里面的term同样会被释放。  	
	
* __ErlNifResourceType *enif_open_resource_type(ErlNifEnv* env, const char* module_str, const char* name, ErlNifResourceDtor* dtor,ErlNifResourceFlags flags, ErlNifResourceFlags* tried)__  
	通过一个字符串`name`去创建或者接管一个资源类型。`dtor`是指向析构函数的函数指针（可以为空）。  
	而`flags`可以是下面的几个值：  
	ERL_NIF_RT_CREATE：  创建一个新的资源类型。  
	ERL_NIF_RT_TAKEOVER： 打开一个已经存在的资源对象，并且接管所有实例，而这个`dtor`会给已经存在的实例和将来创建的实例析构时候使用。  
	两个flags值可以通过位操作“与”来组合。资源类型的名字对于调用模块是局部的。`module_str`是目前未使用，记得设为`NULL`。  
	函数调用成功会返回一个指向资源类型的指针，然后`*tried`会被设成`ERL_NIF_RT_CREATE`或者`ERL_NIF_RT_TAKEOVER`去表明实际动作是什么。如果失败就返回`NULL`，并且把`*tried`赋值为`flags`的值。允许`tried`是`NULL`。  
	
	注意：该函数之允许在三个回调函数中调用（`load`,`reload`,`upgrade`）  
	
	
	
* __void *enif_alloc_resource(ErlNifResourceType* type, unsigned size)__  
	分配内存给资源对象，要求传入类型和大小。   
	
	
* __void enif_release_resource(void* obj)__  
	释放资源对象。  
	
	
* __ERL_NIF_TERM enif_make_resource(ErlNifEnv* env, void* obj)__  	
	对`enif_alloc_resource`分配的内存资源对象创建一个不透明处理。拥有权并没有发生转移，该资源对象还是需要用`enif_release_resource`释放。		
	注意如果在`enif_make_resource`分配term之后马上调用`enif_release_resource`的话，在term被垃圾回收时候，该资源对象会马上释放。  
	注意：在erlang编程里面，一个资源term只可以存储或者在同节点的不同进程间发送。其他操作如：匹配或者`term_to_binary`会有个不可以预知的结果。  
	
	
* __int enif_keep_resource(void* obj)__  
	对资源添加一个引用。该资源对象必须由`enif_alloc_resource`分配。  	
	
	
* __int enif_alloc_binary(size_t size, ErlNifBinary* bin)__  
	分配一个`size`个字节大小的二进制空间，初始化一个`ErlNifbinary`，把指针传入，会关联分配的空间。该二进制空间必须用`enif_release_binary`释放或者拥有者通过`enif_make_binary`转义成erlang的term。一个分配好空间的`ErlNifBinary`可以在多次NIF调用中保存下来。  
	成功返回`true`，失败返回`false`。  
	
	
* __int enif_inspect_binary(ErlNifEnv* env, ERL_NIF_TERM bin_term, ErlNifBinary* bin)__  
	通过`bin_term`的信息初始化`bin`指向的结构体。成功返回`true`，如果`bin_term`不是一个binary会失败。  
	
	
* __int enif_inspect_iolist_as_binary(ErlNifEnv* env, ERL_NIF_TERM term,ErlNifBinary* bin)__  
	`term`是iolist，通过`term`初始化bin指向的结构体，跟`enif_inspect_binary`一样，`bin`指向的数据生命周期很短不需要手动释放。   
	（这里我认为是将一个iolist的erlang字符串转换成一个erlang的binary字符串）   
	成功返回`true`，如果`term`不是连续buffer会失败。   



* __int enif_realloc_binary(ErlNifBinary* bin, size_t size)__  
	改变`bin`的大小。该binary可能是只读的，在这种情况下，会它会被抛弃，并且重新分配内存给`*bin`。（it will be left untouched and a
mutable copy is allocated and assigned to *bin） ` 这里重新分配后的binary可以使erlang看来变量可变了。因为Erlang中的binary数据与NIF C中操作的是同一块内存的数据。`  


* __void enif_release_binary(ErlNifBinary* bin)__  
	释放binary。  



	
* __ERL_NIF_TERM enif_make_binary(ErlNifEnv* env, ErlNifBinary* bin)__  
	创建一个binary term。拥有权会移交给创建的term。而`bin`会被认为只读的(实际上你还可以操作的，这只是建议)。  
	
	
* __unsigned char *enif_make_new_binary(ErlNifEnv* env, size_t size,ERL_NIF_TERM* termp)__  
	分配`size`大小字节的内存并创建term。这个binary的数据在NIF返回之前都是可变的。这是个快速创建binary的方法。缺点是该binary不能在多次NIF调用里保存，而且不可以重新分配内存大小。  
	
	
* __ERL_NIF_TERM enif_make_sub_binary(ErlNifEnv* env, ERL_NIF_TERM bin_term,size_t pos, size_t size)__  
	复制部分binary来创建新的binary。`pos`是位置，`size`是长度。（是用来在C中模仿erlang的字符串binary操作的）  


* __ERL_NIF_TERM enif_make_resource_binary(ErlNifEnv* env, void* obj, const void*data, size_t size)__  
	创建一个binary term，该binary term在被析构之前必须保持可读和不可变，可能会包含外部的资源对象，在这种情况下，析构函数里面就应该去释放那些对象。  
	多个binary terms可能属于同一个资源对象，那么该析构函数不会被马上调用，直到最后一个binary被垃圾回收后。如果要返回一个大binary buffer的不同部分，这是一个好方法，  
	该函数内，拥有权没有发送改变，该资源仍然需要`enif_release_resource`去释放。  
	
	
* __void enif_system_info(ErlNifSysInfo *sys_info_ptr, size_t size)__  
	该方法会写运行系统的信息到`ErlNifSysInfo`。  
		
	
	




#接口-线程操作类   <a name="exports-thread"></a> 
	
* __void enif_cond_create(ErlNifCond *cnd)__  	
	通过一个字符串创建一个条件变量，返回一个指向这个变量的指针，时候返回`NULL`，NIF在unloaded之前需要把这个条件变量销毁。  
	该方法线程安全。  
	
	
* __void enif_cond_destory(ErlNifCond *cnd)__  
	摧毁一个条件变量。该方法线程安全。  
	
	
* __void enif_cond_wait(ErlNifCond *cnd, ErlNifMutex *mtx)__  
	该函数会使线程阻塞知道其他线程唤醒。可以通过条件变量去唤醒单个或者全部线程。在线程阻塞之前它会解锁传递过来的互斥量，当被唤醒时候它会锁住改互斥量。所以在调用该方法前，那个互斥量需要先锁住。  
	该方法线程安全。  
	
	
* __void enif_cond_signal(ErlNifCond *cnd)__  
	该函数通过条件变量去单播。如果其他线程等待这个这个信号，其中一个都会被唤醒。  
	该方法线程安全。  
	
	
* __void enif_cond_broadcast(ErlNifCond *cnd)__  
	该函数通过条件变量去广播。如果其他线程等待这个这个信号，全部都会被唤醒。  
	该方法线程安全。  
	
* __int enif_send(ErlNifEnv* env, ErlNifPid* to_pid, ErlNifEnv* msg_env,ERL_NIF_TERM msg)__  
* __int enif_equal_tids(ErlNifTid tid1, ErlNifTid tid2)__  
	对比两个线程标识是否相同，不相同返回0，相同返回非0.  
	该方法线程安全。  
* __ErlNifMutex *enif_mutex_create(char *name)__  
* __void enif_mutex_destroy(ErlNifMutex *mtx)__  
* __void enif_mutex_lock(ErlNifMutex *mtx)__  
* __int enif_mutex_trylock(ErlNifMutex *mtx)__  
* __void enif_mutex_unlock(ErlNifMutex *mtx)__  

* __ErlNifRWLock *enif_rwlock_create(char *name)__  
* __void enif_rwlock_destroy(ErlNifRWLock *rwlck)__  
* __void enif_rwlock_rlock(ErlNifRWLock *rwlck)__  
* __void enif_rwlock_runlock(ErlNifRWLock *rwlck)__  
* __void enif_rwlock_rwlock(ErlNifRWLock *rwlck)__  
* __void enif_rwlock_rwunlock(ErlNifRWLock *rwlck)__  
* __int enif_rwlock_tryrlock(ErlNifRWLock *rwlck)__  
* __int enif_rwlock_tryrwlock(ErlNifRWLock *rwlck)__  

* __int enif_thread_create(char *name,ErlNifTid *tid,void * (*func)(void *),void *args,ErlNifThreadOpts *opts)__  
* __void enif_thread_exit(void *resp)__  
* __int enif_thread_join(ErlNifTid, void **respp)__  
* __ErlNifThreadOpts *enif_thread_opts_create(char *name)__  
* __void enif_thread_opts_destroy(ErlNifThreadOpts *opts)__  
* __ErlNifTid enif_thread_self(void)__  
* __int enif_tsd_key_create(char *name, ErlNifTSDKey *key)__  
* __void enif_tsd_key_destroy(ErlNifTSDKey key)__  
* __void *enif_tsd_get(ErlNifTSDKey key)__  
* __void enif_tsd_set(ErlNifTSDKey key, void *data)__  
* __ErlNifPid *enif_self(ErlNifEnv* caller_env, ErlNifPid* pid)__  	

#接口-类型操作类   <a name="exports-type"></a> 
	
* __int enif_compare(ERL_NIF_TERM lhs, ERL_NIF_TERM rhs)__  
	返回一个大于，等于，小于0整形数字来表示lhs 大于，等于，小于rhs。分别相当于erlang的运算符 ==,/=,=<,<,>,>=,>  (但不包括 =:=  和 =/=)。  
* __ERL_NIF_TERM enif_make_copy(ErlNifEnv* dst_env, ERL_NIF_TERM src_term)__  	
* __int enif_get_atom(ErlNifEnv* env, ERL_NIF_TERM term, char* buf, unsigned size, ErlNifCharEncoding encode)__  

* __int enif_get_atom_length(ErlNifEnv* env, ERL_NIF_TERM term, unsigned* len,ErlNifCharEncoding encode)__  
* __int enif_get_double(ErlNifEnv* env, ERL_NIF_TERM term, double* dp)__  
* __int enif_get_int(ErlNifEnv* env, ERL_NIF_TERM term, int* ip)__  
* __int enif_get_int64(ErlNifEnv* env, ERL_NIF_TERM term, ErlNifSInt64* ip)__  
* __int enif_get_local_pid(ErlNifEnv* env, ERL_NIF_TERM term, ErlNifPid* pid)__  
* __int enif_get_list_cell(ErlNifEnv* env, ERL_NIF_TERM list, ERL_NIF_TERM* head,ERL_NIF_TERM* tail)__  
* __int enif_get_list_length(ErlNifEnv* env, ERL_NIF_TERM term, unsigned* len)__  
* __int enif_get_long(ErlNifEnv* env, ERL_NIF_TERM term, long int* ip)__  
* __int enif_get_resource(ErlNifEnv* env, ERL_NIF_TERM term, ErlNifResourceType* type, void** objp)__  
* __int enif_get_string(ErlNifEnv* env, ERL_NIF_TERM list, char* buf, unsigned size, ErlNifCharEncoding encode)__  
* __int enif_get_tuple(ErlNifEnv* env, ERL_NIF_TERM term, int* arity, const ERL_NIF_TERM** array)__  
* __int enif_get_uint(ErlNifEnv* env, ERL_NIF_TERM term, unsigned int* ip)__  
* __int enif_get_uint64(ErlNifEnv* env, ERL_NIF_TERM term, ErlNifUInt64* ip)__  
* __int enif_get_ulong(ErlNifEnv* env, ERL_NIF_TERM term, unsigned long* ip)__  

* __void *enif_priv_data(ErlNifEnv* env)__  

* __类型判断，如果是该类型就返回true__  
	int enif_is_atom(ErlNifEnv* env, ERL_NIF_TERM term)   
	int enif_is_binary(ErlNifEnv* env, ERL_NIF_TERM term)  
	int enif_is_empty_list(ErlNifEnv* env, ERL_NIF_TERM term)  
	int enif_is_exception(ErlNifEnv* env, ERL_NIF_TERM term)  
	int enif_is_number(ErlNifEnv* env, ERL_NIF_TERM term)  
	int enif_is_fun(ErlNifEnv* env, ERL_NIF_TERM term)  
	int enif_is_identical(ERL_NIF_TERM lhs, ERL_NIF_TERM rhs)  
	int enif_is_pid(ErlNifEnv* env, ERL_NIF_TERM term)  
	int enif_is_port(ErlNifEnv* env, ERL_NIF_TERM term)  
	int enif_is_ref(ErlNifEnv* env, ERL_NIF_TERM term)  
	int enif_is_tuple(ErlNifEnv* env, ERL_NIF_TERM term)  
	int enif_is_list(ErlNifEnv* env, ERL_NIF_TERM term)  

	
* __unsigned enif_sizeof_resource(void* obj)__  

* __ERL_NIF_TERM enif_make_atom(ErlNifEnv* env, const char* name)__  
* __ERL_NIF_TERM enif_make_atom_len(ErlNifEnv* env, const char* name, size_t len)__  
* __ERL_NIF_TERM enif_make_badarg(ErlNifEnv* env)__  

* __ERL_NIF_TERM enif_make_double(ErlNifEnv* env, double d)__  
* __int enif_make_existing_atom(ErlNifEnv* env, const char* name, ERL_NIF_TERM* atom, ErlNifCharEncoding encode)__  
* __int enif_make_existing_atom_len(ErlNifEnv* env, const char* name, size_t len,ERL_NIF_TERM* atom, ErlNifCharEncoding encoding)__  
* __ERL_NIF_TERM enif_make_int(ErlNifEnv* env, int i)__  
* __ERL_NIF_TERM enif_make_int64(ErlNifEnv* env, ErlNifSInt64 i)__  
* __ERL_NIF_TERM enif_make_list(ErlNifEnv* env, unsigned cnt, ...)__  
* __ERL_NIF_TERM enif_make_list_cell(ErlNifEnv* env, ERL_NIF_TERM head,ERL_NIF_TERM tail)__  
* __ERL_NIF_TERM enif_make_list_from_array(ErlNifEnv* env, const ERL_NIF_TERM arr[], unsigned cnt)__  
* __int enif_make_reverse_list(ErlNifEnv* env, ERL_NIF_TERM term, ERL_NIF_TERM *list)__  
* __ERL_NIF_TERM enif_make_long(ErlNifEnv* env, long int i)__  

* __ERL_NIF_TERM enif_make_pid(ErlNifEnv* env, const ErlNifPid* pid)__  
* __ERL_NIF_TERM enif_make_ref(ErlNifEnv* env)__  

* __ERL_NIF_TERM enif_make_string(ErlNifEnv* env, const char* string,ErlNifCharEncoding encoding)__  
* __ERL_NIF_TERM enif_make_string_len(ErlNifEnv* env, const char* string, size_t len, ErlNifCharEncoding encoding)__  

* __ERL_NIF_TERM enif_make_tuple(ErlNifEnv* env, unsigned cnt, ...)__  
* __ERL_NIF_TERM enif_make_tuple_from_array(ErlNifEnv* env, const ERL_NIF_TERM arr[], unsigned cnt)__  
* __ERL_NIF_TERM enif_make_uint(ErlNifEnv* env, unsigned int i)__  
* __ERL_NIF_TERM enif_make_uint64(ErlNifEnv* env, ErlNifUInt64 i)__  
* __ERL_NIF_TERM enif_make_ulong(ErlNifEnv* env, unsigned long i)__  













	
---
by dp

  

  


