# 写在前面

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

# 对象转字符串

将一个对象转JSON字符串(通常称为对象序列化)主要使用 `ObjectMapper` 的 `writeValueAsString()` 方法，将需要转换的对象转入即可（可以是一个普通的类对象，也可以是一个集合和 Map 对象）如下：

```java
public String writeValueAsString(Object value);
```

当然，Jackson 除了能够将对象转换为 Json 字符串还能转换为二进制流或输出到文件。

现在来看下如何将对象转换为 JSON 字符串，声明一个 User 类：

```java
public class User {

    private String name;

    private Integer age;

    private LocalDate date;

    private LocalTime time;

    private LocalDateTime dateTime;

    private List<String> tags;

    // Getter And Setter, 下文略...
}
```

直接使用 `ObjectMapper#writeValueAsString()` 方法就能够实现 JSON 转换：

```java
ObjectMapper objectMapper = new ObjectMapper();

User user = User.builder().name("张三").age(18).build();

// user 对象转 JSON 字符串
String json = objectMapper.writeValueAsString(user);

System.out.println(json);
```

输出结果：

```json
{
    "name": "张三",
    "age": 18,
    "date": null,
    "time": null,
    "dateTime": null,
    "tags": null
}
```

## 忽略NULL或空字段

上面的输出结果中包含了值为 NULL 的字段，如果想要忽略值为 NULL 的字段可以使用 `ObjectMapper#setSerializationInclusion()` API，如下：

```java
public ObjectMapper setSerializationInclusion(JsonInclude.Include incl);
```

`JsonInclude.Include` 枚举类定义如下：

```java
public enum Include {
    /**
     * 包含所有字段
     */
    ALWAYS,

    /**
     * 忽略值为 NULL 的字段
     */
    NON_NULL,

    NON_ABSENT,

    /**
     * 忽略值为 空 的字段. 如字符串: "", 数组或集合: [].
     */
    NON_EMPTY,

    NON_DEFAULT,

    CUSTOM,

    USE_DEFAULTS;
}
```

如果不显示的设置 `JsonInclude.Include` ，它的默认值是 `ALWAYS`。

如果要忽略值为 NULL 的字段，直接设置枚举值 `JsonInclude.Include.NON_NULL` 即可，如下：

```java
ObjectMapper objectMapper = new ObjectMapper();
// 忽略值为 NULL 的字段
objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);

// 忽略值为 "空" 的字段. 如字符串: "", 数组或集合: []
// objectMapper.setSerializationInclusion(JsonInclude.Include.NON_EMPTY);

User user = User.builder().name("张三").age(18).build();

String json = objectMapper.writeValueAsString(user);

System.out.println(json);
```

输出结果：

```json
{
    "name": "张三",
    "age": 18
}
```

| **注意** |
| :--- |
| 如果不显示的调用  `ObjectMapper#setSerializationInclusion()` 方法进行设置 `JsonInclude.Include` ，那么它的默认值是 `ALWAYS`，即默认会序列化类中的所有属性字段（不包括 `static`）。 |

## `@JsonIgnoreProperties` 忽略指定字段

`com.fasterxml.jackson.annotation.JsonIgnoreProperties#value` 接受一个数组字符串，每个字符串代表类中的一个属性字段，将想要忽略的字段以字符串数组的形式写上即可，如下：

```java
@JsonIgnoreProperties({"name", "age"})
public class User {

    private String name;

    private Integer age;

    private LocalDate date;

    private LocalTime time;

    private LocalDateTime dateTime;

    private List<String> tags;
}
```

`@JsonIgnoreProperties` 注解不仅可以作用于类上，还可以直接写在某个属性上，表示在序列化反序列化是忽略该字段：

```java
public class User {

    @JsonIgnoreProperties
    private String name;

    private Integer age;

    private LocalDate date;

    private LocalTime time;

    private LocalDateTime dateTime;

    private List<String> tags;
}
```

## 使用 FilterProvider 忽略指定字段

该 `FilterProvider` 是最不推荐的使用方式，因为冗余性太强。看下使用示例吧：

```java
ObjectMapper objectMapper = new ObjectMapper();
// 除了 name 字段都忽略
SimpleBeanPropertyFilter propertyFilter = SimpleBeanPropertyFilter.filterOutAllExcept("name");
filterProvider.addFilter("userPropertyFilter", propertyFilter);
objectMapper.setFilterProvider(filterProvider);

User user = User.builder().name("张三").age(18).build();

String json = objectMapper.writeValueAsString(user);

System.out.println(json);
```

注意 `SimpleBeanPropertyFilter.filterOutAllExcept()` API，该 API 指的是除了指定字段都忽略。即除了我们输入的 `name` 字段都被忽略掉，感觉有点反人类。

之后我们设置了一个过滤器ID：userPropertyFilter，我们还需要在 User 类上使用 `@JsonFilter("userPropertyFilter")` 注解，值就是我们上面设置的值：

```java
@JsonFilter("userPropertyFilter")
public class User {
    // ...
}
```

现在运行才能达到我们的效果：

```json
{"name":"张三"}
```

但，怎么说了...... 这个 API 用起来很难受反正。

## 忽略 transient 字段

Jackson 在序列化时默认不会忽略被 transient 修饰的属性字段。如果想要忽略该字段需要做如下配置：

```java
ObjectMapper objectMapper = new ObjectMapper();

// 忽略 transient 字段  
objectMapper.configure(MapperFeature.PROPAGATE_TRANSIENT_MARKER, true);
```

## Java8 日期序列化问题

现在来再看一个示例，代码如下：

```java
ObjectMapper objectMapper = new ObjectMapper();

// 注意 date 字段值
User user = User.builder().name("张三").age(18).date(LocalDate.now()).build();

String json = objectMapper.writeValueAsString(user);

System.out.println(json);
```

输出如下：

```json
{
    "name": "张三",
    "age": 18,
    "date": {
        "year": 2021,
        "month": "JULY",
        "monthValue": 7,
        "dayOfMonth": 26,
        "chronology": {
            "id": "ISO",
            "calendarType": "iso8601"
        },
        "era": "CE",
        "dayOfYear": 207,
        "dayOfWeek": "MONDAY",
        "leapYear": false
    }
}
```

你会发现 `date` 字段输出的日期感觉有点反人类是不？这是 Jackson 对 Java8 日期序列化的问题。

# 对象序列化到本地文件

Jackson 不仅仅能够将一个对象序列化成一个 Json 字符串，还能够将序列化的结果输出到本地文件、IO流以及二进制数据等等。

ObjectMapper 提供了如下方法：

```java
public void writeValue(DataOutput out, Object value);
public void writeValue(Writer w, Object value);
public byte[] writeValueAsBytes(Object value);
public void writeValue(File resultFile, Object value);

```

对象系列化输出到本地文件示例：

```java
ObjectMapper objectMapper = new ObjectMapper();

File file = new File("/Users/xx/Desktop/test.txt");

User user = User.builder().name("张三").age(18).date(LocalDate.now()).build();

objectMapper.writeValue(file, user);
```

# 字符串转对象

JSON字符串转对象(通常称为反序列化)主要使用的是 `ObjectMapper#readValue` 方法。Jackson 提供了多种重载方法，这些方法都定义在 ObjectMapper 类中，下面是 String Json 字符串的三种重载 API：

```java
public <T> T readValue(String content, Class<T> valueType);
public <T> T readValue(String content, JavaType valueType);
public <T> T readValue(String content, TypeReference valueTypeRef);
```

ObjectMapper 类中除了定义 String 的重载 API 还定义了多种流的重载 API，见下文。

上面方法中的 `valueType` 就是我们要转换的对象类型，可以看到有两种重载方式，分别是 `Class` 和 `JavaType`，单对象转换我使用 `Class` 作为参数的比较多。

`TypeReference` 相比较 `Class` 更加强大，因为 `Class` 只能接受单类型对象，如果想要将 Json 字符串转换为集合或者 Map 对象 `Class` 是实现不了的，这种情况下只能使用 `TypeReference`。下面还是具体说明：

## 字符串转对象

话不多说，直接上代码：

```java
String jsonStr = "{\"name\":\"张三\",\"age\":18,\"date\":null,\"time\":null,\"dateTime\":null,\"tags\":null}";

ObjectMapper objectMapper = new ObjectMapper();

User user = objectMapper.readValue(jsonStr, User.class);
System.out.println(user);
```


上面的示例使用的是 Class 重载方法 `readValue(String content, Class<T> valueType)`。

另外我们还可以使用 TypeReference 重载方法 `readValue(String content, TypeReference valueTypeRef)` 来实现相同的效果：

```java
String jsonStr = "{\"name\":\"张三\",\"age\":18,\"date\":null,\"time\":null,\"dateTime\":null,\"tags\":null}";

ObjectMapper objectMapper = new ObjectMapper();

// 注意 TypeReference 的用法
User user = objectMapper.readValue(jsonStr, new TypeReference<User>(){});
System.out.println(user);
```

不过实际使用时并不推荐使用 `TypeReference`，因为 Java 的泛型类型擦除机制使得在运行时无法获取具体的类型信息。

## 字符串转集合

在实际使用中我们比较多的应用就是 JSON 数组形式的字符串转集合，想要解决这个问题就不能使用 `Class` 重载方法了，取而代之的是使用 TypeFactory：

```java
String jsonStr = "[{\"name\":\"张三\",\"age\":18,\"date\":null,\"time\":null,\"dateTime\":null,\"tags\":null},{\"name\":\"李四\",\"age\":20,\"date\":null,\"time\":null,\"dateTime\":null,\"tags\":null}]";

CollectionType collectionType = objectMapper.getTypeFactory().constructCollectionType(List.class, User.class);

List<User> userList = objectMapper.readValue(json, collectionType);
```

不过也可以使用 `TypeReference` 实现（虽然不推荐）。看下示例：

```java
// 注意 TypeReference 的用法
List<User> user = objectMapper.readValue(jsonStr, new TypeReference<List<User>>(){});
```

封装一个转任意集合的方法：

```java
@SuppressWarnings("unchecked")
public static <E> ArrayList<E> toCollection(String json, Class<E> element) {
    return toCollection(json, ArrayList.class, element);
}

public static <C extends Collection<E>, E> C toCollection(String json, Class<C> collection, Class<E> element) {
    try {
        CollectionType type = MAPPER.getTypeFactory().constructCollectionType(collection, element);
        return MAPPER.readValue(json, type);
    } catch (Exception e) {
        throw new RuntimeException("Parse json to Collection<E> failed:" + json, e);
    }
}
```

使用示例：

```java
@SuppressWarnings("unchecked")  
List<User> userList = toCollection(json, ArrayList.class, User.class);
```

## 字符串转Map

同样了，字符串转 Map 也不能少：

```java
String jsonStr = "[{\"name\":\"张三\",\"age\":18,\"date\":null,\"time\":null,\"dateTime\":null,\"tags\":null},{\"name\":\"李四\",\"age\":20,\"date\":null,\"time\":null,\"dateTime\":null,\"tags\":null}]";

MapType mapType = objectMapper.getTypeFactory().constructMapType(Map.class, String.class, String.class);

Map<String, String> map = objectMapper.readValue(jsonStr, mapType);
```

我们同样需要使用 `TypeReference` API：

```java
// 注意 TypeReference 的用法
Map<String, String> map = objectMapper.readValue(jsonStr, new TypeReference<Map<String, String>>() {});
```

可以进一步的封装成工具类使用：

```java
@SuppressWarnings("unchecked")
public static <K, V> HashMap<K, V> toMap(String json, Class<K> key, Class<V> value) {
    return toMap(json, HashMap.class, key, value);
}

public static <K, V, H extends Map<K, V>> H toMap(String json, Class<H> map, Class<K> key, Class<V> value) {
    try {
        MapType type = MAPPER.getTypeFactory().constructMapType(map, key, value);
        return MAPPER.readValue(json, type);
    } catch (Exception e) {
        throw new RuntimeException("Parse json to Map<K, V> failed:" + json, e);
    }
}
```

继续封装一个 `Collection<Map>` 方法：

```java
@SuppressWarnings("unchecked")
public static <K, V> ArrayList<HashMap<K, V>> toCollectionMap(String json, Class<K> key, Class<V> value) {
    return toCollectionMap(json, ArrayList.class, HashMap.class, key, value);
}

public static <K, V, H extends Map<K, V>, C extends Collection<H>> C toCollectionMap(String json, Class<C> collection, Class<H> map, Class<K> key, Class<V> value) {
    try {
        MapType mapType = MAPPER.getTypeFactory().constructMapType(map, key, value);
        CollectionType collectionType = MAPPER.getTypeFactory().constructCollectionType(collection, mapType);
        return MAPPER.readValue(json, collectionType);
    } catch (Exception e) {
        throw new RuntimeException("Parse json to Collection<Map<K, V>>> failed:" + json, e);
    }
}
```

## Java8 日期反序列化问题

TypeReference 看起来很强，但是有个数据它是解决不了的，那就是 Java8 的日期 API。比如下面的 JSON：

```json
{
    "name": "张三",
    "age": 18,
    "date": null,
    "time": null,
    "dateTime": "2021-07-26 18:35:02",
    "tags": null
}
```

当你尝试转换成 User 对象时会提示如下错误：

![deserializationJava8DateFail1708509575.png](http://java-media.knowledge.ituknown.cn/jackson/deserializationJava8DateFail1708509575.png)

归根结底，这是 Jackson 对 Java8 日期格式化得问题，解决方式见下文 [Java8 日期格式问题](#java8-日期格式问题)。

# 文件转对象

Jackson 不仅能够将普通的 JSON 字符串反序列化为对象，还能够直接从文件中读取 JSON 内容反序列化成对象。

ObjectMapper 提供的 File 三种重载方法：

```java
public <T> T readValue(File src, Class<T> valueType);
public <T> T readValue(File src, JavaType valueType);
public <T> T readValue(File src, TypeReference valueTypeRef);
```

用法与 [JSON字符串转对象](#字符串转对象) 一致，看下下面的示例：

```java
ObjectMapper objectMapper = new ObjectMapper();
User user = objectMapper.readValue(new File("/Users/Desktop/test.json"), User.class);
```

需要说明的是文件并不限制一定是 json 文件，只要文件内容是 JSON 格式就能够正常解析，比如 txt 文件：

```java
ObjectMapper objectMapper = new ObjectMapper();
User user = objectMapper.readValue(new File("/Users/Desktop/test.txt"), User.class);
```

# 网络文件转对象

Jackson 同样提供了从网络文件（URL）读取 JSON 内容的 API，ObjectMapper URL 的三种重载方法定义如下：

```java
public <T> T readValue(URL src, Class<T> valueType);
public <T> T readValue(URL src, JavaType valueType);
public <T> T readValue(URL src, TypeReference valueTypeRef);
```

直接看示例：

```java
ObjectMapper objectMapper = new ObjectMapper();
User user = objectMapper.readValue(new URL("file:/Users/Desktop/test.txt"), User.class);
```

# 二进制流转对象

既然能够从文件和网络中获取 JSON 内容，肯定也能够解析 IO 流：

```java
public <T> T readValue(InputStream src, Class<T> valueType);
public <T> T readValue(InputStream src, JavaType valueType);
public <T> T readValue(InputStream src, TypeReference valueTypeRef);
```

示例：

```java
ObjectMapper objectMapper = new ObjectMapper();
User user = objectMapper.readValue(new FileInputStream(new File("/Users/Desktop/test.txt")), User.class);
```

# Java8 日期格式问题

先看下示例：

```java
ObjectMapper objectMapper = new ObjectMapper();

User user = User.builder().name("张三").age(18)
    	        .time(LocalTime.now())
                .date(LocalDate.now())
                .dateTime(LocalDateTime.now())
                .build();

String json = objectMapper.writeValueAsString(user);
```

输出结果为：

```json
{
    "name": "张三",
    "age": 18,
    "date": {
        "year": 2021,
        "month": "JULY",
        "era": "CE",
        "dayOfYear": 207,
        "dayOfWeek": "MONDAY",
        "leapYear": false,
        "dayOfMonth": 26,
        "monthValue": 7,
        "chronology": {
            "id": "ISO",
            "calendarType": "iso8601"
        }
    },
    "time": {
        "hour": 13,
        "minute": 38,
        "second": 52,
        "nano": 514000000
    },
    "dateTime": {
        "dayOfYear": 207,
        "dayOfWeek": "MONDAY",
        "month": "JULY",
        "dayOfMonth": 26,
        "year": 2021,
        "monthValue": 7,
        "hour": 13,
        "minute": 38,
        "second": 52,
        "nano": 515000000,
        "chronology": {
            "id": "ISO",
            "calendarType": "iso8601"
        }
    },
    "tags": null
}
```

注意看 `date`、`time` 和 `dateTime` 字段，这个输出信息与我们预想的似乎不太一样。

解决该问题主要有两种方式：使用 Jackson 的 JavaTimeModule 或者 使用自定义序列化反序列化方式。分别来看下：

## 使用 JavaTimeModule 方式(推荐)

想要使用 JavaTimeModule 解决 Java8 日期格式化问题我们需要引入 Jackson 的 jsr310 依赖：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
    <version>${jaskon.version}</version>
</dependency>
```

这样，我们就能够创建一个 JavaTimeModule 对象了：

```java
JavaTimeModule javaTimeModule = new JavaTimeModule();
```

当然我们还是要做些相应的配置才行，不过最终这个 `JavaTimeModule` 对象是要注册到 `ObjectMapper` 对象中的，如下：

```java
public ObjectMapper registerModule(Module module);
```

所以，为了方便我们还是创建一个静态方法来配置 `JavaTimeModule`，如下：

```java
private static void configureObjectMapper4Jsr310(ObjectMapper objectMapper) {

    JavaTimeModule javaTimeModule = new JavaTimeModule();

    // config JavaTimeModule ...

    objectMapper.registerModule(javaTimeModule);

}
```

这样做的主要原因是 JavaTimeModule 是 Module 的一个子实现，也就是说在实际使用中我们可能还会为其他子实现进行定制化配置。通过将配置进行公共提取，当其他 ObjectMapper 也需要相应配置时直接调用该方法进行注册即可。

现在就来配置 JavaTimeModule 来解决 Java8 日期格式的问题。

### 配置序列化和反序列化

先看下 JavaTimeModule 类继承图：

![JavaTimeModule.png](http://java-media.knowledge.ituknown.cn/jackson/JavaTimeModule1708509985.png)

配置日期格式问题我们主要借助它的两个方法：

```java
// 序列化使用
public <T> SimpleModule addSerializer(Class<? extends T> type, JsonSerializer<T> ser);

// 反序列化使用
public <T> SimpleModule addDeserializer(Class<T> type, JsonDeserializer<? extends T> deser);
```

这两个方法都是在父类中的 SimpleModule 中定义。

形参 `type` 指的是我们要序列化的类型，如 `LocalDateTime.class`。

形参 `ser` 和 `deser` 指的是我们序列化和反序列化的具体实现方式，比如我们要配置 `LocalDateTime.class` 的序列化和反序列化方法，需要传递的序列化和反序列化对象就是 `LocalDateTimeSerializer` 和 `LocalDateTimeDeserializer`。

具体就不做过多说明了，现在来看下该具体配置吧，直接上代码：

```java
private static void configureObjectMapper4Jsr310(ObjectMapper objectMapper) {

    JavaTimeModule javaTimeModule = new JavaTimeModule();

    // LocalTime 序列化和反序列化配置
    DateTimeFormatter timeFormatter = DateTimeFormatter.ofPattern("HH:mm:ss");
    javaTimeModule.addSerializer(LocalTime.class, new LocalTimeSerializer(timeFormatter));
    javaTimeModule.addDeserializer(LocalTime.class, new LocalTimeDeserializer(timeFormatter));

    // LocalDate 序列化和反序列化配置
    DateTimeFormatter dateFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
    javaTimeModule.addSerializer(LocalDate.class, new LocalDateSerializer(dateFormatter));
    javaTimeModule.addDeserializer(LocalDate.class, new LocalDateDeserializer(dateFormatter));

    // LocalDateTime 序列化和反序列化配置
    DateTimeFormatter datetimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    javaTimeModule.addSerializer(LocalDateTime.class, new LocalDateTimeSerializer(datetimeFormatter));
    javaTimeModule.addDeserializer(LocalDateTime.class, new LocalDateTimeDeserializer(datetimeFormatter));

    objectMapper.registerModule(javaTimeModule);

}
```

这样，Java8 日期格式化的配置就完成了。

### 日期序列化测试

```java
ObjectMapper objectMapper = new ObjectMapper();
// 配置 jsr310 日期序列化和反序列化问题
configureObjectMapper4Jsr310(objectMapper);

User user = User.builder().name("张三").age(18)
    			.time(LocalTime.now())
                .date(LocalDate.now())
                .dateTime(LocalDateTime.now())
                .build();

String json = objectMapper.writeValueAsString(user);
```

输出结果：

```json
{
    "name": "张三",
    "age": 18,
    "date": "2021-07-26",
    "time": "15:27:16",
    "dateTime": "2021-07-26 15:27:16",
    "tags": null
}
```

这回日期就是我们想要的输出格式了~

### 日期反序列化测试

```java
String jsonStr = "{\"name\":\"张三\",\"age\":18,\"date\":null,\"time\":null,\"dateTime\":\"2021-07-26 18:35:02\",\"tags\":null}";

ObjectMapper objectMapper = new ObjectMapper();
// 配置 jsr310 日期序列化和反序列化问题
configureObjectMapper4Jsr310(objectMapper);

User user = objectMapper.readValue(jsonStr, User.class);
System.out.println(user);
```

看下截图：

![HandlerJava8DateTime1708510099.png](http://java-media.knowledge.ituknown.cn/jackson/HandlerJava8DateTime1708510099.png)

这就完结解决 Java8 日期格式的问题了~

## 自定义序列化和反序列化方式

注意，这种配置方式与上面的 JavaTimeModule 本质上是一样的。主要是扩展 `com.fasterxml.jackson.databind.JsonSerializer` 和 `com.fasterxml.jackson.databind.JsonDeserializer` 来实现自定义某个类的序列化问题。

话不多说，直接上代码：

### 日期序列化配置

序列化 LocalDate：
```java
public class LocalDateJsonSerializer extends JsonSerializer<LocalDate> {

    @Override
    public void serialize(LocalDate date, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        gen.writeString(date.format(DateTimeFormatter.ofPattern("yyyy-MM-dd", Locale.CHINA)));
    }
}
```

序列化 LocalTime：
```java
public class LocalTimeJsonSerializer extends JsonSerializer<LocalTime> {
    @Override
    public void serialize(LocalTime time, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        gen.writeString(time.format(DateTimeFormatter.ofPattern("HH:mm:ss", Locale.CHINA)));
    }
}
```

序列化 LocalDateTime：
```java
public class LocalTimeJsonSerializer extends JsonSerializer<LocalDateTime> {
    @Override
    public void serialize(LocalDateTime datetime, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        gen.writeString(datetime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss", Locale.CHINA)));
    }
}
```

### 日期反序列化配置

反序列化LocalDate：
```java
public class LocalDateJsonDeserializer extends JsonDeserializer<LocalDate> {
    @Override
    public LocalDate deserialize(JsonParser parser, DeserializationContext context) throws IOException {
        return LocalDate.parse(parser.getText(), DateTimeFormatter.ofPattern("yyyy-MM-dd"));
    }
}
```

反序列化LocalTime：
```java
public class LocalTimeJsonDeserializer extends JsonDeserializer<LocalTime> {

    @Override
    public LocalTime deserialize(JsonParser parser, DeserializationContext context) throws IOException {
        return LocalTime.parse(parser.getText(), DateTimeFormatter.ofPattern("HH:mm:ss"));
    }
}
```

反序列化LocalDateTime：
```java
public class LocalDateTimeJsonDeserializer extends JsonDeserializer<LocalDateTime> {
    @Override
    public LocalDateTime deserialize(JsonParser parser, DeserializationContext context) throws IOException {
        return LocalDateTime.parse(parser.getText(), DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
    }
}
```

### 日期序列化测试

想要使用上面我们自定义的序列化和反序列化配置主要有两种方式。第一种比较简单，借助 Jackson 提供的 `@JsonSerialize` 和 `@JsonDeserialize` 注解。如下：

```java
public class User {

    private String name;

    private Integer age;

    @JsonSerialize(using = LocalDateJsonSerializer.class)
    @JsonDeserialize(using = LocalDateJsonDeserializer.class)
    private LocalDate date;

    @JsonSerialize(using = LocalTimeJsonSerializer.class)
    @JsonDeserialize(using = LocalTimeJsonDeserializer.class)
    private LocalTime time;

    @JsonSerialize(using = LocalDateTimeJsonSerializer.class)
    @JsonDeserialize(using = LocalDateTimeJsonDeserializer.class)
    private LocalDateTime dateTime;

    private List<String> tags;
}
```

测试一下：

```java
ObjectMapper objectMapper = new ObjectMapper();

User user = User.builder().name("张三").age(18)
    			.time(LocalTime.now())
                .date(LocalDate.now())
                .dateTime(LocalDateTime.now())
                .build();

String json = objectMapper.writeValueAsString(user);
```

输出结果：

```json
{
    "name": "张三",
    "age": 18,
    "date": "2021-07-26",
    "time": "15:48:59",
    "dateTime": "2021-07-26 15:48:59",
    "tags": null
}
```

看起来似乎有点啰嗦，可能会有同学问直接使用 `@JsonFormat` 注解不行吗？如下：

```java
public class User {

    private String name;

    private Integer age;

    @JsonFormat(pattern = "yyyy-MM-dd")
    private LocalDate date;

    private LocalTime time;

    private LocalDateTime dateTime;

    private List<String> tags;
}
```

抱歉，不行的。`@JsonFormat` 配置形式只能格式化 `java.util.Date` 日期格式~

实际上，还有一种使用形式。不过这种主要是在 SpringMVC 的消息转换器中使用。因为需要借助 SpringMVC 的 Jackson 消息转换器 `org.springframework.http.converter.json.Jackson2ObjectMapperBuilder`。如下示例：

```java
private static void configureObjectMapper4Jsr310(ObjectMapper objectMapper) {

    Jackson2ObjectMapperBuilder mapperBuilder = new Jackson2ObjectMapperBuilder();
    mapperBuilder.serializerByType(LocalDate.class, new LocalDateJsonSerializer());
    mapperBuilder.serializerByType(LocalTime.class, new LocalTimeJsonSerializer());
    mapperBuilder.serializerByType(LocalDateTime.class, new LocalDateTimeJsonSerializer());

    mapperBuilder.deserializerByType(LocalDate.class, new LocalDateJsonDeserializer());
    mapperBuilder.deserializerByType(LocalTime.class, new LocalTimeJsonDeserializer());
    mapperBuilder.deserializerByType(LocalDateTime.class, new LocalDateTimeJsonDeserializer());

    mapperBuilder.configure(objectMapper);
}
```

反序列化就不做测试了~

# 使用 ObjectNode

有时候我们可能需要创建一个对象具有 Map 能够 put 一个 `Key - Value` 的功能，这个时候我们就需要使用 ObjectNode 对象：

```java
ObjectNode objectNode = objectMapper.createObjectNode();
```

需要说明一点：ObjectnNode 继承至 JsonNode。

有了 ObjectNode 对象我们就能够调用它的相关 put 对象进行设置一个 Key -Value 了。

看下示例：

```java
ObjectMapper objectMapper = new ObjectMapper();

ObjectNode objectNode = objectMapper.createObjectNode();
objectNode.put("money", new BigDecimal("100.00"));
```

不过在获取值得时候需要注意了，调用 `get(Key)` 方法返回的是一个 JsonNode 对象：

```java
JsonNode jsonNode = objectNode.get("money");
```

想要获取具体的数据类型需要相应的转换，比如上面设置的 money 是个 BigDecimal 类型对象，就需要调用 Decimal 方法进行转换：

```java
JsonNode jsonNode = objectNode.get("money");
BigDecimal money = jsonNode.decimalValue();
```

JsonNode 还有一个用法，就是将一个对象转换成 JsonNode：

```java
ObjectMapper objectMapper = new ObjectMapper();

User user = User.builder().name("张三").age(18).date(LocalDate.now()).build();

JsonNode jsonNode = objectMapper.valueToTree(user);
```

同理，也能够将一个 Json 字符串转换成 JsonNode 对象：

```java
ObjectMapper objectMapper = new ObjectMapper();

String jsonStr = "{\"name\":\"张三\",\"age\":18,\"date\":null,\"time\":null,\"dateTime\":null,\"tags\":null}";

JsonNode jsonNode = objectMapper.readTree(jsonStr);
```

特别强调的一点是：ObjectMapper 通常都是声明为全局对象，这个全局对象都是做过相应配置的。所以在创建 ObjectNode 时最好使用下面的方式：

```java
ObjectMapper objectMapper = new ObjectMapper();

// config objectMapper

ObjectNode objectNode = new ObjectNode(objectMapper.getNodeFactory());
```

# 使用 ArrayNode

ObjectNode 都有了怎么能没有 ArrayNode？

ObjectNode 的功能类似于 Map，而 ArrayNode 就是 List 了。

创建 ArrayNode 对象：

```java
ArrayNode arrayNode = objectMapper.createArrayNode();
```

推荐方式：

```java
ArrayNode arrayNode = new ArrayNode(objectMapper.getNodeFactory())
```

具体使用也就没必要介绍了。


完结，撒花~
