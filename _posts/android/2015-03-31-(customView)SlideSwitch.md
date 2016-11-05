---
layout: post
title: "Android自定义View-------IOS风格的滑动开关"
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

源码和测试例子已经放到github:[https://github.com/Leaking/SlideSwitch](https://github.com/Leaking/SlideSwitch)

项目开发中，经常会有一些关于用户个性化的设置，此时经常需要一个开关控件，周末将之前写的自定义开关控件优化了一下，效果图如下。先说说大概思路：按钮绘制了三个图层，最下面是覆盖整个View的灰色，第二个是覆盖整个View的自定义的颜色，它可以改变透明度。第三个是白色。当白色部分移动时，修改第二个图层的透明度，即可作出如图滑动过程中颜色渐变的效果。

<figure>
  <img src="{{ site.url }}/images/slideswitch.gif" alt="search screenshot" width="200" height="200">
  <figcaption>效果图</figcaption>
</figure>

大概记录一下重写一个组件的实现过程。

+ 定义属性
+ 在Java代码中的构造器获取属性的值
+ 重写onMeasure()
+ 重写onDraw()
+ 如果需要，重写onTouch监听
+ 在自己项目的布局文件中，定义一个命名空间，使用属性

# 1，定义属性

新建一个项目，命名为SlieSwitch，并新建属性文件SlideSwitch\res\values\attrs.xml

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>  
<resources>  

    <declare-styleable name="slideswitch">  
        <attr name="themeColor" format="color" />  
        <attr name="isOpen" format="boolean" />  
        <attr name="shape">  
            <enum name="rect" value="1" />  
            <enum name="circle" value="2" />  
        </attr>  
    </declare-styleable>  

</resources>  
{% endhighlight %}

其中定义了三个属性，一个是按钮的颜色，一个是按钮的打开状态，一个是枚举类型，它用来描述按钮的形状，关于这些的属性以及其它几个类型的属性的使用，可以容易百度谷歌到相关知识，这里暂不作记录。


# 2,，在Java代码中的构造器获取属性的值

在SlieSwitch中新建一个包com.leaking.slideswitch，然后再包里面新建一个文件文件SlieSwitch.java。自定义组件首先得继承View类，然后再构造器中获取自定义属性，如下艾玛片段：

{% highlight java %}
public class SlideSwitch extends View {  

    //,,,,,,,省略,,,,,,,,,,  

    public SlideSwitch(Context context, AttributeSet attrs, int defStyleAttr) {  
        super(context, attrs, defStyleAttr);  
        listener = null;  
        paint = new Paint();  
        paint.setAntiAlias(true);  
        TypedArray a = context.obtainStyledAttributes(attrs,  
                R.styleable.slideswitch);  
        color_theme = a.getColor(R.styleable.slideswitch_themeColor,  
                COLOR_THEME);  
        isOpen = a.getBoolean(R.styleable.slideswitch_isOpen, false);  
        shape = a.getInt(R.styleable.slideswitch_shape, SHAPE_RECT);  
        a.recycle();  
    }  

    public SlideSwitch(Context context, AttributeSet attrs) {  
        this(context, attrs, 0);  
    }  

    public SlideSwitch(Context context) {  
        this(context, null);  
    }  
         //,,,,,,,省略,,,,,,,,,,  
}  
{% endhighlight %}

一般View有三个构造器，我习惯让只有一个参数的构造方法调用有两个参数的构造方法，然后两个的调用三个的，在有三个参数的构造方法里使用TypeArray读取自定义属性，然后调用TypeArray的recycle()方法回收资源。

# 3，重写onMeasure()

上网稍微搜索一下，很快能只奥怎么重写onMeasure()方法，但是为什么需要重写onMeasure()方法这一点，倒是不好搜索到。这个问题我记录在另一篇文章。

我们在onMeasure()中此处主要做了两件事，第一是计算View的大小，也就是它的长和宽，第二是记录了白色色块的起始位置，以及绘制按钮的一些其他参数。需要注意的是，当你调用view的invalidate()方法进行重绘时，系统只会去调用onDraw()方法，但是如果有关于其他组件的大小、内容（比如TextView的setText()）发生修改，都会触发onMeasure()，所以在onMeasure()中计算的位置变量，要避免受到重新调用onMeasure()的影响。编码过程遇到的一个bug就是因为这个原因。

# 4，重写onDraw()

onDraw()部分比较容易，只需按照onMeasure()中计算出来的参数绘制三个图层即可。

# 5，如果需要，重写onTouch监听(这里需要)

该部分也是比较容易，就是分别按照down,move,up三个事件计算位置，修改第二个图层颜色的透明度，然后调用invaliate重绘View。在up事件中启动一个线程，让按钮的滑动自动进行到结束。

# 6，在自己项目的布局文件中，定义一个命名空间，使用属性

{% highlight xml %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:tools="http://schemas.android.com/tools"  
    xmlns:slideswitch="http://schemas.android.com/apk/res/com.example.testlibs"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent"  
    android:background="#ffffffff"  
    android:gravity="center_horizontal"  
    android:orientation="vertical"  
    android:padding="10dip"  
    tools:context="com.example.testlibs.MainActivity" >  

    <com.leaking.slideswitch.SlideSwitch  
        android:id="@+id/swit"  
        android:layout_width="150dip"  
        android:layout_height="60dip"  
        slideswitch:isOpen="true"  
        slideswitch:shape="rect"  
        slideswitch:themeColor="#ffee3a00" >  
    </com.leaking.slideswitch.SlideSwitch>  

</LinearLayout>  
{% endhighlight %}

需要注意的是，上述代码片段的第二行，它引入了一个命名空间，其格式如下

xmlns:随便起名字="http://schemas.android.com/apk/res/你的应用的包名"  

注意最后一部分，是你使用这个自定义VIew的那个项目的应用包名。

接下来，在Java代码中设置开关的监听以及状态的设置，如下代码

{% highlight java %}
public class MainActivity extends Activity implements SlideListener {  

    TextView txt;  
    SlideSwitch slide;  
    SlideSwitch slide2;  

    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        slide = (SlideSwitch) findViewById(R.id.swit);  
        slide2 = (SlideSwitch) findViewById(R.id.swit2);  

        slide.setState(false);  
        txt = (TextView) findViewById(R.id.txt);  
        slide.setSlideListener(this);  
    }  

    @Override  
    public void open() {  
        // TODO Auto-generated method stub  
        txt.setText(" is opend ");  
    }  

    @Override  
    public void close() {  
        // TODO Auto-generated method stub  
        txt.setText(" is close ");  
    }  
}  
{% endhighlight %}
