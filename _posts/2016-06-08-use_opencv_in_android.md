---
layout: post
title: Android Studio中使用OpenCV Android SDK
date: 2016-06-08 09:18:22
categories: code
tags: [Android]
---

[**OpenCV**](http://opencv.org/)是著名的跨平台计算机视觉开源库，广泛应用于计算机视觉相关领域。

**OpenCV** 已经发布 **Android** 平台下的 SDK，可以直接导入 **Android Studio**。

**OpenCV Android SDK** 下载地址： [Download OpenCV Android SDK](http://docs.opencv.org/2.4/doc/tutorials/introduction/android_binary_package/O4A_SDK.html#get-the-opencv4android-sdk) 。

## Android Studio项目中配置使用OpenCV Android SDK

1. 在项目根目录下创建 ```libraries``` 目录。

2. 复制 **Android SDK** 中目录 ```sdk``` 下的 ```java``` 文件夹到刚刚创建的 ```libraries``` 目录中。

3. 将复制的 ```java``` 文件夹重命名为 ```opencv``` 。

4. 在重命名的 ```opencv``` 文件夹下创建一个 ```build.gradle``` 文件，内容如下( ```compileSdkVersion``` , ```buildToolsVersion``` , ```minSdkVersion``` , ```targetSdkVersion``` , ```versionCode``` ,  ```versionName``` 等可依实际情况而定)：

    ```gradle

    apply plugin: 'android-library'

    buildscript {
        repositories {
            mavenCentral()
        }
        dependencies {
            classpath 'com.android.tools.build:gradle:0.9.+'
        }
    }

    android {
        compileSdkVersion 23
        buildToolsVersion "23.0.3"

        defaultConfig {
            minSdkVersion 15
            targetSdkVersion 23
            versionCode 2411
            versionName "2.4.11"
        }

        sourceSets {
            main {
                manifest.srcFile 'AndroidManifest.xml'
                java.srcDirs = ['src']
                resources.srcDirs = ['src']
                res.srcDirs = ['res']
                aidl.srcDirs = ['src']
            }
        }
    }

    ```

5. 编辑项目根目录下的 ```settings.gradle``` 文件，添加一行代码：

    ```gradle
    include ':libraries:opencv'
    ```

6. 在 **Android Studio** 中同步 **Gradle** 。

7. 右键工程， ```Open Module Settings``` ， 左边选中应用的module名称，右边点击 ```Dependencies``` 选项，再点击 **+** 按钮，添加依赖。

8. 选择 ```Module dependency```， 会出现一个含有多个module的列表，选择 ```:libraries:opencv``` 。

9. 在 ```/app/src/main/``` 下创建一个 ```jniLibs``` 文件夹，再将 **OpenCV Android SDK** 中 ```sdk/native/libs``` 下的所有文件夹复制到创建的 ```jniLibs``` 目录下。

10. 同步Gradle， 完成配置。

## 参考

[stackoverflow: how-to-use-opencv-in-android-studio-using-gradle-build-tool](http://stackoverflow.com/questions/17767557/how-to-use-opencv-in-android-studio-using-gradle-build-tool/22427267#22427267)

[OpenCV Documentation: OpenCV4Android SDK](http://docs.opencv.org/2.4/doc/tutorials/introduction/android_binary_package/O4A_SDK.html)

[OpenCV Documentation: Android Platform](http://opencv.org/platforms/android.html)
