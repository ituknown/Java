# Maven 依赖

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>${okhttp.version}</version>
</dependency>
```

需要知道一点的是：okhttp 自 4.0.0 开始，使用 kotlin 语言进行了重写。

# 组件及请求流程

| **组件** | **描述** |
| :--- | :--- |
| `OkHttpClient` | 一个线程安全的客户端实例（类似浏览器）。在实际在应用中应该全局唯一，通常创建为单例模式。 |
| `Request` | 表示一次 HTTP 请求。可以设置请求方式、URL、Header及消息体。通常工厂 `new Request.Builder()...build()` 进行构建。 |
| `Call` | 用于发送 HTTP 请求并处理响应。`Call` 接口定义了一个 `execute()` 方法和一个 `enqueue()` 方法，分别用于同步执行和异步执行 HTTP 请求。 |
| `Response` | HTTP 响应结果。 |

基本的请求流程为：

1、创建OkHttpClient客户端

```java
// 应该设计为单例模式，全局共用一个 CLIENT 实例
private static final OkHttpClient CLIENT = new OkHttpClient();
```

推荐使用下面的方式：

```java
private static final OkHttpClient CLIENT = new OkHttpClient().newBuilder().build();
```

因为在构造实例是我们通常会做些请求设置，比如超时时间、网络拦截器和DNS等。使用 Builder 模式更加方便，实例：

```java
private static final OkHttpClient CLIENT = new OkHttpClient().newBuilder()
	.connectTimeout(3, TimeUnit.SECONDS) // 设置超时时间
    .addInterceptor(...) // 添加拦截器
    .dns(...)  // 设置DNS
    .build();
```

当然，使用第一种也是可以的。但是你不觉得 Builder 模式阅读起来更爽 :) ？

2、创建请求对象：设置URL、Header信息、请求体信息

- GET 请求：

```java
Request request = new Request.Builder()
	.url("...")
	.addHeader("...", "...")
	.get()
	.build();
```

- POST 请求：

```java
Request request = new Request.Builder()
	.url("...")
	.addHeader("...", "...")
	.post(RequestBody)
	.build();
```

- DELETE 请求：

```java
Request request = new Request.Builder()
	.url("...")
	.addHeader("...", "...")
	.delete(RequestBody) // RequestBody 是可选参数
	.build();
```

- PUT 请求：

```java
Request request = new Request.Builder()
	.url("...")
	.addHeader("...", "...")
	.put(RequestBody)
	.build();
```

3、使用客户端对象执行请求

```java
Call call = CLIENT.newCall(request);
```

4、获取到 Response 对象

同步请求：

```java
try (Response execute = call.execute()) {  
    if (execute.isSuccessful()) { // 请求执行成功  
        // HTTP Code  
        int code = execute.code();  
  
        // 响应体  
        ResponseBody response = execute.body();  
        if (response != null) {  
            // 消息长度  
            long length = response.contentLength();  
  
            // 消息媒体类型  
            MediaType mediaType = response.contentType();  
  
            // 可以使用下面任意一种方式获取消息体  
  
            String string = response.string(); // 字符串  
            byte[] bytes = response.bytes(); // 字节数组  
            InputStream in = response.byteStream(); // 字节流  
            Reader reader = response.charStream(); // 字符流  
        }  
    }  
} catch (Exception e) {  
    // handler exception  
}
```

异步请求：

```java
call.enqueue(new Callback() {  
    @Override  
    public void onFailure(@NotNull Call call, @NotNull IOException e) {  
        LOGGER.error("request fail: {}, exception: {}", call.request(), e.getMessage(), e);  
    }  
  
    @Override  
    public void onResponse(@NotNull Call call, @NotNull Response response) throws IOException {  
        if (response.isSuccessful()) {  
            // 与同步请求一样，继续处理 response        } else {  
            // 请求失败处理  
        }  
    }  
});
```

5、释放资源

# GET 请求

GET 请求比较简单，只需要创建一个 Request 对象即可，实例：

```java
Request request = new Request.Builder()
        .url("http://localhost:8080/user?id=1")
        .addHeader("...", "...")
        .get()
        .build();

Call call = CLIENT.newCall(request);
```

# POST 请求

POST 请求相对于 GET 请求唯一多的一步就是构造请求体数据：

```java
Request request = new Request.Builder()
	.url("http://localhost:8080/")
	.addHeader("...", "...")
	.post(RequestBody) // 注意这里
	.build();
```

okhttp3.RequestBody 类内部提供了多种静态方法，用于创建请求体数据的形式：

```java
public static RequestBody create(final MediaType contentType, final byte[] content);
public static RequestBody create(final MediaType contentType, final byte[] content, final int offset, final int byteCount);
public static RequestBody create(final MediaType contentType, final ByteString content);
public static RequestBody create(final MediaType contentType, final byte[] content);
public static RequestBody create(final MediaType contentType, final byte[] content, final int offset, final int byteCount);
public static RequestBody create(final MediaType contentType, final File file);
```

其中 okhttp3.MediaType 对象是用于设置请求体类型，比如我想要请求的是 JSON 格式请求体就可以使用如下形式：

```java
String plaintext = "{...}";
RequestBody requestBody = RequestBody.create(MediaType.parse("application/json;charset=utf-8"), plaintext);
```

之后设置到 okhttp3.Request 对象即可：

```java
Request request = new Request.Builder()
                .url("http://localhost:8080/")
                .addHeader("...", "...")
                .post(requestBody) // 注意这里
                .build();
```

这些静态方法基本上满足我们日常使用了。不过，okhttp 还提供了两个 okhttp3.RequestBody 的扩展类，分别是用于文件上传的 `okhttp3.MultipartBody` 和 Form 表单数据 `okhttp3.FormBody`：

![RequestBody.png](http://java-media.knowledge.ituknown.cn/okhttp/RequestBody.png)

## JSON 请求

```java
User user = new User();
user.setUsername("HanMeimei");
user.setAge(18);

ObjectMapper objectMapper = new ObjectMapper();
String plaintext = objectMapper.writeValueAsString(user);

// 创建 JSON 请求体数据
RequestBody requestBody = RequestBody.create(MediaType.parse("application/json;charset=utf-8"), plaintext);

// 请求
Request request = new Request.Builder()
    	.url("http://localhost:8080/")
    	.addHeader("...", "...")
    	.post(requestBody) // 请求体数据
    	.build();

Call call = CLIENT.newCall(request);
```

## 表单请求

Form 表单请求使用的 Media 类型是 `application/x-www-form-urlencoded`。okhttp 提供了一个对应的 Form 表单请求体类：`FormBody`。我们可以直接使用该对象构造 Form 表单数据即可：

```java
FormBody requestBody = new FormBody.Builder()
        .add("username", "HanMeimei")
        .addEncoded("charset", "UTF-8")
        .build();

// 请求
Request request = new Request.Builder()
        .url("http://localhost:8080/")
        .addHeader("...", "...")
        .post(requestBody)
        .build();

Call call = CLIENT.newCall(request);
```

需要说明的是，`FormBody` 对象内部设置了一个默认 MediaType：

```java
private static final MediaType CONTENT_TYPE = MediaType.get("application/x-www-form-urlencoded");
```

所以我们在发送请求是无需手动设置 MediaType。

## 文件上传

okhttp 同样提供了一个用于构造文件上传的请求体类：`MultipartBody`。我们可以直接借助该类构造文件上传对象，示例如下：

```java
// 文件
File file = new File("/path/RequestBody.png");
String fileName = file.getName();

MultipartBody multipartBody = new MultipartBody.Builder()
        // 后台接收的 key, 文件名称, 文件对象
        .addFormDataPart("file", fileName, RequestBody.create(MediaType.parse("image/png"), file))
        .build();

Request request = new Request.Builder()
        .url("http://localhost:8080/upload")
        .post(multipartBody)
        .build();

Call call = CLIENT.newCall(request);
```

# 文件下载

文件下载就是普通的 GET 请求，唯一的区别就是注意响应数据的处理。进行文件下载时，我们应该获取响应数据的二进制流数据而不是文本字符串，然后再根据具体的媒体进行进行响应的处理即可：

```java
Request request = new Request.Builder()
        .url("http://localhost:8080/head?id=1")
        .addHeader("Accept", "application/json")
        .get()
        .build();

// 以同步请求为例
Response response = CLIENT.newCall(request).execute();

int code = response.code();  // 状态码
Protocol protocol = response.protocol(); // 请求协议
ResponseBody body = response.body();  // 响应体对象

// 注意这里
InputStream inputStream = body.byteStream(); // 二进制流数据
MediaType mediaType = body.contentType(); // 二进制流的媒体类型
```

# 资源链接

[https://github.com/square/okhttp](https://github.com/square/okhttp)

[https://square.github.io/okhttp/](https://square.github.io/okhttp/)

[https://stackoverflow.com/questions/tagged/okhttp?sort=active](https://stackoverflow.com/questions/tagged/okhttp?sort=active)

[https://springboot.io/t/topic/154](https://springboot.io/t/topic/154)
