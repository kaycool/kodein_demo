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
     * [Non-synced singleton](#4.3.1)
     * [Eager singleton](#4.3.2)
   * [Factory binding](#4.4)
      * [Multi-arguments factories](#4.4.1)
   * [Multiton binding](#4.5)
      * [non-synced multiton](#4.5.1)
   * [Referenced singleton or multiton binding](#4.6)
   * [Instance binding](#4.7)
   * [Constant binding](#4.8)
   * [Direct binding](#4.9)
   * [Subtypes bindings](#4.10)
   * [Transitive dependencies](#4.11)
   * [Being responsible for its own retrieval](#4.12)
   
   
* [五.Bindings separation](#5)
   * [Modules](#5.1)
      * [Definition](#5.1.1)
      * [Name uniqueness](#5.1.2)
      * [Import once](#5.1.3)
   * [Extension (composition)](#5.2)
   * [Overriding](#5.3)
   * [Overridden access from parent](#5.4)


* [六.Dependency injection & retrieval](#6)
   * [Retrieval rules](#6.1)
   * [Injection & Retrieval](#6.2)
      * [Base methods](#6.2.1)
   * [Injection](#6.3)
      * [Simple case](#6.3.1)
      * [Multi-arguments factories](#6.3.2)
      * [Currying factories](#6.3.3)
      * [Defining context](#6.3.4)
   * [Retrieval: the Kodein container](#6.4)
      * [everything is lazy by default!](#6.4.1)
      * [Kodein methods](#6.4.2)
      * [Constants](#6.4.3)
      * [Named bindings](#6.4.4)
      * [Multi-arguments factories](#6.4.5)
      	* [Factory retrieval](#6.4.5.1)
      * [Currying factories](#6.4.6)
      * [Defining context](#6.4.7)
      * [Using a Trigger](#6.4.8)
      * [Lazy access](#6.4.9)
      * [Late init](#6.4.10)
      * [All matches](#6.4.11)
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
在Kodein初始化块中声明绑定
> 如果你是用kodein-generic-jvm，Kodein不受类型擦除的影响(例如 您可以绑定List <Int>和List <String>).
> 当使用kodein-erased-jvm，kodein-erased-js或kodein-erasesed-native时，情况并非如此。 默认情况下，使用已擦除的版本，绑定List <Int>和List <String>实际上意味着绑定List <*>两次

一个板顶通常以’bind<具体类型>() with‘开始

声明绑定有不同的方法:

#### <h2 id="4.1">4.1. Tagged bindings</h2>

可以标记所有绑定以允许您绑定相同类型的不同实例。
示例：不同的Dice对象绑定
```
val kodein = Kodein {
    bind<Dice>() with ... 
    bind<Dice>(tag = "DnD10") with ... 
    bind<Dice>(tag = "DnD20") with ... 
}
```
1. 默认的Dice对象绑定(无标记)
2. 通过标记绑定("DnD10" 和 "DnD20")

> 标签可以是任意类型，它不必一定要是String.
> 无论是在定义，注入还是检索时，标记都应始终作为命名参数传递。
> 标签对象必须支持 相等&哈希值 比较。因此，建议使用基本类型(Strings,Ints,etc.)或者 data 类

#### <h2 id="4.2">4.2. Provider binding</h2>
这会将类型绑定到 provider 函数上，该函数不带参数并且返回绑定类型的对象(例如。（）→T)。
provided 函数会在你每次需要这个绑定类型对象的时候都会被调用。

示例: 每次需要时创建一个新的6面骰子条目

```
val kodein = Kodein {
    bind<Dice>() with provider { RandomDice(6) }
}
```

#### <h2 id="4.3">4.3. Singleton binding</h2>

这种类型绑定到此类型的实例将在第一次通过singleton函数绑定的时候延迟创建，该函数不带参数并且返回绑定类型的对象(例如。（）→T)。

因此，Singleton函数只会被调用一次: 在第一次需要实例的时候

示例: 创建将在首次访问到达初始化的DataSource 单例
```
val kodein = Kodein {
    bind<DataSource>() with singleton { SqliteDS.open("path/to/file") }
}
```

##### <h2 id="4.3.1">4.3.1. 非同步单例</h2>

根据定义，只能有一个单例实例，这意味着只能构造一个实例。为了达到这个目的，Kodein synchronizes 构造器。这意味着，当请求单个实例并且不可用时，Kodein使用同步互斥锁来确保对同一类型的其他请求将等待构造此实例

虽然这种行为是确保单例正确性的唯一方法，但它也很昂贵（由于互斥锁）并降低了启动性能。

如果你需要提高启动性能，并且您清楚您在做什么，你可以禁用这个同步锁

示例: 创建一个DataSource非同步单例
```
val kodein = Kodein {
    bind<DataSource>() with singleton(sync = false) { SqliteDS.open("path/to/file") }
}
```
使用sync = false意味着:
* 构造器将没有同步锁。
* 可能有多个实例构建。
* 实例将尽可能重用。

##### <h2 id="4.3.2">4.3.2 渴望单例</h2>
这与常规singleton相同，除了一旦创建Kodein实例并定义了所有绑定，它就会调用提供的函数。
示例: 创建一个DataSource单例，一旦绑定代码块结束它将被创建
```
val kodein = Kodein {
    // The SQLite connection will be opened as soon as the kodein instance is ready
    bind<DataSource>() with eagerSingleton { SqliteDS.open("path/to/file") }
}
```

#### <h2 id="4.4">4.4 Factory binding</h2>
这将类型绑定到工厂函数，该工厂函数接受已定义类型的参数并返回绑定类型的对象（例如（A）→T）。
每当需要绑定类型的实例时，就会调用提供的函数。
示例: 在每次需要时创建一个新骰子，根据表示边数的整数
```
val kodein = Kodein {
    bind<Dice>() with factory { sides: Int -> RandomDice(sides) }
}
```
##### <h2 id="4.4.1">4.4.1 Multi-arguments factories</h2>
工厂可以接受多个（最多5个）参数：
示例: 在每次需要时创建一个新骰子，根据表示边数的整数
```
val kodein = Kodein {
    bind<Dice>() with factory { startNumber: Int, sides: Int -> RandomDice(sides) }
}
```

#### <h2 id="4.5">4.5. Multiton binding</h2>
A multiton can be thought of a "singleton factory": it guarantees to always return the same object given the same argument. In other words, for a given argument, the first time a multiton is called with this argument, it will call the function to create an instance; and will always yield that same instance when called with the same argument.

Example: creates one random generator for each value
```
val kodein = Kodein {
    bind<RandomGenerator>() with multiton { max: Int -> SecureRandomGenerator(max) }
}
```
Just like a factory, a multiton can take multiple (up to 5) arguments.

##### <h2 id="4.5.1">4.5.1 non-synced multiton</h2>
就像单例一样，可以禁用多例同步：

示例: 非同步多态
```
val kodein = Kodein {
    bind<RandomGenerator>(sync = false) with multiton { max: Int -> SecureRandomGenerator(max) }
}
```

#### <h2 id="4.6">4.6. Referenced singleton or multiton binding</h2>
A referenced singleton is an object that is guaranteed to be single as long as a reference object can return it. A referenced multiton is an object that is guaranteed to be single for the same argument as long as a reference object can return it.

A referenced singleton or multiton needs a "reference maker" in addition to the classic construction function that determines the type of reference that will be used.

Kodein comes with three reference makers for the JVM:

引用的单例是一个对象，只要引用对象可以返回它，它就可以保证是单一的。引用的多态对象是一个对象，只要引用对象可以将其返回，它可以保证对于同一参数是单一的。

引用的单例或多例除了需要确定将要使用的引用类型的经典构造函数之外，还需要“引用创建者”。

Kodein随附三个JVM参考制造者：

JVM: Soft & weak

These are objects that are guaranteed to be single in the JVM at a given time, but not guaranteed to be single during the application lifetime. If there are no more strong references to the instances, they may be GC’d and later, re-created.

Therefore, the provided function may or may not be called multiple times during the application lifetime.

Example: creates a Cache object that will exist only once at a given time

```
val kodein = Kodein {
    bind<Map>() with singleton(ref = softReference) { WorldMap() } 
    bind<Client>() with singleton(ref = weakReference) { id -> clientFromDB(id) } 
}
```
① Because it’s bound by a soft reference, the JVM will GC it before any OutOfMemoryException can occur.
② Because it’s bound by a weak reference, the JVM will GC it is no more referenced.


JVM: 软引用 & 弱引用

这些对象在给定的时间保证在jvm中是单个的，但在应用程序生存期内不保证是单个的。如果没有更多强引用，则可以通过GC进行创建，然后再重新创建。

因此，在应用程序的生命周期中，可能多次调用提供的函数，也可能不会多次调用该函数。

示例：创建一个在给定时间仅存在一次的Cache对象
```
val kodein = Kodein {
    bind<Map>() with singleton(ref = softReference) { WorldMap() } 
    bind<Client>() with singleton(ref = weakReference) { id -> clientFromDB(id) } 
}
```
① 由于它是通过软引用绑定的，JVM将在任何OutOfMemoryException可能发生的时候回收它
② 由于它是通过弱引用绑定的，JVM将在没有更多引用的时候回收它。

Weak singletons use JVM’s WeakReference while soft singletons use JVM’s SoftReference.

JVM: Thread local

This is the same as the standard singleton binding, except that each thread gets a different instance. Therefore, the provided function will be called once per thread that needs the instance, the first time it is requested.

Example: creates a Cache object that will exist once per thread

```
val kodein = Kodein {
    bind<Cache>() with singleton(ref = threadLocal) { LRUCache(16 * 1024) }
}
```
> Semantically, thread local singletons should use [scoped-singletons], the reason it uses a referenced singleton is because Java’s ThreadLocal acts like a reference.

> Thread locals are not available in JavaScript.

弱单例使用JVM的WeakReference，而软单例使用JVM的SoftReference

JVM：本地线程

这与标准单例绑定相同，除了每个线程获得不同的实例。因此，所提供的函数将在需要实例的每个需要实例的线程中被调用一次。

示例：创建一个每个线程将存在一次的Cache对象

```
val kodein = Kodein {
    bind<Cache>() with singleton(ref = threadLocal) { LRUCache(16 * 1024) }
}
```

> 从语义上讲，线程局部单例应该使用[scoped-singletons]，它使用引用的单例的原因是因为Java的ThreadLocal就像引用一样。

> 线程本地语言在JavaScript中不可用。



#### <h2 id="4.7">4.7. Instance binding</h2>
This binds a type to an instance that already exist.

Example: a DataSource binding to an already existing instance.

```
val kodein = Kodein {
    bind<DataSource>() with instance(SqliteDataSource.open("path/to/file")) 
}
```
①Instance is used with parenthesis: it is not given a function, but an instance.


#### <h2 id="4.8">4.9. Constant binding</h2>
It is often useful to bind "configuration" constants.

> Constants are always tagged.

Example: two constants
```
val kodein = Kodein {
    constant(tag = "maxThread") with 8 
    constant(tag = "serverURL") with "https://my.server.url" 
}
```
① Note the absence of curly braces: it is not given a function, but an instance.

>	You should only use constant bindings for very simple types without inheritance or interface (e.g. primitive types and data classes).

#### <h2 id="4.8">4.9. Direct binding</h2>
Sometimes, it may seem overkill to specify the type to bind if you are binding the same type as you are creating.

For this use case, you can transform any bind<Type>() with …​ to bind() from …​.
	
Example: direct bindings
```
val kodein = Kodein {
    bind() from singleton { RandomDice(6) }
    bind("DnD20") from provider { RandomDice(20) }
    bind() from instance(SqliteDataSource.open("path/to/file"))
}
```
>This should be used with care as binding a concrete class and, therefore, having concrete dependencies is an anti-pattern that later prevents modularisation and mocking / testing.

>	When using kodein-generic-* and binding a generic type, the bound type will be the specialized type,
e.g. bind() from singleton { listOf(1, 2, 3, 4) } registers the binding to List<Int>.

> 	If you are using Kodein/Native, because of this bug, you need to use the uppercase version: Bind() from. This issue has been fixed and the bind() from syntax will be available to Kodein/Native as soon as Kotlin/Native 0.6 is released.

#### <h2 id="4.9">4.10. Subtypes bindings</h2>
Kodein allows you register a "subtype bindings factory". These are big words for a simple concept that’s best explained with an example:

Example: direct bindings
```
val kodein = Kodein {
    bind<Controller>().subtypes() with { type ->
        when (type.jvmType) { 
            MySpecialController::class.java -> singleton { MySpecialController() }
            else -> provider { myControllerSystem.getController(type.jvmType) }
        }
    }
}
```
① 	As type is a TypeToken<*>, you can use .jvmType to get the JVM type (e.g. Class or ParameterizedType).

In essence, bind<Whatever>().subtypes() with { type → binding } allows you to register, in Kodein, a binding factory that will be called for subtypes of the provided type.


#### <h2 id="4.10">4.11. Transitive dependencies</h2>
With those lazily instantiated dependencies, a dependency (very) often needs another dependency. Such classes can have their dependencies passed to their constructor. Thanks to Kotlin’s killer type inference engine, Kodein makes retrieval of transitive dependencies really easy.

Example: a class that needs transitive dependencies
```
class Dice(private val random: Random, private val sides: Int) {
/*...*/
}
```

It is really easy to bind this RandomDice with its transitive dependencies, by simply using instance() or instance(tag).

Example: bindings of a Dice and of its transitive dependencies
```
val kodein = Kodein {
    bind<Dice>() with singleton { Dice(instance(), instance(tag = "max")) } 

    bind<Random>() with provider { SecureRandom() } 
    constant(tag "max") with 5 
}
```
① Binding of Dice. It gets its transitive dependencies by using instance() and instance(tag).
② Bindings of Dice transitive dependencies.

> The order in which the bindings are declared has no importance whatsoever.

The binding functions are in the same environment as the newInstance function described in the dependency injection section. You can read it to learn more about the instance, provider and factory functions available to the function.

Transitive factory dependencies

Maybe you need a dependency to use one of its functions to create the bound type.

Example: using a DataSource to create a Connection.

```
val kodein = Kodein {
    bind<DataSource>() with singleton { MySQLDataSource() }
    bind<Connection>() with provider { instance<DataSource>().openConnection() } 
}
```
① 	Using a DataSource as a transitive factory dependency. 

#### <h2 id="4.11">4.12. Being responsible for its own retrieval</h2>
If the bound class is KodeinAware, you can pass the kodein object to the class so it can itself use the Kodein container to retrieve its own dependencies.

Example: bindings of Manager that is responsible for retrieving its own dependencies
```
val kodein = Kodein {
    bind<Manager>() with singleton { ManagerImpl(kodein) } 
}
```
① 	ManagerImpl is given a Kodein instance.



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
