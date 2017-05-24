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

Disposable的意思是一次性用品，我们可以把它理解成两根管道之间的一个机关, 当调用它的dispose()方法时, 它就会将两根管道切断, 从而导致下游收不到事件，但是上游仍然可以继续发送事件

> 举个例子

``` stylus
enter code here
```
