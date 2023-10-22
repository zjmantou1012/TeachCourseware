gradle版本：

若gradle配置过低，可能会出现aar包生成失败的情况

![](https://cdn.nlark.com/yuque/0/2022/jpeg/26044650/1648992316133-3f71ab8f-8b2f-4dc1-b950-00786348ab26.jpeg)

```
apply plugin: 'maven-publish'

// 判断版本是Release or Snapshots
def isReleaseBuild() {
    return !VERSION.contains("SNAPSHOT");
}
// 获取仓库url
def getRepositoryUrl() {
    return isReleaseBuild() ? RELEASE_URL : SNAPSHOT_URL;
}
publishing {
    publications {
        //6寸渠道包
        liucun(MavenPublication) {//cqtest即为渠道的名称，可以随意取
            groupId = GROUP_ID//公司域名
            artifactId = ARTIFACT_ID_R6//该aar包的名称
            version = VERSION//版本号
            def projectName = project.getName()
            artifact "build/outputs/aar/reader-liucun-release.aar"//对应该渠道打包出来的aar所在的目录（如何打包请往下看）
            pom.withXml{
                def dependenciesNode = asNode().appendNode("dependencies")
                configurations.implementation.allDependencies.forEach(){
                    Dependency dependency ->
                    if (dependency.version != "unspecified" && dependency.name != "unspecified"){
                        def dependencyNode = dependenciesNode.appendNode('dependency')
                        dependencyNode.appendNode('groupId', dependency.group)
                        dependencyNode.appendNode('artifactId', dependency.name)
                        dependencyNode.appendNode('version', dependency.version)
                    }
                }
            }
        }
        
        //9寸渠道包
        jiucun(MavenPublication) {
            groupId = GROUP_ID
            artifactId = ARTIFACT_ID_R9
            version = VERSION
            def projectName = project.getName()
            artifact "build/outputs/aar/reader-jiucun-release.aar"
            pom.withXml{
                def dependenciesNode = asNode().appendNode("dependencies")
                configurations.implementation.allDependencies.forEach(){
                    Dependency dependency ->
                    if (dependency.version != "unspecified" && dependency.name != "unspecified"){
                        def dependencyNode = dependenciesNode.appendNode('dependency')
                        dependencyNode.appendNode('groupId', dependency.group)
                        dependencyNode.appendNode('artifactId', dependency.name)
                        dependencyNode.appendNode('version', dependency.version)
                    }
                }
            }
        }
        
    }
    repositories {
        maven {
            //maven仓库的地址
            url = getRepositoryUrl()
            //仓库的用户名及密码
            credentials {
                username='smpdeploer'
                password ='Smpdeploer_123'
            }
            //这是一个本地仓库地址即D盘的maventestrepository文件夹，可以用来测试发布aar包，发布之后的aar将存放于该地址
            // url = "file://d:/maventestrepository"
        }
    }
}
```

```
apply plugin: 'com.android.library'
//应用butterknife插件
apply plugin: 'com.jakewharton.butterknife'
apply from:'mavenpublish.gradle'
android {
    productFlavors {
        jiucun {

        }
        liucun {


        }

    }


    flavorDimensions "default"

 compileSdkVersion rootProject.ext.compileSdkVersion
    buildToolsVersion rootProject.ext.buildToolsVersion


    defaultConfig {
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
        versionCode 1
 versionName "1.0"

 consumerProguardFiles 'consumer-rules.pro'
 //使用阿里的ARouter配置
 javaCompileOptions {

            annotationProcessorOptions {

                arguments = [AROUTER_MODULE_NAME: project.getName()]

            }
        }
    }

    buildTypes {
        release {
            minifyEnabled false
 proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
 }
    }

}

configurations.all {
    // 动态版本
 resolutionStrategy.cacheDynamicVersionsFor 0, 'minutes'
 // 变化模块
 resolutionStrategy.cacheChangingModulesFor 0, 'minutes'
}


dependencies {
    api fileTree(dir: 'libs', include: ['*.jar'])
       
}
```

渠道包java文件存放

在src文件下 新建两个以渠道名命名的包，将各自渠道独有的代码存放至该位置。公共代码不需要移动

![](https://cdn.nlark.com/yuque/0/2022/jpeg/26044650/1648992561778-c12a445e-0fa1-48c2-b73f-296439b3d597.jpeg)

生成的aar包

![](https://cdn.nlark.com/yuque/0/2022/jpeg/26044650/1648992593031-89067798-05b2-46b1-a39c-1d233c838bfc.jpeg)

执行该命令，会在下图目录中生成两个aar包

![](https://cdn.nlark.com/yuque/0/2022/jpeg/26044650/1648992613265-169cae6f-590c-4cb0-be71-5f02f13f9db2.jpeg)

上传至maven仓库

![](https://cdn.nlark.com/yuque/0/2022/jpeg/26044650/1648993089050-c1feae88-a2db-4b62-95bc-987c20719110.jpeg)

执行该命令可上传至maven仓库