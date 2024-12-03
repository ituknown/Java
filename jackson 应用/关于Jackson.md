Jackson Github 地址：[https://github.com/FasterXML](https://github.com/FasterXML)

Jackson 是干什么的也没必要过多说明，相信各位应该都很熟悉。Jackson 和 GJson 以及 Fastjson 的性能比较啊，本篇也不做说明。因为啰里啰嗦的实在没意义，有兴趣的自行百度谷歌吧。

Jaskson 在实际中主要有两方面的应用：JSON 转换以及 XML 的转换，本篇介绍 JSON 字符串和对象之间的转换。

想要使用 Jaskson 需要在 pom 文件中引入它的依赖：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>${jaskon.version}</version>
</dependency>
```

| **说明** |
|:--- |
|在实际使用中我们只需要引入 `jackson-databind` 这个依赖即可，因为该依赖内嵌了 jaskson 核心依赖包和注解依赖包，在实际中基本上已经满足我们的需要了。|

想要使用 Jackson 的 JSON 转换 API 只需要声明一个 ObjectMapper 对象即可：

```java
ObjectMapper objectMapper = new ObjectMapper();
```

另外，ObjectMapper 是线程安全的类，所以为了运行效率我们通常声明一个全局的 Jackson 对象即可。如下：

```java
public final class JacksonUtil {

    private static final ObjectMapper OBJECT_MAPPER = new ObjectMapper();

    public static ObjectMapper getObjectMapper () {
        return OBJECT_MAPPER;
    }

}
```
