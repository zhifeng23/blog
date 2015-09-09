---
layout: post
title: Jetty httpclient和Apache httpclient使用总结（上）
tags: jetty apache httpclient 
categories: Java
---

<div class="toc"></div>

<br/>
调用Restful接口的方式一般是使用httpclient实现的，而httpclient有两个版本，Jetty和Apache，以http方式访问服务的实现，很容易google到，在这里具体的讨论下以https访问服务的实现方式，下面逐个分析：

Apache：
-------


----------
**Apache HttpClient信任所有证书**：

    private static class TrustAnyTrustManager implements X509TrustManager {
        public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException {
        }
        
        public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException {
        }
        
        public X509Certificate[] getAcceptedIssuers() {
            return new X509Certificate[] {};
        }
    }
    
    public static CloseableHttpClient getHttpClient() throws GeneralSecurityException 
	{
        // Trust own CA and all self-signed certs
        SSLContext sslcontext = SSLContext.getInstance("SSL");
        sslcontext.init(null, new TrustAnyTrustManager[] {new TrustAnyTrustManager()}, new SecureRandom());
        
        // Allow TLSv1 protocol only
        SslContextFactory sslctx = new SslContextFactory();
        sslctx.setSslContext(sslcontext);
        
        SSLConnectionSocketFactory sslsf =
            new SSLConnectionSocketFactory(sslcontext, new String[] {"TLSv1.2"}, null,
                SSLConnectionSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER);
        
        return HttpClients.custom().setSSLSocketFactory(sslsf).build();
    }
 <br>   
**Apache HttpClient 信任指定证书：**

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
有关Jetty HttpClient的实现，下一篇记录。

