# 参考资料
+ [HttpClient教程](http://www.httpclient.cn/category/httpclient/)
+ [HttpClient常见问题汇总](http://www.httpclient.cn/archives/101.html)
+ [HttpClient 4.5.7 菜鸟入门教程](http://www.httpclient.cn/archives/61.html)

# HttpClient maven依赖
```
<dependency>
	<groupId>org.apache.httpcomponents</groupId>
	<artifactId>httpclient</artifactId>
	<version>4.5.7</version>
</dependency>```

# HttpClient 使用步骤
使用HttpClient发送请求、接收响应很简单，一般需要如下几步即可。
+ 创建HttpClient对象。
+ 创建请求方法的实例，并指定请求URL。如果需要发送GET请求，创建HttpGet对象；如果需要发送POST请求，创建HttpPost对象。
+ 设置请求参数。如果需要发送请求参数，可调用HttpGet、HttpPost共同的setParams(HetpParams params)方法来添加请求参数；对于HttpPost对象而言，也可调用setEntity(HttpEntity entity)方法来设置请求参数。
+ 发送请求。调用HttpClient对象的execute(HttpUriRequest request)发送请求，该方法返回一个HttpResponse。
+ 调用HttpResponse的getAllHeaders()、getHeaders(String name)等方法可获取服务器的响应头；调用HttpResponse的getEntity()方法可获取HttpEntity对象，该对象包装了服务器的响应内容。程序可通过该对象获取服务器的响应内容。
+ 释放连接。无论执行方法是否成功，都必须释放连接。

## 发送请求
### GET请求
```
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpGet = new HttpGet("http://targethost/homepage");
CloseableHttpResponse response = httpclient.execute(httpGet);
    
//建立的http连接，仍旧被response保持着，允许我们从网络socket中获取返回的数据
//为了释放资源，我们必须手动消耗掉response或者取消连接（使用CloseableHttpResponse类的close方法）
try 
{
    System.out.println(response.getStatusLine());
    HttpEntity entity = response.getEntity();
    EntityUtils.consume(entity);
} 
finally 
{
    response.close();
}

```

### POST请求
```
HttpPost httpPost = new HttpPost("http://targethost/login");
//拼接参数
List <NameValuePair> nvps = new ArrayList <NameValuePair>();
nvps.add(new BasicNameValuePair("username", "vip"));
nvps.add(new BasicNameValuePair("password", "secret"));
httpPost.setEntity(new UrlEncodedFormEntity(nvps));
CloseableHttpResponse response = httpclient.execute(httpPost);
try 
{
    System.out.println(response.getStatusLine());
    HttpEntity entity = response.getEntity();
    EntityUtils.consume(entity);
} 
finally 
{
    response.close();
}
```
