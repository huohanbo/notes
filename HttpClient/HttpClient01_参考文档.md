# 参考资料
+ [http://hc.apache.org/](http://hc.apache.org/)
+ [http://hc.apache.org/httpcomponents-client-4.5.x/index.html](http://hc.apache.org/httpcomponents-client-4.5.x/index.html)
+ [http://hc.apache.org/httpcomponents-client-4.5.x/tutorial/html/index.html](http://hc.apache.org/httpcomponents-client-4.5.x/tutorial/html/index.html)
+ [http://hc.apache.org/httpcomponents-client-4.5.x/tutorial/pdf/httpclient-tutorial.pdf](http://hc.apache.org/httpcomponents-client-4.5.x/tutorial/pdf/httpclient-tutorial.pdf)

# HttpClient概述
超文本传输​​协议（HTTP）可能是当今Internet上使用的最重要的协议。
Web服务，支持网络的设备和网络计算的增长继续将HTTP协议的作用扩展到用户驱动的Web浏览器之外，同时增加了需要HTTP支持的应用程序的数量。

尽管java.net软件包提供了用于通过HTTP访问资源的基本功能，但它并未提供许多应用程序所需的全部灵活性或功能。
HttpClient试图通过提供高效，最新且功能丰富的程序包来实现此空白，以实现最新HTTP标准和建议的客户端。

HttpClient是为扩展而设计的，同时提供了对基本HTTP协议的强大支持，对于构建HTTP感知的客户端应用程序（例如Web浏览器，Web服务客户端或利用或扩展HTTP协议进行分布式通信的系统）的任何人来说，HttpClient都可能会感兴趣。

# HttpClient特征
+ 基于标准的纯Java，HTTP版本1.0和1.1的实现
+ 在可扩展的OO框架中完全实现所有HTTP方法（GET，POST，PUT，DELETE，HEAD，OPTIONS和TRACE）。
+ 支持使用HTTPS（基于SSL的HTTP）协议进行加密。
+ 通过HTTP代理的透明连接。
+ 通过CONNECT方法通过HTTP代理建立的隧道HTTPS连接。
+ 基本，摘要，NTLMv1，NTLMv2，NTLM2会话，SNPNEGO，Kerberos身份验证方案。
+ 自定义身份验证方案的插件机制。
+ 可插拔的安全套接字工厂，使使用第三方解决方案更加容易
+ 连接管理支持在多线程应用程序中使用。支持设置最大总连接数以及每个主机的最大连接数。检测并关闭陈旧的连接。
+ 自动Cookie处理，用于从服务器读取Set-Cookie：标头，并在适当时在Cookie：标头中发回。
+ 自定义Cookie策略的插件机制。
+ 请求输出流，以避免通过直接流传输到服务器的套接字来缓冲任何内容主体。
+ 响应输入流通过直接从套接字流传输到服务器来有效读取响应主体。
+ 在HTTP / 1.0中使用KeepAlive的持久连接以及在HTTP / 1.1中的持久性
+ 直接访问服务器发送的响应代码和标头。
+ 设置连接超时的能力。
+ 支持HTTP / 1.1响应缓存。
+ 源代码可根据Apache许可免费获得。

# HttpClient快速入门
## 接入配置
HttpClient
```
<dependency>
  <groupId>org.apache.httpcomponents</groupId>
  <artifactId>httpclient</artifactId>
  <version>4.5.12</version>
</dependency>
```

Fluent HC
```
<dependency>
  <groupId>org.apache.httpcomponents</groupId>
  <artifactId>fluent-hc</artifactId>
  <version>4.5.12</version>
</dependency>
```

HttpClient 4.5 需要Java 1.5 或更高版本。

## 代码示例
```
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpGet = new HttpGet("http://targethost/homepage");
CloseableHttpResponse response1 = httpclient.execute(httpGet);
// Response仍保留基础HTTP连接，允许直接从网络套接字流式传输响应内容。
// 为了确保正确释放系统资源，用户必须从finally子句中调用CloseableHttpResponse＃close()。
// 请注意，如果响应内容没有完全消耗掉，无法安全地重用连接，连接将被连接管理器关闭并丢弃。
try {
    System.out.println(response1.getStatusLine());
    HttpEntity entity1 = response1.getEntity();
    // 对Response进行一些有用的操作，并确保将其完全消耗
    EntityUtils.consume(entity1);
} finally {
    response1.close();
}

HttpPost httpPost = new HttpPost("http://targethost/login");
List <NameValuePair> nvps = new ArrayList <NameValuePair>();
nvps.add(new BasicNameValuePair("username", "vip"));
nvps.add(new BasicNameValuePair("password", "secret"));
httpPost.setEntity(new UrlEncodedFormEntity(nvps));
CloseableHttpResponse response2 = httpclient.execute(httpPost);

try {
    System.out.println(response2.getStatusLine());
    HttpEntity entity2 = response2.getEntity();
    // 对Response进行一些有用的操作，并确保将其完全消耗
    EntityUtils.consume(entity2);
} finally {
    response2.close();
}
```

使用 fluent API 执行相同请求
```
// fluent API使用户不必手动处理系统的重新分配。在某些情况下，资源是以必须在内存中缓冲响应内容为代价的
Request.Get("http://targethost/homepage")
    .execute().returnContent();

Request.Post("http://targethost/login")
    .bodyForm(Form.form().add("username",  "vip").add("password",  "secret").build())
    .execute().returnContent();
```

# HttpClient教程
## 前言
超文本传输​​协议（HTTP）可能是当今Internet上使用的最重要的协议。Web服务，支持网络的设备和网络计算的增长继续将HTTP协议的作用扩展到用户驱动的Web浏览器之外，同时增加了需要HTTP支持的应用程序的数量。

尽管java.net软件包提供了用于通过HTTP访问资源的基本功能，但它并未提供许多应用程序所需的全部灵活性或功能。HttpClient试图通过提供高效，最新且功能丰富的程序包来实现此空白，以实现最新HTTP标准和建议的客户端。

HttpClient是为扩展而设计的，同时提供了对基本HTTP协议的强大支持，对于构建HTTP感知的客户端应用程序（例如Web浏览器，Web服务客户端或利用或扩展HTTP协议进行分布式通信的系统）的任何人来说，HttpClient都可能会感兴趣。

### HttpClient是什么

### HttpClient不是什么

## 基本使用
### 执行请求
HttpClient的最基本功能是执行HTTP方法。HTTP方法的执行涉及一个或多个HTTP请求/ HTTP响应交换，通常由HttpClient在内部处理。
期望用户提供要执行的请求对象，并且HttpClient希望将请求传输到目标服务器，以返回相应的响应对象，如果执行不成功，则抛出异常。

代码示列：
```
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://localhost/");
CloseableHttpResponse response = httpclient.execute(httpget);
try {
    <...>
} finally {
    response.close();
}
```

#### HTTP request
所有HTTP请求均包含请求方法，请求URI和HTTP协议版本。

HttpClient支持了在HTTP / 1.1规范中定义的所有HTTP方法：GET，HEAD， POST，PUT，DELETE， TRACE和OPTIONS。
存在用于每种方法类型的特定类：HttpGet， HttpHead，HttpPost， HttpPut，HttpDelete， HttpTrace，和HttpOptions。

Request-URI是一个统一资源标识符，用于标识要在其上应用请求的资源。
HTTP请求URI由协议方案，主机名，可选端口，资源路径，可选查询和可选片段组成。
```
HttpGet httpget = new HttpGet("http://www.google.com/search?hl=en&q=httpclient&btnG=Google+Search&aq=f&oq=");
```

HttpClient提供了URIBuilder实用程序类，以简化请求URI的创建和修改。
```
URI uri = new URIBuilder()
        .setScheme("http")
        .setHost("www.google.com")
        .setPath("/search")
        .setParameter("q", "httpclient")
        .setParameter("btnG", "Google Search")
        .setParameter("aq", "f")
        .setParameter("oq", "")
        .build();
HttpGet httpget = new HttpGet(uri);
System.out.println(httpget.getURI());
```
stdout >
```
http://www.google.com/search?q=httpclient&btnG=Google+Search&aq=f&oq=
```

#### HTTP response
HTTP响应是服务器在收到并解释了请求消息后发送回客户端的消息。
该消息包含协议版本，数字状态代码及其关联的文本短语。
```
HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1, HttpStatus.SC_OK, "OK");

System.out.println(response.getProtocolVersion());
System.out.println(response.getStatusLine().getStatusCode());
System.out.println(response.getStatusLine().getReasonPhrase());
System.out.println(response.getStatusLine().toString());
```
stdout >
```
HTTP/1.1
200
OK
HTTP/1.1 200 OK
```

#### HTTP message headers
HTTP消息可以包含许多描述消息属性的消息头，例如内容长度，内容类型等。
HttpClient提供了检索，添加，删除和枚举消息头的方法。
```
HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1, HttpStatus.SC_OK, "OK");
response.addHeader("Set-Cookie", "c1=a; path=/; domain=localhost");
response.addHeader("Set-Cookie", "c2=b; path=\"/\", c3=c; domain=\"localhost\"");

Header h1 = response.getFirstHeader("Set-Cookie");
System.out.println(h1);
Header h2 = response.getLastHeader("Set-Cookie");
System.out.println(h2);
Header[] hs = response.getHeaders("Set-Cookie");
System.out.println(hs.length);
```
stdout >
```
Set-Cookie: c1=a; path=/; domain=localhost
Set-Cookie: c2=b; path="/", c3=c; domain="localhost"
2
```

使用 HeaderIterator接口获取给定类型的所有消息头。
```
HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1, HttpStatus.SC_OK, "OK");
response.addHeader("Set-Cookie", "c1=a; path=/; domain=localhost");
response.addHeader("Set-Cookie", "c2=b; path=\"/\", c3=c; domain=\"localhost\"");

HeaderIterator it = response.headerIterator("Set-Cookie");
while (it.hasNext()) {
    System.out.println(it.next());
}
```
stdout >
```
Set-Cookie: c1=a; path=/; domain=localhost
Set-Cookie: c2=b; path="/", c3=c; domain="localhost"
```

将HTTP消息头解析为单独的消息头元素。
```
HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1, HttpStatus.SC_OK, "OK");
response.addHeader("Set-Cookie", "c1=a; path=/; domain=localhost");
response.addHeader("Set-Cookie", "c2=b; path=\"/\", c3=c; domain=\"localhost\"");

HeaderElementIterator it = new BasicHeaderElementIterator(response.headerIterator("Set-Cookie"));
while (it.hasNext()) {
    HeaderElement elem = it.nextElement(); 
    System.out.println(elem.getName() + " = " + elem.getValue());
    NameValuePair[] params = elem.getParameters();
    for (int i = 0; i < params.length; i++) {
        System.out.println(" " + params[i]);
    }
}
```
stdout >
```
c1 = a
path=/
domain=localhost
c2 = b
path=/
c3 = c
domain=localhost
```

#### HTTP entity
HTTP消息可以包含与请求或响应关联的内容实体。
实体可以在某些请求和响应中找到，因为它们是可选的。使用实体的请求称为实体封装请求。

HTTP规范定义了两个包含请求实体的方法：POST和 PUT。
通常期望响应包含内容实体。有例外的情况，如应对 HEAD方法204 No Content， 304 Not Modified，205 Reset Content 响应。

HttpClient根据它们的内容起源来区分三种实体：
+ 流式：从流中接收内容，或即时生成内容。特别地，此类别包括从HTTP响应接收的实体。**流式实体通常不可重复**。
+ 自包含的：内容在内存中或通过独立于连接或其他实体的方式获取。自包含实体通常是可重复的。这种类型的实体将主要用于封装HTTP请求的实体。
+ 包装：内容是从另一个实体获得的。

当从HTTP响应中传输内容时，此区别对于连接管理很重要。
对于由应用程序创建且仅使用HttpClient发送的请求实体，流式传输和自包含式之间的区别并不重要。
在这种情况下，建议将不可重复的实体视为流，将可重复的实体视为独立的。

##### 可重复实体
一个实体可以是可重复的，这意味着其内容可以被读取多次。这仅适用于自包含的实体（如 ByteArrayEntity或 StringEntity）

##### 使用HTTP实体
由于实体既可以表示二进制内容又可以表示字符内容，因此它支持字符编码（以支持后者，即字符内容）。

当执行带有封闭内容的请求时，或者当请求成功时，将创建实体，并使用响应主体将结果发送回客户端。

要从实体中读取内容，可以通过HttpEntity#getContent()方法检索输入流，该方法返回java.io.InputStream，
或者可以向该HttpEntity#writeTo(OutputStream)方法提供输出流，该方法将所有内容都写入指定流后返回。

当实体已经被传入消息接收，HttpEntity#getContentType() 和 HttpEntity#getContentLength() 方法可用于读取所述公共元数据，例如Content-Type 与 Content-Length报头（如果可用）。
由于 Content-Type标头可以包含针对文本mime类型（例如text / plain或text / html）的字符编码，因此该 HttpEntity#getContentEncoding()方法用于读取此信息。
如果标题不可用，则将返回-1的长度，并且内容类型为NULL。如果Content-Type 标题可用，Header则将返回一个对象。

在为外发消息创建实体时，此元数据必须由实体的创建者提供。
```
StringEntity myEntity = new StringEntity("important message", ContentType.create("text/plain", "UTF-8"));
System.out.println(myEntity.getContentType());
System.out.println(myEntity.getContentLength());
System.out.println(EntityUtils.toString(myEntity));
System.out.println(EntityUtils.toByteArray(myEntity).length);
```
stdout >
```
Content-Type: text/plain; charset=utf-8
17
important message
17
```

#### 确保释放资源
为了确保正确释放系统资源，必须关闭与实体关联的内容流或响应本身。
```
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://localhost/");
CloseableHttpResponse response = httpclient.execute(httpget);
try {
    HttpEntity entity = response.getEntity();
    if (entity != null) {
        InputStream instream = entity.getContent();
        try {
            // do something useful
        } finally {
            instream.close();
        }
    }
} finally {
    response.close();
}
```
关闭内容流和关闭响应之间的区别在于，**前者将通过消耗实体内容来尝试使基础连接保持活动状态，而后者将立即关闭并丢弃该连接**。

请注意，HttpEntity#writeTo(OutputStream) 一旦实体被完全写出，还需要该方法来确保适当释放系统资源。
通过调用 HttpEntity#getContent() 获得的实例 java.io.InputStream，则还应在finally子句中关闭流。
使用流式实体时，可以使用该 EntityUtils#consume(HttpEntity)方法来确保实体内容已被完全消耗，并且基础流已关闭。

在某些情况下，仅需要检索整个响应内容的一小部分，并且消耗剩余内容使连接可重用的性能损失过高，在这种情况下，可以通过关闭响应来终止内容流。
```
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://localhost/");
CloseableHttpResponse response = httpclient.execute(httpget);
try {
    HttpEntity entity = response.getEntity();
    if (entity != null) {
        InputStream instream = entity.getContent();
        int byteOne = instream.read();
        int byteTwo = instream.read();
        // Do not need the rest
    }
} finally {
    response.close();
}
```
连接将不会被重用，但是它所拥有的所有级别资源都将被正确地释放。

#### 消费 entity
推荐使用消费实体内容的方法是使用其 HttpEntity#getContent()或 HttpEntity#writeTo(OutputStream)方法。
也可以使用EntityUtils类，该类公开了几个静态方法，可以更轻松地从实体中读取内容或信息。
通过java.io.InputStream直接使用此类的方法，可以读取字符串/字节数组中的整个内容主体，而不必直接读取该内容。
但是，强烈建议不要使用，除非响应实体源自受信任的HTTP服务器并且已知长度有限。
```
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://localhost/");
CloseableHttpResponse response = httpclient.execute(httpget);
try {
    HttpEntity entity = response.getEntity();
    if (entity != null) {
        long len = entity.getContentLength();
        if (len != -1 && len < 2048) {
            System.out.println(EntityUtils.toString(entity));
        } else {
            // Stream content out
        }
    }
} finally {
    response.close();
}
```
	
在某些情况下，可能需要多次读取实体内容。在这种情况下，实体内容必须以某种方式缓冲在内存或磁盘中。
最简单的方法是用 类BufferedHttpEntity 包装原始实体。这将导致原始实体的内容被读入内存缓冲区中。
```
CloseableHttpResponse response = <...>
HttpEntity entity = response.getEntity();
if (entity != null) {
    entity = new BufferedHttpEntity(entity);
}
```

#### 生产 entity
HttpClient提供了几个类，可用于通过HTTP连接有效地流式传输内容。
这些类的实例可以与诸如POST和PUT的实体封装请求相关联，以便将实体内容封装到传出的HTTP请求中。

HttpClient的提供了几个类为最常见的数据的容器，如字符串，字节数组，输入流，和文件：StringEntity， ByteArrayEntity， InputStreamEntity，和 FileEntity。
```
File file = new File("somefile.txt");
FileEntity entity = new FileEntity(file, ContentType.create("text/plain", "UTF-8"));        
HttpPost httppost = new HttpPost("http://localhost/action.do");
httppost.setEntity(entity);
```
请注意InputStreamEntity不可重复，因为它只能从基础数据流读取一次。
通常，建议实现一个自HttpEntity包含的自定义类，而不是使用generic InputStreamEntity。 FileEntity可以是一个很好的起点。

##### HTML forms
许多应用程序需要模拟提交HTML表单的过程，以便登录Web应用程序或提交输入数据。HttpClient提供了实体类 UrlEncodedFormEntity来简化该过程。
```
List<NameValuePair> formparams = new ArrayList<NameValuePair>();
formparams.add(new BasicNameValuePair("param1", "value1"));
formparams.add(new BasicNameValuePair("param2", "value2"));
UrlEncodedFormEntity entity = new UrlEncodedFormEntity(formparams, Consts.UTF_8);
HttpPost httppost = new HttpPost("http://localhost/handler.do");
httppost.setEntity(entity);
```
UrlEncodedFormEntity实例将使用所谓的URL编码来编码参数并产生以下内容：
```
param1=value1&param2=value2
```

##### Content编码
通常，建议让HttpClient根据要传输的HTTP消息的属性选择最合适的传输编码。
但是，可以通过设置HttpEntity#setChunked()为true 来通知HttpClient首选块编码。
请注意，HttpClient仅将此标志用作提示。当使用不支持块编码的HTTP协议版本（例如HTTP / 1.0）时，将忽略此值。
```
StringEntity entity = new StringEntity("important message", ContentType.create("plain/text", Consts.UTF_8));
entity.setChunked(true);
HttpPost httppost = new HttpPost("http://localhost/acrtion.do");
httppost.setEntity(entity);
```

#### 响应处理 Response handlers
处理响应的最简单，最方便的方法是使用ResponseHandler包含handleResponse(HttpResponse response)方法的接口。
这种方法完全使用户不必担心连接管理。
使用ResponseHandler 时，无论请求执行成功还是引起异常，HttpClient都会自动确保将连接释放回连接管理器。
```
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://localhost/json");
ResponseHandler<MyJsonObject> rh = new ResponseHandler<MyJsonObject>() {
    @Override
    public JsonObject handleResponse(final HttpResponse response) throws IOException {
        StatusLine statusLine = response.getStatusLine();
        HttpEntity entity = response.getEntity();
        if (statusLine.getStatusCode() >= 300) {
            throw new HttpResponseException(statusLine.getStatusCode(), statusLine.getReasonPhrase());
        }
        if (entity == null) {
            throw new ClientProtocolException("Response contains no content");
        }
        Gson gson = new GsonBuilder().create();
        ContentType contentType = ContentType.getOrDefault(entity);
        Charset charset = contentType.getCharset();
        Reader reader = new InputStreamReader(entity.getContent(), charset);
        return gson.fromJson(reader, MyJsonObject.class);
    }
};

MyJsonObject myjson = client.execute(httpget, rh);
```

### HttpClient接口
HttpClient接口 是HTTP请求执行的最基本实现。
它不对请求执行过程施加任何限制或特定细节，并将连接管理，状态管理，身份验证和重定向处理的细节留给各个实现。
这应该使附加功能（例如响应内容缓存）装饰界面更加容易。

通常，HttpClient实现充当许多专用处理程序或策略接口实现的立面，这些处理程序或策略接口实现负责处理HTTP协议的特定方面，
例如重定向或身份验证处理或做出有关连接持久性和保持生存期的决策。
这使用户可以用定制的，特定于应用程序的方面有选择地替换那些方面的默认实现。
```
ConnectionKeepAliveStrategy keepAliveStrat = new DefaultConnectionKeepAliveStrategy() {

    @Override
    public long getKeepAliveDuration(
            HttpResponse response,
            HttpContext context) {
        long keepAlive = super.getKeepAliveDuration(response, context);
        if (keepAlive == -1) {
            // Keep connections alive 5 seconds if a keep-alive value
            // has not be explicitly set by the server
            keepAlive = 5000;
        }
        return keepAlive;
    }

};

CloseableHttpClient httpclient = HttpClients.custom()
        .setKeepAliveStrategy(keepAliveStrat)
        .build();
```

#### HttpClient线程安全
HttpClient实现是线程安全的。**建议将此类的同一实例重用于多个请求执行。**

#### HttpClient资源释放
当一个CloseableHttpClient实例不再需要并且即将超出范围时，必须通过调用该CloseableHttpClient#close()方法，关闭与该实例关联的连接管理器。
```
CloseableHttpClient httpclient = HttpClients.createDefault();
try {
    <...>
} finally {
    httpclient.close();
}
```

### HTTP执行上下文!!!
最初，HTTP被设计为无状态，面向响应请求的协议。
但是，现实世界中的应用程序通常需要能够通过几次逻辑相关的请求-响应交换来保留状态信息。
为了使应用程序能够保持处理状态，HttpClient允许HTTP请求在称为HTTP上下文的特定执行上下文中执行。
如果在连续请求之间重用同一上下文，则多个逻辑相关的请求可以参与逻辑会话。

HTTP上下文的功能类似于 java.util.Map<String, Object>。它是任意命名值的集合。
应用程序可以在请求执行之前填充上下文属性，或者在执行完成之后检查上下文。

**HttpContext可以包含任意对象，因此在多个线程之间共享可能不安全。建议每个执行线程都维护自己的上下文。**

在HTTP请求执行过程中，HttpClient将以下属性添加到执行上下文：
+ HttpConnection 实例，表示与目标服务器的实际连接。
+ HttpHost 代表连接目标的实例。
+ HttpRoute 代表完整连接路径的实例
+ HttpRequest代表实际HTTP请求的实例。在执行上下文的最后HttpRequest对象始终表示消息的状态准确 ，因为它被发送到目标服务器。默认情况下，HTTP / 1.0和HTTP / 1.1使用相对请求URI。但是，如果请求是通过代理以非隧道模式发送的，则URI将是绝对的。
+ HttpResponse 代表实际HTTP响应的实例。
+ java.lang.Boolean 对象，表示指示实际请求是否已完全发送到连接目标的标志。
+ RequestConfig 代表实际请求配置的对象。
+ java.util.List<URI> 代表在请求执行过程中收到的所有重定向位置的集合的对象。

可以使用HttpClientContext适配器类来简化与上下文状态的干扰。
```
HttpContext context = <...>
HttpClientContext clientContext = HttpClientContext.adapt(context);
HttpHost target = clientContext.getTargetHost();
HttpRequest request = clientContext.getRequest();
HttpResponse response = clientContext.getResponse();
RequestConfig config = clientContext.getRequestConfig();
```

代表逻辑相关会话的多个请求序列应使用同一HttpContext实例执行，以确保请求之间会话上下文和状态信息的自动传播。
在以下示例中，由初始请求设置的请求配置将保留在执行上下文中，并传播到共享相同上下文的连续请求中。
```
CloseableHttpClient httpclient = HttpClients.createDefault();
RequestConfig requestConfig = RequestConfig.custom()
        .setSocketTimeout(1000)
        .setConnectTimeout(1000)
        .build();

HttpGet httpget1 = new HttpGet("http://localhost/1");
httpget1.setConfig(requestConfig);
CloseableHttpResponse response1 = httpclient.execute(httpget1, context);
try {
    HttpEntity entity1 = response1.getEntity();
} finally {
    response1.close();
}
HttpGet httpget2 = new HttpGet("http://localhost/2");
CloseableHttpResponse response2 = httpclient.execute(httpget2, context);
try {
    HttpEntity entity2 = response2.getEntity();
} finally {
    response2.close();
}
```

### HTTP协议拦截器!!!
HTTP协议拦截器是一个实现HTTP协议特定方面的例程。
通常，协议拦截器应作用于传入消息的一个特定标头或一组相关标头，或使用一个特定标头或一组相关标头填充输出消息。
一个很好的例子就是协议拦截器还可以操纵包含在消息中的内容实体-透明的内容压缩/解压缩。
通常，这是通过使用“装饰器”模式完成的，其中使用包装实体类来装饰原始实体。
几个协议拦截器可以组合在一起形成一个逻辑单元。

协议拦截器可以通过HTTP执行上下文共享信息（例如处理状态）进行协作。
协议拦截器可以使用HTTP上下文存储一个请求或几个连续请求的处理状态。

通常，拦截器的执行顺序无关紧要，只要它们不依赖于执行上下文的特定状态即可。
如果协议拦截器具有相互依赖性，因此必须按特定的顺序执行，则应按照与它们预期的执行顺序相同的顺序将它们添加到协议处理器。

协议拦截器必须实现为线程安全的。与Servlet相似，协议拦截器不应使用实例变量，除非对这些变量的访问已同步。

这是一个示例，该示例说明了如何使用本地上下文在连续的请求之间保留处理状态：
```
CloseableHttpClient httpclient = HttpClients.custom()
        .addInterceptorLast(new HttpRequestInterceptor() {

            public void process(
                    final HttpRequest request,
                    final HttpContext context) throws HttpException, IOException {
                AtomicInteger count = (AtomicInteger) context.getAttribute("count");
                request.addHeader("Count", Integer.toString(count.getAndIncrement()));
            }

        })
        .build();

AtomicInteger count = new AtomicInteger(1);
HttpClientContext localContext = HttpClientContext.create();
localContext.setAttribute("count", count);

HttpGet httpget = new HttpGet("http://localhost/");
for (int i = 0; i < 10; i++) {
    CloseableHttpResponse response = httpclient.execute(httpget, localContext);
    try {
        HttpEntity entity = response.getEntity();
    } finally {
        response.close();
    }
}
```

### 异常处理!!!
HTTP协议处理器会引发两种类型的异常： 
java.io.IOException，IO异常，例如套接字超时或套接字重置。
HttpException，HTTP协议错误，例如违反HTTP协议。
通常，IO异常被认为是**非致命错误且可恢复**，而HTTP协议错误被认为是**致命错误且无法恢复**。

请注意，HttpClient 实现将重新抛出HttpException为ClientProtocolException，它是java.io.IOException的子类。
这样，的用户就HttpClient可以从单个catch子句中处理 IOException 和 HttpException。

#### HTTP传输安全
要知道，HTTP协议并不适合所有类型的应用程序。
HTTP是一种简单的面向请求/响应的协议，最初旨在支持静态或动态生成的内容检索。它从未打算支持事务操作。

例如，如果HTTP服务器成功接收和处理请求，生成响应并将状态代码发送回客户端，则它将认为合同的一部分已完成。
如果客户端由于读取超时，请求取消或系统崩溃而无法完整接收响应，则服务器将不尝试回滚事务。
如果客户端决定重试相同的请求，则服务器将不可避免地结束多次执行同一事务。
在某些情况下，这可能导致应用程序数据损坏或应用程序状态不一致。

即使HTTP从未被设计为支持事务处理，但只要满足特定条件，它仍可以用作关键任务应用程序的传输协议。
为了确保HTTP传输层的安全，系统必须确保HTTP方法在应用程序层上的幂等性。

#### 幂等方法
HTTP / 1.1规范将幂等方法定义为，多个相同请求（因错误或过期问题除外）的副作用与单个请求相同。
换句话说，应用程序应确保准备好应对同一方法的多次执行所带来的影响。
例如，这可以通过提供唯一的事务ID以及通过其他避免执行相同逻辑操作的方式来实现。

请注意，此问题并非特定于HttpClient。基于浏览器的应用程序会遇到与HTTP方法非幂等性完全相同的问题。

默认情况下，HttpClient假定非实体封闭方法（例如 GET和HEAD）是幂等的，而出于兼容性原因实体封闭方法（例如POST和PUT）是非幂等的。

#### 异常自动恢复
默认情况下，HttpClient尝试自动从**IO异常**中恢复。默认的自动恢复机制仅限于一些已知安全的异常情况。
+ HttpClient不会尝试从任何逻辑或HTTP协议错误（从HttpException类派生的错误）中恢复 。
+ HttpClient将自动重试那些被认为是幂等的方法。
+ 当HTTP请求仍被传输到目标服务器时（即请求尚未完全传输到服务器），HttpClient将自动重试那些由于传输异常而失败的方法。

#### 请求重试处理
为了启用**自定义异常**恢复机制，可以使用实现HttpRequestRetryHandler接口。
```
HttpRequestRetryHandler myRetryHandler = new HttpRequestRetryHandler() {
    public boolean retryRequest(
            IOException exception,
            int executionCount,
            HttpContext context) {
        if (executionCount >= 5) {
            // Do not retry if over max retry count
            return false;
        }
        if (exception instanceof InterruptedIOException) {
            // Timeout
            return false;
        }
        if (exception instanceof UnknownHostException) {
            // Unknown host
            return false;
        }
        if (exception instanceof ConnectTimeoutException) {
            // Connection refused
            return false;
        }
        if (exception instanceof SSLException) {
            // SSL handshake exception
            return false;
        }
        HttpClientContext clientContext = HttpClientContext.adapt(context);
        HttpRequest request = clientContext.getRequest();
        boolean idempotent = !(request instanceof HttpEntityEnclosingRequest);
        if (idempotent) {
            // Retry if the request is considered idempotent
            return true;
        }
        return false;
    }

};

CloseableHttpClient httpclient = HttpClients.custom()
        .setRetryHandler(myRetryHandler)
        .build();
```
可以使用StandardHttpRequestRetryHandler 替代默认的，用来自动重试 这些幂等请求方法：GET， HEAD，PUT，DELETE， OPTIONS，和TRACE。

### 中止请求
在某些情况下，由于目标服务器上的高负载或客户端发出的并发请求太多，HTTP请求执行无法在预期的时间内完成。
在这种情况下，可能有必要提前终止请求并取消阻塞在IO操作中阻塞的执行线程。

我们可以在HttpClient执行的任何阶段，通过调用**HttpUriRequest#abort()**方法中止HTTP请求。此方法是线程安全的，可以从任何线程调用。

当HTTP请求中止时，其执行线程（即使当前在I / O操作中被阻止）会通过抛出 InterruptedIOException 来解除锁定。

### 重定向处理
HttpClient自动处理所有类型的重定向，但HTTP规范明确需要用户干预的重定向除外。
按照HTTP规范的要求，POS请求和PUT请求的 See Other 重定向（状态代码303），会转换为GET请求。

可以使用自定义重定向策略来放宽对HTTP规范施加的POST方法自动重定向的限制。
```
LaxRedirectStrategy redirectStrategy = new LaxRedirectStrategy();
CloseableHttpClient httpclient = HttpClients.custom()
        .setRedirectStrategy(redirectStrategy)
        .build();
```

HttpClient通常必须在执行过程中重写请求消息。
默认情况下，HTTP / 1.0和HTTP / 1.1通常使用相对请求URI。
同样，原始请求可能会多次从位置重定向到另一个位置。可以使用原始请求和上下文构建最终解释的绝对HTTP位置。

URIUtils#resolve可以使用Utility方法来构建用于生成最终请求的解释后的绝对URI。此方法包括重定向请求或原始请求中的最后一个片段标识符。
```
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpClientContext context = HttpClientContext.create();
HttpGet httpget = new HttpGet("http://localhost:8080/");
CloseableHttpResponse response = httpclient.execute(httpget, context);
try {
    HttpHost target = context.getTargetHost();
    List<URI> redirectLocations = context.getRedirectLocations();
    URI location = URIUtils.resolve(httpget.getURI(), target, redirectLocations);
    System.out.println("Final HTTP location: " + location.toASCIIString());
    // Expected to be an absolute URI
} finally {
    response.close();
}
```

## 连接管理
### 连接持久性
建立从一台主机到另一台主机的连接的过程非常复杂，并且涉及两个端点之间的多次数据包交换，这可能会非常耗时。
连接握手的开销很大，尤其是对于小型HTTP消息而言。
如果可以将打开的连接重新用于执行多个请求，则可以获得更高的数据吞吐量。

HTTP / 1.1，默认情况下，HTTP连接可以重复用于多个请求。
HTTP / 1.0，可以使用一种机制来显式地传达其首选项，以使连接保持活动状态并用于多个请求。
HTTP代理还可以在一段时间内保持空闲连接处于活动状态，以防后续请求需要连接到同一目标主机。

**使连接保持活动的能力通常称为连接持久性。HttpClient完全支持连接持久性。**

### HTTP连接路由
HttpClient能够直接或通过可能涉及多个中间连接的路由建立与目标主机的连接。
HttpClient将路由的连接区分为**普通，隧道和分层**。使用多个中间代理来隧道连接到目标主机的过程称为代理链接。

通过连接到目标或第一个也是唯一的代理来建立普通路由。
通过连接到第一个隧道并通过代理链到目标隧道来建立隧道路径。没有代理的路由无法建立隧道。
通过在现有连接上对协议进行分层来建立分层路由。协议只能在通向目标的隧道上或在没有代理的直接连接上分层。

#### 路线计算
RouteInfo接口表示有关到目标主机的确定路由的信息，涉及到一个或多个中间步骤或跃点。
HttpRoute是的具体实现，RouteInfo不能更改（不可变）。

HttpTracker是RouteInfoHttpClient内部使用的可变 实现，用于跟踪到最终路由目标的剩余跃点。
HttpTracker在成功执行朝向路由目标的下一跳之后，可以更新。

HttpRouteDirector 是一个辅助类，可用于计算路线的下一步。此类由HttpClient内部使用。

HttpRoutePlanner是一个接口，表示基于执行上下文计算到给定目标的完整路由的策略。HttpClient 有两个HttpRoutePlanner实现。
SystemDefaultRoutePlanner，基于java.net.ProxySelector。默认情况下，它将从系统属性或运行应用程序的浏览器中获取JVM的代理设置。
DefaultProxyRoutePlanner，不使用任何Java系统属性实现，也不使用任何系统或浏览器代理设置。它总是通过相同的默认代理来计算路由。

#### 安全的HTTP连接
如果未经授权的第三方无法读取或篡改两个连接端点之间传输的信息，则认为HTTP连接是安全的。
SSL / TLS协议是确保HTTP传输安全性的最广泛使用的技术。但是，也可以采用其他加密技术。

通常，HTTP传输位于SSL / TLS加密连接上。

### HTTP连接管理器
#### 连接托管和连接管理器
HTTP连接是复杂的，有状态的，线程不安全的对象，需要对其进行适当的管理以使其正常运行。
HTTP连接一次只能由一个执行线程使用。

HttpClient使用一个特殊的实体来管理对HTTP连接的访问​​，该实体称为HTTP连接管理器，由HttpClientConnectionManager接口实现 。
HTTP连接管理器的目的是充当新HTTP连接的工厂，以管理持久性连接的生命周期并同步对持久性连接的访问​​，以**确保一次只有一个线程可以访问连接**。

内部HTTP连接管理器使用以下实例 ManagedHttpClientConnection 充当管理连接状态并控制I / O操作执行的真实连接的代理。
如果托管连接被其使用者释放或显式关闭，则基础连接将从其代理分离，并返回给管理器。
即使服务使用者仍然拥有对代理实例的引用，它也不再能够有意或无意地执行任何I / O操作或更改实际连接的状态。

以下从连接管理器获取连接的示例：
```
HttpClientContext context = HttpClientContext.create();
HttpClientConnectionManager connMrg = new BasicHttpClientConnectionManager();
HttpRoute route = new HttpRoute(new HttpHost("localhost", 80));
// 请求新的连接。这可能是一个漫长的过程
ConnectionRequest connRequest = connMrg.requestConnection(route, null);
// 等待连接最多10秒
HttpClientConnection conn = connRequest.get(10, TimeUnit.SECONDS);
try {
    // 如果未打开
    if (!conn.isOpen()) {
        // 根据其路由信息建立连接
        connMrg.connect(conn, route, 1000, context);
        // 并将其标记为路由完成
        connMrg.routeComplete(conn, route, context);
    }
    // 使用连接做一些事情
} finally {
    connMrg.releaseConnection(conn, null, 1, TimeUnit.MINUTES);
}
```
如有必要，可以通过调用 ConnectionRequest#cancel() 提前终止连接请求 。这将取消 ConnectionRequest#get() 方法中阻塞的线程。

#### 简单连接管理器
BasicHttpClientConnectionManager是一个简单的连接管理器，一次仅维护一个连接。
即使此类是线程安全的，也应仅由一个执行线程使用。

BasicHttpClientConnectionManager会将连接重新用于具有相同路由的后续请求。
但是，如果连接的路由与连接请求的路由不匹配，它将关闭现有连接并针对给定的路由重新打开它。
如果已经分配了连接，则 java.lang.IllegalStateException抛出该连接。

此连接管理器实现应在EJB容器内使用。

#### 池连接管理器
PoolingHttpClientConnectionManager是一种更复杂的实现，它管理客户端连接池并能够为来自多个执行线程的连接请求提供服务。
连接是基于每个路由池的。通过在池中租用连接而不是创建全新的连接，可以为管理器在池中已经具有持久连接的路由请求提供服务。

PoolingHttpClientConnectionManager在每个路由上总共维护最大连接数限制。
默认情况下，此实现将为每个给定路由**创建不超过2个并发连接，并且总共不超过20个连接**。
对于许多现实应用而言，这些限制可能过于严格，特别是如果它们使用HTTP作为其服务的传输协议。

以下示例显示如何调整连接池参数：
```
PoolingHttpClientConnectionManager cm = new PoolingHttpClientConnectionManager();
// 将最大总连接数增加到200
cm.setMaxTotal(200);
// 将每个路由的默认最大连接数增加到20
cm.setDefaultMaxPerRoute(20);
// 将特定路由（localhost:80）的最大连接数增加到50
HttpHost localhost = new HttpHost("locahost", 80);
cm.setMaxPerRoute(new HttpRoute(localhost), 50);
CloseableHttpClient httpClient = HttpClients.custom()
        .setConnectionManager(cm)
        .build();
```

#### 连接管理器关闭
当HttpClient实例不再需要时，关闭其连接管理器很重要。
确保关闭管理器保持活动的所有连接，以及释放由这些连接分配的系统资源。
```
CloseableHttpClient httpClient = <...>
httpClient.close();
```

### 多线程请求执行!!!
如果配备了池连接管理器（例如PoolingClientConnectionManager），则HttpClient可用于使用多个执行线程同时执行多个请求。

PoolingClientConnectionManager将根据其配置分配连接。
如果给定路由的所有连接都已经租用，则对连接的请求将阻塞，直到将连接释放回池中为止。

通过设置'http.conn-manager.timeout' 为正值，可以确保连接管理器不会无限期地阻塞连接请求操作。
如果在给定的时间段内无法满足连接请求，则会抛出ConnectionPoolTimeoutException错误。

**尽管HttpClient实例是线程安全的，并且可以在多个执行线程之间共享，但强烈建议每个线程都维护自己的专用实例HttpContext。**
```
PoolingHttpClientConnectionManager cm = new PoolingHttpClientConnectionManager();
CloseableHttpClient httpClient = HttpClients.custom()
        .setConnectionManager(cm)
        .build();

// 用于执行GET的URI
String[] urisToGet = {
    "http://www.domain1.com/",
    "http://www.domain2.com/",
    "http://www.domain3.com/",
    "http://www.domain4.com/"
};

// 为每个URI创建一个线程
GetThread[] threads = new GetThread[urisToGet.length];
for (int i = 0; i < threads.length; i++) {
    HttpGet httpget = new HttpGet(urisToGet[i]);
    threads[i] = new GetThread(httpClient, httpget);
}

// 启动线程
for (int j = 0; j < threads.length; j++) {
    threads[j].start();
}

// 加入线程
for (int j = 0; j < threads.length; j++) {
    threads[j].join();
}
```
```
static class GetThread extends Thread {

    private final CloseableHttpClient httpClient;
    private final HttpContext context;
    private final HttpGet httpget;

    public GetThread(CloseableHttpClient httpClient, HttpGet httpget) {
        this.httpClient = httpClient;
        this.context = HttpClientContext.create();
        this.httpget = httpget;
    }

    @Override
    public void run() {
        try {
            CloseableHttpResponse response = httpClient.execute(httpget, context);
            try {
                HttpEntity entity = response.getEntity();
            } finally {
                response.close();
            }
        } catch (ClientProtocolException ex) {
            //处理协议错误
        } catch (IOException ex) {
            //处理IO错误
        }
    }

}
```

### 连接驱逐策略
经典阻塞I / O模型的主要缺点之一是，仅当在I / O操作中阻塞时，网络套接字才能对I / O事件作出反应。
当将连接释放回管理器时，它可以保持活动状态，但是它无法监视套接字的状态并对任何I / O事件作出反应。
如果连接在服务器端被关闭，则客户端连接将无法 检测到连接状态的变化以及通过关闭其端部的套接字来适当地做出反应。

HttpClient尝试通过在使用连接执行HTTP请求之前测试该连接是否为“过时的”（不再有效，因为该连接已在服务器端关闭）来缓解此问题。
过时的连接检查不是100％可靠的。

唯一不涉及每个套接字模型一个线程用于空闲连接的可行解决方案是专用监视器线程，该线程用于驱逐由于长时间不活动而被视为过期的连接。
监视线程可以定期调用 ClientConnectionManager#closeExpiredConnections()方法以关闭所有过期的连接，并从池中逐出已关闭的连接。
它也可以选择调用ClientConnectionManager#closeIdleConnections() 关闭给定时间段内所有空闲连接的方法。
```
public static class IdleConnectionMonitorThread extends Thread {
    
    private final HttpClientConnectionManager connMgr;
    private volatile boolean shutdown;
    
    public IdleConnectionMonitorThread(HttpClientConnectionManager connMgr) {
        super();
        this.connMgr = connMgr;
    }

    @Override
    public void run() {
        try {
            while (!shutdown) {
                synchronized (this) {
                    wait(5000);
                    // Close expired connections
                    connMgr.closeExpiredConnections();
                    // Optionally, close connections
                    // that have been idle longer than 30 sec
                    connMgr.closeIdleConnections(30, TimeUnit.SECONDS);
                }
            }
        } catch (InterruptedException ex) {
            // terminate
        }
    }
    
    public void shutdown() {
        shutdown = true;
        synchronized (this) {
            notifyAll();
        }
    }
    
}
```

### 连接保活策略!!!
HTTP规范没有指定持久连接可以保持多长时间，并且应该保持有效状态。
一些HTTP服务器使用非标准标Keep-Alive 头与客户端通信，以秒为单位的时间段，它们打算使连接在服务器端保持活动状态。HttpClient使用此信息（如果可用）。
如果Keep-Alive标头不存在于响应中，HttpClient假定可以无限期保持连接。
但是，为了节省系统资源，许多常用的HTTP服务器被配置为在一段时间不活动后删除持久性连接，这通常是在不通知客户端的情况下进行的。

如果默认策略过于乐观，则可能需要提供一种自定义的“保持活动状态”策略。
```
ConnectionKeepAliveStrategy myStrategy = new ConnectionKeepAliveStrategy() {

    public long getKeepAliveDuration(HttpResponse response, HttpContext context) {
        // Honor 'keep-alive' header
        HeaderElementIterator it = new BasicHeaderElementIterator(
                response.headerIterator(HTTP.CONN_KEEP_ALIVE));
        while (it.hasNext()) {
            HeaderElement he = it.nextElement();
            String param = he.getName();
            String value = he.getValue();
            if (value != null && param.equalsIgnoreCase("timeout")) {
                try {
                    return Long.parseLong(value) * 1000;
                } catch(NumberFormatException ignore) {
                }
            }
        }
        HttpHost target = (HttpHost) context.getAttribute(
                HttpClientContext.HTTP_TARGET_HOST);
        if ("www.naughty-server.com".equalsIgnoreCase(target.getHostName())) {
            // Keep alive for 5 seconds only
            return 5 * 1000;
        } else {
            // otherwise keep alive for 30 seconds
            return 30 * 1000;
        }
    }

};
CloseableHttpClient client = HttpClients.custom()
        .setKeepAliveStrategy(myStrategy)
        .build();
```

### socket连接工厂
HTTP连接在java.net.Socket内部使用一个对象来处理网络上的数据传输。但是，它们依赖于ConnectionSocketFactory接口来创建，初始化和连接socket。
这使HttpClient的用户可以在运行时提供应用程序特定的socket初始化代码。
PlainConnectionSocketFactory 是用于创建和初始化普通（未加密）socket的默认工厂。
创建socket的过程与将其连接到主机的过程是分离的，因此可以在连接操作被阻塞的同时关闭socket。
```
HttpClientContext clientContext = HttpClientContext.create();
PlainConnectionSocketFactory sf = PlainConnectionSocketFactory.getSocketFactory();
Socket socket = sf.createSocket(clientContext);
int timeout = 1000; //ms
HttpHost target = new HttpHost("localhost");
InetSocketAddress remoteAddress = new InetSocketAddress(
        InetAddress.getByAddress(new byte[] {127,0,0,1}), 80);
sf.connectSocket(timeout, socket, target, remoteAddress, null, clientContext);
```

#### socket安全分层
LayeredConnectionSocketFactory是ConnectionSocketFactory接口的扩展。分层socket工厂能够在现有普通socket字上创建分层的socket。socket分层主要用于通过代理创建安全socket。
HttpClient附带了SSLSocketFactory实现SSL / TLS分层的功能。请注意，HttpClient不使用任何自定义加密功能。它完全依赖于标准Java加密（JCE）和安全socket（JSEE）扩展

#### 与连接管理器集成
定制连接套接字工厂可以与特定协议方案（例如HTTP或HTTPS）关联，然后用于创建定制连接管理器。
```
ConnectionSocketFactory plainsf = <...>
LayeredConnectionSocketFactory sslsf = <...>
Registry<ConnectionSocketFactory> r = RegistryBuilder.<ConnectionSocketFactory>create()
        .register("http", plainsf)
        .register("https", sslsf)
        .build();

HttpClientConnectionManager cm = new PoolingHttpClientConnectionManager(r);
HttpClients.custom()
        .setConnectionManager(cm)
        .build();
```

#### SSL / TLS定制
HttpClient SSLConnectionSocketFactory 用于创建SSL连接。SSLConnectionSocketFactory允许高度定制。它可以将实例 javax.net.ssl.SSLContext作为参数，并使用它来创建自定义配置的SSL连接。
```
KeyStore myTrustStore = <...>
SSLContext sslContext = SSLContexts.custom()
        .loadTrustMaterial(myTrustStore)
        .build();
SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslContext);
```

#### 主机名验证
建立连接后，除了在SSL / TLS协议级别执行的信任验证和客户端身份验证外，HttpClient还可选择验证目标主机名是否与服务器X.509证书中存储的名称匹配。
该验证可以为服务器信任材料的真实性提供其他保证。重要提示：主机名验证不应与SSL信任验证混淆。

javax.net.ssl.HostnameVerifier接口表示用于主机名验证的策略。HttpClient附带了两种 javax.net.ssl.HostnameVerifier实现。
+ DefaultHostnameVerifier：HttpClient使用的默认实现应符合RFC2818。主机名必须与证书指定的任何替代名称匹配，或者在没有给替代名称提供证书主题的最特定CN的情况下。通配符可以出现在CN和任何主题替换中。
+ NoopHostnameVerifier：此主机名验证程序实际上关闭了主机名验证。它接受任何有效且匹配目标主机的SSL会话。

默认情况下，HttpClient使用该DefaultHostnameVerifier 实现。如果需要，可以指定其他主机名验证程序实现。
```
SSLContext sslContext = SSLContexts.createSystemDefault();
SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(
        sslContext,
        NoopHostnameVerifier.INSTANCE);
```

从4.4版开始，HttpClient使用Mozilla Foundation友好维护的公共后缀列表，以确保SSL证书中的通配符不会被滥用以应用于具有公共顶级域的多个域。HttpClient附带了在发行时检索到的列表的副本。
列表的最新版本可以在 [https://publicsuffix.org/list/](https://publicsuffix.org/list/) 中找到 。强烈建议制作列表的本地副本，每天从其原始位置下载列表最多不超过一次。
```
PublicSuffixMatcher publicSuffixMatcher = PublicSuffixMatcherLoader.load(
    PublicSuffixMatcher.class.getResource("my-copy-effective_tld_names.dat"));
DefaultHostnameVerifier hostnameVerifier = new DefaultHostnameVerifier(publicSuffixMatcher);
```
可以使用null匹配器禁用针对公共后备列表的验证 。
```
DefaultHostnameVerifier hostnameVerifier = new DefaultHostnameVerifier(null);
```

### HttpClient代理配置!!!
即使HttpClient知道复杂的路由方案和代理链接，它也仅支持开箱即用的简单直接或单跳代理连接。

使用代理连接到目标主机的最简单方法是设置默认代理参数：
```
HttpHost proxy = new HttpHost("someproxy", 8080);
DefaultProxyRoutePlanner routePlanner = new DefaultProxyRoutePlanner(proxy);
CloseableHttpClient httpclient = HttpClients.custom()
        .setRoutePlanner(routePlanner)
        .build();
```

使用标准的JRE代理选择器来获取代理信息：
```
SystemDefaultRoutePlanner routePlanner = new SystemDefaultRoutePlanner(
        ProxySelector.getDefault());
CloseableHttpClient httpclient = HttpClients.custom()
        .setRoutePlanner(routePlanner)
        .build();
```

使用一种自定义RoutePlanner 实现，以便对HTTP路由计算过程进行完全控制：
```
HttpRoutePlanner routePlanner = new HttpRoutePlanner() {
    public HttpRoute determineRoute(
            HttpHost target,
            HttpRequest request,
            HttpContext context) throws HttpException {
        return new HttpRoute(target, null,  new HttpHost("someproxy", 8080),
                "https".equalsIgnoreCase(target.getSchemeName()));
    }
};

CloseableHttpClient httpclient = HttpClients.custom()
        .setRoutePlanner(routePlanner)
        .build();
    }
}
```

## HTTP状态管理
3.1。HTTP Cookie
3.2。Cookie规格
3.3。选择Cookie政策
3.4。自定义Cookie政策
3.5。Cookie的持久性
3.6。HTTP状态管理和执行上下文

## HTTP认证
4.1。用户凭证
4.2。认证方案
4.3。凭证提供者
4.4。HTTP身份验证和执行上下文
4.5。缓存认证数据
4.6。抢先认证
4.7。NTLM身份验证
4.7.1。NTLM连接持久性
4.8。SPNEGO/ Kerberos身份验证
4.8.1。SPNEGOHttpClient中的支持
4.8.2。GSS / Java Kerberos设置
4.8.3。login.conf文件
4.8.4。krb5.conf/ krb5.ini档案
4.8.5。Windows特定配置

## Fluent API
### 易于使用的Facade API
从4.2版本开始，HttpClient附带了一个基于Fluent API接口概念的易于使用的Facade API。
Fluent Facade API仅公开HttpClient的最基本功能，适用于不需要HttpClient完全灵活性的简单用例。
例如，流畅的Facade API使用户不必处理连接管理和资源释放。

通过HC Fluent API执行的HTTP请求的几个示例:
```
// 使用超时设置执行GET，并将响应内容作为String返回。
Request.Get("http://somehost/")
        .connectTimeout(1000)
        .socketTimeout(1000)
        .execute().returnContent().asString();

// 使用HTTP / 1.1通过“期望-继续”握手执行POST，
// 包含一个为String的请求正文，并以字节数组形式返回响应内容。
Request.Post("http://somehost/do-stuff")
        .useExpectContinue()
        .version(HttpVersion.HTTP_1_1)
        .bodyString("Important stuff", ContentType.DEFAULT_TEXT)
        .execute().returnContent().asBytes();

// 通过包含请求正文的代理使用自定义标头执行POST
// 作为HTML表单，并将结果保存到文件中
Request.Post("http://somehost/some-form")
        .addHeader("X-Custom-header", "stuff")
        .viaProxy(new HttpHost("myproxy", 8080))
        .bodyForm(Form.form().add("username", "vip").add("password", "secret").build())
        .execute().saveContent(new File("result.dump"));
```

Executor为了在特定的安全上下文中执行请求，还可以直接使用，从而将身份验证详细信息缓存并重新用于后续请求。
```
Executor executor = Executor.newInstance()
        .auth(new HttpHost("somehost"), "username", "password")
        .auth(new HttpHost("myproxy", 8080), "username", "password")
        .authPreemptive(new HttpHost("myproxy", 8080));

executor.execute(Request.Get("http://somehost/"))
        .returnContent().asString();

executor.execute(Request.Post("http://somehost/do-stuff")
        .useExpectContinue()
        .bodyString("Important stuff", ContentType.DEFAULT_TEXT))
        .returnContent().asString();
```

### 响应处理
流利的Facade API通常使用户不必处理连接管理和资源重新分配。
但是，在大多数情况下，这是以必须在内存中缓冲响应消息的内容为代价的。
强烈建议将其ResponseHandler用于HTTP响应处理，以避免必须在内存中缓冲内容。
```
Document result = Request.Get("http://somehost/content")
        .execute().handleResponse(new ResponseHandler<Document>() {

    public Document handleResponse(final HttpResponse response) throws IOException {
        StatusLine statusLine = response.getStatusLine();
        HttpEntity entity = response.getEntity();
        if (statusLine.getStatusCode() >= 300) {
            throw new HttpResponseException(
                    statusLine.getStatusCode(),
                    statusLine.getReasonPhrase());
        }
        if (entity == null) {
            throw new ClientProtocolException("Response contains no content");
        }
        DocumentBuilderFactory dbfac = DocumentBuilderFactory.newInstance();
        try {
            DocumentBuilder docBuilder = dbfac.newDocumentBuilder();
            ContentType contentType = ContentType.getOrDefault(entity);
            if (!contentType.equals(ContentType.APPLICATION_XML)) {
                throw new ClientProtocolException("Unexpected content type:" +
                    contentType);
            }
            String charset = contentType.getCharset();
            if (charset == null) {
                charset = HTTP.DEFAULT_CONTENT_CHARSET;
            }
            return docBuilder.parse(entity.getContent(), charset);
        } catch (ParserConfigurationException ex) {
            throw new IllegalStateException(ex);
        } catch (SAXException ex) {
            throw new ClientProtocolException("Malformed XML document", ex);
        }
    }

});
```

## HTTP缓存
6.1。一般概念
6.2。符合RFC-2616
6.3。用法示例
6.4。组态
6.5。储存后端

## 高级主题
7.1。自定义客户端连接
7.2。状态HTTP连接
7.2.1。用户令牌处理程序
7.2.2。持久状态连接
7.3。使用FutureRequestExecutionService
7.3.1。创建FutureRequestExecutionService
7.3.2。安排请求
7.3.3。取消任务
7.3.4。回呼
7.3.5。指标

# HttpClient示例
## 响应处理
此示例演示如何使用响应处理程序处理HTTP响应。
这是执行HTTP请求和处理HTTP响应的推荐方法。
这种方法使调用者可以专注于摘要HTTP响应的过程，并将系统资源释放任务委派给HttpClient。
HTTP响应处理程序的使用保证了在所有情况下基础HTTP连接都会自动释放回连接管理器。
```
CloseableHttpClient httpclient = HttpClients.createDefault();
try {
	HttpGet httpget = new HttpGet("http://httpbin.org/");

	System.out.println("Executing request " + httpget.getRequestLine());

	//创建一个自定义响应处理程序
	ResponseHandler<String> responseHandler = new ResponseHandler<String>() {
		@Override
		public String handleResponse(final HttpResponse response) throws ClientProtocolException, IOException {
			int status = response.getStatusLine().getStatusCode();
			if (status >= 200 && status < 300) {
				HttpEntity entity = response.getEntity();
				return entity != null ? EntityUtils.toString(entity) : null;
			} else {
				throw new ClientProtocolException("Unexpected response status: " + status);
			}
		}
	};
	
	String responseBody = httpclient.execute(httpget, responseHandler);
	System.out.println("----------------------------------------");
	System.out.println(responseBody);
} finally {
	httpclient.close();
}
```

## 手动连接释放
此示例演示了在手动处理HTTP响应的情况下如何确保将基础HTTP连接释放回连接管理器。
```
CloseableHttpClient httpclient = HttpClients.createDefault();
try {
	HttpGet httpget = new HttpGet("http://httpbin.org/get");

	System.out.println("Executing request " + httpget.getRequestLine());
	CloseableHttpResponse response = httpclient.execute(httpget);
	try {
		System.out.println("----------------------------------------");
		System.out.println(response.getStatusLine());

		//获取响应实体
		HttpEntity entity = response.getEntity();

		// 如果响应未包含实体，则无需担心连接释放
		if (entity != null) {
			InputStream inStream = entity.getContent();
			try {
				inStream.read();
				// 对响应做一些事情
			} catch (IOException ex) {
				// 如果发生IOException，则连接将被释放自动返回到连接管理器
				throw ex;
			} finally {
				// 关闭输入流将触发连接释放
				inStream.close();
			}
		}
	} finally {
		response.close();
	}
} finally {
	httpclient.close();
}
```

## HttpClient配置!!!
此示例演示如何自定义和配置HTTP请求执行和连接管理的最常见方面。
```
// 使用自定义消息解析器，来定制从数据流解析HTTP消息并将其写出到数据流的方式。
HttpMessageParserFactory<HttpResponse> responseParserFactory = new DefaultHttpResponseParserFactory() {
	@Override
	public HttpMessageParser<HttpResponse> create(
		SessionInputBuffer buffer, MessageConstraints constraints) {
		LineParser lineParser = new BasicLineParser() {

			@Override
			public Header parseHeader(final CharArrayBuffer buffer) {
				try {
					return super.parseHeader(buffer);
				} catch (ParseException ex) {
					return new BasicHeader(buffer.toString(), null);
				}
			}

		};
		return new DefaultHttpResponseParser(
			buffer, lineParser, DefaultHttpResponseFactory.INSTANCE, constraints) {

			@Override
			protected boolean reject(final CharArrayBuffer line, int count) {
				// try to ignore all garbage preceding a status line infinitely
				return false;
			}

		};
	}
};

HttpMessageWriterFactory<HttpRequest> requestWriterFactory = new DefaultHttpRequestWriterFactory();

// 使用自定义连接工厂自定义输出HTTP连接的初始化过程。
// 除了标准连接配置参数外，HTTP连接工厂还可以定义消息 解析器/编写器例程，以供各个连接使用。
HttpConnectionFactory<HttpRoute, ManagedHttpClientConnection> connFactory = new ManagedHttpClientConnectionFactory(
		requestWriterFactory, responseParserFactory);

// 可以基于系统或应用程序特定的属性创建安全连接的SSL上下文。
SSLContext sslcontext = SSLContexts.createSystemDefault();

// 创建自定义连接套接字工厂的注册表
Registry<ConnectionSocketFactory> socketFactoryRegistry = RegistryBuilder.<ConnectionSocketFactory>create()
	.register("http", PlainConnectionSocketFactory.INSTANCE)
	.register("https", new SSLConnectionSocketFactory(sslcontext))
	.build();

// 使用自定义DNS解析器覆盖系统DNS解析
DnsResolver dnsResolver = new SystemDefaultDnsResolver() {

	@Override
	public InetAddress[] resolve(final String host) throws UnknownHostException {
		if (host.equalsIgnoreCase("myhost")) {
			return new InetAddress[] { InetAddress.getByAddress(new byte[] {127, 0, 0, 1}) };
		} else {
			return super.resolve(host);
		}
	}

};

//// 创建具有自定义配置的连接管理器!!!
PoolingHttpClientConnectionManager connManager = new PoolingHttpClientConnectionManager(
		socketFactoryRegistry, connFactory, dnsResolver);

// 创建套接字配置
SocketConfig socketConfig = SocketConfig.custom()
	.setTcpNoDelay(true)
	.build();
// 设置连接管理器的套接字配置，默认使用 或 用于特定主机使用
connManager.setDefaultSocketConfig(socketConfig);
connManager.setSocketConfig(new HttpHost("somehost", 80), socketConfig);

// 闲置1秒钟后验证连接
connManager.setValidateAfterInactivity(1000);

// 创建消息约束
MessageConstraints messageConstraints = MessageConstraints.custom()
	.setMaxHeaderCount(200)
	.setMaxLineLength(2000)
	.build();
// 创建连接配置
ConnectionConfig connectionConfig = ConnectionConfig.custom()
	.setMalformedInputAction(CodingErrorAction.IGNORE)
	.setUnmappableInputAction(CodingErrorAction.IGNORE)
	.setCharset(Consts.UTF_8)
	.setMessageConstraints(messageConstraints)
	.build();
// 设置连接管理器的连接配置，默认使用 或 针对特定主机使用 
connManager.setDefaultConnectionConfig(connectionConfig);
connManager.setConnectionConfig(new HttpHost("somehost", 80), ConnectionConfig.DEFAULT);

//配置连接的最大总数或每个路由限制!!!
connManager.setMaxTotal(100);
connManager.setDefaultMaxPerRoute(10);
connManager.setMaxPerRoute(new HttpRoute(new HttpHost("somehost", 80)), 20);

// 如有必要，请使用自定义Cookie存储。
CookieStore cookieStore = new BasicCookieStore();

// 如有必要，请使用自定义凭据提供程序。
CredentialsProvider credentialsProvider = new BasicCredentialsProvider();

// 创建全局请求配置!!!
RequestConfig defaultRequestConfig = RequestConfig.custom()
	.setCookieSpec(CookieSpecs.DEFAULT)
	.setExpectContinueEnabled(true)
	.setTargetPreferredAuthSchemes(Arrays.asList(AuthSchemes.NTLM, AuthSchemes.DIGEST))
	.setProxyPreferredAuthSchemes(Arrays.asList(AuthSchemes.BASIC))
	.build();

// 根据自定义配置，创建HttpClient!!!
CloseableHttpClient httpclient = HttpClients.custom()
	.setConnectionManager(connManager)
	.setDefaultCookieStore(cookieStore)
	.setDefaultCredentialsProvider(credentialsProvider)
	.setProxy(new HttpHost("myproxy", 8080))
	.setDefaultRequestConfig(defaultRequestConfig)
	.build();

try {
	HttpGet httpget = new HttpGet("http://httpbin.org/get");
	// 可以在请求级别覆盖请求配置。
	// 它们将优先于客户端级别的一组。
	RequestConfig requestConfig = RequestConfig.copy(defaultRequestConfig)
		.setSocketTimeout(5000)
		.setConnectTimeout(5000)
		.setConnectionRequestTimeout(5000)
		.setProxy(new HttpHost("myotherproxy", 8080))
		.build();
	httpget.setConfig(requestConfig);

	// 可以在本地自定义执行上下文。
	HttpClientContext context = HttpClientContext.create();
	// 设置本地上下文级别的上下文属性将优先于在客户端级别设置的上下文属性。
	context.setCookieStore(cookieStore);
	context.setCredentialsProvider(credentialsProvider);

	System.out.println("executing request " + httpget.getURI());
	CloseableHttpResponse response = httpclient.execute(httpget, context);
	try {
		// EntityUtils消费response，把连接释放回连接池
		System.out.println("----------------------------------------");
		System.out.println(response.getStatusLine());
		System.out.println(EntityUtils.toString(response.getEntity()));
		System.out.println("----------------------------------------");

		//一旦执行了请求，就可以使用本地上下文检查更新状态和受请求执行影响的各种对象。

		//最后执行的请求
		context.getRequest();
		//执行路由
		context.getHttpRoute();
		//目标身份验证状态
		context.getTargetAuthState();
		//代理身份验证状态
		context.getProxyAuthState();
		// Cookie来源
		context.getCookieOrigin();
		//使用的Cookie规范
		context.getCookieSpec();
		//用户安全令牌
		context.getUserToken();

	} finally {
		response.close();
	}
} finally {
	httpclient.close();
}
```

## 中止方法
此示例演示如何在正常完成之前中止HTTP请求。
```
CloseableHttpClient httpclient = HttpClients.createDefault();
try {
	HttpGet httpget = new HttpGet("http://httpbin.org/get");

	System.out.println("Executing request " + httpget.getURI());
	CloseableHttpResponse response = httpclient.execute(httpget);
	try {
		System.out.println("----------------------------------------");
		System.out.println(response.getStatusLine());
		// Do not feel like reading the response body
		// Call abort on the request object
		httpget.abort();
	} finally {
		response.close();
	}
} finally {
	httpclient.close();
}
```

客户端认证
本示例使用HttpClient对需要用户身份验证的目标站点执行HTTP请求。

通过代理请求
本示例演示如何通过代理发送HTTP请求。

代理验证
一个简单的示例，显示通过身份验证代理通过隧道建立的安全连接上的HTTP请求执行。

块编码的POST
本示例说明如何使用块编码流式传输请求实体。

自定义执行上下文
本示例演示如何使用本地HTTP上下文填充的自定义属性。

基于表单的登录
本示例演示了如何使用HttpClient执行基于表单的登录。

线程请求执行
一个示例，它执行来自多个工作线程的HTTP请求。

自定义SSL上下文
本示例演示了如何使用自定义SSL上下文创建安全连接。

抢占式BASIC验证
本示例说明如何使用BASIC方案自定义HttpClient以抢先进行身份验证。通常，可以认为抢先式身份验证的安全性不如对身份验证质询的响应，因此不鼓励使用。

抢占式DIGEST身份验证
本示例说明如何使用DIGEST方案自定义HttpClient以抢先进行身份验证。通常，可以认为抢先式身份验证的安全性不如对身份验证质询的响应，因此不鼓励使用。

代理隧道
本示例说明如何使用ProxyClient来通过HTTP代理为任意协议建立隧道。

多部分编码的请求实体
本示例说明如何执行包含多部分编码实体的请求。

本机Windows Negotiate / NTLM
本示例说明了在Windows OS上运行时如何利用本机Windows Negotiate / NTLM身份验证。


# HttpClient连接管理示例
[HttpClient连接管理](https://www.cnblogs.com/filozofio/p/12155218.html)

HttpClient中的连接是有状态且线程不安全的，它使用专门的连接管理器来管理这些连接，作为连接的工厂，负责连接的生命周期管理，以及对连接的并发访问进行同步。
连接管理器抽象为 HttpClientConnectionManager 接口，接口有两种实现，分别是 BasicHttpClientConnectionManager 和 PoolingHttpClientConnectionManager。

## BasicHttpClientConnectionManager
BasicHttpClientConnectionManager，http连接管理器最简单的一种实现，用于创建和管理单个连接，只用于单线程，显然也是线程安全的。

下面是基于 BasicHttpClientConnectionManager 的底层API的使用。
requestConnection 方法从 connectionManager 管理的连接池取出一个连接，连接到 route 对象定义的“www.baidu.com”。
```
BasicHttpClientConnectionManager connectionManager = new BasicHttpClientConnectionManager();
HttpRoute route = new HttpRoute(new HttpHost("www.baidu.com", 80));
ConnectionRequest connectionRequest = connectionManager.requestConnection(route, null);
```

## PoolingHttpClientConnectionManager
PoolingHttpClientConnectionManager 可以创建并管理一个连接池，为多个路由或目标主机提供连接。

简单的用法如下：
```
//为一个HttpClient对象配置连接池
PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager();
CloseableHttpClient client = HttpClients.custom().setConnectionManager(connectionManager).build();
try {
    client.execute(new HttpGet("https://www.baidu.com"));
} catch (IOException e) {
    e.printStackTrace();
}
System.out.println(connectionManager.getTotalStats().getLeased());
```

单个连接池可以供多个线程的多个HttpClient对象使用：
```
//可以使用一个连接池，管理面向不同目标主机的请求
HttpGet get1 = new HttpGet("https://www.zhihu.com");
HttpGet get2 = new HttpGet("https://www.baidu.com");
PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager();

CloseableHttpClient client1 = HttpClients.custom().setConnectionManager(connectionManager).build();
CloseableHttpClient client2 = HttpClients.custom().setConnectionManager(connectionManager).build();

MultiHttpClientConnThread t1 = new MultiHttpClientConnThread(client1, get1);
MultiHttpClientConnThread t2 = new MultiHttpClientConnThread(client2, get2);
t1.start();
t2.start();
try {
    t1.join();
    t2.join();
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

其中 MultiHttpClientConnThread 是自定义的类，定义如下
```
@Slf4j
public class MultiHttpClientConnThread extends Thread {
    private final CloseableHttpClient client;
    private final HttpGet get;
    private PoolingHttpClientConnectionManager connectionManager;

    public MultiHttpClientConnThread(final CloseableHttpClient client, final HttpGet get) {
        this.client = client;
        this.get = get;
    }

    public MultiHttpClientConnThread(final CloseableHttpClient client, final HttpGet get, final PoolingHttpClientConnectionManager connectionManager) {
        this.client = client;
        this.get = get;
        this.connectionManager = connectionManager;
    }
    @Override
    public void run() {
        try {
            log.info("Thread Running:" + getName());
            if (connectionManager != null) {
                log.info("Before - Leased Connections = " + connectionManager.getTotalStats().getLeased());
                log.info("Before - Available Connections = " + connectionManager.getTotalStats().getAvailable());
            }
            HttpResponse response = client.execute(get);
            if (connectionManager != null) {
                log.info("After - Leased Connections = " + connectionManager.getTotalStats().getLeased());
                log.info("After - Available Connections = " + connectionManager.getTotalStats().getAvailable());
            }
            //消费response，为了把连接释放回连接池
            EntityUtils.consume(response.getEntity());
        } catch (IOException e) {
            log.error("", e);
        }
    }	
}
```
注意EntityUtils.consume(response.getEntity())，需要消费掉response的全部内容，连接管理器才会把这个连接释放回归连接池。

## 配置连接管理器
PoolingHttpClientConnectionManager 可配置的选项如下：
+ 连接总数，默认值为20
+ 单个普通路由的最大连接数，默认值为2
+ 特定某个路由的最大连接数，默认值为2
```
//调整默认的连接池参数
PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager();
//总连接数为5
connectionManager.setMaxTotal(5);
//单个路由最大连接数为4
connectionManager.setDefaultMaxPerRoute(4);
//特定路由www.baidu.com的最大连接数为5
HttpHost httpHost = new HttpHost("www.baidu.com", 80);
HttpRoute route = new HttpRoute(httpHost);
connectionManager.setMaxPerRoute(route, 5);
```

如果使用默认设置，在多线程请求的情况下，单个路由很容易就达到最大连接数了
```
PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager();
CloseableHttpClient client = HttpClients.custom().setConnectionManager(connectionManager).build();
MultiHttpClientConnThread t1 = new MultiHttpClientConnThread(client, new HttpGet("http://www.baidu.com"), connectionManager);
MultiHttpClientConnThread t2 = new MultiHttpClientConnThread(client, new HttpGet("http://www.baidu.com"), connectionManager);
MultiHttpClientConnThread t3 = new MultiHttpClientConnThread(client, new HttpGet("http://www.baidu.com"), connectionManager);
t1.start();
t2.start();
t3.start();
t1.join();
t2.join();
t3.join();
```

运行以上代码可以看到以下结果：
```
INFO chenps3.httpclient.MultiHttpClientConnThread(36) - Thread Running:Thread-0
INFO chenps3.httpclient.MultiHttpClientConnThread(36) - Thread Running:Thread-1
INFO chenps3.httpclient.MultiHttpClientConnThread(36) - Thread Running:Thread-2
INFO chenps3.httpclient.MultiHttpClientConnThread(38) - Before - Leased Connections = 0
INFO chenps3.httpclient.MultiHttpClientConnThread(38) - Before - Leased Connections = 0
INFO chenps3.httpclient.MultiHttpClientConnThread(39) - Before - Available Connections = 0
INFO chenps3.httpclient.MultiHttpClientConnThread(38) - Before - Leased Connections = 0
INFO chenps3.httpclient.MultiHttpClientConnThread(39) - Before - Available Connections = 0
INFO chenps3.httpclient.MultiHttpClientConnThread(39) - Before - Available Connections = 0
INFO chenps3.httpclient.MultiHttpClientConnThread(43) - After - Leased Connections = 2
INFO chenps3.httpclient.MultiHttpClientConnThread(43) - After - Leased Connections = 2
INFO chenps3.httpclient.MultiHttpClientConnThread(44) - After - Available Connections = 0
INFO chenps3.httpclient.MultiHttpClientConnThread(44) - After - Available Connections = 0
INFO chenps3.httpclient.MultiHttpClientConnThread(43) - After - Leased Connections = 1
INFO chenps3.httpclient.MultiHttpClientConnThread(44) - After - Available Connections = 1
```

可以看到，即使有3个线程的请求并发执行，最多只有2个连接被使用。没有拿到连接的线程则会暂时阻塞，直到有连接归还到连接池。

## keep-alive策略
如果没有在响应头部找到keep-alive，HttpClient假定是无限大，因此通常需要自定义一个keep-alive策略。
```
//优先使用响应头的keep-alive值，如果没找到，设置为5秒
ConnectionKeepAliveStrategy strategy = new ConnectionKeepAliveStrategy() {
    @Override
    public long getKeepAliveDuration(HttpResponse httpResponse, HttpContext httpContext) {
        HeaderElementIterator it = new BasicHeaderElementIterator(httpResponse.headerIterator(HTTP.CONN_KEEP_ALIVE));
        while (it.hasNext()) {
            HeaderElement he = it.nextElement();
            String param = he.getName();
            String value = he.getValue();
            if (value != null && param.equalsIgnoreCase("timeout")) {
                return Long.parseLong(value) * 1000;
            }
        }
        return 5000;
    }
};
//自定义策略应用到client
CloseableHttpClient client = HttpClients.custom()
        .setKeepAliveStrategy(strategy)
        .build();
```

## 连接持久化与复用
HTTP/1.1 规范中声明，如果连接没有被关闭，就可以被复用。HttpClient中，连接一旦被连接管理器释放，就会保持可复用的状态。

BasicHttpClientConnectionManager只能使用一个连接，因此使用前必须要先显式释放:
```
BasicHttpClientConnectionManager basic = new BasicHttpClientConnectionManager();
HttpClientContext ctx = HttpClientContext.create();
HttpGet get = new HttpGet("https://www.baidu.com");
//使用底层api实现一次请求
HttpRoute route = new HttpRoute(new HttpHost("www.baidu.com", 80));
ConnectionRequest request = basic.requestConnection(route, null);
HttpClientConnection connection = request.get(10, TimeUnit.SECONDS);
basic.connect(connection, route, 1000, ctx);
basic.routeComplete(connection, route, ctx);

HttpRequestExecutor executor = new HttpRequestExecutor();
ctx.setTargetHost(new HttpHost("www.baidu.com", 80));

executor.execute(get, connection, ctx);
//显式释放连接，允许被复用
basic.releaseConnection(connection, null, 1, TimeUnit.SECONDS);
//使用高层api实现一次请求
CloseableHttpClient client = HttpClients.custom()
        .setConnectionManager(basic)
        .build();
client.execute(get);
```
如果没有显式释放连接，执行最后一行代码会有以下异常：
```
Exception in thread "main" java.lang.IllegalStateException: Connection is still allocated
```

PoolingHttpClientConnectionManager 可以隐式释放连接。以下代码使用10个线程执行10个请求，共享5个连接：
```
PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager();
connectionManager.setDefaultMaxPerRoute(5);
connectionManager.setMaxTotal(5);
CloseableHttpClient client = HttpClients.custom()
        .setConnectionManager(connectionManager)
        .build();
		
MultiHttpClientConnThread[] threads = new MultiHttpClientConnThread[10];
for (int i = 0; i < threads.length; i++) {
    threads[i] = new MultiHttpClientConnThread(client, new HttpGet("http://www.baidu.com"), connectionManager);
}
for (MultiHttpClientConnThread i : threads) {
    i.start();
}
for (MultiHttpClientConnThread i : threads) {
    i.join();
}
```

## 配置超时时间
虽然HttpClient支持设置多种超时时间，但通过连接管理器，只能设置socket的超时时间。
```
//设置socket超时时间为5秒
PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager();
connectionManager.setSocketConfig(
        new HttpHost("www.baidu.com", 80),
        SocketConfig.custom().setSoTimeout(5000).build());
```

## 连接驱逐
连接驱逐是指，探测空闲和过期的连接，并关闭它们。连接驱逐有两种实现方式：
+ 在HttpClient执行请求前检测连接是否过期
+ 使用一个监控线程来探测并关闭过期连接

```
//通过定义一个RequestConfig对象，令client在请求前检查连接是否过期，有性能损耗
PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager();
RequestConfig requestConfig = RequestConfig.custom().setStaleConnectionCheckEnabled(true).build();
CloseableHttpClient client = HttpClients.custom()
        .setDefaultRequestConfig(requestConfig)
        .setConnectionManager(connectionManager)
        .build();
```

```
//定义一个监视器线程类，探测并关闭过期连接和超过30秒的空闲连接
public class IdleConnectionMonitorThread extends Thread {
    private final HttpClientConnectionManager connectionManager;
    private volatile boolean shutdown;
    public IdleConnectionMonitorThread(HttpClientConnectionManager connectionManager) {
        this.connectionManager = connectionManager;
    }
    @Override
    public void run() {
        try {
            while (!shutdown) {
                synchronized (this) {
                    wait(1000);
                    //关闭过期连接
                    connectionManager.closeExpiredConnections();
                    //关闭空闲超过30秒的连接
                    connectionManager.closeIdleConnections(30, TimeUnit.SECONDS);
                }
            }
        } catch (InterruptedException e) {
            showdown();
        }
    }
    private void showdown() {
        shutdown = true;
        synchronized (this) {
            notifyAll();
        }
    }
}
```

## 关闭连接
正确关闭连接的步骤如下：
+ 消费并关闭响应
+ 关闭client对象
+ 关闭connection manager对象

如果连接关闭之前就关闭掉了连接管理器，管理器所管理的所有连接和资源都会直接释放。
```
PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager();
CloseableHttpClient client = HttpClients.custom()
        .setConnectionManager(connectionManager)
        .build();
HttpGet get = new HttpGet("https://www.baidu.com");
CloseableHttpResponse response = client.execute(get);

EntityUtils.consume(response.getEntity());      //消费响应
response.close();           //关闭响应
client.close();             //关闭client对象
connectionManager.close();  //关闭connection manager对象
```
