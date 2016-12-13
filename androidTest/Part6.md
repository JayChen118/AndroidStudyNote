安卓自动化测试入门-6-Espresso Test
=========================

> 本文翻译自Riggaroo的《 [Introduction to Automated Android Testing – Part 6](https://riggaroo.co.za/introduction-automated-android-testing-part-6/) 》 
注意：以下的测试特指“程序员编写的自动化代码测试” 
水平有限，欢迎指教。如有错漏，多多包涵。 
作者的项目地址： 
https://github.com/riggaroo/GithubUsersSearchApp。 
请注意：每个分支对应这一系列博客的每一篇文章。

在前面5篇博客中，我们覆盖了从草稿建立一个Android App的各个方面的知识。我们专注于在这个过程中编写单元测试。以下是前面几篇博客的链接：

 - [Part1](http://blog.csdn.net/jaychen2011/article/details/52712130) - 为什么我们应该编写测试？
 - [Part2](http://blog.csdn.net/jaychen2011/article/details/52723025) - 配置项目
 - [Part3](http://blog.csdn.net/jaychen2011/article/details/52735028) - 网络请求的单元测试
 - [Part4](http://blog.csdn.net/jaychen2011/article/details/52947368) - Presenter的单元测试
 - [Part5](http://blog.csdn.net/jaychen2011/article/details/53364620) - 创建UI

原文：

 - [Post #1](https://riggaroo.co.za/introduction-automated-android-testing/) – Why should we write tests?
 - [Post #2](https://riggaroo.co.za/automated-android-testing-part-2-setup/) – Set up your app for testing
 - [Post #3](https://riggaroo.co.za/introduction-android-testing-part3/) – Creating API calls
 - [Post #4](https://riggaroo.co.za/introduction-android-testing-part-4/) – Creating repositories
 - [Post #5](https://riggaroo.co.za/introduction-automated-android-testing-part-5/) – Following the MVP pattern

在本系列博客的最后一篇文章里面，我们将会介绍如何为Part5创建的View编写Espresso测试。相关的GitHub资源可以在 [这里](https://github.com/riggaroo/GithubUsersSearchApp) 找到。

当数据是动态的时候，要测试一个View包含预期的确切数据是不现实的。我们的测试不应该因为随时可能变动的数据而不通过。为了让测试变得可靠并可重复，我们不应该直接调用生产环境的API。

通过模拟出API调用的返回值，我们可以编写基于模拟数据的测试。有以下几种方法可以模拟API代用的返回值：

 - **Option 1** - 使用 [WireMock](http://wiremock.org/) 来运行一个独立的服务器，这个服务器提供相同的静态JSON给API调用。
 - **Option 2** - 使用OkHttp的 [MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver) 功能，可以在你的设备上运行一个网络服务器来响应你的网络请求。
 - **Option 3** - 创建Retrofit REST接口的特殊实现，并返回假数据。

显然，如何选择完全是根据你个人的需要及习惯。在我的情况下，WireMock意味着额外的工作量，因为我还需要在一个静态的IP地址上架设一台独立的服务器。

MockWebServer比WireMock容易使用一些，因为你不需要去架设一台独立服务器（服务器运行于你的设备上）。MockWebServer还可以灵活地配置出各种不同的应用场景。还具有一些有用的特性，例如设定某个请求的调用失败频率，或者模拟低网速下的网络请求返回（ [了解更多](https://riggaroo.co.za/retrofit-2-mocking-http-responses/) ）。

我准备使用Option 3，这样可以方便地测试UI显示的数据是否与模拟的响应数据匹配。如果我需要添加低网速的测试（或者一些非功能性的测试），我会选择Option 2。如果你无法使用OkHttp，
可以选择Option 1，因为WireMock可以与任何Http客户端合作。

使用Gradle flavors模拟数据
--------------------

通过使用Gradle flavors我们可以轻易地模拟API返回值。如果你阅读了Part2的Gradle flavors部分，你应该已经有了一个"mock"和一个"production" flavor。

1 . 确保你切换到了mockDebug flavor。
![build_variants_mock_debug](http://img.blog.csdn.net/20161211204752935?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSmF5Q2hlbjIwMTE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2 . 在**src**目录下新建一个**mock**文件夹，然后在里面新建一个**java**文件夹，然后在里面新建一个包，包名跟主包名相同(`za.co.riggaroo.gus.data.remote`)。新建一个类，名为` MockGithubUserRestServiceImpl`。最后你的文件目录应该像下图所示：
![resulting_file_structure](http://img.blog.csdn.net/20161211205619731?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSmF5Q2hlbjIwMTE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

3 . 新建一个**prod**文件夹，移动之前定义的`Injection`类到这个文件夹（包也一样）。我们将会创建另一个`Injection`类到**mock**文件夹里面。这个类将会注入**模拟**出来的GitHub服务，而不是生产环境API。