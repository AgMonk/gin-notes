# 为什么要使用Jetpack Compose

`Andriod Studio`新建空白项目时已经默认使用`Jetpack Compose`，显然是官方强烈推荐，具体的好处自己百度

对我来说，摆脱xml就是最大的好处

# 新建一个Compose项目

`Andriod Studio`中依次选择`File`-`New`-`New Project`，模板选择`Empty Activity`，下一步，界面会提示将使用`Jetpack Compose`创建一个空的`activity`，这里构建配置语言我们选`Grovvy`，填写其他信息，`finish`

# 加速依赖导入（可选）

项目创建好之后，以此选择`View`-`Tool Windows`-`Build`（或左下角Build按钮），打开构建工具窗口，先把同步操作停掉

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

# 使用Navigation制作一个带底部导航按钮的首页

## 导入依赖

```groovy
  implementation "androidx.navigation:navigation-compose:2.8.4"
```

最新版本见 [官网](https://developer.android.google.cn/jetpack/androidx/releases/navigation?hl=zh-cn)

## 代码及分析

先新建一个路由类：

```kotlin
/**
 * 路由
 * @param route 路由地址(navigation使用)
 * @param title 路由标题(导航栏按钮)
 * @param content 路由内部组件
 * @constructor
 */
data class Route(
    val route: String,
    val title: String,
    val content: @Composable AnimatedContentScope.(NavBackStackEntry) -> Unit
)

```

打开`MainActivity`，在`MainActivity`类外新建三个可组合函数，他们将是三个导航按钮各自对应的页面，这里我们先简单地放一个全宽的文本组件：

```kotlin
@Composable
fun DailyWork() {
    Text("日常: ${ZonedDateTime.now().toLocalDateTime()}", modifier = Modifier.fillMaxWidth())
}

@Composable
fun CommunityIndex() {
   Text("社区: ${ZonedDateTime.now().toLocalDateTime()}", modifier = Modifier.fillMaxWidth())
}

@Composable
fun Setting() {
    Text("设置: ${ZonedDateTime.now().toLocalDateTime()}", modifier = Modifier.fillMaxWidth())
}
```

新建一个可组合函数，作为首页的根组件：

- 根组件中持有：`Navigation`的导航控制器、当前选中的路由序号（默认为1）、路由配置
- 导航栏也路由页根据路由配置来生成
- 使用`Scaffold`脚手架，将导航栏(NavigationBar)放入`bottomBar`，将路由页(NavHost)放入`content`

```kotlin

@Preview
@Composable
fun Index(initialIndex: Int = 1) {
    // 导航控制器
    val navController = rememberNavController()
    // 选中的index
    var selectedIndex by remember { mutableIntStateOf(initialIndex) }

    // 路由配置
    val routes = listOf(
        Route("daily", "日常") { DailyWork() },
        Route("community", "社区") { CommunityIndex() },
        Route("setting", "设置") { Setting() },
    )

    Scaffold(
        modifier = Modifier.fillMaxSize(),
        bottomBar = {
            // 选中的index
            NavigationBar {
                routes.forEachIndexed { index, route ->
                    NavigationBarItem(
                        label = { Text(route.title) },
                        selected = selectedIndex == index,
                        onClick = {
                            // 这个判断是必要的，否则重复点击同一个按钮时也会重新加载该路由
                            if (selectedIndex != index) {
                                navController.navigate(route.route)
                                selectedIndex = index
                            }
                        },
                        // 导航按钮图标先统一使用内置图标
                        icon = { Icon(Icons.Rounded.Home, contentDescription = null) },
                    )
                }
            }
        }
    ) { innerPadding ->
        NavHost(
            navController = navController,
            startDestination = routes[initialIndex].route,
            modifier = Modifier.padding(innerPadding),
        ) {
            routes.forEach { route -> composable(route=route.route,content = route.content)  }
        }
    }
}
```

修改`MainActivity`中的`setContent`方法为，并清理预设的无效可组合函数：

```kotlin
setContent {
    MyApplicationTheme {
        Index()
    }
}
```

在手机上运行APP，可以看到路由切换已经实现，但是很明显可以发现一个问题：切换过程中文字重叠了，而不是我们常见的滑动切换的效果，下一节将解决这个问题
