### 概述
　补间动画的机制是指定两个动画之间起点末点，中间的变化部分由系统来完成
 
### 分类
   |  名称   |     标签     |          子类          |     效果     |
   | :---: | :--------: | :------------------: | :--------: |
   | 平移动画  | tanslate | TranslationAnimation |   移动View   |
   | 缩放动画  |  scale   |    ScaleAnimation    | 防盗或者缩小View |
   | 旋转动画  |  rotate  |   RotateAnimation    |   旋转View   |
   | 透明度动画 |  scale   |    AlphaAnimation    | 改变View的透明度 |
   
   
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


 2. 