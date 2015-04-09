---
layout: post
category: android
title: 使用Gradle构建Android程序  
toc: true  
comments: true  
tagline: by Rinvay.T
tags: [android, gradle]
---




Android Studio正式版早已经发布了，默认使用Gradle构建，GitHub上80%的Android项目也都是使用Gradle构建的，我们还有什么理由不使用Gradle呢？

## 环境要求

- JDK [下载地址][0]
- Android SDK [下载地址][1]
- Gradle [下载地址1][2][下载地址2][3]  

给一个国内的下载地址：[AndroidDevTools](http://www.androiddevtools.cn/)

<!--more-->

## 开始(改造Eclipse项目)

eclipse可以直接用adt导出为gradle项目，android studio直接创建gradle项目。但是，我们还是有必要了解下手动改造，这让我们更清楚每个文件的作用。

### 独立项目构建

在我的另一篇博客[Gradle插件用户指南(译)][4]中，3.1节可以看到，  
一个最简单的build.gradle文件是这样的：

    buildscript {
        repositories {
            jcenter()
        }
     
        dependencies {
            classpath 'com.android.tools.build:gradle:1.1.0'
        }
    }
     
    apply plugin: 'com.android.application'
     
    android {
        compileSdkVersion 19
        buildToolsVersion "22.0.1"
    }   

这里，要求项目目录结构是gradle的默认目录：

    project-root  
        |-src
            |-main
                |-java
                |-res
                    |-drawable
                    |-layout
                    |-values
                |-AndroidManifest.xml

如果是eclipse下的项目，需要配置下项目目录，修改后的build.gradle文件是这样的:

    buildscript {
        repositories {
            jcenter()
        }
    
        dependencies {
            classpath 'com.android.tools.build:gradle:1.1.0'
        }
    }
    
    apply plugin: 'com.android.application'
    
    android {
        compileSdkVersion 19
        buildToolsVersion "22.0.1"
    
        sourceSets {
            main{
                manifest.srcFile 'AndroidManifest.xml'
    
                java.srcDirs = ['src']
                aidl.srcDirs = ['src']
                renderscript.srcDirs = ['src']
    
                res.srcDirs = ['res']
    
                assets.srcDirs = ['assets']
    
                jniLibs.srcDirs = ['libs']
    
            }
        }
    
        dependencies {
            compile fileTree(dir:'libs', include:['*.jar'])
        }
    } 

**把这个build.gradle拷贝到项目根目录下，就可以使用gradle来构建你的app了。**

打开终端/命令行，切换到项目根目录，运行`gradle assemble`，在`main-project/build/outputs/apk`下就可以看到生成的apk文件

**Note:**   
要注意原项目的依赖，比如 ApiDemos ，是依赖support-v4的，要添加到dependencies中。  

    compile 'com.android.support:support-v4:19.0.0'



### 多项目构建

如果程序是由多个子工程构成的话，上面的build.gradle文件就不够用了。 
 
假设你的项目目录是这样的

    root
        |-main-project
        |-library-project1
        |-library-project2
    
并且，依赖关系如下：

    main-project -> library-project1
    library-project1 -> library-project2


那么，Gradle构建文件应该包含以下几个文件：

    settings.gradle  
    build.gradle  
    main-project/build.gradle  
    library-project1/build.gradle  
    library-project2/build.gradle  

文件位置是这样：

    root
        |-main-project
            |- ...
            |-build.gradle
        |-library-project1
            |- ...
            |-build.gradle
        |-library-project2
            |- ...
            |-build.gradle
        |-build.gradle
        |-settings.gradle
 

文件内容如下： 
 
- settings.gradle:

        include ':main-project','library-project1','library-project2'
    
settings.gradle声明了程序包含的子项目，如果子项目时包含在三级子目录下，例如`root/library/library-project1`，那么声明是这样的：

    ':library:library-project1'


- build.gradle:

        buildscript {
            repositories {
                jcenter()
            }
            dependencies {
                classpath 'com.android.tools.build:gradle:1.0.0'
            }
        }
        
    根目录下的buidle.gradle声明了项目的编译环境，指定了Android Gradle插件的版本。和上一个章节，那个单一的build.gradle的前半部分内容是一样的。

- main-project/build.gradle:


        apply plugin: 'com.android.application'
        
        android {
            compileSdkVersion 19
            buildToolsVersion "22.0.1"
        
            sourceSets {
                main{
                    manifest.srcFile 'AndroidManifest.xml'
        
                    java.srcDirs = ['src']
                    aidl.srcDirs = ['src']
                    renderscript.srcDirs = ['src']
        
                    res.srcDirs = ['res']
        
                    assets.srcDirs = ['assets']
        
                    jniLibs.srcDirs = ['libs']
        
                }
            }
        
            dependencies {
                compile fileTree(dir:'libs', include:['*.jar'])
                compile project(':library-project1')
            }
        } 

这个build.gradle的内容与上一章节build.gradle的后半部分基本一致，只是多了一个库项目依赖的声明。  
这里假设目录结构和eclipse下的android项目目录结构是一样的，如果实际目录和gradle的默认目录结构一样，可以去掉`sourceSets{...}`部分

- library-project1/build.gradle和main-project/build.gradle基本一致，只有两个地方不一样。

    - `apply plugin: 'com.android.application'`改成`apply plugin: 'com.android.library'`
    - `compile project(':library-project1')`改成`compile project(':library-project2')`

- library-project1/build.gradle和library-project2/build.gradle基本一致，只有一个地方不一样。

    - 去掉`compile project(':library-project2')`

至此，多项目的工程完成了gradle改造，可以在项目根目录下运行`gradle assemble`进行构建了。

构建的其他细节，请参考我的另一篇博客[Gradle插件用户指南(译)][4]。

### gradle wrapper

和Eclipse导出或者Android Studio创建的项目相比，我们改造完的项目还差了几个文件。  
在项目根目录下，运行终端命令`gradle wrapper`，就会生成下面几个文件：

    project-root
        |-gradle
            |-wrapper
                |-gradle-wrapper.jar
                |-gradle-wrapper.properties
        |-gradlew
        |-gradlew.bat

`gradlew`, `gradlew.bat` 是支持多平台的gradle运行命令，如果运行时，发现系统没有对应版本的gradle，会通过`gradle-wrapper.jar`下载`gradle-wrapper.properties`中指定的gradle版本。这样的话，**任何人获取代码后，不用安装gradle，就可以构建工程**。  

生成的`gradle-wrapper.properties`文件中指定的版本，就是运行`gradle wrapper`命令时的gradle版本。

通过gradlew，可以执行gradle构建任务，例如：

    ./gradlew assemble      

## 使用Android Studio

Android Studio创建的项目默认使用Gradle构建，并且目录结构也是Gradle默认的。

### 使用Android Studio 创建 Android 项目

这个很简单，新建项目，按照引导一步一步往下走就行了。

新建项目的结构如下：

    project-root
        |-app
            |-src
                |-androidTest
                |-main
                    |-java
                    |-res
                    |-AndroidManifest.xml
            |-build.gradle
        |-gradle
            |-wrapper
                |-gradle-wrapper.jar
                |-gradle-wrapper.properties
        |-build.gradle
        |-gradle.properties
        |-gradlew
        |-gradlew.bat
        |-local.properties
        |-settings.gradle
    
`app`  
是项目代码和资源目录

`gradlew`,`gradlew.bat`,`gradle/wrapper`  
上一章节已经介绍过了

`build.gradle`,`settings.gradle`  
上一章节已经介绍过了

`gradle.properties`  
gradle运行的参数

`local.properties`  
本地环境参数，例如android sdk路径：sdk.dir

### 导入Gradle项目到Android Studio

对于非Android Studio项目，完成gradle改造后，可以导入到Android Studio中。

    File -> Open...  
    或者Android Studio启动页的  
    Open an existing Android Studio project

直接选择打开项目根目录下的settings.gradle(如果没有的话，打开build.gradle)文件，或者setting.gradle文件的父目录，结果如下图：  

![image](/assets/images/import-from-gradle.png)

如果运行过前面的`gradle wrapper`命令的话(或者是由eclipse导出的gradle项目)，目录下有`wrapper`文件夹，`Use default gradle wrapper(recommanded)`选项就是可选的，否则只能选择`Use local gradle distribution`。

## 混淆与签名

请看我另一篇博客 [Gradle插件用户指南(译)](http://rinvay.github.io/android/2015/03/26/Gradle-Plugin-User-Guide\(Translation\)/#1090403) 3.4.3节和3.4.4节


## 构建多渠道包

### 动态替换AndroidManifest.xml中的文本

以友盟为例，是通过在AndroidManifest.xml中添加`meta-data`标签来区分渠道：

    <meta-data
        android:name="UMENG_CHANNEL"
        android:value="official" />

不同的value值对应不同的渠道。

android gradle plugin从0.13.0版本开始，productFlavor开始支持manifestPlaceHolder，我们使用这个就可以实现友盟的多渠道打包。

1. 修改AndroidManifest.xml，添加placeHolder变量

        <meta-data
            android:name="UMENG_CHANNEL"
            android:value="${channel_param}" />


2. 配置build.gradle文件
    
        android {
            ...
            defaultConfig {
                ...
                manifestPlaceholders = [channel_param: "official"]
            }
            
            productFlavors {
                channel1 {
                    ...
                }
                ...
                channelN {
                    ...
                }                
            }
            
            productFlavors.all {
                flavor -> flavor.manifestPlaceholders = [channel_param: name]
            }
            
        }    
    
    `productFlavors.all`是合并的写法，也可以分开写：
    
        channel1 {
            manifestPlaceholder = [channel_param:name]
        }


再进一步，国内的应用商店可以说数以百计，如果每个都在build.gradle文件里写一个flavor，那么build.gradle文件就会变得冗长。  
我们把渠道号写到一个单独的文件里`channels.txt`，内容如下：

    channel-360
    channel-baidu

然后在build.gradle里定义函数`flavors()`，作用就是根据`channels.txt`里的内容动态创建productFlavor，然后在build.gradle中调用这个函数，优化后的结果是这样：

    android {
        ...
        defaultConfig {
            ... 
            manifestPlaceholders = [channel_param: "official"]
        }
    
        flavors()
    }
    
    def flavors() {
        def path = './channels.txt' //channels.txt的路径
        file(path).eachLine {
            line ->
            if(!line.startsWith("//")) {
                //动态创建productFlavor
                android.productFlavors.create(line, {
                    manifestPlaceholders = [channel_param: name]
                })
    
            }
        }
    }

在Android Studio中，点击`Sync Project with Gradle Files`，

[0]:http://www.oracle.com/technetwork/java/javase/downloads/index.html
[1]:http://developer.android.com/sdk/index.html#Other
[2]:http://gradle.org/
[3]:http://services.gradle.org/distributions/
[4]:http://rinvay.github.io/android/2015/03/26/Gradle-Plugin-User-Guide(Translation)/
