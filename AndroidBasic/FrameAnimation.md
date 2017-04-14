### 概述
　逐帧动画是一种常见的动画形式（Frame By Frame），其原理理是在“连续的关键帧”中分解动画动作，也就是在时间轴的每帧上逐帧绘制不不同的内容，使其连续播放而成动画
　因为逐帧动画的帧序列列内容不一样，不但给制作增加了负担而且最终输出的文件量量也很大
　但它的优势也很明显：逐帧动画具有非常大的灵活性，几乎可以表现任何想表现的内容，而它类似与电影的播放模式，很适合于表演细腻的动画
 
### 用法
#### 使用xml

 1. drawable中定义my_animlist.xml

``` xml
<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:oneshot="true" >
    <!-- android:oneshot 动画只演示⼀一次-->
    <item android:drawable="@drawable/p1" android:duration="200"/>
    <item android:drawable="@drawable/p2" android:duration="200"/>
    <item android:drawable="@drawable/p3" android:duration="200"/>
    <item android:drawable="@drawable/p4" android:duration="200"/>
    <item android:drawable="@drawable/p5" android:duration="200"/>
    <item android:drawable="@drawable/p6" android:duration="200"/>
    <item android:drawable="@drawable/p7" android:duration="200"/>
    <item android:drawable="@drawable/p8" android:duration="200"/>
</animation-list>
```


 2. 1