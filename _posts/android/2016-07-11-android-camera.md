---
layout: post
title: "Android Camera"
excerpt: "Custom written post descriptions are the way to go... if you're not lazy."
categories: android
tags: [android]
comments: true
share: true
image:
  feature: so-simple-sample-image-1.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
comments: true
share: true
---


来T公司公司半年有余了，好久没写东西。

近来在做一个关于Android中Camera的需求，此处做点简要记录。提供给大家参考，避免以后猜到这些坑。


# Camera遇到的坑

Android中Camera主要分5.0以前还有5.0以后，从5.0开始Android支持Camera2，现在项目中这个模块只使用了Camera（你可以理解为Camera1）。

使用Camera基本的API我就不一一列举了，此处只讲经验积累的东西。下文将打开相机预览窗口称为`Preview`，拍照结果称为`Picture`。

*  首先大多相机打开的方向并不是正常的，需要我们进行处理，你可能最终需要旋转度数为`degree`（90或180或270）。

*  `Picture`返回的照片也可能不是摆正的，需要旋转上面提到的度数`degree`.

*  我们可以获得`Preview`照片进行一些处理，比如二维码扫描或者ROI（感兴趣区域）识别，而`Preview`也是需要旋转`degree`角度才摆正.

*  使用相机时，你可以获得`Preview`和`Picture`可以支持的大小，但是你最好不要随便选择一个，首先，`Preview`的宽高比例需要接近你的UI中给相机预览窗口的宽高比，否则会觉得相机中图片被拉高了或者压扁了，你只需要做到比例接近就够了。另外无论是`Preview`的大小和`Picture`的大小，都需要考虑内存的情况。另外拍出来的照片大小取决于`Picture`的分辨率。

*  `Preview`回调的图片byte[]数组，是`YUV`存储格式的图片信息，我也是第一次见到这个概念，它和RGB一样是用于存储图片的编码格式，它需要通过某种公式转换成RGB，它的优点在于存储空间小，主要原因大概是几个`Y` 可以共享同一个`UV`.关于`YUV`的详细原理这里不拓展。然后如果你在项目中只拿`Preview`的图片，那么获取图片时有YuvImage相关的API可以使用。如果你是需要将图片信息传到JNI层进行图片处理，那么可能就需要进行进一步转换，转成RGB格式。

*  聚焦，关于聚焦的问题，我们经常使用的方法就是设置一个`FOCUS_MODE`,然后调用`camera.autofocus`的方法并且获取聚焦结果的回调，但是需要注意一点的是，并非所有`FOCUS_MODE`都是可以支持`camera.autofocus`方法的，比如`FOCUS_MODE_CONTINUOUS_VIDEO`，所以设置几种`FOCUS_MODE`时最好看看相关注释。

{% highlight java %}

 		 /**
         * Continuous auto focus mode intended for video recording. The camera
         * continuously tries to focus. This is the best choice for video
         * recording because the focus changes smoothly . Applications still can
         * call {@link #takePicture(Camera.ShutterCallback,
         * Camera.PictureCallback, Camera.PictureCallback)} in this mode but the
         * subject may not be in focus. Auto focus starts when the parameter is
         * set.
         *
         * <p>Since API level 14, applications can call {@link
         * #autoFocus(AutoFocusCallback)} in this mode. The focus callback will
         * immediately return with a boolean that indicates whether the focus is
         * sharp or not. The focus position is locked after autoFocus call. If
         * applications want to resume the continuous focus, cancelAutoFocus
         * must be called. Restarting the preview will not resume the continuous
         * autofocus. To stop continuous focus, applications should change the
         * focus mode to other modes.
         *
         * @see #FOCUS_MODE_CONTINUOUS_PICTURE
         */
        public static final String FOCUS_MODE_CONTINUOUS_VIDEO = "continuous-video";

{% endhighlight %}

另外，如果不是设置自动持续聚焦，而是手动聚焦，我们可以通过检查设备位置变化来调用`camera.autofocus`。另外有一个小技巧，就是在调用拍照时，先调用`camera.autoFocus`然后在成功聚焦回调里拍照。当然，对于聚焦失败你应该设置最大的失败次数限制。

*  关于crash，`camera.autofocus`聚焦还有`camera.takePicture`这两个API都不能持续重复调用，否则会crash，执行完一次聚焦才能执行下一次聚焦，执行完一次拍照才能执行下一次拍照，所以你做好做一个保护，另外，根据官方文档解释，`camera.takePicture`返回拍照数据后，preview是自动被stop了的，所以你需要重新start一次，否则下次拍照会crash。

*  另外关于`setFocusArea`，这个模块的概念在于理解其特殊的坐标系，见下图

<figure>
  <img src="{{ site.url }}/images/camera_focus_area.png" alt="search screenshot">
  <figcaption></figcaption>
</figure>	


这幅图网上到处都有，其实是来自Android开发官网，官网如是解释：左上角永远是(-1000,-1000)，右下角永远是（1000,1000）,这个坐标系最开始看起来有点奇怪，但是仔细一看只是X坐标Y坐标都是偏移缩小了1000而已，另外将X坐标Y坐标的最大值缩放为2000的长度。

那么问题来了，按照上面所说，你的镜头旋转过了，那么你此处的坐标系是否也要旋转呢，答案是不需要，直接拿旋转后的视图计算即可。亲测有效。


*  几个有用camera链接

[QQ空间开发团队-Android相机开发那些坑](https://mp.weixin.qq.com/s?__biz=MzI1MTA1MzM2Nw==&mid=401454605&idx=1&sn=d5a16f6dc13e7581fec08a4e704cd5d0&scene=1&srcid=070755OBPvgLTborfTSdRBPK&key=77421cf58af4a653bd19893e9cedc5f214e24a9f43d1b6a648950d96ec071465a25b23ef315340e73e13e493193a91ce&ascene=0&uin=Mjk1NzEwNjYxMA%3D%3D&devicetype=iMac15%2C1+OSX+OSX+10.10.4+build(14E46)&version=11020201&pass_ticket=R3QSQ71v43i%2FmEv8zhmbY40zPeaKdZYL1p4zPkxxBDSA4%2BGBnJaKkuXxDe42CW2Q)

[CSDN某博主一系列不错的Camera文章](http://blog.csdn.net/yanzi1225627/article/details/33313707)




# 关于图片格式 YUV


YUV的存储格式比较特别，一般都是多个Y公用UV，常见如下

I420: YYYYYYYY UU VV    =>YUV420P

YV12: YYYYYYYY VV UU    =>YUV420P

NV12: YYYYYYYY UVUV     =>YUV420SP

NV21: YYYYYYYY VUVU     =>YUV420SP

 
[[http://www.cnblogs.com/azraelly/archive/2013/01/01/2841269.html]([http://www.cnblogs.com/azraelly/archive/2013/01/01/2841269.html)

[http://blog.csdn.net/beyond_cn/article/details/12998247](http://blog.csdn.net/beyond_cn/article/details/12998247)

