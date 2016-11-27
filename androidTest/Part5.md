安卓自动化测试入门-5-创建UI
================

在这个系列的博客中，我们新建了一个叫做[Github User Search](https://github.com/riggaroo/GithubUsersSearchApp)的Android App范例。在Part1-4的博客中，介绍了[为什么我们应该写测试](http://blog.csdn.net/jaychen2011/article/details/52712130)， [如何为了测试而配置项目](http://blog.csdn.net/jaychen2011/article/details/52723025)， [创建API调用](http://blog.csdn.net/jaychen2011/article/details/52735028) 和 [创建一个Presenter](http://blog.csdn.net/jaychen2011/article/details/52947368) 。请看一看前面的博客，因为Part5将是这个系列的延续。

>原文[Part 1](https://riggaroo.co.za/introduction-automated-android-testing/), [Part 2](https://riggaroo.co.za/automated-android-testing-part-2-setup/), [Part 3](https://riggaroo.co.za/introduction-android-testing-part3/) 和 [Part4](https://riggaroo.co.za/introduction-android-testing-part-4/) 。

在Part5中，我们将会了解如何与Part4创建的Presenter交互，同时我们将会创建一个展示搜索结果列表的UI。

> 本文翻译自Riggaroo的《 [Introduction to Automated Android Testing – Part 5](https://riggaroo.co.za/introduction-automated-android-testing-part-5/) 》 
注意：以下的测试特指“程序员编写的自动化代码测试” 
水平有限，欢迎指教。如有错漏，多多包涵。 
作者的项目地址： 
https://github.com/riggaroo/GithubUsersSearchApp。 
请注意：每个分支对应这一系列博客的每一篇文章。


创建UI
----

关于用户交互界面，我们想要一个简单的列表用于显示每个用户的头像，姓名和一些用户的其它信息。

在[Part4](https://riggaroo.co.za/introduction-android-testing-part-4/)中，我们定义了一个Activity应该实现的`View`约定。这就是编写Android特有代码的地方（例如某个控件的可见性改变，或者任何UI的变更都应该写在这里）。重温一下View约定的定义：
```
interface UserSearchContract {

    interface View extends MvpView {
        void showSearchResults(List<User> githubUserList);

        void showError(String message);

        void showLoading();

        void hideLoading();
    }

    interface Presenter extends MvpPresenter<View> {
        void search(String term);
    }
}
```
让我们来实现这个View吧！

1 . 创建或导航到`UserSearchActivity`。这个类将实现`UserSearchContract.View`约定并继承`AppCompatActivity`。定义一个UserSearchContract.Presenter类型的变量`userSearchPresenter`。这个就是我们用于调用网络访问的对象。
```
public class UserSearchActivity extends AppCompatActivity implements UserSearchContract.View {

    private UserSearchContract.Presenter userSearchPresenter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_user_search);
         userSearchPresenter = new UserSearchPresenter(Injection.provideUserRepo(), Schedulers.io(),
                AndroidSchedulers.mainThread());
        userSearchPresenter.attachView(this);

    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        userSearchPresenter.detachView();
    }

    @Override
    public void showSearchResults(List<User> githubUserList) {
        
    }

    @Override
    public void showError(String message) {
     
    }

    @Override
    public void showLoading() {
        
    }

    @Override
    public void hideLoading() {
    }
}
```
在`onCreate()`中，创建presenter对象。将Injection类定义的User Repo作为第一个参数。（译者注：有跟我一样喜欢即时拷代码进项目看的朋友吗？我就当有了，我把原文的Injection代码提前到下面。）传递`ios()`和`AndroidSchedulers.mainThread()`计划进构造器，这样RxJava的Subscription就知道应该在哪条线程上面执行自己的代码了。

在下一行，你可以看到我调用`userSearchPresenter.attachView(this)`。这个操作将View依附到Presenter，这样Presenter就可以将变动通知给View。因为Presenter并不会自动与Activity的生命周期联动，所以在`onDestroy()`中我们需要通知Presenter这时View已经不存在了，具体做法是调用`userSearchPresenter.detachView()`。这样就可以注销RxJava的所有订阅并防止内存泄露。
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

2 . 在layout文件夹创建`activity_user_search.xml`。这个文件将会包含一个`RecyclerView`，一个`ProgressBar`，一个错误`TextView`和一个`Toolbar`。我准备使用`ConstraintLayout`来设计我的屏幕，所以我不会很详细地述说细节，因为大部分操作都是拖和放。（如果你想要知道更多ConstraintLayout的信息，你可以打开我的 [另一篇博客](https://riggaroo.co.za/constraintlayout-101-new-layout-builder-android-studio/) 。）

![UserSearchActivity](http://img.blog.csdn.net/20161127155428137)

activity_user_search.xml
```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_user_search"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="za.co.riggaroo.gus.presentation.search.UserSearchActivity">

    <android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:background="?attr/colorPrimary"
        android:minHeight="?attr/actionBarSize"
        android:theme="?attr/actionBarTheme"
        app:layout_constraintHorizontal_bias="0.0"
        app:layout_constraintLeft_toLeftOf="@+id/activity_user_search"
        app:layout_constraintRight_toRightOf="@+id/activity_user_search"
        app:layout_constraintTop_toTopOf="@+id/activity_user_search">

    </android.support.v7.widget.Toolbar>

    <android.support.v7.widget.RecyclerView
        android:id="@+id/recycler_view_users"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_marginBottom="16dp"
        android:clipToPadding="false"
        android:scrollbars="vertical"
        app:layoutManager="android.support.v7.widget.LinearLayoutManager"
        app:layout_constraintBottom_toBottomOf="@+id/activity_user_search"
        app:layout_constraintLeft_toLeftOf="@+id/activity_user_search"
        app:layout_constraintRight_toRightOf="@+id/activity_user_search"
        app:layout_constraintTop_toBottomOf="@+id/toolbar"
        tools:listitem="@layout/list_item_user">

    </android.support.v7.widget.RecyclerView>

    <TextView
        android:id="@+id/text_view_error_msg"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="@string/search_for_some_users"
        android:visibility="visible"
        app:layout_constraintBottom_toBottomOf="@+id/recycler_view_users"
        app:layout_constraintLeft_toLeftOf="@+id/toolbar"
        app:layout_constraintRight_toRightOf="@+id/recycler_view_users"
        app:layout_constraintTop_toBottomOf="@+id/toolbar"
        tools:text="No Data has loaded"/>

    <ProgressBar
        android:id="@+id/progress_bar"
        style="@style/Widget.AppCompat.ProgressBar"
        android:layout_width="60dp"
        android:layout_height="60dp"
        android:layout_marginBottom="16dp"
        android:layout_marginTop="16dp"
        android:visibility="gone"
        app:layout_constraintBottom_toBottomOf="@+id/activity_user_search"
        app:layout_constraintLeft_toLeftOf="@+id/recycler_view_users"
        app:layout_constraintRight_toRightOf="@+id/recycler_view_users"
        app:layout_constraintTop_toBottomOf="@+id/toolbar"
        tools:visibility="visible"/>

</android.support.constraint.ConstraintLayout>
```

strings.xml
```
<resources>
    <string name="app_name">Gus</string>
    <string name="search_users">Search for users on Github...</string>
    <string name="search_icon_title">Search</string>
    <string name="search_for_some_users">Start typing to search</string>
</resources>

```

3 . 我们还需要添加一个SearchView到ToolBar，这样我们就有地方键入搜索词。































