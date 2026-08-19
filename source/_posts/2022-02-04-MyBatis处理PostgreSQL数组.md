---
title: MyBatis 处理 PostgreSQL 数组
date: 2022-02-04 00:53:35
categories:
- Data Engineering
- PostgreSQL
tags:
- Java
- MyBatis
- PostgreSQL
- TypeHandler
- SQL Arrays
---

[上回书](/2022/02/01/PostgreSQL将字符串类型的列转为数组)说到，我们把数据库存储数组的字符串转为了原生数组 `bigint[]` 类型，SQL
写起来是方便了，在 Java 代码里查询插入都会报错，要怎么使用呢？这里使用的是 [MyBatis](https://mybatis.org/)，所以就要扩展 MyBatis 的
[typeHandler](https://mybatis.org/mybatis-3/configuration.html#typeHandlers) 了，具体内容如下，原理我也不清楚，因为都是
[Copilot](https://copilot.github.com/) 帮我写的😁

<!--more-->

## 代码
```Java ExampleTypeHandler.java
// 这个 BaseTypeHandler 里的泛型就是你真实要使用的数组类型，这里写成 List<Object> 能少写几个 typeHandler
public class ExampleTypeHandler extends BaseTypeHandler<List<Object>> {

    /**
     * 这个是 Java 类型往数据库存储时的处理逻辑
     */
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, List<Object> parameter, JdbcType jdbcType) throws SQLException {
        // 如果是空数组的话也给它塞个空数组进去，在 PostgreSQL 中是 {} 这种，应该可以根据业务上的需求调整
        if (parameter.size() == 0) {
            Connection conn = ps.getConnection();
            Array array = conn.createArrayOf(JDBCType.BIGINT.getName(), new Long[0]);
            ps.setArray(i, array);
            array.free();
            return;
        }

        String typeName = null;
        if (parameter.get(0) instanceof Integer) {
            typeName = JDBCType.INTEGER.getName();
        } else if (parameter.get(0) instanceof String) {
            typeName = JDBCType.VARCHAR.getName();
        } else if (parameter.get(0) instanceof Boolean) {
            typeName = JDBCType.BOOLEAN.getName();
        } else if (parameter.get(0) instanceof Double) {
            typeName = JDBCType.DOUBLE.getName();
        } else if (parameter.get(0) instanceof Float) {
            typeName = JDBCType.FLOAT.getName();
        } else if (parameter.get(0) instanceof Long) {
            typeName = JDBCType.BIGINT.getName();
        } else if (parameter.get(0) instanceof Short) {
            typeName = JDBCType.SMALLINT.getName();
        } else if (parameter.get(0) instanceof Byte) {
            typeName = JDBCType.TINYINT.getName();
        }
        if (typeName == null) {
            throw new TypeException("ArrayTypeHandler parameter typeName error, your type is " + parameter.getClass().getName());
        }

        Connection conn = ps.getConnection();
        Array array = conn.createArrayOf(typeName, parameter.toArray());
        ps.setArray(i, array);
        array.free();
    }

    @Override
    public List<Object> getNullableResult(ResultSet resultSet, String s) throws SQLException {
        return getList(resultSet.getArray(s));
    }

    @Override
    public List<Object> getNullableResult(ResultSet resultSet, int i) throws SQLException {
        return getList(resultSet.getArray(i));
    }

    @Override
    public List<Object> getNullableResult(CallableStatement callableStatement, int i) throws SQLException {
        return getList(callableStatement.getArray(i));
    }

    /**
     * 上面都是从数据库读取后存入 Java PO 对象时的处理逻辑
     * 这里写了一个统一的方法，因为逻辑都一样
     */
    private List<Object> getList(Array array) {
        // 如果数据库字段为 null，这里会转成一个空的 List，根据业务需求更改，赋值为 null 的话，后面对 PO 处理要注意 NPE
        if (array == null) {
            return Collections.emptyList();
        }
        try {
            // 这里要把 java.sql.Array 转成 java.util.List，尝试了多种写法，只有 Copilot 写的不会出警告
            Object[] objects = (Object[]) array.getArray();
            array.free();
            return Arrays.stream(objects).collect(Collectors.toList());
        } catch (Exception ignored) {
        }
        return null;
    }
}
```

## 配置
完成了自定义的 typeHandler 后，只需要在 MyBatis 中配置好 typeHandler 就可以正确处理数组类型的数据了。

文档中是使用的 xml 配置文件来配置的。
```XML mybatis-config.xml
<!-- mybatis-config.xml -->
<typeHandlers>
  <typeHandler handler="org.mybatis.example.ExampleTypeHandler"/>
</typeHandlers>
```

这里使用的是注解的方式来配置，首先是 POJO 属性上指定 `@ColumnType(typeHandler = ExampleTypeHandler.class)`
```Java ExamplePO.java
@ColumnType(typeHandler = ExampleTypeHandler.class)
private List<Long> longList;
```

然后在 mapper.xml 文件中设置 typeHandler
```XML ExampleMapper.xml
<resultMap id="BaseResultMap" type="org.mybatis.example.ExamplePO">
    ...
    <result column="bigint_array" jdbcType="ARRAY" property="longList" typeHandler="org.mybatis.example.ExampleTypeHandler"/>
    ...
</resultMap>

<select id="list" resultMap="BaseResultMap">
    ...
    bigint_array <![CDATA[&&]]> #{queryList, jdbcType=ARRAY, typeHandler=org.mybatis.example.ExampleTypeHandler}
    ...
</select>
```
