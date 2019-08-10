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
Kodein允许你:
* 延迟初始化你的依赖项在被需要的时候
* 不用关心依赖项初始化的顺序
* 轻松绑定类或接口的 instance，provider或者factory
* 轻松调试依赖项绑定和递归
Kodein是一个很好的选择因为:
* 它轻量，快速并且经过优化(广泛使用inline关键字)
* 它提供了一个非常简单并且可读的声明性DSL风格
* 它不受类型擦除的影响 (对比 Java)
* 它与Android完美集成
* 它提出了kotlin非常惯用风格的API.
* 它可以被用在java平台


#### <h2 id="1.2">1.2. Example</h2>


###  <h2 id="2">二.Platform compatibility & Genericity</h2>


###  <h2 id="3">三.Install</h2>
#### <h2 id="3.1">3.1. JVM</h2>
#### <h2 id="3.2">3.2. JavaScript (Gradle)</h2>
#### <h2 id="3.3">3.3. Native (Gradle)</h2>


###  <h2 id="4">四.Bindings: Declaring dependencies</h2>
#### <h2 id="4.1">4.1. Tagged bindings</h2>
#### <h2 id="4.2">4.2. Provider binding</h2>
#### <h2 id="4.3">4.3. Singleton binding</h2>
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
