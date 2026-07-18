## @Scheduled

**用前需要先在启动类加入@EnableScheduling**
**需要被定时调度的方法返回值必须为 void，并且不能携带任何入参。**
倘若该方法需要调用应用上下文里的其他对象，一般通过依赖注入的方式完成对象装配。

@Scheduled 属于可重复注解。如果同一个方法上标注了多条 @Scheduled 定时配置，每条配置都会被单独解析生效，各自拥有独立的触发器触发执行。

下面这个任务会以每5s一个间隔执行
```java
@Scheduled(fixedDelay = 5000)
public void doSomething() {
	// something that should run periodically
}
```
另一种写法
```java
@Scheduled(fixedRate = 5, timeUnit = TimeUnit.SECONDS)
public void doSomething() {
	// something that should run periodically
}
```

第一次运行时间为1s，然后5秒一次的间隔
```java
@Scheduled(initialDelay = 1000, fixedRate = 5000)
public void doSomething() {
	// something that should run periodically
}
```

不加fixedRate就只会运行一次
```java
@Scheduled(initialDelay = 1000)
public void doSomething() {
	// something that should run periodically
}
```

也可以有一些其他写法如cron表达式啥的，自定义时间周期
```java
@Scheduled(cron="*/5 * * * * MON-FRI")
public void doSomething() {
	// something that should run on weekdays only
}
```