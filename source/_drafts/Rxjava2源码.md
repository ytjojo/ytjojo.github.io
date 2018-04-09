Rxjava2源码解析
=====================

##Flowable流程
测试代码

```java

 @Test
    public void testFlowable() {
        Flowable.just(1).subscribe(new Subscriber<Integer>() {
            @Override
            public void onSubscribe(Subscription s) {
                s.request(1);
                System.out.println("onSubscribe");
            }

            @Override
            public void onNext(Integer integer) {
                System.out.println("");
            }

            @Override
            public void onError(Throwable t) {
                System.out.println("");
            }

            @Override
            public void onComplete() {
                System.out.println("");
            }
        });
    }
```

1. 创建 FlowableJust类并返回值，new FlowableJust<T>(item)

2. 当我们调用subscribe时候，进入Flowable.subscribe(Subscriber<? super T> s)方法

```java
 public final void subscribe(Subscriber<? super T> s) {
        if (s instanceof FlowableSubscriber) {
            subscribe((FlowableSubscriber<? super T>)s);
        } else {
            ObjectHelper.requireNonNull(s, "s is null");
            subscribe(new StrictSubscriber<T>(s));
        }
    }
```
    首先判断了是不是FlowableSubscriber的子类，FlowableSubscriber与Subscriber区别在于前者onSubscribe方法中Subscription参数
	有@NonNull注解，不能传null值
	然后创建了严格的Subscriber ，类名为StrictSubscriber，入参是我们testFlowable方法中创建Subscriber
3 进入 `subscribe(new StrictSubscriber<T>(s));`内部	
```java
 public final void subscribe(FlowableSubscriber<? super T> s) {
        ObjectHelper.requireNonNull(s, "s is null");
        try {
            Subscriber<? super T> z = RxJavaPlugins.onSubscribe(this, s);

            ObjectHelper.requireNonNull(z, "Plugin returned null Subscriber");

            subscribeActual(z);
        } catch (NullPointerException e) { // NOPMD
            throw e;
        } catch (Throwable e) {
            Exceptions.throwIfFatal(e);
            // can't call onError because no way to know if a Subscription has been set or not
            // can't call onSubscribe because the call might have set a Subscription already
            RxJavaPlugins.onError(e);

            NullPointerException npe = new NullPointerException("Actually not, but can't throw other exceptions due to RS");
            npe.initCause(e);
            throw npe;
        }
    }
```
 `Subscriber<? super T> z = RxJavaPlugins.onSubscribe(this, s);`这一行未做处理直接返回s并赋值给z
然后进入subscribeActual(z)方法，因为我们创建的是FlowableJust,它重写了这个方法
```java
    protected void subscribeActual(Subscriber<? super T> s) {
        s.onSubscribe(new ScalarSubscription<T>(s, value));
    }
```
调用上面创建的StrictSubscriber中的onSubscribe方法，入参是ScalarSubscription，可以翻译标量订阅
ScalarSubscription的构造方法传入StrictSubscriber和FlowableJust创建传入数值1.
进入StrictSubscribe.onSubscribe方法

```java
    @Override
    public void onSubscribe(Subscription s) {
        if (once.compareAndSet(false, true)) {

            actual.onSubscribe(this);

            SubscriptionHelper.deferredSetOnce(this.s, requested, s);
        } else {
            s.cancel();
            cancel();
            onError(new IllegalStateException("§2.12 violated: onSubscribe must be called at most once"));
        }
    }
```
once.compareAndSet(false, true)这个方法作用是判断之前是否调用过，保证只调用一次
下一步是 actual.onSubscribe(this);actual对象是我们testFlowable创建Subscriber对象
```java
      @Override
            public void onSubscribe(Subscription s) {
                s.request(1);
                System.out.println("onSubscribe");
            }
```
我们在这个方法里调用Subscription的request(1)方法表示只请求1个值，如果上游Flowable缓存或发出超出数量1将不会传递给发起请求的Subscriber
此时又回到了StrictSubscriber这个类中
```java
    @Override
    public void request(long n) {
        if (n <= 0) {
            cancel();
            onError(new IllegalArgumentException("§3.9 violated: positive request amount required but it was " + n));
        } else {
            SubscriptionHelper.deferredRequest(s, requested, n);
        }
    }
```
首先判断请求数量是不是大于0，小于等于0抛出异常
大于0调用`SubscriptionHelper.deferredRequest(s, requested, n);`方法

```java
    public static void deferredRequest(AtomicReference<Subscription> field, AtomicLong requested, long n) {
        Subscription s = field.get();
        if (s != null) {
            s.request(n);
        } else {
            if (SubscriptionHelper.validate(n)) {
                BackpressureHelper.add(requested, n);

                s = field.get();
                if (s != null) {
                    long r = requested.getAndSet(0L);
                    if (r != 0L) {
                        s.request(r);
                    }
                }
            }
        }
    }
```
首先判断`AtomicReference<Subscription> field`是否为空
如果为空继续判断n是不是大于0
然后进入`BackpressureHelper.add(requested, n);`方法
次方法作用是将请求的数量赋值给AtomicLong requested这个入参中
此时又回到StrictSubscribe.onSubscribe方法中的 这一行SubscriptionHelper.deferredSetOnce(this.s, requested, s);
this.s为 final AtomicReference<Subscription> s；
requested是我们调用onSubscribe()回调方法中调用 Subscription s.request(1)传入数字
s为ScalarSubscription实例

```java
    public static boolean deferredSetOnce(AtomicReference<Subscription> field, AtomicLong requested,
            Subscription s) {
        if (SubscriptionHelper.setOnce(field, s)) {
            long r = requested.getAndSet(0L);
            if (r != 0L) {
                s.request(r);
            }
            return true;
        }
        return false;
    }
```
SubscriptionHelper.setOnce(field, s）这个判断主要作用是没有值就直接赋值，否则调用引用Subscription对象的cancel方法
deferredSetOnce主要作用是根据条件调用Subscription.requet(r)方法，同时重置AtomicLong requested的值为0
Subscription为ScalarSubscription实例，
进入s.request(r);
```java
    @Override
    public void request(long n) {
        if (!SubscriptionHelper.validate(n)) {
            return;
        }
        if (compareAndSet(NO_REQUEST, REQUESTED)) {
            Subscriber<? super T> s = subscriber;

            s.onNext(value);
            if (get() != CANCELLED) {
                s.onComplete();
            }
        }

    }

```
重点是这两行
```java
Subscriber<? super T> s = subscriber;
s.onNext(value);
```
subscriber 是StrictSubscriber实例
回到StrictSubscribe.onNext(T t);方法中来

```java
  @Override
    public void onNext(T t) {
        HalfSerializer.onNext(actual, t, this, error);
    }
```
  HalfSerializer.onNext(actual, t, this, error);
方法入参actual是我们自己创建Subscriber<Integer> t是传递的值1，AtomicInteger 是StrictSubscriber，error为null的引用
```java
    public static <T> void onNext(Subscriber<? super T> subscriber, T value,
            AtomicInteger wip, AtomicThrowable error) {
        if (wip.get() == 0 && wip.compareAndSet(0, 1)) {
            subscriber.onNext(value);
            if (wip.decrementAndGet() != 0) {
                Throwable ex = error.terminate();
                if (ex != null) {
                    subscriber.onError(ex);
                } else {
                    subscriber.onComplete();
                }
            }
        }
    }
```
`if (wip.get() == 0 && wip.compareAndSet(0, 1)) {`这句先判断StrictSubscriber的值是不是为0，如果为0继续原子操作如果为0把值变为1
然后调用subscriber.onNext(value);传值给我们创建subscriber
最后调用终止方法onComplete()或者onError()

整个流程走完了








	




