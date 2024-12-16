# 为什么要使用Jetpack Compose

`Andriod Studio`新建空白项目时已经默认使用`Jetpack Compose`，显然是官方强烈推荐，具体的好处自己百度

对我来说，摆脱xml就是最大的好处

# 新建一个Compose项目

`Andriod Studio`中依次选择`File`-`New`-`New Project`，模板选择`Empty Activity`，下一步，界面会提示将使用`Jetpack Compose`创建一个空的`activity`，这里构建配置语言我们选`Grovvy`，填写其他信息，`finish`

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
                .fillMaxSize()
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

# Navigation的使用

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
