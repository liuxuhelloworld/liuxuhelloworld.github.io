# FactoryBean
One of the problems that you will face when using Spring is how to create and then inject dependencies that cannot be created simply by using the **new** operator. To overcome this problem, Spring provides the **FactoryBean** interface that acts as an adapter for objects that cannot be created and managed using the standard Spring semantics.

Simply put，a **FactoryBean** is a bean that acts as a factory for other beans. **FactoryBean**s are configured within your **ApplicationContext** like any normal bean, but when Spring uses the **FactoryBean** interface to satisfy a dependency or lookup request, it does not return **FactoryBean**; instead, it invokes the **FactoryBean.getObject()** method and returns the result of that invocation.

**FactoryBean**s are the perfect solution when you are working with classes that cannot be created by using the **new** operator. If you work with objects that are created by using a factory method and you want to use these classes in a Spring application, create a **FactoryBean** to act as an adapter, allowing your classes to take full advantage of Spring's IoC capabilities.

```java
public class MessageDigestFactoryBean implements FactoryBean<MessageDigest>, InitializingBean {
    private String algorithmName = "MD5";

    private MessageDigest messageDigest = null;

    @Override
    public MessageDigest getObject() throws Exception {
        return messageDigest;
    }

    @Override
    public Class<MessageDigest> getObjectType() {
        return MessageDigest.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        messageDigest = MessageDigest.getInstance(algorithmName);
    }

    public void setAlgorithmName(String algorithmName) {
        this.algorithmName = algorithmName;
    }

}
```

# factory-bean and factory-method
Sometimes you need to instantiate JavaBeans that were provided by a non-Spring-powered third-party application. You don't know how to instantiate that class, but you know that the third-party application provides a class that can be used to get an instance of the JavaBean that your Spring application needs. In this case, Spring bean's **factory-bean** and **factory-method** attributes in the \<bean\> tag can be used.

```xml
<bean id="shaDigestFactory" class="MessageDigestFactory">
    <property name="algorithmName" value="SHA1" />
</bean>

<bean id="defaultDigestFactory" class="MessageDigestFactory" />

<bean id="shaDigest" factory-bean="shaDigestFactory" factory-method="createInstance" />

<bean id="defaultDigest" factory-bean="defaultDigestFactory" factory-method="createInstance" />

<bean id="digester" class="MessageDigester">
    <property name="digest1" ref="shaDigest" />
    <property name="digest2" ref="defaultDigest" />
</bean>
```