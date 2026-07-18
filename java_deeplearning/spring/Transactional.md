# Transactional
事物注解，打在方法上，如果方法失败，就会回滚，防止做一半停止

```java
@Service
public class UserService {
    @Transactional
    public void transfer() {
        userMapper.minusMoney(1,100);V
        int i = 1 / 0;   // 故意报错
        userMapper.addMoney(2,100);
    }
}
```

这个注解基于AOP
同样也基于代理对象， 所以自调用也是不生效的