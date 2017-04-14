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

### 使用xml配置属性动画

 1. 在res/animator下创建xml
 
``` xml
<?xml version="1.0" encoding="utf-8"?>
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1000"
    android:propertyName="x"
    android:valueFrom="0"
    android:valueTo="100"
    android:valueType="floatType" >
</objectAnimator>

```

 2. java代码中

``` java
Animator animator = AnimatorInflater.loadAnimator(this, R.animator.anim);
animator.setTarget(iv);
animator.start();
```

### 属性动画集合AnimatorSet