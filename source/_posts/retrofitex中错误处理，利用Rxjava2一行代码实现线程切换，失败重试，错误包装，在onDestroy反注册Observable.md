
---
title: retrofitex中错误处理，利用Rxjava2一行代码实现线程切换，失败重试，错误包装，在onDestroy反注册Observable
date: 2018-04-4 14:26:10
categories: http
tags: [http,错误处理,失败重试,线程切换,compose操作符,ObservableTransformer]
---

### 开源库
https://github.com/ytjojo/androidPractice/tree/master/retrofitex
这是用与商业项目的Retrofit扩展库，最大亮点是支持自定义注解
支持带进度的上传下载，多线程断点下载，支持cookie管理，支持token自动刷新（刷新条件可以根据http statucode 默认401 我们公司是403），token失效http status code 为 403 retrofit是不支持自动刷新token或者cookie的。支持错误重试
支持post请求缓存，可以设置过期时间，支持无网络返回缓存。
对https做了封装
对https 在4.4及一下手机做了兼容。
4.4手机下载或者上传图片的时候会出现如下错误
>javax.net.ssl.SSLHandshakeException: javax.net.ssl.SSLProtocolException: SSL handshake aborted: 
支持在单元测试请求接口，这样验证新接口不用跑在手机上，当然用Postman更好
封装RxBus，支持tag，post的时候可以指定标签，只有注册该标签才能收到，还支持最后一个注册者才能收到事件，在多个注册者但是只允许最后一个响应的时候特别有用。


### 错误重试

错误重试是用retryWhen操作符
还要根据不同的错误类型是重试或者直接回调OnError
目前需要重试的异常有
retrofit2.HttpException
ConnectException
SocketTimeoutException
TimeoutException
UnknownHostException EOFException
需要重新登录的异常有
AuthException
TokenInvalidException
这两个异常都是继承自RuntimeExceptin

TokenInvalidException是在okHttp 拦截器中Http status code 401和403 409时候抛出的。
 异常是在类com.ytjojo.http.interceptor.HeaderInterceptor抛出的;
HeaderInterceptor发现Http status code 为401或者用户决定的code（我们项目中403或者409）之后，调用
HeaderCallable的call()方法尝试刷新token，在这里是同步的网络请求，一般是登录接口。如果刷新失败则会抛出
AuthException

```java

 public static Function<Observable<? extends Throwable>, Observable<?>> getRetryFunc1() {
        return new Function<Observable<? extends Throwable>, Observable<?>>() {
            private int retryDelaySecond = 5;
            private int retryCount = 0;
            private int maxRetryCount = 2;

            @Override
            public Observable<?> apply(Observable<? extends Throwable> observable) {
                return observable.flatMap(new Function<Throwable, ObservableSource<?>>() {
                    @Override
                    public ObservableSource<?> apply(Throwable throwable) {
                        return checkApiError(throwable);
                    }
                });
            }

            private Observable<?> checkApiError(Throwable throwable) {
                retryCount++;
                if (retryCount <= maxRetryCount) {
                    if (throwable instanceof ConnectException
                            || throwable instanceof SocketTimeoutException
                            || throwable instanceof TimeoutException || throwable instanceof UnknownHostException || throwable instanceof EOFException) {
                        return retry(throwable);
                    } else if (throwable instanceof AuthException) {
                        login();
                        return Observable.error(HttpExceptionHandle.handleException(throwable));
                    } else if (throwable instanceof TokenInvalidException) {
                        login();
                        return Observable.error(HttpExceptionHandle.handleException(throwable));
                    }
                    if (throwable instanceof HttpException) {
                        HttpException he = (HttpException) throwable;
                        if (he.code() != 401 && he.code() != 403 && he.code() != 409) {
                            return Observable.error(HttpExceptionHandle.handleException(throwable));
                        }else {
                            return retry(throwable);
                        }
                    }
                    return Observable.error(HttpExceptionHandle.handleException(throwable));
                }else{
                    if (throwable instanceof HttpException) {
                        HttpException he = (HttpException) throwable;
                        if (he.code() == 401 || he.code() == 403 || he.code() == 409) {
                            login();
                        }
                    }
                    return Observable.error(HttpExceptionHandle.handleException(throwable));
                }

            }

            /**
             *
             * @param throwable
             * @return
             */
            private Observable<?> retry(Throwable throwable) {
                if (retryCount <= maxRetryCount) {
                    return Observable.timer(retryDelaySecond,
                            TimeUnit.SECONDS).observeOn(Schedulers.io());
                } else {
                    return Observable.error(throwable);
                }
            }

            private void login() {
                if(sOnErrorLoginCallback != null){
                    sOnErrorLoginCallback.onErrorLogin(OnErrorLoginCallback.INTENT_FLAG);
                }
            }
        };
    }
```
使用

```java

observable.subscribeOn(Schedulers.io())
                        .observeOn(AndroidSchedulers.mainThread()).retryWhen(getRetryFunc1())
```

### 拦截器如何刷新token和抛出TokenInvalidException AuthException这两个异常的
```java

public class HeaderInterceptor implements Interceptor {
    private static final Charset UTF8 = Charset.forName("UTF-8");
    private final Map<String, String> mHeaders = new ConcurrentHashMap<>();
    private final HeaderCallable mTokenCallable;
    private final CountDownLatch mCountDownLatch = new CountDownLatch(1);
    private final String baseUrl;
    AtomicInteger mRefreshTokenFlag = new AtomicInteger(0);
    volatile boolean restoreCachedValue;

    public void putHeader(String key, String value) {
        if (key != null && value != null)
            mHeaders.put(key, value);
    }

    public void putHeaders(HashMap<String, String> headers) {
        if (headers != null) {
            mHeaders.putAll(headers);
        }

    }

    public void onUserLoginGetToken(String key, String token) {
        mRefreshTokenFlag.set(0);
        if (key != null && token != null) {
            mHeaders.put(key, token);
        }
    }

    public HeaderInterceptor(HeaderCallable tokenCallable, String baseUrl) {
        this.mTokenCallable = tokenCallable;
        this.baseUrl = baseUrl;
    }

    void clearAuth() {
        mHeaders.clear();
    }

    void processAuth() throws AuthException {
        try {
            String value = mTokenCallable.call();
            if (value != null) {
                mHeaders.put(mTokenCallable.key(), value);
                HashMap<String, String> extraHeaders = mTokenCallable.extraHeaders();
                if (extraHeaders != null && !extraHeaders.isEmpty()) {
                    mHeaders.putAll(extraHeaders);
                }
            } else {
                throw new AuthException("HeaderCallable.call()获得的headervalue为null");
            }
        } catch (Exception e) {
            throw new AuthException(e);
        }

    }

    @Override
    public Response intercept(Chain chain) throws IOException {
        final Request request = chain.request();

        if (request.url().toString().startsWith(baseUrl)) {
            Response response = chain.proceed(updateHeadaerIfNeeded(chain));
            if (isTokenExpired(response)) {
                //token 已经失效了
                if (mTokenCallable != null) {
                    //判断是否已经刷新了header
                    String oldTokenValue = request.header(mTokenCallable.key());
                    if (mRefreshTokenFlag.compareAndSet(2, 0)) {
                        if (oldTokenValue != null && oldTokenValue.equals(mHeaders.get(mTokenCallable.key()))) {
                            //已经更新了header,但是服务器仍然验证失败
                            throw new TokenInvalidException(response.code(),response.message());
                        }
                    }
                    requestTokenAync();
                }

                final Request.Builder requestBuilder = request.newBuilder();
                for (HashMap.Entry<String, String> entry : mHeaders.entrySet()) {
                    String key = entry.getKey();
                    String value = entry.getValue();
                    if (value != null && key != null) {
                        requestBuilder.header(key, value);
                    }
                }
                Request newSigned = requestBuilder.build();
                response =  chain.proceed(newSigned);
                if(isTokenExpired(response)){
                    throw new TokenInvalidException(response.code(),response.message());
                }
            } else {
                return response;
            }
        }
        return chain.proceed(request);
    }

    public static boolean equals(Object o1, Object o2) {
        if (o1 == null || o2 == null) {
            return false;
        }
        return o1.equals(o2);
    }

    private void requestTokenAync() throws IOException {
        if (mRefreshTokenFlag.compareAndSet(0, 1)) {
            clearAuth();
            try {
                processAuth();
                mRefreshTokenFlag.set(2);
            } catch (AuthException e) {
                mRefreshTokenFlag.set(3);
                throw e;
            } catch (Exception e) {
                mRefreshTokenFlag.set(3);
                throw new AuthException(e);
            } finally {
                mCountDownLatch.countDown();
            }
        } else if (mRefreshTokenFlag.compareAndSet(1, 1)) {
            try {
                mCountDownLatch.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }

    public Request updateHeadaerIfNeeded(Chain chain) throws IOException {
        Request request = chain.request();
        if (mHeaders.isEmpty()) {
            return request;
        }
        final Request.Builder requestBuilder = request.newBuilder();
        if (mTokenCallable != null && !restoreCachedValue) {
            synchronized (HeaderInterceptor.class) {
                if (!restoreCachedValue) {
                    if (mHeaders.get(mTokenCallable.key()) == null) {
                        String cachedValue = mTokenCallable.getCachedValue();
                        if (cachedValue != null) {
                            mHeaders.put(mTokenCallable.key(), cachedValue);
                        }
                    }
                    restoreCachedValue = true;
                }
            }


        }
        for (HashMap.Entry<String, String> entry : mHeaders.entrySet()) {
            String key = entry.getKey();
            String value = entry.getValue();
            if (value != null && key != null) {
                requestBuilder.header(key, value);
            }
        }
        return requestBuilder.build();
    }

    private boolean isTokenExpired(Response originalResponse) {
        if (originalResponse.code() == 401) {
            return true;
        }
        if (mTokenCallable != null) {
            return mTokenCallable.isExpired(originalResponse.code(), originalResponse);
        }
//        ResponseBody responseBody = originalResponse.body();
//        BufferedSource source = responseBody.source();
//        source.request(Long.MAX_VALUE); // Buffer the entire body.
//        Buffer buffer = source.buffer();
//        Charset charset = UTF8;
//        MediaType contentType = responseBody.contentType();
//        if (contentType != null) {
//            charset = contentType.charset(UTF8);
//        }
//        String bodyValue = buffer.clone().readString(charset);
//
//        JSONObject jsonObject = new JSONObject(bodyValue);
//        int code = jsonObject.optInt("code");
//        if(code == ServerResponse.EXCEPTION_TOKEN_NOTVALID){
//            return true;
//        }
        return false;

    }

}
```
配置刷新token的HeaderCallable
```java
  RetrofitClient.Builder builder = RetrofitClient.newBuilder().baseUrl(BuildConfig.ENVIRONMENT.getHostUrl())
                .cache(new File(getContext().getCacheDir(), "cachehttp"))
                .showLog(isDebug)
                .addConverterFactory(JacksonConverterFactory.create(JsonParse.getInstance().jsonMapper()))
                .mergeParameterHandler(NgariParamHandlar.create())
                .unsafeSSLSocketFactory()
                .headerCallable(new HeaderCallable() {
                    String token = null;

                    @Override
                    public String key() {
                        return LoginSdk.KEY_ACCESS_TOKEN;
                    }

                    @Override
                    public boolean isExpired(int code, Response response) {
                        return code == 403 || code == 409;
                    }

                    @Override
                    public String getCachedValue() {
                        return LoginSdk.getInstance().getToken();
                    }

                    @Override
                    public HashMap<String, String> extraHeaders() {
                        return null;
                    }

                    @Override
                    public String call() throws Exception {

                        LoginSdk.getInstance().refreshToken().subscribe(new Consumer<String>() {
                            @Override
                            public void accept(String s) {
                                token = s;
                            }
                        }, new Consumer<Throwable>() {
                            @Override
                            public void accept(Throwable e) throws Exception {
                            }
                        });
                        return token;
                    }
                });
        RetrofitClient.init(builder);
```
### 对错误信息的包装
后台接口返回错误信息有的包含“server error” "HeibernateExceptrion" "DaoException",我们toast提示是不希望看到这些异常信息的，需要替换为中文，如“跟服务器的交互发生点小问题，请重试..”，处理代码如下

```java


public class HttpExceptionHandle {

    private static final int UNAUTHORIZED = 401;
    private static final int FORBIDDEN = 403;
    private static final int NOT_FOUND = 404;
    private static final int REQUEST_TIMEOUT = 408;
    private static final int REQUEST_CONFLICT = 409;
    private static final int INTERNAL_SERVER_ERROR = 500;
    private static final int BAD_GATEWAY = 502;
    private static final int SERVICE_UNAVAILABLE = 503;
    private static final int GATEWAY_TIMEOUT = 504;

    public static ResponeThrowable handleException(Throwable e) {
        ResponeThrowable ex;
        Log.i("tag", "e.toString = " + e.toString());
        if (e instanceof HttpException) {
            HttpException httpException = (HttpException) e;

            switch (httpException.code()) {
                case UNAUTHORIZED:
                case FORBIDDEN:
                case REQUEST_CONFLICT:
                    ex = new ResponeThrowable("登录信息已经过期，请重新登陆...", e, ERROR.TOKEN_ERROR);
                    break;
                case NOT_FOUND:
                case REQUEST_TIMEOUT:
                case GATEWAY_TIMEOUT:
                case INTERNAL_SERVER_ERROR:
                case BAD_GATEWAY:
                case SERVICE_UNAVAILABLE:
                default:
                    ex = new ResponeThrowable("网络错误", e, ERROR.HTTP_ERROR);
                    //ex.code = httpException.code();
                    break;
            }
            return ex;
        } else if (e instanceof APIException) {
            ex = new ResponeThrowable(e.getMessage(), e, ((APIException) e).code);
            return ex;
        } else if (e instanceof ConnectException
                || e instanceof SocketTimeoutException
                || e instanceof TimeoutException || e instanceof UnknownHostException || e instanceof EOFException
                ) {
            ex = new ResponeThrowable("网络无法连接", e, ERROR.NETWORD_ERROR);
            return ex;
        } else if (e instanceof javax.net.ssl.SSLHandshakeException) {
            ex = new ResponeThrowable("证书验证失败", e, ERROR.SSL_ERROR);
            return ex;

        } else if (e instanceof TokenInvalidException) {
            ex = new ResponeThrowable("登录信息已经过期，请重新登陆...", e.getCause(), ERROR.TOKEN_ERROR);
            return ex;

        } else if (e instanceof AuthException) {
            ex = new ResponeThrowable("登录信息已经过期，请重新登陆...", e.getCause(), ERROR.AUTH_ERROR);
            return ex;
        } else {
            String className =e.getClass().getName().toLowerCase();
            if (className.contains("json")) {
                ex = new ResponeThrowable("解析错误", e, ERROR.PARSE_ERROR);
                return ex;
            }
            ex = new ResponeThrowable("跟服务器的交互发生点小问题，请重试..", e.getCause() == null ? e : e.getCause(), ERROR.UNKNOWN);
            return ex;
        }
    }


    /**
     * 约定异常
     */
    public static final class ERROR {
        /**
         * 未知错误
         */
        public static final int UNKNOWN = 1000;
        /**
         * 解析错误
         */
        public static final int PARSE_ERROR = 1001;
        /**
         * 网络错误
         */
        public static final int NETWORD_ERROR = 1002;
        /**
         * 协议出错
         */
        public static final int HTTP_ERROR = 1003;

        /**
         * 证书出错
         */
        public static final int SSL_ERROR = 1005;
        /**
         * 证书出错
         */
        public static final int TOKEN_ERROR = 1006;
        /**
         * 证书出错
         */
        public static final int AUTH_ERROR = 1007;
    }

    public static class ResponeThrowable extends RuntimeException {
        public int code;

        public ResponeThrowable(String message, Throwable throwable, int code) {
            super(message, throwable);
            this.code = code;
        }
    }

}
```
### 一行代码实现线程切换失败重试异常封装，onDestroy()反注册Obaservable
用compose操作符，传入RxHttpHelper中创建的ObservableTransformer

```java

        RetrofitClient.getDefault().create(TransferService.class)
                .updateTransferStatusToHaveClinic(mTransfer.getTransferId())
				//下面一行代码实现上述功能
                .compose(RxHttpHelper.applySchedulers(TransferDetailActivity.this
                        .<Boolean>bindUntilEvent(ActivityEvent.DESTROY)))
                .subscribe()
```
RxHttpHelper完整源码

```java

    public static interface OnErrorLoginCallback{
        public static final int INTENT_FLAG= Intent.FLAG_ACTIVITY_NEW_TASK
                | Intent.FLAG_ACTIVITY_CLEAR_TASK
                | Intent.FLAG_ACTIVITY_CLEAR_TOP;
        void onErrorLogin(int defaultIntentFlag);
    }
    private static  OnErrorLoginCallback sOnErrorLoginCallback;
    public static void setOnErrorLoginCallback(OnErrorLoginCallback c){
        sOnErrorLoginCallback = c;
    }

    public static Function<Observable<? extends Throwable>, Observable<?>> getRetryFunc1() {
        return new Function<Observable<? extends Throwable>, Observable<?>>() {
            private int retryDelaySecond = 5;
            private int retryCount = 0;
            private int maxRetryCount = 3;

            @Override
            public Observable<?> apply(Observable<? extends Throwable> observable) {
                return observable.flatMap(new Function<Throwable, ObservableSource<?>>() {
                    @Override
                    public ObservableSource<?> apply(Throwable throwable) {
                        return checkApiError(throwable);
                    }
                });
            }

            private Observable<?> checkApiError(Throwable throwable) {
                retryCount++;
                if (retryCount <= maxRetryCount) {
                    if (throwable instanceof ConnectException
                            || throwable instanceof SocketTimeoutException
                            || throwable instanceof TimeoutException || throwable instanceof UnknownHostException || throwable instanceof EOFException) {
                        return retry(throwable);
                    } else if (throwable instanceof AuthException) {
                        login();
                        return Observable.error(HttpExceptionHandle.handleException(throwable));
                    } else if (throwable instanceof TokenInvalidException) {
                        login();
                        return Observable.error(HttpExceptionHandle.handleException(throwable));
                    }
                    if (throwable instanceof HttpException) {
                        HttpException he = (HttpException) throwable;
                        if (he.code() != 401 && he.code() != 403 && he.code() != 409) {
                            return Observable.error(HttpExceptionHandle.handleException(throwable));
                        }else {
                            return retry(throwable);
                        }
                    }
                    return Observable.error(HttpExceptionHandle.handleException(throwable));
                }else{
                    if (throwable instanceof HttpException) {
                        HttpException he = (HttpException) throwable;
                        if (he.code() == 401 || he.code() == 403 || he.code() == 409) {
                            login();
                        }
                    }
                    return Observable.error(HttpExceptionHandle.handleException(throwable));
                }

            }

            /**
             *
             * @param throwable
             * @return
             */
            private Observable<?> retry(Throwable throwable) {
                if (retryCount <= maxRetryCount) {
                    return Observable.timer(retryDelaySecond,
                            TimeUnit.SECONDS).observeOn(Schedulers.io());
                } else {
                    return Observable.error(throwable);
                }
            }

            private void login() {
                if(sOnErrorLoginCallback != null){
                    sOnErrorLoginCallback.onErrorLogin(OnErrorLoginCallback.INTENT_FLAG);
                }
            }
        };
    }

    public static <T> ObservableTransformer<T, T> applySchedulers() {
        return new ObservableTransformer<T, T>() {
            @Override
            public ObservableSource<T> apply(Observable<T> observable) {
                return observable.subscribeOn(Schedulers.io())
                        .observeOn(AndroidSchedulers.mainThread()).retryWhen(getRetryFunc1());

            }
        };
    }

    public static <T> ObservableTransformer<T, T> applySchedulers(final LifecycleTransformer transformer, Class<T> tClass) {
        return new ObservableTransformer<T, T>() {
            @Override
            public Observable<T> apply(Observable<T> observable) {
                return observable.subscribeOn(Schedulers.io())
                        .observeOn(AndroidSchedulers.mainThread()).retryWhen(getRetryFunc1()).compose(transformer);
            }
        };
    }
    public static <T> ObservableTransformer<T, T> applySchedulers(final LifecycleTransformer<T> transformer) {
        return new ObservableTransformer<T, T>() {
            @Override
            public Observable<T> apply(Observable<T> observable) {
                return observable.subscribeOn(Schedulers.io())
                        .observeOn(AndroidSchedulers.mainThread()).retryWhen(getRetryFunc1()).compose(transformer);
            }
        };
    }

```