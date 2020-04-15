# AOP 切面运行两次的问题

问题：通过 spring aop 开发一个切面，开发过程中发现，transaction interceptor 执行了两次，transaction interceptor是 spring 内置的用于事务管理的 AOP 拦截器。

通过 debuger 观察运行时数据结构，发现代理类对同一个类代理了两次。

在 main 方法中添加日志代码，打印出所有的 `AbstractAdvisorAutoProxyCreator`

main 方法中添加的日志代码：
```
    ConfigurableApplicationContext context = SpringApplication.run(***Application.class, args);
    Map<String, AbstractAdvisorAutoProxyCreator> map = context.getBeansOfType(AbstractAdvisorAutoProxyCreator.class);
    map.forEach((name, obj) -> {
        log.info("bean name: {}, class: {}, toString: {}", name, obj.getClass(), obj);
    });
```

打印结果：
```
    bean name: org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator#0, class: class org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator, toString: proxyTargetClass=true; optimize=false; opaque=false; exposeProxy=false; frozen=false
    bean name: org.springframework.aop.config.internalAutoProxyCreator, class: class org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator, toString: proxyTargetClass=true; optimize=false; opaque=false; exposeProxy=false; frozen=false
```

日志显示 AbstractAdvisorAutoProxyCreator 的两个派生类都参与了 AOP 代理，通过阅读代码，可以看出 DefaultAdvisorAutoProxyCreator 和 AnnotationAwareAspectJAutoProxyCreator 这两个类都继承自AbstractAdvisorAutoProxyCreator，这两个类的功能都是为当前 bean factory 中的切面创建代理类，区别是AnnotationAwareAspectJAutoProxyCreator 额外支持 aspectj annotation。

现有项目是一个 spring-boot 项目，且已经依赖了 aop starter，又显式配置了 DefaultAdvisorAutoProxyCreator bean，所以是有问题的，aop 功能只需通过引入 starter 开启，没有必要显式地配置一个 DefaultAdvisorAutoProxyCreator，移除项目 spring beans 配置中的类名为 DefaultAdvisorAutoProxyCreator 的 bean 之后，问题解决。
