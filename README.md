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
如果你想在onCreate的时候就检索这些依赖对象，使用trigger可以很方便的实现它。
示例：在实现KodeinAware的Android Activity中使用trigger

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

①、KodeinTrigger()->只需要创建trigger,Kodein就会自动使用它 
②、kodeinTrigger.trigger()->此时将检索kodein和所有依赖项。

> 使用这种方式对应用来说是一种非常重要的提升：因为所有的依赖项都将在onCreate方法中被检索，你可以确保在编译时已正确检索kodein所有依赖项，意味着没有kodein非声明的依赖。
如果你仅仅使用 instance（no provider or factory），你同样可以确保没有依赖循环

#### <h2 id="2.4">4. View Models</h2>
Kodein，你需要Android context，为此，View Models需要实现AndroidViewModel


在View Models中使用Kodein非常容易:

>如果您希望View Models与Kodein无关，你可以使用View Model Factory.
示例：在实现KodeinAware的Android Activity中使用trigger

`class MyViewModel(app: Application) : ApplicationViewModel(app), KodeinAware {

    override val kodein by kodein() 

    val repository : Repository by instance()
}`
①、kodein()->检索应用程序的Kodein容器

###  <h2 id="3">三.Android module</h2>

Kodein-Android提出了一个module，可以轻松检索许多标准的Android服务
>这个module绝对是可选的，你可以选择自由使用它或者保留它。

`class MyApplication : Application(), KodeinAware {
    override val kodein by Kodein.lazy {
        import(androidXModule(this@MyApplication)) 
	    /* bindings */
    }
}`

① 可以是androidXModule或androidSupportModule或androidCoreModule

您可以在Kodein-Android [module.kt](https://github.com/Kodein-Framework/Kodein-DI/blob/6.3/framework/android/kodein-di-framework-android-core/src/main/java/org/kodein/di/android/module.kt)文件中看到此module提供的所有内容
示例: 使用kodein获取LayoutInflater

`class MyActivity : Activity(), KodeinAware {
    override val kodein by kodein()
    val inflater: LayoutInflater by instance() 
}`

如果你是非Android 类中获取这些类，你需要确定一个Android Context 作为 Kodein context：
示例：使用带有 context 的 kodein 去获取 LayoutInflater

`val inflater: LayoutInflater by kodein.on(getActivity()).instance()`

or

示例：使用带有类context的kodein来获取LayoutInflater

`class MyUtility(androidContext: Context) : KodeinAware {

    override val kodein by androidContext.kodein()

    override val kodeinContext = kcontext(androidContext) 

    val inflater: LayoutInflater by instance()
}`

① 定义一个默认的context：用于获取Android系统服务的Android context


###  <h2 id="4">四.Android context translators</h2>

The android module provides a number of context translators. For example, they allow you to retrieve an activity scoped singleton inside a fragment, without manually specifying the activity.
> The android modules automatically register these translators.
However, if you don’t want to use the android modules, but still need these translators, you can register them easily:
Example: importing the android module

`class MyApplication : Application(), KodeinAware {
    override val kodein by Kodein.lazy {
        import(androidXContextTranslators) 
	    /* bindings */
    }
}`
①	Can either be androidXContextTranslators or androidSupportContextTranslators or androidCoreContextTranslators.


###  <h2 id="5">五.Android scopes</h2>
#### <h2 id="5.1">1. Component scopes</h2>
Kodein provides a standard scope for any component (Android or not). The WeakContextScope will keep singleton and multiton instances as long as the context (= component) lives.

Example: using an Activity scope

`val kodein = Kodein {
    bind<Controller>() with scoped(WeakContextScope.of<Activity>()).singleton { ControllerImpl(context) } 
}`

①	context is of type Activity because we are using the WeakContextScope.of<Activity>().
> WeakContextScope is NOT compatible with ScopeCloseable.
  
#### <h2 id="5.2">2. Activity retained scope</h2>
Kodein-Android provides the ActivityRetainedScope, which is a scope that allows activity-scoped singletons or multitons that are independent from the activity restart.
This means that for the same activity, you’ll get the same instance, even if the activity restarts.
> This means that you should never retain the activity passed at creation because it may have been restarted and not valid anymore!

`val kodein = Kodein {
    bind<Controller>() with scoped(ActivityRetainedScope).singleton { ControllerImpl() }
}`

>This scope IS compatible with ScopeCloseable: see documentation.

#### <h2 id="5.3">3. Lifecycle scope</h2>
Kodein-Android provides the AndroidLifecycleScope, which is a scope that allows activity-scoped singletons or multitons that are bound to a component lifecycle. It uses Android support Lifecycle, so you need to use Android support’s LifecycleOwner components.
Example: using an Activity retained scope

`val kodein = Kodein {
    bind<Controller>() with scoped(AndroidLifecycleScope<Fragment>()).singleton { ControllerImpl(context) }
}`

① These lifecycles are NOT immune to activity restart due to configuration change.
② This scope IS compatible with ScopeCloseable: see documentation.

###  <h2 id="6">六.Layered dependencies</h2>
#### <h2 id="6.1">1. The closest Kodein pattern</h2>
Android components can be thought as layers. For example, a View defines a layer, on top of an Activity layer, itself on top of the Application layer.

The kodein function will always return the kodein of the closest parent layer. In a View or a Fragment, for example, it will return the containing Activity’s Kodein, if it defines one, else it will return the "global" Application Kodein.

In the following code example, if MyActivity contains Fragments, and that these fragments get their Kodein object via kodein(), they will receive the MyActivity Kodein object, instead of the Application one.

#### <h2 id="6.2">2. Component based sub Kodein</h2>
In Android, each component has its own lifecycle, much like a "mini application". You may need to have dependencies that are defined only inside a specific component and its subcomponents (such as an activity). Kodein allows you to create a Kodein instance that lives only inside one of your components:

Example: defining an Activity specific Kodein

`class MyActivity : Activity(), KodeinAware {

    override val kodein by subKodein(kodein()) { 
        /* activity specific bindings */
    }

}`
①、Creating a sub Kodein container that is valid for this activity and all components of this activity.

> You can define the way the parent kodein is extended by defining the copy mode:
> Example: defining an Activity specific Kodein that copies all parent bindings
> `override val kodein by subKodein(kodein(), copy = Copy.All) {
    /* component specific bindings */
}`

#### <h2 id="6.3">3. Activity retained sub Kodein</h2>
Kodein-Android provides retainedSubKodein for Activities. It creates a Kodein object that is immune to activity restarts.
> 	This means that you should never access the containing activity it may have been restarted and not valid anymore!
Example: defining an Activity specific Kodein

`class MyActivity : Activity(), KodeinAware {

    override val kodein: Kodein by retainedSubKodein(kodein()) { 
        /* activity specific bindings */
    }

}`
① 	Using retainedSubKodein instead of subKodein ensures that the Kodein object is retained and not recreated between activity restarts.

Example: defining an Activity specific Kodein

> You can define the way the parent kodein is extended by defining the copy mode:
> Example: defining an Activity specific Kodein that copies all parent bindings
> `override val kodein by retainedSubKodein(kodein(), copy = Copy.All) {
    /* component specific bindings */
}`

###  <h2 id="7">七.Independant Activity retained Kodein</h2>
Kodein provides the retainedKodein function that creates a Kodein instance that is independendant from the parent.
> This means that all bindings in the application context are NOT available through this new Kodein.
Example: defining an independant Kodein Container.

`class MyActivity : Activity() {

    val activityKodein: Kodein by retainedKodein { 
        /* activity specific bindings */
    }

}`



###  <h2 id="8">八.Kodein in Android without the extension</h2>
#### <h2 id="8.1">1. Being KodeinAware</h2>
It is quite easy to have your Android components being KodeinAware (provided that your Application class is KodeinAware).
##### <h2 id="8.1.1">1. Using lazy</h2>
Example: a KodeinAware Activity

`class MyActivity : Activity(), KodeinAware {
    override val kodein: Kodein by lazy { (applicationContext as KodeinAware).kodein }
}`

##### <h2 id="8.1.2">2. Using lateinit</h2>
Example: a KodeinAware Activity

`class MyActivity : Activity(), KodeinAware {
    override lateinit var kodein: Kodein
    override fun onCreate(savedInstanceState: Bundle?) {
        kodein = (applicationContext as KodeinAware).kodein
    }
}`

#### <h2 id="8.2">2. Using LateInitKodein</h2>
If you don’t want the component classes to be KodeinAware, you can use a LateInitKodein:
Example: an Activity with LateInitKodein

`class MyActivity : Activity() {
    val kodein = LateInitKodein()
    override fun onCreate(savedInstanceState: Bundle?) {
        kodein.baseKodein = (applicationContext as KodeinAware).kodein
    }
}`

#### <h2 id="8.3">3. Being Kodein independant</h2>
If you want your components to be Kodein-independent, you can use the dependency holder pattern:
##### <h2 id="8.3.1">1. The dependency holder pattern</h2>

`class MyActivity : Activity() {

    class Deps(
            val ds: DataSource,
            val ctrl: controller
    )

    val deps by lazy { (applicationContext as MyApplication).creator.myActivity() }

    val ds by lazy { deps.ds }
    val ctrl by lazy { deps.ctrl }

    /* ... */
}

class MyApplication : Application() {

	interface Creator {
	    fun myActivity(): MyActivity.Deps
	}

	val creator: Creator = KodeinCreator()

    /* ... */
}

class KodeinCreator : MyApplication.Creator {

    private val kodein = Kodein {
        /* bindings */
    }.direct

    override fun myActivity() = kodein.newInstance { MyActivity.Deps(instance(), instance()) }
}`

##### <h2 id="8.3.2">2. View Model Factory</h2>
If you want your view models to be independant from Kodein, then you need to inject them (meaning passing their dependencies by constructor). To do that, you need to create your own ViewModelProvider.Factory.
Here is a simple one:
A Kodein View Model Factory

`class KodeinViewModelFactory(val kodein: Kodein) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T =
            kodein.direct.Instance(TT(modelClass))
}`


