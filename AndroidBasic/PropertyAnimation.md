### 概述
　由于逐帧动画和补间动画的一些局限性，如，只能操作View，不能真正改变View的属性，Android3.0之后，系统给我们提供了一种新的动画，属性动画，它可以真正改变目标对象的属性，而不仅仅局限在View
 
### ValueAnimator
　ValueAnimator是属性动画提供给开发者最重要的类，属性动画的运行机制是通过不断对值的操作来实现的，ValueAnimator这个类就负责计算值的变化，我们把初始值和结束值和运行时长告诉ValueAnimator，它会自动帮助我们平滑的从初始值到结束值之间的过渡
 
　比如使用ValueAnimator来平滑的改变一个view的alpha值

``` java
ValueAnimator animator = ValueAnimator.ofFloat(0, 1.0f);//平滑的从0变化到1
animator.setDuration(3000);//3s内
animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
	@Override
	public void onAnimationUpdate(ValueAnimator animation) {
		float values = (float) animation.getAnimatedValue();//平滑变化过程中拿到值
		iv.setAlpha(values);
	}
});
animator.start();
```
　其他常用的方法

 - ofInt - 不需要小数过渡，用int过渡
 - setStartDelay - 设置动画延时时间
 - setRepeatCount - 重复次数
 - setRepeatMode - 循环模式

### ObjectAnimator
　相比于ValueAnimator，ObjectAnimator可能才是我们最常接触到的类，因为ValueAnimator只不过是对值进行了一个平滑的动画过渡，而ObjectAnimator则就不同了，它是可以直接对任意对象的任意属性进行动画操作的，比如说View的alpha属性

``` java
ObjectAnimator animator = ObjectAnimator.ofFloat(iv, "alpha", 0, 1.0f);
animator.setDuration(2000);
animator.setRepeatCount(1);
animator.setRepeatMode(ValueAnimator.REVERSE);
animator.start();
```
　当然也可以添加监听来同步变化其他属性
 
``` java
ObjectAnimator animator = ObjectAnimator.ofFloat(iv, "x", 0, 100);
animator.setDuration(2000);
animator.setRepeatCount(1);
animator.setRepeatMode(ValueAnimator.REVERSE);
animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
	@Override
	public void onAnimationUpdate(ValueAnimator animation) {
		float value = (float) animation.getAnimatedValue();
		iv.setY(value);
	}
});
animator.start();
```

### 属性动画集合AnimatorSet
#### 使用xml方式

 1. 同样是在animator文件夹下定义xml

``` xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:ordering="together"
    android:repeatCount="1"
    android:repeatMode="reverse">
    <objectAnimator
        android:duration="1000"
        android:propertyName="x"
        android:valueFrom="0"
        android:valueTo="300"
        android:valueType="floatType" />
    <objectAnimator
        android:duration="2000"
        android:propertyName="y"
        android:valueFrom="0"
        android:valueTo="200"
        android:valueType="floatType" />
    <objectAnimator
        android:duration="3000"
        android:propertyName="rotationY"
        android:valueFrom="0"
        android:valueTo="180"
        android:valueType="floatType" />
</set>
```

 2. 加载动画集合

``` java
Animator animator = AnimatorInflater.loadAnimator(this, R.animator.anim);
animator.setTarget(iv);
animator.start();
```
#### 使用java代码

 1. 两个动画同时执行

``` java
 /**
* 两个动画同时执行
* @param view
*/
public void togetherRun(View view)
{
   ObjectAnimator anim1 = ObjectAnimator.ofFloat(mBlueBall, "scaleX",
           1.0f, 2f);
   ObjectAnimator anim2 = ObjectAnimator.ofFloat(mBlueBall, "scaleY",
           1.0f, 2f);
   AnimatorSet animSet = new AnimatorSet();
   animSet.setDuration(2000);
   animSet.setInterpolator(new LinearInterpolator());
   //两个动画同时执行
   animSet.playTogether(anim1, anim2);
   animSet.start();
}
```


 2. 按照定制顺序执行

``` java
/**
* 三个动画执行之后再执行一个动画
* @param view
*/
public void playWithAfter(View view)
{
   float cx = mBlueBall.getX();

   ObjectAnimator anim1 = ObjectAnimator.ofFloat(mBlueBall, "scaleX",
           1.0f, 2f);
   ObjectAnimator anim2 = ObjectAnimator.ofFloat(mBlueBall, "scaleY",
           1.0f, 2f);
   ObjectAnimator anim3 = ObjectAnimator.ofFloat(mBlueBall,
           "x",  cx ,  0f);
   ObjectAnimator anim4 = ObjectAnimator.ofFloat(mBlueBall,
           "x", cx);

   /**
    * anim1，anim2,anim3同时执行 
    * anim4接着执行 
    */
   AnimatorSet animSet = new AnimatorSet();
   animSet.play(anim1).with(anim2);
   animSet.play(anim2).with(anim3);
   animSet.play(anim4).after(anim3);
   animSet.setDuration(1000);
   animSet.start();
}
```
