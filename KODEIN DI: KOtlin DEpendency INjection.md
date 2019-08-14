[原英文文档地址](https://kodein.org/Kodein-DI/index.html?latest/core)

# KODEIN DI: KOtlin DEpendency INjection

* [一.introduction](#1)
   * [Description](#1.1)
   * [Example](#1.2)

* [二.Platform compatibility & Genericity](#2)
   
* [三.Install](#3)
   * [JVM](#3.1)
     * [With Maven](#3.1.1)
     * [With Gradle](#3.1.2)
   * [JavaScript (Gradle)](#3.2)
   * [Native (Gradle)](#3.3)

* [四.Bindings: Declaring dependencies](#4)
   * [Tagged bindings](#4.1)
   * [Provider binding](#4.2)
   * [Singleton binding](#4.3)
   * [Factory binding](#4.4)
   * [Multiton binding](#4.5)
   * [Referenced singleton or multiton binding](#4.6)
   * [Instance binding](#4.7)
   * [Direct binding](#4.8)
   * [Subtypes bindings](#4.9)
   * [Transitive dependencies](#4.10)
   * [Being responsible for its own retrieval](#4.11)
   
   
* [五.Bindings separation](#5)
   * [Modules](#5.1)
   * [Extension (composition)](#5.2)
   * [Overriding](#5.3)
   * [Overridden access from parent](#5.4)


* [六.Dependency injection & retrieval](#6)
   * [Retrieval rules](#6.1)
   * [Injection & Retrieval](#6.2)
   * [Injection](#6.3)
   * [Retrieval: the Kodein container](#6.4)
   * [Retrieval: being KodeinAware](#6.5)
   * [Retrieval: Direct](#6.6)
   * [Error messages](#6.7)
   
* [七.Using the environment](#7)
   * [Context](#7.1)
   * [Scope](#7.2)
   * [Scope creation](#7.3)

* [八.Multi-binding](#8)
   * [In a Set](#8.1)
   * [In a map](#8.2)
   * [Print bindings](#8.3)
   * [Recursive dependency loop](#8.4)

* [九.OnReady callbacks](#9)

* [十.External Source](#10)

* [十一.Erased version pitfalls](#11)
   * [The type erasure problem](#11.1)
   * [Using generic and erased function forms](#11.2)
   * [Erased parameterized generic types](#11.3)

* [十二.Bind the same type to different factories](#12)

* [十三.Hack the container!](#13)
   * [Tag vs context vs argument](#13.1)
   * [Explore bindings](#13.2)

* [十四.Community](#14)
   * [Contribute](#14.1)
   * [Let’s talk!](#14.2)



###  <h2 id="1">一.introduction</h2>
#### <h2 id="1.1">1.1. Description</h2>
Kodein是一个非常有用的依赖注入/检索容器，它非常易于使用和配置。
> Kodein允许你:
* 延迟初始化你的依赖项在被需要的时候
* 不用关心依赖项初始化的顺序
* 轻松绑定类或接口的 instance，provider或者factory
* 轻松调试依赖项绑定和递归
> Kodein是一个很好的选择因为:
* 它轻量，快速并且经过优化(广泛使用inline关键字)
* 它提供了一个非常简单并且可读的声明性DSL风格
* 它不受类型擦除的影响 (对比 Java)
* 它与Android完美集成
* 它提出了kotlin非常惯用风格的API.
* 它可以被用在java平台


#### <h2 id="1.2">1.2. Example</h2>
Kodein 很容易绑定一种类型
> 绑定示例:
```
val kodein = Kodein {
    bind<Dice>() with provider { RandomDice(0, 5) }
    bind<DataSource>() with singleton { SqliteDS.open("path/to/file") }
}
```
绑定一旦声明，Kodein允许您注入或检索类的依赖项。如果您希望您的类不知道依赖检索，那么您可以在构造时注入依赖项：
> 通过构造使用Kodein的依赖注入:
```
class Controller(private val ds: DataSource) {
    /*...*/
}
val controller by kodein.newInstance { Controller(instance()) }
```
如果您希望您的类本身处理它的依赖项，那么您可以轻松地检索它们：
> 通过构造使用Kodein的依赖注入:
```
class Controller(override val kodein: Kodein): KodeinAware {
    private val ds: DataSource by instance()
}
```

###  <h2 id="2">二.Platform compatibility & Genericity</h2>

Kodein与Kotlin语言编译的所有平台兼容:JVM兼容（Android),Javascript和所有Kotlin / Native目标。
在JVM兼容（Android）上,您需要选择使用已擦除版本或通用版本.
在Javascript和Native目标上，只有已擦除的版本可用
区别很简单:通用版本对比擦除版本在IS时不受类型擦除的影响
当然，它有点复杂！
为了能够规避JVM字节码固有的类型擦除，通用版本使用了大量使用反射性的trix。因为擦除版本不使用该trix，所以在Kodein中处理泛型类型要复杂得多

|   |  类型擦除 |优化|非通用绑定|通用绑定|
|  ----  | ----  | ----  | ----  | ----  |
|generic  |免疫|no|容易|容易|
|erased  |影响|yes|容易|复杂|


>	#性能。但是，作者的拙见如下：
>
> * 在性能，可读性，安全性和可调试性之间存在平衡。
>
> * 优化在关键路径中非常重要,而不是所有地方
>
> * Kodein已经优化的非常好
>
> * 在绝大多数情况下，使用擦除版本将导致您的应用程序没有显着的性能变化，因为IoC只发生一次并且不是性能缺陷！
>
> 因此，在使用已擦除版本之前，请确保适合您的用例;）。 查看您的代码！

在JVM上，如果符合以下条件，您可能更喜欢擦除版本:

  * 您确信您没有绑定/注入/检索泛型类型，并且您确定没有使用第三方库

  * 您没有使用set绑定。

如果您对代码进行概要分析并发现注入是一个性能缺陷，则可能是实例化：您在关键路径中创建了太多对象。在关键路径中重用对象将增强依赖注入/检索和GC中的性能

如果您使用的是擦除版本，可以选择JVM，也可以默认使用JS＆Native，您应该通读擦除版本的缺陷。


###  <h2 id="3">三.Install</h2>
#### <h2 id="3.1">3.1. JVM</h2>
##### <h2 id="3.1.1">3.1.1 With Maven</h2>
Add the JCenter repository:
```
<repositories>
    <repository>
      <id>jcenter</id>
      <url>https://jcenter.bintray.com</url>
    </repository>
</repositories>
```
Then add the dependency:
```
<dependencies>
    <dependency>
        <groupId>org.kodein.di</groupId>
        <artifactId>kodein-di-generic-jvm</artifactId>
        <version>6.3.3</version>
    </dependency>
</dependencies>
```
> 使用 kodein-generic-jvm 或者 kodein-erased-jvm.
##### <h2 id="3.1.2">3.1.2 With Gradle</h2>
在项目层级的build.gradle文件中添加JCenter库:

```
buildscript {
    repositories {
        jcenter()
    }
}
```

之后在项目app层级的build.gradle文件中添加依赖库:

```
dependencies {
    implementation 'org.kodein.di:kodein-di-generic-jvm:6.3.3'
}
```
> 使用 kodein-generic-jvm 或者 kodein-erased-jvm.
#### <h2 id="3.2">3.2. JavaScript (Gradle)</h2>
因为Kodein for JavaScript被编译为UMD模块，所以可以导入它：
* 在浏览器里:
  * 作为AMD模块（例如使用RequireJS） (请参阅演示项目中的index.html).
  * 直接在带有<script>标签的HTML页面中使用 (请参阅演示项目中的index2.html).
* 在NodeJS中，作为常规CJS模块

Add the JCenter repository:
```
buildscript {
    repositories {
        jcenter()
    }
}
```
Then add the dependency:
```
dependencies {
    compile 'org.kodein.di:kodein-di-erased-js:6.3.3'
}
```

#### <h2 id="3.3">3.3. Native (Gradle)</h2>
>Kodein支持以下目标:<br>
> androidArm32, androidArm64, iosArm32, iosArm64, iosX64, linuxArm32Hfp, linuxMips32, linuxMipsel32, linuxX64, macosX64, 
> mingwX64

Kodein-DI 使用新的gradle原生依赖模型。因为
Kodein-DI uses the new gradle native dependency model. 因为该模型在gradle中是实验性的，所以它与下一版本的Gradle不向前兼容。
Add the JCenter repository:
```
buildscript {
    repositories {
        jcenter()
    }
}
```
Then add the dependency:
```
kotlin {
    sourceSets {
        commonMain {
            dependencies {
                implementation "org.kodein.di:kodein-di-erased:6.3.3"
            }
        }
    }
}
```
###  <h2 id="4">四.Bindings: Declaring dependencies</h2>
示例: Kodein容器的初始化
```
val kodein = Kodein {
	/* Bindings */
}
```
Bindings are declared inside a Kodein initialization block.
>	If you are using kodein-generic-jvm, Kodein not subject to type erasure (e.g. You can bind both a List<Int> and a List<String>).
> This is NOT the case when using kodein-erased-jvm, kodein-erased-js or kodein-erased-native. With the erased version by default, binding List<Int> and List<String> actually means binding List<*> twice.

A binding always starts with bind<TYPE>() with. 
  
There are different ways to declare bindings:

#### <h2 id="4.1">4.1. Tagged bindings</h2>

All bindings can be tagged to allow you to bind different instances of the same type.
Example: different Dice bindings
```
val kodein = Kodein {
    bind<Dice>() with ... 
    bind<Dice>(tag = "DnD10") with ... 
    bind<Dice>(tag = "DnD20") with ... 
}
```
1. Default binding (with no tag)
2. Bindings with tags ("DnD10" and "DnD20")

> The tag is of type Any, it does not have to be a String.
> Whether at define, at injection or at retrieval, tag should always be passed as a named argument.
> Tag objects must support equality & hashcode comparison. It is therefore recommended to either use primitives (Strings, Ints, etc.) or data classes.



#### <h2 id="4.2">4.2. Provider binding</h2>
This binds a type to a provider function, which is a function that takes no arguments and returns an object of the bound type (eg. () → T).
The provided function will be called each time you need an instance of the bound type.

Example: creates a new 6 sided Dice entry each time you need one

```
val kodein = Kodein {
    bind<Dice>() with provider { RandomDice(6) }
}
```

#### <h2 id="4.3">4.3. Singleton binding</h2>
This binds a type to an instance of this type that will lazily be created at first use via a singleton function, which is a function that takes no arguments and returns an object of the bound type (eg. () → T).
Therefore, the provided function will be called only once: the first time an instance is needed.

Example: creates a DataSource singleton that will be initialized on first access
```
val kodein = Kodein {
    bind<DataSource>() with singleton { SqliteDS.open("path/to/file") }
}
```



#### <h2 id="4.4">4.4. Non-synced singleton</h2>

#### <h2 id="4.4">4.4. Eager singleton</h2>

#### <h2 id="4.4">4.4. Factory binding</h2>
#### <h2 id="4.5">4.5. Multiton binding</h2>
#### <h2 id="4.6">4.6. Referenced singleton or multiton binding</h2>
#### <h2 id="4.7">4.7. Instance binding</h2>
#### <h2 id="4.8">4.8. Direct binding</h2>
#### <h2 id="4.9">4.9. Subtypes bindings</h2>
#### <h2 id="4.10">4.10. Transitive dependencies</h2>
#### <h2 id="4.11">4.11. Being responsible for its own retrieval</h2>



###  <h2 id="5">五.Bindings separation</h2>
#### <h2 id="5.1">5.1. Modules</h2>
#### <h2 id="5.2">5.2. Extension (composition) (Gradle)</h2>
#### <h2 id="5.3">5.3. Overriding</h2>
#### <h2 id="5.4">5.4. Overridden access from parent</h2>


###  <h2 id="6">六.Dependency injection & retrieval</h2>
#### <h2 id="6.1">6.1. Retrieval rules</h2>
#### <h2 id="6.2">6.2. Injection & Retrieval</h2>
#### <h2 id="6.3">6.3. Injection</h2>
#### <h2 id="6.4">6.4. Retrieval: the Kodein container</h2>
#### <h2 id="6.5">6.5. Retrieval: being KodeinAware</h2>
#### <h2 id="6.6">6.6. Retrieval: Direct</h2>
#### <h2 id="6.7">6.7. Error messages</h2>

###  <h2 id="7">七.Using the environment</h2>
#### <h2 id="7.1">7.1. Context</h2>
#### <h2 id="7.2">7.2. Scope</h2>
#### <h2 id="7.3">7.3. Scope creation</h2>

###  <h2 id="8">八.Multi-binding</h2>
#### <h2 id="8.1">8.1. In a Set</h2>
#### <h2 id="8.2">8.2. In a map</h2>
#### <h2 id="8.3">8.3. Print bindings</h2>
#### <h2 id="8.4">8.4. Recursive dependency loop</h2>

###  <h2 id="9">九.OnReady callbacks</h2>

###  <h2 id="10">10.External Source</h2>

###  <h2 id="11">11.Erased version pitfalls</h2>
#### <h2 id="11.1">11.1. The type erasure problem</h2>
#### <h2 id="11.2">11.2. Using generic and erased function forms</h2>
#### <h2 id="11.3">11.3. Erased parameterized generic typess</h2>

###  <h2 id="12">12.Bind the same type to different factories</h2>

###  <h2 id="13">13.Hack the container</h2>
#### <h2 id="13.1">13.1. Tag vs context vs argument</h2>
#### <h2 id="13.2">13.2. Explore bindings</h2>

###  <h2 id="14">14.Community</h2>
#### <h2 id="14.1">14.1. Contribute</h2>
#### <h2 id="14.2">14.2. Let’s talk!</h2>


正在准备中...
