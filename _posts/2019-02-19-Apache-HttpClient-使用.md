---
layout: post
title: Apache HttpClient 使用                              # Title of the page
hide_title: false                                  # Hide the title when displaying the post, but shown in lists of posts
excerpt_separator: <!--more-->
#feature-img: "assets/img/sample.png"              # Add a feature-image to the post
#thumbnail: "assets/img/avatar.png"  # Add a thumbnail image on blog view
#color: rgb(80,140,22)                             # Add the specified color as feature image, and change link colors in post
bootstrap: true                                   # Add bootstrap to the page
tags: [java, http]
---
## 一、实列化 HttpClient 对象
<!--more-->
``` 
4.3以下
DefaultHttpClient client = new DefaultHttpClient();
4.3及以上
HttpClientBuilder httpClientBuilder = HttpClients.custom();
CloseableHttpClient httpClient = httpClientBuilder.build();
```
## 二、设置超时
ConnectTimeout： 链接建立的超时时间；  
SocketTimeout：响应超时时间，超过此时间不再读取响应；  
ConnectionRequestTimeout： http clilent中从connetcion pool中获得一个connection的超时时间；
```
4.3以下
int timeout = 5; // seconds
HttpParams httpParams = httpClient.getParams();
httpParams.setParameter(CoreConnectionPNames.CONNECTION_TIMEOUT, timeout * 1000);
httpParams.setParameter(CoreConnectionPNames.SO_TIMEOUT, timeout * 1000);
// httpParams.setParameter(ClientPNames.CONN_MANAGER_TIMEOUT, new Long(timeout * 1000));
或者
int timeout = 5; // seconds
HttpParams httpParams = httpClient.getParams();
HttpConnectionParams.setConnectionTimeout(httpParams, timeout * 1000); // http.connection.timeout
HttpConnectionParams.setSoTimeout(httpParams, timeout * 1000); // http.socket.timeout
4.3及以上
RequestConfig config = RequestConfig.custom()
 .setConnectTimeout(timeout * 1000)
 .setConnectionRequestTimeout(timeout * 1000)
 .setSocketTimeout(timeout * 1000).build();
CloseableHttpClient client =
HttpClientBuilder.create().setDefaultRequestConfig(config).build();
超时强制关闭
HttpGet getMethod = new HttpGet("http://localhost:8080/spring-security-rest-template/api/bars/1");
int hardTimeout = 5; // seconds
TimerTask task = new TimerTask() {
 @Override
 public void run() {
 if (getMethod != null) {
 getMethod.abort();
 }
 }
};
new Timer(true).schedule(task, hardTimeout * 1000);
HttpResponse response = httpClient.execute(getMethod);
System.out.println("HTTP Status of response: " + response.getStatusLine().getStatusCode());
连接池设置
4.3及以上
PoolingHttpClientConnectionManager connManager = new PoolingHttpClientConnectionManager();
connManager.setMaxTotal(100);
connManager.setDefaultMaxPerRoute(20);
httpClient = HttpClientBuilder.create().setConnectionManager(connManager).build();
```
## 三、设置header
```
4.3以下
HttpClient client = new DefaultHttpClient();
HttpGet request = new HttpGet(SAMPLE_URL);
request.setHeader(HttpHeaders.CONTENT_TYPE, "application/json");
client.execute(request);
4.3及以上
HttpClient client = HttpClients.custom().build();
HttpUriRequest request = RequestBuilder.get().setUri(SAMPLE_URL)
 .setHeader(HttpHeaders.CONTENT_TYPE, "application/json").build();
client.execute(request);
4.3及以上设置默认header
Header header = new BasicHeader(HttpHeaders.CONTENT_TYPE, "application/json");
List headers = Lists.newArrayList(header);
HttpClient client = HttpClients.custom().setDefaultHeaders(headers).build();
HttpUriRequest request = RequestBuilder.get().setUri(SAMPLE_URL).build();
client.execute(request);
```
## 四、设置参数
``` 
List params = new ArrayList();
params.add(new BasicNameValuePair("key1", "value1"));
params.add(new BasicNameValuePair("key2", "value2"));
request.setEntity(new UrlEncodedFormEntity(params, Consts.UTF_8));
//get 设置方式
URIBuilder builder = new URIBuilder("http://example.com/");
builder.setParameter("parts", "all").setParameter("action", "finish");
HttpPost post = new HttpPost(builder.build());
```
## 五、Https设置
``` 
4.3以下
TrustStrategy acceptingTrustStrategy = (cert, authType) -> true;
SSLSocketFactory sf = new SSLSocketFactory(acceptingTrustStrategy,
 SSLSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER);
SchemeRegistry registry = new SchemeRegistry();
registry.register(new Scheme("https", 8443, sf));
ClientConnectionManager ccm = new PoolingClientConnectionManager(registry);
DefaultHttpClient httpClient = new DefaultHttpClient(ccm);
Spring RestTemplate设置
HttpComponentsClientHttpRequestFactory requestFactory =
 new HttpComponentsClientHttpRequestFactory();
DefaultHttpClient httpClient = (DefaultHttpClient) requestFactory.getHttpClient();
TrustStrategy acceptingTrustStrategy = (cert, authType) -> true
SSLSocketFactory sf = new SSLSocketFactory(
 acceptingTrustStrategy, ALLOW_ALL_HOSTNAME_VERIFIER);
httpClient.getConnectionManager().getSchemeRegistry()
 .register(new Scheme("https", 8443, sf));
String urlOverHttps = "https://localhost:8443/spring-security-rest-basic-auth/api/bars/1";
ResponseEntity response = new RestTemplate(requestFactory).
 exchange(urlOverHttps, HttpMethod.GET, null, String.class);
assertThat(response.getStatusCode().value(), equalTo(200));
4.3及以上
SSLContextBuilder builder = new SSLContextBuilder();
builder.loadTrustMaterial(null, (chain, authType) -> true);
SSLConnectionSocketFactory sslsf = new
SSLConnectionSocketFactory(builder.build(), NoopHostnameVerifier.INSTANCE);
CloseableHttpClient httpClient = HttpClients.custom().setSSLSocketFactory(sslsf).build();
Spring RestTemplate设置
CloseableHttpClient httpClient
 = HttpClients.custom()
 .setSSLHostnameVerifier(new NoopHostnameVerifier())
 .build();
HttpComponentsClientHttpRequestFactory requestFactory
 = new HttpComponentsClientHttpRequestFactory();
requestFactory.setHttpClient(httpClient);
ResponseEntity response
 = new RestTemplate(requestFactory).exchange(
 urlOverHttps, HttpMethod.GET, null, String.class);
assertThat(response.getStatusCode().value(), equalTo(200));
```
## 六、cookie设置
``` 
4.3以下
BasicCookieStore cookieStore = new BasicCookieStore();
BasicClientCookie cookie = new BasicClientCookie("JSESSIONID", "1234");
cookie.setDomain(".github.com");
cookie.setPath("/");
cookieStore.addCookie(cookie);
DefaultHttpClient client = new DefaultHttpClient();
client.setCookieStore(cookieStore);
4.3及以上
BasicCookieStore cookieStore = new BasicCookieStore();
BasicClientCookie cookie = new BasicClientCookie("JSESSIONID", "1234");
cookie.setDomain(".github.com");
cookie.setPath("/");
cookieStore.addCookie(cookie);
HttpClient client = HttpClientBuilder.create().setDefaultCookieStore(cookieStore).build();
在Request上设置cookie
BasicCookieStore cookieStore = new BasicCookieStore();
BasicClientCookie cookie = new BasicClientCookie("JSESSIONID", "1234");
cookie.setDomain(".github.com");
cookie.setPath("/");
cookieStore.addCookie(cookie);
instance = HttpClientBuilder.create().build();
HttpGet request = new HttpGet("http://www.github.com");
HttpContext localContext = new BasicHttpContext();
localContext.setAttribute(HttpClientContext.COOKIE_STORE, cookieStore);
// localContext.setAttribute(ClientContext.COOKIE_STORE, cookieStore); // before 4.3
response = instance.execute(request, localContext);
header方式设置cookie
instance = HttpClientBuilder.create().build();
HttpGet request = new HttpGet("http://www.github.com");
request.setHeader("Cookie", "JSESSIONID=1234");
response = instance.execute(request);
```
## 七、文件上传
``` 
1，上传两个String和一个文件
File file = new File(textFileName);
HttpPost post = new HttpPost("http://echo.200please.com");
FileBody fileBody = new FileBody(file, ContentType.DEFAULT_BINARY);
StringBody stringBody1 = new StringBody("Message 1", ContentType.MULTIPART_FORM_DATA);
StringBody stringBody2 = new StringBody("Message 2", ContentType.MULTIPART_FORM_DATA);
//
MultipartEntityBuilder builder = MultipartEntityBuilder.create();
builder.setMode(HttpMultipartMode.BROWSER_COMPATIBLE);
builder.addPart("upfile", fileBody);
builder.addPart("text1", stringBody1);
builder.addPart("text2", stringBody2);
HttpEntity entity = builder.build();
//
post.setEntity(entity);
HttpResponse response = client.execute(post);
2，上传String和文本文件
HttpPost post = new HttpPost("http://echo.200please.com");
File file = new File(textFileName);
String message = "This is a multipart post";
MultipartEntityBuilder builder = MultipartEntityBuilder.create();
builder.setMode(HttpMultipartMode.BROWSER_COMPATIBLE);
builder.addBinaryBody("upfile", file, ContentType.DEFAULT_BINARY, textFileName);
builder.addTextBody("text", message, ContentType.DEFAULT_BINARY);
// 
HttpEntity entity = builder.build();
post.setEntity(entity);
HttpResponse response = client.execute(post);
3，上传String和Zip文件和图片文件
HttpPost post = new HttpPost("http://echo.200please.com");
InputStream inputStream = new FileInputStream(zipFileName);
File file = new File(imageFileName);
String message = "This is a multipart post";
MultipartEntityBuilder builder = MultipartEntityBuilder.create();
builder.setMode(HttpMultipartMode.BROWSER_COMPATIBLE);
builder.addBinaryBody
 ("upfile", file, ContentType.DEFAULT_BINARY, imageFileName);
builder.addBinaryBody
 ("upstream", inputStream, ContentType.create("application/zip"), zipFileName);
builder.addTextBody("text", message, ContentType.TEXT_PLAIN);
// 
HttpEntity entity = builder.build();
post.setEntity(entity);
HttpResponse response = client.execute(post);
4，上传字节数组和String
HttpPost post = new HttpPost("http://echo.200please.com");
String message = "This is a multipart post";
byte[] bytes = "binary code".getBytes();
// 
MultipartEntityBuilder builder = MultipartEntityBuilder.create();
builder.setMode(HttpMultipartMode.BROWSER_COMPATIBLE);
builder.addBinaryBody("upfile", bytes, ContentType.DEFAULT_BINARY, textFileName);
builder.addTextBody("text", message, ContentType.TEXT_PLAIN);
// 
HttpEntity entity = builder.build();
post.setEntity(entity);
HttpResponse response = client.execute(post);
```
## 八、RetryHandler
``` 
HttpRequestRetryHandler requestRetryHandler = new HttpRequestRetryHandler() {
 public boolean retryRequest(IOException exception, int executionCount, HttpContext context) {
 if (executionCount >= retryTimes) {
 // do not retry if over max retry times
 return false;
 }
 if (exception instanceof ConnectTimeoutException) {
 // connection refused by peer
 return false;
 }
 if (exception instanceof SocketTimeoutException || exception instanceof InterruptedIOException) {
 // IO exception
 return true;
 }
 if (exception instanceof UnknownHostException) {
 return false;
 }
 if (exception instanceof SSLException) {
 // ssl handshake exception
 return false;
 }
 HttpClientContext clientContext = HttpClientContext.adapt(context);
 HttpRequest request = clientContext.getRequest();
 // idempotent
 return !(request instanceof HttpEntityEnclosingRequest);
 }
}；
return HttpClients.custom()
 .setConnectionManager(connectionManager)
 .setRetryHandler(requestRetryHandler)
 .build();
```