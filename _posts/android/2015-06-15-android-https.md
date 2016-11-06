---
layout: post
title: "Android处理https请求的证书问题"
excerpt: "Custom written post descriptions are the way to go... if you're not lazy."
categories: android
tags: [android]
comments: true
share: true
comments: true
share:
---

android中对部分站点发送https请求会报错，原因是该站点的证书时自定义的，而非官方的，android手机不信任其证书，为了解决这个问题，一般有两种解决方案

+ 忽略证书验证

+ 下载证书到本地，添加到信任证书列表

在android中我们访问网络一般有两种方式，一种是使用httpURLConnection，一种是使用httpClient。这里先讲前者，然后再讲解后者。顺便会讲解在volley框架中如何使用https。


## 一，HttpURLConnection访问https站点

### 1，忽略证书


如果要忽略证书的验证，我们只需提供一个HostnameVerifier和X509TrustManager的空实现，如下代码

{% highlight java linenos %}

public void requestWithoutCA() {
	try {

		SSLContext sc = SSLContext.getInstance("TLS");
		sc.init(null, new TrustManager[] { new MyTrustManager() },
				new SecureRandom());
		HttpsURLConnection
				.setDefaultSSLSocketFactory(sc.getSocketFactory());
		HttpsURLConnection
				.setDefaultHostnameVerifier(new MyHostnameVerifier());

		URL url = new URL("https://certs.cac.washington.edu/CAtest/");
		HttpURLConnection urlConnection = (HttpURLConnection) url
				.openConnection();

		InputStream in = urlConnection.getInputStream();
		// 取得输入流，并使用Reader读取
		BufferedReader reader = new BufferedReader(
				new InputStreamReader(in));
		System.out.println("=============================");
		System.out.println("Contents of get request");
		System.out.println("=============================");
		String lines;
		while ((lines = reader.readLine()) != null) {
			System.out.println(lines);
		}
		reader.close();
		// 断开连接
		urlConnection.disconnect();
		System.out.println("=============================");
		System.out.println("Contents of get request ends");
		System.out.println("=============================");
	} catch (MalformedURLException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	} catch (NoSuchAlgorithmException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	} catch (KeyManagementException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}
}


private class MyHostnameVerifier implements HostnameVerifier {
	@Override
	public boolean verify(String hostname, SSLSession session) {
		// TODO Auto-generated method stub
		return true;
	}

}

private class MyTrustManager implements X509TrustManager {
	@Override
	public void checkClientTrusted(X509Certificate[] chain, String authType)
			throws CertificateException {
		// TODO Auto-generated method stub
	}

	@Override
	public void checkServerTrusted(X509Certificate[] chain, String authType)

	throws CertificateException {
		// TODO Auto-generated method stub
	}

	@Override
	public X509Certificate[] getAcceptedIssuers() {
		// TODO Auto-generated method stub
		return null;
	}

}


{% endhighlight %}



### 2，使用证书

使用证书这部分我是参考于官网：
[Security with HTTPS and SSL](https://developer.android.com/training/articles/security-ssl.html)

该代码时访问华盛顿大学的一个网页，证书可以到以下链接下载：

[https://www.washington.edu/itconnect/security/ca/load-der.crt](https://www.washington.edu/itconnect/security/ca/load-der.crt)

代码如下：

注意，此处使用的是HttpsURLConnection，而非HttpURLConnection。

下面的代码，加载证书等等操作，最终都是为了setSSLSocketFactory这一步。

{% highlight java %}


 public void requestByHttps() {
	try {

		// Load CAs from an InputStream
		// (could be from a resource or ByteArrayInputStream or ...)

		CertificateFactory cf = CertificateFactory.getInstance("X.509");
		// From
		// https://www.washington.edu/itconnect/security/ca/load-der.crt
		InputStream caInput = this.getAssets().open("load_der.crt");
		Certificate ca;
		try {
			ca = cf.generateCertificate(caInput);
			System.out.println("ca="
					+ ((X509Certificate) ca).getSubjectDN());
		} finally {
			caInput.close();
		}

		// Create a KeyStore containing our trusted CAs
		String keyStoreType = KeyStore.getDefaultType();
		KeyStore keyStore = KeyStore.getInstance(keyStoreType);
		keyStore.load(null, null);
		keyStore.setCertificateEntry("ca", ca);

		// Create a TrustManager that trusts the CAs in our KeyStore
		String tmfAlgorithm = TrustManagerFactory.getDefaultAlgorithm();
		TrustManagerFactory tmf = TrustManagerFactory
				.getInstance(tmfAlgorithm);
		tmf.init(keyStore);

		// Create an SSLContext that uses our TrustManager
		SSLContext context = SSLContext.getInstance("TLS");
		context.init(null, tmf.getTrustManagers(), null);

		URL url = new URL("https://certs.cac.washington.edu/CAtest/");
		HttpsURLConnection urlConnection = (HttpsURLConnection) url
				.openConnection();

		urlConnection.setSSLSocketFactory(context.getSocketFactory());
		InputStream in = urlConnection.getInputStream();
		// 取得输入流，并使用Reader读取
		BufferedReader reader = new BufferedReader(
				new InputStreamReader(in));
		System.out.println("=============================");
		System.out.println("Contents of get request");
		System.out.println("=============================");
		String lines;
		while ((lines = reader.readLine()) != null) {
			System.out.println(lines);
			// tv.setText(tv.getText().toString() + lines);
		}
		reader.close();
		// 断开连接
		urlConnection.disconnect();
		System.out.println("=============================");
		System.out.println("Contents of get request ends");
		System.out.println("=============================");
	} catch (MalformedURLException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	} catch (NoSuchAlgorithmException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	} catch (KeyManagementException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	} catch (KeyStoreException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	} catch (CertificateException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}
}

{% endhighlight %}


### 3， Volley中使用HttpURLConnection访问https

Volley中使用HttpURLConnection访问网络时，是由HurlStack类处理，该类的几个构造方法如下：


{% highlight java %}


public HurlStack() {
    this(null);
}

/**
 * @param urlRewriter Rewriter to use for request URLs
 */
public HurlStack(UrlRewriter urlRewriter) {
    this(urlRewriter, null);
}

/**
 * @param urlRewriter Rewriter to use for request URLs
 * @param sslSocketFactory SSL factory to use for HTTPS connections
 */
public HurlStack(UrlRewriter urlRewriter, SSLSocketFactory sslSocketFactory) {
    mUrlRewriter = urlRewriter;
    mSslSocketFactory = sslSocketFactory;
}



{% endhighlight %}


打开连接的方法如下，注意其中也有上面提到的的setSSLSocketFactory方法。

结合构造方法以及以下的代码，不难理解，为了访问https，只需奥在构建HurlStack时，传入一个已经设置了证书的SLSocketFactory即可，设置代码和上面的代码类似。

{% highlight java lineno %}

private HttpURLConnection openConnection(URL url, Request<?> request) throws IOException {
    HttpURLConnection connection = createConnection(url);

    int timeoutMs = request.getTimeoutMs();
    connection.setConnectTimeout(timeoutMs);
    connection.setReadTimeout(timeoutMs);
    connection.setUseCaches(false);
    connection.setDoInput(true);

    // use caller-provided custom SslSocketFactory, if any, for HTTPS
    if ("https".equals(url.getProtocol()) && mSslSocketFactory != null) {
        ((HttpsURLConnection)connection).setSSLSocketFactory(mSslSocketFactory);
    }

    return connection;
}


{% endhighlight %}


## 二，使用httpClient访问https站点

### 1,忽略证书



{% highlight java lineno %}

public class HttpClientHelper {

    private static HttpClient httpClient;

    private HttpClientHelper() {
    }

    public static synchronized HttpClient getHttpClient() {

        if (null == httpClient) {
            // 初始化工作
            try {
                KeyStore trustStore = KeyStore.getInstance(KeyStore
                        .getDefaultType());
                trustStore.load(null, null);
                SSLSocketFactory sf = new SSLSocketFactoryEx(trustStore);
                sf.setHostnameVerifier(SSLSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER);  //允许所有主机的验证

                HttpParams params = new BasicHttpParams();

                HttpProtocolParams.setVersion(params, HttpVersion.HTTP_1_1);
                HttpProtocolParams.setContentCharset(params,
                        HTTP.DEFAULT_CONTENT_CHARSET);
                HttpProtocolParams.setUseExpectContinue(params, true);

                // 设置连接管理器的超时
                ConnManagerParams.setTimeout(params, 10000);
                // 设置连接超时
                HttpConnectionParams.setConnectionTimeout(params, 10000);
                // 设置socket超时
                HttpConnectionParams.setSoTimeout(params, 10000);

                // 设置http https支持
                SchemeRegistry schReg = new SchemeRegistry();
                schReg.register(new Scheme("http", PlainSocketFactory
                        .getSocketFactory(), 80));
                schReg.register(new Scheme("https", sf, 443));

                ClientConnectionManager conManager = new ThreadSafeClientConnManager(
                        params, schReg);

                httpClient = new DefaultHttpClient(conManager, params);
            } catch (Exception e) {
                e.printStackTrace();
                return new DefaultHttpClient();
            }
        }
        return httpClient;
    }

}

class SSLSocketFactoryEx extends SSLSocketFactory {

    SSLContext sslContext = SSLContext.getInstance("TLS");

    public SSLSocketFactoryEx(KeyStore truststore)
            throws NoSuchAlgorithmException, KeyManagementException,
            KeyStoreException, UnrecoverableKeyException {
        super(truststore);

        TrustManager tm = new X509TrustManager() {

            @Override
            public java.security.cert.X509Certificate[] getAcceptedIssuers() {
                return null;
            }

            @Override
            public void checkClientTrusted(
                    java.security.cert.X509Certificate[] chain, String authType)
                    throws java.security.cert.CertificateException {

            }

            @Override
            public void checkServerTrusted(
                    java.security.cert.X509Certificate[] chain, String authType)
                    throws java.security.cert.CertificateException {

            }
        };

        sslContext.init(null, new TrustManager[] { tm }, null);
    }

    @Override
    public Socket createSocket(Socket socket, String host, int port,
            boolean autoClose) throws IOException, UnknownHostException {
        return sslContext.getSocketFactory().createSocket(socket, host, port,
                autoClose);
    }

    @Override
    public Socket createSocket() throws IOException {
        return sslContext.getSocketFactory().createSocket();
    }
}


{% endhighlight %}



### 2,使用证书



{% highlight java lineno %}


AssetManager am = context.getAssets();

InputStream ins = am.open("robusoft.cer");

try {

        //读取证书

        CertificateFactory cerFactory = CertificateFactory.getInstance("X.509");  //问1

        Certificate cer = cerFactory.generateCertificate(ins);

        //创建一个证书库，并将证书导入证书库

        KeyStore keyStore = KeyStore.getInstance("PKCS12", "BC");   //问2

        keyStore.load(null, null);

        keyStore.setCertificateEntry("trust", cer);

        return keyStore;

} finally {

        ins.close();

}

//把咱的证书库作为信任证书库

SSLSocketFactory socketFactory = new SSLSocketFactory(keystore);

Scheme sch = new Scheme("https", socketFactory, 443);

//完工

HttpClient mHttpClient = new DefaultHttpClient();

mHttpClient.getConnectionManager().getSchemeRegistry().register(sch);

//接下来这个httpclient就可以用语https请求了。

{% endhighlight %}



### 3,Volley中使用httpClient访问https

Volley使用HttpCLient访问网络是借助HttpClientStack类，其中的构造方法不同于HurlStack那么多，而只有一个

{% highlight java lineno %}


public HttpClientStack(HttpClient client) {
    mClient = client;
}

{% endhighlight %}

很简单，我们只需要在构造HttpClientStack时传入一个已经设置了证书的HttpClient，按照上面讲述的那样设置即可。
