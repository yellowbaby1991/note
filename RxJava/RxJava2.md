

# RxJava2
## 配置

``` glide
compile 'io.reactivex.rxjava2:rxjava:2.0.1'
compile 'io.reactivex.rxjava2:rxandroid:2.0.1'
```

## 线程控制

RxJava默认上游和下游会处于同一线程，也就是上游发送的事件在上面线程，下游默认在什么线程处理，想切换线程就要使用subscribeOn和observeOn方法，代码如下

``` java

        Observable<Integer> observable = Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                Log.d(TAG, "emit 1");
                emitter.onNext(1);
                Log.d(TAG, "emit 2");
                emitter.onNext(2);
                Log.d(TAG, "emit 3");
                emitter.onNext(3);
                Log.d(TAG, "emit complete");
                emitter.onComplete();
                Log.d(TAG, "emit 4");
                emitter.onNext(4);
            }
        });

        Consumer<Integer> consumer = new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                Log.d(TAG, "onNext: " + integer);
            }
        };

        observable.subscribeOn(Schedulers.newThread())//指定上游的线程
                .observeOn(AndroidSchedulers.mainThread())//指定下游的线程
                .subscribe(consumer);
```

RxJava内置了以下几种线程选择

 1. Schedulers.io() 代表io操作的线程, 通常用于网络,读写文件等io密集型的操作
 2. Schedulers.computation() 代表CPU计算密集型的操作, 例如需要大量计算的操作
 3. Schedulers.newThread() 代表一个常规的新线程
 4. AndroidSchedulers.mainThread() 代表Android的主线程


## 观察者模式

一个经典的观察者模式，上游发送事件，下游处理事件，然后上游监听下游

如图所示：

![enter description here][1]

代码实现如下：

``` java

//上游发送事件
Observable observable = Observable.create(new ObservableOnSubscribe<String>() {
	@Override
	public void subscribe(ObservableEmitter<String> emitter) throws Exception {
		emitter.onNext("1");
		emitter.onNext("2");
		emitter.onNext("3");
	}
});

//下游处理事件
Observer<String> observer = new Observer<String>() {
	@Override
	public void onSubscribe(Disposable d) {
		Log.d(TAG, "onSubscribe");
	}

	@Override
	public void onNext(String value) {
		Log.d(TAG, value);
	}

	@Override
	public void onError(Throwable e) {
		Log.d(TAG, "onError");
	}

	@Override
	public void onComplete() {
		Log.d(TAG, "onComplete");
	}
};

//上游订阅下游
observable.subscribe(observer);
```

写成链式结构如下：

``` java
Observable.create(new ObservableOnSubscribe<String>() {
	@Override
	public void subscribe(ObservableEmitter<String> emitter) throws Exception {
		emitter.onNext("1");
		emitter.onNext("2");
		emitter.onNext("3");
	}
}).subscribe(new Observer<String>() {
	@Override
	public void onSubscribe(Disposable d) {
		Log.d(TAG, "onSubscribe");
	}

	@Override
	public void onNext(String value) {
		Log.d(TAG, value);
	}

	@Override
	public void onError(Throwable e) {
		Log.d(TAG, "onError");
	}

	@Override
	public void onComplete() {
		Log.d(TAG, "onComplete");
	}
});
```


> ObservableEmitter

ObservableEmitter可以发送出三种事件onNext(T value)， onComplete()和 onError，但是发送需要满足一定规则

 1. 上游可以发送无限个onNext, 下游也可以接收无限个onNext
 2. 当上游发送了一个onComplete后, 上游onComplete之后的事件将会**继续**发送, 而下游收到onComplete事件之后将**不再继续**接收事件
 3. 当上游发送了一个onError后, 上游onError之后的事件将**继续**发送, 而下游收到onError事件之后将**不再继续**接收事件
 4. 上游可以不发送onComplete或onError
 5. 最为关键的是onComplete和onError必须唯一并且互斥

> Disposable

Disposable的意思是一次性用品，我们可以把它理解成两根管道之间的一个机关, 当调用它的dispose()方法时, 它就会将两根管道切断, 从而导致下游收不到事件，但是上游仍然可以继续发送事件，举例如下：


``` java
public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                Log.d(TAG, "emit 1");
                emitter.onNext(1);
                Log.d(TAG, "emit 2");
                emitter.onNext(2);
                Log.d(TAG, "emit 3");
                emitter.onNext(3);
                Log.d(TAG, "emit complete");
                emitter.onComplete();
                Log.d(TAG, "emit 4");
                emitter.onNext(4);
            }
        }).subscribe(new Observer<Integer>() {

            private Disposable mDisposable;
            private int i;

            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, "subscribe");
                mDisposable = d;
            }

            @Override
            public void onNext(Integer value) {
                Log.d(TAG, "onNext: " + value);
                i++;
                if (i == 2) {
                    Log.d(TAG, "dispose");
                    mDisposable.dispose();
                    Log.d(TAG, "isDisposed : " + mDisposable.isDisposed());
                }
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "error");
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "complete");
            }
        });
    }
}
```

运行结果如下，可见，在调用dispose后，下游已经不处理事件，而上游还在继续发送事件

``` java
D/MainActivity: subscribe
D/MainActivity: emit 1
D/MainActivity: onNext: 1
D/MainActivity: emit 2
D/MainActivity: onNext: 2
D/MainActivity: dispose
D/MainActivity: isDisposed : true
D/MainActivity: emit 3
D/MainActivity: emit complete
D/MainActivity: emit 4
```

> subscribe方法

订阅方法有几个重载方法

``` java
public final Disposable subscribe() {}
//只接受onNext事件
public final Disposable subscribe(Consumer<? super T> onNext) {}
//只接受onNext事件和onError事件
public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError) {} 
//分别接受onNext事件，onError事件和onComplete事件
public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError, Action onComplete) {}
//上面调用过的，最后一个不带参数的上面已经用过，代表下游不关心
public final void subscribe(Observer<? super T> observer) {}
```

如果只想处理onNext事件写法如下：

``` java
Observable.create(new ObservableOnSubscribe<Integer>() {
	@Override
	public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
		Log.d(TAG, "emit 1");
		emitter.onNext(1);
		Log.d(TAG, "emit 2");
		emitter.onNext(2);
		Log.d(TAG, "emit 3");
		emitter.onNext(3);
		Log.d(TAG, "emit complete");
		emitter.onComplete();
		Log.d(TAG, "emit 4");
		emitter.onNext(4);
	}
}).subscribe(new Consumer<Integer>() {
	@Override
	public void accept(Integer integer) throws Exception {
		Log.d(TAG, "onNext: " + integer);
	}
});
```

## just
just可以快速的创建几个上游事件，如下：

``` java
Observable
		.just("1", "2")
		.subscribe(new Consumer<String>() {
			@Override
			public void accept(String s) throws Exception {
				Logger.d(s);
			}
		});
```

上面代码等价于：

``` java
Observable
		.create(new ObservableOnSubscribe<String>() {
			@Override
			public void subscribe(ObservableEmitter<String> e) throws Exception {
				e.onNext("1");
				e.onNext("2");
			}
		})
		.subscribe(new Consumer<String>() {
			@Override
			public void accept(String s) throws Exception {
				Logger.d(s);
			}
		});
```

## fromIterable
fromIterable作用是将一个集合转换成一系列上游事件，代码如下：

``` java
ArrayList<String> names = new ArrayList<>();
names.add("1");
names.add("2");
names.add("3");
Observable
		.fromIterable(names)
		.subscribe(new Consumer<String>() {
			@Override
			public void accept(String s) throws Exception {
				Logger.d(s);
			}
		});
```

## filter
filter负责过滤上游的事件，早餐上游发送了包子，馒头，肠粉，春卷，饺子，炒粉，等食物，通过filter方法只选择了包子

代码如下：

``` java
Observable
		.just("包子", "馒头", "肠粉", "春卷", "饺子", "炒粉")
		.filter(new Predicate<String>() {
			@Override
			public boolean test(String s) throws Exception {
				return s.equals("包子");
			}
		})
		.subscribe(new Consumer<String>() {
			@Override
			public void accept(String s) throws Exception {
				Log.d(TAG, "accept: " + s);//这里只能吃上饺子
			}
		});
```

## map
map的作用就是对上游发送的每一个事件应用一个函数，使得每一个事件都按指定的函数去变化，用事件图表示如下：

![enter description here][2]


 举个例子，将上游的integer对象转换成下游的String对象

``` java
Observable
		.just(1, 2, 3, 4)
		.map(new Function<Integer, String>() {
			@Override
			public String apply(Integer integer) throws Exception {
				return "value:" + integer;
			}
		})
		.subscribe(new Consumer<String>() {
			@Override
			public void accept(String s) throws Exception {
				Logger.d(s);
			}
		});
```


## flatMap
FlatMap将上游的一个Observable变换成多个发送事件的Observables，然后将它们发送的事件合并后放入一个单独的Observable里，如图所示

![enter description here][3]


 中间flatMap的作用是将圆形的事件转换为一个发送矩形事件和三角形事件的新的上游Observable
 
 举例，得到所有的班级然后输出所有的学生姓名，代码如下：

``` java
public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ArrayList<Class> classes = new ArrayList<>();
        classes.add(new Class("一班"));
        classes.add(new Class("二班"));
        classes.add(new Class("三班"));

        Observable.fromIterable(classes)
                .flatMap(new Function<Class, ObservableSource<Student>>() {//输入的是班级，输出的是学生
                    @Override
                    public ObservableSource<Student> apply(Class aClass) throws Exception {
                        return Observable.fromIterable(aClass.getStudents());
                    }
                })
                .subscribe(new Consumer<Student>() {
                    @Override
                    public void accept(Student student) throws Exception {
                        Log.d(TAG, student.name);
                    }
                });

    }


    class Class {
        ArrayList<Student> students;

        Class(String name) {
            students = new ArrayList<Student>();
            students.add(new Student(name + "1"));
            students.add(new Student(name + "2"));
            students.add(new Student(name + "3"));
        }

        ArrayList<Student> getStudents() {
            return students;
        }
    }

    class Student {
        String name;

        public Student(String nameString) {
            name = nameString;
        }
    }
}
```
运行结果如下：

``` java
05-25 05:20:15.105 16892-16892/app.yellow.rxjava2 D/MainActivity: 一班1
05-25 05:20:15.105 16892-16892/app.yellow.rxjava2 D/MainActivity: 一班2
05-25 05:20:15.105 16892-16892/app.yellow.rxjava2 D/MainActivity: 一班3
05-25 05:20:15.105 16892-16892/app.yellow.rxjava2 D/MainActivity: 二班1
05-25 05:20:15.105 16892-16892/app.yellow.rxjava2 D/MainActivity: 二班2
05-25 05:20:15.105 16892-16892/app.yellow.rxjava2 D/MainActivity: 二班3
05-25 05:20:15.105 16892-16892/app.yellow.rxjava2 D/MainActivity: 三班1
05-25 05:20:15.105 16892-16892/app.yellow.rxjava2 D/MainActivity: 三班2
05-25 05:20:15.105 16892-16892/app.yellow.rxjava2 D/MainActivity: 三班3
```

上游每发送一个事件，flatMap都将创建一个新的水管，然后发送转换之后的新事件，下游接受到的就是这些新水管的事件，注意，**flatMap并不保证事件的顺序**，想保证顺序就使用concatMap

## concatMap
作用和flatMap几乎相同，区别在于concatMap会严格保证事件的顺序




## fromIterable


  [1]: http://upload-images.jianshu.io/upload_images/1008453-7133ff9a13dd9a59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
  [2]: http://upload-images.jianshu.io/upload_images/1008453-2a068dc6b726568a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
  [3]: http://upload-images.jianshu.io/upload_images/1008453-2ccce5cf25e8023a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240