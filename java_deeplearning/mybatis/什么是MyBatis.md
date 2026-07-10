# MyBatis 精简笔记

## 1. MyBatis 是什么？

MyBatis 是一个 **持久层框架**，主要用来帮助 Java 程序操作数据库。

简单理解：

> MyBatis 可以把 Java 方法和 SQL 语句对应起来，让我们通过调用 Java 方法来执行 SQL。

例如：

```java
User getById(Long id);
```

对应 SQL：

```sql
select * from user where id = #{id}
```

调用 `getById(1L)` 时，MyBatis 会自动执行对应 SQL，并把结果封装成 `User` 对象。

------

## 2. MyBatis 在项目中的位置

常见后端三层结构：

```text
Controller   接收前端请求
Service      处理业务逻辑
Mapper       操作数据库
```

MyBatis 主要负责 **Mapper 层**。

执行流程：

```text
浏览器请求
   ↓
Controller
   ↓
Service
   ↓
Mapper
   ↓
数据库
```

------

## 3. 为什么需要 MyBatis？

如果不用 MyBatis，原生 JDBC 操作数据库比较麻烦，需要自己写：

```text
获取数据库连接
创建 PreparedStatement
执行 SQL
遍历 ResultSet
手动封装对象
关闭资源
```

MyBatis 可以帮我们简化这些重复代码。

------

## 4. `application.yml` 配置什么？

`application.yml` 是项目的总配置文件。

它一般配置：

```text
数据库连接信息
MyBatis 的 XML 文件位置
实体类别名
驼峰映射
项目端口号等
```

常见配置：

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/test?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8
    username: root
    password: 123456

mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.example.entity
  configuration:
    map-underscore-to-camel-case: true
```

重点解释：

```yaml
mapper-locations: classpath:mapper/*.xml
```

表示：

> 告诉 MyBatis 去 `resources/mapper/` 目录下找 XML 文件。

```yaml
type-aliases-package: com.example.entity
```

表示：

> 给实体类起别名，XML 里可以少写全类名。

例如原来要写：

```xml
resultType="com.example.entity.User"
```

配置后可以写：

```xml
resultType="User"
map-underscore-to-camel-case: true
```

表示开启下划线转驼峰：

```text
user_name   -> userName
create_time -> createTime
```

------

## 5. MyBatis 的 XML 配置什么？

`application.yml` 是项目总配置。

而 MyBatis 的 `Mapper.xml` 是 **SQL 映射配置**。

简单说：

```text
yml：告诉项目数据库在哪、XML 文件在哪
xml：告诉 MyBatis 某个 Java 方法对应哪条 SQL
```

例如：

```java
User getById(Long id);
```

对应 XML：

```xml
<select id="getById" resultType="User">
    select * from user where id = #{id}
</select>
```

意思是：

> 调用 `getById` 方法时，执行这条 SQL。

------

## 6. Mapper 接口和 XML 的对应关系

假设 Mapper 接口是：

```java
package com.example.mapper;

import com.example.entity.User;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface UserMapper {

    User getById(Long id);

}
```

对应 XML 文件：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.example.mapper.UserMapper">

    <select id="getById" resultType="User">
        select *
        from user
        where id = #{id}
    </select>

</mapper>
```

核心对应关系：

```text
namespace  对应 Mapper 接口的全类名
id         对应 Mapper 接口的方法名
resultType 对应方法返回值类型
#{参数名}  对应方法参数或对象属性
```

也就是：

```xml
<mapper namespace="com.example.mapper.UserMapper">
```

对应：

```java
public interface UserMapper
<select id="getById">
```

对应：

```java
User getById(Long id);
#{id}
```

对应：

```java
Long id
```

------

## 7. XML 常见标签

MyBatis XML 中常见的四类标签：

```xml
<select>
<insert>
<update>
<delete>
```

对应关系：

| 标签       | 作用     | 常见返回值      |
| ---------- | -------- | --------------- |
| `<select>` | 查询数据 | 对象、集合      |
| `<insert>` | 插入数据 | `int` 或 `void` |
| `<update>` | 修改数据 | `int` 或 `void` |
| `<delete>` | 删除数据 | `int` 或 `void` |

示例：

```java
User getById(Long id);

int insert(User user);

int update(User user);

int deleteById(Long id);
```

对应 XML：

```xml
<select id="getById" resultType="User">
    select * from user where id = #{id}
</select>

<insert id="insert">
    insert into user(username, password)
    values (#{username}, #{password})
</insert>

<update id="update">
    update user
    set username = #{username}
    where id = #{id}
</update>

<delete id="deleteById">
    delete from user where id = #{id}
</delete>
```

------

## 8. `<select>` 里面必须是 select 吗？

严格来说，`<select>` 标签表示这是一个 **查询操作**。

所以里面最终应该是查询 SQL。

正确写法：

```xml
<select id="getById" resultType="User">
    select * from user where id = #{id}
</select>
```

如果是删除操作，不应该写成：

```xml
<select id="deleteById">
    delete from user where id = #{id}
</select>
```

虽然数据库可能能执行，但 MyBatis 会把它当成查询操作处理，容易出问题。

正确写法应该是：

```xml
<delete id="deleteById">
    delete from user where id = #{id}
</delete>
```

记法：

> 外层标签决定 MyBatis 怎么处理 SQL，里面 SQL 决定数据库执行什么。二者最好保持一致。

------

## 9. 动态 SQL

`<select>` 里面不一定全是纯 SQL，也可以写 MyBatis 的动态 SQL 标签。

例如：

```xml
<select id="list" resultType="User">
    select *
    from user
    <where>
        <if test="username != null and username != ''">
            username = #{username}
        </if>
        <if test="status != null">
            and status = #{status}
        </if>
    </where>
</select>
```

这里的：

```xml
<where>
<if>
```

不是普通 SQL，而是 MyBatis 的动态 SQL 标签。

它们最终会被 MyBatis 拼成真正的 SQL。

------

## 10. `#{}` 是什么？

`#{}` 是 MyBatis 的参数占位符。

例如：

```xml
where id = #{id}
```

对应方法参数：

```java
User getById(Long id);
```

`#{id}` 会把方法参数 `id` 安全地传入 SQL。

它类似 JDBC 中的 `?`，可以防止 SQL 注入。

推荐使用：

```xml
#{}
```

不要随便使用：

```xml
${}
```

因为 `${}` 是字符串拼接，容易有 SQL 注入风险。

------

## 11. 返回值对象需要什么？

MyBatis 查询结果要封装成 Java 对象。

例如数据库查出来：

```text
id = 1
username = zhangsan
password = 123456
```

MyBatis 大概会做类似的事：

```java
User user = new User();
user.setId(1L);
user.setUsername("zhangsan");
user.setPassword("123456");
```

所以实体类最好有：

```text
无参构造方法
getter / setter
属性名能和数据库字段对应
属性类型和数据库字段类型兼容
```

推荐写法：

```java
public class User {
    private Long id;
    private String username;
    private String password;

    public User() {
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    // 其他 getter / setter
}
```

也可以使用 Lombok 简化：

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private Long id;
    private String username;
    private String password;
}
```

------

## 12. 没有 getter / setter 会怎样？

如果实体类没有 setter，MyBatis 可能无法把查询结果封装进去。

可能出现：

```text
对象创建了，但是属性是 null
```

或者报错：

```text
Could not set property
```

所以一般实体类都要写 getter 和 setter，或者使用 Lombok 的 `@Data`。

------

## 13. 返回值对象不存在会怎样？

比如 XML 中写：

```xml
<select id="getById" resultType="User">
    select * from user where id = #{id}
</select>
```

但是项目里没有 `User` 类，或者没有配置别名，可能会报错：

```text
Could not resolve type alias 'User'
```

解决方法有两个。

方法一：在 yml 中配置别名包：

```yaml
mybatis:
  type-aliases-package: com.example.entity
```

方法二：XML 中写完整类名：

```xml
<select id="getById" resultType="com.example.entity.User">
    select * from user where id = #{id}
</select>
```

------

## 14. 返回值对象不兼容会怎样？

如果数据库字段类型和 Java 属性类型不匹配，可能会出现类型转换错误。

例如数据库字段：

```text
id bigint
age int
username varchar
create_time datetime
```

推荐 Java 类型：

```java
private Long id;
private Integer age;
private String username;
private LocalDateTime createTime;
```

常见对应关系：

| MySQL 类型 | Java 类型       |
| ---------- | --------------- |
| `bigint`   | `Long`          |
| `int`      | `Integer`       |
| `varchar`  | `String`        |
| `datetime` | `LocalDateTime` |
| `date`     | `LocalDate`     |
| `decimal`  | `BigDecimal`    |

------

## 15. 查询字段多了或少了怎么办？

### 查询字段多了

SQL：

```sql
select id, username, password from user
```

Java 类：

```java
public class User {
    private Long id;
    private String username;
}
```

多出来的 `password` 一般会被忽略，不会出大问题。

### 查询字段少了

SQL：

```sql
select id, username from user
```

Java 类：

```java
public class User {
    private Long id;
    private String username;
    private String password;
}
```

少查的 `password` 会是：

```java
null
```

------

## 16. 字段名对不上怎么办？

数据库字段：

```text
user_name
create_time
```

Java 属性：

```java
private String userName;
private LocalDateTime createTime;
```

如果不开启驼峰映射，可能封装不上。

推荐在 yml 中开启：

```yaml
mybatis:
  configuration:
    map-underscore-to-camel-case: true
```

这样 MyBatis 会自动映射：

```text
user_name   -> userName
create_time -> createTime
```

------

## 17. Service 如何调用 Mapper？

Mapper：

```java
@Mapper
public interface UserMapper {
    User getById(Long id);
}
```

Service：

```java
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;

    public User getById(Long id) {
        return userMapper.getById(id);
    }
}
```

Controller：

```java
@RestController
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/{id}")
    public User getById(@PathVariable Long id) {
        return userService.getById(id);
    }
}
```

请求流程：

```text
访问 /user/1
   ↓
UserController.getById(1)
   ↓
UserService.getById(1)
   ↓
UserMapper.getById(1)
   ↓
执行 UserMapper.xml 中的 SQL
   ↓
返回 User 对象
```

------

## 18. 总结

MyBatis 的关键就是三件事：

```text
Mapper 接口：定义 Java 方法
Mapper XML：编写 SQL
application.yml：配置数据库和 XML 位置
```

最重要的对应关系：

```text
namespace  = Mapper 接口全类名
id         = Mapper 接口方法名
resultType = 查询返回对象类型
#{参数名}  = 方法参数或对象属性
```

> 