# 为什么要使用Jetpack Compose

`Andriod Studio`新建空白项目时已经默认使用`Jetpack Compose`，显然是官方强烈推荐，具体的好处自己百度

对我来说，摆脱xml就是最大的好处

# 新建一个Compose项目

`Andriod Studio`中依次选择`File`-`New`-`New Project`，模板选择`Empty Activity`，下一步，界面会提示将使用`Jetpack Compose`创建一个空的`activity`，这里构建配置语言我们选`Grovvy`，填写其他信息，`finish`



注：本文使用的`compose-bom`的版本为`2024.12.01`，创建好项目后可在`libs.versions.toml`文件中修改

# 加速依赖导入（可选）

项目创建好之后，依次选择`View`-`Tool Windows`-`Build`（或左下角Build按钮），打开构建工具窗口，先把同步操作停掉

配置阿里云依赖和插件仓库，打开项目的`settings.gradle`配置文件，将其内容修改为：

```groovy
pluginManagement {
    repositories {
        mavenLocal()
        maven { url 'https://www.jitpack.io' }
        maven { url 'https://maven.aliyun.com/repository/gradle-plugin' }
        maven { url 'https://maven.aliyun.com/repository/spring-plugin' }
        maven { url 'https://maven.aliyun.com/repository/public' }
        maven { url 'https://maven.aliyun.com/repository/google' }
        maven { url 'https://maven.aliyun.com/repository/jcenter' }
        google {
            content {
                includeGroupByRegex("com\\.android.*")
                includeGroupByRegex("com\\.google.*")
                includeGroupByRegex("androidx.*")
            }
        }
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        mavenLocal()
        maven { url 'https://www.jitpack.io' }
        maven { url 'https://maven.aliyun.com/repository/gradle-plugin' }
        maven { url 'https://maven.aliyun.com/repository/google' }
        maven { url 'https://maven.aliyun.com/repository/public' }
        maven { url 'https://maven.aliyun.com/repository/jcenter' }
        google()
        mavenCentral()
    }
}

rootProject.name = "你的项目名"
include ':app'

```

修改gradle包装器下载地址为国内地址，打开`gradle/wrapper/gradle-wrapper.properties`，找到`distributionUrl`项，将其网址前缀（除文件名部分）修改为

```
https\://mirrors.cloud.tencent.com/gradle/
```

修改完毕后形如

```
distributionUrl=https\://mirrors.cloud.tencent.com/gradle/gradle-8.9-bin.zip
```

执行同步

# 官方教程

[官方教程](https://developer.android.google.cn/jetpack/compose/tutorial)

强烈推荐先跟着做一个示例项目，体会一下`可组合函数`的概念，即：函数即组件



我的个人理解：执行一个可组合函数 = 在界面上渲染了一个组件

# APP首页制作

参考资料：

- [分页器](https://developer.android.com/develop/ui/compose/layouts/pager?hl=zh-cn#horizontalpager)
- [底部导航栏](https://developer.android.google.cn/reference/kotlin/androidx/compose/material3/package-summary#navigationbar)

首先制作一个非常常见的首页框架：底部有导航栏按钮，点击导航栏按钮切换上方的页面，上方页面也可以左右滑动翻页。新建一个文件命名为`HorizontalPagerIndex.kt`，代码如下：

```kotlin

/**
 * 页面状态
 * @param name 页面名称（底部导航按钮名称）
 * @param icon 底部导航栏icon
 * @param content 页面正文内容
 * @constructor
 */
data class PageState(val name: String, val icon: @Composable () -> Unit, val content: @Composable () -> Unit)

/**
 * 带底部导航栏的横向翻页首页
 * @param states 页面状态
 * @param initialIndex 初始页位置
 */
@Composable
fun HorizontalPagerIndex(states: List<PageState>, initialIndex: Int = 1) {
    // 选中的index
    var selectedIndex by remember { mutableIntStateOf(initialIndex) }
    // 分页器状态
    val pagerState = rememberPagerState(pageCount = { states.size }, initialPage = initialIndex)
    val coroutineScope = rememberCoroutineScope()
    // 监控当前页的变化，切换页面时同步修改  selectedIndex 的值，无论是通过手势操作还是点击按钮均会触发
    LaunchedEffect(pagerState) { snapshotFlow { pagerState.currentPage }.collect { page -> selectedIndex = page } }
    // 界面
    Scaffold(
        modifier = Modifier.fillMaxSize(),
        // 底部导航栏
        bottomBar = {
            NavigationBar {
                states.forEachIndexed { index, state ->
                    NavigationBarItem(
                        // 按钮名称
                        label = { Text(state.name) },
                        // 按钮是否已选中，根据按钮序号与 selectedIndex 是否一致来决定
                        selected = selectedIndex == index,
                        // 点击按钮时的操作，通过 pagerState 切换页面
                        onClick = { coroutineScope.launch { pagerState.scrollToPage(index) } },
                        // 按钮的icon
                        icon = state.icon,
                    )
                }
            }
        }
    ) { innerPadding ->
        // 横向分页器
        HorizontalPager(
            state = pagerState,
             modifier = Modifier
                .padding(innerPadding)
                .fillMaxHeight(),
            verticalAlignment = Alignment.Top
        ) { 
            // 根据当前页数，展示对应页的正文内容
            page -> states[page].content()
        }
    }
}
```

使用，假设我们在制作一个社区论坛APP，正文部分先简单放一个文本：

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            GF2GameCommunityTheme {
                val states = listOf(
                    PageState("日常", { Icon(Icons.Rounded.DateRange, contentDescription = null) }, { DateTimeText("日常") }),
                    PageState("社区", { Icon(Icons.Rounded.Home, contentDescription = null) }, { DateTimeText("社区") }),
                    PageState("设置", { Icon(Icons.Rounded.Settings, contentDescription = null) }, { DateTimeText("设置") }),
                )
                HorizontalPagerIndex(states)
            }
        }
    }
}

@Composable
fun DateTimeText(text: String) = Text("$text: ${ZonedDateTime.now().toLocalDateTime()}")
```

# Navigation

## 综述

参考资料：[导航](https://developer.android.google.cn/guide/navigation?hl=zh-cn#set-up)



Navigation是一种替代原生`Activity`+`Fragment`的架构形式，官方推荐一个APP原则上只使用一个`Activity`



本节来自对[官方教程](https://developer.android.google.cn/guide/navigation?hl=zh-cn#set-up)的总结和理解，也继承他其中的概念：

- 宿主`NavHost`：屏幕上的一块区域，导航目的地页面将在这块区域中展示
- 导航控制器`NavController`：用来执行导航操作
- 目的地`Destination`：导航操作需要呈现的页面（由可组合函数生成）
- 路线：唯一标识目的地及其所需的任何数据；可以当做是目的地名称+参数

如果你使用过`vue-router`，会更容易理解这些概念：

- 宿主 = `<router-view/>`
- 导航控制器 = `router.push(***)`中的`router`
- 目的地 = 路由配置中的组件
- 路线 = 路由配置中的地址，不过这里路线可以是一个自定义对象，而不只是字符串



Navigation执行导航时会接管系统的`后退`操作（包括物理按键和手势），这也和`vue-router`很像，我们每次执行导航操作相当于新建了一个`Fragment`，后退时则回到上一个`Fragment`

## 导入依赖和插件

来自官方教程，我们使用compose架构所以只需要导入：

```groovy
plugins {
  id 'org.jetbrains.kotlin.plugin.serialization' version '2.0.21'
}
dependencies {
  implementation "androidx.navigation:navigation-compose:2.8.4"
  implementation "org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.3"
}
```

最新版本见 [官网](https://developer.android.google.cn/jetpack/androidx/releases/navigation?hl=zh-cn)

## 创建控制器和宿主

原则上我们应当在较高的层级创建他们，比如`Activity`中，或者其`setContent`方法中的根组件里



继续刚才的社区论坛APP例子，现在我们的`MainActivity`长这样：

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            GF2GameCommunityTheme {
                val navController = rememberNavController()
                NavHost(
                    navController = navController,
                    startDestination = "起始路线",
                ) {
                   TODO("路线和目的地配置")
                }

            }
        }
    }
}
```



## 路线和目的地

参考资料：[导航图概览](https://developer.android.google.cn/guide/navigation/design?hl=zh-cn)

- 官方推荐我们使用可序列化的(`@Serializable`)对象或者数据类作为路线，这里以数据类举例

- 作为数据类很自然地，它可以配置字段是否可null，以及默认值，并可以使用copy方法进行克隆，比起网上旧教程中querystring的传参方式要好用多了
- 如果你使用过实现了`Parcelable`接口的数据类向`Activity`或`Fragment`传参，会发现这两种做法非常类似

继续刚才的社区论坛APP例子，现在我们的`MainActivity`长这样：

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            GF2GameCommunityTheme {
                val navController = rememberNavController()
                NavHost(
                    navController = navController,
                    startDestination = IndexRoute(),
                ) {
                    // 首页路线
                    composable<IndexRoute> { IndexComposable(it.toRoute()) }
                }

            }
        }
    }
}

@Composable
fun DateTimeText(text: String) = Text("$text: ${ZonedDateTime.now().toLocalDateTime()}")

/**
 * 首页路线
 * @param initialIndex 导航栏的初始位置
 * @constructor
 */
@Serializable
data class IndexRoute(val initialIndex: Int = 1)
/**
 * 首页组件
 * @param route IndexRoute 路线
 */
@Composable
fun IndexComposable(route: IndexRoute = IndexRoute()) {
    // 路由配置
    val states = listOf(
        PageState("日常", { Icon(Icons.Rounded.DateRange, contentDescription = null) }, { DateTimeText("日常") }),
        PageState("社区", { Icon(Icons.Rounded.Home, contentDescription = null) }, { DateTimeText("社区") }),
        PageState("设置", { Icon(Icons.Rounded.Settings, contentDescription = null) }, { DateTimeText("设置") }),
    )
    HorizontalPagerIndex(states, route.initialIndex)
}
```

## 执行导航操作

参考资料：[导航图概览](https://developer.android.google.cn/guide/navigation/design?hl=zh-cn#compose-minimal)

- 执行导航操作需要调用`navController`的`navigate`方法

- 官方不主张我们将`navController`传入到可组合函数中，而是向其传入一个`onNavigateTo*****`的回调方法，可组合函数只要在需要时调用传入的方法即可



继续刚才的社区论坛APP例子，根据上述思想作以下修改：

1. 新建一个`主题列表路线`（数据类）和一个`主题列表组件`(可组合方法)，参照上一节的方法在`NavHost`中注册它们，并顺带给它设置一个进入和退出的动画
2. 为`IndexComposable`方法增加一个回调方法参数`onNavigateToTopicList`，将`社区`页正文修改为一个按钮，点击按钮时调用这个回调方法
3. 修改`NavHost`中的首页路线配置，传入回调方法的实现，即调用`navController`的`navigate`方法导航到`主题列表路线`

现在我们的`MainActivity`长这样：

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            GF2GameCommunityTheme {
                val navController = rememberNavController()
                NavHost(
                    navController = navController,
                    startDestination = IndexRoute(),
                ) {
                    // 首页路线
                    composable<IndexRoute> {
                        IndexComposable(it.toRoute(), onNavigateToTopicList = { route: TopicListRoute -> navController.navigate(route) })
                    }
                    // 主题列表路线
                    composable<TopicListRoute>(
                        // 从屏幕右侧进入，持续500毫秒
                        enterTransition = { slideInHorizontally(tween(500)) { it } },
                        // 从屏幕右侧退出，持续500毫秒
                        exitTransition = { slideOutHorizontally(tween(500)) { it } }
                    ) { TopicListComposable(it.toRoute()) }
                }

            }
        }
    }
}

@Composable
fun DateTimeText(text: String) = Text("$text: ${ZonedDateTime.now().toLocalDateTime()}")

/**
 * 首页路线
 * @param initialIndex 导航栏的初始位置
 * @constructor
 */
@Serializable
data class IndexRoute(val initialIndex: Int = 1)

/**
 * 首页组件
 * @param route IndexRoute 路线
 * @param onNavigateToTopicList 导航到主题列表
 */
@Composable
fun IndexComposable(route: IndexRoute = IndexRoute(), onNavigateToTopicList: (route: TopicListRoute) -> Unit) {
    // 路由配置
    val states = listOf(
        PageState("日常", { Icon(Icons.Rounded.DateRange, contentDescription = null) }, { DateTimeText("日常") }),
        PageState("社区", { Icon(Icons.Rounded.Home, contentDescription = null) }, {
            Button(onClick = { onNavigateToTopicList(TopicListRoute(1)) }) { Text("打开主题列表") }
        }),
        PageState("设置", { Icon(Icons.Rounded.Settings, contentDescription = null) }, { DateTimeText("设置") }),
    )
    HorizontalPagerIndex(states, route.initialIndex)
}

/**
 * 主题列表路线
 * @param categoryId 分类ID
 * @constructor
 */
@Serializable
data class TopicListRoute(val categoryId: Int)

/**
 * 主题路线组件
 * @param route 路线
 */
@Composable
fun TopicListComposable(route: TopicListRoute) {
    Scaffold { innerPadding ->
        Column(modifier = Modifier.padding(innerPadding)) {
            DateTimeText("主题列表 - ${route.categoryId}")
        }
    }
}
```

## 封装导航代码

参考资料：[封装代码](https://developer.android.google.cn/guide/navigation/design/encapsulate?hl=zh-cn)

显然我们已经发现`MainActivity`变得有点臃肿了，所以官方建议我们把`路线`和`组件`，甚至`NavHost`中的配置也使用扩展方法移到单独的文件中去，所以现在：



`Index.kt`

```kotlin
/**
 * 扩展方法
 * @receiver [NavGraphBuilder]
 * @param onNavigateToTopicList 导航到主题列表
 */
fun NavGraphBuilder.index(onNavigateToTopicList: (route: TopicListRoute) -> Unit) = composable<IndexRoute> { IndexComposable(
    route = it.toRoute(),
    onNavigateToTopicList = onNavigateToTopicList
) }

/**
 * 首页路线
 * @param initialIndex 导航栏的初始位置
 * @constructor
 */
@Serializable
data class IndexRoute(val initialIndex: Int = 1)

/**
 * 首页组件
 * @param route IndexRoute 路线
 * @param onNavigateToTopicList 导航到主题列表
 */
@Composable
fun IndexComposable(route: IndexRoute = IndexRoute(), onNavigateToTopicList: (route: TopicListRoute) -> Unit) {
    // 路由配置
    val states = listOf(
        PageState("日常", { Icon(Icons.Rounded.DateRange, contentDescription = null) }, { DateTimeText("日常") }),
        PageState("社区", { Icon(Icons.Rounded.Home, contentDescription = null) }, {
            Button(onClick = { onNavigateToTopicList(TopicListRoute(1)) }) { Text("打开主题列表") }
        }),
        PageState("设置", { Icon(Icons.Rounded.Settings, contentDescription = null) }, { DateTimeText("设置") }),
    )
    HorizontalPagerIndex(states, route.initialIndex)
}
```

`TopicList.kt`

```kotlin
/**
 * 扩展方法
 * @receiver [NavGraphBuilder]
 */
fun NavGraphBuilder.topicList() = composable<TopicListRoute>(
    // 从屏幕右侧进入，持续500毫秒
    enterTransition = { slideInHorizontally(tween(500)) { it } },
    // 从屏幕右侧退出，持续500毫秒
    exitTransition = { slideOutHorizontally(tween(500)) { it } })
{ TopicListComposable(route = it.toRoute()) }

/**
 * 主题列表路线
 * @param categoryId 分类ID
 * @constructor
 */
@Serializable
data class TopicListRoute(val categoryId: Int)

/**
 * 主题路线组件
 * @param route 路线
 */
@Composable
fun TopicListComposable(route: TopicListRoute) {
    Scaffold { innerPadding ->
        Column(modifier = Modifier.padding(innerPadding)) {
            DateTimeText("主题列表 - ${route.categoryId}")
        }
    }
}
```

`MainActivity.kt`

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            GF2GameCommunityTheme {
                val navController = rememberNavController()
                NavHost(
                    navController = navController,
                    startDestination = IndexRoute(),
                ) {
                    // 首页路线
                    index(onNavigateToTopicList = { navController.navigate(it) })
                    // 主题列表路线
                    topicList()
                }

            }
        }
    }
}

@Composable
fun DateTimeText(text: String) = Text("$text: ${ZonedDateTime.now().toLocalDateTime()}")
```

# ViewModel和LiveData

之前我们用到了`DateTimeText`这个自定义可组合函数，我们会发现每次打开一个用到它的页面，上面的时间都会更新，也就是说每次我们“看到”它的时候这个函数都被重新执行了一遍，而不是像`ViewPager2`那样会缓存已经加载过的页面，那么问题来了，如果我要显示的东西是通过**网络请求**拿到的，那岂不是每次都要重新请求？



所以这就该用到`ViewModel`和`LiveData`了，参考资料：[ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel?hl=zh_cn)



另外扩展阅读：[可组合项的生命周期](https://developer.android.com/develop/ui/compose/lifecycle?hl=zh-cn)

## 简介

`ViewModel`和`LiveData`通常会搭配使用来封装：业务逻辑、业务数据、状态数据等；而组件只负责通过`绑定`的形式显示数据，绑定的数据有变化时界面会自动更新（又是和`vue`相似的机制）；由于`ViewModel`生命周期长于界面组件，通过网络请求拿到的数据可以缓存在其中。

标准用法为：

1. 通过自定义类继承`ViewModel`，在其中编写业务逻辑
2. 在`ViewModel`中定义`LiveData`字段，用来保存业务数据和状态数据
3. 在需要显示数据的组件外部实例化自定义的`ViewModel`并将其传入
4. 在需要显示数据的组件内部将需要显示数据的`LiveData`中的数据绑定到需要的位置

## 代码示例

继续刚才的社区论坛APP例子

新建一个类`CommunityIndexViewModel`继承`ViewModel`

- `api`字段是我使用`retrofit`封装的一个请求接口
- `obtainBanners`方法用于请求和更新数据

```kotlin
class CommunityIndexViewModel : ViewModel() {
    private val api = App.INSTANCE.api.indexApi
    private var loadingBanners = false

    val banners = MutableLiveData<List<Banner>>()

    fun obtainBanners(force: Boolean = false) {
        if (loadingBanners) return
        // 如果强制更新 或 数据为空，执行
        if (force || banners.value.isNullOrEmpty()) {
            loadingBanners = true
            // 执行请求
            api.getBanner().enqueue(object : Callback<Res<Banner.Body>?> {
                override fun onResponse(call: Call<Res<Banner.Body>?>, response: Response<Res<Banner.Body>?>) {
                    banners.value = response.body()?.data?.bannerList
                    loadingBanners = false
                }

                override fun onFailure(call: Call<Res<Banner.Body>?>, throwable: Throwable) {
                    loadingBanners = false
                    throwable.printStackTrace()
                    TODO("Not yet implemented")
                }
            })
        }
    }
}
```

新建可组合函数`CommunityIndex`，接收`CommunityIndexViewModel`作为参数，在其中执行请求并绑定`banners`列表的长度到一个文本框中；然后将`IndexComposable`函数的`社区`页正文修改为`CommunityIndex`

```kotlin
/**
 * 社区首页
 * @param viewModel [CommunityIndexViewModel]
 * @param onNavigateToTopicList 导航到主题列表
 */
@Composable
fun CommunityIndex(viewModel: CommunityIndexViewModel, onNavigateToTopicList: (route: TopicListRoute) -> Unit) {
    viewModel.obtainBanners()
    val bannerState = viewModel.banners.observeAsState()
    Text("banner: ${bannerState.value?.size?:0}")
}

/**
 * 首页组件
 * @param route IndexRoute 路线
 * @param onNavigateToTopicList 导航到主题列表
 */
@Composable
fun IndexComposable(route: IndexRoute = IndexRoute(), onNavigateToTopicList: (route: TopicListRoute) -> Unit) {
    // 社区首页ViewModel
    val communityIndexViewModel = viewModel<CommunityIndexViewModel>()

    // 路由配置
    val states = listOf(
        PageState("日常", { Icon(Icons.Rounded.DateRange, contentDescription = null) }, { DateTimeText("日常") }),
        PageState("社区", { Icon(Icons.Rounded.Home, contentDescription = null) }, { CommunityIndex(communityIndexViewModel, onNavigateToTopicList) }),
        PageState("设置", { Icon(Icons.Rounded.Settings, contentDescription = null) }, { DateTimeText("设置") }),
    )
    HorizontalPagerIndex(states, route.initialIndex)
}

```

运行APP即可看到文本框中的数字先是0，请求完成后则变为对应数字

# 图片加载

官方推荐使用`Coil`库来加载图片：[Coil文档](https://github.com/coil-kt/coil/blob/main/README-zh.md)

## 导入依赖和官方示例

```groovy
implementation("io.coil-kt.coil3:coil-compose:3.0.4")
implementation("io.coil-kt.coil3:coil-network-okhttp:3.0.4")
```

简单使用

```kotlin
AsyncImage(
    model = "https://example.com/image.jpg",
    contentDescription = null,
)
```

## 代码示例

继续刚才的社区论坛APP例子，既然我们请求到了一个banner列表，自然需要显示对应的图片，简单套用官方示例结合之前使用过的`HorizontalPager`即可：

```kotlin
/**
 * 社区首页
 * @param viewModel [CommunityIndexViewModel]
 * @param onNavigateToTopicList 导航到主题列表
 */
@Composable
fun CommunityIndex(viewModel: CommunityIndexViewModel, onNavigateToTopicList: (route: TopicListRoute) -> Unit) {
    viewModel.obtainBanners()
    val bannerState = viewModel.banners.observeAsState()
    Banner(bannerState.value)
}


@Composable
fun Banner(banners: List<Banner>?) {
    val data = banners?.takeIf { it.isNotEmpty() } ?: return
    val pagerState = rememberPagerState(pageCount = { data.size })
    HorizontalPager(state = pagerState, modifier = Modifier.fillMaxWidth()) { page ->
        AsyncImage(
            model = banners[page].imgAddr,
            contentDescription = null,
            modifier = Modifier.fillMaxWidth(),
        )
    }
}
```

不过看到这个效果，自然我们会想到这种图片常规来说应该是轮播图才对：图片会自动翻页，有小点或色块显示当前为第几页，点击还能跳转到其他页面

## 轮播图

参考资料：

- [官方组件](https://juejin.cn/post/7396252674239610916)
- [自己组合](https://blog.csdn.net/u011897062/article/details/131032644)

说明：

- 官方组件，需要比较新的`material3`版本，目前`compose-bom`的版本为`2024.12.01`，已支持该功能， 因为是近期添加的组件会有`实验性`的注解。

- 官方的两种组件我试用了一下均不是我想要的功能：没有页数指示器，也没有自动翻页；和`HorizontalPager`比没有显著区别

所以我们还是来自己做一个：

1. 把它做成通用组件方便复用，传入的数据对象使用泛型，对图片的渲染方法也通过外部传入
2. 可以修改页数指示器的样式，包括：
   - 形状（默认圆形）
   - 大小及相互间距离（默认8dp）
   - 在两种状态下的颜色（默认蓝色和灰色）
   - 位于轮播图的内部还是外部，及距离底边的距离
3. 可以选择是否自动滚动，且可以设置滚动间隔



定义：

```kotlin
/**
 * 横向轮播图，带页数指示器及自动滚动
 * @param T 数据类型
 * @param list 数据列表
 * @param selectedColor 页数指示器：选中时的颜色
 * @param unselectedColor 页数指示器：未选中时的颜色
 * @param spotSize 页数指示器：大小
 * @param spotSpacing 页数指示器：间隔
 * @param spotsInside 页数指示器：在轮播图内还是外
 * @param spotsRange 页数指示器：距离轮播图底边的距离
 * @param spotShape 页数指示器：形状
 * @param autoScrollDuration 自动滚动的间隔，设置为0则不滚动
 * @param item 渲染单个轮播图的方法
 */
@Composable
fun <T> HorizontalCarousel(
    list: List<T>?,
    selectedColor: Color = Color.Blue,
    unselectedColor: Color = Color.Gray,
    spotSize: Dp = 10.dp,
    spotSpacing: Dp = 8.dp,
    spotsInside: Boolean = true,
    spotsRange: Dp = 8.dp,
    spotShape: Shape = CircleShape,
    autoScrollDuration: Long = 5000L,
    item: @Composable (T) -> Unit
) {
    val data = list?.takeIf { it.isNotEmpty() } ?: return
    val pagerState = rememberPagerState(pageCount = { data.size })

    // 轮播图
    @Composable
    fun Pager() = HorizontalPager(state = pagerState, modifier = Modifier.fillMaxWidth(), pageSpacing = 8.dp) { page -> item(data[page]) }

    // 页数指示器
    @Composable
    fun Spots() {
        repeat(data.size) {
            Box(Modifier
                .clip(spotShape)
                .background(if (it == pagerState.currentPage) selectedColor else unselectedColor)
                .width(spotSize)
                .height(spotSize))
            Spacer(Modifier.width(spotSpacing))
        }
    }

    // 自动滚动
    if (autoScrollDuration > 0) {
        val isDragged by pagerState.interactionSource.collectIsDraggedAsState()
        if (isDragged.not()) {
            with(pagerState) {
                var currentPageKey by remember { mutableIntStateOf(0) }
                LaunchedEffect(key1 = currentPageKey) {
                    launch {
                        delay(timeMillis = autoScrollDuration)
                        val nextPage = (currentPage + 1).mod(pageCount)
                        animateScrollToPage(page = nextPage)
                        currentPageKey = nextPage
                    }
                }
            }
        }
    }

    // 根据指示器是否在图片内决定布局
    if (spotsInside) {
        Box {
            Pager()
            Row(Modifier
                .align(Alignment.BottomCenter)
                .padding(spotsRange)) { Spots() }
        }
    } else {
        Column {
            Pager()
            Spacer(Modifier.height(spotsRange))
            Row(modifier = Modifier.fillMaxWidth(), horizontalArrangement = Arrangement.Center) { Spots() }
        }
    }
}
```

使用：

```kotlin
/**
 * 社区首页
 * @param viewModel [CommunityIndexViewModel]
 * @param onNavigateToTopicList 导航到主题列表
 */
@Composable
fun CommunityIndex(viewModel: CommunityIndexViewModel, onNavigateToTopicList: (route: TopicListRoute) -> Unit) {
    viewModel.obtainBanners()
    val bannerState = viewModel.banners.observeAsState()

    HorizontalCarousel(bannerState.value) {
        AsyncImage(model = it.imgAddr, contentDescription = null, modifier = Modifier.fillMaxWidth())
    }
}
```

# 顶部应用栏

参考资料：[应用栏](https://developer.android.com/develop/ui/compose/components/app-bars?hl=zh-cn)

## 首页

由于是首页的应用栏，暂不涉及复用问题，我们直接在原本的`HorizontalPagerIndex`函数中修改，在`Scaffold`函数中增加参数`topBar`，直接抄官方示例的居中对齐样式，把标题改成APP名字，略微改动一下按钮icon：

```kotlin
topBar = {
    CenterAlignedTopAppBar(
        colors = TopAppBarDefaults.centerAlignedTopAppBarColors(
            containerColor = MaterialTheme.colorScheme.primaryContainer,
            titleContentColor = MaterialTheme.colorScheme.primary,
        ),
        title = { Text(text = stringResource(R.string.app_name)) },
        actions = {
            IconButton(onClick = { println("点击菜单按钮") }) {
                Icon(imageVector = Icons.Filled.Menu, contentDescription = "菜单按钮")
            }
        },
        navigationIcon = {
            IconButton(onClick = { println("用户按钮") }) {
                Icon(imageVector = Icons.Filled.Person, contentDescription = "用户按钮")
            }
        },
    )
},
```

## 主题列表页

主题列表页的应用栏我们选择常规模式，并需要在标题栏显示当前所属主题分类名，导航按钮则选择左箭头，并实现后退功能

```kotlin
/**
 * 扩展方法
 * @receiver [NavGraphBuilder]
 */
fun NavGraphBuilder.topicList(onPopBackStack: () -> Unit) = composable<TopicListRoute>(
    // 从屏幕右侧进入，持续500毫秒
    enterTransition = { slideInHorizontally(tween(500)) { it } },
    // 从屏幕右侧退出，持续500毫秒
    exitTransition = { slideOutHorizontally(tween(500)) { it } }) {
    TopicListComposable(route = it.toRoute(), onPopBackStack = onPopBackStack)
}


@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun TopicListComposable(route: TopicListRoute, onPopBackStack: () -> Unit) {
    // 社区首页ViewModel
    val communityIndexViewModel = viewModel<CommunityIndexViewModel>()
    val categoryState = communityIndexViewModel.categories.observeAsState()

    communityIndexViewModel.obtainCategoryList()

    Scaffold(
        topBar = {
            TopAppBar(
                navigationIcon = {
                    IconButton(onClick = { onPopBackStack() }) {
                        Icon(imageVector = Icons.AutoMirrored.Filled.ArrowBack, contentDescription = "后退按钮")
                    }
                },
                colors = TopAppBarDefaults.topAppBarColors(
                    containerColor = MaterialTheme.colorScheme.primaryContainer,
                    titleContentColor = MaterialTheme.colorScheme.primary,
                ),
                title = {
                    Text(categoryState.value?.firstOrNull { it.id == route.categoryId }?.name ?: "<主题分类>")
                }
            )
        },
    ) { innerPadding ->
        Column(modifier = Modifier.padding(innerPadding)) {
            DateTimeText("主题列表 - ${route.categoryId}")
        }
    }
}
```

依然是在`NavHost`中实现后退方法并传入：

```kotlin
NavHost(
    navController = navController,
    startDestination = IndexRoute(),
) {
    // 首页路线
    index(onNavigateToTopicList = { navController.navigate(it) })
    // 主题列表路线
    topicList(onPopBackStack = { navController.popBackStack() })
}
```

# 网格布局

参考资料：[延迟网格](https://developer.android.com/develop/ui/compose/lists?hl=zh-cn#lazy-grids)

这里即将用到的延迟网格和官方教程示例里用过的延迟行列，显然和`RecyclerView`有着非常相似的行为

现在我们用它来在首页显示主题分类入口：

1. 新建一个可组合函数`CategoryGrid`传入从参数分类数据，如果分类数据为空则显示一个占位提示，否则使用延迟网格渲染分类入口，入口由一张图片和名称组成
2. 修改可组合函数`CommunityIndex`和`CommunityIndexViewModel`增加对分类数据的请求，并将`CategoryGrid`放在轮播图下方，中间间隔`20dp`
3. 顺带把之前写好的`onNavigateToTopicList`回调功能也一并实现了

示例：

```kotlin
/**
 * 社区首页
 * @param viewModel [CommunityIndexViewModel]
 * @param onNavigateToTopicList 导航到主题列表
 */
@Composable
fun CommunityIndex(viewModel: CommunityIndexViewModel, onNavigateToTopicList: (route: TopicListRoute) -> Unit) {
    viewModel.obtainBanners()
    viewModel.obtainCategoryList()
    val bannerState = viewModel.banners.observeAsState()
    val categoryState = viewModel.categories.observeAsState()

    Column {
        HorizontalCarousel(bannerState.value) {
            AsyncImage(model = it.imgAddr, contentDescription = null, modifier = Modifier.fillMaxWidth())
        }

        Spacer(Modifier.height(20.dp))

        CategoryGrid(categoryState.value, onNavigateToTopicList = onNavigateToTopicList)
    }
}

@Composable
fun CategoryGrid(categories: List<TopicCategory>?, onNavigateToTopicList: (route: TopicListRoute) -> Unit) {
    if (categories.isNullOrEmpty()) {
        Text(text = "暂无数据:主题分类", modifier = Modifier.fillMaxSize(), textAlign = TextAlign.Center)
        return
    }
    LazyVerticalGrid(
        columns = GridCells.Adaptive(minSize = 128.dp),
    ) {
        items(categories.size) {
            val item = categories[it]
            Column(horizontalAlignment = Alignment.CenterHorizontally, modifier = Modifier.clickable {
                onNavigateToTopicList(TopicListRoute(item.id))
            }) {
                AsyncImage(model = item.imgC, contentDescription = null, modifier = Modifier.size(80.dp))
                Text(item.name)
            }
        }
    }
}

```

# 延迟列表

参考资料：[延迟列表](https://developer.android.com/develop/ui/compose/lists?hl=zh-cn#lazy)

官方入门示例中已经试用过了，这次我们把它加上`ViewModel`一起用来实现主题列表内容的展示，并配上`自动加载更多`功能：

1. 修改`TopicListRoute`路线数据类的定义，使之包含请求所需参数；对应修改`CategoryGrid`可组合函数中对它的调用参数

```kotlin
/**
 * 主题列表路线
 * @param queryType 查询类型
 * @param userId 用户ID ， 查询用户发帖时使用
 * @param themeId 话题ID
 * @param categoryId 类型ID
 * @param sortType 排序 todo
 * @constructor
 */
@Serializable
data class TopicListRoute(
    val queryType: QueryType,
    val userId: Int? = null,
    val themeId: Int? = null,
    val categoryId: Int? = null,
    val sortType: Int = 1,
)
```

2. 新建`TopicListViewModel`类，参照`CommunityIndexViewModel`模式编写业务内容（代码省略）
3. 新建`LazyTopicList`可组合函数，传入`TopicListRoute`路线数据类对象和`TopicListViewModel`类对象；使用`LazyColumn`显示主题列表内容，并加上`自动加载更多`功能（`TopicItem`可组合函数代码省略）

```kotlin
@Composable
fun LazyTopicList(route: TopicListRoute, viewModel: TopicListViewModel, modifier: Modifier = Modifier) {
    val state = viewModel.topics.observeAsState()
    if (state.value.isNullOrEmpty()) {
        viewModel.obtainTopicList(route)
        Text(text = "暂无数据:主题列表", modifier = modifier.fillMaxWidth(), textAlign = TextAlign.Center)
        return
    }

    // 列表状态
    val listState = rememberLazyListState()
    // 是否需要加载更多数据
    val loadMore = remember {
        derivedStateOf {
            val layoutInfo = listState.layoutInfo
            val lastIndex = layoutInfo.visibleItemsInfo.lastOrNull()?.index ?: 0
            // 当”当前展示的项目序号+1“ 大于等于 "列表项总数-2" 时加载更多
            (lastIndex + 1) >= layoutInfo.totalItemsCount - 2
        }
    }
    // 监控loadMore变化，当其为 true 时加载更多数据
    LaunchedEffect(loadMore) { snapshotFlow { loadMore.value }.distinctUntilChanged().collect { if (it) viewModel.obtainTopicList(route) } }

    LazyColumn(state = listState,
        modifier = modifier,
        verticalArrangement = Arrangement.spacedBy(2.dp),
        contentPadding = PaddingValues(4.dp)) {
        itemsIndexed(state.value!!, key = { _, e -> e.topicId }) { _, item -> TopicItem(item) }
    }
}
```

这里使用了key参数来优化性能，另外官方表示[延迟布局在调试模式下性能较差](https://developer.android.com/develop/ui/compose/lists?hl=zh-cn#measuring-performance)，属于正常情况。

4. 使用`LazyTopicList`替换`TopicListComposable`中原先的占位文本
