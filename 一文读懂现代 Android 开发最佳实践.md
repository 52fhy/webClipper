# 一文读懂现代 Android 开发最佳实践
What is MAD？
------------

![](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOjU6ictVsS5JBlicic9vewgea1ECACthClLUyOYiaKQWCADeFhcETKmnxOuGLRDD7AcGbGYpF2XzXtoOw/640?wx_fmt=png)

> https://developer.android.com/series/mad-skills

MAD 的全称是 Modern Android Development，它是一系列技术栈和工具链的集合，涵盖了从编程语言到开发框架等各个环节。

![](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOjU6ictVsS5JBlicic9vewgea1y3kIibXC93HicpMRqcnN0QeyVSicV2ia1rCjJAY7fW5RCV7AiamXYtXCXKw/640?wx_fmt=png)

Android 自 08 年诞生之后的多年间 SDK 变化一直不大，开发方式较为固定。13 年起技术更新逐渐加速，特别是 17 年之后， 随着 Kotlin 及 Jetpack 等新技术的出现 Android 开发方式发生了很大变化，去年推出的 Jetpack Compose 更是将这种变化推向了新阶段。Goolge 将这些新技术下的开发方式命名为 MAD ，以此区别于旧有的低效的开发方式。

MAD 可以指导开发者更高效地开发出优秀的移动应用，它的优势主要体现在以下几点：

*   **值得信赖**：汇聚 Google 在 Android 行业十余年的前沿开发经验
    
*   **入门友好**：提供大量 Demo 和参考文档，适用于不同阶段不同规模的项目
    
*   **高效启动**：通过 Jeptack 以及 Jetpack Compose 等框架，可以迅速搭建你的项目
    
*   **自由选择**：框架丰富多样，可与传统语言、原生开发、开源框架自由搭配
    
*   **体验一致**：不同设备不同版本系统下也具备一致的开发体验
    

### MAD 助力应用出海

近期我们完成了一款 AI 特效类应用在 GooglePlay 的上架，此应用可将用户自己的头像图片经算法加工成各种艺术效果。应用一经上架便广受好评，这一切正是得益于我们在项目中对 MAD 技术的综合运用，我们在最短时间内完成了全部开发，并打造了出色的用户体验。

![](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOjU6ictVsS5JBlicic9vewgea1brdiafFvRxswElCVUhS2mBk0gfgKmfk4759ZfHefVia493XNruGsG66A/640?wx_fmt=png)

在 MAD 的指导下项目的代码架构也更加合理、更具可维护性。下图是项目中 MAD 的整体应用情况：

![](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOjU6ictVsS5JBlicic9vewgea1GVm6mjwlpu9W5FuHbZRdVeq0FFre2bsTEsNvon1E66aUAIhia3xTgkg/640?wx_fmt=png)

接下来，本文将分享一些我们在对 MAD 实践过程中的心得和案例。

1\. Kotlin
----------

![](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOjU6ictVsS5JBlicic9vewgea1AgyobktR2zLJMUEqv5lb6diaINjP5apHhoWk1QyhMs34f0q4jEHH3gQ/640?wx_fmt=png)

Kotlin 是 Andorid 认可的首选开发语言，我们的项目中，所有代码都使用 Kotlin 开发。Kotlin 的语法十分简洁，相对于 Java 同等功能的代码规模可以减少 25%。此外 Kotlin 还具有很多 Java 所不具备的优秀特性：

### 1.1 Safety

Kotlin 在安全性方面有很多优秀的设计，比如空安全以及数据的不可变性。

#### Null Safety

Kotlin 的空安全特性让很多运行时 NPE 提前到编译期暴露和发现，有效降低线上崩溃的发生。我们在代码中重视对 Nullable 类型的判断和处理，我们在数据结构定义时都力求避免出现可空类型，最大限度降低判空成本；

`interface ISelectedStateController<DATA> {  
    fun getStateOrNull(data: DATA): SelectedState?  
    fun selectAndGetState(data: DATA): SelectedState  
    fun cancelAndGetState(data: DATA): SelectedState  
    fun clearSelectState()  
}

// 使用 Elvis 提前处理 Nullable  
fun <DATA> ISelectedStateController<DATA>.getSelectState(data: DATA): SelectedState {  
    return getStateOrNull(data) ?: SelectedState.NON_SELECTED  
}

`

Java 时代我们只能通过 `getStateOrNull` 这类的命名规范来提醒返回值的可空，Kotlin 通过 `？`让我们可以更好地感知 Nullable 的风险；我们还可以使用 Elvis 操作符 `?:` 将 Nullable 转成 NonNull 便于后续使用；Kotlin 的 `!!` 让我们更容易发现 NPE 的潜在风险并可以诉诸静态检查给予警告。

Kotlin 的默认参数值特性也可以用来防止 NPE 的出现，像下面这样的结构体定义，在反序列化等场景中不必担心 Null 的出现。

`data class BannerResponse(  
    @SerializedName("data") val data: BannerData = BannerData(),  
    @SerializedName("message") val message: String = "",  
    @SerializedName("status_code") val statusCode: Int = 0  
)  
`

我们在全面拥抱 Kotlin 之后，NPE 方面的崩溃率只有 0.3 ‰，而通常 Java 项目的 NPE 会超过 1 ‰

#### Immutable

Kotlin 的安全性还体现在数据不会被随意修改。我们在代码中大量使用 `data class` 并且要求属性使用 `val` 而非 `var` 定义，这有利于单向数据流范式在项目中的推广，在架构层面实现数据的读写分离。

`data class HomeUiState(  
    val bannerList: Result<BannerItemModel> = Result.Success(emptyList()),  
    val contentList: Result<ContentViewModel> = Result.Success(emptyList()),  
)

sealed class Result<T> {  
    data class Success<T>(val list: List<T> = emptyList()) : Result<T>()  
    data class Error<T>(val message: String) : Result<T>()  
}

`

如上，我们使用 data class 定义 `UiState` 用在 ViewModel 中。val 声明属性保证了 State 的不可变性。使用密封类定义 `Result` 有利于对各种请求结果进行枚举，简化逻辑。

`private val _uiState = MutableStateFlow(HomeUiState())  
val uiState: StateFlow<HomeUiState> = _uiState.asStateFlow()

_uiState.value =  
    _uiState.value.copy(bannerList = Result.Success(it))

`

需要更新 State 时，借助 data class 的 `copy` 方法可以快捷地拷贝构造一个新实例。

Immutable 还体现在集合类的类型上。我们在项目中提倡非必要不使用 `MutableList` 这样的 Mutable 类型，可以减少 `ConcurrentModificationException` 等多线程问题的发生，同时更重要的是避免了因为 Item 篡改带来的数据一致性问题：

`viewModel.uiState.collect {  
    when (it) {  
        Result.Success -> bannerAdapter.updateList(it.list)  
        else {...}  
    }  
}

fun updateList(newList: List<BannerItemModel>) {  
    val diffResult = DiffUtil.calculateDiff(BannerDiffCallback(mList, newList), true)  
    diffResult.dispatchUpdatesTo(this)  
}

`

比如上面例子中 UI 侧接收到 UiState 更新通知后，提交 `DiffUtil` 刷新列表。DiffUtil 正常运作的基础正是因为 `mList` 和 `newList` 能时刻保持 Immutable 类型。

### 1.2 Functional

函数在 Kotlin 中是一等公民，可以作为参数或返回值的类型组成高阶函数，高阶函数可以在集合操作符等场景下提供更加易用的 API。

#### Collection operations

`val bannerImageList: List<BannerImageItem> =  
bannerModelList.sortedBy {  
    it.bType  
}.filter {  
    !it.isFrozen()  
}.map {  
    it.image  
}  
`

上面的代码中我们对 `BannerModelList` 依次完成排序、过滤，并转换成 `BannerImageItem` 类型的列表，集合操作符的使用让代码一气呵成。

#### Scope functions

作用域函数是一系列 inline 的高阶函数。它们可以作为代码的粘合剂，减少临时变量等多余代码的出现。

`GalleryFragment().apply {  
    setArguments(arguments ?: Bundle().apply {  
        putInt("layoutId", layoutId())  
    })  
}.let { fragment ->  
   supportFragmentManager.beginTransaction()  
    .apply {  
        if (needAdd) add(R.id.fragment_container, fragment, tag)  
        else replace(R.id.fragment_container, fragment, tag)  
    }.also{  
        it.setCustomAnimations(R.anim.slide_in, R.anim.slide_out)  
    }.commit()  
}  
`

当我们创建并启动一个 Fragment 时，可以基于作用域函数完成各种初始化工作，就像上面例子那样。这个例子同时也提醒我们过度使用这些作用域函数（或集合操作符），也会影响代码的可读性和可调试性，只有“恰到好处”的使用函数式编程才能真正发挥 Kotlin 的优势。

### 1.3 Corroutine

Kotlin 协程让开发者摆脱了回调地狱的出现，同时结构化并发的特性也有助于对子任务更好地管理，Android 的各种原生库和三方库在处理异步任务时都开始转向 Kotlin 协程。

#### Suspend function

在项目中，我们倡导使用挂起函数封装异步逻辑。在数据层 Room 或者 Retorfit 使用挂起函数风格的 API 自不必说，一些表现层逻辑也可以基于挂起函数来实现：

`suspend fun doShare(  
    activity: Activity,  
    contentBuilder: ShareContent.Builder.() -> Unit  
): ShareResult = suspendCancellableCoroutine { cont ->  
    val shareModel = ShareContent.Builder()  
        .setEventCallBack(object : ShareEventCallback.EmptyShareEventCallBack() {  
            override fun onShareResultEvent(result: ShareResult) {  
                super.onShareResultEvent(result)  
                if (result.errorCode == 0) {  
                    cont.resume(result)  
                } else {  
                    cont.cancel()  
                }  
            }  
        }).apply(contentBuilder)  
        .build()  
    ShareSdk.showPanel(createPanelContent(activity, shareModel))  
}  
`

上例的 doShare 用挂起函数处理照片的分享逻辑：弹出分享面板供用户选择分享渠道，并将分享结果返回给调用方。调用方启动分享并同步获取分享成功或失败的结果，代码风格更符合直觉。

#### Flow

项目中使用 Flow 替代 RxJava 处理流式数据，减少包体积的同时，CoroutineScope 可以有效避免数据泄露：

`fun CoroutineScope.getBannerList(): Flow<List<BannerItemModel>> =  
    DatabaseManager.db.bannerDao::getAll.asFlow()  
            .onCompletion {  
                this@Repository::getRemoteBannerList.asFlow().onEach {  
                    launch {  
                        DatabaseManager.db.bannerDao.deleteAll()  
                        DatabaseManager.db.bannerDao.insertAll(*(it.toTypedArray()))  
                    }  
                }  
            }.distinctUntilChanged()  
`

上面的例子用于从多个数据源获取 `BannerList` 。我们增加了磁盘缓存的策略，先请求本地数据库数据，再请求远程数据。Flow 的使用可以很好地满足这类涉及多数据源请求的场景。而另一面在调用侧，只要提供合适的 CoroutineScope 就不必担心泄露的发生。

### 1.4 KTX

一些原本基于 Java 实现的 Android 库通过 KTX 提供了针对 Kotlin 的扩展 API，让它们在 Kotlin 工程中更容易地被使用。

我们的项目使用 Jetpack Architecture Components 搭建 App 基础架构，KTX 帮助我们大大降低了 Kotlin 项目中的 API 使用成本，举几个最常见的 KTX 的例子：

#### fragment-ktx

fragment-ktx 提供了一些针对 Fragment 的 Kotlin 扩展方法，比如 ViewModel 的创建：

`class HomeFragment : Fragment() {  
    private val homeViewModel : HomeViewModel by viewModels()  
    ...  
}  
`

相对于 Java 代码在 Fragment 中创建 ViewMoel 变得极其简单，其背后的是现实活用了各种 Kotlin 特性，十分巧妙。

`inline fun <reified VM : ViewModel> Fragment.viewModels(  
    noinline ownerProducer: () -> ViewModelStoreOwner = { this },  
    noinline factoryProducer: (() -> Factory)? = null  
) = createViewModelLazy(VM::class, { ownerProducer().viewModelStore }, factoryProducer)  
`

`viewModels` 是 Fragment 的 inline 扩展方法，通过 `reified` 关键字在运行时获取泛型类型用来创建具体 ViewModel 实例：

`fun <VM : ViewModel> Fragment.createViewModelLazy(  
    viewModelClass: KClass<VM>,  
    storeProducer: () -> ViewModelStore,  
    factoryProducer: (() -> Factory)? = null  
): Lazy<VM> {  
    val factoryPromise = factoryProducer ?: {  
        defaultViewModelProviderFactory  
    }  
    return ViewModelLazy(viewModelClass, storeProducer, factoryPromise)  
}  
`

`createViewModelLazy` 返回了一个 `Lazy<VM>` 实例，这似的我们可以通过 `by` 关键字创建 ViewModel，这里借助 Kotlin 的代理特性实现了实例的延迟创建。

#### viewmodle-ktx

viewModel-ktx 提供了针对 ViewModel 的扩展方法， 例如 `viewModelScope`，可以随着 ViewModel 的销毁及时终止过期的异步任务，让 ViewModel 更安全地作为数据层与表现层之间的桥梁使用。

`viewModelScope.launch {  
    //监听数据层的数据  
    repo.getMessage().collect {  
        //向表现层发送消息  
        _messageFlow.emit(message)  
    }  
}  
`

实现原理也非常简单：

`val ViewModel.viewModelScope: CoroutineScope  
        get() {  
            val scope: CoroutineScope? = this.getTag(JOB_KEY)  
            if (scope != null) {  
                return scope  
            }  
            return setTagIfAbsent(JOB_KEY,  
                CloseableCoroutineScope(SupervisorJob() + Dispatchers.Main.immediate))  
        }  
`

viewModelScope 本质上是 ViewModle 的扩展属性，通过 custom get 创建 `CloseableCoroutineScope` 的同时，记录到 `JOB_KEY` 的位置中。

`internal class CloseableCoroutineScope(context: CoroutineContext) : Closeable, CoroutineScope {  
    override val coroutineContext: CoroutineContext = context

    override fun close() {  
        coroutineContext.cancel()  
    }  
}

`

CloseableCoroutineScope 其实是一个 `Closeable`，在 ViewModel 的 `onClear` 时查找 JOB\_KEY 并被调用 `close` 以取消 `SupervisorJob` ，终止所有子协程。KTX 活用了 Kotlin 的各种特性和语法糖 ，后面 Jetpack 章节会看到更多 KTX 的使用。

2\. Android Jetpack
-------------------

![](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOjU6ictVsS5JBlicic9vewgea1C01WVGJqEnIjSWJ05Q8ViaGymiatq7ronBZ9aNgz808Ng9P5O34tv84Q/640?wx_fmt=png)

Android 通过 Jetpack 为开发者提供 AOSP 之上的基础能力支持，其范围覆盖了从 UI 到 Data 各个层级，降低了开发者们自造轮子的需求。近期 Jetpack 组件的架构规范又进行了全面升级，帮助我们在开发过程中能更好地贯彻关注点分离这一设计目标。

### 2.1 Architecture

Android 倡导表现层和数据层分离的架构设计，并使用单向数据流（Unidirectional Data Flow）完成数据通信。Jetpack 通过一系列 Lifecycle-aware 的组件支持了 UDF 在 Android 中的落地。

![](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOjU6ictVsS5JBlicic9vewgea14XuXdCCUicGYzFZXYoXTGhQK1icFzgOuzyzRib0g3sd9ot7A9QP8liapUA/640?wx_fmt=png)

UDF 的主要特点和优势如下:

*   **唯一真实源（SSOT）**：UI State 在 ViewModel 集中管理，降低了多数据源之间的同步成本
    
*   **数据自上而下流动**：UI 的更新来 VM 的状态变化，UI 自身不持有状态、不耦合业务逻辑
    
*   **事件自下而上传递**：UI 发送 event 给 VM 对状态集中修改，状态变化可回溯、利于单测
    

项目中凡是涉及 UI 的业务场景都是基于 UDF 打造的。以 `HomePage` 为例，其中包括 `BannerList` 和 `ContentList` 两组数据展示，所有的数据集中管理在 UiState 中。

`

class HomeViewModel() : ViewModel() {

    private val _uiState = MutableStateFlow(HomeUiState())  
    val uiState: StateFlow<HomeUiState> = _uiState.asStateFlow()

    fun fetchHomeData() {  
        fetchJob?.cancel()  
        fetchJob = viewModelScope.launch {  
            with(repo) {  
                //request BannerList  
                try {  
                    getBannerList().collect {  
                        _uiState.value =  
                            _uiState.value.copy(bannerList = Result.Success(it))  
                    }  
                } catch (ioe: IOException) {  
                    // Handle the error and notify the UI when appropriate.  
                    _uiState.value =  
                        _uiState.value.copy(  
                            bannerList = Result.Error(getMessagesFromThrowable(ioe))  
                        )  
                }

                //request ContentList  
                try {  
                    getContentList().collect {  
                        _uiState.value =  
                            _uiState.value.copy(contentList = Result.Success(it))  
                    }  
                } catch (ioe: IOException) {  
                    _uiState.value =  
                        _uiState.value.copy(  
                            contentList = Result.Error(getMessagesFromThrowable(ioe))  
                        )  
                }  
            }  
        }

    }  
}

`

如上代码所示，`HomeViewModel` 从 Repo 获取数据并更新 UiState，View 订阅此状态并刷新 UI。`viewModelScope.launch` 提供的 CoroutineScope 可以随着 ViewModel 的 `onClear` 结束运行中的协程，避免泄露。

数据层我们使用 Repository Pattern 封装本地数据源和远程数据源的具体实现：

`class Repository {  
    fun CoroutineScope.getBannerList(): Flow<List<BannerItemModel>> {

        return DatabaseManager.db.bannerDao::getAll.asFlow()  
            .onCompletion {  
                this@Repository::getRemoteBannerList.asFlow().onEach {  
                    launch {  
                        DatabaseManager.db.bannerDao.deleteAll()  
                        DatabaseManager.db.bannerDao.insertAll(*(it.toTypedArray()))  
                    }  
                }  
            }.distinctUntilChanged()  
    }

    private suspend fun getRemoteBannerList(): List<BannerItemModel> {  
        TODO("Not yet implemented")  
    }  
}

`

以 `getBannerList` 为例，先从数据库请求本地数据加速显示，然后再请求远程数据源更新数据，同时进行持久化，便于下次请求。

UI 层的逻辑很简单，订阅 ViewModel 的数据并刷新 UI 即可。

`@AndroidEntryPoint  
class HomeFragment : Fragment()  {

    @Inject  
    lateinit var viewModel : HomeViewModel

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {  
        super.onViewCreated(view, savedInstanceState)  
        lifecycleScope.launch {  
            repeatOnLifecycle(Lifecycle.State.STARTED) {  
                viewModel.uiState.collect {  
                    // Update UI elements  
                }  
            }  
        }  
    }  
}

`

我们使用 Flow 代替 LiveData 对 UiState 进行封装，`lifecycleScope` 使得 Flow 变身 Lifecycle-aware 组件；`repeatOnLifecycle` 让 Flow 像 LiveData 一样在 Fragment 前后台切换时自动停止数据流的发射，节省资源开销。

### 2.2 Navigation

作为“单 Activity 架构”的实践者，我们选择了使用 Jetpack Navigation 作为 App 的导航组件。Navigation 组件实现了导航设计原则，为跨应用切换或应用内页面间的切换提供了一致的用户体验，并且提供了各种优势，包括：

*   处理 Fragment 事务；
    
*   默认情况下，正确处理往返操作；
    
*   为动画和转场提供标准化资源；
    
*   实现和处理深层链接；
    
*   包括导航界面模式（例如抽屉式导航栏和底部导航），开发者只需完成极少的额外工作；
    
*   提供 Gradle 插件用以保证在不同页面传递参数时类型安全；
    
*   提供了导航图范围的 ViewModel，以在同导航图内的页面进行数据共享；
    

![](https://mmbiz.qpic.cn/mmbiz_gif/5EcwYhllQOjU6ictVsS5JBlicic9vewgea1fw88ssgKvDE8z3f0V722Y6q3PIevibPGejVFtahZnoGq0lxp40ibkpQw/640?wx_fmt=gif)

Navigation 提供了 XML 以及 Kotlin DSL 两种配置方式。我们在项目中发挥 Kotin 的优势，基于类型安全的 DSL 创建导航图，同时通过函数提取为页面统一指定转场动画：

`fun NavHostFragment.initGraph() = run {  
    createGraph(nav_graph.id, nav_graph.dest.home) {  
        fragment<HomeFragment>(nav_graph.dest.effect_detail) {  
            action(nav_graph.action.home_to_effect_detail) {  
                destinationId = nav_graph.dest.effect_detail  
                navOptions {  
                    applySlideInOut()  
                }  
            }  
        }  
    }  
}

//统一指定转场动画  
internal fun NavOptionsBuilder.applySlideInOut() {  
    anim {  
        enter = R.anim.slide_in  
        exit = R.anim.slide_out  
        popEnter = R.anim.slide_in_pop  
        popExit = R.anim.slide_out_pop  
    }  
}

`

在 Activity 中，调用 `initGraph()` 为 Root Fragment 初始化导航图：

`@AndroidEntryPoint  
class MainActivity : AppCompatActivity() {

    private val navHostFragment: NavHostFragment by lazy {  
        supportFragmentManager.findFragmentById(R.id.nav_host) as NavHostFragment  
    }

    override fun onCreate(savedInstanceState: Bundle?) {  
        super.onCreate(savedInstanceState)  
        setContentView(R.layout.activity_main)

        navHostFragment.navController.apply {  
            graph = navHostFragment.initGraph()  
        }  
    }  
}

`

而在 Fragment 中，使用 navigation-fragment-ktx 提供的 `findNavController()` 可以随时基于当前 Destination 进行正确地页面跳转：

`@AndroidEntryPoint  
class EffectDetailFragment : Fragment() {

    /* ... */

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {  
        nextButton.setOnClickListener {  
            findNavController().navigate(nav_graph.action.effect_detail_to_loading))  
        }

        // Back to previous page  
        backButton.setOnClickListener {  
            findNavController().popBackStack()  
        }

        // Back to home page  
        homeButton.setOnClickListener {  
            findNavController().popBackStack(nav_graph.dest.home, false)  
        }  
    }  
}

`

除此以外，我们可以声明全局页面导航，这种方式在引导用户登录注册或前往反馈页等场景有很大用处：

`fun NavHostFragment.initGraph() = run {  
    createGraph(nav_graph.id, nav_graph.dest.home) {  
        /* ... some Fragment destination declaration ... */  
        // --------------- Global ---------------  
        action(nav_graph.action.global_to_register) {  
            destinationId = nav_graph.dest.register  
            navOptions {  
                applyBottomSheetInOut()  
            }  
        }  
    }  
}  
`

### 2.3 Hilt

依赖注入 (Dependency Injection) 是多 Module 工程中的常用的技术，依赖注入作为控制反转设计原则的一种实现方式，有利于实例的生产侧与消费侧的解耦，践行了关注点分离的设计原则，也更有助于单元测试的编写。

![](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOjU6ictVsS5JBlicic9vewgea1tIsNDicngMicpX0ZGWXqxscgQMkciaHALtgq3rn3sW2oKMcrs24XUj6HQ/640?wx_fmt=png)

Hilt 在 Dagger 的基础上构建而成，继承了 Dagger 编译时检查、运行时高性能、可伸缩等优点的同时提供了更友好的 API ，使得 Dagger 使用成本大幅降低。Android Studio 也内置了对 Dagger/Hilt 的支持，后文会介绍。

项目中大量使用了 Hilt 完成依赖注入，进一步提升了代码的编写效率。我们使用 `@Singleton` 提供 Repository 的单例实现，当 Repository 需要 Context 来创建 SharedPreferences 或者 DataStore 时，使用 `@ApplicationContext` 注解传入应用级别的 Context，在需要的地方只需要 `@Inject` 即可注入对象：

`@AndroidEntryPoint  
class RecommendFragment : Fragment() {  
    @Inject  
    lateinit var recommendRepository: RecommendRepository

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {  
        super.onViewCreated(view, savedInstanceState)

        recommendRepository.doSomeThing()  
    }  
}

`

对于一些无法在构造函数中增加注解的三方库的类，我们可以使用 `@Provides` 来告诉 Hilt 如何创建相关实例。例如提供创建 Retorfit API 的实现，省去每次手动创建的工作。

`@Module  
@InstallIn(ActivityComponent::class)  
object ApiModule {

    @Provides  
    fun provideRecommendServiceApi(): RecommendServiceApi {  
        return Retrofit.Builder()  
                .baseUrl("https://example.com")  
                .build()  
                .create(RecommendServiceApi::class.java)  
    }  
}

`

得益于 Hilt 对 Jetpack 其他组件的支持，在 ViewModel 或者 WorkManager 中也同样可以使用 Hilt 进行依赖注入。

`@HiltViewModel  
class RecommendViewModel @Inject constructor(  
    private val recommendRepository: RecommendRepository  
) {

    val recommendList = recommendRepository.fetchRecommendList()  
        .flatMapLatest {  
            flow { emit(it) }  
        }  
        .stateIn(  
            scope = viewModelScope,  
            started = SharingStarted.WhileSubscribed(5000),  
            initialValue = emptyList()  
        )  
}

`

### 2.4 WorkManager

WorkManager 是针对持久性工作而推出的 Jetpack 库，所谓持久性工作指可以跨越应用或者系统重启持续执行的任务，比如应用数据与服务器之间进行同步，或者是上传日志等。WorkManager 对内会根据策略自动选择 `FirebaseJobDispatcher`、`GcmNetworkManager` 或 `JobScheduler` 等执行调度任务，对外则提供了简单一致的 API 方便使用。

WorkManager 默认使用 Jetpack StartUp 库进行初始化，开发者只需关注定义与实现 Worker 即可，无需其他额外工作。WorkManager 向后兼容到 Android 6.0 、覆盖了市面上绝大多数的机型，可以有效取代 Service 完成那些需要长期执行的后台任务。

产品为了减少用户生成头像时上传图片所需时间与流量消耗，会在上传之前对图片进行压缩，但是压缩过程的临时文件会增加 App 所占存储空间，所以我们使用 WorkManager 对清理压缩图片缓存的工作进行调度，在 App 启动后将任务提交给 WorkManager：

`val deleteImageCacheRequest = OneTimeWorkRequestBuilder<DeleteImageCacheWorker>().build()  
WorkManager.getInstance(this).enqueue(deleteImageCacheRequest)

class DeleteImageCacheWorker(  
    context: Context,  
    workParams: WorkerParameters  
) : Worker(context, workParams) {

    override fun doWork(): Result {  
        return try {  
            /* ... do the work ... */  
            Result.success()  
        } catch (e: Exception) {  
            /* return failure() or retry() */  
            Result.failure()  
        }  
    }  
}

`

还有一种场景是用户下载图片。下载需要网络，并且此工作的优先级比较高，因此可以使用 WorkManager 提供的工作约束以及加急工作 (WorkManager 2.7 及以上) 等能力，除此以外还可以对工作的结果信息进行监听，以对用户进行提示：

`val downloadImageRequest = OneTimeWorkRequestBuilder<DownLoadImageWorker>()  
    .setInputData(workDataOf("url" to "https://the-url-of-image.com"))  
    // set network constraint  
    .setConstraints(  
        Constraints.Builder().setRequiredNetworkType(NetworkType.CONNECTED).build()  
    )  
    // make worker expedited  
    .setExpedited(OutOfQuotaPolicy.RUN_AS_NON_EXPEDITED_WORK_REQUEST)  
    .build()  
WorkManager.getInstance(context).enqueue(downloadImageRequest)

val downloadImageFlow = WorkManager.getInstance(context)  
    .getWorkInfoByIdLiveData(downloadImageRequest.id)  
    .asFlow()  
    .shareIn(  
        scope = viewModelScope,  
        started = SharingStarted.WhileSubscribed(5000),  
        replay = 1  
    )

// in Fragment  
viewLifecycleOwner.lifecycleScope.launchWhenCreated {  
    viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {  
        downloadImageFlow.collectLatest {  
            when (it?.state) {  
                WorkInfo.State.ENQUEUED -> {}  
                WorkInfo.State.RUNNING -> {}  
                WorkInfo.State.SUCCEEDED -> {}  
                WorkInfo.State.BLOCKED -> {}  
                WorkInfo.State.FAILED -> {}  
                WorkInfo.State.CANCELLED -> {}  
            }  
        }  
    }  
}

`

### 2.5 StartUp

应用启动时需要做大量初始化工作，例如 SDK 的初始化、基础模块的配置等。StartUp 出现之前我们使用 ContentProvider 完成“无侵”的初始化，避免 `init(Context)` 这类代码在 Application 中的出现。但是 ContentProvider 的创建成本较高，多个 ContentProvider 同时创建会拖慢应用启动速度且初始化时序不可控。

StartUp 只使用一个 ContentProvider 来完成多个组件的初始化，很好地解决了上述 ContentProvider 的各种问题。此外，StartUp 还可以避免 app 模块对其他模块的非必要依赖。例如我们在项目中需要为 local test 渠道单独依赖一个 Module，此 Module 依赖 Context 完成初始化，但我们不希望它被打入 release 包。此时要像下面这样添加 Gradle 依赖即可，app 不需要在代码层面依赖 local\_test 模块。

`if (BuildContext.isLocalTest()) {  
    implementation project(':local_test')  
}  
`

StartUp 库的使用非常简单，只需定义一个 `Initializer` 即可， 定义的同时还可以配置初始化的依赖项，确保核心组件可以最先完成初始化：

`class ServerInitializer : Initializer<ServerManager> {  
    override fun create(context: Context): ServerManager {  
        TODO("init ServerManager and return")  
    }

    override fun dependencies(): List<Class<out Initializer<*>>> {  
        return emptyList()  
    }  
}

class AccountInitializer : Initializer<Unit> {  
    override fun create(context: Context) {  
        TODO("init Account")  
    }

    override fun dependencies(): List<Class<out Initializer<*>>> {  
        return listOf(ServerInitializer::class.java)  
    }  
}

`

在上面的例子中，Account 模块的初始化将会等待 Server 模块初始化完成后才会继续。

### 2.6 Room

local-first 架构的 App 可以提供良好的用户体验，当设备无法访问网络时，用户仍可在离线状态下浏览相应内容。Android 提供了 SQLite 作为访问数据库的 API，但是 SQLite API 比较底层，需要人工确保 SQL 语句的正确性，除此以外，还需要编写大量的模板代码来完成 PO 与 DO 之间的转换。Jetpack Room 在 SQLite 的基础上提供了一个抽象层，帮助开发者更流畅的访问数据库。

Room 主要包含 3 个组件：Database 是数据库持有者，是与底层数据库连接的主要接入点；Entity 代表数据库中的表；DAO 包含用于访问数据库的方法。3 个组件通过注解进行声明：

`@Entity(tableName = "tb_banner")  
data class Banner(  
    @PrimaryKey  
    val id: Long,  
    @ColumnInfo(name = "url")  
    val url: String  
)

@Dao  
interface BannerDao {  
    @Query("SELECT * FROM tb_banner")  
    fun getAll(): List<Banner>

    @Insert(onConflict = OnConflictStrategy.REPLACE)  
    fun insertBanner(banner: Banner)  
}

@Database(entities = arrayOf(Banner::class), version = 1)  
abstract class AppDatabase : RoomDatabase() {  
    abstract fun bannerDao(): BannerDao  
}

`

需要注意的是创建数据库的成本比较高，所以单进程 App 内要保证数据库为单例：

`@Module  
@InstallIn(SingletonComponent::class)  
object AppModule {

    @Provides  
    @Singleton  
    fun provideDatabase(  
        @ApplicationContext applicationContext: Context  
    ): AppDatabase {  
        return Room.databaseBuilder(  
                    applicationContext,  
                    AppDatabase::class.java, "database-name"  
                ).build()  
    }

    @Provides  
    @Singleton  
    fun provideBannerDao(  
        appDatabase: AppDatabase  
    ): BannerDao {  
        return appDatabase.bannerDao()  
    }

}

`

当数据库中的数据发生更新时，我们希望 UI 也能随之自动刷新。得益于 Room 对 Coroutine 以及 RxJava 良好的支持，只需要引入 room-ktx 库或者 room-rxjava2/3 库，DAO 中的方法也可以直接返回 `Flow` 或者 `Observable`，或者直接使用挂起函数：

`@Dao  
interface BannerDao {  
    @Query("SELECT * FROM tb_banner")  
    fun getAll(): Flow<List<Banner>>

    @Query("SELECT * FROM tb_banner")  
    suspend fun getAllSuspend(): List<Banner>>  
}

`

这时候我们只需要在 UI 层对 `Flow` 进行订阅，便可以做到当数据库内容更新时 UI 也随之更新：

`@HiltViewModel  
class BannerViewModel @Inject constructor (  
    // we should use repository rather than access BannerDao directly  
    private val bannerDao: BannerDao  
) : ViewModel() {

    val bannerList: Flow<BannerVO> = bannerDao.getAll().map {  
        it.toVO()  
    }  
}

// in Fragment  
viewLifecycleOwner.lifecycleScope.launchWhenCreated {  
    viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {  
        bannerViewModel.bannerList.collectLatest {  
            bannerAdapter.submitList(it)  
        }  
    }  
}

`

更进一步，在 UI 层我们只订阅数据库中的数据，而在后台使用 WorkManager 发起网络请求，获取到数据后再将最新的数据写入到数据库中。由于数据库访问速度远远快于网络，因此页面可以更快的呈现给用户。

3\. Android Studio
------------------

![](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOjU6ictVsS5JBlicic9vewgea1XhNg25MsgsbHnUOK56TPbM2MJKtTtkpbLREzibOF6MJLMXorde9f4CA/640?wx_fmt=png)

Android Studio 诞生至今一直保持着活跃的版本更新，当前最新版本已经更新至 `Bumblebee | 2021.1.1.21` ，自 4.3 Canary 1 以来 Android Studio 在命名风格上有所调整，更好的对齐了 IntelliJ 平台版本。除了定期发布的稳定版，开发者还可以通过 RC 和 Preview 版本提前体验更多新鲜功能。

随着版本的不断更新，编写和调试代码的体验得到持续的优化，且集成了越来越多的新功能。Layout Instpector ，Device Exploer 等既有功能自不必说，以下这些新特性也为我们的开发、调试提供了巨大的便利。

### 3.1 Database Inspector

![](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOjU6ictVsS5JBlicic9vewgea1g9lDSkeotqxECfLibrb7gYiaFoOUB0DiabkgOIzNTic9eK0PLjhicKNdZZA/640?wx_fmt=png)

我们使用 Room 进行数据持久化，Database Inspector 可以实时查看 Jetpack Room 框架生成的数据库文件，同时也支持实时编辑和部署到设备当中。相较之前需要的 SQLite 命令或者额外导出并借助 DB 工具的方式更为高效和直观。

### 3.2 Realtime Profilers

![](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOjU6ictVsS5JBlicic9vewgea1NUVTMYSuria9FcQEKspUbDicnwPLyJAfwlDAJH2RW7zoImFOlyIk47zg/640?wx_fmt=png)

Android Studio 的 Realtime Profilers 工具可以帮助我们在如下四个方面监测和发现问题，有时在缺少工程代码的情况下通过 Memory Profilers 还可以查看其内部的实例和变量细节。

*   **CPU**：性能剖析器检查 CPU 活动，切换到 Frames 视图还可以界面卡顿追踪
    
*   **Memory**：识别可能会导致应用卡顿、冻结甚至崩溃的内存泄漏和内存抖动，可以捕获堆转储、强制执行垃圾回收以及跟踪内存分配以定位内存方面的问题
    
*   **Battery**：会监控 CPU、网络无线装置和 GPS 传感器的使用情况，并直观地显示其中每个组件消耗的电量，了解应用在哪里耗用了不必要的电量
    
*   **Network**：显示实时网络活动，包括发送和接收的数据以及当前的连接数。这便于您检查应用传输数据的方式和时间，并适当优化代码
    

### 3.3 APK Analyzer

![](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOjU6ictVsS5JBlicic9vewgea1BmA1cxuJ3eQ5DibwtkpkibPQm7EcygjQiaQDwZvU4cND98ChXsujiavGFw/640?wx_fmt=png)

Apk 的下载会耗费网络流量，安装了还会占用存储空间。其体积的大小会对 App 安装和留存产生影响，分析和优化其体积显得尤为必要。

借助 AS 的 APK Analyzer 可以帮助完成如下几项工作：

*   快速分析 Apk 构成，包括 DEX、Resources 和 Manifest 的 Size 和占比，助力我们优化代码或资源的方向
    
*   Diff Apk 以了解版本的前后差异，精准定位体积变大的源头
    
*   分析其他 Apk，包括查看大致的资源和分析代码逻辑，进而拆解、Bug 定位
    

### 3.4 DI Navigation

依赖注入有助于模块间的解耦，践行了关注点分离的设计原则。我们使用 Dagger / Hilt 通过编译期代码生成隐藏了相关具体实现，这在降低构建依赖关系图的成本的同时，也增加了开发者调试代码的成本：寻找被注入实例的来源变得困难起来。

如今 Android Studio 帮开发者解决了这个痛点。自 4.1 我们可以在基于 Dagger 的代码（例如 Components，Subcomponents，Modules 等）中跳转，找寻依赖关系。

![](https://mmbiz.qpic.cn/mmbiz_gif/5EcwYhllQOjU6ictVsS5JBlicic9vewgea1ByCYJNLsg9vaDWqT4HsxOetzia2oqI0KLdyFZqnu2mHxWpYPhjXyzQA/640?wx_fmt=gif)

在 Dagger 或 Hilt 相关的代码旁可以看到下面的 icon：

![](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOjU6ictVsS5JBlicic9vewgea1NSkgmk2VjLKqnd8uHiaIgeDaGl0BQz8UScUNZ7qKmDV7Z0ufSZOCPBw/640?wx_fmt=png)

点击左侧 icon 可以跳转到实例对象的提供处，点击右侧 icon 则可以跳转到对象的使用处，当有多处使用时则会给出候选列表供选择。

Android 4.2 起还增加了对 `@EnterPoint` 的依赖查询，对于 ContentProvider 这样的不能自动注入的组件，也可以通过 Hilt 扩大依赖注入的使用范围。

4\. App Bundle
--------------

![](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOjU6ictVsS5JBlicic9vewgea1SU8W49ssEm60QMdaDOzPAoQmJib8aOX4iap0ib3qWoQBFChRp9vQt1qDQ/640?wx_fmt=png)

Android App Bundle 是 Google 推出的用于动态化分发的打包格式。当应用程序以 AAB 的格式上传 Google Play（或其他支持 AAB 的应用市场）后，可以根据需要实现功能或资源的动态下发。

![](https://mmbiz.qpic.cn/mmbiz_gif/5EcwYhllQOjU6ictVsS5JBlicic9vewgea1CLJp42Cs3JiaYSKpWN54zVLFIw7V7kkJc6Ic0lV1nfZUH9EhQc4t2NQ/640?wx_fmt=gif)

Split APKs 机制是 AAB 实现动态下发的基础，AAB 上传 GP 后被拆分成一个 base APK 和多个 Split APKs。首次下载只下发 Base APK，然后根据使用场景动态下发 Split APKs。Split 可以使 Configuration APKs ，也可以是一个 Dynamic Features APKs：  

*   **Configuration APKs**：根据 language，density，abi 三个维度拆分资源，比如 res/drawable-xhdpi 会被拆分到 xhdpi 的 Apk 中，res/values-en 会被拆分到 en 的 apk 中，当 Configurations Changed 发生时请求必要资源
    
*   **Dynamic Features APKs**：可以实现 Feature 的按需动态加载，这类似于国内流行的“插件化”技术，通过将一些非常用的功能做成 Dynamic Feature 可以实现功能的按需加载。
    

Google 重视 AAB 格式的推广，自 21 年 8 月起，规定新 App 必须使用 AAB 格式才能在 Google Play 上架。作一款要在海外上架的产品，我们自然也选择了 AAB 的交付方式，除了在包体积方面的显著受益，也较好地助力了产品推广和装机率的提升。

![](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOjU6ictVsS5JBlicic9vewgea19h8Ika3uibK4EZoHjg5CaDlYSSmjbPP8J7MjjQiceb8TyibGOCOG5SPdw/640?wx_fmt=png)

### 4.1 Language Split

我们的应用在多个国家同时上架，需要支持英语、印尼语、葡语等多种语言，借助 AAB 可以避免下载其他国家的语言资源。

语言动态下发非常简单，首先在 Gradle 开启 language 的 `enableSplit`：

`bundle {  
    language {  
        enableSplit = true  
    }  
}  
`

切换系统语言时，应用会通过 GP 自动下载所需的语言。当然也可以根据业务需求手动请求语言资源，比如在我们内置的语言切换界面中选择其他语言时：

`private val _splitListener = SplitInstallStateUpdatedListener { state ->  
    val lang = if (state.languages().isNotEmpty())  
                state.languages().first() else ""  
    when (state.status()) {  
        SplitInstallSessionStatus.INSTALLED -> {  
            //...  
        }  
        SplitInstallSessionStatus.FAILED -> {  
            //...  
        }  
        else -> {}  
    }  
}

//创建SplitManager  并注册回调  
val splitManager = SplitInstallManagerFactory.create(requireContext())  
splitManager.registerListener(_splitListener)

//安装语言资源  
val request = SplitInstallRequest.newBuilder()  
    .addLanguage(Locale.forLanguageTag(language))  
    .build()  
splitManager.startInstall(request);

`

### 4.2 Dynamic Feature

产品中有一些高级功能，并非所有用户都会用到，比如某些高级相机特效，却依赖了比较多的 so 以及底层库，将它们做成 Dynamic Feature 实现功能的按需加载：

创建 Dynamic Feature 就如同创建一个 Gradle Module。

![](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOjU6ictVsS5JBlicic9vewgea1CYRqyT0o648IFoPd2oBaVWuQo79icfOeoLxYjjwutfbcsT2llU7s1Mg/640?wx_fmt=png)

DF 创建时可以配置两种下载方式：

*   **on-demand**：是否走动态下发，如果勾选，表示根据用户请求去动态下载，否则用户安装 Apk 时 Module 就会被安装
    
*   **fusing**：此配置主要是为了兼容 5.0 以下不支持 AAB 的情况，如果勾选，在 5.0 以下设备会直接安装 Module，否则，5.0 以下设备不包含此 Module
    

DF 创建后会在 app/build.gradle 中添加响应注册：

`dynamicFeatures = [':dynamicfeature']  
`

在需要的场景请求 Dynamic Feature，与请求语言的代码类似，都是使用 `SplitInstallManager`：

`val splitManager = SplitInstallManagerFactory.create(requireContext())

//动态安装模块  
SplitInstallRequest request =  
    SplitInstallRequest  
        .newBuilder()  
        .addModule("FaceLab")  
        .addModule("Avator")  
        .build();

splitManager  
    .startInstall(request)  
    .addOnSuccessListener { sessionId -> ... }  
    .addOnFailureListener { exception -> ... }

`

### 4.3 Bundletool

AAB 格式没法在本地安装和调试，通过 Google 提供的 AAB > APK 的打包工具，我们可以在本地编译成 APK ，便于 QA 的测试和开发人员的自测。

AAB 生成 APK 的过程如下，中间会生成 .apks ，然后再针对不同设备生成具体 .apk。

![](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOjU6ictVsS5JBlicic9vewgea1OWoGA0mJbFjwWxq2mbj0lGp8bBRa4WY9mj7SOVpJhnDR7CnMhkSt7w/640?wx_fmt=png)

`// 通过 aab 生成 apks 文件  
bundletool build-apks  
--bundle=/MyApp/my_app.aab  
--output=/MyApp/my_app.apks  
--ks=/MyApp/keystore.jks  
--ks-pass=file:/MyApp/keystore.pwd  
--ks-key-alias=MyKeyAlias  
--key-pass=file:/MyApp/key.pwd  
--device-spec=file:device-spec.json  
`

通过 device.json 生成本地 Apk：

`bundletool extract-apks  
--apks=${apksPath}  
--device-spec={deviceSpecJsonPath}  
--output-dir={outputDirPath}  
`

也可以直接通过 apks 进行安装，此时实际上是安装 apk 到手机上，只是该命令会自动读取手机配置，然后先生成相应的 apk，再安装到手机。

`bundletool install-apks  
--apks=/MyApp/my_app.apks  
`

最终的安装包通过语言等资源以及 Dynamic Feature 的动态下发，包体积减小近 40%，从 90M+ 压缩到 55M。

5\. ML Kit
----------

![](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOjU6ictVsS5JBlicic9vewgea1mzMCtsibSO1Gm03gfU2IliaGAE6F8bSIPN6XCbWHqfvp4AaOotSGbnYg/640?wx_fmt=png)

除了 Jetpack 的相关类库， Google 还为我们的应用提供了不少其他技术支持，比如 ML Kit 。ML Kit 是 Google 推出的针对移动端的一款移动 SDK，支持 Android 与 iOS 平台，封装了文字识别、人脸位置检测、对象跟踪及检测等诸多机器学习能力，对于机器学习开发者，ML Kit 也同样提供了 API 帮助开发者自定义 TensorFlow lite 模型。ML Kit 也支持 Google Play 运行时下发，以减少包体积。

作为一款 AI 特效应用，需要支持用户选择多人脸图片中的某个人脸进行渲染，因此人脸检测能力必不可少，经过调研，我们选择了 ML Kit 来实现快速人脸检测。

![](https://mmbiz.qpic.cn/mmbiz_gif/5EcwYhllQOjU6ictVsS5JBlicic9vewgea1kCa8ysqibfohiajxZZwLGhjUbAtBiaO1ZYPRfYsl7NiaQAXwWTB55HlpAw/640?wx_fmt=gif)

ML Kit 将几种机器学习能力进行了拆分，App 只需引入需要的能力即可。以人脸检测为例，引入人脸检测 Google Play 动态下发库，并使用挂起函数简化 API 的使用：

`dependencies {  
    implementation 'com.google.android.gms:play-services-mlkit-face-detection:17.0.0'  
}  
`

在 AndroidManifest.xml 文件中进行配置：

`<application ...>  
        ...  
    <meta-data  
        android:name="com.google.mlkit.vision.DEPENDENCIES"  
        android:value="face" />  
</application>  
`

使用协程提供的`suspendCancellableCoroutine`API 将回调改造成挂起函数

`suspend fun faceDetect(input: Bitmap): List<Face> = suspendCancellableCoroutine { continuation ->  
    val image = InputImage.fromBitmap(bitmap, 0)  
    val detector = FaceDetection.getClient()  
    detector.process(image)  
        .addOnSuccessListener {  
            continuation.resumeWith(Result.success(it))  
        }  
        .addOnFailureListener {  
            continuation.resumeWithException(RuntimeException(it))  
        }  
        .addOnCanceledListener {  
            continuation.cancel()  
        }  
}  
`

最后
--

MAD 帮助我们完成了产品的高效开发和快速上架，未来我们还会引入 Jetpack Compose 来进一步提升开发效率，缩短需求迭代周期。受限于篇幅，文中内容只是点到为止，希望能够为其他同类的出海应用在技术选型上提供启发和参考。

随着 Jetpack 为代表的 Google 移动开发生态的不断完善，开发者们可以将更多精力聚焦到业务创新，为广大用户开发出更多丰富的功能。底层技术的不断统一，也有利于开发者们更好地展开技术交流和共建，摆脱各自为战、重复造轮子的开发窘境。