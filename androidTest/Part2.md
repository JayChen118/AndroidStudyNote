#安卓自动化测试入门-2-配置项目

>本文翻译自Riggaroo的[《Introduction to Automated Android Testing – Part 2 – Setup》](https://riggaroo.co.za/automated-android-testing-part-2-setup/)
>注意：以下的测试特指“程序员编写的自动化代码测试”
>水平有限，欢迎指教。如有错漏，多多包涵。
>作者的项目地址：
>https://github.com/riggaroo/GithubUsersSearchApp。
>请注意：每个分支对应这一系列博客的每一篇文章。

在这一系列博客中，我们将会介绍安卓的自动化测试。在[Part 1](http://blog.csdn.net/jaychen2011/article/details/52712130)中，我们了解了为什么要写测试，测试代码放在哪里和安卓中有几种类型的代码测试。

在这篇文章中，我们将会把项目配置成典型的架构以便进行测试。我将会根据本系列博客的草稿创建一个例子项目，并演示我设想中的每一个步骤。我们将会创建的例子是一个通过Github API搜索用户的App。下图是App的简单原型：

![SampleApp-Android_Automated_Testing](http://img.blog.csdn.net/20161002102535013)

我们将会从0开始。如果你跟着每个步骤做，你应该能得到一个[这样子](https://github.com/riggaroo/GithubUsersSearchApp/tree/testing-tutorial-part2-complete)的项目。

##新建一个App项目
1. 打开Android Studio，选择“Start a new Android Project”.
2. 将项目命名为“Gus”（Github User Search），将域名定义为“riggaroo.co.za”。这样将会创建一个包名 - `za.co.riggaroo.gus`。点击“Next”。
![Step1-AutomatedAndroidTestProject](http://img.blog.csdn.net/20161002111715332)
3. 接下来选择要支持的Android版本号（我一般不会选择低于API 16，选择API 16，你可以获得95.2%的覆盖率）。选择API 16并点击“Next”。![Step2-AutomatedAndroid_Test_Project](http://img.blog.csdn.net/20161002151402483)
4. 选择“Empty Activity”并点击“Next”。![Step3-AutomatedTestProjectAndroid](http://img.blog.csdn.net/20161002151640498)
5. 修改Activity的名字为`UserSearchActivity`，修改layout的名字为`activity_user_search`。点击“Finish”。![Step4-UserActivityName](http://img.blog.csdn.net/20161002151945455)
6. 如果你运行App - 你会看到一个空白的activity，如下：![Step4-BlankApp](http://img.blog.csdn.net/20161002152206645)

##添加测试依赖

导航到你的app中的`build.gradle`文件并添加以下依赖。需要添加的依赖包括 [Espresso](https://google.github.io/android-testing-support-library/downloads/), [Mockito](http://mockito.org/), [PowerMock](https://github.com/jayway/powermock) 和 [Hamcrest](http://hamcrest.org/)。[Retrofit](http://square.github.io/retrofit/), [OkHttp](http://square.github.io/okhttp/), [RxJava](https://github.com/ReactiveX/RxJava) 和 [RxAndroid](https://github.com/ReactiveX/RxAndroid) 也被添加，它们可以帮忙实现高效的网络访问以及更整洁的代码。

    dependencies {
        compile fileTree(dir: 'libs', include: ['*.jar'])

        compile 'com.squareup.okhttp3:logging-interceptor:3.4.0-RC1'
        compile 'com.android.support:appcompat-v7:24.2.1'
        compile 'com.android.support.constraint:constraint-layout:1.0.0-alpha8'

        //Retrofit, RxJava and OkHttp.
        compile 'com.squareup.retrofit2:retrofit:2.1.0'
        compile 'com.squareup.retrofit2:converter-gson:2.1.0'
        compile 'com.squareup.okhttp3:okhttp:3.4.0-RC1'
        compile 'com.squareup.retrofit2:adapter-rxjava:2.1.0'
        compile 'io.reactivex:rxandroid:1.2.1'
        compile 'io.reactivex:rxjava:1.1.6'
        compile 'com.squareup.picasso:picasso:2.5.2'

        //Dependencies for JUNit and unit tests.
        testCompile "junit:junit:4.12"
        testCompile "org.mockito:mockito-all:1.10.19"
        testCompile "org.hamcrest:hamcrest-all:1.3"
        testCompile("org.powermock:powermock-module-junit4:1.6.2")
        testCompile("org.powermock:powermock-api-mockito:1.6.2")
        testCompile 'com.squareup.okhttp3:mockwebserver:3.4.0-RC1'

        //Dependencies for Espresso
        androidTestCompile 'com.android.support:appcompat-v7:24.2.1'
        androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
            exclude group: 'com.android.support', module: 'support-annotations'
        })
        androidTestCompile("com.android.support.test:runner:0.5") {
            exclude module: 'support-annotations'
            exclude module: 'support-v4'
        }
        androidTestCompile("com.android.support.test:rules:0.5") {
            exclude module: 'support-annotations'
            exclude module: 'support-v4'
        }
        androidTestCompile("com.android.support.test.espresso:espresso-intents:2.2.2") {
            exclude module: 'recyclerview-v7'
            exclude module: 'support-annotations'
            exclude module: 'support-v4'
        }
        androidTestCompile('com.android.support.test.espresso:espresso-contrib:2.2.1') {
            exclude module: 'recyclerview-v7'
            exclude module: 'support-annotations'
            exclude module: 'support-v4'

        }
    }

>译者注：在我翻译的时候，有些依赖包已经更新了版本，所以跟原文会有些许出入。如果出现“Failed to resolve：包名”，点击“Install artifact and sync project”下载安装依赖包，应该可以解决。其它的依赖包冲突一般更新为最新版本都可解决。例如，我将原文中的`appcompat-v7:24.1.1`修改为`appcompat-v7:24.2.1`。

`testCompile`是配置单元测试的依赖（位于`src/test`）。而`androidTestCompile`则是用来配置instrumentation测试（位于`src/androidTest`）。两者的不同之处你可以在本系列的[第一篇博客](http://blog.csdn.net/jaychen2011/article/details/52712130)中找到。

##使用Gradle Build Flavors激活Mocking

为了能轻松搞笑地测试UI，我们不会直接调用Github的生产环境API。我们将会mock响应，并模拟不同的网络情况。想达到这样的目的有几种方式，我将会展示的是使用[Gradle Product Flavors](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Product-flavors)。

Flavors允许你为App构建不同的版本，不同版本之间可以有不同的源代码或资源文件。例如，当你想要一个免费版本和一个拥有更多功能的付费版本，使用Product Flavors会是一个绝佳的实现方式。

在当前的情况下，我们要创建一个“Production”和一个“Mock”flavor。如果我们需要App指向某个展示环境，我们也可以添加一个“Staging”flavor。这样我们就可以同时安装production版本和mock版本的app。

1. 为了使用`productFlavor`，导航到app的`build.gradle`文件并添加以下代码进`android{}`区间。下面的代码意味着，当变量为`mock`时，applicationId的值会不同于`prod`版本，这样就可以同时安装在同一台设备上。`prod`版本的applicationId的值将会从`defaultConfig`设置获取。

        productFlavors {
            prod {

            }
            mock {
                applicationId "za.co.riggaroo.gus.mock"
            }
        }

2. 执行Gradle Sync，然后在IDE的左边你应该可以看到“Build Variants”tab。打开它，你就可以选择不同的flavors。![Gradle_Build_Variants_Android_Studio](http://img.blog.csdn.net/20161003171347454)

如果你选择不同的variant，当你点击“run”去编译运行项目，这时部署到你的设备的将会是与variant相关的一套源码及`applicationId`。

##运行单元测试

想运行已经存在于项目里面的默认的单元测试（`ExampleUnitTest`），可以用下面两种方法：

- 在Android Studio里面导航到`app/src/test/java`文件夹，右键点击并选择弹出菜单的“Run Tests”。![Running_Unit_Tests_with_JUnit_in_Android_Studio](http://img.blog.csdn.net/20161003172311854)
- 使用Gradle ： 在terminal窗口运行 `./gradlew check`
>译者注：Windows下可能需要使用反斜杠`.\gradlew check'

##运行Instrumentation测试（需要真机或模拟器）

想运行已经存在于项目里面的默认的instrumentation测试（`ExampleInstrumentationTest`）,可以有下面两种方法：

 - 在Android Studio里面导航到`app/src/androidTest/java`文件夹，右键点击并选择弹出菜单的“Run All Tests”。![Run_Connected_tests_android_studio](http://img.blog.csdn.net/20161003204522539)
 - 使用Gradle ： 在terminal窗口运行 `./gradlew connectedAndroidTest`
>译者注：Windows下可能需要使用反斜杠`.\gradlew connectedAndroidTest'

##如何构建你的代码，让它便于测试

为了能够方便地写测试代码，我准备使用[依赖注入](https://en.wikipedia.org/wiki/Dependency_injection)。你可以在不使用任何框架的情况下实现依赖注入（我经常就是这样干的）。不过也有许多人推荐使用[Dagger2](http://google.github.io/dagger/)。本系列博客不准备使用Dagger。

按照下图的包结构新建以下的类：

![SimpleStructureOfAndroidApp](http://img.blog.csdn.net/20161003211312054)

我为APP的不同部分创建了四个顶级的文件夹。如果你的APP更大型的的话，也可以更复杂。下面是我创建的基本的文件夹：

 - `presentation` - 在这个文件夹里面，我创建了子文件夹并根据特性进行分组。在当前的情况下，我创建了一个文件夹并命名为`search`，里面用来放置搜索界面的view跟presenter。它也会包含适配器与其它跟搜索界面相关的视图。
 - `data` - 这里包含用于从Github API获取数据的repositories。
 - `model` - 用于放置用于展现层和从服务访问返回的models。
 - `injection` - 放置用于依赖注入的类。

我们已经了解了如何创建一个简单APP的配置，如何运行不同类型的测试，我们将遵循什么样的架构。这样我们已经朝着一个能轻松为自己的APP写测试的道路前进了。如果你想看看本博客完成的样例，请check out[本博客对应的已完成的代码](https://github.com/riggaroo/GithubUsersSearchApp/tree/testing-tutorial-part2-complete)。

下一篇博客将会对深入对实现特性及测试代码的细节的了解。
《[安卓自动化测试入门-3-网络请求的单元测试](http://blog.csdn.net/jaychen2011/article/details/52735028)》

寻找广州Android开发工程师工作，邮箱hengzhechenjay@163.com 电话：13580579413 陈捷尉 2016.11.22

