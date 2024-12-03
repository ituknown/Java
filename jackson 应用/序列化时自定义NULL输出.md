
```java
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.JavaType;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;
import com.fasterxml.jackson.databind.ser.BeanPropertyWriter;

import java.io.IOException;
import java.math.BigDecimal;

public class NullSerializer extends JsonSerializer<Object> {

    private final BeanPropertyWriter property;

    public NullSerializer(BeanPropertyWriter property) {
        this.property = property;
    }

    @Override
    public void serialize(Object value, JsonGenerator gen, SerializerProvider provider) throws IOException {
        // 非 null 值不处理
        if (value != null) {
            gen.writeObject(value);
            return;
        }

        JavaType javaType = property.getType();

        if (javaType.isArrayType() || javaType.isCollectionLikeType()) { // 处理数组和集合 []
            gen.writeStartArray();
            gen.writeEndArray();
        } else if (javaType.isMapLikeType()) { // 处理 Map {}
            gen.writeStartObject();
            gen.writeEndObject();
        } else if (javaType.isTypeOrSubTypeOf(String.class)) { // 处理 String
            gen.writeString("");
        } else if (javaType.isTypeOrSubTypeOf(BigDecimal.class)) { // 处理 BigDecimal
            gen.writeString("0");
        } else {
            gen.writeNull();
        }
    }
}
```





```java
import com.fasterxml.jackson.databind.BeanDescription;
import com.fasterxml.jackson.databind.SerializationConfig;
import com.fasterxml.jackson.databind.ser.BeanPropertyWriter;
import com.fasterxml.jackson.databind.ser.BeanSerializerModifier;

import java.util.List;

public class NullBeanSerializerModifier extends BeanSerializerModifier {

    @Override
    public List<BeanPropertyWriter> changeProperties(SerializationConfig config, BeanDescription beanDesc, List<BeanPropertyWriter> beanProperties) {
        // 当值为 null 时, 使用自定义序列化处理类 NullSerializer
        beanProperties.forEach(p -> p.assignNullSerializer(new NullSerializer(p)));
        return beanProperties;
    }
}
```



```java
ObjectMapper objectMapper = new ObjectMapper();

// 处理 Null 值
objectMapper.setSerializerFactory(objectMapper.getSerializerFactory().withSerializerModifier(new NullBeanSerializerModifier()));
```



```java
public class User {

    private String str;
    private String[] array;
    private List<Integer> collect;
    private Map<String, Object> map;
    private BigDecimal decimal;

    // Getter/Setter
}
```



```java
User user = new User();

System.out.println(objectMapper.writeValueAsString(user));
```



```json
{
  "str" : "",
  "array" : [ ],
  "collect" : [ ],
  "map" : { },
  "decimal" : "0"
}
```







```java
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface DecimalDefauleVal {
    String value() default "0.0";
}
```





```java
public class User {
    @DecimalDefauleVal("10")
    private BigDecimal decimal;
    // ...
}
```



```java
@Override
public void serialize(Object value, JsonGenerator gen, SerializerProvider provider) throws IOException {
    DecimalDefauleVal annotation = property.getAnnotation(DecimalDefauleVal.class);
    if (annotation != null) {
        gen.writeString(annotation.value());
    } else {
        // ...
    }
}
```





```java
{
  "str" : "",
  "array" : [ ],
  "collect" : [ ],
  "map" : { },
  "decimal" : "10"
}
```

