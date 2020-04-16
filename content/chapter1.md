## 第一章 Android与Android Studio

### 1.Android系统重要概念

* 架构 自底向上：

| Linux内核 硬件驱动                        |
| ----------------------------------------- |
| 系统运行库层 SQLite OpenGL Webkit Runtime |
| 应用框架层 系统API能力                    |
| 应用层                                    |

* 四大组件

  Activity、Service、Broadcast Receiver（电话、短信等）、Content Provider（应用程序间共享数据）

### 2.Android Studio的项目组织架构

（切换成Project模式下的真实目录）

####  .gradle 和 .idea

AS自动生成的文件，无需关心

#### app

项目中的代码、资源等，开发工作基于此目录

#### build

编译时自动生成的文件

#### gradle

gradle wrapper配置文件

#### .gitignore

#### build.gradle

项目全局的构建脚本，~~通常不需要修改~~

```
 dependencies {
        classpath 'com.android.tools.build:gradle:3.5.1'
        classpath 'com.google.ar.sceneform:plugin:1.15.0'}
```



#### gradle.properties

全局的gradle构建脚本，这里配置的脚本会影响到所有gradle编译脚本

例如：

```
android.useAndroidX=true
```

#### gradlew与gradle.bat

用于命令行中执行gradle命令。前者用于unix-like，后者用于win

#### [项目名].iml

标识这是一个IntelliJ IDEA项目

#### local.properities

指定本机的Android SDK路径

```
sdk.dir=D\:\\AndroidSDK
```

#### setting.gradle

指定项目中引入的所有模块，如只引入了`app`模块

```
rootProject.name='ScenceFormDemo'
include ':app'
```

### 3.app目录下的内容及作用

#### build

编译文件

#### libs

第三方jar包，其下的jar包会自动添加进构建路径中

#### androidTest

存放AndroidTest自动化测试用例

#### java

java代码

#### res

资源，包括了

* drawable 图片资源

  * mipmap开头的文件夹用于保存应用图标(适配各种分辨率)

    若没有指定，建议放在`mipmap-xxhdpi`中

* layout 布局资源

* values 字符串资源 等

  * 在代码中`R.String.stringName`实现引用字符串
  * 在XML在`@String/stringName`实现引用字符串

#### AndroidManifest.xml

整个安卓项目的配置文件,四大组件,权限声明等

#### test

Unit Test测试

#### app.iml

#### build.gradle

app模块的构建脚本

* defaultconfig--> targetSdkVersion: 系统将在指定的版本号开放该版本的特有功能vv

* buildTypes -> release -> minifyEnable 是否开启混淆

* dependences 的依赖分为三种

  * compile fileTree 本地依赖

  * implementation 远程依赖   `域名:组名:版本`

    ```
     'androidx.appcompat:appcompat:1.1.0'
    ```

#### proguard-rules.pro

指定代码的混淆规则,难以破解安装包文件的源代码

### 4.注册活动及声明主活动

任何活动都要到AndroidManifest.xml中进行注册

注册主活动

```xml
<activity android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

**所有活动都要继承Activity类**

一般继承`AppCompatActivity`类, 是`Activity`的子类这是一种实现最低兼容到Android 2.1的向下兼容的Activity

**补充**：

activity中可以通过android：theme属性指定主题：

```xml
<activity android:name=".MainActivity"
    android:theme="@style/Theme.AppCompat.Dailog"
</activity>
```

**指定了主题是Dailog对话框式主题**

### 5.Log工具

格式` Log.x(tag, msg)` x见下  tag用于过滤,建议使用类名, msg填写具体信息                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      

* `Log.v()`  打印verbose 是按照日志中最低的一级

* `Log.d()`  打印debug

* `Log.i()`  打印info

* `Log.w()`  打印warm

* `Log.e()`  打印error 严重问题

  在logcat中设置了级别时,只会向上显示相应的语句.

  如Log.v()语句 不会被logcat中的error条件显示

  