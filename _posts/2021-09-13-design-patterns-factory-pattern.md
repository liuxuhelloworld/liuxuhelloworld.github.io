为什么需要factory pattern？There is more to making objects than just using the **new** operator. Instantiation is an activity that shouldn't always be done in public and can often lead to coupling problems. Factory patterns can help save you from embarrasing dependencies.

```java
Duck duck;
if (xxx1) { 
		duck = new MallardDuck(); 
} else if (xxx2) { 
		duck = new DecoyDuck(); 
} else if (xxx3) { 
		duck = new RubberDuck(); 
}
```
When you see code like this, you know that when it comes time for changes or extensions, you will have to reopen this code and examine what needs to be added (or deleted). Often this kind of code ends up in several parts of the application making maintenance and updates more difficult and error-prone.

# Factory Method
如何解决上面的问题呢？首先，我们可以把这部分代码单独拎出来，比如：

```java
public class SimplePizzaFactory {
    public Pizza createPizza(String type) {
        Pizza pizza = null;
        if (type.equals("cheese")) { 
        		pizza = new CheesePizza(); 
        } else if (type.equals("pepperoni")) { 
        		pizza = new PepperoniPizza(); 
        } else if (type.equals("clam")) { 
        		pizza = new CalmPizza(); 
        } else if (type.equals("veggie")) { 
        		pizza = new VeggiePizza(); 
        }
        return pizza;
    }
}
```

这种方法我们称为factory method,  factory method不是design pattern，只是一种编程规范，by encapsulating the pizza creation in one class, we now have only one place to make modifications when the implementation changes.