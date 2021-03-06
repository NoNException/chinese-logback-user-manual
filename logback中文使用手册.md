# Logback User Manual

[TOC]





## 引言

**士气的影响力是惊人的。任何一个使是最简单系统的运行,也足以振奋人心。而当一个图形系统能显示图片时，即使那仅仅是一个矩形，这样的热情也会翻倍。类似的情形总是会出现在一个可运行系统发展的各个阶段上。我发现这样的团队可以在四个月内构建出比他们想象中更复杂的系统**

--FREDERICK P. BROOKS, JR., *The Mythical Man-Month*

### logback简述

Logback的设计者Ceki Gülcü(同时也是log4j的创始人)，他的初衷是使其成为流行框架log4j项目的继承者。logback框架基于作者在工业级强度日志系统十几年来的经验设计而成，它比目前所有已经存在的日志系统更加显著的性能提升，更重要的是logback提供了一些其他的日志系统所没有的有用特性。

### 开始的第一步

#### 前期准备

- 为了能够运行本章中的实例，你需要保证的你类路径已经包含了特定的jar包。请到[初始页](https://logback.qos.ch/setup.html)面查看更详细的内容。jar的内容如下:

  - logback-core-1.3.0-alpha4.jar
  - logback-classic-1.3.0-alpha4.jar
  - logback-examples-1.3.0-alpha4.jar
  - slf4j-api-1.8.0-beta1.jar

  其中所有的logback-*.jar包都是logback的模块，而slf4j-api-1.8.0-beta1.jar来自于另外一个项目[SLF4J](http://www.slf4j.org/)

- Logback-classic 模块需要依赖slf4j-api.jar和logback-core.jar才能正常工作

下面让我们开始使用logback的第一个示例

##### 示例1.1：记录日志的基本用法([*(logback-examples/src/main/java/chapters/introduction/HelloWorld1.java)*](https://logback.qos.ch/xref/chapters/introduction/HelloWorld1.html))

```java
package chapters.introduction;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld1 {

  public static void main(String[] args) {

    Logger logger = LoggerFactory.getLogger("chapters.introduction.HelloWorld1");
    logger.debug("Hello world.");

  }
}
```

*HelloWorld1*类被定义在chapters.introduction包下，他的main方法中使用了SELF4J API中定义Logger类和LoggerFactory工厂方法。

分析这个简单的例子我们可以发现main方法中声明了一个打印DEBUG级别内容“Hello world”的日志。具体的过程是：main方法首先调用`LoggerFactory`工厂的静态方法创建了一个*Logger*类的实例*logger*，它名字是“chapters.introduction.HelloWorld1”，接下来调用了logger的debug方法，同时以"Hello world"作为方法入参。

值得注意的是，上面的例子没有引用任何一个logback的类。在大多数情况下，当你需要在你的类中考虑使用日志的时候，你只需要引入SLF4J的类就足够了。因此，如果你的代码使用SLF4J API,你可以完全忽略logback的存在。

你可以使用如下的java 命令来运行上面的示例

```shell
java chapters.introduction.HelloWorld1
```

运行成功之后，你的终端将获得如下的内容

```java
01:01:53.497 [main] DEBUG chapters.introduction.HelloWorld1 - Hello world.
```

根据logback的默认配置策略，当没有默认的配置文件（etc. Logback.xml等），框架会默认添加一个 ConsoleAppender 作为根日志。

Logback可以同过内建的转改系统来报告其状态。StatusManager组件可以汇报logback声明周期内重要的事件，现在，让我们通过调用StatusPrinter类的print()方法来打印logback的内部状态吧

##### 示例1.2：打印日志状态([logback-examples/src/main/java/chapters/introduction/HelloWorld2.java](https://logback.qos.ch/xref/chapters/introduction/HelloWorld2.html))

```java
package chapters.introduction;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import ch.qos.logback.classic.LoggerContext;
import ch.qos.logback.core.util.StatusPrinter;

public class HelloWorld2 {

  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger("chapters.introduction.HelloWorld2");
    logger.debug("Hello world.");

    // print internal state
    LoggerContext lc = (LoggerContext) LoggerFactory.getILoggerFactory();
    StatusPrinter.print(lc);
  }
}	
```

运行Helloworld2将会产生如下的日志输出

```
21:53:15.159 [main] DEBUG chapters.introduction.HelloWorld2 - Hello world.
21:53:15,082 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback-test.xml]
21:53:15,083 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback.groovy]
21:53:15,083 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback.xml]
21:53:15,092 |-INFO in ch.qos.logback.classic.BasicConfigurator@9536ecf - Setting up default configuration.
```

上面的日志表明Logback没有找到 logback-test.xml或者logback.xml配置文件（稍后解释），于是它使用了默认的配置策略：使用ConsoleAppender（Apender类决定了输出日志的方式），Appenders使日志可以通过很多方式输出，包括控制台、文件、系统日志、TCP套接字，JMS甚至更多，使用者甚至可以根据情况定制自己的Appender。

 **注意：**logback会自动打印内部的错误到控制台。

虽然实例1.2相对来说比较简单，然而即使对于一个稍大的应用来说，日志声明模式也不会有什么改变，只是日志的配置过程会有一些不同。如果你想根据自己的需要来定制化logback，接下来章节会给你提供帮助。

在上面的例子里，我们看到logback通过调用静态方法StatusPrinter.print()来打印其内部状态状态。

**注意：**了解logback的内部状态对于诊断logback相关的问题非常有用。

下面的列表说明在你的应用中使用logback的必要步骤：

1. 配置logback环境，方式可繁可简，点击此处查看.
2. 在需要使用日志的类中通过调用org.slf4j.LoggerFactory.getLogger()方法获得Logger实例来记录日志，此方法允许你传入类名或者类本身。
3. 用过调用Logger类的debug()，info()，warn()，error() 方法类记录不同级别的日志，记录的内容将通过配置的appnder进行输出。

#### 构建logback

logback通过[Maven](http://maven.apache.org/)来构建，如果你已经安装Maven，解压Logback发行包，你可以用过mvn install来构建logback项目中所有的模块，Maven会自动下载logback所依赖的所有类库。

logback发行包中包含了所有的源代码，在遵从LGPL或者EPL开源许可下，你可以修改l任意代码甚至构建、发布你自己的版本。

对于logback在IDE（集成开发环境）下的构建，请查看[类路径设置的相关章节](https://logback.qos.ch/setup.html#ide)。

### 第二章：架构

#### Logback系统架构

logback拥有在不同环境下都相对通用的架构。当前Logback拥有三个主要的模块，分别是logback-core，logback-classic和logback-access。

logback-core模块是其他两个基础，logback-classic是logback-core模块的扩展。其中logback-classic相对于log4j有了显著的提升，并且其实现了[SLF4J API](http://www.slf4j.org/),所以你可以很容易地从logback切换到其他日志系统，例如log4j或者JDK1.4中引入的java.util.logging(JUL)。而logback-access模块集成了Servlet容器，可以提供HTTP日志访问功能。相关的文档参阅[logback-access模块文档](https://logback.qos.ch/access.html)

在当前文档中，我们提到的logback模块默认指的是logback-classic模块。

#### Logger,Appenders and Layouts

Logback建立在三个主要类之上：Logger,Appender和layout。这三个组件协同工作可以使开发者根据消息类型和日志级别记录信息，或者组织运行时日志的打印格式和输出方式。

从模块归属结构来讲，Logger类属于logback-classic模块，Appender和Layout属于logback-core模块。作为一个通用模块，logback不包含任何日志记录的概念。

#### Logger上下文 

日志API相对于Systm.out.println语句，前者最重要的要求是能够在禁用一些日志输出的同时又不能影响其他日志的输出。达到这样的要求，需要将所有的日志能够按照开发者的自主选择进行分类。在logback-classic模块中，日志类别是每个日志记录器（logger）的固有属性。每一个logger都会被归属于LoggerContext类，既日志上下文。LoggerContext类将负责每一个日志记录器的生成和管理。

每一个logger都必须被命名，并且他们的名字是**大小写敏感**的其应该遵守下面的层次化规则。

**logger之间的继承关系通过“.”来实现，如果一个logger A的名称加上“.”之后是另一个logger B 名称的前缀，那么A是B的祖先，与此同时，如果当前日志上下文中没有位于A和B继承关系之间的logger，那么A为B之父。**

举例来说，名为“com.foo"的logger为名为“com.foo.Bar”的logger之父。类似的，"java"是“java.util”之父，同时也是"java.util.Vecor"的祖先。大多数的开发者都应该熟悉这样的命名格式。

根日志记录器位于整个日志层级的最顶端，它的特别之处是它是所有日志记录器的共同始祖。与所有日志记录器（logger）一样，它也可以通过名字获取，方式如下：

```java
Logger rootLogger = LoggerFactory.getLogger(org.slf4j.Logger.ROOT_LOGGER_NAME);	
```

所有的其他日志记录器（Logger类）同样也能通过org.slf4j.LoggerFactory的静态方法getLogger获取。这个方法接收日志记录器的名字作为入参。下面列举Logger接口的基本方法：

```java
package org.slf4j; 
public interface Logger {

  // 打印方法 
  public void trace(String message);
  public void debug(String message);
  public void info(String message); 
  public void warn(String message); 
  public void error(String message); 
}
```

Effective Level aka Level Inheritance

#### 日志级别继承

日志记录器可以被指定级别。上面的级别（`TRACE`，`DEBUG`，`INFO`，`ERROR`)被定义在`cn.qos.logback.classic.Level`类中，在logback中Level被定义为final类型的且不能继承，而`Marker`对象则提供了更加灵活的方法。

如果一个日志记录器没有被指定级别，那么他会从他最近的一个祖先的日志记录器中继承日志级别，下面是更加正式的表示：

**对于一个给定的日志记录器L的有效日志级别，等于从L本身的级别(直到根日志记录器（root logger）开始的第一个非空（non-null）的日志级别。**

为了保证所有的日志记录器最终都能继承到一个日志级别，根日志记录器总是会被关联一个日志级别，其默认的日志等级是DEBUG。

下面四个例子说明了在日志记录器的级别继承规则下，其有效级别的例子

*示例1*

| 日志记录器 | 关联日志级别 | 有效日志级别 |
| ---------- | ------------ | ------------ |
| root       | DEBUG        | DEBUG        |
| X          | none         | DEBUG        |
| X.Y        | none         | DEBUG        |
| X.Y.Z      | none         | DEBUG        |

示例1中，只有根日志记录器被关联的DEBUG级别，所以DEBUG的日志级别被所有的其他日志记录器所继承（X,X.Y,X.Y.Z）

*示例2*

| 日志记录器 | 关联日志级别 | 有效日志级别 |
| ---------- | ------------ | ------------ |
| root       | ERROR        | ERROR        |
| X          | INFO         | INFO         |
| X.Y        | DEBUG        | DEBUG        |
| X.Y.Z      | WARN         | WARN         |

示例2中，所有的日志记录器都被关联了日志级别，日志的继承规则没有起作用

*示例3*

| 日志记录器 | 关联日志级别 | 有效日志级别 |
| ---------- | ------------ | ------------ |
| root       | DEBUG        | DEBUG        |
| X          | INFO         | INFO         |
| X.Y        | none         | INFO         |
| X.Y.Z      | ERROR        | ERROR        |

示例3中，`root looger`、`X`、`X.Y.Z`分别被关联了`DEBUG`、`INFO`和`ERROE`级别，而`X.Y`则继承了他的父辈`X`的级别

*示例4*

| 日志记录器 | 关联日志级别 | 有效日志级别 |
| ---------- | ------------ | ------------ |
| root       | DEBUG        | DEBUG        |
| X          | INFO         | INFO         |
| X.Y        | none         | INFO         |
| X.Y.Z      | none         | INFO         |

示例4中，`root logger` 和 `X` 分别被关联了` DEBUG`和 `INFO`级别，而没有关联日志级别的`X.Y` 和` X.Y.Z`则从他们的最近的，已经关联了日志级别`INFO`的父辈`X`中继承到了对应的日志级别

#### 级别的打印策略和选择规则

根据定义，日志记录器的打印方法决定了打印请求的级别。例如，L代表了一个日志记录器，那么L.info(...)方法则表示一个INFO级别的**日志记录请求**。

一个日志记录请求的只有当其本身的级别等于或者高于日志记录器的级别时才是有效(enabled)的，否则称为无效的(disabled)。如同之前描述的，一个没有关联级别的日志记录器会从其祖辈中继承一个级别。下面是总结的表述

#### **基本选择规则：**

#### **一个有效级别为q的日志记录器发起了级别为p的日志记录请求，当且只当p>=q时，日志请求才是有效的**

这个规则是logback的核心，级别之间则按照如下排序：TRACE<DEBUG<INFO<WARN<ERROR

下面的图标对于选择规则的的表示更加直观，表格的行头说明了日志请求的级别p，列头则代表的日志记录器的有效级别q，而行列交叉处的布尔值则代表了基本选择规则下的日志请求的有效值。

| 日志请求级别p | 日志记录器的有效级别 q |           |          |          |           |         |
| :------------ | :--------------------- | :-------- | -------- | :------- | --------- | ------- |
| \             | **TRACE**              | **DEBUG** | **INFO** | **WARN** | **ERROR** | **OFF** |
| **TRACE**     | *YES*                  | NO        | NO       | NO       | NO        | NO      |
| **DEBUG**     | *YES*                  | *YES*     | NO       | NO       | NO        | NO      |
| **INFO**      | *YES*                  | *YES*     | *YES*    | NO       | NO        | NO      |
| **WARN**      | *YES*                  | *YES*     | *YES*    | *YES*    | NO        | NO      |
| **ERROR**     | *YES*                  | *YES*     | *YES*    | *YES*    | *YES*     | NO      |

下面的也可以说明基本选择规则

```java
import ch.qos.logback.classic.Level;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
....

// 获取名为 "com.foo"的日志记录器. 为了能设置日志记录器等级，我们将其转换为
// ch.qos.logback.classic.Logger 类
ch.qos.logback.classic.Logger logger = 
        (ch.qos.logback.classic.Logger) LoggerFactory.getLogger("com.foo");
// 设置日志记录器等级为INFO
logger.setLevel(Level. INFO);

Logger barlogger = LoggerFactory.getLogger("com.foo.Bar");

// 此日志记录请求有效，因为WARN(日志记录请求等级)>= INFO(日志记录器"com.foo"日志等级)
logger.warn("Low fuel level.");

// 此日志记录请求无效, 因为DEBUG(日志记录请求等级) < INFO(日志记录器"com.foo"日志等级)
logger.debug("Starting search for nearest gas station.");

// 日志记录器“com.foo.Bar"继承了"com.foo"的日志等级"INFO"，
// 因此其INFO级别的日志请求是有效的
// 因为 INFO(日志记录请求等级) >= INFO(日志记录器"com.foo.Bar"日志等级)
barlogger.info("Located nearest gas station.");

// 此日志记录请求无效, 因为DEBUG(日志记录请求等级) < INFO(日志记录器"com.foo.Bar"日志等级) 
barlogger.debug("Exiting gas station search");
```

#### 获取日志记录器

当使用相同的名称从`LoggerFactory.getLogger`方法中获取日志记录器时，总是会返回同一个日志记录器对象的引用。对于如下的例子

```java
Logger x = LoggerFactory.getLogger("wombat");
Logger y = LoggerFactory.getLogger("wombat");
```

x 和 y 指向的都是同一个日志记录器对象。

因此，对于一个已经配置好的日志记录器来说，即使不在代码中传递该对象的引用，你也能获取到同一个对象。与生物当中的父辈先于子辈出现不同，logback中的日志记录器会被无序的实例化，即使父辈晚于子辈被实例化，其也能正确的关联到子辈。

logback环境的配置通常是在应用初始化的过程中完成。建议优先选用读取配置文件的方式来，此方式将在稍后讨论。

Logback 简化了日志记录器的命名，方法是在每个类里初始化日志记录器，以类的全限定名作为其命名。这种定义的方法即有用又直观。由于日志输出时会包含日志记录器的名字，此命名方式可以很容易地确定记录消息来源。Logback 不限制 logger 名，作为一个开发者，你也可以随意命名日志记录器。 

然而，以所在类来为logger命名似乎是目前已知最好的命名方式。

#### 日志投递器与日志排版

根据日志记录器的等级决定日志请求的消失与否的功能仅仅是logback的冰山一角。logback还可以将日志请求输出到不同的目的地。在loggback中输出目的地被称为投递器（appender)。当前logback支持的输出方式有控制台、文件、远程服务器、MySQL、PstgreSQL、Oracle或者其他的数据库，JMS(?)和远程UNIX系统日志线程等。

一个日志记录器可以同时绑定多个输出方向。

日志记录器可以通过`addAppender`方法添加日志投递器。一个有效的日志请求会被投递到日志记录器，以及其所有父辈们绑定的所有投递器上。换句话说，日志投递器附着于日志记录器的继承关系。例如，如果一个根日志记录器添加了控制台投递器，因为日志记录器的继承关系，所有有效的日志请求都将至少输出到控制台、与此同时，如果日志记录器L额外附加了一个文件投递器，那么L以及L所有的子类日志记录器接收到的有效日志请求都将同时输出到控制台和文件当中。当然这种日志记录器上默认的添加行为是可以修改的，你可以设置一个日志记录器的可累加标志位(additivity flag)为`false`来取消来自父辈的日志投递器的累积情况。

正式日志投递器累加原则总结如下：

**日志记录器L所能输出的所有日志记录将同时输出到L以及L的父辈所绑定的日志投递器中，这就是所说的“日志投递器累加“。**

**然而，如果有一个L的祖先P，设置可累加标志位为false，那么L所输出的日志记录，只能输出继承关系在包括L到P之间的日志记录器绑定的日志投递器上**

**所有的日志记录器的可添加标志位都默认设置为true**

下面的表格里整理了一个直观的例子

| 日志记录器      | 绑定的日志记录器 | 可添加标志位 | 输出目标           | 备注                                                         |
| --------------- | ---------------- | ------------ | ------------------ | ------------------------------------------------------------ |
| root            | A1               | 不可更改     | A1                 | 因为根日志记录器是所有日志记录器的祖先，所以不可禁用其的可累加标志位 |
| x               | A-x1,A-x2        | true         | A1,A-x1,A-x2       | 输出目标包括X从root中获的A1，也包括自己绑定的A-x1以及A-x2    |
| x.y             | none             | true         | A1,A-x1,A-x2       | 继承x全部的输出目标                                          |
| x.y.z           | A-xyz            | true         | A1,A-x1,A-x2,A-xyz | 继承x全部的输出目标，以及自己绑定的A-xyz                     |
| security        | A-sec            | false        | A-sec              | 设置累加标志位为false，取消了自己从root累加的日志投递器A1，所以security只会输出到A-sec上 |
| security.access | none             | true         | A-sec              | 只会集成security上绑定的日志记录器A-sec                      |

不仅仅是这样，用户会希望在定制化输出目标的同时也可以定制日志的输出格式。这部分工作可以由日志投递器的日志排版器（*layout*）完成，日志投递器只负责将日志记录的输出，而日志排版器器则负责根据用户的期望格式化日志内容。在logback发行版中，`PatternLayout`可以让用户像C语言中的`printf`方法类似的转换模式来格式化日志输出。

举例来说，使用”%-4relative [%thread] %-5level %logger{32} - %msg%n“转换模式的日志排版器会输出下面的语句
```java
176  [main] DEBUG manual.architecture.HelloWorld2 - Hello world.
```
第一个字段表示从程序启动到日志输出经过的毫秒数，第二个字段表示发出日志请求的线程名称，第三个字段表示了日志请求的等级，第四个字段表示了日志记录器的名称，”-“之后则是日志请求中的打印信息

#### 参数化日志记录

因为logback-classic中默认的日志记录器都实现了[`SLF4J.Logger`](http://www.slf4j.org/api/org/slf4j/Logger.html)接口,所有特定的打印方法可以接受不止一个参数。打印方法的这种形式的变化旨在提高新能的同时最小化其对代码可读性的影响。

有一些日志的打印方式如下

```java
ogger.debug("Entry number: " + i + " is " + String.valueOf(entry[i]));
```

上面的语句在把整数 i 和 entry[i]都转换为字符串时，以及连接多个字符串时，会有性能消耗，且不管消息最终会不会被记录。

在日志输出语句之前，添加日志记录器级别测试，是一种可能免除上述性能消耗的操作。举例如下

```java
if(logger.isDebugEnabled()) { 
  logger.debug("Entry number: " + i + " is " + String.valueOf(entry[i]));
}
```

当日志记录器logger的输出级别高于DEBUG时，这确实是有效的。但是，如果logger的输出级别是DEBUG级别时，性能消耗就不可避免了，此时性能消耗的地方还变成了两个：`debugEnable`和`debug`方法各一个。在实际情况下，前者消耗的时间是微不足道的，其时间消耗不足后者的1%

#### 更好的选择

logback提供更好的选择。假设`entry`是一个对象，你可以按照下面的方式来写

```java
Object entry = new SomeObject();
logger.debug("The entry is {}.", entry);
```

只有当日志请求时有效的时候，日志记录器才会将`{}`替换成entry的字符串值。换句话说，当日志请求不可用的时候，这句话不会因为参数构造而产的消耗。

下面的两个语句会产生两个相同的输出结果，然而当日志请求不可用的情况下，后者的表现至少比前者好30%

```java
logger.debug("The new entry is " + entry + ".");
logger.debug("The new entry is {}.", entry);
```

两个入参的方式也是可行的，比如说你可以这么写

```java
logger.debug("The new entry is {}. It replaces {}.", entry,oldEntry);
```

如果有三个或者更多的输入参数需要被记录，你可以使用`Object[]`的形式，如下

```java
Object[] paramArray = {newVal, below, above};
logger.debug("Value {} was inserted between {} and {}.", paramArray);
```

#### 内部一览

在我们介绍完重要的logback组件之后，我们现在来讲解一下当用户调用日志记录器的打印方法时，日志框架的处理步骤。我们就以名为"com.wombat"日志记录器调用`info()`方法举例。

1. ##### 获得责任链的过滤结果

   如果存在，`TurboFilter`责任链会被滴啊用。Turbo 过滤器会设置一个全局级别的阈值，或者根据日志记录器上绑定的信息例如`Marker`,`Level`,`Logger`,`message`,`Throwable`的来筛选出特定的日志请求。如果责任链返回的结果是`FilterReply.DENY`，那么这个日志请求就会被抛弃，如果返回`FilterReply.NEUTRAL`则进行第二步，同时，如果返回的是`FilterReply.ACCEPT`，我们跳过第二步，直接进入第三步。

2. ##### 应用基准选择规则

   在这一步，logback比较日志请求对于日志记录器的有效用。如果日志请求是无效的，那么logback框架就会抛弃日志请求，否则，进入下一步。

3. ##### 创建`LoggingEvent`对象

   如果日志请求都通过了之前的所有过滤器，logback会创建一个包含了所有与日志请求相关参数的`ch.qos.logback.classic.LoggingEvent`对象，比如发出请日志请求的日志记录器，日志请求等级，需要打印的日志本身，也可能包括随着日志传递的异常信息，当前时间，当前线程，关联的类信息，`MDC`等等，需要注意的是，这些信息都是懒加载的，即只有当你需要的时候才去加载。`MDC`通常是用来给日志请求添加额外的上下文信息的，相关的知识将在下面的章节讨论

4. ##### 调用日志添加器

   创建完`LoggingEvent`之后，logback将会调用所有可用的日志添加器的`doAppend()`方法，既从日志上文中继承下来的日志添加器。

   所有logback发行版本中的日志添加器都继承了抽象类`AppenderBase`的`doAppend()`方法，`AppenderBase`类的该方法添加了同步块来保证线程安全

   `doAppend()`方法也会调用绑定到日志添加器上的（如果存在的话）自定义的过滤器。自定义过滤器可以动态地绑定到日志添加器上，之后的章节将会讨论

5. ##### 格式化输出

   格式化日志是被调用的日志添加器的职责。然而，一些（不是全部）会把这部分功能代理给排版器。日志排版器格式化`LoggingEvent`对象，并且返回一个字符串。注意一些特殊的日志添加器，比如说`socketAppender`会将`LoggingEvent`序列化，总的来说他们没有也不需要一个排版器。

6. ###### 发送日志事件`LoggingEvent`

   当日志事件被格式化之后，就会被添加器发送到各自的目的地。

下面的UML图，解释了上述的一切的工作过程，你可以点击查看大图

![underTheHoodSequence2_small](/Users/zongzi/Desktop/underTheHoodSequence2_small.gif)

#### 性能表现

日志记录功经常被拿出来讨论的是它的计算消耗。这么做是无可厚非的，因为即使是中等规模应用也会产生成千上万的日志记录。我们工作的大部分精力都在测量和提升logback的性能表现。除此之外，用户自身也要注意下面几个情况下，logback的性能表现。

##### 1. 当完全关闭日志记录时

你可以通过设置根日志记录器的等级为`Level.OFF`来完全关闭日志输出。如果你这么做了，日志模块的消耗只剩下了一个方法调用加上一个整型比较。在一个3.2Ghz的机器上，这将只需要消耗20纳秒。

然而，任何包含了隐式的参数构造的方法调用，如需下面的写法

```java
x.debug("Entry number: " + i + "is " + entry[i]);
```

都将产生参数构造的性能消耗，即无论最后日志是否输出，性能都将消耗在i和entry[i]转换成字符串以及构建出中间的字符串上。

这一过程也有可能因为参数的大小而产生大量的性能消耗。为了消除这样的影响，你应该使用`SLF4J`推荐的方式：

```java
x.debug("Entry number: {} is {}", i, entry[i]);
```

这样的写法不会产生参数构造的性能消耗，与之前的写法相比，它还有大量的性能提升，因为他只有在被发送到相关的日志添加器的时候才会被格式化。而这格式化的过程本身也被极大地优化了。

然而在一个非常紧凑的循环中放置上面的语句，比如某些经常地调用的代码里，是一个双输的提议，非常容易导致性能崩溃。即使已经将日志关闭了，在紧致的循环中记录日志也会降低应用的速度，另外，如果你打开了日志，也会产生大量无用的日志

##### 2.记录日志内容对性能的影响

在logback中没有必要准从日志记录器的继承关系。一个日志记录器被创建时就知道自己的有效日志等级（在考虑了等级继承的情况下），一个父辈日志记录器的等级改变，他的又有子辈日志记录器都会注意到这个变化。因此日志记录器可以不用考虑其祖先的情况下，根据其自身的有效等级就可以决定是否接受一个日志请求

#### 3.实际记录日志时（格式化及输出到目标设备）

这部分指的是格式化日志内容并将其输出到其目的地时的消耗。再次强调，我们已经尽力使排版器以及添加器的性能达到最优。一般来说将日志输出到本地机器的文件中的消耗基本在9到12微妙之间。而当输出日志到远程中的数据库也只会提高几微妙。

尽管功能丰富，logback 最首要设计目标可靠性和执行速度。logback 的一些组件已经被多次重写来提高性能。

### 第三章 配置

本章的内容将按照一系列配置文件的形式来展现logback的配置方式。Joran，作为logback的配置框架将在之后的章节讨论

#### Logback中的配置

在应用代码中插入日志需要经过谨慎的思考，应用中大约4%的代码与日志相关。因此即使是一个中等规模的应用也需要考虑插入上千条的日志语句。基于这样的考虑，需要引用工具来管理这些日志语句。

Logback可以通过代码和类似于XML或者Groovy的脚本来管理配置。与此同时，log4j用户可以通过我们的[PropertiesTranslator](http://logback.qos.ch/translator/)应用来实现log4j.properties文件到logback.xml文件的转换。

让我们以logback的配置初始化步骤展开讨论：

- Logback尝试在类路径（classpath）下查找*logback.xml*文件
- 如果没有找这个文件，logback转为在类路径下查找*logback.grooy*文件
- 如果还是没有找到，则继续在相同目录下查找*logback.xml*文件
- 如果还是没有，则通过java 1.6中的[服务加载](http://docs.oracle.com/javase/6/docs/api/java/util/ServiceLoader.html)形式，即通过在类路径下的*META-INF\services\ch.qos.logback.classic.spi.Configurator*文件中查找其实现类的全类名类加载`Configuration`的实现类.
- 如果上面的尝试都没有成功，logback会使用`BasicConfigurator`来完成配置，当然其也会造成所有的日志输出都指向了控制台









 



























