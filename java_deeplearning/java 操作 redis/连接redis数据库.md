## 配置的相关代码模板


```java
    @Configuration // 配置类，Spring 启动时会加载该类，并执行其中的 @Bean 方法
    @Slf4j // Lombok 自动生成 log 日志对象
    public class RedisConfiguration {
    /**
     * 创建 RedisTemplate，并交给 Spring IOC 容器管理
     *
     * @param redisConnectionFactory Redis 连接工厂，由 Spring Boot 自动创建并注入
     * @return 配置好的 RedisTemplate
     */
    @Bean
    public RedisTemplate redisTemplate(RedisConnectionFactory redisConnectionFactory) {
    
        // 输出日志，方便观察程序是否成功创建 RedisTemplate
        log.info("开始创建 RedisTemplate 对象...");
    
        // 创建 RedisTemplate，它是 Spring 操作 Redis 的核心工具类
        RedisTemplate redisTemplate = new RedisTemplate();
    
        // 设置连接工厂
        // 告诉 RedisTemplate 应该连接哪一个 Redis 服务器
        redisTemplate.setConnectionFactory(redisConnectionFactory);
    
        // 设置 Key 的序列化方式
        // 默认使用 JDK 序列化，保存到 Redis 后会显示为乱码
        // StringRedisSerializer 会将 Key 按普通字符串(UTF-8)存储，方便查看和管理
        redisTemplate.setKeySerializer(new StringRedisSerializer());
    
        // 返回配置好的 RedisTemplate，Spring 会将它放入 IOC 容器
        return redisTemplate;
    }
}
```
## 相关的操作

### String 类型

```java
   //模板为redisTemplate.opsValue.操作();
   
   //opsValue是操作redis中String对象的， 其包括了key与value值，
   redisTemplate.opsValue.set(key, value);
   redisTemplate.opsValue.set(key, value, timeout, (一般用TimeUnit.时间类型));
   
   //setAbsent参数与上面一致，这表示如果不存在这个key就创建，存在就不改变
   redisTemplate.opsValue.setAbsent(...);
```

### hash类型
```java
	//hash类型操作需要创建对象
	HashOperations hashOperations = redisTemplate.opForHash();
	
	// hash表的插入操作
	hashOperations.put(key, hashkey, value);
	
	// hash表的获取操作返回值看序列化操作
	hashOperations.get(key, hashkey);
	
	// 用keys可以获得这个hash表的所有键值(对应hashkey) 返回一个Set
	hashOperations.keys(key);
	
	// 用values 可以获取hash表中所有值， 返回一个List
	hashOperations.values(key);
	
	// 删除
	hashOperations.delete(key, hashkey);
	
```



### list类型
```java
	//List类型操作需要创建对象
	ListOperations listOperations = redisTemplate.opForList();
	
	// List的插入操作
	ListOperations.leftPushAll(key, value1, value2, ...);
	ListOperations.leftPush(key, value);
	
	// 范围查询， start从0开始， -1代表链表结尾
	List mylist = listOperations.range(key, start, end);
	
	// 弹出
	ListOperations.rightPop(key);
	
	// 得到list的大小, 返回long
	listOperations.size(key);
	
```

### set类型
```java
	//Set类型操作需要创建对象
	SetOperations setOperations = redisTemplate.opForSet();
	
	// Set的插入操作
	SetOperations.add(key, value1, value2, ...);
	
	// 获取Set中的所有元素
	Set members = SetOperations.members(key);

	// 得到Set的大小, 返回long
	SetOperations.size(key);

	// intersect, 计算两个集合的交集
	SetOperations.intersect(key1, key2);
	
	// union, 计算两个集合的并集
	SetOperations.union(key1, key2);
	
	// 删除集合中的元素
	SetOperations.remove(key, value1, value2, ...);
```

### zset有序集合类型
```java
	//ZSet类型操作需要创建对象
	ZSetOperations zSetOperations = redisTemplate.opForZSet();
	
	// ZSet的插入操作
	zSetOperations.add(key, value1, score)
	
	// 获取Set中的元素，分数范围，-1表示无穷大
	Set zset1 = zSetOperations.range(key, start, end);
	
	// 为某个元素加分
	zSetOperations.incrementScore(key, value, score);
	
	// 删除
	zSetOperations.remove(key, value1, value2, ...);
```



### 通用命令
```java
	// 获取集合类型， pattern是通配符匹配
	Set keys = redisTemplate.keys(pattern); 
	
	// 判断某个类型是否存在, 返回bool
	redisTemplate.haskey(key);
    // 查询某个key的类型
    redisTemplate.type(key);
    
    // 删除
    redisTemplate.delete(key); 
```