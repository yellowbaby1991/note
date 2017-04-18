### 概述
　MVP是专门为了优化Android开发的一种设计模式，核心思想是将Activity中的UI逻辑抽象成View接口，把业务逻辑抽象成Presenter接口

> 类图

![enter description here][1]

> MVP的优势

 1. 分离了视图逻辑和业务逻辑，降低了耦合
 2. Activity只处理生命周期的任务，代码变得更加简洁
 3. 视图逻辑和业务逻辑分别抽象到了View和Presenter的接口中去，提高代码的可阅读性
 4. Presenter被抽象成接口，可以有多种具体的实现，所以方便进行单元测试
 5. 把业务逻辑抽到Presenter中去，避免后台线程引用着Activity导致Activity的资源无法被系统回收从而引起内存泄露和OOM

  [1]: https://segmentfault.com/image?src=http://7xih5c.com1.z0.glb.clouddn.com/15-10-12/94032090.jpg&objectId=1190000003927200&token=62cb9888184d6fe02a4b3ae814ca17e8