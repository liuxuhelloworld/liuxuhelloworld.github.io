# BeanNameAware
The **BeanNameAware** interface, which can be implemented by a bean that wants to obtain its own name, has a simple method: **setBeanName(String)**. Spring calls the **setBeanName()** method after it has finished configuring your bean but before any life-cycle callbacks are called.

```java
public class Singer implements BeanNameAware {
    private String name;

    @Override
    public void setBeanName(String name) {
        this.name = name;
    }
}
```

# ApplicationContextAware
**ApplicationContextAware** is a Spring-specific interface that forces an implementation of a setter for an **ApplicationContext** object. It is automatically detected by the Spring IoC container, and the **ApplicationContext** that the bean is created in is injected into it. This is done after the constructor of the bean is called, so obviously using **ApplicationContext** in the constructor will lead to a **NullPointerException**.

Using the **ApplicationContextAware** interface, it is possible for your beans to get a reference to the **ApplicationContext** instance that configured them. The main reason this interface was created is to allow a bean to access Spring's **ApplicationContext** in your application, for example, to acquire other Spring beans programmatically, using **getBean()**. You should, however, avoid this practice and use dependency injection to provide your beans with their collaborators. If you use the lookup-based **getBean()** approach to obtain dependencies when you can use dependency injection, you are adding unnecessary complexity to your beans and coupling them to the Spring framework without any good reason.

```java
public class ShutdownHookBean implements ApplicationContextAware {
    @Override
    public void setApplicationContext(ApplicationContext ctx) throws BeansException {
        if (ctx instanceof GenericApplicationContext) {
            ((GenericApplicationContext)ctx).registerShutdownHook();
        }
    }
}
```