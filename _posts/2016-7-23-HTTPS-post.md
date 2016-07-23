---
layout: post
title:  "利用HttpClient发送HTTPS-POST请求"
date:   2016-7-23 
categories: HTTPS
tags: HTTPS
---

* content
{:toc}
　　现在很多就扣都使用了HTTPS,本人现在公司做的项目也是需要使用HTTPS与服务端进行通信，网上查阅了许多，结合本人项目的实际情况总结了一下。




### 背景
　　首先，项目背景是ssm框架的，使用maven管理，出于方便使用apache的HttpClient来实现本功能。

### 添加依赖
　　打开项目pom文件，在<dependencies>节点中增加如下配置：   
    <dependency>
    	<groupId>org.apache.httpcomponents</groupId>
    	<artifactId>httpclient</artifactId>
    	<version>4.0</version>
    </dependency>

### 重写证书验证方法
    private static HttpClient wrapClient(HttpClient httpClient) {
	    try {
		    SSLContext ctx = SSLContext.getInstance("TLS");
		    X509TrustManager tm = new X509TrustManager() {
			    public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException {
			    }
			    public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException {
			    }
			    public X509Certificate[] getAcceptedIssuers() {
			    	return null;
			    }
		    };
		    ctx.init(null, new TrustManager[] { tm }, null);
		    SSLSocketFactory ssf = new SSLSocketFactory(ctx);
		    ssf.setHostnameVerifier(SSLSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER);
		    ClientConnectionManager ccm = httpClient.getConnectionManager();
		    sr = ccm.getSchemeRegistry();
		    //设置要使用的端口，默认是443
		    sr.register(new Scheme("https", ssf, 443));
		    return new DefaultHttpClient(ccm, httpClient.getParams());
	    } catch (Exception ex) {
		    ex.printStackTrace();
		    return null;
	    }
    }

### HTTPS-Post方法主体
    private static final String APPLICATION_JSON = "application/json";
    private static final String CONTENT_TYPE_TEXT_JSON = "text/json";

    public static String sendPostSSLRequest(String requestUrl, String jsonParam) {
	    HttpClient httpClient = null;
	    HttpPost httpPost;
	    String result = null;
	    try{
	        httpClient = wrapClient(new DefaultHttpClient());
	        httpPost = new HttpPost(requestUrl);
	        httpPost.addHeader(HTTP.CONTENT_TYPE, APPLICATION_JSON);
	
	        // 设置参数
	        String encoderJson = URLEncoder.encode(jsonParam, HTTP.UTF_8);
	        StringEntity se = new StringEntity(encoderJson);
	        se.setContentType(CONTENT_TYPE_TEXT_JSON);
	        se.setContentEncoding(new BasicHeader(HTTP.CONTENT_TYPE, APPLICATION_JSON));
	        httpPost.setEntity(se);
	
	        HttpResponse response = httpClient.execute(httpPost);
	        if(null != response && response.getStatusLine().getStatusCode() == 200){
	            HttpEntity resEntity = response.getEntity();
	            if(null != resEntity){
	                result = EntityUtils.toString(resEntity,HTTP.UTF_8);
	            }
	        }
	    } catch (UnsupportedEncodingException e) {
	        e.printStackTrace();
	    } catch (ClientProtocolException e) {
	        e.printStackTrace();
	    } catch (IOException e) {
	        e.printStackTrace();
	    } catch (Exception e) {
	        e.printStackTrace();
	    }finally {
	        // 关闭连接，释放资源
	        httpClient.getConnectionManager().shutdown();
	    }
	    return result;
    }
