---
layout: post
title: Jetty httpclient和Apache httpclient使用总结（下）
tags: jetty apache httpclient 
categories: Java
---

<div class="toc"></div>

<br/>
这一片介绍一下Jetty Httpclient的有关实现，Jetty Httpclient相比Apache来说更轻量。

Jetty：
-------


----------
**Jetty HttpClient信任所有证书**：
 

    public static HttpClient getHttpClient() throws GeneralSecurityException {
        // Trust own CA and all self-signed certs
		String protocols = "TLSv1.2";
        SSLContext sslContext = SSLContext.getInstance("SSL");
        sslContext.init(null, new TrustAnyTrustManager[] {new TrustAnyTrustManager()}, new SecureRandom());

        // Allow TLSv1 protocol only
        SslContextFactory sslctx = new SslContextFactory();
        sslctx.setSslContext(sslContext);
        sslctx.setIncludeProtocols(protocols.split(","));

        return new HttpClient(sslctx);
    }

	private static class TrustAnyTrustManager implements X509TrustManager {
		public void checkClientTrusted(X509Certificate[] chain, String authType)
				throws CertificateException {
		}

		public void checkServerTrusted(X509Certificate[] chain, String authType)
				throws CertificateException {
		}

		public X509Certificate[] getAcceptedIssuers() {
			return new X509Certificate[] {};
		}
	}

**Jetty HttpClient 信任指定证书：**

通常证书文件指的是keystore和truststore，keystore一般存放的是私钥，用来加解密或者进行签名，truststore中保存的是一些可信任的证书。具体如何生成这两个证书文件，这里先不做介绍。

    private static SSLContext getCert()
    {
        String keyStore = DefaultEnvUtil.getAppRoot() + File.separator
                + "*******"; // 证书的路径，jks格式
        String trustStore = DefaultEnvUtil.getAppRoot() + File.separator
                + "*******"; // 密钥库文件，jks格式
        String keyPass = "******"; // pfx文件的密码
        String trustPass = "*******"; // jks文件的密码

        SSLContext sslContext = null;
        try
        {
            KeyStore ks = KeyStore.getInstance("JKS");

            ks.load(new FileInputStream(keyStore), keyPass.toCharArray());
            KeyManagerFactory kmf = KeyManagerFactory.getInstance("sunx509");
            kmf.init(ks, keyPass.toCharArray());

            KeyStore ts = KeyStore.getInstance("JKS");
            // 加载jks文件
            ts.load(new FileInputStream(trustStore), trustPass.toCharArray());
            TrustManager[] tm;
            TrustManagerFactory tmf = TrustManagerFactory
                    .getInstance("sunx509");
            tmf.init(ts);
            tm = tmf.getTrustManagers();

            sslContext = SSLContext.getInstance("TLS");
            // 初始化
            sslContext.init(kmf.getKeyManagers(), tm, null);
        }
        catch (IOException e)
        {
            logger.error("HttpsClient::getCert read keystore or truststore get error.", e);
        }
        catch (Exception e)
        {
            logger.error("HttpsClient::getCert get error.", e);
        }
        return sslContext;
    }
	private static HttpClient getHttpClient()
    {
        // 下面是重点
        SSLSocketFactory socketFactory = new SSLSocketFactory(getCert());
        socketFactory.setHostnameVerifier(new AllowAllHostnameVerifier());

        HttpClient httpclient = new DefaultHttpClient();
        httpclient.getConnectionManager().getSchemeRegistry().register(new Scheme("https", 443, socketFactory));
        return httpclient;
    }

<br>
以上用到的jar包包括：

    import java.security.GeneralSecurityException;
	import java.security.SecureRandom;
	import java.security.cert.CertificateException;
	import java.security.cert.X509Certificate;

	import javax.net.ssl.SSLContext;
	import javax.net.ssl.X509TrustManager;

	import org.eclipse.jetty.client.HttpClient;
	import org.eclipse.jetty.util.ssl.SslContextFactory;

