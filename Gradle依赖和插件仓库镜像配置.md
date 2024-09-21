来源： https://juejin.cn/post/7067505778353143815



# 插件仓库镜像

在`settings.gradle`第一行加入：

```groovy
pluginManagement {
    println "aliyun pluginManagement"
    repositories {
            maven { url "https://maven.aliyun.com/repository/gradle-plugin" }
            maven { url "https://maven.aliyun.com/repository/spring-plugin" }
            gradlePluginPortal()
    }
}
```

# 依赖仓库镜像

在`build.gradle`中加入

```groovy
repositories {
    mavenLocal()
    maven { url 'https://maven.aliyun.com/repository/public/' }
    maven { url 'https://maven.aliyun.com/repository/spring/' }
    maven { url 'https://maven.aliyun.com/repository/google/' }
    maven { url 'https://maven.aliyun.com/repository/gradle-plugin/' }
    maven { url 'https://maven.aliyun.com/repository/spring-plugin/' }
    maven { url 'https://maven.aliyun.com/repository/grails-core/' }
    maven { url 'https://maven.aliyun.com/repository/apache-snapshots/' }
    maven { url 'https://maven.aliyun.com/nexus/content/groups/public/' }
    mavenCentral()
}
```

如需加入Nexus私服格式为：

```groovy
maven { 
        allowInsecureProtocol = true
        url 'http://192.168.0.10:8081/repository/Aliyun-Group/' 
}
```

