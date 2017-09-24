## 导语  

此篇文章，主要是总结我在搜狗立懂百科无线视频播放器开发项目中遇到的一些坑。video标签虽然早就已经被各大浏览器支持，但在实践中浏览器的表现却并不能与标准一致，一些奇怪的表现真的令人抓狂，需要开发者不断摸索，做到更好的用户体验。本文从以下几点介绍开发中遇到一些坑：  
 
* 浏览器劫持问题  
* 自动播放与连续播放  
* 全屏与伪全屏
* html5video标准踩坑指南

## 浏览器劫持问题
最初进行无线视频开发都会发现一个问题![](https://pic4.zhimg.com/50/10c3f495f539147dd238317a89d4ab4b_hd.jpg)  
比如在[https://www.zhihu.com/question/27472010]()提到的这个问题，在uc，qq以及一些手机自带浏览器中，无论怎么调节z-index，页面上的dom元素都不会高过video的层级。这并不是浏览器关于的层级的兼容bug，而是浏览器有意为之，此时显示的视频，是浏览器用native层级进行显示的视频，有些浏览器甚至可以在暂停状态下看到广告。。。  
显然页面上所有的交互的操作如果和视频显示范围有重合，一定会在视频层级之下。  
### **解决办法：**  
### 上策：获得浏览器厂商的白名单    
也许你会发现，为什么专业的视频网站和一些大厂他们的h5播放体验完全没有这种问题，视频上方甚至可以做各种交互，这样的网站显然拥有浏览器的白名单，浏览器不会劫持视频，完全按照html5标准解析页面。（话说浏览器不就应该按标准解析网页么。。。）  
建议尽量获取到项目中用户量最大的浏览器的白名单，本文后续提到的一些问题也有因为浏览器劫持引起的，这会大大降低开发量，体验也会更好。  
### 中策：进入二级页播放，避免交互  
当然获取全部浏览器的白名单非常有难度，如果不是一定要在视频浮层上进行交互，比如点击按钮下拉二级菜单遮挡video，可以考虑进入新的没有浮层交互的二级进行播放（单页面，iframe），在设计阶段就避免一系列奇怪的bug，比如优酷进入二级页播放。
![](https://raw.githubusercontent.com/zhengx666/summarize/master/img/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-09-24%2022.08.30.png) 
### 下策：兼容方案
如果既无法获取到浏览器的白名单，设计又不能改，只能想办法兼容，尽量提高用户体验。基本方法是：  
1.用户点击视频外交互操作时，暂停视频。   
2.将视频移到用户可见区域之外，例如css transform属性。 
`注: transform在低版本ios中可能有兼容问题`  
3.交互操作结束将视频还原。   
这样的兼容方案，有一个视频突然消失的操作，而且不一定今后在哪个版本浏览器会有兼容问题（国内浏览器video真的不按标准来)，不建议优先使用。
  
浏览器劫持问题在很多国内浏览器都存在，最好在开发初期先用一个video标签进行兼容实验，能否满足设计需求。
## 自动播放与连续播放
一些设计需求可能要求用户进入页面或者调转页面（非单页面）的时候视频自动开始播放，这在无线端几乎所有浏览器都不允许，谁也不想点开一个链接就开始疯狂的走流量。但用户只要进行任何交互操作，就可以监听这一事件，在事件内调用video的play事件进行播放。 
![](https://raw.githubusercontent.com/zhengx666/summarize/master/img/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-09-24%2023.23.44.png)  
  
```
<div class="v-thumb">
		<div class="v-pic-default">
			<img src="//static.youku.com/v1.0.166/index/img/mobile/video_defaultpic1.png">
		</div>
		<div class="v-pic-real" style="background-image:url('//r1.ykimg.com/050C000059C7BBEDADC0B00745086B09');"></div>
		<a class="v-link" href="//m.youku.com/video/id_XMzA0NDU0NzQ3Mg==.html?f=51074588&amp;ev=1" data-from="1-1" target="_self" title="[蒙面唱将猜猜猜]惊呆！飞行大咖曝范冰冰那些事"></a>
		<div class="v-tagrb"><span class="v-time">59:52</span></div>		<div class="v-overlay-large"></div>
				<div class="v-description v-large">
						<div class="v-title"><a href="//m.youku.com/video/id_XMzA0NDU0NzQ3Mg==.html?f=51074588&amp;ev=1" data-from="1-2" target="_self">[蒙面唱将猜猜猜]惊呆！飞行大咖曝范冰冰那些事</a></div>
						<div class="v-desc">
				&nbsp;<i class="ico-stat-play" title="播放"></i><span class="v-num">41.3万</span>&nbsp;<i title="评论" class="ico-stat-comment"></i><span class="v-num">81</span>			</div>
		</div>
</div>
```  
	 
大多数无线端页面，初始状态并不加载视频，也不采用video的poster属性做占位(更灵活)，而是直接使用img做为占位，监听用户点击事件后进入二级页播放，或者调用video的play事件进行播放。  
`ps：点击链接进入二级页播放需要用单页面实现，否则浏览器限制无法自动播放`  
 如果需要播放连续播放视频，最直接的方法是监听video的ended事件，然后调用下一个video的play方法，不幸的是无线端这样调用play方法同样无效，仍然需要用户的交互。但是在ended方法中改变video的src确是有效的，可以采用这种方法达到连续播放的效果。
## 全屏与伪全屏  
video中的全屏的实现有两种方式，一种是系统全屏，一种是伪全屏。  
系统全屏指的是调用系统提供的全屏接口，由浏览器来进行视频全屏控制，兼容写法如下：  
  
```    
if (video.requestFullscreen) {
	 video.requestFullscreen();
} else if (video.webkitEnterFullScreen) {
    video.webkitEnterFullScreen();
} else if (video.mozRequestFullScreen) {
    video.mozRequestFullScreen();
} else if (video.webkitRequestFullScreen) {
    video.webkitRequestFullScreen();
}  
```  
这里有一个坑，安卓下chrome浏览器进入全屏状态，默认不会提供控制条，全屏时z-index的值为2147483647，自定义的z-index必须大于2147483647的值。

伪全屏指的是不调用系统全屏接口，使视频容器撑满整个屏幕，全屏的是普通dom元素而不是video，此时手机端依然会有进入全屏状态的提示，用户感知不到是系统全屏还是伪全屏状态。  
推荐一个非常好的伪全屏兼容的库[https://github.com/sindresorhus/screenfull.js]()  
如果在系统全屏时遇到一些坑时，可以考虑使用伪全屏来做兼容，这时可以使用css3来实现横屏显示。需要注意的是对于手机来说此时播放状态仍然是竖屏，进行交互操作时需要小心。
## html5video标准避坑指南  
如今PC端浏览器对于标准的实现越来越规范，但无线端感觉却越来越混乱，一些浏览器厂商有自己的内核，比如qq浏览器x5，一些手机厂商不允许浏览器用自己的内核比如苹果。。。这样即使同一款浏览器在ios和安卓上的表现都可能有很大差异，尤其对于video来说。所以白名单真的是很重要，没有的话多写兼容代码吧。
### ios下无法监听canplay事件  
一些视频加载时需要一个等待动画，一般来说canplay事件比较合适做为终止等待动画的事件，但ios监听不到，ios可以使用play事件来做兼容。
### ios video不写controls，默认控制条仍在
控制条以及退出全屏时的播放按钮可以通过css隐藏，如：  
```  
video::-webkit-media-controls-enclosure {
        display: none !important;
}  
```
### 小窗播放
一部分浏览器video默认会进入全屏播放，如需小窗播放，没有白名单的同学可以尝试以下兼容写法，并不能兼容所有浏览器，比如uc浏览器就是无效的。。。  
```  
<video src="" webkit-playsinline="true" playsinline="true" x-webkit-airplay="allow" x5-playsinline=""></video>
```  
qq浏览器没有白名单的同学遇到一些问题可以去[http://bbs.mb.qq.com/forum-110-1.html]() x5的官方论坛看一下。  
### 浏览器劫持video显示位置问题  
如果视频被浏览器劫持进入全屏播放，退出全屏后回到小窗位置显示，此时视频在浏览器中显示的位置可能会错乱，比如识别不到margin。猜测浏览器是根据video的定位，在同样区域native播放，但这个区域的识别并不一定准确，尽量不要在这个位置做复杂的定位以及交互。
### qq浏览器后退时视频自动播放  
对于单页面程序，通过浏览器后退按钮退回到播放过视频的页面，可能会引起视频的自动播放，对于qq浏览器，可以通过监听popstate事件做兼容处理，如重新load视频。



