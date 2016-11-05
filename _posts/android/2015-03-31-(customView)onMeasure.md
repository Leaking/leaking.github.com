---
layout: post
title: "Android自定义View-------为什么重写onMeasure()以及怎么重写"
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

这两天在看关于android自定义组件的知识，刚开始查阅了很多资料，依然觉得对onMeasure()方法的理解不够透彻，后来大致知道onMeasure怎么用了之后，又很好奇为什么需要去实现onMeasure()这个方法。

这篇文章主要记录两个问题，

+ 自定义组件时什么情况下需要实现onMeasure()
+ 怎么实现onMeasure()方法


# 为什么需要实现onMeasure()方法

我们看以下例子，我们自己定义一个View，很简单，只是继承View类。

{% highlight java %}
public class MyView extends View {  

    public MyView(Context context, AttributeSet attrs, int defStyleAttr) {  
        super(context, attrs, defStyleAttr);  
        // TODO Auto-generated constructor stub  
    }  

    public MyView(Context context, AttributeSet attrs) {  
        super(context, attrs);  
        // TODO Auto-generated constructor stub  
    }  

    public MyView(Context context) {  
        super(context);  
        // TODO Auto-generated constructor stub  
    }  

    @Override  
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {  
        // TODO Auto-generated method stub  
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);  
    }  
}  
{% endhighlight %}

然后在布局文件中定义如下布局:

{% highlight xml %}

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:tools="http://schemas.android.com/tools"  
    xmlns:MyView="http://schemas.android.com/apk/res/com.notes.notes_onmeasure"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent"  
    android:orientation="vertical"  
    android:padding="@dimen/activity_vertical_margin"  
    tools:context="com.notes.notes_onmeasure.EasyActivity" >  



    <com.notes.notes_onmeasure.MyView  
        android:layout_width="wrap_content"  
        android:layout_height="wrap_content"  
        android:background="#78787878" />  

    <TextView  
        android:layout_width="wrap_content"  
        android:layout_height="wrap_content"  
        android:text="能看到我吗？" />  

</LinearLayout>  

{% endhighlight %}

结果显示如

<figure>
  <img src="{{ site.url }}/images/onMeasure.jpg" alt="search screenshot" width="200" height="200">
  <figcaption>效果图</figcaption>
</figure>


当MyView的宽和高设置为match_parent时，MyView会填充整个界面是毋庸置疑的，当将其宽和高设置为某个确定值时，MyView的大小也会按这个确定的值显示。但是上面的例子中，我们把MyView的宽和高设置为wrap_content，它依然填充整个界面。问题就出在这里，所以我们需要重写onMeasure()方法控制其大小。其它继承于View的组件，如Button，TextView，它们都已经通过重写onMeasuer()方法。


#  怎么重写onMeasure()方法

android中，每个View的大小，不仅取决于自身，而且也受父视图的影响，可以参考我的另一篇文章

而onMeasure()是在父视图计算子视图大小时被调用的，其中的细节就不在此详细讲述了，我们暂时只需要这一点即可。

{% highlight java %}
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {  
   //,,,  
}  

{% endhighlight %}

onMeasure()方法有两个参数，这两个参数就是父视图发过来给子视图的限制条件。我们直接来重写上面的MyView中的onMeasure()方法，通过一个例子了解onMeasure()方法怎么实现。

{% highlight java %}
    @Override  
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {  
        // TODO Auto-generated method stub  
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);  
        int width = measureDimension(200, widthMeasureSpec);  
        int height = measureDimension(200, heightMeasureSpec);  
        setMeasuredDimension(width, height);  
    }  

    public int measureDimension(int defaultSize, int measureSpec){  
        int result;  

        int specMode = MeasureSpec.getMode(measureSpec);  
        int specSize = MeasureSpec.getSize(measureSpec);  

        if(specMode == MeasureSpec.EXACTLY){  
            result = specSize;  
        }else{  
            result = defaultSize;   //UNSPECIFIED  
            if(specMode == MeasureSpec.AT_MOST){  
                result = Math.min(result, specSize);  
            }  
        }  
        return result;  
    }  
{% endhighlight %}

重写onMeasure()方法时，必须最后调用setMeasureDimension()方法，否则会报错。然后我们最终setMeasureDimension()的width和height是怎么计算得到的呢？

首先我们解析widthMeasureSpec获得两个数据，一个是父视图希望子视图的大小是多少，一个是关于大小的模式，这两个数据分别通过以下两句代码获得

{% highlight java %}
int specMode = MeasureSpec.getMode(measureSpec);  
int specSize = MeasureSpec.getSize(measureSpec);
{% endhighlight %}

MeasureSpec.getMode()方法返回的结果有三种：

+ **UNSPECIFIED**：父结点对子结点的大小没有任何要求。
+ **EXACTLY**:  父结点要求其子节点的大小指定为某个确切的值。其子节点以及其他子孙结点都需要适应该大小。   
+ **AT MOST**：父结点要求其子节点的大小不能超过某个最大值，其子节点以及其他子孙结点的大小都需要小于这个值
