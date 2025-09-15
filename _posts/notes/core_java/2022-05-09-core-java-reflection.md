1. 常用的reflection方法：
Class.getDeclaredMethods()
Class.getDeclaredField(String name)
Class.getMethod(String name, Class<?>... parameterTypes)
Method.getAnnotation(Class<T> annotationClass)

5.7 Reflection
(1)a program that can analyze the capabilities of classes is called reflective;

(2)The Class class
当你的程序运行时，Java runtime system会维护所有objects的runtime type identification信息，即每个object所属的class是什么，这一信息最典型的应用就是虚拟机以此为根据选择正确的methods来执行
那么给定一个object，如何知道它所属的class是什么呢？ the getClass() method in the Object class returns an instance of Class type
Employee e;
Class cl = e.getClass();
也就是说，我们有一个名为Class的类，这个Class类用来描述properties of a particular type
the virtual machine manages a unique Class object for each type. Therefore, you can use the == operator to compare Class objects
if (e.getClass() == Employee.class) {...}
T.class
有object，你可以调用getClass()得到对应的Class object；没有object，只有type呢？If T is any Java type, then T.class is the mathcing Class object
Class cl1 = Random.class;
Class cl2 = int.class;
Class cl3 = Double[].class;
注意，a Class object really describes a type, which may or may not be a class
常用methods
getName()
Class最常用的method，返回类名称
forName(String className)
这是个static method，你可以通过这个method由class name得到一个Class object
String className = "java.util.Random";
Class cl = Class.forName(className);
要求className是有效的class or interface name，否则抛出一个checked exception
newInstance()
String className = "java.util.Random";
Object m = Class.forName(className).newInstance();

(3)At startup, the class containing your main method is loaded. It loads all classes that it needs. Each of those loaded classes loads the classes that it needs, and so on. That can take a long time for a big application, frustrating the user. You can give the users of your program an illusion of a faster start with the following trick. Make sure the class containing the main method does not explicitly refer to other classes. In it, display a splash screen. Then manually force the loading of other classes by calling Class.forName;