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

2 . 在**src**目录下新建一个**mock**文件夹，然后在里面新建一个**java**文件夹，然后在里面新建一个包，包名跟主包名相同(`za.co.riggaroo.gus.data.remote`)。新建一个类，名为`MockGithubUserRestServiceImpl`。最后你的文件目录应该像下图所示：

![resulting_file_structure](http://img.blog.csdn.net/20161211205619731?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSmF5Q2hlbjIwMTE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

3 . 新建一个**prod**文件夹，移动之前定义的`Injection`类到这个文件夹（包也一样）。我们将会创建另一个`Injection`类到**mock**文件夹里面。这个类将会注入**模拟**出来的GitHub服务，而不是生产环境API。

![prod_mock](http://img.blog.csdn.net/20161213215939639?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSmF5Q2hlbjIwMTE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

在**mock**文件夹里面的`Injection`类中，我们仅仅只是返回之前创建的`MockGithubUserRestServiceImpl`。而在**prod**文件夹中的`Injection`类，我们返回真实的Retrofit GitHub服务。
Mock `Injection` class:
```
public class Injection {

    private static GithubUserRestService userRestService;

    public static UserRepository provideUserRepo() {
        return new UserRepositoryImpl(provideGithubUserRestService());
    }

    static GithubUserRestService provideGithubUserRestService() {
        if (userRestService == null) {
            userRestService = new MockGithubUserRestServiceImpl();
        }
        return userRestService;
    }

}
```
Prod `Injection` class:
```
public class Injection {

    private static final String BASE_URL = "https://api.github.com";
    private static OkHttpClient okHttpClient;
    private static GithubUserRestService userRestService;
    private static Retrofit retrofitInstance;

    public static UserRepository provideUserRepo() {
        return new UserRepositoryImpl(provideGithubUserRestService());
    }

    static GithubUserRestService provideGithubUserRestService() {
        if (userRestService == null) {
            userRestService = getRetrofitInstance().create(GithubUserRestService.class);
        }
        return userRestService;
    }

    static OkHttpClient getOkHttpClient() {
        if (okHttpClient == null) {
            HttpLoggingInterceptor logging = new HttpLoggingInterceptor();
            logging.setLevel(HttpLoggingInterceptor.Level.BASIC);
            okHttpClient = new OkHttpClient.Builder().addInterceptor(logging).build();
        }

        return okHttpClient;
    }

    static Retrofit getRetrofitInstance() {
        if (retrofitInstance == null) {
            Retrofit.Builder retrofit = new Retrofit.Builder().client(Injection.getOkHttpClient()).baseUrl(BASE_URL)
                    .addConverterFactory(GsonConverterFactory.create())
                    .addCallAdapterFactory(RxJavaCallAdapterFactory.create());
            retrofitInstance = retrofit.build();

        }
        return retrofitInstance;
    }
}
```

4 . Mock服务返回的数据依赖于你的特定需要。下面是我的实现：
```
public class MockGithubUserRestServiceImpl implements GithubUserRestService {

    private final List<User> usersList = new ArrayList<>();
    private User dummyUser1, dummyUser2;

    public MockGithubUserRestServiceImpl() {
        dummyUser1 = new User("riggaroo", "Rebecca Franks",
                "https://riggaroo.co.za/wp-content/uploads/2016/03/rebeccafranks_circle.png", "Android Dev");
        dummyUser2 = new User("riggaroo2", "Rebecca's Alter Ego",
                "https://s-media-cache-ak0.pinimg.com/564x/e7/cf/f3/e7cff3be614f68782386bfbeecb304b1.jpg", "A unicorn");
        usersList.add(dummyUser1);
        usersList.add(dummyUser2);
    }

    @Override
    public Observable<UsersList> searchGithubUsers(final String searchTerm) {
        return Observable.just(new UsersList(usersList));
    }

    @Override
    public Observable<User> getUser(final String username) {
        if (username.equals("riggaroo")) {
            return Observable.just(dummyUser1);
        } else if (username.equals("riggaroo2")) {
            return Observable.just(dummyUser2);
        }
        return Observable.just(null);
    }
}
```

在当前情况下，我仅仅只是返回一些假数据。让我们跑一下mock版本的app，我们应该看到不管我们搜索什么，都会返回一样的结果。

![gif-dummydata](http://img.blog.csdn.net/20161214091627672?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSmF5Q2hlbjIwMTE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

Cool！我们现在有了一个可用的假数据App了。现在可以开始写Espresso UI测试了。


编写Espresso测试的基础知识
------------------------------

当我们编写 [Espresso](https://google.github.io/android-testing-support-library/docs/espresso/index.html) 测试，下面的范式用来执行你的UI的功能：
```
onView(withId(R.id.menu_search))      // withId(R.id.menu_search) is a ViewMatcher
  .perform(click())               // click() is a ViewAction
  .check(matches(isDisplayed())); // matches(isDisplayed()) is a ViewAssertion
```

 - **ViewMatcher** - 用于查找一个Activity里面的View。Espresso定义了各种各样的matcher。例如：`withId(R.id.menu_search)`，`withText("Search")`，`withTag("custom_tag")`。

 - **ViewAction** - 用于模拟人与View的交互，点击View什么的。例如：`click()`，`doubleClick()`，`swipeUp()`，` typeText()`。

 - **ViewAssertion** - 用于对View的某些状态做出断言。例如，`doesNotExist()`，`isAbove()`，`isBelow()`。

这里给大家一份PDF版本的Espresso方法小抄： [android-espresso-testing.pdf](https://google.github.io/android-testing-support-library/downloads/espresso-cheat-sheet-2.1.0.pdf) 。值得注意的是传统的 [hamcrest matcher](http://hamcrest.org/JavaHamcrest/) 也能用在Espresso测试中。（译者注：就是用来合并多个matcher形成合集的一些方法，像与、或、非。）例如：`not()`，`allOf()`，`anyOf()`。

编写Espresso UI测试
------------------

如果你可以想起来，我们在 [Part2](http://blog.csdn.net/jaychen2011/article/details/52723025) 已经介绍了Espresso需要的依赖了。现在我们来了解如何编写Espresso测试。

1 . 创建一个**androidTestMock**文件夹。在这个文件夹里面的测试只运行于mock的环境，而不是运行于production的环境。接着创建一个包`za.co.riggaroo.gus.presentation.search`。在包内新建一个类`UserSearchActivityTest`。你的项目结构应该像下图所示：

![AndroidTestMock_folder](http://img.blog.csdn.net/20161214105010710?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSmF5Q2hlbjIwMTE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2 . 我们先从简单的开始，验证当Activity启动后，会显示"Start typing to search"文本。
```
public class UserSearchActivityTest {

    @Rule
    public ActivityTestRule<UserSearchActivity> testRule = new ActivityTestRule<>(UserSearchActivity.class);

    @Test
    public void searchActivity_onLaunch_HintTextDisplayed() {
        //让Activity自启动
        //用户没有进行操作
        //然后
        onView(withText("Start typing to search"))
                .check(matches(isDisplayed()));
    }
}
```
`@Rule`和`ActivityTestRule`指明了这个测试要运行的是哪个Activity。当前这个测试运行的是`UserSearchActivity`。这样就可以自启动`UserSearchActivity`。通过传递额外参数，可以指定是否自启动该Activity。

这个`searchActivity_onLaunch_HintTextDisplayed()`测试相当简单。它搜索包含指定文本的View，并断定这个文本在UI上是可见的。

3 . 下一个测试稍微复杂一些：
```
    @Test
    public void searchText_ReturnsCorrectlyFromWebService_DisplaysResult() {
        //让Activity自启动

        //When
        onView(allOf(withId(R.id.menu_search), withEffectiveVisibility(ViewMatchers.Visibility.VISIBLE))).perform(
                click());  // 当使用SearchView时，会有两个View匹配menu_search id - 一个是图标，另一个是文本框。我们想要点击那个可见的。
        onView(withId(R.id.search_src_text)).perform(typeText("riggaroo"), pressKey(KeyEvent.KEYCODE_ENTER));

        //Then
        onView(withText("Start typing to search")).check(matches(not(isDisplayed())));
        onView(withText("riggaroo - Rebecca Franks")).check(matches(isDisplayed()));
        onView(withText("Android Dev")).check(matches(isDisplayed()));
        onView(withText("A unicorn")).check(matches(isDisplayed()));
        onView(withText("riggaroo2 - Rebecca's Alter Ego")).check(matches(isDisplayed()));
    }
```
在输入文本到`SearchView`之后，点击enter，我们断定假数据会显示在UI上。

4 . 我们已经给正面的场景写了测试。现在我们应该为负面的情况写测试。我们需要调整`MockGithubUserRestServiceImpl`，让它可以返回定制的error observable。
```
    private static Observable dummyGithubSearchResult = null;

    public static void setDummySearchGithubCallResult(Observable result) {
        dummyGithubSearchResult = result;
    }

    @Override
    public Observable<UsersList> searchGithubUsers(final String searchTerm) {
        if (dummyGithubSearchResult != null) {
            return dummyGithubSearchResult;
        }
        return Observable.just(new UsersList(usersList));
    }
```
在上面的代码中，新建了一个可以设置假搜索结果的方法。当调用`searchGithubUsers()`时，如果那个Observable不为null，将返回它。

5 . 现在我们创建一个测试，检查错误信息是否显示在UI上。
```
    @Test
    public void searchText_ServiceCallFails_DisplayError() {
        String errorMsg = "Server Error";
        MockGithubUserRestServiceImpl.setDummySearchGithubCallResult(Observable.error(new Exception(errorMsg)));

        onView(allOf(withId(R.id.menu_search), withEffectiveVisibility(ViewMatchers.Visibility.VISIBLE))).perform(
                click());  // 当使用SearchView时，会有两个View匹配menu_search id - 一个是图标，另一个是文本框。我们想要点击那个可见的。
        onView(withId(R.id.search_src_text)).perform(typeText("riggaroo"), pressKey(KeyEvent.KEYCODE_ENTER));

        onView(withText(errorMsg)).check(matches(isDisplayed()));
    }
```
在这个测试里，我们先确保service返回一个异常，然后我们断定错误信息被显示到UI上。

6 . 让我们运行这些测试：

![Passing_UI_Tests](http://img.blog.csdn.net/20161214155244492?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSmF5Q2hlbjIwMTE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**全部通过了！**

Android代码的覆盖率
------------------

为了知道你写的测试的有效性，进行代码覆盖率度量是很好的做法。

1 . 为了让UI测试的代码覆盖率功能可用，添加`testCoverageEnabled = true`到build.gradle.

```
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        debug {
            testCoverageEnabled = true
        }
    }
```

2 . 代码覆盖率功能目前并不兼容Jack编译器。我们需要切换成 [Retrolambda](https://github.com/evant/gradle-retrolambda) 来获得代码覆盖率报告。相关的分支地址在 [这里](https://github.com/riggaroo/GithubUsersSearchApp/tree/retrolamda-usage-part6) 。
在app的目录下的build.gradle添加以下代码启用Retrolambda。
```
apply plugin: 'me.tatarka.retrolambda'

        /*jackOptions {
            enabled true
        }*/
```

在根目录下的build.gradle添加相关的资源
```
buildscript {
    repositories {
        jcenter()
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.3'
        classpath 'me.tatarka:gradle-retrolambda:3.3.0-beta4'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter()
        mavenCentral()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

3 . 在Terminal中运行任务：**createMockDebugCoverageReport**。你将在这个目录找到HTML报告： **app/build/reports/coverage/mock/debug/index.html.**

Windows下的命令为`.\gradlew createMockDebugCoverageReport`

![mock_debug_android_test](http://img.blog.csdn.net/20161215174447780?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSmF5Q2hlbjIwMTE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

Yay！ - 我们的Mock UI测试有了82%的覆盖率。加上我们在Part4中看到的覆盖率报告，这给了我们一个很不错的基于整个APP的测试评价。现在我们可以重复之前的方式，努力提高我们的代码的测试覆盖率。

结语
----

**哇！我们通过6篇博客完成了测试编写！**很明显，还可以为这个APP编写更多的测试。非功能性的测试，例如测试你的APP在低内存的设备上的表现，或者不稳定的网络状态下的表现。
这个系列到这里就结束了，希望你享受到测试的快乐。如果觉得写得好，请推荐给你的朋友，欢迎订阅博客推送。

原作者Riggaroo博客地址：https://riggaroo.co.za/

译者结语：英文有点菜，翻译得磕磕绊绊的，大家见谅了。本人失业中，有广州的Android工程师招聘的话，麻烦推荐一下我哈。陈捷尉 13580579413 hengzhechenjay@163.com

