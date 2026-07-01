# 苍穹外卖 Day1 学习笔记

> Day1 的重点不是复杂业务，而是先理解一个 Spring Boot 后端项目的基本骨架：  
> **Controller 接请求，Service 处理业务，Mapper 操作数据库，MySQL 存数据。**

---

## 1. 项目整体结构

苍穹外卖后端大致采用经典三层结构：

```text
前端请求
   ↓
Controller 层：接收请求、返回结果
   ↓
Service 层：处理业务逻辑
   ↓
Mapper 层：操作数据库
   ↓
MySQL 数据库
```

配套技术：

```text
Spring Boot：搭建后端项目
Spring MVC：处理 Web 请求
MyBatis：操作数据库
Swagger / Knife4j：生成接口文档
JWT：登录身份令牌
application.yml：项目配置文件
```

---

## 2. `.sql` 文件

`.sql` 文件本质上是保存 SQL 语句的文本文件。

常见作用：

```text
创建数据库
创建表
插入初始化数据
备份数据库
恢复数据库
```

示例：

```sql
CREATE DATABASE sky_take_out;

USE sky_take_out;

CREATE TABLE employee (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(32),
    password VARCHAR(64)
);
```

可以理解为：

```text
.sql 文件 = 数据库脚本
执行 .sql 文件 = 按脚本初始化数据库
```

---

## 3. 命令行 MySQL

进入 MySQL：

```bash
mysql -u root -p
```

常用命令：

```sql
SHOW DATABASES;
USE sky_take_out;
SHOW TABLES;
DESC employee;
```

执行 `.sql` 文件：

```sql
SOURCE D:/sky.sql;
```

Windows 路径建议使用 `/`，不要写成 `\`，否则容易出错。

---

## 4. Controller、Service、Mapper

### 4.1 Controller：接收请求

Controller 负责接收前端请求，并返回结果。

```java
@RestController
@RequestMapping("/admin/employee")
public class EmployeeController {

    @Autowired
    private EmployeeService employeeService;

    @PostMapping("/login")
    public Result<EmployeeLoginVO> login(@RequestBody EmployeeLoginDTO employeeLoginDTO) {
        return employeeService.login(employeeLoginDTO);
    }
}
```

Controller 主要负责：

```text
接收参数
调用 Service
返回结果
```

不要在 Controller 中写大量业务逻辑。

---

### 4.2 Service：处理业务逻辑

Service 是业务层，负责真正的业务判断。

例如：

```text
判断账号是否存在
判断密码是否正确
判断账号是否被禁用
生成 JWT 令牌
```

示例：

```java
@Service
public class EmployeeServiceImpl implements EmployeeService {

    @Autowired
    private EmployeeMapper employeeMapper;

    public Employee login(EmployeeLoginDTO employeeLoginDTO) {
        Employee employee = employeeMapper.getByUsername(employeeLoginDTO.getUsername());

        if (employee == null) {
            throw new AccountNotFoundException("账号不存在");
        }

        return employee;
    }
}
```

Service 可以调用一个或多个 Mapper。

---

### 4.3 Mapper：操作数据库

Mapper 负责执行 SQL。

```java
@Mapper
public interface EmployeeMapper {

    @Select("select * from employee where username = #{username}")
    Employee getByUsername(String username);
}
```

Mapper 只负责数据库操作，不负责复杂业务判断。

---

## 5. MyBatis

MyBatis 是 Java 后端常用的数据库操作框架。

它解决的问题是：

> Java 方法如何对应 SQL 语句，以及 SQL 查询结果如何变成 Java 对象。

例如：

```java
employeeMapper.getByUsername("admin");
```

MyBatis 背后执行：

```sql
select * from employee where username = 'admin';
```

然后把查询结果封装为：

```java
Employee employee;
```

一句话总结：

```text
MyBatis = 让 Mapper 接口能够执行 SQL 的框架
```

---

## 6. 注解编程

Java 后端中常见很多 `@xxx`，这些叫注解。

注解本身不真正干活，它只是给框架看的标记。真正干活的是 Spring、MyBatis、Swagger 等框架。

常见注解：

```java
@RestController
@Service
@Mapper
@Autowired
@RequestMapping
@GetMapping
@PostMapping
```

示例：

```java
@RestController
```

表示：

```text
告诉 Spring：这个类是 Controller，需要处理前端请求。
```

```java
@Service
```

表示：

```text
告诉 Spring：这个类是 Service，需要交给 Spring 管理。
```

```java
@Mapper
```

表示：

```text
告诉 MyBatis：这个接口是数据库操作接口，需要生成代理对象。
```

注解编程的本质：

```text
程序员用注解声明意图
框架扫描注解
框架自动完成对象创建、接口绑定、SQL 代理等工作
```

---

## 7. Spring 容器与 Bean

Spring 容器可以理解为一个“大对象仓库”。

项目启动时，Spring 会扫描这些注解：

```text
@Controller
@RestController
@Service
@Component
@Configuration
@Mapper
```

然后创建对应对象，并放进 Spring 容器中。

被 Spring 管理的对象叫：

```text
Bean
```

例如：

```java
@Service
public class EmployeeServiceImpl {
}
```

Spring 启动后会创建一个 `EmployeeServiceImpl` 对象，这个对象就是一个 Bean。

---

## 8. 依赖注入与控制反转

以前写法：

```java
EmployeeService employeeService = new EmployeeServiceImpl();
```

这是程序员自己创建对象。

Spring 推荐写法：

```java
@Autowired
private EmployeeService employeeService;
```

意思是：

```text
我需要 EmployeeService，你 Spring 帮我从容器里找一个塞进来。
```

这叫 **依赖注入 DI**。

背后的思想叫 **控制反转 IoC**。

简单理解：

```text
以前：程序员自己 new 对象
现在：Spring 创建对象、管理对象、注入对象
```

好处：

```text
降低耦合
统一管理对象
方便测试
方便加事务、日志、权限等增强功能
```

---

## 9. Spring 默认创建几个对象？

Spring 不是靠猜，而是按规则来。

默认情况下：

```text
一个 Bean 创建一个对象
```

这叫单例：

```text
singleton
```

例如：

```java
@Service
public class EmployeeServiceImpl {
}
```

默认整个 Spring 容器中只有一个 `EmployeeServiceImpl` 对象。

即使有很多 Controller 使用它，也都是共用同一个对象。

因此，Controller、Service 中不要随便放某个用户独有的数据作为成员变量。

不推荐：

```java
private Long currentUserId;
```

因为多个请求可能共用同一个对象，容易造成数据混乱。

---

## 10. 代理对象

代理对象可以理解为：

```text
你表面上调用的是一个对象
实际上中间有个“代理”拦截调用
然后执行额外逻辑
```

### 10.1 MyBatis 的 Mapper 代理

Mapper 是接口：

```java
@Mapper
public interface EmployeeMapper {
    Employee getByUsername(String username);
}
```

接口本身没有实现类，不能直接创建对象。

MyBatis 会帮它生成代理对象。

当你调用：

```java
employeeMapper.getByUsername("admin");
```

代理对象背后做：

```text
找到对应 SQL
执行 SQL
封装结果
返回 Employee 对象
```

所以 Mapper 不需要自己写实现类。

---

### 10.2 Spring 的事务代理

例如：

```java
@Transactional
public void createOrder() {
    ...
}
```

Spring 可以给这个 Service 套一层代理对象。

调用时流程类似：

```text
开启事务
↓
执行 createOrder()
↓
成功：提交事务
失败：回滚事务
```

代理对象的核心作用：

```text
帮接口生成实现逻辑
在方法前后添加额外功能
```

---

## 11. Spring MVC

Spring MVC 是 Spring 中负责处理 Web 请求的模块。

它主要做：

```text
接收浏览器/前端请求
根据 URL 找到 Controller 方法
调用 Controller
把返回值转成 JSON
返回给前端
```

核心调度者叫：

```text
DispatcherServlet
```

请求流程：

```text
前端请求 /admin/employee/login
        ↓
DispatcherServlet
        ↓
找到 EmployeeController.login()
        ↓
调用 Service
        ↓
调用 Mapper
        ↓
返回 JSON
```

一句话：

```text
Spring MVC = Spring 中负责接请求、分发请求、返回响应的 Web 框架
```

---

## 12. JWT 令牌

JWT 可以理解为：

> 用户登录成功后，后端发给前端的一张“通行证”。

登录流程：

```text
1. 前端提交用户名和密码
2. 后端验证账号密码
3. 验证成功，生成 JWT
4. 后端把 JWT 返回给前端
5. 前端保存 JWT
6. 后续请求都带上 JWT
7. 后端验证 JWT，判断用户是否登录
```

JWT 一般长这样：

```text
eyJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOjEsInVzZXJuYW1lIjoiadminIn0.xxxxx
```

它由三部分组成：

```text
Header.Payload.Signature
```

常见保存内容：

```text
用户 ID
用户名
角色
过期时间
```

注意：

```text
JWT 中不要放密码、身份证号等敏感信息。
```

因为 JWT 前两段只是编码，不是加密，别人拿到后可以解码看到。

---

## 13. 配置与 `@ConfigurationProperties`

配置就是程序运行时需要的参数。

例如：

```text
服务器端口
数据库地址
数据库账号密码
JWT 密钥
文件上传路径
OSS 配置
```

这些通常写在：

```text
application.yml
```

示例：

```yaml
sky:
  jwt:
    admin-secret-key: itcast
    admin-ttl: 7200000
    admin-token-name: token
```

然后用：

```java
@ConfigurationProperties(prefix = "sky.jwt")
```

把配置绑定到 Java 对象。

示例：

```java
@Component
@ConfigurationProperties(prefix = "sky.jwt")
public class JwtProperties {

    private String adminSecretKey;
    private long adminTtl;
    private String adminTokenName;

    // getter setter
}
```

这样 Spring Boot 启动后会自动把：

```yaml
sky:
  jwt:
    admin-secret-key: itcast
```

绑定到：

```java
adminSecretKey
```

一句话：

```text
@ConfigurationProperties = 把配置文件中的一组配置，批量装进一个 Java 对象
```

---

## 14. Swagger / Knife4j

Swagger / Knife4j 是接口文档工具。

作用：

```text
自动扫描 Controller
生成接口文档
可以在线测试接口
方便前后端联调
```

苍穹外卖中常用访问地址：

```text
http://localhost:8080/doc.html
```

注意不要写成：

```text
doc.hrml
```

---

## 15. Swagger 配置类

常见配置：

```java
@Bean
public Docket docket() {
    log.info("准备生成接口文档...");

    ApiInfo apiInfo = new ApiInfoBuilder()
            .title("苍穹外卖项目接口文档")
            .version("2.0")
            .description("苍穹外卖项目接口文档")
            .build();

    Docket docket = new Docket(DocumentationType.SWAGGER_2)
            .apiInfo(apiInfo)
            .select()
            .apis(RequestHandlerSelectors.basePackage("com.sky.controller"))
            .paths(PathSelectors.any())
            .build();

    return docket;
}
```

这段代码的作用：

```text
创建 Swagger 的核心配置对象 Docket
设置接口文档标题、版本、描述
指定扫描 com.sky.controller 包下的 Controller
```

其中：

```java
@Bean
```

表示：

```text
Spring 启动时调用这个方法，把返回的 Docket 对象放进 Spring 容器。
```

---

## 16. 静态资源映射

Swagger / Knife4j 页面本质上也是静态资源，例如：

```text
doc.html
CSS
JavaScript
图片
```

因此需要配置静态资源映射：

```java
@Override
protected void addResourceHandlers(ResourceHandlerRegistry registry) {
    log.info("开始设置静态资源映射...");

    registry.addResourceHandler("/doc.html")
            .addResourceLocations("classpath:/META-INF/resources/");

    registry.addResourceHandler("/webjars/**")
            .addResourceLocations("classpath:/META-INF/resources/webjars/");
}
```

### 16.1 `/doc.html`

```java
registry.addResourceHandler("/doc.html")
        .addResourceLocations("classpath:/META-INF/resources/");
```

含义：

```text
浏览器访问 /doc.html
Spring 去 classpath:/META-INF/resources/ 下面找 doc.html
```

也就是：

```text
http://localhost:8080/doc.html
        ↓
classpath:/META-INF/resources/doc.html
```

### 16.2 `/webjars/**`

```java
registry.addResourceHandler("/webjars/**")
        .addResourceLocations("classpath:/META-INF/resources/webjars/");
```

含义：

```text
浏览器访问 /webjars/ 下的 JS、CSS、图片等资源
Spring 去 classpath:/META-INF/resources/webjars/ 下找
```

例如：

```text
/webjars/swagger-ui/swagger-ui.css
/webjars/swagger-ui/swagger-ui-bundle.js
```

这些是 Swagger 页面需要的前端资源。

---

## 17. `classpath`、`META-INF`、jar 包

### 17.1 jar 包

jar 包是 Java 项目的打包文件。

执行：

```bash
mvn package
```

会生成：

```text
target/xxx.jar
```

这个 jar 中通常包含：

```text
你的 .class 代码
application.yml 配置
依赖的 jar 包
META-INF 信息
```

你的 `.java` 源代码会先编译成 `.class` 文件，再放进 jar 包中。

---

### 17.2 `META-INF`

`META-INF` 是 jar 包中的元信息目录。

常见结构：

```text
knife4j.jar
├── META-INF/
│   ├── MANIFEST.MF
│   └── resources/
│       ├── doc.html
│       └── webjars/
```

`META-INF/resources/` 常用来放 jar 包自带的 Web 静态资源。

Knife4j 的 `doc.html` 就可能位于依赖 jar 包的：

```text
META-INF/resources/doc.html
```

所以你项目里看不到 `doc.html`，但它仍然能被访问，因为它在依赖 jar 包中。

---

## 18. 静态资源和动态资源

### 18.1 静态资源

静态资源是现成文件，服务器直接返回。

例如：

```text
html
css
js
png
jpg
doc.html
```

访问：

```text
/doc.html
```

服务器直接找文件并返回。

---

### 18.2 动态资源

动态资源需要后端程序处理后生成结果。

例如：

```text
/admin/employee/login
/admin/dish/page
/user/shop/status
```

请求流程：

```text
Controller
↓
Service
↓
Mapper
↓
Database
↓
返回 JSON
```

一句话：

```text
静态资源 = 现成文件
动态资源 = 后端程序处理后的结果
```

---

## 19. Swagger 常见注解

这些注解只影响接口文档，不影响业务逻辑。

### 19.1 `@Api`

用在 Controller 类上，表示一组接口。

```java
@Api(tags = "员工相关接口")
@RestController
@RequestMapping("/admin/employee")
public class EmployeeController {
}
```

作用：

```text
Swagger 页面显示：员工相关接口
```

---

### 19.2 `@ApiOperation`

用在 Controller 方法上，说明某个接口。

```java
@ApiOperation("员工登录")
@PostMapping("/login")
public Result<EmployeeLoginVO> login(@RequestBody EmployeeLoginDTO employeeLoginDTO) {
    ...
}
```

作用：

```text
说明这个接口是“员工登录”
```

---

### 19.3 `@ApiModel`

用在 DTO、VO、实体类上，说明这个数据模型。

```java
@ApiModel(description = "员工登录请求参数")
public class EmployeeLoginDTO {
}
```

---

### 19.4 `@ApiModelProperty`

用在字段上，说明字段含义。

```java
@ApiModelProperty("用户名")
private String username;

@ApiModelProperty("密码")
private String password;
```

总结：

```text
@Api：说明一组接口
@ApiOperation：说明一个接口
@ApiModel：说明一个数据类
@ApiModelProperty：说明类里的字段
```

---

## 20. Day1 整体流程

可以把 Day1 串成这条线：

```text
1. 导入项目
2. 配置数据库
3. 执行 .sql 文件初始化表
4. 启动 Spring Boot
5. Spring 扫描 Controller、Service、Mapper
6. 创建 Bean，放入 Spring 容器
7. MyBatis 为 Mapper 生成代理对象
8. Swagger 扫描 Controller，生成接口文档
9. 访问 /doc.html 查看接口
10. 调用登录接口，后端生成 JWT
11. 前端后续请求携带 JWT
```

---

## 21. 总体关系图

```text
浏览器 / 前端
    |
    | 访问 /doc.html
    v
Knife4j / Swagger 静态页面
    |
    | 在线测试接口
    v
Controller
    |
    | 调用
    v
Service
    |
    | 调用
    v
Mapper 代理对象
    |
    | 执行 SQL
    v
MySQL 数据库
```

同时：

```text
Spring 容器负责管理 Controller、Service 等 Bean
MyBatis 负责生成 Mapper 代理对象
Swagger 负责生成接口文档
JWT 负责登录身份认证
application.yml 负责保存配置参数
```

---

## 22. Day1 必须记住的几句话

```text
Controller 管请求，Service 管业务，Mapper 管数据库。
```

```text
Spring 容器负责创建对象、管理对象、注入对象。
```

```text
Bean 就是被 Spring 管理的对象。
```

```text
MyBatis 的 Mapper 是代理对象，不需要自己写实现类。
```

```text
Swagger / Knife4j 是接口文档和接口测试工具。
```

```text
doc.html 是静态资源，不是 Controller 接口。
```

```text
@ConfigurationProperties 用来把 application.yml 配置绑定到 Java 对象。
```

```text
JWT 是登录成功后后端发给前端的身份令牌。
```

```text
jar 包里不仅可以有代码，也可以有静态资源，比如 META-INF/resources/doc.html。
```



## 24. Java 反射机制

### 24.1 什么是反射？

反射可以理解为：

> **程序在运行时，读取和操作类的信息。**

普通写法：

```
User user = new User();
user.sayHello();
```

这种写法是：写代码时就知道要创建 `User`，调用 `sayHello()`。

反射写法：

```
Class<?> clazz = User.class;
Object obj = clazz.getConstructor().newInstance();
Method method = clazz.getDeclaredMethod("sayHello");
method.invoke(obj);
```

这种写法是：运行时再去读取 `User` 类的信息，然后创建对象、调用方法。

一句话：

```
普通调用：提前知道操作谁
反射调用：运行时再去找操作谁
```

------

### 24.2 反射读的不是 `.java` 源码

Java 程序运行时，真正执行的是编译后的 `.class` 文件。

```
User.java
   ↓ 编译
User.class
   ↓ JVM 运行
```

反射读取的是 `.class` 里的类信息，比如：

```
类名
属性
方法
构造方法
注解
参数类型
返回值类型
```

所以反射不是“看源码”，而是“看类的结构信息”。

------

### 24.3 反射中几个重要对象

假设有一个类：

```
public class User {
    private String name;
    private int age;

    public User() {
    }

    public void sayHello() {
        System.out.println("你好");
    }
}
```

反射中常见 4 个核心类：

```
Class：表示整个类的信息，比如 User.class
Field：表示属性，比如 name、age
Method：表示方法，比如 sayHello()
Constructor：表示构造方法，比如 User()
```

可以这样理解：

```
User 类
├── Class：User 类的说明书
├── Field：属性的说明书
├── Method：方法的说明书
└── Constructor：构造方法的说明书
```

------

### 24.4 读取属性和方法

```
import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class ReflectionDemo {
    public static void main(String[] args) {

        Class<?> clazz = User.class;

        Field[] fields = clazz.getDeclaredFields();

        for (Field field : fields) {
            System.out.println("属性名：" + field.getName());
            System.out.println("属性类型：" + field.getType().getName());
        }

        Method[] methods = clazz.getDeclaredMethods();

        for (Method method : methods) {
            System.out.println("方法名：" + method.getName());
            System.out.println("返回值类型：" + method.getReturnType().getName());
        }
    }
}
```

变量解释：

```
clazz：User 类的说明书对象
fields：User 类中所有属性组成的数组
field：当前遍历到的某一个属性
methods：User 类中所有方法组成的数组
method：当前遍历到的某一个方法
```

------

### 24.5 通过反射创建对象

普通创建对象：

```
User user = new User();
```

反射创建对象：

```
Class<?> clazz = User.class;

Object obj = clazz.getConstructor().newInstance();

User user = (User) obj;
```

解释：

```
clazz.getConstructor()
获取 User 的无参构造方法

newInstance()
调用构造方法创建对象

Object obj
反射创建出来的对象默认是 Object 类型

User user = (User) obj
把 Object 强制转换成 User
```

------

### 24.6 通过反射修改 private 属性

```
Class<?> clazz = User.class;

Object obj = clazz.getConstructor().newInstance();

Field nameField = clazz.getDeclaredField("name");

nameField.setAccessible(true);

nameField.set(obj, "张三");
```

解释：

```
getDeclaredField("name")
获取 name 这个属性

setAccessible(true)
允许访问 private 属性

nameField.set(obj, "张三")
把 obj 对象里的 name 属性改成 "张三"
```

------

### 24.7 通过反射调用方法

```
Class<?> clazz = User.class;

Object obj = clazz.getConstructor().newInstance();

Method method = clazz.getDeclaredMethod("sayHello");

method.invoke(obj);
```

解释：

```
getDeclaredMethod("sayHello")
获取 sayHello 这个方法

method.invoke(obj)
让 obj 对象执行 sayHello 方法
```

相当于普通写法：

```
user.sayHello();
```

------

### 24.8 反射和框架的关系

很多框架底层都用到了反射。

#### Spring

Spring 启动时会扫描类：

```
@Service
public class UserService {
}
```

它通过反射判断：

```
这个类上有没有 @Service？
有的话，就创建对象，放进 Spring 容器。
```

------

#### `@Autowired`

```
@Autowired
private UserService userService;
```

Spring 通过反射发现：

```
这个属性需要注入 UserService
```

然后把容器里的 `UserService` 对象塞进去。

------

#### MyBatis

```
@Mapper
public interface UserMapper {
    User getById(Long id);
}
```

MyBatis 通过反射读取 Mapper 接口的方法信息，然后生成代理对象，帮你执行 SQL。

------

#### Swagger / Knife4j

Swagger 通过反射读取 Controller 上的注解：

```
@ApiOperation("员工登录")
@PostMapping("/login")
```

然后生成接口文档。

------

### 24.9 反射能做什么？

反射能做：

```
读取类名
读取属性
读取方法
读取注解
创建对象
修改属性
调用方法
```

反射一般不直接做：

```
生成 getter/setter 源代码
修改 .java 文件
```

自动生成 getter/setter 更常见的是用：

```
IDEA 自动生成
Lombok
代码生成工具
```

------

### 24.10 总结

反射的核心是：

> **把类、属性、方法、构造方法都当成对象来操作。**

最重要的一句话：

```
反射 = Java 程序在运行时读取和操作类结构的能力。
```

Spring、MyBatis、Swagger 能够自动创建对象、自动注入、自动生成接口文档，本质上都离不开反射。