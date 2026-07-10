## Component

`@Component` 写在**类上**，表示：

> 这个类交给 Spring 管理，由 Spring 自动创建对象。

```
@Component
public class UserService {
}
```

使用时可以自动注入：

```
@Autowired
private UserService userService;
```