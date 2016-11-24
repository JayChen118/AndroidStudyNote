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
在`onCreate()`中，创建presenter对象。将Injection类定义的User Repo作为第一个参数。（译者注：有跟我一样喜欢即时拷代码进项目看的朋友吗？我就当有了，我把原文的Injection代码提前到下面。）
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







































