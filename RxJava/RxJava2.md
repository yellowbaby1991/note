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
ObservableEmitter可以发送出三种事件

 1. onNext(T value)
 2. onComplete()
 3. onError


