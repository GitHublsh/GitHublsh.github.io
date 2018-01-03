---
title: Android Gradle构建入门 (一)
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


#### 二、查看整个项目的build.gradle

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

#### 三、查看app module的build.gradle

看一下 build.gradle

	apply plugin: 'com.android.application'
	
	android {
	    compileSdkVersion 26
	    defaultConfig {
	        applicationId "com.example.liushihan.gradletest"
	        minSdkVersion 21
	        targetSdkVersion 26
	        versionCode 1
	        versionName "1.0"
	        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
	    }
	    buildTypes {
	        release {
	            minifyEnabled false
	            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
	        }
	    }
	}
	
	dependencies {
	    implementation fileTree(dir: 'libs', include: ['*.jar'])
	    implementation 'com.android.support:appcompat-v7:26.1.0'
	    implementation 'com.android.support.constraint:constraint-layout:1.0.2'
	    testImplementation 'junit:junit:4.12'
	    androidTestImplementation 'com.android.support.test:runner:1.0.1'
	    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.1'
	}


第一行：

	apply plugin: 'com.android.application'
	
应用Android插件（在项目构建文件的构建脚本部分中引用）到当前项目。Gradle中的插件可以将自定义任务，新配置，依赖关系和其他功能添加到Gradle项目中。 在这种情况下，应用Android插件会添加各种各样的任务，这些任务由接下来显示的android块配置。


	android {
	    compileSdkVersion 26
	    defaultConfig {
	        applicationId "com.example.liushihan.gradletest"
	        minSdkVersion 21
	        targetSdkVersion 26
	        versionCode 1
	        versionName "1.0"
	        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
	    }
	    buildTypes {
	        release {
	            minifyEnabled false
	            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
	        }
	    }
	}
	
对这些属性进行简单的介绍：

* compileSdkVersion：Android SDK版本
* defaultConfig:defaultConfig部分包含应用程序的所有变体（构建类型和产品风格的组合）共享的属性。
	* applicationId:applicationId基于创建应用程序时指定的域名和项目名称，并且在Google Play商店中必须是唯一的。

	
	* minSdkVersion:minSdkVersion是你愿意支持这个应用程序的最低Android API，targetSdkVersion应该是最新的Android版本。

	
	* versionCode:versionCode的值应该是在将新版本的应用上传到Google Play商店之前递增的整数。 这个值和applicationId一起告诉Google，这是一个现有应用程序的新版本，而不是一个新的应用程序。

	* versionName:版本名称值用于自己的内部版本跟踪。

	* testInstrumentationRunner：testInstrumentationRunner属性配置为使用为Android应用程序配置的JUnit 4测试运行器。



在这个代码块下面，就是buildTypes.默认情况下，Android应用程序支持两种构建类型，debug和release.这里没有显示debug配置部分，即使用所有调试的默认设置。

	  buildTypes {
	        release {
	            minifyEnabled false
	            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
	        }
	    }


继续往下面看，就是这个应用程序所依赖的库

	dependencies {
	    implementation fileTree(dir: 'libs', include: ['*.jar'])
	    implementation 'com.android.support:appcompat-v7:26.1.0'
	    implementation 'com.android.support.constraint:constraint-layout:1.0.2'
	    testImplementation 'junit:junit:4.12'
	    androidTestImplementation 'com.android.support.test:runner:1.0.1'
	    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.1'
	}


配置依赖关系是构建Gradle应用程序的基础部分。 在这种情况下，依赖项部分显示了Implementation，testImplementation和androidTestImplementation配置的值。

testImplementation依赖只包含最新的稳定的JUnit 4发行版。 JUnit类和测试注释将在编译时在src / test / java层次结构中可用。

androidTestImplementation依赖项是指Espresso测试库，用于Android应用程序的集成测试。 在这种情况下，Espresso在没有通常包含的support-annotations库的情况下被请求，因为已经通过其他依赖项包含了不同的版本。 在后面的步骤中，您将看到如何找出该库的版本以及原因。


	fileTree(dir: 'libs', include: ['*.jar'])
	
是一个fileTree依赖项，它将libs文件夹中的所有jar文件添加到编译类路径中

	com.android.support:appcompat-v7:26.1.0
	
将Android兼容性库添加到项目中。 这使您可以在SDK版本7以前的任何Android应用程序中使用材质设计主题和其他功能。

	com.android.support.constraint:constraint-layout:1.0.2
	
将Android约束布局添加到项目中。 这允许您在任何像SDK版本9一样早的Android应用程序中使用ConstraintLayout布局类。


### 运行标准的Gradle任务

Android Studio通过IDE可以轻松构建和部署应用程序的调试版本，但最终Gradle还是参与其中。可以在Android Studio中命令行使用gradle命令构建

	$ ./gradlew build
	

