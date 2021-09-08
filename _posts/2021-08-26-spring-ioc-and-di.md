# 什么是IoC?
IoC is a technique that externalizes the creation and management of component dependencies.

At its core, IoC aims to offer a simpler mechanism for provisioning component dependencies and managing these dependencies throughout their life cycles.

# IoC分类
IoC可以分为两类：
- dependency lookup
- dependency injection

## dependency lookup
Dependency lookup is a much more traditional approach, and it seems more familiar to Java programmers.

Dependency lookup可以分为两类：
- dependency pull: in dependency pull, dependencies are pulled from a registry as required
- contextualized dependency lookup (CDL): in CDL, lookup is performed against the container that is managing the resource, not from some central registry, and it is usually performed at some set point

## dependency injection
Dependency injection appears counterintuitive at first, but it is actually much more flexible and usable than dependency lookup.

Dependency injection可以分为两类：
- constructor dependency injection
- setter dependency injection

### constructor dependency injection
Constructor dependency injection occurs when a component's dependencies are provided to it in its constructor.
```java
public class ConstructorInjection {
    private Dependency dependency;
    
    public ConstructorInjection(Dependency dependency) {
        this.dependency = dependency;
    }
}
```

### setter dependency injection
In setter dependency injection, the IoC container injects a component's dependencies via JavaBean-style setter methods.
```java
public class SetterInjection {
    private Dependency dependency;
    
    public void setDependency(Dependency dependency) {
        this.dependency = dependency;
    }
}
```

### setter injection vs constructor injection
Constructor injection is particularly useful when you absolutely must have an instance of the dependency class before your component is used. Constructor injection also helps achieve the use of immutable objects.

Setter injection allows dependencies to be declared on an interface, but this is an often-touted benefit. Unless you are absolutely sure that all implementations of a particular business interface require a particular dependency, let each implementation class define its own dependencies and keep the business interface for business methods. 
```java
public interface Oracle {
		String defineMeaningOfLife();
}

public class BookwormOracle implements Oracle {
		private Encyclopedia encyclopedia;
		public void setEncyclopedia(Encyclopedia encyclopedia) {
				this.encyclopedia = encyclopedia;
		}
		
		@Override
		public String defineMeaningOfLife() {
				return "Encyclopedias are a waste of money - go see the world instead";
		}
}
```

However, placing setters and getters for configuration parameters in the business interface is a good idea and makes setter injection a valuable tool.
```java
public interface NewsletterSender {
		void setSmtpServer(String smtpServer);
		String getSmtpServer();
		void setFromAddress(String fromAddress);
		String getFromAddress();
		void send();
}
```

# IoC and DI in Spring
In a Spring-based application, it is always preferable to use dependency injection. Wherever it is possible to use dependency injection with Spring, you should do so; otherwise, you can fall back on the dependency lookup capabilities.

# BeanFactory
The core of Spring's dependency injection container is the **BeanFactory** interface. **BeanFactory** is responsible for managing components, including their dependencies as well as their life cycles.

If you application needs only DI support, you can interact with the Spring DI container via the **BeanFactory** interface. In this case, your application must create an instance of a class that implements the **BeanFactory** interface and configures it with bean and dependency information. After this is complete, your application can access the beans via **BeanFactory** and get on with its processing.

Although the **BeanFactory** can be configured programmatically, it is more common to see it configured externally using some kind of configuration file. Internally, bean configuration is represented by instances of classes that implement the **BeanDefinition** interface. The bean configuration stores information not only about a bean itself but also about the beans that it depends on. For any **BeanFactory** implementation classes that also implement the **BeanDefinitionReader** interface, you can read the **BeanDefinition** data from a configuration file, using either **PropertiesBeanDefinitionReader** or **XmlBeanDefinitionReader**. **PropertiesBeanDefinitionReader** reads the bean definition from properties files, while **XmlBeanDefinitionReader** reads from XML files.

So, you can identify your beans within **BeanFactory**; each bean can be assigned an ID, a name, or both. A bean can also be instantiated without any ID or name (known as anonymous bean) or as an inner bean within another bean. Each bean has at least one name but can have any numbers of names. Any names after the first are considered aliases for the same bean. You use bean IDs or names to retrieve a bean from **BeanFactory** and also to establish dependency relationships.

```java
		public static void main(String[] args) {
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();

        XmlBeanDefinitionReader rdr = new XmlBeanDefinitionReader(factory);
        rdr.loadBeanDefinitions(new ClassPathResource("beans.xml"));

        Oracle oracle = (Oracle) factory.getBean("oracle");

        System.out.println(oracle.defineMeaningOfLife());
    }
```

# ApplicationContext
The **ApplicationContext** interface is an extension to **BeanFactory**. In developing Spring-based applications, it's recommended that you interact with Spring via the **ApplicationContext** interface. Spring supports the bootstrapping of **ApplicationContext** by manual coding (instantiate it manually and loads the appropriate configuration) or in a web container environment via **ContextLoaderListener**. 

## bean definition example 1
```xml
<bean id="provider" class="HelloWorldMessageProvider"/>

<bean id="renderer" class="StandardOutMessageRenderer" p:messageProvider-ref="provider"/>
```
## bean definition example 2
To create bean definitions using annotations, the bean classes must be annotated with the appropriate stereotype annotation, and the methods or constructors must be annotated with **@Autowired** to tell the Spring IoC container to look for a bean of that type and use it as an argument when calling that method.

```java
@Component("provider")
public class HelloWorldMessageProvider implements MessageProvider {
    @Override
    public String getMessage() {
        return "Hello, World";
    }
}

@Service("renderer")
public class StandardOutMessageRenderer implements MessageRenderer {

    private MessageProvider messageProvider;

    @Override
    public void render() {
        if (messageProvider == null) {
            throw new RuntimeException("messageProvider is null");
        }
        System.out.println(messageProvider.getMessage());
    }

    @Override
    @Autowired
    public void setMessageProvider(MessageProvider messageProvider) {
        this.messageProvider = messageProvider;
    }

    @Override
    public MessageProvider getMessageProvider() {
        return messageProvider;
    }
}
```

```xml
<context:component-scan base-package="xxx"/>
```
The \<context:component-scan\> tag tells Spring to scan the code for injectable beans under the package specified.

## bean definition example 3
```java
@Configuration
public class HelloWorldConfig {

    @Bean
    public MessageProvider provider() {
        return new HelloWorldMessageProvider();
    }

    @Bean
    public MessageRenderer renderer() {
        MessageRenderer renderer = new StandardOutMessageRenderer();
        renderer.setMessageProvider(provider());
        return renderer;
    }
}
```

## bean definition example 4
```java
@Component("provider")
public class HelloWorldMessageProvider implements MessageProvider {
    @Override
    public String getMessage() {
        return "Hello, World";
    }
}

@Service("renderer")
public class StandardOutMessageRenderer implements MessageRenderer {

    private MessageProvider messageProvider;

    @Override
    public void render() {
        if (messageProvider == null) {
            throw new RuntimeException("messageProvider is null");
        }
        System.out.println(messageProvider.getMessage());
    }

    @Override
    @Autowired
    public void setMessageProvider(MessageProvider messageProvider) {
        this.messageProvider = messageProvider;
    }

    @Override
    public MessageProvider getMessageProvider() {
        return messageProvider;
    }
}
```

```java
@ComponentScan(basePackages = {"xxx"})
@Configuration
public class HelloWorldConfiguration {
}
```
To be able to look for bean definitions inside Java classes, component scanning has to be enabled. This is done by annotating the configuration class with **@ComponentScanning** that is the equivalent of the \<context:component-scanning\> element.

## @Autowired vs @Resource vs @Inject
@Resource是JSR-250定义的annotation，@Inject是JSR-299定义的annotation，它们的作用和@Autowired类似。

## 注入simple values
```
<bean id="injectSimple" class="InjectSimple">
    <property name="name" value="John Mayer" />
    <property name="age" value="39" />
    <property name="height" value="1.92" />
    <property name="programmer" value="false" />
    <property name="ageInSeconds" value="1241401112" />
</bean>
```

```
@Service("injectSimple")
public class InjectSimple {
    @Value("John Mayer")
    private String name;

    @Value("39")
    private int age;

    @Value("1.92")
    private float height;

    @Value("false")
    private boolean programmer;

    @Value("1241401112")
    private Long ageInSeconds;
    
    ......
}
```

## 注入SpEL Values
```
<bean id="injectSimpleConfig" class="InjectSimpleConfig" />

<bean id="injectSimple" class="InjectSimple">
    <property name="name" value="#{injectSimpleConfig.name}" />
    <property name="age" value="#{injectSimpleConfig.age + 1}" />
    <property name="height" value="#{injectSimpleConfig.height}" />
    <property name="programmer" value="#{injectSimpleConfig.programmer}" />
    <property name="ageInSeconds" value="#{injectSimpleConfig.ageInSeconds}" />
</bean>
```

```
@Service("injectSimple")
public class InjectSimple {
    @Value("#{injectSimpleConfig.name}")
    private String name;

    @Value("#{injectSimpleConfig.age + 1}")
    private int age;

    @Value("#{injectSimpleConfig.height}")
    private float height;

    @Value("#{injectSimpleConfig.programmer}")
    private boolean programmer;

    @Value("#{injectSimpleConfig.ageInSeconds}")
    private Long ageInSeconds;
    
    ......
}
```
## 嵌套ApplicationContext
Spring支持嵌套ApplicationContext，child context可以reference parent context的beans.

## ApplicationContextAware
如果某个bean需要access ApplicationContext，那么让这个bean实现ApplicationContextAware接口就可以了。

This is a Spring-specific interface that forces an implementation of a setter for an ApplicationContext
object. It is automatically detected by the Spring IoC container, and the ApplicationContext that the bean
is created in is injected into it.

注意ApplicationContext的注入发生在调用constructor后面，所以不能在constructor内使用ApplicationContex.

## Method Injection
### Lookup Method Injection
Lookup Method Injection主要用来解决不同生命周期的beans间的依赖问题，比如，a singleton依赖a nonsingleton，in this situation, both setter and constructor injection result in the singleton maintaining a single instance of what should be a nonsingleton bean. In some cases, you will want to have the singleton bean obtain a new instance of the nonsingleton every time it requires the bean in question.

Lookup Method Injection works by having your singleton declare a method, the lookup method, which returns an instance of the nonsingleton bean. When you obtain a reference to the singleton in your application, you are acutally receiving a reference to a dynamically created subclass on which Spring has implemented the lookup method.

一般的，我们将lookup method和singleton都定义为abstract.

不要使用不必要的Lookup Method Injection，Consider a situation in which you have three singletons that share a dependency in common. You want each singleton to have its own instance of the dependency, so you create the dependency as a nonsingleton, but you are happy with each singleton using the same instance of the collaborator throughout its life. In this case, setter injection is the ideal solution; Lookup Method Injection just adds unnecessary overhead.

### Method Replacement
Using method replacement, you can replace the implementation of any method on any beans arbitrarily without having to change the source of the bean you are modifying.

Internally, you achieve this by creating a subclass of the bean class dynamically. You use CGLIB and redirect calls to the method you want to replace to another bean that implements the MethodReplacer interface.

## Bean Naming
Spring有一套复杂的、灵活的bean命名机制。

Every bean must have at least one name that is unique within the containing ApplicationContext.

Spring如何确定bean的name呢？以XML配置为例，如果你设置了id attribute，那么id就是bean name；如果没有设置id attribute，那么就取name attribute的第一个值；如果都没有设置，那么就取bean的class name作为bean name.

```
<bean id="string1" class="java.lang.String" />

<bean name="string2" class="java.lang.String" />

<bean class="java.lang.String" />
```

一般的，我们通过id attribute设置唯一的bean name，然后可以通过name aliasing关联多个names.

### Bean Name Aliasing
Spring允许一个bean有多个别名，比如：
```
<bean id="john" name="john johnny,jonathan;jim" class="java.lang.String" />
<alias name="john" alias="ion" />
```
什么时候使用别名呢？

## Bean Instantiation Mode
默认的，所有Spring beans都是singletons，this means Spring maintains a single instance of the bean, all dependent objects use the same instance, and all calls to ApplicationContext.getBean() return the same instance.

Spring有哪些instantiation mode呢？
- singleton: the default scope. Only one object will be created per Spring IoC container
- prototype: a new instance will be created by Spring when requested by the application
- request: for web application use. When using Spring MVC for web applications, beans with *request* scope will be instantiated for every HTTP request and then be destroyed when the request is completed
- session: for web application use. When using Spring MVC for web applications, beans with *session* scope will be instantiated for every HTTP session and then be destroyed when the session is over
- global session: for portlet-bases web applications. The *global* session scope beans can be shared among all portlets within the same Spring MVC-powered portal application
- therad: a new bean instance will be created by Spring when requested by a new thread, while for the same thread, the same bean instance will be returned. Note that this scope is not registered by default
- custom: custom bean scope that can be created by implementing the interface org.springframework.beans.factory.config.Scope and registering the custom scope in Spring's configuration


### singleton instantiation mode
The term singleton is used interchangeably in Java to refer to two distinct concepts: an object that has a single instance within the application and the Singleton design pattern. The problem arises when people confuse the need for singleton instances with the need to apply the Singleton pattern.

singleton instantiation mode适用于以下场景：
- shared object with no state: you have an object that maintains no state and has many dependent objects. Because you do not need synchronization if there is no state, you do not need to create a new instance of the bean each time a dependent object needs to use it for some processing
- shared object with read-only state: 与no state类似，不需要synchronization
- shared object with shared state: if you have a bean that has state that must be shared, singleton is the ideal choice. In this case, ensure that your synchronization for state writes is as granular as possible
- high-throughput objects with writable states: if you have a bean that is used a great deal in your application, you may find that keeping a singleton and synchronizing all write access to the bean state allows for better performance than constantly creating hundreds of instances of the bean. You will find that this approach is particularly useful when your application creates a large number of instances over a long period of time, when your shared object has only a small amount of writable states, or when the instantiation of a new instance is expensive

以下场景就不适合使用singleton instantiation mode:
- objects with large number of writable states: if you have a bean that has a lot of writable states, you may find that the cost of synchronization is greater than the cost of creating a new instance to handle each request from a dependent object
- objects with private state: some dependent objects need a bean that has private state so that they can conduct their processing separately from other objects that depend on that bean

### prototype instantiation mode
the prototype scope instructs Spring to instantiate a new instance of the bean every time a bean instance is required by the application.

# 依赖解析
一般的，Spring根据xml或annotation的配置就可以完成依赖解析，以正确的顺序创建并配置beans.

# Autowiring Bean Properties
Spring支持以下几种bean properties autowiring：
- no, 默认不使用autowiring
- byName, Spring attempts to wire each property to a bean of the same name in ApplicationContext
- byType, Spring attempts to wire each of the properties on the target bean by automatically using a bean of the same type in ApplicationContext
- constructor, 与byType类似，区别是byType使用的是setters
- default, Spring自动在byType和constructor之间选择

当基于annotation做配置时，默认的autowiring是byType.

一般的，不要依赖bean properties的autowiring，对于小工程，autowiring可能为你节省了一些工作，但是对于稍大的工程，还是显式地自己维护依赖关系更简洁清楚。

# Spring Aware
## BeanNameAware
```
public class Singer implements BeanNameAware {
    private String name;

    @Override
    public void setBeanName(String name) {
        this.name = name;
    }
}
```
## ApplicationContextAware
```
public class ShutdownHookBean implements ApplicationContextAware {
    @Override
    public void setApplicationContext(ApplicationContext ctx) throws BeansException {
        if (ctx instanceof GenericApplicationContext) {
            ((GenericApplicationContext)ctx).registerShutdownHook();
        }
    }
}
```