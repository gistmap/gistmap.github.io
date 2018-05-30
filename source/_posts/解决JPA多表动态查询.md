---
title: 解决JPA多表动态查询
date: 2018-05-30 22:03:52
tags: [java]
categories: java
---

### 前言

由于公司一直在使用 **JPA** 作为 ORM 框架，因此分配到需要多表复杂查询或动态查询都很头大。很多人对 JPA 抱有偏见，比如 JPA 只能处理简单的单表查询。下面总结下使用过的几种方法，前几种简单略过。

### EntityManager

主要代码：

```java
Query query = entityManager.createNativeQuery(sql);
List list=query.setFirstResult(start).setMaxResults(end).getResultList();
Iterator iterator = list.iterator();
List<Obj> objList;
while (iterator.hasNext()) {
            Object[] o = (Object[]) iterator.next();
            Obj obj = new Obj();
    		obj.setId(String)o[1];
    		....
            //各种类型转换塞值
            objList.add(obj);
}

```

总结：sql 拼接麻烦，手动取对象数组相当麻烦，当sql 字段顺序变了之后，每个取值都需要变化。



### 使用 Spring 提供的 JdbcTemplate

主要代码：

```java
List<Obj> list = jdbcTemplate.query(sql.toString(), new BeanPropertyRowMapper<>(Obj.class));
```

  总结：相比第一种方法简直舒服多了，BeanPropertyRowMapper 可以自动完成映射，也可以自定义实现特殊类型转换，不过自带的已基本够用。另外还支持下划线转驼峰。最后还要写sql，拼代码。



### Predicate+Join

单表动态分页查询用 Predicate 就能做到，Join 是多表查询的关键。下面用简单的一对一来做例子

```java
@Entity(name = "tb_apple")
@Data
public class Apple {
    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private Integer id;
    private String name;
    private BigDecimal weight;
    @Enumerated(EnumType.STRING)
    private ColorEnum color;
    private Date createTime;
}

@Entity(name = "tb_person")
@Data
public class Person {
    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private Integer id;
    private String name;
    @OneToOne
    @JoinColumn(name="apple_id")
    private Apple apple;
}

```

Repository接口 除了继承 JpaRepository 之外还需额外继承**JpaSpecificationExecutor**，有兴趣的可以点进去看看方法。

```java
public interface PersonRepository extends JpaRepository<Person,Integer>,JpaSpecificationExecutor<Person> {
}
```

这里动态查询已Person的name 和 所持有apple 的name 为例：

```java
@Component
public class PersonService {
    @Autowired
    private PersonRepository personRepository;

    public Page<Person> findAll(String name, String appleName, Integer page, Integer size){
        //这里可在 Pageable 里构建 Sort 用来排序
        Pageable pageable = PageRequest.of(page-1, size);
        Page<Person> housings = personRepository.findAll((root, criteriaQuery, criteriaBuilder)
                -> getPredicate(name, appleName, root, criteriaBuilder), pageable);
        return housings;
    }
    private Predicate getPredicate(String name, String appleName, Root<Person> root, CriteriaBuilder criteriaBuilder) {
        List<Predicate> list = Lists.newArrayList();
        if (StringUtils.isNotEmpty(name)) {
            list.add(criteriaBuilder.like(root.get("name").as(String.class), "%" + name + "%"));
        }
        if (StringUtils.isNotEmpty(appleName)) {
            Join<Apple, Person> join = root.join("apple", JoinType.LEFT);
            list.add(criteriaBuilder.like(join.get("name").as(String.class), "%" + appleName + "%"));
        }
        Predicate[] p = new Predicate[list.size()];
        return criteriaBuilder.and(list.toArray(p));
    }
}
```

这样完成了多表动态分页查询。

### 最后
为 JPA 正名！~
值得一说的我样例用的 Spring Boot 的版本为 **2.0.1.RELEASE**，因此以下的问题可能是由于新的Spring Boot 版本提高了 JPA 的版本所导致的。

- PageRequest 的构造函数被标记为 @deprecated ，因此我用了静态方法of，我也觉得这种方式更好
- 在写单元测试的时候发现repository.findById()返回的类型是Optional<Obj>，嚯嚯嚯
- 原本使用的repository.findOne()方法的参数变成了 **Example<S>** ,Example 的例子文档很多

