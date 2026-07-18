`@Bean` 写在**方法上**，表示：

> 这个方法返回的对象，交给 Spring 管理。

通常写在 `@Configuration` 配置类中：

```
@Configuration
public class AppConfig {

    @Bean
    public UserService userService() {
        return new UserService();
    }
}
```

简单理解：

> 你自己创建对象，然后交给 Spring 管理。