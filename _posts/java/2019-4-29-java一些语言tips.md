---
layout: post
title: java一些语言tips
categories: java
tag: java
---

- https://github.com/Snailclimb/JavaGuide  Java学习+面试指南

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