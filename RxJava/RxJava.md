### 简介
　[RxJava][1] 是一个RxJava到底是什么？让我们直接跳过官方那种晦涩的追求精确的定义，其实初学RxJava只要把握两点：**观察者模式**和**异步**,就基本可以熟练使用RxJava了

### RxJava 好在哪
　一个词，**简洁**。随着程序逻辑变得越来越复杂，它依然能够保持简洁。
 
 　假如有需求：一个视图需要显示多张图片，需要选出png格式的图片切换到主线程显示
  
> 常规代码：800个回调

``` java
new Thread() {
    @Override
    public void run() {
        super.run();
        for (File folder : folders) {
            File[] files = folder.listFiles();
            for (File file : files) {
                if (file.getName().endsWith(".png")) {
                    final Bitmap bitmap = getBitmapFromFile(file);
                    getActivity().runOnUiThread(new Runnable() {
                        @Override
                        public void run() {
                            imageCollectorView.addImage(bitmap);
                        }
                    });
                }
            }
        }
    }
}.start();
```

> RxJava

``` java
Observable.from(folders)
    .flatMap(new Func1<File, Observable<File>>() {
        @Override
        public Observable<File> call(File file) {
            return Observable.from(file.listFiles());
        }
    })
    .filter(new Func1<File, Boolean>() {
        @Override
        public Boolean call(File file) {
            return file.getName().endsWith(".png");
        }
    })
    .map(new Func1<File, Bitmap>() {
        @Override
        public Bitmap call(File file) {
            return getBitmapFromFile(file);
        }
    })
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Action1<Bitmap>() {
        @Override
        public void call(Bitmap bitmap) {
            imageCollectorView.addImage(bitmap);
        }
    });
```
　所以， RxJava 好在哪？就好在简洁，好在那把什么复杂逻辑都能穿成一条线的简洁

 


### 基本概念
> 观察者模式四大基本概念

 1. Observable：可观察者，即被观察者
 2. Observer ：观察者
 3. subscribe：订阅
 4. event：事件

　Observable和Observer通过subscribe()方法来实现订阅关系，从而Observable可以在需要的时候发出事件通知Observer

> 观察者模式图

![enter description here][2]


  [1]: https://github.com/ReactiveX/RxJava
  [2]: http://ww3.sinaimg.cn/mw1024/52eb2279jw1f2rx46dspqj20gn04qaad.jpg