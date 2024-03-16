TypeHandler 是 MyBatis 中一个重要的组件，它的作用是将数据库中的数据类型与 Java 中的数据类型进行转换。

在开发过程中，当默认的 TypeHandler 无法满足需求时，就需要自己实现或扩展对应的 TypeHandler（例如遇到 MyBatis 不支持的数据类型或需要特殊处理的数据）。

在实际项目中我就遇到过在数据保存或者查询时需要脱敏的问题，就用到了自定义的 TypeHandler。

当然，TypeHandler 的作用还不仅仅如此，还可以处理 Null 值问题。在数据查询或者结果映射过程中，当数据库字段的值为NULL时，TypeHandler 会将其转换为 Java 中合适的表示方式，例如 Java 中的 null 或者空字符串。

想要自定义扩展 TypeHandler 可以实现 MyBatis 提供的接口 org.apache.ibatis.type.TypeHandler。不过官网文档建议使用 org.apache.ibatis.type.BaseTypeHandler。

下面是一个执行身份证号码保存和查询时脱敏的示例：

```java
import org.apache.ibatis.type.BaseTypeHandler;  
import org.apache.ibatis.type.JdbcType;  
  
import java.sql.CallableStatement;  
import java.sql.PreparedStatement;  
import java.sql.ResultSet;  
import java.sql.SQLException;  
  
public class IdCardDesensitizationTypeHandler extends BaseTypeHandler<String> {  
  
    /**  
     * 保存时设置数据库类型  
     */  
    @Override  
    public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {  
        // 验证身份证合法性  
        if (!verify(parameter)) {  
            // 如果不合法...  
        }  
        ps.setString(i, parameter); // 数据库类型  
    }  
  
    /**  
     * 从数据库获取时转换为 Java 对应的数据类型  
     */  
    @Override  
    public String getNullableResult(ResultSet rs, String columnName) throws SQLException {  
        return desensitization(rs.getString(columnName));  
    }  
  
    @Override  
    public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {  
        return desensitization(rs.getString(columnIndex));  
    }  
  
    @Override  
    public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {  
        return desensitization(cs.getString(columnIndex));  
    }  
  
    /**  
     * 身份证件号码脱敏  
     */  
    private String desensitization(String number) {  
        // 脱敏逻辑  
        return "";  
    }  
  
    /**  
     * 校验身份证合法性  
     * <p>  
     * <a href="https://zh.wikipedia.org/wiki/中华人民共和国公民身份号码">中华人民共和国公民身份号码</a></p>  
     * <p>     * <a href="http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm">http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm</a></p>     */    private static boolean verify(String number) {  
        if (number == null || number.length() != 18) {  
            return false;  
        }  
  
        int sum = 0;  
        for (int idx = 0; idx < 17; ++idx) {  
            sum += Math.pow(2, (17 - idx)) % 11 * (number.charAt(idx) - '0');  
        }  
  
        int checkCode = (12 - (sum % 11)) % 11;  
  
        if (checkCode < 10) {  
            return checkCode == number.charAt(17) - '0';  
        } else {  
            return number.charAt(17) == 'X' || number.charAt(17) == 'x';  
        }  
    }  
}
```

在 Mapper 中想使用自定义的 TypeHandler，可以通过 `@Result` 注解来指定某个字段使用该 TypeHandler，如：

```java
@Select("select * from user where id = #{id}")
@Results(id = "userMap", value = {
	// 其他字段的映射...
	@Result(
		property = "idCardNumber", 
		column = "id_card_number", 
		jdbcType = JdbcType.VARCHAR, 
		typeHandler = IdCardDesensitizationTypeHandler.class)
	})
UserPo getById(@Param("id") int id);
```

如果使用的是 XML，可以在 resultMap 中指定：

```xml
<resultMap id="ResultMap" type="com...User">
  <id column="id" property="id"/>
  <result column="id_card_number" property="idCardNumber" typeHandler="com...IdCardDesensitizationTypeHandler"/>
</resultMap>
```

虽然不推荐，但是还是可以在全局中使用。具体的方式实在 mybatis 的配置文件中，示例：

```xml
<configuration>

  ...

  <typeHandlers>
    <typeHandler handler="com...IdCardDesensitizationTypeHandler" />
  </typeHandlers>

  ...

</configuration>
```


不过有一点需要注意，MyBatis 配置文件中的 `<configuration>` 元素的子元素的顺序必须按照以下的顺序出现，否则会提示错误：

1. `<properties>`
2. `<settings>`
3. `<typeAliases>`
4. `<typeHandlers>`
5. `<objectFactory>`
6. `<objectWrapperFactory>`
7. `<reflectorFactory>`
8. `<plugins>`
9. `<environments>`
10. `<databaseIdProvider>`
11. `<mappers>`

每个元素都是可选的，但是如果存在，它们必须按照上述的顺序出现。

https://mybatis.org/mybatis-3/configuration.html#typeHandlers

