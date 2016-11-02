安卓自动化测试入门-4-Presenter的单元测试
==========================

在这个系列的博客中，我们新建了一个叫做[Github User Search](https://github.com/riggaroo/GithubUsersSearchApp)的Android App范例。在前面的博客中，我们了解了如何为了测试而配置项目，创建API调用并为API数据转换写了第一个单元测试。查看[Part 1](http://blog.csdn.net/jaychen2011/article/details/52712130)， [Part 2](http://blog.csdn.net/jaychen2011/article/details/52723025) 和 [Part 3](http://blog.csdn.net/jaychen2011/article/details/52735028)。

>原文[Part 1](https://riggaroo.co.za/introduction-automated-android-testing/)，[Part 2](https://riggaroo.co.za/automated-android-testing-part-2-setup/) 和 [Part 3](https://riggaroo.co.za/introduction-android-testing-part3/)。

这篇博客将会带你了解如何创建一个Presenter，用来和repository通信并传输数据到View层。也同样会为Presenter编写单元测试。源码可从Github检出，[点击这里](https://github.com/riggaroo/GithubUsersSearchApp)。

> 本文翻译自Riggaroo的《Introduction to Android Testing – Part 4》 
注意：以下的测试特指“程序员编写的自动化代码测试” 
水平有限，欢迎指教。如有错漏，多多包涵。 
作者的项目地址： 
https://github.com/riggaroo/GithubUsersSearchApp。 
请注意：每个分支对应这一系列博客的每一篇文章。
