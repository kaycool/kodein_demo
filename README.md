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

```
class MyApp : Application(), KodeinAware {
	override val kodein by Kodein.lazy { 
	    /* bindings */
	}
}
使用Kodein.lazy可以在绑定时访问Context`
```
       
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

```
class MyActivity : Activity(), KodeinAware {

    override val kodein by kodein() 

    val ds: DataSource by instance()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ds.connect() 
        /* ... */
    }

}
```

① 通过上下文检索一个应用的 Kodein 对象<br>
② 因为Kodein所有的过程都是延迟加载，kodein 和 DataSource 对象都是仅在被需要的时候才开始加载


#### <h2 id="2.3">3. 使用Trigger</h2>
如果你想在onCreate的时候就加载这些依赖对象，使用trigger可以很方便的实现它。
示例：在实现KodeinAware的Android Activity中使用trigger

```
class MyActivity : Activity(), KodeinAware {

    override val kodein by kodein()

    override val kodeinTrigger = KodeinTrigger() 

    val ds: DataSource by instance()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        kodeinTrigger.trigger() 
        /* ... */
    }

}
```

①、KodeinTrigger()->只需要创建trigger,Kodein就会自动使用它 
②、kodeinTrigger.trigger()->此时将加载kodein和所有依赖项的对象。

> 使用这种方式对应用来说是一种非常重要的提升：因为所有的依赖项都将在onCreate方法中被加载，你可以确保在编译时已正确加载kodein所有依赖项，意味着程序中不会出现kodein非声明的依赖对象。
如果你仅仅使用 instance（no provider or factory），你同样可以确保没有依赖循环

#### <h2 id="2.4">4. View Models</h2>
Kodein，你需要Android context，为此，View Models需要实现AndroidViewModel


在View Models中使用Kodein非常容易:

>如果您希望View Models与Kodein无关，你可以使用View Model Factory.
示例：在实现KodeinAware的Android Activity中使用trigger

```
class MyViewModel(app: Application) : ApplicationViewModel(app), KodeinAware {

    override val kodein by kodein() 

    val repository : Repository by instance()
}
```

①、kodein()->加载应用程序的Kodein容器

###  <h2 id="3">三.Android module</h2>

Kodein-Android提出了一个module，可以轻松加载许多标准的Android服务
>这个module绝对是可选的，你可以选择自由使用它或者保留它。

```
class MyApplication : Application(), KodeinAware {
    override val kodein by Kodein.lazy {
        import(androidXModule(this@MyApplication)) 
	    /* bindings */
    }
}
```

① 可以是androidXModule或androidSupportModule或androidCoreModule

您可以在Kodein-Android [module.kt](https://github.com/Kodein-Framework/Kodein-DI/blob/6.3/framework/android/kodein-di-framework-android-core/src/main/java/org/kodein/di/android/module.kt)文件中看到此module提供的所有内容
示例: 使用kodein获取LayoutInflater
```
class MyActivity : Activity(), KodeinAware {
    override val kodein by kodein()
    val inflater: LayoutInflater by instance() 
}
```

如果你是非Android 类中获取这些类，你需要确定一个Android Context 作为 Kodein context：
示例：使用带有 context 的 kodein 去获取 LayoutInflater

```val inflater: LayoutInflater by kodein.on(getActivity()).instance()```

or

示例：使用带有类context的kodein来获取LayoutInflater

```
class MyUtility(androidContext: Context) : KodeinAware {

    override val kodein by androidContext.kodein()

    override val kodeinContext = kcontext(androidContext) 

    val inflater: LayoutInflater by instance()
}
```

① 定义一个默认的context：用于获取Kodein定义的标准Android系统服务的Android context


###  <h2 id="4">四.Android context translators</h2>
module provides 提供许多context 翻译器。例如,它容许在fragment内部获取一个activity作用域内的单例，无序手动指定这个activity
> android modules 自动注册了这些翻译器
然而，如果你不想使用kodein定义的android modules依赖库，但是又需要这些translators，你可以很简单的注册它们：
示例：引入android module依赖库
```
class MyApplication : Application(), KodeinAware {
    override val kodein by Kodein.lazy {
        import(androidXContextTranslators) 
	    /* bindings */
    }
}
```
	
①可以是androidXContextTranslators或androidSupportContextTranslators或androidCoreContextTranslators

###  <h2 id="5">五.Android scopes</h2>
#### <h2 id="5.1">1. Component scopes</h2>
Kodein 为所有的component(Android or not)提供标准的作用域. 只要 context (= component)存在，WeakContextScope 将保留singleton 和 multiton 

示例：使用 Activity 作用域

```
val kodein = Kodein {
    bind<Controller>() with scoped(WeakContextScope.of<Activity>()).singleton { ControllerImpl(context) } 
}
```

① context的类型为Activity，因为我们使用的是WeakContextScope.of<Activity>(). 
> WeakContextScope与ScopeCloseable不兼容
  
#### <h2 id="5.2">2. Activity retained scope</h2>

Kodein-Android 提供ActivityRetainedScope，它允许 activity-scoped 内的 singletons或者multitons 不受activity restart影响
这就意味着，对于相同的activity，你将获得相同的instance，即使这个activity重启
>这也就意味着你不必保留在创建时传递的activity，因为它可能已经启动并且不在有效

`val kodein = Kodein {
    bind<Controller>() with scoped(ActivityRetainedScope).singleton { ControllerImpl() }
}`

>ActivityRetainedScope与ScopeCloseable不兼容： see documentation.

#### <h2 id="5.3">3. Lifecycle scope</h2>

Kodein-Android提供 AndroidLifecycleScope，它允许 activity作用域内的 singletons 或者 multitons 绑定到component的生命周期上。它使用Android support Lifecycle，所以你需要依赖Android support’s LifecycleOwner components

示例：使用一个AndroidLifecycleScope

```
val kodein = Kodein {
    bind<Controller>() with scoped(AndroidLifecycleScope<Fragment>()).singleton { ControllerImpl(context) }
}
```

① 由于配置更改，这些生命周期不会对activity重启产生影响
② AndroidLifecycleScope与ScopeCloseable不兼容： see documentation.

###  <h2 id="6">六.Layered dependencies</h2>
#### <h2 id="6.1">1. The closest Kodein pattern</h2>
Android components 可以被视作 图层。例如，一个view在Activity 图层定义了一个图层，它本身位于Application图层的顶部

kodein 函数将总是返回最近父层kodein，例如：在一个View或者Fragment当中，它将返回容器Activity 的Kodein，如果这个activity定义了的话，否则它将返回“全局的”Application 层级的Kodein

在一下代码示例中，MyActivity 包含 Fragments，并且这些fragments通过kodein()获取它们的Kodein 对象，它将获得MyActivity Kodein对象，而不是Application的Kodein对象

#### <h2 id="6.2">2. Component based sub Kodein</h2>
在Android，每个component都有它自己的生命周期，很像”mini 应用程序“。你可能需要这些仅在特定组件component及其子组件subcomponents定义的依赖项（例如activity）。Kodein 允许你去创建一个仅在你的组件components存活的Kodein 对象

示例：定义一个特定的Activity Kodein

```
class MyActivity : Activity(), KodeinAware {

    override val kodein by subKodein(kodein()) { 
        /* activity specific bindings */
    }

}
```
①、创建一个对此activity 和 此activity的所有组件有效的 子 Kodein容器

> 您可以通过定义复制模式来定义父kodein的扩展方式、


示例：定义一个复制所有父类Binding依赖项的特定activity kodein
```
override val kodein by subKodein(kodein(), copy = Copy.All) {
    /* component specific bindings */
}
```

#### <h2 id="6.3">3. Activity retained sub Kodein</h2>
Kodein-Android 为Activities提供了retainedSubKodein。它创建了一个不受activity重启影响的Kodein对象
> 这意味着您永远不应该访问可能已重新启动且不再有效的activity容器，如果只是使用subKodein的话！


```
class MyActivity : Activity(), KodeinAware {

    override val kodein: Kodein by retainedSubKodein(kodein()) { 
        /* activity specific bindings */
    }

}
```

① 用retainedSubKodein来代替subKodein可确保保留Kodein对象 ，而不会在activity重启的时候创建它

比如：定义一个特定Activity的Kodein

> 您可以通过定义复制模式来定义父kodein的扩展方式、

示例：定义一个复制所有父类Binding依赖项的特定activity kodein
 
```
override val kodein by retainedSubKodein(kodein(), copy = Copy.All) {
    /* component specific bindings */
}
```

###  <h2 id="7">七.Independant Activity retained Kodein</h2>
Kodein提供retainedKodein函数创建一个不依赖于父容器的 Kodein对象
> 这意味着在application context父容器中绑定的所有依赖项在这个新的Kodein对象里是不适用的，你无法通过它获取到父容器中的任何依赖项

示例：定义一个不依赖Kodein 父容器的Kodein对象

```
class MyActivity : Activity() {

    val activityKodein: Kodein by retainedKodein { 
        /* activity specific bindings */
    }

}
```


###  <h2 id="8">八.Kodein in Android without the extension</h2>
#### <h2 id="8.1">1. Being KodeinAware</h2>
让您的Android组件成为KodeinAware非常容易(假设你的Application类是KodeinAware)
##### <h2 id="8.1.1">1. Using lazy</h2>
示例：KodeinAware Activity

```
class MyActivity : Activity(), KodeinAware {
    override val kodein: Kodein by lazy { (applicationContext as KodeinAware).kodein }
}
```

##### <h2 id="8.1.2">2. Using lateinit</h2>
示例：KodeinAware Activity

```
class MyActivity : Activity(), KodeinAware {
    override lateinit var kodein: Kodein
    override fun onCreate(savedInstanceState: Bundle?) {
        kodein = (applicationContext as KodeinAware).kodein
    }
}
```

#### <h2 id="8.2">2. Using LateInitKodein</h2>
如果你不想你的组件component类成为KodeinAware，你可以使用LateInitKodein
If you don’t want the component classes to be KodeinAware, you can use a LateInitKodein;
示例: 一个LateInitKodein的Activity

```
class MyActivity : Activity() {
    val kodein = LateInitKodein()
    override fun onCreate(savedInstanceState: Bundle?) {
        kodein.baseKodein = (applicationContext as KodeinAware).kodein
    }
}
```

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


