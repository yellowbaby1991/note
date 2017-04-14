### 概述
　补间动画的机制是指定两个动画之间起点末点，中间的变化部分由系统来完成
 
### 分类
   |  名称   |     标签     |          子类          |     效果     |
   | :---: | :--------: | :------------------: | :--------: |
   | 平移动画  | tanslate | TranslationAnimation |   移动View   |
   | 缩放动画  |  scale   |    ScaleAnimation    | 防盗或者缩小View |
   | 旋转动画  |  rotate  |   RotateAnimation    |   旋转View   |
   | 透明度动画 |  scale   |    AlphaAnimation    | 改变View的透明度 |
   | 动画集合 |  set   |    AnimationSet    | 动画集合 |   
   
   
### 常用API

 1. setDuration(2000) - 设置动画时间
 2. setFillAfter(true) - 设置动画停留留在最后一帧
 3. setInterpolator(new AccelerateDecelerateInterpolator()) - 设置加速器器
 4. mIv.startAnimation(animation) - 控件启动动画，注意只要是View都能启动动画

### 使用代码

 1. TranslateAnimation
 
``` java
//从(0,0)平移到(parentWith/2,0)
Animation animation = new TranslateAnimation(Animation.RELATIVE_TO_SELF, 0,
		Animation.RELATIVE_TO_PARENT, 0.5f,
		Animation.RELATIVE_TO_SELF, 0,
		Animation.RELATIVE_TO_SELF, 0);
animation.setDuration(3000);
animation.setFillAfter(true);
animation.setInterpolator(new DecelerateInterpolator());
iv.startAnimation(animation);
```

 2. RotateAnimation
 
``` java
//以自身为圆心，从180度旋转到360度
Animation animation = new RotateAnimation(180, 360,
		Animation.RELATIVE_TO_SELF,0.5f,
		Animation.RELATIVE_TO_SELF,0.5f);
animation.setDuration(3000);
animation.setFillAfter(true);
iv.startAnimation(animation);
```


 3. AlphaAnimation
 
``` java
//动画的透明度从1渐变成0,然后再渐变回去
Animation animation = new AlphaAnimation(1.0f,0);
animation.setDuration(3000);
animation.setFillAfter(true);
animation.setRepeatCount(1);//重复次数
animation.setRepeatMode(Animation.REVERSE);//重复效果反过来
iv.startAnimation(animation);
```


 4. ScaleAnimation

``` java
// x缩放到原来的一半，y放大到原来的两倍，以自己中心为轴心
Animation animation = new ScaleAnimation(
		1.0f,0.5f,
		1.0f,2.0f,
		Animation.RELATIVE_TO_SELF,0.5f,
		Animation.RELATIVE_TO_SELF,0.5f);
animation.setDuration(3000);
animation.setFillAfter(true);
iv.startAnimation(animation);
```

 5. AnimationSet
 
``` java
AnimationSet set = new AnimationSet(false);//设置为FALSE，让子动画的Interpolator各自生效

Animation animRight = new TranslateAnimation(
		Animation.RELATIVE_TO_SELF, 0, Animation.RELATIVE_TO_SELF, 3f,
		Animation.RELATIVE_TO_SELF, 0, Animation.RELATIVE_TO_SELF, 3f);
animRight.setInterpolator(new AccelerateInterpolator());
animRight.setDuration(2000);
set.addAnimation(animRight);

Animation rotateAnim = new RotateAnimation(0, 360,
		Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF,
		0.5f);
rotateAnim.setDuration(2000);
rotateAnim.setInterpolator(new DecelerateInterpolator());
set.addAnimation(rotateAnim);

iv.startAnimation(set);
```


### 使用xml
　语法：定义在res/anim中的xml文件
 ```xml
   <?xml version="1.0" encoding="utf-8"?>

   <set xmlns:android="http://schemas.android.com/apk/res/android"	
        //set可以表示单个动画也可以是多个动画的集合
   	android:interpolator="@[package:]anim/interpolator_resource"	//插值器，影响动画速度
   	android:shareInterpolator=["true" | "false"] 	//是否共享插值器>	
   	<alpha			//透明度动画
    	   android:fromAlpha="float"	//透明度起始值
    	   android:toAlpha="float"		//透明度结束值 />
   	<scale			//缩放动画
     	  android:fromXScale="float"	//x的缩放起始值
     	  android:toXScale="float"		//x的缩放结束值
     	  android:fromYScale="float"	//y的缩放起始值
     	  android:toYScale="float"		//y的缩放结束值
     	  android:pivotX="float"		//缩放轴点的x坐标
      	  android:pivotY="float" 		//缩放轴点的y坐标/>
   	<translate  	//平移动画
      	  android:fromXDelta="float" 	//x的起始值
         android:toXDelta="float"		//x的结束值
         android:fromYDelta="float"	//y的起始值
         android:toYDelta="float" 		//y的结束值/>
       <rotate			//旋转动画
         android:fromDegrees="float"	//旋转开始的角度
         android:toDegrees="float"		//旋转结束的角度
         android:pivotX="float"		//旋转轴点的x坐标
         android:pivotY="float" 		//旋转轴点的x坐标/>
     	  ...
   </set>
 ``` 
　调用方法：
 
``` java
Animation animation = AnimationUtils.loadAnimation(this,R.anim.anim);
iv.startAnimation(animation);
```

 1. TranslateAnimation  

``` xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 直接写数据代表移动确定的像素，单位为px-->
<!-- 如果设置百分比，那就是本身的宽高的倍数 -->
<!-- 如果设置百分比p，那就是父控件的宽高的倍数 -->
<translate xmlns:android="http://schemas.android.com/apk/res/android"
android:fromXDelta="0"
android:toXDelta="50%p"
android:fromYDelta="0"
android:toYDelta="0"
android:duration="2000"
android:fillAfter="true"
android:interpolator="@android:anim/accelerate_decelerate_interpolator" />
```

 2. AlphaAnimation 
 
``` xml
<?xml version="1.0" encoding="utf-8"?>
<alpha xmlns:android="http://schemas.android.com/apk/res/android"
android:fromAlpha="0.5"
android:toAlpha="1.0"
android:repeatCount="1"
android:repeatMode="reverse"
android:duration="1000"
android:fillAfter="true" />
```
       
 3. RotateAnimation

``` xml
<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
android:fromDegrees="0"
android:toDegrees="270"
android:pivotX="50%"
android:pivotY="50%"
android:duration="3000"
android:fillAfter="true"
android:interpolator="@android:anim/accelerate_interpolator" />
```
                   
 4. ScaleAnimation
 
``` xml
<?xml version="1.0" encoding="utf-8"?>
<scale xmlns:android="http://schemas.android.com/apk/res/android"
android:fromXScale="1.0"
android:toXScale="2.0"
android:fromYScale="1.0"
android:toYScale="2.0"
android:pivotX="50%"
android:pivotY="50%"
android:fillAfter="true"
android:duration="2000" />
```

 5. AnimationSet
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             