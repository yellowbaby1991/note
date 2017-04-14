### 概述
　补间动画的机制是指定两个动画之间起点末点，中间的变化部分由系统来完成
 
### 分类
   |  名称   |     标签     |          子类          |     效果     |
   | :---: | :--------: | :------------------: | :--------: |
   | 平移动画  | tanslate | TranslationAnimation |   移动View   |
   | 缩放动画  |  scale   |    ScaleAnimation    | 防盗或者缩小View |
   | 旋转动画  |  rotate  |   RotateAnimation    |   旋转View   |
   | 透明度动画 |  scale   |    AlphaAnimation    | 改变View的透明度 |
   
   
### 常用方法

 1. setDuration(2000)-设置动画时间
 2. setFillAfter(true)-设置动画停留留在最后一帧
 3. setInterpolator(new AccelerateDecelerateInterpolator())-设置加速器器