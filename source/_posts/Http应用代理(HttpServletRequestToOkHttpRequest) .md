title: Http应用代理(HttpServletRequestToOkHttpRequest)
date: 2018-8-07 00:00:00
categories: httpProxy
tags: [httpProxy]

---
[TOC]

---
# 场景
当某客户端想调用的某服务端,受到网络隔离等因素造成无法访问. 
如果存在一台两边互通的应用(proxy),可以借此为`跳板应用`.
有了物理层的网络支持,还需要应用层的HttpProxy代理支持.

- http新构建,兼容代理:  手动获取参数,header及请求类型. Switch各请求类型分支定向构建请求方式,补全数据,再代理发送.
- http流切换,兼容代理:  基于inputStream自动切换request(HttpServletRequestToOkHttpRequest)可全类型自动兼容,再代理发送.

- 示例:
localhost:8010/httpProxy?uri=localhost:8010/serverMock&aa=bb

---
# Http流切换,兼容代理示例
- HttpProxyController.java
```
    // 超时配置(连接,读)默认1s.
    OkHttpClient client = new OkHttpClient.Builder()
            .connectTimeout(1000, TimeUnit.MILLISECONDS)// 默认值:10000
            .readTimeout(10000, TimeUnit.MILLISECONDS)// 默认值:10000
            .build();

    @RequestMapping(value = "")
    public Object httpProxy(@RequestParam String uri,HttpServletRequest request) throws Exception {
        uri = new URL(request.getRequestURL().toString()).getProtocol() + "://" + uri;

        okhttp3.Request okhttpRequest = new HttpServletRequestToOkHttpRequest
                .Builder()
                .Builder()
                .setQueryParamsRemove(Arrays.asList("uri"))// 剔除query参数
                .setHeadersAdd(null)// 补充header
                .setIgnoredHeaders(null)// 剔除header
                .setInputStreamSet(null)// 自定义流(更新body内容)
                .build()
                .buildOkhttpRequest((HttpServletRequest)request, URI.create(uri));

        okhttp3.Response response = client.newCall(request).execute();
        return JSON.parse(response.body().string());// 实际出参依据接口而定.
        // 如需代理响应,需将okhttp3.Response拆装为HttpServletResponse即可.
    }
```

- HttpServletRequestToOkHttpRequest.java
```
import okhttp3.*;
import okhttp3.internal.http.HttpMethod;
import okio.BufferedSink;
import okio.Okio;
import okio.Source;
import org.apache.catalina.connector.CoyoteInputStream;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpOutputMessage;
import org.springframework.http.converter.support.AllEncompassingFormHttpMessageConverter;
import org.springframework.util.CollectionUtils;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;

import javax.servlet.ReadListener;
import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServletRequest;
import java.io.*;
import java.net.URI;
import java.util.Collection;
import java.util.Enumeration;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentMap;

public class HttpServletRequestToOkHttpRequest {

    final Map<String, List<String>> queryParamsSet;
    final List<String> queryParamsRemove;
    final Map<String, String> headersAdd;
    final InputStream inputStreamSet;
    final List<String> ignoredHeaders;

    ConcurrentMap ctx = new ConcurrentHashMap();

    public HttpServletRequestToOkHttpRequest() {
        this(new HttpServletRequestToOkHttpRequest.Builder());
    }

    public HttpServletRequestToOkHttpRequest(Builder builder) {
        this.queryParamsSet = builder.queryParamsSet;
        this.queryParamsRemove = builder.queryParamsRemove;
        this.headersAdd = builder.headersAdd;
        this.inputStreamSet = builder.inputStreamSet;
        this.ignoredHeaders = builder.ignoredHeaders;
    }

    /**
     * 构建器
     */
    public static final class Builder {

        Map<String, List<String>> queryParamsSet;
        List<String> queryParamsRemove;
        Map<String, String> headersAdd;
        InputStream inputStreamSet;
        List<String> ignoredHeaders;

        public Builder() {
            // default
        }

        public Builder setQueryParamsRemove(List<String> queryParamsRemove) {
            this.queryParamsRemove = queryParamsRemove;
            return this;
        }

        public Builder setQueryParamsSet(Map<String, List<String>> queryParamsSet) {
            this.queryParamsSet = queryParamsSet;
            return this;
        }

        public Builder setHeadersAdd(Map<String, String> headersAdd) {
            this.headersAdd = headersAdd;
            return this;
        }

        public Builder setInputStreamSet(InputStream inputStreamSet) {
            this.inputStreamSet = inputStreamSet;
            return this;
        }

        public Builder setIgnoredHeaders(List<String> ignoredHeaders) {
            this.ignoredHeaders = ignoredHeaders;
            return this;
        }

        public HttpServletRequestToOkHttpRequest build() {
            return new HttpServletRequestToOkHttpRequest(this);
        }

    }

    /**
     * okHttp
     *
     * @param request
     * @param uri
     * @return
     */
    public okhttp3.Request buildOkhttpRequest(HttpServletRequest request, URI uri) throws IOException {
        // headers
        MultiValueMap<String, String> headers = buildRequestHeaders(request);

        // QueryParameter
        HttpUrl.Builder url = HttpUrl.get(uri).newBuilder();
        url.query(request.getQueryString());// 补全原文
        if (queryParamsSet != null) {// 追加定义
            for (Map.Entry<String, List<String>> entry : queryParamsSet.entrySet()) {
                for (String value : entry.getValue()) {
                    url.setQueryParameter(entry.getKey(), value);
                }
            }
        }
        if (queryParamsRemove != null) {
            for (String queryParam : queryParamsRemove) {
                url.removeAllQueryParameters(queryParam);
                url.removeAllEncodedQueryParameters(queryParam);
            }
        }

        // contentLength // long contentLength = useServlet31 ? request.getContentLengthLong(): request.getContentLength();
        long contentLength = -1;
        try {
            contentLength = request.getContentLength();
        } catch (Throwable e) {
            contentLength = request.getContentLengthLong();
        }

        InputStream requestInputStream = request.getInputStream();
        if (requestInputStream instanceof CoyoteInputStream && request.getContentType() != null) {
            requestInputStream = new ServletInputStreamWrapper(buildContentData(request));
        }

        // inputStream
        InputStream inputStream = (inputStreamSet != null) ? inputStreamSet : requestInputStream;

        return buildOkhttpRequest(url.build(), contentLength, request.getMethod(), headers, inputStream);
    }

    private Request buildOkhttpRequest(HttpUrl url, long contentLength, String method, MultiValueMap<String, String> headersMap, InputStream inputStream) {

        Headers.Builder headers = new Headers.Builder();
        for (String name : headersMap.keySet()) {
            List<String> values = headersMap.get(name);
            for (String value : values) {
                headers.add(name, value);
            }
        }

        method = (method != null) ? method : "GET";
        RequestBody requestBody = null;
        if (inputStream != null && HttpMethod.permitsRequestBody(method)) {
            MediaType mediaType = null;
            if (headers.get("Content-Type") != null) {
                mediaType = MediaType.parse(headers.get("Content-Type"));
            }
            requestBody = new InputStreamRequestBody(inputStream, mediaType, contentLength);
        }

        Request.Builder builder = new Request.Builder()
                .url(url)
                .headers(headers.build())
                .method(method, requestBody);

        return builder.build();
    }

    /**
     * 参考: org.springframework.cloud.netflix.zuul.filters.pre.FormBodyWrapperFilter.FormBodyRequestWrapper#buildContentData
     *
     * @param request
     * @return
     */
    synchronized byte[] buildContentData(HttpServletRequest request) {
        try {
            MultiValueMap<String, Object> builder = RequestContentDataExtractor.extract(request);
            FormHttpOutputMessage data = new FormHttpOutputMessage();

            org.springframework.http.MediaType contentType = org.springframework.http.MediaType.valueOf(request.getContentType());
            data.getHeaders().setContentType(contentType);

            // FormBodyWrapperFilter.this.formHttpMessageConverter.write(builder, contentType, data);
            new AllEncompassingFormHttpMessageConverter().write(builder, contentType, data);// ★ 关键转换

            // copy new content type including multipart boundary
// this.contentType = data.getHeaders().getContentType();
// this.contentData = data.getInput();
// this.contentLength = this.contentData.length;
            return data.getInput();
        } catch (Exception e) {
            throw new IllegalStateException("Cannot convert form data", e);
        }
    }

    private class FormHttpOutputMessage implements HttpOutputMessage {

        private HttpHeaders headers = new HttpHeaders();
        private ByteArrayOutputStream output = new ByteArrayOutputStream();

        @Override
        public HttpHeaders getHeaders() {
            return this.headers;
        }

        @Override
        public OutputStream getBody() {
            return this.output;
        }

        public byte[] getInput() throws IOException {
            this.output.flush();
            return this.output.toByteArray();
        }
    }

    /**
     * 参考:com.netflix.zuul.http.ServletInputStreamWrapper
     */
    public class ServletInputStreamWrapper extends ServletInputStream {

        private byte[] data;
        private int idx = 0;

        /**
         * Creates a new <code>ServletInputStreamWrapper</code> instance.
         *
         * @param data a <code>byte[]</code> value
         */
        public ServletInputStreamWrapper(byte[] data) {
            if (data == null)
                data = new byte[0];
            this.data = data;
        }

        @Override
        public int read() throws IOException {
            if (idx == data.length)
                return -1;
            // I have to AND the byte with 0xff in order to ensure that it is returned as an unsigned integer
            // the lack of this was causing a weird bug when manually unzipping gzipped request bodies
            return data[idx++] & 0xff;
        }

        @Override
        public boolean isFinished() {
            return false;
        }

        @Override
        public boolean isReady() {
            return false;
        }

        @Override
        public void setReadListener(ReadListener readListener) {

        }
    }

    @Deprecated
    private String getVerb(String method) {
        if (method == null) {
            return "GET";
        }
        return method;
    }

    private class InputStreamRequestBody extends RequestBody {

        private InputStream inputStream;
        private MediaType mediaType;
        private Long contentLength;

        InputStreamRequestBody(InputStream inputStream, MediaType mediaType, Long contentLength) {
            this.inputStream = inputStream;
            this.mediaType = mediaType;
            this.contentLength = contentLength;
        }

        @Override
        public MediaType contentType() {
            return mediaType;
        }

        @Override
        public long contentLength() {
            if (contentLength != null) {
                return contentLength;
            }
            try {
                return inputStream.available();
            } catch (IOException e) {
                return 0;
            }
        }

        @Override
        public void writeTo(BufferedSink sink) throws IOException {
            Source source = null;
            try {
                source = Okio.source(inputStream);
                sink.writeAll(source);
            } finally {
                if (source != null) {
                    source.close();
                }
            }
        }
    }

    private MultiValueMap<String, String> buildRequestHeaders(HttpServletRequest request) {
        return buildRequestHeaders(request, null);
    }

    private MultiValueMap<String, String> buildRequestHeaders(HttpServletRequest request, Map<String, String> requestHeaders) {
        MultiValueMap<String, String> headers = new HttpHeaders();
        Enumeration<String> headerNames = request.getHeaderNames();
        if (headerNames != null) {
            while (headerNames.hasMoreElements()) {
                String name = headerNames.nextElement();
                if (isIncludedHeader(name)) {
                    Enumeration<String> values = request.getHeaders(name);
                    while (values.hasMoreElements()) {
                        String value = values.nextElement();
                        headers.add(name, value);
                    }
                }
            }
        }

        if (requestHeaders != null) {
            for (String header : requestHeaders.keySet()) {
                headers.set(header, requestHeaders.get(header));
            }
        }
        headers.set(HttpHeaders.ACCEPT_ENCODING, "gzip");
        return headers;
    }

    private boolean isIncludedHeader(String headerName) {
        String name = headerName.toLowerCase();
        if (!CollectionUtils.isEmpty(ignoredHeaders)) {
            Object object = ignoredHeaders;
            if (object instanceof Collection && ((Collection<?>) object).contains(name)) {
                return false;
            }
        }
        switch (name) {
            case "host":
            case "connection":
            case "content-length":
            case "content-encoding":
            case "server":
            case "transfer-encoding":
            case "x-application-context":
                return false;
            default:
                return true;
        }
    }

    public MultiValueMap<String, String> buildZuulRequestQueryParams(HttpServletRequest request) {
        Map<String, List<String>> map = HTTPRequestUtils.getInstance().getQueryParams(request);
        MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
        if (map == null) {
            return params;
        }
        for (String key : map.keySet()) {
            for (String value : map.get(key)) {
                params.add(key, value);
            }
        }
        return params;
    }

}
```