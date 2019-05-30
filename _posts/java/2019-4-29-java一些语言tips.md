---
layout: post
title: java一些语言tips
categories: java
tag: tips
---

- https://github.com/Snailclimb/JavaGuide  Java学习+面试指南

## maven

```bash
mvn clean package -DskipTests
```

## java 代理 proxy

> https://www.jianshu.com/p/28286f460f1e

### 介绍

为程序中已经存在的许多具有相同接口的目标类的各个方法增加功能，比如记录日志、计算方法运行时间、权限管理、异常处理等

### 静态代理

要统计一个类的某个方法的运行时间，这个类没有源代码，如何增加代理。

- 目标类

```java
public class Hello {
    public void sayHello(){
        System.out.println("Hello Java!");
    }
}
```

- 自定义代理类

```java
public class HelloProxy {

    private Hello hello = new Hello();
    /**
     * 统计sayHello方法所用的时间
     */
    public void sayHello(){
        //前置功能代码
        long startTime = System.currentTimeMillis();
        //调用目标方法
        hello.sayHello();
        //后置功能代码
        long endTime = System.currentTimeMillis();
        System.out.println("Hello类的sayHello方法花费了:" + (endTime - startTime) + "毫秒");
    }
}
```

显然,采用静态代理，不同接口的类需要不同的代理类。

### 动态代理

![](/styles/images/2019-4/javaproxy.png)

- JVM虚拟机可以在运行期动态生成出类的字节码，这种动态生成的类往往被用作代理类，即动态代理类。
- JVM生成的动态类必须实现一个或多个接口，所以，JVM生成的动态类只能用作具有相同接口的目标代理类。

- 添加 java.lang.reflect.Proxy 类

- 动态生成类的字节码,并打印生成类的构造函数

```java
    /**
     * 动态生成类的字节码,并打印生成类的构造函数
     */
    public static void getConStructors(){

        //动态生成代理类
        //ClassLoader:  每一个Class就必须有一个类加载器加载进来的，比如每个人都有一个妈妈。既然需要JVM动态生成Java类，所以要为动态生成类的字节码指定类加载器
        //Class Interfaces: 动态生成的字节码实现了哪些接口
        Class clazzProxy1 = Proxy.getProxyClass(Collection.class.getClassLoader(), Collection.class);

        //获取这个代理类的构造方法
        Constructor[] constructors = clazzProxy1.getConstructors();

        System.out.println("---------------------begin Construstors-----------------");
        //遍历构造方法
        for (Constructor constructor: constructors) {
            //获取每个名称
            String name = constructor.getName();
            StringBuilder sb = new StringBuilder(name);
            sb.append("(");
            //获取每个构造方法的参数类型
            Class[] clazzTypes = constructor.getParameterTypes();
            for (Class clazzType : clazzTypes) {
                sb.append(clazzType.getName()).append(".");
            }
            if(clazzTypes != null && clazzTypes.length != 0){
                sb.deleteCharAt(sb.length() - 1);
            }
            sb.append(")");
            System.out.println(sb.toString());
        }
    }

    /*
    输出：
    ---------------------begin Construstors-----------------
    com.sun.proxy.$Proxy0(java.lang,reflect,InvocationHandler)
    ---------------------begin Construstors-----------------
    */
```

- 要通过Proxy类来动态生成代理类，就必须要传入两个参数

  - 动态生成类的字节码指定类加载器
  - 动态生成的字节码实现了哪些接口

- 动态生成类的字节码,并打印生成类的所有方法

```java
   /**
     * 动态生成类的字节码,并打印动态类的每个方法
     */
    public static void getMethods(){
        //动态生成代理类
        Class clazzProxy1 = Proxy.getProxyClass(Collection.class.getClassLoader(), Collection.class);

        //获取这个代理类的构造方法
        Method[] methods = clazzProxy1.getMethods();

        System.out.println("---------------------begin Construstors-----------------");
        //遍历构造方法
        for (Method method: methods) {
            //获取每个名称
            String name = method.getName();
            StringBuilder sb = new StringBuilder(name);
            sb.append("(");
            //获取每个构造方法的参数类型
            Class[] clazzTypes = method.getParameterTypes();
            for (Class clazzType : clazzTypes) {
                sb.append(clazzType.getName()).append(".");
            }
            if(clazzTypes != null && clazzTypes.length != 0){
                sb.deleteCharAt(sb.length() - 1);
            }
            sb.append(")");
            System.out.println(sb.toString());
        }
    }
```
![输出结果](/styles/images/2019-4/proxymethod.png)

通过打印结果可知，动态代理类 clazzProxy1 的每个方法都有Collection接口的每个方法和Object类的每个方法。

- 完成InvocationHandler对象的内部功能

java.lang.reflect.Proxy 类为我们直接提供创建出代理对象的方式，就是调用Proxy.newProxyInstance方法。就省去了先获取动态类的Class对象，再通过Class对象获取动态类的对象的过程了。

```java
    public static void getMyInstance(){
        //Proxy.newInstance方法直接创建出代理对象
        Collection proxy1 = (Collection) Proxy.newProxyInstance(
                Collection.class.getClassLoader(), 
                new Class[]{Collection.class},
                new InvocationHandler() {
                    //方法外部指定目标
                    List target = new ArrayList<>();
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        //在调用代码之前加入系统功能代码
                        long startTime = System.currentTimeMillis();
                        //睡眠1秒钟
                        Thread.sleep(1000);
                        //目标方法
                        Object retVal = method.invoke(target, args);
                        //在调用代码之后加入系统功能代码
                        long endTime = System.currentTimeMillis();
                        System.out.println( method.getName() + "方法花费了:" + (endTime - startTime) + "毫秒");
                        return retVal;
                    }
                });

        proxy1.add("a");
        proxy1.add("b");
        proxy1.add("c");
        //3
        System.out.println(proxy1.size());
    }


/*
输出：
add方法花费了:1001毫秒
add方法花费了:1001毫秒
add方法花费了:1001毫秒
size方法花费了：1001毫秒
3
*/
```

每次调用代理对象的每个方法，都会调用InvocationHandler的invoke方法。

- 为了在系统运行的时候，可以临时的灵活加入系统功能，实现系统功能灵活解耦。要为InvocationHandler传递两个对象：

  - 目标对象target
  - 系统功能对象

```java
import java.lang.reflect.Method;

/**
 * 实现系统功能接口
 * @author
 *
 */
public interface Advice {

    void beforeMethod(Method method);
    void afterMethod(Method method);
```

```java
import java.lang.reflect.Method;
/**
 * 自定义系统功能的实现类
 * @author tianshuo
 *
 */
public class MyAdvice implements Advice {

    @Override
    public void beforeMethod(Method method) {
        System.out.println("在目标方法之前调用！");
    }

    @Override
    public void afterMethod(Method method) {
        System.out.println("在目标方法之后调用！");
    }

}
```

```java
/**
     * 使用传递参数的方式灵活创建代理对象
     * @param target:目标对象
     * @param advice:系统功能对象
     * @return Proxy Object:代理对象
     */
    public static Object getProxy(final Object target,final Advice advice){

        //Proxy.newInstance方法直接创建出代理对象
        Object proxy3 = Proxy.newProxyInstance(
                target.getClass().getClassLoader(), 
                target.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

                        advice.beforeMethod(method);

                        Object retVal = method.invoke(target, args);

                        advice.afterMethod(method);

                        return retVal;
                    }
                });
        return proxy3;
    }

}
```

```java
/**
     * 调用代理对象
     * @param args
     * @throws Exception
     */
    public static void main(String[] args) throws Exception{
        List target = new ArrayList<>();
        List proxyObject = (List) getProxy(target, new MyAdvice());
        proxyObject.add("abc");
        System.out.println(proxyObject.size());

    }

/*
输出：
在目标方法之前调用！
在目标方法之后调用！
在目标方法之前调用！
在目标方法之后调用！
1
*/
```



## java supertype

> https://stackoverflow.com/questions/15130067/what-is-a-supertype-method


面向对象程序设计中有一个超类型和子类型的概念，在java中这种关系是通过继承实现的，即使用extends关键字:

```java
class A{} // super class
class B extends A {} // sub class
```

所有在super class 中定义的member（字段、方法） 均可以被叫做 supertype

因此在上下文中 如果 class A 有一个方法如下

```java
class A{
    void set()
}
```

set 就是class B的一个 supertype


## java bean

1. 所有属性为private
2. 提供默认构造方法
3. 提供getter和setter
4. 实现serializable接口

## Idea 配置maven

- File-Settings 
    配置maven 路径
    ![](/styles/images/2019-4/ideamaven.png)

## JNDI

JNDI即Java Naming and Directory Interface，翻译成中文就Java命令和目录接口，JNDI提供了很多实现方式，主要有RMI，LDAP，CORBA等。

## 

## java 反射基础
---
### 反射的功能

我们可以在运行时获得程序或程序集中每一个类型的成员和成员的信息

- 在运行时判断任意一个对象所属的类；
- 在运行时构造任意一个类的对象；
- 在运行时判断任意一个类所具有的成员变量和方法（通过反射甚至可以调用private方法）；
- 在运行时调用任意一个对象的方法

### Java的反射主要提供以下类

用于获取不同场景的类的信息,位于java.lang.reflect包中:

- Class类：代表一个类
- Filed类：代表类的成员变量
- Method类：代表类的方法
- Constructor类：代表类的构造方法
- Array类：提供了动态创建数组，以及访问数组的元素的静态方法。该类中的所有方法都是静态方法.

### 执行calc.exe

```java
//直接执行
Runtime.getRuntime().exec("calc.exe");

//反射调用
try{
    Method method = Foo.class.getMethod("bar");
    method.invoke(foo);
}catch(IllegalAccessException | InvocationTargetException e){
}


```

## 设计模式

---

### 工厂模式

引用 https://segmentfault.com/a/1190000015050674

在基类中定义创建对象的一个接口，让子类决定实例化哪个类。工厂方法让一个类的实例化延迟到子类中进行。

- 解耦 ：把对象的创建和使用的过程分开

- 降低代码重复: 如果创建某个对象的过程都很复杂，需要一定的代码量，而且很多地方都要用到，那么就会有很多的重复代码。

- 降低维护成本 ：由于创建过程都由工厂统一管理，所以发生业务逻辑变化，不需要找到所有需要创建对象B的地方去逐个修正，只需要在工厂里修改即可，降低维护成本。


1. **工厂方法模式**

```java
//工厂接口
public interface Factory {
    public Shape getShape();
}


//工厂类
//圆形工厂类
public class CircleFactory implements Factory {

    @Override
    public Shape getShape() {
        // TODO Auto-generated method stub
        return new Circle();
    }

}

public class RectangleFactory implements Factory{

    @Override
    public Shape getShape() {
        // TODO Auto-generated method stub
        return new Rectangle();
    }

}

public class SquareFactory implements Factory{

    @Override
    public Shape getShape() {
        // TODO Auto-generated method stub
        return new Square();
    }

}

//test
public class Test {

    public static void main(String[] args) {
        Factory circlefactory = new CircleFactory();
        Shape circle = circlefactory.getShape();
        circle.draw();
    }

}

/*
输出
Circle
Draw Circle
*/
```

2. **抽象工厂模式**

    在工厂方法模式中，其实我们有一个潜在意识的意识。那就是我们生产的都是同一类产品。抽象工厂模式是工厂方法的仅一步深化，在这个模式中的工厂类不单单可以创建一种产品，而是可以创建一组产品。

    抽象工厂是生产一整套有产品的（至少要生产两个产品)，这些产品必须相互是有关系或有依赖的，而工厂方法中的工厂是生产单一产品的工厂。

```java

public interface Gun {
    public void shooting();
}

public interface Bullet {
    public void load();
}

//AK类
public class AK implements Gun{

    @Override
    public void shooting() {
        System.out.println("shooting with AK");
        
    }

}

//M4A1类
public class M4A1 implements Gun {

    @Override
    public void shooting() {
        System.out.println("shooting with M4A1");

    }

}

//AK子弹类
public class AK_Bullet implements Bullet {

    @Override
    public void load() {
        System.out.println("Load bullets with AK");
    }

}

//M4A1子弹类
public class M4A1_Bullet implements Bullet {

    @Override
    public void load() {
        System.out.println("Load bullets with M4A1");
    }

}

//创建工厂接口
public interface Factory {
    public Gun produceGun();
    public Bullet produceBullet();
}

//创建具体工厂

//生产AK和AK子弹的工厂
public class AK_Factory implements Factory{

    @Override
    public Gun produceGun() {
        return new AK();
    }

    @Override
    public Bullet produceBullet() {
        return new AK_Bullet();
    }

}

//生产M4A1和M4A1子弹的工厂
public class M4A1_Factory implements Factory{

    @Override
    public Gun produceGun() {
        return new M4A1();
    }

    @Override
    public Bullet produceBullet() {
        return new M4A1_Bullet();
    }

}

//测试
public class Test {

    public static void main(String[] args) {  
     Factory factory;
     Gun gun;
     Bullet bullet;
     factory =new AK_Factory();
     bullet=factory.produceBullet();
     bullet.load();
     gun=factory.produceGun();
     gun.shooting(); 
    }

}
/*

输出结果：

Load bullets with AK
shooting with AK
*/

```