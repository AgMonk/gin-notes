```groovy
plugins {
    id 'maven-publish'
    id 'java-library'
}


//打包版本
java {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_17
}

//打包源码任务
tasks.register('sourcesJar', Jar) {
    from sourceSets.main.allSource
    archiveClassifier = 'sources'
}

// 发布配置
publishing {
    publications {
        myPublish(MavenPublication) {
            from components.java            
            artifact sourcesJar

            versionMapping {
                usage('java-api') {
                    fromResolutionOf('runtimeClasspath')
                }
                usage('java-runtime') {
                    fromResolutionResult()
                }
            }
        }
    }

    // 发布到的仓库
    repositories {
        mavenLocal()
        maven {
            // 根据版本后缀决定打包到那个目录
            name 'project'
            def rUrl = rootProject.layout.projectDirectory.dir("repo/releases")
            def sUrl = rootProject.layout.projectDirectory.dir("repo/snapshots")
            url version.endsWith("SNAPSHOT") ? sUrl : rUrl
        }
    }
}
```