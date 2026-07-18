## @SpringBootApplication

@SpringBootApplication 是一个简化开发的复合注解，它整合了以下全部注解的作用：
@Configuration
将当前类标记为应用上下文的 Bean 定义配置源。
@EnableAutoConfiguration
Spring Boot 会根据你项目中引入的依赖包，尝试自动对 Spring 应用完成配置。
@ComponentScan
告知 Spring 去扫描项目里其他的组件、配置类与业务服务。
若未手动指定扫描包路径，会从标注该注解的类所在包开始，递归扫描其子包。
