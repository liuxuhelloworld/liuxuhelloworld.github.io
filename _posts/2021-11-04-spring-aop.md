AOP (Aspect-Oriented Programming) is often referred to as a tool for implementing crosscutting concerns.

The term *crosscutting concerns* refers to logic in an application that cannot be decomposed from the rest of the application and may result in code duplication and tight coupling. Logging and security are typical examples of crosscutting concerns.

It is important to understand that AOP complements OOP, rather than competing with it.

# AOP Concepts
**Joinpoints**: A joinpoint is a well-defined point during the execution of your application. Typical examples of joinpoints include a call to a method, the method invocation itself, class initialization,  and object instantiation. Joinpoints are a core concept of AOP and define the points in your application at which you can insert additional logic using AOP.

**Advice**: The code that is executed at a particular joinpoint is the advice, defined by a method in your class. There are many types of advice, such as before, which executes before the joinpoint, and after, which executes after it.

**Pointcuts**: A pointcut is a collection of joinpoints that you use to define when advice should be executed.

**Aspects**: An aspect is the combination of advice and pointcuts encapsulated in a class. This combination results in a definition of the logic that should be included in the application and where it should execute.

**Weaving**: This is the process of inserting aspects into the application code at the appropriate point. For compile-time AOP solutions, this weaving is generally done at build time. Likewise, for runtime AOP solutions, the weaving process is executed dynamically at runtime.

**Target**: An object whose execution flow is modified by an AOP process is referred to as the target object. Often you see the target object referred to as the advised object.

**Introduction**: This is the process by which you can modify the structure of an object by introducing additional methods or fields to it.

# AOP分类
## static AOP
weaving发生在build阶段，weaving的结果体现在字节码中。

AspectJ的compile-time weaving是static AOP的典型实现。
## dynamic AOP
weaving发生在runtime阶段。

Spring AOP就是dynamic AOP.

# Spring AOP
Spring AOP主要包含两部分内容：第一部分是AOP core，也称为Spring AOP API；第二部分是一组framework services that make AOP easier to use in your applications.

## 简单示例
```
public class AgentDecorator implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        System.out.print("James ");

        Object ret = invocation.proceed();

        System.out.println("!");

        return ret;
    }
}

public class AopSimpleExampleApplication {
    public static void main(String[] args) {
        Agent agent = new Agent();

        ProxyFactory proxyFactory = new ProxyFactory();
        proxyFactory.setTarget(agent);
        proxyFactory.addAdvice(new AgentDecorator());
        Agent proxy = (Agent)proxyFactory.getProxy();

        agent.speak();
        System.out.println("");
        proxy.speak();
    }
}
```

## Joinpoints in Spring
Spring AOP只支持一种joinpoint，即method invocation.

## Aspects in Spring
In Spring AOP, an aspect is represented by an instance of a class that implements the Advisor interface.

## ProxyFactory
The ProxyFactory class controls the weaving and proxy creation process in Spring AOP.

## Creating Advice in Spring
Spring支持6种advice：
- MethodBeforeAdvice
- AfterReturningAdvice
- AfterAdvice
- MethodInterceptor
- ThrowsAdvice
- IntroductionInterceptor

### MethodBeforeAdvice: 
Using before advice, you can perform custom processing before a joinpoint executes.

If the before advice throws an exception, further execution of the interceptor chain (as well as the target method) will be aborted, and the exception will propagate back up the interceptor chain.

Before advice常用于前置的参数或权限校验。

### AfterReturningAdvice
After-returning advice is executed after the method invocation at the joinpoint has finished executing and has returned a value.

If the target method throws an exception, the after-returning advice will not be run.

注意，你不能修改target的return value，但是你可以抛出异常。A good use of after-returning advice is to perform some additional error checking when it is possible for a method to return an invalid value.

### AfterAdvice
与AfterReturningAdvice的区别在于无论targe method invocation是否异常，都会执行。

### MethodInterceptor
The MethodInterceptor interface is a standard AOP Alliance interface for implementing around advice for method invocation joinpoints.

Your advice is allowed to execute before and after the method invocation, and you can control the point at which the method invocation is allowed to proceed.

### ThrowsAdvice
Throws advice is executed after a method invocation returns, but only if that invocation threw an exception.

After-throwing advice is useful in a variety of situations; it allows you to reclassify entire Exception hierarchies as well as build centralized Exception logging for your application.

ThrowsAdvice没有定义任何methods，它是一个functional interface，the first thing Spring looks for in throws advice is one or more public methods called afterThrowing().

### IntroductionInterceptor
Spring models introductions as special types of interceptors. Using an introduction interceptor, you can specify the implementation for methods that are being introduced by the advice.

## Pointcuts and Advisors
通过ProxyFactory.addAdvice()配置的advice，适用于target的所有methods，但是有的时候，你可能希望只作用于部分methods，怎么办呢？

你当然可以在advice的实现中显式地校验当前method是不是你的目标method，但是显然这种硬编码的方式不可取。一方面强耦合，一方面降低了整体性能。

正确的选择是使用pointcuts，by using pointcuts, you can configure the methods to which an advice applies.

对于是使用pointcuts还是直接在advice中校验mehtod，有这样一个概念，target affinity。如果你的advice的target affinity很小或者没有，那么就应当使用pointcuts；如果你的advice有很强的target affinity，那么直接在advice中校验method就更合适。

### Pointcut interface
在Spring中，通过实现Pointcut接口来创建pointcuts.

Pointcut接口定义了两个methods：getClassFilter()和getMethodMatcher()，它们分别返回ClassFilter和MethodMatcher instances.

#### ClassFilter
When determining whether a Pointcut applies to a particular method, Spring first checks to see whether the Pointcut interface applies to the method's class by using the ClassFilter instance returned by getClassFilter().

#### MethodMatcher
Spring支持两种MethodMatcher，static和dynamic，通过MethodMatcher的isRuntime()可以判断是哪种类型。

Spring首先通过isRuntime()判断MethodMatcher的类型。然后：
- for a static pointcut, Spring calls the matches(Method, Class<T>) method once for every method on the target, caching the return value for subsequent invocations of those methods.
- for dynamic pointcuts, 除了类似static pointcut调用matches(Method, Class<T>)外，还要在每次invocation时再调用一次matches(Method, Class<T>, Object[] args)进一步校验

static的执行效率更高，因为不用每次invocation都附加一次校验；dynamic更灵活。一般的，尽量使用static.

推荐实践：关于class的限制在getClassFilter()中控制，关于method的限制在matches(Method, Class<?>)中控制，关于arguments的限制在matches(Method, Class<?>, Object[])中控制。

### Poiontcut Implementations
事实上，你很少需要自己实现Pointcut，因为Spring提供了很多备选项。

#### StaticMethodMatcherPointcut
The StaticMethodMatcherPointcut class is intended as a base for building static pointcuts.

```
public class SimpleStaticPointcut extends StaticMethodMatcherPointcut {
    @Override
    public boolean matches(Method method, Class<?> cls) {
        return "sing".equals(method.getName());
    }

    @Override
    public ClassFilter getClassFilter() {
        return cls -> cls == GoodGuitarist.class;
    }
}
```

#### DynamicMethodMatcherPointcut
The DynamicMethodMatcherPointcut class is intended as a base for building dynamic pointcuts.

```
public class SimpleDynamicPointcut extends DynamicMethodMatcherPointcut {
    @Override
    public boolean matches(Method method, Class<?> cls) {
        System.out.println("static check for " + method.getName());
        return "foo".equals(method.getName());
    }

    @Override
    public boolean matches(Method method, Class<?> cls, Object[] args) {
        System.out.println("dynamic check for " + method.getName());
        int x = ((Integer)args[0]).intValue();
        return x != 100;
    }

    @Override
    public ClassFilter getClassFilter() {
        return cls -> cls == SampleBean.class;
    }
}
```

#### NameMatchMethodPointcut
Using NameMatchMethodPointcut, you can create a pointcut that performs simple matching against a list of method names.

NameMatchMethodPointcut继承了StaticMethodMatcherPointcut，适用于根据method name做过滤。

#### AnnotationMatchingPointcut
The AnnotationMatchingPointcut looks for a specific Java annotation on a class or method.

#### JdkRegexpMethodPointcut
The JdkRegexpMethodPointcut allows you to define pointcuts using regular expression.

相比NameMatchMethodPointcut，可以通过正则表达式更灵活地控制匹配哪些methods。注意，匹配method name时使用的是method的全路径名。

#### ComposablePointcut
The ComposablePointcut class is used to compose two or more pointcuts together with operations such as union() and intersection().

#### AspectJExpressionPointcut
The AspectJExpressionPointcut uses an AspectJ weaver to evaluate a pointcut expression in AspectJ syntax.

#### ControlFlowPointcut
ControlFlowPointcut is a special case pointcut
that matches all methods within the control flow of another method, that is, any method that is invoked either directly or indirectly as the result of another method being invoked.

Control flow pointcuts can be extremely useful, allowing you to advise on an object selectively only when it is executed in the context of another.

Suppose we have a transaction processing system, which contains a TransactionService interface as well as an AccountService interface. We would like to apply after advice so that when the AccountService.updateBalance() method is called by TransactionService.reverseTransaction(), an email notification is sent to the customer, after the account balance is updated. However, an email will not be sent under any other circumstances. In this case, the control flow pointcut will be useful.

### Advisor interface
Advisor is Spring's representation of an aspect, which is a coupling of advice and pointcuts that governs which methods shoule be advised and how they should be advised.

Advisor有两个子接口：
- PointcutAdvisor
- IntroductionAdvisor

### DefaultPointcutAdvisor
DefaultPointcutAdvisor is a simple PointcutAdvisor for associating a single Pointcut with a single Advice.

### DefaultIntroductionAdvisor

# Proxies in Spring AOP

Spring AOP的核心就是proxies: 
- when you want to create an advised instance of a class, you must use ProxyFactory to create a proxy instance of that class, first providing ProxyFactory with all the aspects that you want to be woven into the proxy
- at runtime, Spring analyzes the crosscutting concerns defined for the beans in ApplicationContext and generates proxy beans (which wrap the underlying target bean) dynamically
- instead of calling the target bean directly, callers are injected with the proxied bean. The proxy bean then analyzes the running condition and weaves in the appropriate advice accordingly

使用Spring AOP的一个限制是不能advise final classes，因为final classes不能被proxied.

## Proxy做什么用？
proxy的核心功能是intercept method invocations and, where necessary, execute chains of advice that apply to a particular method.

The management and invocation of advice is largely proxy independent and is managed by the Spring AOP framework. However, the proxy is reponsible for intercepting calls to all methods and passing them as necessary to the AOP framework for the advice to be applied.

## Proxy的实现

Spring有两种proxy实现：
- JDK dynamic proxies: 当target实现了接口时，使用JDK dynamic proxy
- CGLIB proxies: 当target没有实现接口时，使用CGLIB proxy

### JDK dynamic proxy
When you are using the JDK proxy, all method calls are intercepted by the JVM and routed to the invoke() method of the proxy. This method then determines whether the method in question is advised (by the rules defined by the pointcut), and if so, it invokes the advice chain and then the method itself by using reflection.

With the JDK proxy, all decisions about how to handle a particular method invocation are handled at runtime each time the method is invoked.

### CGLIB proxy

### JDK proxy vs CGLIB proxy
CGLIB proxy既可以代理classes，也可以代理interfaces；JDK proxy只能代理interfaces.

一般的，CGLIB proxy效率更好一些，尤其是使用frozen mode时。

## ProxyFactory
事实上，大部分情况下你不需要显式使用ProxyFactory，而是使用Spring提供的declarative AOP configuration mechanisms to take advantage of declarative proxy creation.

# Introductions in Spring AOP
## 什么是Introductions?
Introductions可以看作特殊类型的advice，a special type of around advice.

## Introductions做什么用？
By using introductions, you can introduce new functionality to an existing object dynamically. 

In Spring, you can introduce an implementation of any interface to an existing object.

也就是说，introductions allow you not only to extend the functionality of existing methods but to extend the set of interfaces and object implementations dynamically.

Introductions非常适合以declaratively地方式使用。

## 如何创建Introductions？
实现IntroductionInterceptor接口即可，Spring提供了一个默认实现，DelegatingIntroductionInterceptor，你只需要继承DelegatingIntroductionInterceptor并且实现要introduce的interfaces就可以了。

## IntroductionAdvisor
Just as you need to use PointcutAdvisor when you are working with pointcut advice, you need to use IntroductionAdvisor to add introductions to a proxy.

The default implementation of IntroductionAdvisor is DefaultIntroductionAdvisor, which should suffice for most of your introduction needs.

## Introduction vs other advices
Introduction和一般的advice有一个重要区别，对于一般的advice，it is possible for the same advice instance to be used for many objects. For introductions, the introduction advice forms part of the state of the advised object, and as a result, you must have a distinct advice instance for every advised object.

Because you must ensure that each advised object has a distinct instance of the introduction, it is often perferable to create a subclass of DefaultIntroductionAdvisor that is responsible for creating the introduction advice.

## Introduction应用示例：Object Modification Detection
Introduction的一个典型应用就是Object Modification Detection，using introductions, we can provide a flexible solution to the modification detection problem without having to write a bunch of repetitive, error-prone code.

首先我们定义一个IsModified接口。

然后，the next step is to create the code that implements IsModified and that is introducted to the objects; this is referred to as a mixin. 这里我们继承DelegatingIntroductionInterceptor并实现IsModified接口。

再然后，我们通过继承DefaultIntroductionAdvisor自定义一个Advisor.

最后，通过ProxyFactory将target和advisor整合起来。

# Declarative Configuration of Spring AOP
## 为什么要declarative configuration?
Using the declarative approach to AOP configuration is preferable to the mannul programmatic mechanism.

When you use the declarative mechanism, not only do you externalize the configuration of advice but you also reduce the chance of coding errors. You can also take advantage of DI and AOP combined to enable AOP so that it can be used in a completely transparent environment.

## Spring AOP提供了三种declarative configuration方式：

### ProxyFactoryBean
ProxyFactoryBean provides a declarative way to configure Spring's ApplicationContext when creating AOP proxies based on defined Spring beans.

```
    <bean id="grammyGuitarist" class="GrammyGuitarist" />

    <bean id="auditAdvice" class="AuditAdvice" />

    <util:list id="interceptorAdviceNames" >
        <value>auditAdvice</value>
    </util:list>

    <bean id="proxyOne" class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="target" ref="grammyGuitarist" />
        <property name="interceptorNames" ref="interceptorAdviceNames" />
    </bean>

    <bean id="documentaristOne" class="Documentarist">
        <property name="grammyGuitarist" ref="proxyOne" />
    </bean>

    <bean id="advisor" class="org.springframework.aop.support.DefaultPointcutAdvisor">
        <property name="advice" ref="auditAdvice" />
        <property name="pointcut">
            <bean class="org.springframework.aop.aspectj.AspectJExpressionPointcut">
                <property name="expression" value="execution(* sing*(..))" />
            </bean>
        </property>
    </bean>

    <util:list id="interceptorAdvisorNames">
        <value>advisor</value>
    </util:list>

    <bean id="proxyTwo" class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="target" ref="grammyGuitarist" />
        <property name="interceptorNames" ref="interceptorAdvisorNames" />
    </bean>

    <bean id="documentaristTwo" class="Documentarist">
        <property name="grammyGuitarist" ref="proxyTwo" />
    </bean>
```

### Spring aop namespace
Introduced in Spring 2.0, the aop namespace provides a simplified way (when compared to ProxyFactoryBean) to define aspects and their DI requirements in Spring applications.

```
    <aop:config>
        <aop:pointcut id="myPointcut" expression="execution(* sing*(..))"/>
        <aop:aspect id="myAspect" ref="auditAdvice">
            <aop:before method="simpleBeforeAdvice" pointcut-ref="myPointcut" />
        </aop:aspect>
    </aop:config>
```

```
    <aop:config>
        <aop:pointcut id="myPointcut" expression="execution(* sing*(Guitar)) and args(guitar) and bean(grammy*)"/>
        <aop:aspect id="myAspect" ref="advice">
            <aop:before method="simpleBeforeAdvice" pointcut-ref="myPointcut" />
        </aop:aspect>
    </aop:config>
```

```
    <aop:config>
        <aop:pointcut id="myPointcut" expression="execution(* sing*(Guitar)) and args(guitar) and bean(grammy*)"/>
        <aop:aspect id="myAspect" ref="advice">
            <aop:before method="simpleBeforeAdvice" pointcut-ref="myPointcut" />
            <aop:around method="simpleAroundAdvice" pointcut-ref="myPointcut" />
        </aop:aspect>
    </aop:config>
```

### @AspectJ-style annotations
Besides the XML-based aop namespace, you can use the @AspectJ-style annotations within your classes for configuring Spring AOP.

注意，只是@AspectJ-style，Spring still uses its own proxying mechanism for advising the target methods, not AspectJ's weaving mechanism.

```
@Component
@Aspect
public class AnnotationAdvice {
    @Pointcut("execution(* sing*(aop.declarative.config.using.aspectj.style.annotation.example.Guitar)) && args(guitar)")
    public void myPointcut(Guitar guitar) {
    }

    @Pointcut("bean(grammy*)")
    public void isGrammy() {
    }

    @Before("myPointcut(guitar) && isGrammy()")
    public void simpleBeforeAdvice(JoinPoint joinPoint, Guitar guitar) {
        if (guitar.getBrand().equals("Gibson")) {
            System.out.println("Executing: " + joinPoint.getSignature().getDeclaringTypeName() + " " +
                joinPoint.getSignature().getName());
        }
    }

    @Around("myPointcut(guitar) && isGrammy()")
    public void simpleAroundAdvice(ProceedingJoinPoint proceedingJoinPoint, Guitar guitar) throws Throwable {
        System.out.println("Before Execution: " +
            proceedingJoinPoint.getSignature().getDeclaringTypeName() + " " +
            proceedingJoinPoint.getSignature().getName() + " " +
            "brand:" + guitar.getBrand());

        Object ret = proceedingJoinPoint.proceed();

        System.out.println("After Execution: " +
            proceedingJoinPoint.getSignature().getDeclaringTypeName() + " " +
            proceedingJoinPoint.getSignature().getName() + " " +
            "brand: " + guitar.getBrand());
    }
}
```

# AspectJ
AspectJ is a fully featured AOP implementation that uses a weaving process (either compile-time or load-time weaving) to introduce aspects into your code.

AspectJ is a general-purpose aspect-oriented extension to Java born out of need to solve issues or concerns that are not well captured by traditional programming methdologies, in other words, crosscutting concerns.

## 为什么还需要AspectJ？
Spring AOP是较小规模的AOP实现，能满足你大部分需求，但总有不满足使用的时候。比如，Spring AOP只能将public nonstatic methods作为joinpoint. 此时，你就需要一个更完备的AOP实现，AspectJ正是你需要的。