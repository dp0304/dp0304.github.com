---
layout: post
title: "unity3D的FingerGestures插件"
description: ""
category: unity3d
tags: []
---
{% include JB/setup %}

FingerGestures是一个unity3D插件，用来处理用户动作，手势。  译自[FingerGestures官方文档](http://fingergestures.fatalfrog.com/docs/manual:start) 

## 目录
---
* [FingerGestures包结构][]
* [FingerGestures例子列表][]
* [设置场景][]
* [教程：识别一个轻敲手势][]
* [教程：识别手势][]

[FingerGestures包结构]: #package_content
[FingerGestures例子列表]: #samples-list
[设置场景]: #setting_up
[教程：识别一个轻敲手势]: #tap_gesture
[教程：识别手势]: #detecting_gesture
-----


##fingerGestures包结构   <a name="package_ontent"></a>     
<table class="table  table-striped  table-condensed">
	<tbody><tr class="row0">
		<th class="col0"> 路径，相对Assets/Plugin/… </th><th class="col1"> 描述 </th>
	</tr>
	<tr class="row1">
		<td class="col0"> FingerGestures/ </td><td class="col1"> 插件的根目录 </td>
	</tr>
	<tr class="row2">
		<td class="col0"> FingerGestures/Prefabs </td><td class="col1"> 可以直接拖放到场景中的预设资源(prefabs)</td>
	</tr>
	<tr class="row3">
		<td class="col0"> FingerGestures/Scripts </td><td class="col1"> 核心脚本和组件</td>
	</tr>
	<tr class="row4">
		<td class="col0"> FingerGestures/Scripts/Gesture Recognizers </td><td class="col1"> 每个手势识别 
的脚本</td>
	</tr>
	<tr class="row5">
		<td class="col0"> FingerGestures/Scripts/Finger Event Detectors </td><td class="col1"> 每个触摸事件检测器的脚本 </td>
	</tr>
	<tr class="row6">
		<td class="col0"> FingerGestures/Scripts/Components </td><td class="col1"> 手势识别和触摸事件所需要添加的额外组件</td>
	</tr>
	<tr class="row7">
		<td class="col0"> FingerGestures/Toolbox </td><td class="col1"> FingerGestures 自带的工具箱脚本 </td>
	</tr>
	<tr class="row8">
		<td class="col0"> FingerGestures/Samples.unitypackage </td><td class="col1"> 所有例子的子包 </td>
	</tr>
	<tr class="row9">
		<td class="col0"> FingerGestures/PlayMaker Actions.unitypackage </td><td class="col1"> FingerGestures对PlayMaker扩展的插件 </td>
	</tr>
	<tr class="row10">
		<td class="col0"> Editor/FingerGestures </td><td class="col1"> FingerGestures对编辑器的扩展 </td>
	</tr>
</tbody></table>  
	
##FingerGestures例子列表   <a name="samples-list"></a>   

* __Finger Event(鼠标或手指事件)__   
	__FingerEventsPart1:__ 展示如何通过不同的检测器（ FingerEventDetectors ）去检测鼠标或者手指的上（down）、下（up），按下不移动（stationary，悬停（hover） 事件。  
	__FingerEventsPart2:__ 展示如何识别不同鼠标或者手指动作（FingerMotionDetector）。  

* __Gestures（手势）__  	
	__BasicGestures:__ 识别单击（react to tap），双击（double tap），拖动（drag），长按（long——press），滑动（swipe）等基础手势。  
	__PinchAndTwist:__ 两个或多个手指同时在触摸屏上挤压（pinch）或扭转（twist）时，触发手势的事件。（PS：通常都是用来缩放或旋转）  
	__PointCloudGestures:__ 示范如何识别一个点云（point cloud）手势。（PS：通常是指用用户画的图案作为识别）   

* __Toolbox（工具箱）__  
	__Camera（放入摄像机的脚本）：__  
	Toolbox-DragView:  展示使用`TBDragView `脚本，实现拖动视角。  
	Toolbox-Orbit:     展示使用`TBOrbit`脚本，实现围绕目标旋转视角。  
	Toolbox-Pan:       展示使用`TBPan`脚本，实现以自身为轴旋转视角。  
	Toolbox-PinchZoom: 展示使用`TBPinchZoom`脚本，实现变焦。  

	__Object-Based（放入普通场景对象的脚本）：__  
	Toolbox-Drag:          展示使用`TBDrag `脚本，实现简单的物体拖动  
	Toolbox-Hover:         展示使用`TBHoverChangeMaterial ` 和 `TBHoverChangeScale `脚本，实现当鼠标或者手指悬停在物体上时候的响应。（PS：类似鼠标放到图标上，图标发亮的效果）   
	Toolbox-PinchToScale   展示使用`TBPinchToScale `脚本，实现缩放物体  
	Toolbox-TwistToRotate: 展示使用`TBTwistToRotate `脚本，实现旋转物体    
 	

##设置场景   <a name="setting_up"></a> 
需要在场景中实例化一个FingerGesture组件才可使用。 FingerGesture在项目中的作用是管理用户输入和识别手势和鼠标或手指事件。   
有两种添加方式，一是直接把Plugins\FingerGestures\Prefabs下的`FingerGestures` prefab文件拖入场景中。二是可以创建一个空物件，然后把`FingerGestures`组件添加进去。

![1](/image/fingergestures/scene_setup_fingergestures.png)  

使用` Make Persistent `标志可以让使FingerGestures 单例在跨场景后一直有效，所以只要保证它在第一个场景设置就足够。  
   

##教程：识别一个轻敲手势   <a name="tap_gesture"></a>  
该章节会学习到如何识别一个简单的单击动作，然后到特殊物件的单击动作识别，最后到识别一个三个手指的双击动作。  
	
* __初始化__  
	第一步，如上章节设置;  
	第二步，创建一个GameObject 命名为`Gestures` ;  
	第三步，给`Gestures`添加一个`TapRecognizer`组件，并保持默认设置，你可以在项目面板搜索到它或者直接打开Component > FingerGestures > Gestures > Tap menu item。  

	![1](/image/fingergestures/tut_tap1.png)   

	TapRecognizer 是其中一种手势识别器，它用于监控用户输入而且当一个有效的单击动作被识别时候工作。  
	第四步，创建一个新的C# script 叫做 `TapTutorial`并添加到第二步创建的`Gestures`中。  


* __轻敲屏幕__  
	第一步，点击TapGestures组件上的`Copy Event To Clipboard`按钮，它会把TapGesture所需要的时间信号代码copy到黏贴板。  
	第二步，粘贴到`TapTutorial`脚本里，如下:  
	
        public class TapTutorial : MonoBehaviour
         {
          void OnTap( TapGesture gesture ) 
         { 
          /* your code here */ 
          }
        } 

  	`OnTap`函数匹配定义在TapRecognizer 组件内的信息名属性，当识别器要识别一个轻敲手势，它会使用unity3d的`SendMessage API`先向Gestures物件内所有的脚本广播`OnTap`信息，只要TapTutorial绑定在该物件上，它的`OnTap`函数就会被调用到。  
  	出于性能考虑，通常使用.net标准的事件模型代替unity3d的SendMessage API。  
  	第三步，修改`OnTop`函数：

        void OnTap( TapGesture gesture ) 
        {
             Debug.Log( "Tap gesture detected at " + gesture.Position + 
                    ". It was sent by " + gesture.Recognizer.name );
        }  
    `gesture`参数包含着手势事件数据，在上面的代码，我们主要输出了位置和`TapRecognizer`内工作的事件。你还可以在`gesture`参数内获得更多属性，例如通过`gesture.Fingers`获得鼠标或手指相关的手势列表，还有可以通过`gesture.Selection`获得当前是哪个场景被轻敲 。  
    第四步，可以测试，通过敲不同位置，可以看到debug信息输出。   


##教程：识别手势   <a name="detecting_gesture"></a>   
在FingerGesture里，用户的手势都由`GestureRecognizers`组件来处理，它是顺序处理被识别匹配的用户动作的。   
	
	
* __找到GestureRecognizers__   	 
	每种手势都有自己的脚本，存放脚本的路径在`Plugins\FingerGestures\Scripts\Gesture Recognizers` 。
	![1](/image/fingergestures/gesture_recognizers_folder.png)   
	你也可以从 `Component > FingerGestures > Gestures menu`里面找到。   
	![1](/image/fingergestures/fingergesture_menu_gestures.png)     

* __基本使用__		
	要识别一个特殊手势，你需要：  
	1、添加对应的`GestureRecognizer `组件到场景中的目标物件。    
	2、配置它的属性。   
	3、监听它的手势事件和对应响应。

	作为手势事件通知的一部分，`GestureRecognizer `传递一个包含相关信息（位置，手指触屏点列表，选择的场景物件，相关的GestureRecognizer等）的事件参数。  

	一个手势识别器有以下监听事件的方式：   
	1、使用标准的.net 委托事件模型，每一个手势识别器都暴露一个.net事件接口。     
	2、使用unity的`SendMessage()`函数 ： 
		手势事件将会被广播到游戏对象里所有的组件。   
		手势事件也可以指向当前相关的场景对象，这时候需要把手势识别器配置到 `Raycaster `组件中才能检测场景对象。   

	这取决于你的选择。.net的事件模型较为高效，unity的`SendMessage()`较为方便。


* __属性__    
	 由同一基类派生出来的各种手势识别器共用一个通用配置和一些函数。例如，我们可以看到`TapRecognizer `和`SwipeRecognizer `组件的配置放置在同一个对象里。  
	 ![1](/image/fingergestures/gesture_recognizer_properties_swipe_tap.png)     
	 设置：    
	 	你可以看到，两个组件共用了一部分配置:`fingers setup`,`reset mode`,`event notification settings`,`references to additional components`...     
	 	同样，每个手势识别器都有自己独特的配置，例如滑动识别器要设置距离阀值、速度、和偏差。而多点触控可以设置最大持续时间等。

	 事件信息广播：     
	 	此处使用`SendMessage() `函数去通知其他系统。你可以使用`Message Name`属性去指定响应的函数名。     
	 	通常，`Message Target`会设置你加入的手势识别器组件。但你也可以设置别的对象。   

	 组件：    
	 	你可以收到手动指定添加组件。例如：添加一个`ScreenRaycaster `组件让手势识别器获知场景内对象碰撞。并把消息发送到相应的监听器。它允许识别器转发消息到正在有关联的场景对象。    

	 	

 --未完2013 08 12


