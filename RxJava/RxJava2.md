# RxJava2
## 基本用法
### 配置

``` glide
compile 'io.reactivex.rxjava2:rxjava:2.0.1'
compile 'io.reactivex.rxjava2:rxandroid:2.0.1'
```

### ObservableEmitter和Disposable

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

public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError, Action onComplete) {}
public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError, Action onComplete, Consumer<? super Disposable> onSubscribe) {}
//上面调用过的，最后一个不带参数的上面已经用过，代表下游不关心
public final void subscribe(Observer<? super T> observer) {}
```
