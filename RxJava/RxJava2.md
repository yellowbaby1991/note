* [RxJava2](#rxjava2)
	* [概述](#概述)
	* [观察者模式](#观察者模式)
		* [Consumer](#consumer)
		* [just](#just)
		* [fromIterable](#fromiterable)
	* [线程控制](#线程控制)
	* [转换操作符](#转换操作符)
		* [map](#map)
		* [flatMap](#flatmap)
		* [concatMap](#concatmap)
		* [flatMapIterable](#flatmapiterable)
		* [switchMap](#switchmap)
	* [过滤操作符](#过滤操作符)
		* [filter](#filter)
		* [take](#take)
		* [takeLast](#takelast)
		* [elementAt](#elementat)
		* [debounce](#debounce)
		* [distinct](#distinct)
		* [skip](#skip)
		* [skipLast](#skiplast)
	* [组合操作符](#组合操作符)
		* [Merge](#merge)
		* [concat](#concat)
		* [startWith](#startwith)
		* [zip](#zip)
	* [背压](#背压)
	* [Flowable](#flowable)
	* [最佳实践一：获得手机上安装的所有用户应用](#最佳实践一获得手机上安装的所有用户应用)
	* [最佳事件二：rxjava2+retrofit2实现网络请求](#最佳事件二rxjava2retrofit2实现网络请求)

# RxJava2
## 概述

> 相关参考

[API][1]
[GitHub][2]

> 配置

``` glide
compile 'io.reactivex.rxjava2:rxjava:2.0.1'
compile 'io.reactivex.rxjava2:rxandroid:2.0.1'
```

## 观察者模式

一个经典的观察者模式，上游发送事件，下游处理事件，然后上游监听下游

如图所示：

![enter description here][3]

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

### Consumer
如果下游只关心next事件，可以使用Consumer类取代Observer方法，代码如下：
``` java
Observable.create(new ObservableOnSubscribe<String>() {
	@Override
	public void subscribe(ObservableEmitter<String> emitter) throws Exception {
		emitter.onNext("1");
		emitter.onNext("2");
		emitter.onNext("3");
	}
}).subscribe(new Consumer<String>() {
	@Override
	public void accept(String s) throws Exception {
		Logger.d(s);
	}
});
```

### just
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

### fromIterable
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
## 转换操作符
### map
map的作用就是对上游发送的每一个事件应用一个函数，使得每一个事件都按指定的函数去变化，用事件图表示如下：

![enter description here][4]


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


### flatMap
FlatMap将上游的一个Observable变换成多个发送事件的Observables，然后将它们发送的事件合并后放入一个单独的Observable里，如图所示

![enter description here][5]


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

### concatMap
作用和flatMap几乎相同，区别在于concatMap会严格保证事件的顺序

``` java
Observable
		.just(1, 2, 3, 4)
		.concatMap(new Function<Integer, ObservableSource<String>>() {
			@Override
			public ObservableSource<String> apply(Integer integer) throws Exception {
				return Observable.fromArray("values:" + integer);
			}
		})
		.subscribe(new Consumer<String>() {
			@Override
			public void accept(String s) throws Exception {
				Logger.d(s);
			}
		});
```



### flatMapIterable
flatMapIterable()和flatMap()几乎是一样的，不同的是flatMapIterable()它转化的多个Observable是使用Iterable作为源数据的

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
                .flatMapIterable(new Function<Class, Iterable<Student>>() {
                    @Override
                    public Iterable<Student> apply(Class aClass) throws Exception {
                        return aClass.students;
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


### switchMap
switchMap和flatMap，很像，区别在于，每当上游发送一个新的事件，下游会取消之前处理的事件转而处理当前事件，使用场景：当上游发送多个网络请求的时候，只处理最近的一个

``` java
Observable
		.just("A", "B", "C", "D", "E")
		.switchMap(new Function<String, ObservableSource<String>>() {
			@Override
			public ObservableSource<String> apply(String s) throws Exception {
				return Observable.just(s);
			}
		})
		.subscribe(new Observer<String>() {
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
}
```


## 过滤操作符
### filter
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

### take
take(int)用一个整数n作为一个参数，从原始的序列中发射前n个元素

``` java
Observable
		.just("A", "B", "C", "D", "E")
		.take(2)
		.subscribe(new Consumer<String>() {
			@Override
			public void accept(String s) throws Exception {
				Log.d(TAG, s);
			}
		});
```
### takeLast
takeLast(int)同样用一个整数n作为参数，只不过它发射的是观测序列中后n个元素

``` java
Observable
		.just("A", "B", "C", "D", "E")
		.takeLast(2)
		.subscribe(new Consumer<String>() {
			@Override
			public void accept(String s) throws Exception {
				Log.d(TAG, s);
			}
		});
```

### elementAt
elementAt(int)用来获取元素Observable发射的事件序列中的第n项数据，并当做唯一的数据发射出去

``` java
Observable
		.just("A", "B", "C", "D", "E")
		.elementAt(2)
		.subscribe(new Consumer<String>() {
			@Override
			public void accept(String s) throws Exception {
				Log.d(TAG, s);
			}
		});
```

### debounce
debounce(long, TimeUnit)过滤掉了由Observable发射的速率过快的数据；如果在一个指定的时间间隔过去了仍旧没有发射一个，那么它将发射最后的那个，通常用来防止button被重复点击

``` java
Observable
		.just("A", "B", "C", "D", "E")
		.debounce(1000, TimeUnit.SECONDS)
		.subscribe(new Consumer<String>() {
			@Override
			public void accept(String s) throws Exception {
				Log.d(TAG, s);
			}
		});
```

上面的代码只有E会被下游接受处理


### distinct
distinct()的过滤规则是只允许还没有发射过的数据通过，所有重复的数据项都只会发射一次

``` java
Observable
		.just("A", "B", "C", "D", "E", "A", "B", "C", "D", "E")
		.distinct()
		.subscribe(new Consumer<String>() {
			@Override
			public void accept(String s) throws Exception {
				Log.d(TAG, s);
			}
		});
```

上面代码执行结果为：

``` java
05-26 11:26:18.268 1454-1454/? D/MainActivity: A
05-26 11:26:18.268 1454-1454/? D/MainActivity: B
05-26 11:26:18.268 1454-1454/? D/MainActivity: C
05-26 11:26:18.268 1454-1454/? D/MainActivity: D
05-26 11:26:18.268 1454-1454/? D/MainActivity: E
```




### skip
skip(int)让我们可以忽略Observable发射的前n项数据
``` java
Observable
		.just("A", "B", "C", "D", "E")
		.skip(2)
		.subscribe(new Consumer<String>() {
			@Override
			public void accept(String s) throws Exception {
				Log.d(TAG, s);
			}
		});
```
### skipLast
skipLast(int)忽略Observable发射的后n项数据

``` java
Observable
		.just("A", "B", "C", "D", "E")
		.skipLast(2)
		.subscribe(new Consumer<String>() {
			@Override
			public void accept(String s) throws Exception {
				Log.d(TAG, s);
			}
		});
```


## 组合操作符
### Merge
merge(Observable, Observable)将两个Observable发射的事件序列组合并成一个事件序列，就像是一个Observable发射的一样。你可以简单的将它理解为两个Obsrvable合并成了一个Observable，合并后的数据是无序的

``` java
Observable stringObservable = Observable
		.just("A", "B", "C", "D", "E");

Observable intObservable = Observable
		.just(1, 2, 3, 4, 5);

Observable
		.merge(stringObservable, intObservable)
		.subscribe(new Consumer() {
			@Override
			public void accept(Object o) throws Exception {
				Log.d(TAG, o.toString());
			}
		});
```

### concat
和Merge类似，但是是按顺序连接多个Observables

``` java
Observable stringObservable = Observable
		.just("A", "B", "C", "D", "E");

Observable intObservable = Observable
		.just(1, 2, 3, 4, 5);

Observable
		.concat(stringObservable, intObservable)
		.subscribe(new Consumer() {
			@Override
			public void accept(Object o) throws Exception {
				Log.d(TAG, o.toString());
			}
		});
```

### startWith
在数据序列的开头增加一项数据，startWith的内部也是调用了concat，Observable.concat(a,b)等价于a.concatWith(b)

### zip
Zip通过一个函数将多个Observable发送的事件结合到一起，然后发送这些组合到一起的事件. 它按照严格的顺序应用这个函数。它只发射与发射数据项最少的那个Observable一样多的数据，如下图所示

![enter description here][6]

代码如下：

``` java
Observable stringObservable = Observable
		.just("A", "B", "C", "D", "E");

Observable intObservable = Observable
		.just(1, 2, 3, 4, 5, 6);

Observable
		.zip(stringObservable, intObservable, new BiFunction<String, Integer, String>() {
			@Override
			public String apply(String s, Integer i) throws Exception {
				return s + i;
			}
		})
		.subscribe(new Consumer() {
			@Override
			public void accept(Object o) throws Exception {
				Log.d(TAG, o.toString());
			}
		});
```

运行结果为：

``` java
D/MainActivity: A1
D/MainActivity: B2
D/MainActivity: C3
D/MainActivity: D4
D/MainActivity: E5
```

## 背压

先看一段代码：

``` java
Observable
		.create(new ObservableOnSubscribe<Integer>() {
			@Override
			public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
				for (int i = 0; ; i++) {  //无限循环发送事件
					emitter.onNext(i);
					Log.d(TAG, "emitter" + i);
				}
			}
		})
		.subscribeOn(Schedulers.io())
		.observeOn(AndroidSchedulers.mainThread())
		.subscribe(new Consumer<Integer>() {
			@Override
			public void accept(Integer integer) throws Exception {
				Thread.sleep(2000);
				Log.d(TAG, "" + integer);
			}
		});
```

上游会无限的发送事件，而下游每两秒处理一次事件，从Monitors可以看出内存占有飙升，当上游发送事件的速度远大于下游处理的速度的时候就是**背压**

注意这是上下游不在同一个线程的时候，此时的订阅为**异步订阅**，上游就会不断的发送事件，从而形成了背压

如果在同一线程，订阅为**同步订阅**，上游会等待下游处理完之后才继续发送，如下代码所示：

``` java
Observable
		.create(new ObservableOnSubscribe<Integer>() {
			@Override
			public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
				for (int i = 0; ; i++) {  //无限循环发送事件
					emitter.onNext(i);
					Log.d(TAG, "emitter" + i);
				}
			}
		})
		.subscribe(new Consumer<Integer>() {
			@Override
			public void accept(Integer integer) throws Exception {
				Thread.sleep(2000);
				Log.d(TAG, "" + integer);
			}
		});
```

为了处理背压我们引出了RxJava特有的Flowable


## Flowable
之前我们所的上游和下游分别是Observable和Observer, 这次不一样的是上游变成了Flowable, 下游变成了Subscriber, 但是水管之间的连接还是通过subscribe(), 我们来看看最基本的用法吧

``` java
Flowable<Integer> upstream = Flowable.create(new FlowableOnSubscribe<Integer>() {
	@Override
	public void subscribe(FlowableEmitter<Integer> emitter) throws Exception {
		Log.d(TAG, "emit 1");
		emitter.onNext(1);
		Log.d(TAG, "emit 2");
		emitter.onNext(2);
		Log.d(TAG, "emit 3");
		emitter.onNext(3);
		Log.d(TAG, "emit complete");
		emitter.onComplete();
	}
}, BackpressureStrategy.ERROR);

Subscriber<Integer> downStream = new Subscriber<Integer>() {
	@Override
	public void onSubscribe(Subscription s) {
		Log.d(TAG, "onSubscribe");
		s.request(Long.MAX_VALUE);  //告诉上游我可以处理一万个事件，尽管发
	}

	@Override
	public void onNext(Integer integer) {
		Log.d(TAG, "onNext: " + integer);
	}

	@Override
	public void onError(Throwable t) {
		Log.w(TAG, "onError: ", t);
	}

	@Override
	public void onComplete() {
		Log.d(TAG, "onComplete");
	}
};

upstream.subscribe(downStream);
```

BackpressureStrategy.ERROR这种方式会在出现上下游流速不均衡的时候直接抛出一个异常,这个异常就是著名的MissingBackpressureException.

> request方法

s.request(long num);的意思是下游可以处理num个事件，上游看着发，代表了下游处理事件的能力，如果不设置或者设置的值过小，也会抛出MissingBackpressureException


  [1]: http://reactivex.io/RxJava/2.x/javadoc/
  [2]: https://github.com/ReactiveX/RxJava
  [3]: http://upload-images.jianshu.io/upload_images/1008453-7133ff9a13dd9a59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
  [4]: http://upload-images.jianshu.io/upload_images/1008453-2a068dc6b726568a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
  [5]: http://upload-images.jianshu.io/upload_images/1008453-2ccce5cf25e8023a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
  [6]: http://upload-images.jianshu.io/upload_images/1008453-e11e9d75b1775e4e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
## 最佳实践一：获得手机上安装的所有用户应用

> 思路

 1. 上游获得ApplicationInfoList，全部往下游发
 2. 使用filter过滤掉系统应用得到用户应用
 3. 使用map将ApplicationInfo转为我们需要的AppInfo
 4. 使用subscribeOn指定上游线程为IO线程
 5. 使用observeOn指定下游线程为主线程
 6. 下游onNext将得到的AppInfo存入列表
 7. onComplete结束后刷新界面

``` java
public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    private ListView mLvAppinfos;
    private List<AppInfo> mAppInfos;
    private QuickAdapter<AppInfo> mAppInfosAdapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initView();
        loadAppInfos();
    }

    private void loadAppInfos() {
        final PackageManager pm = MainActivity.this.getPackageManager();
        Observable
                .fromIterable(getApplicationInfoList(pm))
                .filter(new Predicate<ApplicationInfo>() {//过滤出非系统应用
                    @Override
                    public boolean test(ApplicationInfo applicationInfo) throws Exception {
                        return (applicationInfo.flags & ApplicationInfo.FLAG_SYSTEM) <= 0;
                    }
                })
                .map(new Function<ApplicationInfo, AppInfo>() {//将ApplicationInfo转为我们需要的AppInfo
                    @Override
                    public AppInfo apply(ApplicationInfo applicationInfo) throws Exception {
                        AppInfo appInfo = new AppInfo();
                        appInfo.setAppIcon(applicationInfo.loadIcon(pm));
                        appInfo.setAppName(applicationInfo.loadLabel(pm).toString());
                        return appInfo;
                    }
                })
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer<AppInfo>() {
                    @Override
                    public void onSubscribe(Disposable d) {

                    }

                    @Override
                    public void onNext(AppInfo appInfo) {
                        mAppInfos.add(appInfo);
                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onComplete() {
                        mAppInfosAdapter.replaceAll(mAppInfos);
                        mAppInfosAdapter.notifyDataSetChanged();
                    }
                });
    }

    private void initView() {
        mAppInfos = new ArrayList<>();
        mAppInfosAdapter = new QuickAdapter<AppInfo>(this, R.layout.app_list_item, mAppInfos) {
            @Override
            protected void convert(BaseAdapterHelper helper, AppInfo item) {
                helper.setText(R.id.textView, item.getAppName());
                helper.setImageDrawable(R.id.imageView, item.getAppIcon());
            }
        };
        mLvAppinfos = (ListView) findViewById(R.id.lv_appinfos);
        mLvAppinfos.setAdapter(mAppInfosAdapter);
    }

    private List<ApplicationInfo> getApplicationInfoList(final PackageManager pm) {
        return pm.getInstalledApplications(PackageManager.GET_UNINSTALLED_PACKAGES);
    }
}
```

## 最佳事件二：rxjava2+retrofit2实现网络请求

> 依赖

``` java
    compile 'io.reactivex.rxjava2:rxandroid:2.0.1'
    compile 'com.squareup.retrofit2:retrofit:2.2.0'
    compile 'com.squareup.retrofit2:adapter-rxjava2:2.2.0'
    compile 'com.squareup.retrofit2:converter-gson:2.2.0'
    compile 'com.orhanobut:logger:1.15'
```

> MainActivity.java

``` java
public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    private WashService mWashService;
    private Observable<WashInfo> mObservable;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mWashService = RetrofitUtil.getWashService();

        mWashService.washCall()
                .compose(RxSchedulers.io_main())
                .subscribe(new Consumer<WashInfo>() {
                    @Override
                    public void accept(WashInfo washInfo) throws Exception {
                        Logger.d(washInfo);
                    }
                });

    }

}
```

> RxSchedulers.java

``` stylus
enter code here
```


> WashService.java

``` java
public interface WashService {
    @GET("d9f6840be6bb07cf/wash_test?clive=wash")
    Observable<WashInfo> washCall();
}
```

> WashInfo.java

``` java
public class WashInfo {

    private List<WashInfoBean> washInfo;

    public List<WashInfoBean> getWashInfo() {
        return washInfo;
    }

    public void setWashInfo(List<WashInfoBean> washInfo) {
        this.washInfo = washInfo;
    }

    public static class WashInfoBean {
        /**
         * WashHead : http://file.bmob.cn/M02/9A/D2/oYYBAFbD7M2Aa_YKAABfpRaTgOg207.png
         * WashName : 帽子
         * Amount : ￥5
         */

        private String WashHead;
        private String WashName;
        private String Amount;

        public String getWashHead() {
            return WashHead;
        }

        public void setWashHead(String WashHead) {
            this.WashHead = WashHead;
        }

        public String getWashName() {
            return WashName;
        }

        public void setWashName(String WashName) {
            this.WashName = WashName;
        }

        public String getAmount() {
            return Amount;
        }

        public void setAmount(String Amount) {
            this.Amount = Amount;
        }
    }
}
```

> RetrofitUtil.java

``` java
public class RetrofitUtil {

    private Retrofit mRetrofit;
    private static RetrofitUtil mInstance;

    private RetrofitUtil() {
        OkHttpClient.Builder builder = new OkHttpClient.Builder();
        mRetrofit = new Retrofit.Builder()
                .client(builder.build())
                .baseUrl("http://cloud.bmob.cn/")
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .build();
    }

    public static RetrofitUtil getInstance() {
        if (mInstance == null) {
            synchronized (RetrofitUtil.class) {
                mInstance = new RetrofitUtil();
            }
        }
        return mInstance;
    }

    public static WashService getWashService() {
        return getInstance().mRetrofit.create(WashService.class);
    }
}
```
