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

# 忽略 transient 字段

Jackson 在序列化时默认不会忽略被 transient 修饰的属性字段。如果想要忽略该字段需要做如下配置：

```java
ObjectMapper objectMapper = new ObjectMapper();

// 忽略 transient 字段  
objectMapper.configure(MapperFeature.PROPAGATE_TRANSIENT_MARKER, true);
```

# 忽略NULL或空字段

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

# 忽略指定字段

## 使用 JsonIgnoreProperties

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

## 使用 FilterProvider

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

# 输出到本地文件

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

# 输出全部字段

想要输出全部字段（包括值为 null 值字段）有两种实现方式：局部配置以及全局配置。局部配置和全局配置各有优缺点，不过如果同时配置的话局部配置的优先级更高。

局部配置可以使用 `@JsonInclude` 注解，这个注解可以应用于类级别或字段级别。下面是一个示例：

```java
import com.fasterxml.jackson.annotation.JsonInclude;

@JsonInclude(JsonInclude.Include.ALWAYS) // 序列化时始终包括 null 值的字段
public class MyObject {
    private String field1;
    private Integer field2;
    // ...
}
```

全局配置则显得简单直接，不过在某些时候将 null 字段也输出出来总显得有些多余。如何取舍看具体应用场景吧：

```java
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;

ObjectMapper objectMapper = new ObjectMapper();

// 2.9 之前可以使用该配置实现输出全部字段
objectMapper.configure(SerializationFeature.WRITE_NULL_MAP_VALUES, true);

// 2.9 及之后推荐使用下面两种配置方式实现输出全部字段
objectMapper.setSerializationInclusion(JsonInclude.Include.ALWAYS);
// 或
objectMapper.setDefaultPropertyInclusion(JsonInclude.Include.ALWAYS);
```

# 格式化输出

格式化输出 JSON 数据，可以通过配置是否启用缩进来实现：

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;

ObjectMapper objectMapper = new ObjectMapper();
objectMapper.configure(SerializationFeature.INDENT_OUTPUT, true); // 启用缩进
```

下面是示例代码：

```java
public class User {
  private String name;
  private int age;
  // 省略 ...
}

User user = new User("John", 30);

String prettyJson = objectMapper.writeValueAsString(user);
System.out.println(prettyJson);
```

输出结果：

```json
{
  "name" : "John",
  "age" : 30
}
```

关于格式化输出有一点需要注意，Jackson 提供了两种输出方式：

- com.fasterxml.jackson.core.util.DefaultPrettyPrinter 
- com.fasterxml.jackson.core.util.MinimalPrettyPrinter

DefaultPrettyPrinter 是缺省时的默认输出方式，它支持缩进输出。即前面的示例配置与下面相等：

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.core.util.DefaultPrettyPrinter;

ObjectMapper objectMapper = new ObjectMapper();
objectMapper.configure(SerializationFeature.INDENT_OUTPUT, true);
objectMapper.setDefaultPrettyPrinter(new DefaultPrettyPrinter()); // 缺省时默认的输出方式, 支持缩进输出
```

而 MinimalPrettyPrinter 与 DefaultPrettyPrinter 的区别是它不支持缩进输出：

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.core.util.MinimalPrettyPrinter;

ObjectMapper objectMapper = new ObjectMapper();
objectMapper.configure(SerializationFeature.INDENT_OUTPUT, true); // 无效
objectMapper.setDefaultPrettyPrinter(new MinimalPrettyPrinter());
```

即使明确配置了缩进输出，它也会忽略该配置：

```json
{"name":"John","age":30}
```

在实际项目中，对于有签名校验需求的场景还是明确配置使用 com.fasterxml.jackson.core.util.MinimalPrettyPrinter 作为输出方式比较好，因为它可以明确杜绝多余的换行符。

# Java8日期序列化问题

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

你会发现 `date` 字段输出的日期感觉有点反人类是不？这是 Jackson 对 Java8 日期序列化的问题，解决方式见 [Java8日期格式问题](../java8日期格式问题.md)。

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
