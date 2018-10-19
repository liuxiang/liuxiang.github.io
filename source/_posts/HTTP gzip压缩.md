title: HTTP gzip压缩
date: 2018-8-13 00:00:00
categories: http
tags: [http]

---
[TOC]

---
# 一.HTTP gzip压缩,概述
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-8-25/13033563.jpg)

- request 
    - header中声明`Accept-Encoding` : gzip，告知服务器客户端接受gzip的数据
- response 
    - body，同时加入以下header：`Content-Encoding`: gzip：表明body是gzip过的数据
    - `Content-Length:117`：表示body gzip压缩后的数据大小，便于客户端使用
    - 或`Transfer-Encoding: chunked`：分块传输编码

---
# 二.如何使用gzip进行压缩
## tomcat开启压缩(gzip)
- tomcat server.xml
```
<Connector
compression="on" # 表示开启压缩
noCompressionUserAgents="gozilla, traviata"
compressionMinSize="2048" # 表示会对大于2KB的文件进行压缩
compressableMimeType="text/html,text/xml,text/css,text/javascript,image/gif,image/jpg" # 是指将进行压缩的文件类型
/>
```

- 弊端
对HTTP传输内容进行压缩是改良前端响应性能的可用方法之一，大型网站都在用。但是也有缺点，就是压缩过程占用cpu的资源，客户端浏览器解析也占据了一部分时间。但是随着硬件性能不断的提高，这些问题正在不断的弱化。

---
## 程序压缩/解压
GZIPInputStream(解压) / GZIPOutputStream(压缩)
- netflix.zuul相关示例
```
# org.springframework.cloud.netflix.zuul.filters.post.SendResponseFilter#writeResponse()
is = context.getResponseDataStream();
InputStream inputStream = is;
if (is != null) {
    if (context.sendZuulResponse()) {
        // if origin response is gzipped, and client has not requested gzip,
        // decompress stream
        // before sending to client
        // else, stream gzip directly to client
        if (context.getResponseGZipped() && !isGzipRequested) {
            // If origin tell it's GZipped but the content is ZERO bytes,
            // don't try to uncompress
            final Long len = context.getOriginContentLength();
            if (len == null || len > 0) {
                try {
                    inputStream = new GZIPInputStream(is);
                }
                catch (java.util.zip.ZipException ex) {
                    log.debug(
                            "gzip expected but not "
                                    + "received assuming unencoded response "
                                    + RequestContext.getCurrentContext()
                                    .getRequest().getRequestURL()
                                    .toString());
                    inputStream = is;
                }
            }
            else {
                // Already done : inputStream = is;
            }
        }
        else if (context.getResponseGZipped() && isGzipRequested) {
            servletResponse.setHeader(ZuulHeaders.CONTENT_ENCODING, "gzip");
        }
        writeResponse(inputStream, outStream);
    }
}

# com.netflix.zuul.http.HttpServletRequestWrapper.UnitTest#handlesGzipRequestBody
@Test
public void handlesGzipRequestBody() throws IOException {
    // creates string, gzips into byte array which will be mocked as InputStream of request
    final String body = "hello";
    final byte[] bodyBytes = body.getBytes();
    // in this case the compressed stream is actually larger - need to allocate enough space
    final ByteArrayOutputStream byteOutStream = new ByteArrayOutputStream(0);
    final GZIPOutputStream gzipOutStream = new GZIPOutputStream(byteOutStream);
    gzipOutStream.write(bodyBytes);
    gzipOutStream.finish();
    gzipOutStream.flush();
    body(byteOutStream.toByteArray());

    final HttpServletRequestWrapper wrapper = new HttpServletRequestWrapper(request);
    assertEquals(body, IOUtils.toString(new GZIPInputStream(wrapper.getInputStream())));
}
```

## 示例: 网关主动对response进行压缩响应(可减少带宽)  `GZIPOutputStream`
- 简单实现示例.实际情况需考虑更新情况,如是否已经被压缩等
```
InputStream inputStream = okResponse.body().byteStream();
try {
    // 网关主动对response进行压缩响应(可减少带宽)
    HttpServletRequest request = RequestContext.getCurrentContext().getRequest();
    boolean isGatewayGZIP = Boolean.parseBoolean(request.getHeader("x-gateway-gzip"));
    if (!isGatewayGZIP) {
        isGatewayGZIP = Boolean.parseBoolean(request.getParameter("x-gateway-gzip"));
    }

    if (isGatewayGZIP) {
        final ByteArrayOutputStream byteOutStream = new ByteArrayOutputStream(0);
        final GZIPOutputStream gzipOutStream = new GZIPOutputStream(byteOutStream);
        gzipOutStream.write(okResponse.body().bytes());
        gzipOutStream.finish();
        gzipOutStream.flush();
        inputStream = new ServletInputStreamWrapper(byteOutStream.toByteArray());
        httpHeaders.add(ZuulHeaders.CONTENT_ENCODING, "gzip");
    }
} catch (Exception e) {
    logger.error("GatewayGZIP error:", e);
}
```

---
# 三.okhttp 压缩相关处理
## okHttp 解压gzip,条件: Content-Encoding = gizp
- okio.GzipSource
```
    if (transparentGzip
        && "gzip".equalsIgnoreCase(networkResponse.header("Content-Encoding"))
        && HttpHeaders.hasBody(networkResponse)) {
      GzipSource responseBody = new GzipSource(networkResponse.body().source());
      Headers strippedHeaders = networkResponse.headers().newBuilder()
          .removeAll("Content-Encoding")
          .removeAll("Content-Length")
          .build();
      responseBuilder.headers(strippedHeaders);
      String contentType = networkResponse.header("Content-Type");
      responseBuilder.body(new RealResponseBody(contentType, -1L, Okio.buffer(responseBody)));
    }
```

---
## okhttp gzip压缩/解压 (示例)
```
//zip压缩
GzipSink gzipSink = new GzipSink(Okio.sink(file));
BufferedSink bufferedSink = Okio.buffer(gzipSink);
bufferedSink.writeUtf8("this is zip file");
bufferedSink.flush();
bufferedSink.close();

//读取zip
GzipSource gzipSource = new GzipSource(Okio.source(file));
BufferedSource bufferedSource = Okio.buffer(gzipSource);
String s = bufferedSource.readUtf8();
```

---
## okhttp框架－如何`对请求(request)数据进行GZIP压缩`-`GzipRequestInterceptor`
```
OkHttpClient okHttpClient = new OkHttpClient.Builder() 
    .addInterceptor(new GzipRequestInterceptor())//开启Gzip压缩
    ...
    .build();
```

- GzipRequestInterceptor
https://github.com/square/okhttp\\issues/350#issuecomment-123105641
```
class GzipRequestInterceptor implements Interceptor {
  @Override public Response intercept(Chain chain) throws IOException {
    Request originalRequest = chain.request();
    if (originalRequest.body() == null || originalRequest.header("Content-Encoding") != null) {
      return chain.proceed(originalRequest);
    }

    Request compressedRequest = originalRequest.newBuilder()
        .header("Content-Encoding", "gzip")
        .method(originalRequest.method(), forceContentLength(gzip(originalRequest.body())))
        .build();
    return chain.proceed(compressedRequest);
  }

  /** https://github.com/square/okhttp\\issues/350 */
  private RequestBody forceContentLength(final RequestBody requestBody) throws IOException {
    final Buffer buffer = new Buffer();
    requestBody.writeTo(buffer);
    return new RequestBody() {
      @Override
      public MediaType contentType() {
        return requestBody.contentType();
      }

      @Override
      public long contentLength() {
        return buffer.size();
      }

      @Override
      public void writeTo(BufferedSink sink) throws IOException {
        sink.write(buffer.snapshot());
      }
    };
  }

  private RequestBody gzip(final RequestBody body) {
    return new RequestBody() {
      @Override public MediaType contentType() {
        return body.contentType();
      }

      @Override public long contentLength() {
        return -1; // We don't know the compressed length in advance!
      }

      @Override public void writeTo(BufferedSink sink) throws IOException {
        BufferedSink gzipSink = Okio.buffer(new GzipSink(sink));
        body.writeTo(gzipSink);
        gzipSink.close();
      }
    };
  }
}
```
`okhttp框架－如何对请求数据进行GZIP压缩`
https://cloud.tencent.com/info/61307ab74137a46628c2ea2ca42a6eb4.html

`Okhttp3请求网络开启Gzip压缩 - CSDN博客`
https://blog.csdn.net/aiynmimi/article/details/77453809

---
# 四.Nginx的Gzip可以对服务器端响应内容进行压缩从而减少一定的客户端响应时间
```
gzip on;
gzip_min_length 1k;
gzip_buffers 4 32k;
gzip_types text/plain application/x-javascript application/javascript text/xml text/css;
gzip_vary on;
```
`API网关那些儿 | I'm Yunlong`
http://ylzheng.com/2017/03/14/the-things-about-api-gateway/

---
**参考**
`聊聊HTTP gzip压缩与常见的Android网络框架`
https://www.cnblogs.com/ct2011/p/5835990.html

`前端性能优化-对HTTP传输进行压缩`
https://www.jianshu.com/p/74c10af7707d
https://blog.csdn.net/clerk0324/article/details/51672933

`java GZIP压缩与解压缩 - 探寻者宇 - 博客园`
https://www.cnblogs.com/searcherY/p/6723615.html

`zuul网关源码解析 - org.springframework.cloud.netflix.zuul.filters.post.SendResponseFilter#writeResponse()`
https://www.cnblogs.com/liangzs/p/8695397.html

`Android使用OkHttp进行网络同步异步操作`
https://www.jb51.net/article/144086.htm

`Okio简化处理IO操作 - CSDN博客`
https://blog.csdn.net/zhangquanit/article/details/53072192