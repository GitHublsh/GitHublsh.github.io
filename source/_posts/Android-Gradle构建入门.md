---
title: Android Gradle构建入门
date: 2017-12-30 17:35:50
tags: [Gradle]
---

### Android Gradle入门

##### 本文内容来自官方文档整理


#### 一、查看生成的Gradle文件列表

默认情况下，Android Studio将以“Android”模式下的“项目视图”开始，如下图所示：

![“asmodel”](http://ot29getcp.bkt.clouddn.com//blog/asandroidmodel.png)



Android项目是Gradle多项目构建，具有顶级build.gradle文件和名为app的子目录，并具有自己的build.gradle文件。 顶层构建文件在图中标记为（Project：HelloWorldGradle），并且应用程序构建文件（Module：app）附加到其中。


在项目中，可能有两个名为gradle.properties的文件。 一个是本地项目。 另一个只有在主目录的.gradle子目录中具有全局gradle.properties文件的同名文件才存在。

* settings.gradle


Gradle使用文件settings.gradle来配置多项目构建。 它应该由一行代码组成：

	include ':app'
	
这就是告诉Gradle，app也是一个Gradle project,当项目中还会依赖别的library，那么还需要在后面添加相应的信息。

* gradle-wrapper.properties


gradle-wrapper.properties，它配置了所谓的Gradle Wrapper。 这使您可以构建Android项目，而无需首先安装Gradle。 该文件的内容应该类似于：

	distributionBase=GRADLE_USER_HOME
	distributionPath=wrapper/dists
	zipStoreBase=GRADLE_USER_HOME
	zipStorePath=wrapper/dists
	distributionUrl=https\://services.gradle.org/distributions/gradle-4.1-all.zip
	
前四行表示当包装首次运行时，它将下载Gradle发行版并将其存储在您的主目录中的.gradle / wrapper / dists目录中。

最后一行显示了distributionUrl的值，这是Gradle将下载指定分配的地址。（具体版本号可能与此处（4.1）中显示的版本号不同，并且URL可能引用二进制版本（-bin），而不是此示例中显示的完整版本（-all）。）


#### 二、查看项目的Gradle构建文件

project的build.geadle文件：

	// Top-level build file where you can add configuration options common to all sub-projects/modules.
	
	buildscript {
	    
	    repositories {
	        google()
	        jcenter()
	    }
	    dependencies {
	        classpath 'com.android.tools.build:gradle:3.0.0'
	        
	
	        // NOTE: Do not place your application dependencies here; they belong
	        // in the individual module build.gradle files
	    }
	}
	
	allprojects {
	    repositories {
	        google()
	        jcenter()
	    }
	}
	
	task clean(type: Delete) {
	    delete rootProject.buildDir
	}


各个代码块具体作用：

1. buildscript - 下载插件
2. dependencies - 标识Android插件
3. allproject - 	top-level and module projects的配置
4. task clean - 特设任务

Gradle定义了一种基于Groovy的DSL语言来构建，buildscript标签是DSL的一部分。 它告诉Gradle构建需要一个可能不是基线Gradle分布的一部分的插件，并告诉Gradle在哪里找到它。 在这种情况下，使用坐标语法“group：name：version”指定所需的插件，其中组为com.android.tools.build，名称为gradle，版本为3.0.1。

当Gradle第一次构建这个项目时，插件将被下载并缓存，所以这个任务只执行一次。

allprojects标签保存适用于顶层项目及其包含的任何子项目的配置细节。 在这种情况下，该块指定任何所需的依赖项应该从https://jcenter.bintray.com上的公共Bintray Artifactory存储库的google或jcenter下载。

最后，构建文件包含一个名为clean的自定义（或临时）任务。 它使用内置的任务类型Delete并对其进行配置，以便干净的任务将删除rootProject中的buildDir。 两者都是项目属性，其值默认为该应用程序驻留的项目中的构建目录。
