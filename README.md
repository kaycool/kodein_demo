# Kodein DI on Android
>如何使用kodein：使用依赖或者导入kodein-android的library<br>
>kodein-android扩展：使用kodein为项目添加多个android特定的公用对象，你根据需要决定是否依赖它，您可以前往[Android demo project](https://github.com/Kodein-Framework/Kodein-DI/tree/6.3/demo/demo-android)查看用法

* [一.Install](#1)

* [二.Retrieving](#2)
   * [Getting a Kodein object](#2.1)
   * [Being KodeinAware](#2.2)
   * [Using a Trigger](#2.3)
   * [View Models](#2.4)
   
* [三.Android module](#3)

* [四.Android context translators](#4)

* [五.Android scopes](#5)
   * [Component scopes](#5.1)
   * [Activity retained scope](#5.2)
   * [Lifecycle scope](#5.3)

* [六.Layered dependencies](#6)
   * [The closest Kodein pattern](#6.1)
   * [Component based sub Kodein](#6.2)
   * [Activity retained sub Kodein](#6.3)

* [七.Independant Activity retained Kodein](#7)

* [八.Kodein in Android without the extension](#8)
   * [Being KodeinAware](#8.1)
     * [Using lazy](#8.1.1)
     * [Using lateinit](#8.1.2)
   * [Using LateInitKodein](#8.2)
   * [Being Kodein independant](#8.3)
     * [The dependency holder pattern](#8.3.1)
     * [View Model Factory](#8.3.2)


###  <h2 id="1">一.Install</h2>
如何使用kodein-android：
1. 在你的app build.gradle 文件中添加以下的依赖：

        implementation 'org.kodein.di:kodein-di-generic-jvm:6.3.3'
        implementation 'org.kodein.di:kodein-di-framework-android-???:6.3.3'
    
    Kodein 提供以下支持库可供您选择，根据您项目依赖的support库的版本 Android support or AndroidX，来选择对应依赖。
    
    |  Android类型   |  依赖具体内容 |
    |  ----  | ----  |
    |Barebone Android  |        implementation 'org.kodein.di:kodein-di-framework-android-core:6.3.3'
    |Android + Support library  |implementation 'org.kodein.di:kodein-di-framework-android-support:6.3.3'  |
    |Android + AndroidX library  |implementation 'org.kodein.di:kodein-di-framework-android-x:6.3.3' |

>除了声明Kodein-Android依赖外，必须声明kodein-di-generic-jvm或者kodein-di-erased-jvm 

>如果您的项目使用的是SupportFragment，您必须依赖-support库，否则您应该依赖 android-x

2. 在Android Application让它实现KodeinAware接口声明依赖绑定 .
Example: an Android Application class that implements KodeinAware

        `class MyApp : Application(), KodeinAware {
          override val kodein by Kodein.lazy { 
              /* bindings */
          }
        }
        使用Kodein.lazy可以在绑定时访问Context`
        > 不要忘记在minifest当中声明MyApp
3.在Activities，Fragments 和其它context实现KodeinAware接口 android类中，使用kodein方法取回这些声明的Kodein对象

4. 检索您的依赖项!


###  <h2 id="2">二.Retrieving</h2>
#### <h2 id="2.1">1. 获取kodein对象</h2>
你可以使用以下方式获取 Kodein 对象
* 在Android上下文类中（比如Context，Activity，Fragment等）使用kodein()
* 在其它非上下文类中(比如不具备上下文的Utils类等）使用 kodein(context) or kodein { context }
>kodein 方法仅在你的Android Application实现KodeinAware接口才有效<br>
>kodein 结果将被缓存 而不是在使用时多次被创建

#### <h2 id="2.2">2. 成为 KodeinAware</h2>
让您的Android类成为KodeinAware非常简单<br>
示例：KodeinAware in Android Activity


`class MyActivity : Activity(), KodeinAware {

    override val kodein by kodein() 

    val ds: DataSource by instance()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ds.connect() 
        /* ... */
    }

}`

① 通过上下文检索一个应用的 Kodein 对象<br>
② 因为Kodein所有的过程都是延迟加载，kodein 和 DataSource 对象都是仅在被需要的时候才开始检索



#### <h2 id="2.3">3. 使用Trigger</h2>
If you want all dependencies to be retrieved at onCreate, you can very easily use a trigger:
Example: using an trigger in a KodeinAware Android Activity

`class MyActivity : Activity(), KodeinAware {

    override val kodein by kodein()

    override val kodeinTrigger = KodeinTrigger() 

    val ds: DataSource by instance()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        kodeinTrigger.trigger() 
        /* ... */
    }

}`

①、	Just create a trigger, and Kodein will automatically use it.
②、	The kodein AND all dependencies will both be retrieved at that time.

>Using this approach has an important advantage: as all dependencies are retrieved in onCreate, you can be sure that all your dependencies have correctly been retrieved, meaning that there were no non-declared dependency.
If you only use instance (no provider or factory), you can also be sure that there were no dependency loop.

#### <h2 id="2.4">4. View Models</h2>
To use Kodein, you need an Android context. For that, View Models need to implement AndroidViewModel.

It is very easy to use Kodein inside View Models:

>If you prefer your View Models to be independant from Kodein, you can use a View Model Factory.
Example: using an trigger in a KodeinAware Android Activity

`class MyViewModel(app: Application) : ApplicationViewModel(app), KodeinAware {

    override val kodein by kodein() 

    val repository : Repository by instance()
}`
①、	Retrieving the application’s Kodein container.

###  <h2 id="3">三.Android module</h2>

###  <h2 id="4">四.Android module</h2>

###  <h2 id="5">五.Android scopes</h2>
#### <h2 id="5.1">1. Component scopes</h2>
#### <h2 id="5.2">2. Activity retained scope</h2>
#### <h2 id="5.3">3. Lifecycle scope</h2>


###  <h2 id="6">六.Layered dependencies</h2>
#### <h2 id="6.1">1. The closest Kodein pattern</h2>
#### <h2 id="6.2">2. Component based sub Kodein</h2>
#### <h2 id="6.3">3. Activity retained sub Kodein</h2>

###  <h2 id="7">七.Independant Activity retained Kodein</h2>

###  <h2 id="8">八.Kodein in Android without the extension</h2>
#### <h2 id="8.1">1. Being KodeinAware</h2>
##### <h2 id="8.1.1">1. Using lazy</h2>
##### <h2 id="8.1.2">2. Using lateinit</h2>
#### <h2 id="8.2">2. Using LateInitKodein</h2>
#### <h2 id="8.3">3. Being Kodein independant</h2>
##### <h2 id="8.3.1">1. The dependency holder pattern</h2>
##### <h2 id="8.3.2">2. View Model Factory</h2>



