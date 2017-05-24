# RxJava2
## 基本用法
### 配置

``` glide
compile 'io.reactivex.rxjava2:rxjava:2.0.1'
compile 'io.reactivex.rxjava2:rxandroid:2.0.1'
```

### HelloWorld

> 一个基本的链式调用

``` java
Observable.create(new ObservableOnSubscribe<Integer>() {
	@Override
	public void subscribe(ObservableEmitter<Integer> e) throws Exception {
		e.onNext(1);
		e.onNext(2);
		e.onNext(3);
		e.onComplete();
	}
}).subscribe(new Observer<Integer>() {
	@Override
	public void onSubscribe(Disposable d) {
		Logger.d("subscribe");
	}

	@Override
	public void onNext(Integer value) {
		Logger.d(value);
	}

	@Override
	public void onError(Throwable e) {
		Logger.d(e);
	}

	@Override
	public void onComplete() {
		Logger.d("onComplete");
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


