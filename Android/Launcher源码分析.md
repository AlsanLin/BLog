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

#### 三、Launcher结构

1. 入口
2. WorkSpace(工作去)
3. SearchDropTargetBar
4. Hotseat
5. AppsView
6. Widget
7. 点击App Icon启动应用

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

Launcher.java是Launcher的启动页面，负责资源初始化和桌面UI创建

#### 五、参考链接

> 1. [Launcher3桌面图标加载流程](https://blog.csdn.net/zxc024000/article/details/134757039)
> 2. [Launcher3源码上实现新应用安装后自动显示到桌面](https://juejin.cn/post/7281486153415917583)
