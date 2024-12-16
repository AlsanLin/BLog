### Launcher源码分析<!-- docsify ingore -->

#### 一、Launcher介绍

- Launcher是Android系统启动后，由SystemServer ActivityManagerService(AMS)加载的第一个应用程序
- 负责Android桌面的启动和管理
- 用户使用其他App的入口

#### 二、下载及编译

**下载**

- 使用Git下载Launcher源码

```
git clone https://android.googlesource.com/platform/packages/apps/Launcher3
或
git clone git://mirrors.ustc.edu.cn/aosp/platform/packages/apps/Launcher3.git
```

- 进入项目目录

```
cd Launcher3
```

- 切换到Android15分支

```
git checkout android15-release
```

**编译**

使用AndroidStudio如何编译下载好的Launcher3工程

1. 新建一个AndroidStudio项目，编译并能正常运行；
2. 将Launcher3项目复制到该项目根目录下；
3. 将Launcher3源码配置到自己的项目中；

```
android {
    
	...

    sourceSets {
        main {
            res.srcDirs = ['src/main/res', '../Launcher3/res']
            main.java.srcDirs = ['src/main/java', '../Launcher3/src']
        }
    }
}
```

4. 编译运行，然后解决报错问题，如下：

**问题一：**

```
		   Dependency 'androidx.appcompat:appcompat-resources:1.7.0' requires libraries and applications that
           depend on it to compile against version 34 or later of the
           Android APIs.
     
           :app is currently compiled against android-33.
     
           Also, the maximum recommended compile SDK version for Android Gradle
           plugin 7.4.2 is 33.
```

**解决**：将app 模块下build.gradle中compileSdk版本改成34；

```
android {
    namespace 'com.demo.customlauncher'
    compileSdk 34

    defaultConfig {
        applicationId "com.demo.customlauncher"
        minSdk 23
        targetSdk 34
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
	...
}
```

**问题二：** Error while processing xxx\customLauncher\Launcher3\res\drawable\ic_private_profile_app_scroller_badge.xml : Invalid color value ?android:attr/textColorPrimaryInverse

**解决**：将该属性替换成其他对应颜色就行

**问题三：** 依赖下载错误

```
A problem occurred configuring root project ‘My Application’.
Could not resolve all artifacts for configuration ‘:classpath’.
Could not find j2objc-annotations-2.8.jar (com.google.j2objc:j2objc-annotations:2.8).
Searched in the following locations:
```

**原因：** jcenter逐步废弃，

**解决：** 在对应的build.gradle中将jcenter和相关的镜像去掉即可



#### 三、Launcher结构

1. 入口：应用是如何拉起的，如何加载桌面的各个组件
2. WorkSpace：该组件是如何绘制的，如果添加快捷方式(AppIcon)、Hotseat、Widget小组件等
3. AllApp：应用入口；启动桌面后，AppIcon是如何添加到桌面上的；安装新应用后，桌面是怎么更新的；点击App图标，是如何启动该App的
4. SearchDropTargetBar：搜索框
5. Hotseat：底部快捷图标
6. Widget：小组件，比如天气、闹钟等
7. 桌面数据集加载LauncherSettings：launcher.db

#### 四、源码解析

1. 在项目根目录的AndroidManifest.xml定义Launcher作为桌面程序的属性

```
<application>
    <activity
        android:name="com.android.launcher3.Launcher"
        android:launchMode="singleTask">
        <intent-filter>
			<!--告诉系统这是一个启动器-->
            <category android:name="android.intent.category.HOME" />
        </intent-filter>
    </activity>
</application>
```

- android.intent.category.HOME: 告诉系统这是一个启动器（Launcher）应用程序，系统在初始化完成后会通过ActivityTaskManagerService的getHomeIntent方法获取和启动桌面程序。

- 开发人员也可以自己开发一个桌面程序(如微软桌面)，用户安装完成后，可以在系统设置中修改默认启动的桌面程序

2. Launcher.java

Launcher.java是Launcher的启动页面，负责资源初始化和桌面UI创建，Launcher被AMS拉起后，进入自己的生命周期流程，在Launcher.java中onCreate被调用，准备开始加载桌面

#### 五、参考链接

> 1. [Launcher3桌面图标加载流程](https://blog.csdn.net/zxc024000/article/details/134757039)
> 2. [Launcher3源码上实现新应用安装后自动显示到桌面](https://juejin.cn/post/7281486153415917583)
