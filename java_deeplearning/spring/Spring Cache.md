# Spring Cache
一般要先打依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

SpringCache会扫描你导入的缓存（redis啥的），有一个就默认选择， 有多个就需要指明操作，没指明就会由SpirngCache来选择
另外
Spring 缓存机制基于**AOP 动态代理**实现。
在同一个类内部，一个方法调用本类中加了缓存注解的方法（自调用），会绕过缓存逻辑，缓存失效。
底层数据库数据发生变更时，必须主动清理 / 失效对应的缓存数据。
分布式多服务项目中，应当使用 Redis 这类共享式分布式缓存，不要使用进程内本地内存缓存。
### @EnableCaching
一般加启动类，表示启用Spring Cache

### @Cacheable
在方法执行前先查询缓存中是否有数据， 有就直接返回，无则调用方法
基于代理对象，查询先会进入代理对象
```java
@Cacheable(value = "products", key = "#id")
public Product getProduct(Long id) {
    return repository.findById(id).orElse(null);
}
```

### @CachePut
将方法的返回值放到缓存之中
会带两个参数，一个是CacheName，另一个是key值，一般会请求到传入参数或返回值，传入参数用#参数变量名直接取，返回值用#result，也可以用#p?指定取第几个参数

```java
@CachePut(value = "products", key = "#product.id")
public Product updateProduct(Product product) {
    return repository.save(product);
}
```

### @CacheEvict
将一条或多条数据从缓存中删除
```java
@CacheEvict(value = "products", key = "#id")
public void deleteProduct(Long id) {
    repository.deleteById(id);
}
```

### @Caching
结合多条cache操作
```java
@Caching(
    put = {
        @CachePut(value = "products", key = "#product.id")
    },
    evict = {
        @CacheEvict(value = "productList", allEntries = true) // allEntries是删除value内所有的
    }
)
public Product updateProduct(Product product) {
    return repository.save(product);
}
```

在缓存中，缓存的key为对应的java为$(value)::$(key),也就是注解括号里的东西，
缓存的value就是java的返回值