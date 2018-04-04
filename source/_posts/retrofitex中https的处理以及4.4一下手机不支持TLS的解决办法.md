
---
title: retrofitex中https的处理以及4.4一下手机不支持TLS的解决办法
date: 2018-04-4 12:12:20
categories: https
tags: [https,TSL,SSLHandshakeException,SSLSocketFactory]
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


### 前言

在4.4手机上传和下载图片的时候发现无法上传，或者显示图片，经查错误信息如下
>javax.net.ssl.SSLHandshakeException: javax.net.ssl.SSLProtocolException: SSL handshake aborted: ssl=0x79f145b0: Failure in SSL library, usually a protocol error
error:1407742E:SSL routines:SSL23_GET_SERVER_HELLO:tlsv1 alert protocol version

原因：Android 4.4及以下的系统默认不支持TLS 1.1和TLS1.2 协议，后台使用的TLS协议的是1.1或者1.2客户端就会请求失败。

<table style="width: 100%" align="left">
<thead>
<tr><th>Protocol</th><th>Supported (API Levels)</th><th>Enabled by default (API Levels)</th></tr>
</thead>
<tbody>
<tr>
<td>SSLv3</td>
<td>1+</td>
<td>1+</td>
</tr>
<tr>
<td>TLSv1</td>
<td>1+</td>
<td>1+</td>
</tr>
<tr>
<td>TLSv1.1</td>
<td>16+</td>
<td>20+</td>
</tr>
<tr>
<td>TLSv1.2</td>
<td>16+</td>
<td>20+</td>
</tr>
</tbody>
</table>

>* TLSv1.0从API 1+就被默认打开
>* TLSv1.1和TLSv1.2只有在API 20+ 才会被默认打开
>* 低于API 20+的版本是默认关闭对TLSv1.1和TLSv1.2的支持，若要支持则必须自己打开

解决方案就是根据当前版本支持的TSL版本，在创建socket的时候开启对应的版本的支持，但是服务器要全部支持1.0 、1.1和1.2，否则在低于16的android手机上即使客户端添加了代码还是不支持1.1和1.2的。
参考文章
https://blog.csdn.net/s003603u/article/details/53907910
https://www.cnblogs.com/daquexian/p/5883585.html
```java



public class SSLSocketFactoryCompat extends SSLSocketFactory {
    private static volatile  SSLSocketFactory defaultFactory;
    // Android 5.0+ (API level21) provides reasonable default settings
    // but it still allows SSLv3
    // https://developer.android.com/about/versions/android-5.0-changes.html#ssl
    static String protocols[] = null, cipherSuites[] = null;
    static {
        try {
            SSLSocket socket = (SSLSocket) SSLSocketFactory.getDefault().createSocket();
            if (socket != null) {
                /* set reasonable protocol versions */
                // - enable all supported protocols (enables TLSv1.1 and TLSv1.2 on Android <5.0)
                // - remove all SSL versions (especially SSLv3) because they're insecure now
                List<String> protocols = new LinkedList<>();
                for (String protocol : socket.getSupportedProtocols())
                    if (!protocol.toUpperCase().contains("SSL"))
                        protocols.add(protocol);
                SSLSocketFactoryCompat.protocols = protocols.toArray(new String[protocols.size()]);
                /* set up reasonable cipher suites */
                if (Build.VERSION.SDK_INT < Build.VERSION_CODES.LOLLIPOP) {
                    // choose known secure cipher suites
                    List<String> allowedCiphers = Arrays.asList(
                            // TLS 1.2
                            "TLS_RSA_WITH_AES_256_GCM_SHA384",
                            "TLS_RSA_WITH_AES_128_GCM_SHA256",
                            "TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256",
                            "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256",
                            "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384",
                            "TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256",
                            "TLS_ECHDE_RSA_WITH_AES_128_GCM_SHA256",
                            // maximum interoperability
                            "TLS_RSA_WITH_3DES_EDE_CBC_SHA",
                            "TLS_RSA_WITH_AES_128_CBC_SHA",
                            // additionally
                            "TLS_RSA_WITH_AES_256_CBC_SHA",
                            "TLS_ECDHE_ECDSA_WITH_3DES_EDE_CBC_SHA",
                            "TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA",
                            "TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA",
                            "TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA");
                    List<String> availableCiphers = Arrays.asList(socket.getSupportedCipherSuites());
                    // take all allowed ciphers that are available and put them into preferredCiphers
                    HashSet<String> preferredCiphers = new HashSet<>(allowedCiphers);
                    preferredCiphers.retainAll(availableCiphers);
                    /* For maximum security, preferredCiphers should *replace* enabled ciphers (thus disabling
                     * ciphers which are enabled by default, but have become unsecure), but I guess for
                     * the security level of DAVdroid and maximum compatibility, disabling of insecure
                     * ciphers should be a server-side task */
                    // add preferred ciphers to enabled ciphers
                    HashSet<String> enabledCiphers = preferredCiphers;
                    enabledCiphers.addAll(new HashSet<>(Arrays.asList(socket.getEnabledCipherSuites())));
                    SSLSocketFactoryCompat.cipherSuites = enabledCiphers.toArray(new String[enabledCiphers.size()]);
                }
                socket.close();
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
    public SSLSocketFactoryCompat(X509TrustManager tm) {
        try {
            SSLContext sslContext = SSLContext.getInstance("TLS");
            sslContext.init(null, (tm != null) ? new X509TrustManager[] { tm } : null, null);
            defaultFactory = sslContext.getSocketFactory();
        } catch (GeneralSecurityException e) {
            throw new AssertionError(); // The system has no TLS. Just give up.
        }
    }
    public static SSLSocketFactoryCompat get(X509TrustManager tm) {
        return new SSLSocketFactoryCompat(tm);
    }
    static SSLSocketFactoryCompat sSSLSocketFactoryCompat;
    public static SSLSocketFactoryCompat get() {
        if (sSSLSocketFactoryCompat == null) {
            synchronized (SSLSocketFactoryCompat.class){
                if(sSSLSocketFactoryCompat == null){
                    final X509TrustManager trustAllCert =
                            new X509TrustManager() {
                                @Override
                                public void checkClientTrusted(java.security.cert.X509Certificate[] chain, String authType) throws CertificateException {
                                }

                                @Override
                                public void checkServerTrusted(java.security.cert.X509Certificate[] chain, String authType) throws CertificateException {
                                }

                                @Override
                                public java.security.cert.X509Certificate[] getAcceptedIssuers() {
                                    return new java.security.cert.X509Certificate[]{};
                                }
                            };
                    sSSLSocketFactoryCompat = get(trustAllCert);
                }
            }
        }

         return sSSLSocketFactoryCompat;
    }

    private void upgradeTLS(SSLSocket ssl) {
        // Android 5.0+ (API level21) provides reasonable default settings
        // but it still allows SSLv3
        // https://developer.android.com/about/versions/android-5.0-changes.html#ssl
        if (protocols != null) {
            ssl.setEnabledProtocols(protocols);
        }
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.LOLLIPOP && cipherSuites != null) {
            ssl.setEnabledCipherSuites(cipherSuites);
        }
    }
    @Override
    public String[] getDefaultCipherSuites() {
        return cipherSuites;
    }
    @Override
    public String[] getSupportedCipherSuites() {
        return cipherSuites;
    }

    @Override
    public Socket createSocket() throws IOException {
        Socket ssl = defaultFactory.createSocket();
        if (ssl instanceof SSLSocket)
            upgradeTLS((SSLSocket)ssl);
        return ssl;
    }

    @Override
    public Socket createSocket(Socket s, String host, int port, boolean autoClose) throws IOException {
        Socket ssl = defaultFactory.createSocket(s, host, port, autoClose);
        if (ssl instanceof SSLSocket)
            upgradeTLS((SSLSocket)ssl);
        return ssl;
    }
    @Override
    public Socket createSocket(String host, int port) throws IOException, UnknownHostException {
        Socket ssl = defaultFactory.createSocket(host, port);
        if (ssl instanceof SSLSocket)
            upgradeTLS((SSLSocket)ssl);
        return ssl;
    }
    @Override
    public Socket createSocket(String host, int port, InetAddress localHost, int localPort) throws IOException, UnknownHostException {
        Socket ssl = defaultFactory.createSocket(host, port, localHost, localPort);
        if (ssl instanceof SSLSocket)
            upgradeTLS((SSLSocket)ssl);
        return ssl;
    }
    @Override
    public Socket createSocket(InetAddress host, int port) throws IOException {
        Socket ssl = defaultFactory.createSocket(host, port);
        if (ssl instanceof SSLSocket)
            upgradeTLS((SSLSocket)ssl);
        return ssl;
    }
    @Override
    public Socket createSocket(InetAddress address, int port, InetAddress localAddress, int localPort) throws IOException {
        Socket ssl = defaultFactory.createSocket(address, port, localAddress, localPort);
        if (ssl instanceof SSLSocket)
            upgradeTLS((SSLSocket)ssl);
        return ssl;
    }

    private Socket enableTLSOnSocket(Socket socket) {
        if(socket != null && (socket instanceof SSLSocket)) {
            ((SSLSocket)socket).setEnabledProtocols(new String[] {"TLSv1.1", "TLSv1.2"});
        }
        return socket;
    }
}
```


关键方法是 `ssl.setEnabledProtocols(protocols)`和` ssl.setEnabledCipherSuites(cipherSuites);`

每次创建SSLSocket时候都要调用上面两个方法

### 
```java

public class HttpsDelegate {

    private static volatile HttpsDelegate mInstance;

    private HttpsDelegate(){

    }

    public static HttpsDelegate getInstance() {
        if(null == mInstance) {
            synchronized (HttpsDelegate.class) {
                if(null == mInstance) {
                    mInstance = new HttpsDelegate();
                }
            }
        }
        return mInstance;
    }

    public static  Pair<SSLSocketFactory,X509TrustManager> getUnsafeSslSocketFactory(){
        return getSslSocketFactory(null,null,null);
    }
    public static  Pair<SSLSocketFactory,X509TrustManager> getUnsafeSslSocketFactory(InputStream[] certificates){
        return getSslSocketFactory(certificates,null,null);
    }
    public static Pair<SSLSocketFactory,X509TrustManager> getSslSocketFactory(InputStream[] certificates, InputStream bksFile, String password)
    {
        Pair<SSLSocketFactory,X509TrustManager > sslParams = null;
        try
        {
            TrustManager[] trustManagers = prepareTrustManager(certificates);
            KeyManager[] keyManagers = prepareKeyManager(bksFile, password);
            SSLContext sslContext = SSLContext.getInstance("TLS");
            X509TrustManager trustManager = null;
            if (trustManagers != null)
            {
                trustManager = new MyTrustManager(chooseTrustManager(trustManagers));
            } else
            {
                trustManager = new UnSafeTrustManager();
            }
            sslContext.init(keyManagers, new TrustManager[]{trustManager},null);
            sslParams =new Pair<>(SSLSocketFactoryCompat.get(),trustManager);
            return sslParams;
        } catch (NoSuchAlgorithmException e)
        {
            throw new AssertionError(e);
        } catch (KeyManagementException e)
        {
            throw new AssertionError(e);
        } catch (KeyStoreException e)
        {
            throw new AssertionError(e);
        }
    }


    private static class UnSafeTrustManager implements X509TrustManager
    {
        @Override
        public void checkClientTrusted(X509Certificate[] chain, String authType)
                throws CertificateException
        {
        }

        @Override
        public void checkServerTrusted(X509Certificate[] chain, String authType)
                throws CertificateException
        {
        }

        @Override
        public X509Certificate[] getAcceptedIssuers()
        {
            return new java.security.cert.X509Certificate[]{};
        }
    }

    private static TrustManager[] prepareTrustManager(InputStream... certificates)
    {
        if (certificates == null || certificates.length <= 0) return null;
        try
        {

            CertificateFactory certificateFactory = CertificateFactory.getInstance("X.509");
            KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());
            keyStore.load(null);
            int index = 0;
            for (InputStream certificate : certificates)
            {
                String certificateAlias = Integer.toString(index++);
                keyStore.setCertificateEntry(certificateAlias, certificateFactory.generateCertificate(certificate));
                try
                {
                    if (certificate != null)
                        certificate.close();
                } catch (IOException e)

                {
                }
            }
            TrustManagerFactory trustManagerFactory = null;

            trustManagerFactory = TrustManagerFactory.
                    getInstance(TrustManagerFactory.getDefaultAlgorithm());
            trustManagerFactory.init(keyStore);

            TrustManager[] trustManagers = trustManagerFactory.getTrustManagers();

            return trustManagers;
        } catch (NoSuchAlgorithmException e)
        {
            e.printStackTrace();
        } catch (CertificateException e)
        {
            e.printStackTrace();
        } catch (KeyStoreException e)
        {
            e.printStackTrace();
        } catch (Exception e)
        {
            e.printStackTrace();
        }
        return null;

    }

    private static KeyManager[] prepareKeyManager(InputStream bksFile, String password)
    {
        try
        {
            if (bksFile == null || password == null) return null;

            KeyStore clientKeyStore = KeyStore.getInstance("BKS");
            clientKeyStore.load(bksFile, password.toCharArray());
            KeyManagerFactory keyManagerFactory = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
            keyManagerFactory.init(clientKeyStore, password.toCharArray());
            return keyManagerFactory.getKeyManagers();

        } catch (KeyStoreException e)
        {
            e.printStackTrace();
        } catch (NoSuchAlgorithmException e)
        {
            e.printStackTrace();
        } catch (UnrecoverableKeyException e)
        {
            e.printStackTrace();
        } catch (CertificateException e)
        {
            e.printStackTrace();
        } catch (IOException e)
        {
            e.printStackTrace();
        } catch (Exception e)
        {
            e.printStackTrace();
        }
        return null;
    }

    private static X509TrustManager chooseTrustManager(TrustManager[] trustManagers)
    {
        for (TrustManager trustManager : trustManagers)
        {
            if (trustManager instanceof X509TrustManager)
            {
                return (X509TrustManager) trustManager;
            }
        }
        return null;
    }


    private static class MyTrustManager implements X509TrustManager
    {
        private X509TrustManager defaultTrustManager;
        private X509TrustManager localTrustManager;

        public MyTrustManager(X509TrustManager localTrustManager) throws NoSuchAlgorithmException, KeyStoreException
        {
            TrustManagerFactory var4 = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
            var4.init((KeyStore) null);
            defaultTrustManager = chooseTrustManager(var4.getTrustManagers());
            this.localTrustManager = localTrustManager;
        }


        @Override
        public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException
        {

        }

        @Override
        public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException
        {
            try
            {
                defaultTrustManager.checkServerTrusted(chain, authType);
            } catch (CertificateException ce)
            {
                localTrustManager.checkServerTrusted(chain, authType);
            }
        }


        @Override
        public X509Certificate[] getAcceptedIssuers()
        {
            return new X509Certificate[0];
        }
    }

}
```
### 添加到OkhttpClient中
```java
OkHttpClient.Builder builder;
Pair<SSLSocketFactory, X509TrustManager> sslFactory =HttpsDelegate.getUnsafeSslSocketFactory()
 builder.sslSocketFactory(sslFactory.first, sslFactory.second);
```