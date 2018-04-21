---
title: MyBatis之TypeHandler
date: 2018-4-21 22:03:55
tags: 
    - framework
categories: framework
---

### 前言

TypeHandler是Mybatis在存储数据前及从数据库拿到数据组装前的一个类型映射处理器，简单的说：在JavaType<-->JdbcType的转化中做点什么。其实Mybatis已经内置了30多种通用的TypeHandler，如：StringHandler，IntegerTypeHandler。在官网Configuration XML下的typeHandlers可以看到全部的内置typeHandler与jdbcType的对应关系。在**TypeHandlerRegistry**类中的TypeHandlerRegistry方法也可看到基本类型和对象类型注册了相同的内置TypeHandler。通常来说，自带的通用类型处理已经足够，但有些特殊情况无法满足，还好框架给了我们自定义TypeHandler来解决这一问题。

### TypeHandler<T>接口

```java
public interface TypeHandler<T> {

  void setParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException;

  T getResult(ResultSet rs, String columnName) throws SQLException;

  T getResult(ResultSet rs, int columnIndex) throws SQLException;

  T getResult(CallableStatement cs, int columnIndex) throws SQLException;

}
```

通过看方法名和参数类型能大致的了解到这些方法是做什么的了。PreparedStatement-插入前设置参数，ResultSet-从结果集里拿数据。

### 准备实体类

```java
public enum Sex {
    MALE(0 , "男"),
    FEMALE(1 , "女");

    private int SexCode;
    private String SexName;

    Sex(int sexCode, String sexName) {
        SexCode = sexCode;
        SexName = sexName;
    }

    public int getSexCode() {
        return SexCode;
    }

    public static Sex getSexFromCode(int code){
        for (Sex sex : Sex.values()) {
            if (sex.getSexCode() == code)
                return sex;
        }
        return null;
    }
}
```

```java
@Data
public class User {
    private Integer id;
    private String name;
    private Sex sex;
}
```



### 自定义TypeHandler

```java
public class SexTypeHandler implements TypeHandler<Sex> {
    public void setParameter(PreparedStatement preparedStatement, int i, Sex sex, JdbcType jdbcType) throws SQLException {
        preparedStatement.setInt(i, sex.getSexCode());
    }

    public Sex getResult(ResultSet resultSet, String s) throws SQLException {
        int result = resultSet.getInt(s);
        return Sex.getSexFromCode(result);
    }

    public Sex getResult(ResultSet resultSet, int i) throws SQLException {
        int result = resultSet.getInt(i);
        return Sex.getSexFromCode(result);
    }

    public Sex getResult(CallableStatement callableStatement, int i) throws SQLException {
        int result = callableStatement.getInt(i);
        return Sex.getSexFromCode(result);
    }
}
```



### 使用

```xml
<resultMap id="userMap" type="com.gistmap.pojo.User">
        <result property="id" column="id"/>
        <result property="name" column="name"/>
        <result property="sex" column="sex" javaType="com.gistmap.pojo.Sex"
                jdbcType="INTEGER" typeHandler="com.gistmap.handler.Sex1TypeHandler"/>
</resultMap>
```



调用接口得到`User(id=1, name=zhangran, sex=MALE)`

但在存入数据库中报错如下：

`Caused by: java.sql.SQLException: Incorrect integer value: 'FEMALE' for column 'sex' at row 1`

Emmm 加上这个

```xml
<insert id="save"  useGeneratedKeys="true" keyProperty="id">
        insert into user (name,sex) values (#{name} , #{sex, typeHandler=com.gistmap.handler.Sex1TypeHandler})
</insert>
```

这样存取都能按照我们如愿的进行。



### 后记

其实自定义TypeHandler还可以通过继承`org.apache.ibatis.type.BaseTypeHandler`类，这样会更简单，两种方式我都测试过了，均可以达到效果。


