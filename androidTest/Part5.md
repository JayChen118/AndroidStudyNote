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

3 . 我们还需要添加一个SearchView到ToolBar，这样我们就有地方键入搜索词。添加一个`menu_user_search.xml`文件到menu资源文件夹，在文件里面，我们添加一个`SearchView`：
menu_user_search.xml
```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/menu_search"
        android:icon="@drawable/ic_search"
        android:title="@string/search_icon_title"
        app:actionViewClass="android.support.v7.widget.SearchView"
        app:showAsAction="always|collapseActionView" />
</menu>
```
添加一个`ic_search.xml`文件到drawable文件夹：
```
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportHeight="24.0"
    android:viewportWidth="24.0">
    <path
        android:fillColor="#FFFFFF"
        android:pathData="M15.5,14h-0.79l-0.28,-0.27C15.41,12.59 16,11.11 16,9.5 16,5.91 13.09,3 9.5,3S3,5.91 3,9.5 5.91,16 9.5,16c1.61,0 3.09,-0.59 4.23,-1.57l0.27,0.28v0.79l5,4.99L20.49,19l-4.99,-5zM9.5,14C7.01,14 5,11.99 5,9.5S7.01,5 9.5,5 14,7.01 14,9.5 11.99,14 9.5,14z" />
</vector>

```

4 . 我们需要为RecyclerView的每一个项创建一个layout。新建一个`list_item_user.xml`文件。我使用ConstraintLayout，包含一个显示头像的ImageView和两个TextView。

![List_item_user_designmode](http://img.blog.csdn.net/20161129165818478)

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/constraintLayout"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">

    <ImageView
        android:id="@+id/imageview_userprofilepic"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:layout_marginLeft="16dp"
        android:layout_marginStart="16dp"
        android:layout_marginTop="16dp"
        app:layout_constraintLeft_toLeftOf="@+id/constraintLayout"
        app:layout_constraintTop_toTopOf="@+id/constraintLayout"
        app:srcCompat="@mipmap/ic_launcher" />

    <TextView
        android:id="@+id/textview_username"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="16dp"
        android:layout_marginStart="16dp"
        android:layout_marginTop="16dp"
        android:textAppearance="@style/TextAppearance.AppCompat.Medium"
        app:layout_constraintLeft_toRightOf="@+id/imageview_userprofilepic"
        app:layout_constraintTop_toTopOf="@+id/constraintLayout"
        tools:text="Rebecca Franks" />

    <TextView
        android:id="@+id/textview_user_profile_info"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginEnd="16dp"
        android:layout_marginLeft="16dp"
        android:layout_marginRight="16dp"
        android:layout_marginStart="16dp"
        android:textAppearance="@style/TextAppearance.AppCompat.Caption"
        app:layout_constraintBottom_toBottomOf="@+id/constraintLayout"
        app:layout_constraintLeft_toRightOf="@+id/imageview_userprofilepic"
        app:layout_constraintRight_toRightOf="@+id/constraintLayout"
        app:layout_constraintTop_toBottomOf="@+id/textview_username"
        tools:text="JHB, South Africa. Lots of code, lots and lots and lots of code." />
</android.support.constraint.ConstraintLayout>
```

5 . 现在我们已经有了所有需要的layout，让我们把它们绑定到Activity吧。首先，在`onCreate()`方法获取View的引用。
```
    private UsersAdapter usersAdapter;
    private SearchView searchView;
    private Toolbar toolbar;
    private ProgressBar progressBar;
    private RecyclerView recyclerViewUsers;
    private TextView textViewErrorMessage;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...

        toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
        progressBar = (ProgressBar) findViewById(R.id.progress_bar);
        textViewErrorMessage = (TextView) findViewById(R.id.text_view_error_msg);
        recyclerViewUsers = (RecyclerView) findViewById(R.id.recycler_view_users);
        usersAdapter = new UsersAdapter(null, this);
        recyclerViewUsers.setAdapter(usersAdapter);

    }
```

添加RecyclerView的依赖，版本与compileSdkVersion匹配就好
```
    compile 'com.android.support:recyclerview-v7:25.0.1'
```

添加UsersAdapter，用于RecyclerView。
```

public class UsersAdapter extends RecyclerView.Adapter<UserViewHolder> {
    private final Context context;
    private List<User> items;

    UsersAdapter(List<User> items, Context context) {
        this.items = items;
        this.context = context;
    }

    @Override
    public UserViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View v = LayoutInflater.from(parent.getContext()).inflate(R.layout.list_item_user, parent, false);
        return new UserViewHolder(v);
    }

    @Override
    public void onBindViewHolder(UserViewHolder holder, int position) {
        User item = items.get(position);

        holder.textViewBio.setText(item.getBio());
        if (item.getName() != null) {
            holder.textViewName.setText(item.getLogin() + " - " + item.getName());
        } else {
            holder.textViewName.setText(item.getLogin());
        }
        Picasso.with(context).load(item.getAvatarUrl()).into(holder.imageViewAvatar);
    }

    @Override
    public int getItemCount() {
        if (items == null) {
            return 0;
        }
        return items.size();
    }

    void setItems(List<User> githubUserList) {
        this.items = githubUserList;
        notifyDataSetChanged();
    }
}


class UserViewHolder extends RecyclerView.ViewHolder {
    final TextView textViewBio;
    final TextView textViewName;
    final ImageView imageViewAvatar;

    UserViewHolder(View v) {
        super(v);
        imageViewAvatar = (ImageView) v.findViewById(R.id.imageview_userprofilepic);
        textViewName = (TextView) v.findViewById(R.id.textview_username);
        textViewBio = (TextView) v.findViewById(R.id.textview_user_profile_info);
    }
}
```

6 . 我们需要将SearchView勾到Activity里面并让它触发presenter的`search()`方法。在`onCreateOptionsMenu()`里面，加上以下代码：
```
@Override
    public boolean onCreateOptionsMenu(Menu menu) {
        super.onCreateOptionsMenu(menu);
        getMenuInflater().inflate(R.menu.menu_user_search, menu);
        final MenuItem searchActionMenuItem = menu.findItem(R.id.menu_search);
        searchView = (SearchView) searchActionMenuItem.getActionView();
        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            @Override
            public boolean onQueryTextSubmit(String query) {
                if (!searchView.isIconified()) {
                    searchView.setIconified(true);
                }
                userSearchPresenter.search(query);
                toolbar.setTitle(query);
                searchActionMenuItem.collapseActionView();
                return false; 
            }

            @Override
            public boolean onQueryTextChange(String s) {
                return false;
            }
        });
        searchActionMenuItem.expandActionView();
        return true;
    }
```
这段代码将会填充正确的菜单，找到搜索视图并设置一个文本查询监听器。在这种情形下，只有当用户点击键盘上的提交按钮，我们才会做出反应，调用presenter的搜索方法。我们也可以在`onQueryTextChange`方法里面做这件事，不过考虑到Github API的调用频率限制，我还是建议用`onQueryTextSubmit`。正常情况下，搜索结果将会展现。

7 . 接下来，我们实现presenter将会在数据加载完成后调用的回调。
```
    @Override
    public void showSearchResults(List<User> githubUserList) {
        recyclerViewUsers.setVisibility(View.VISIBLE);
        textViewErrorMessage.setVisibility(View.GONE);
        usersAdapter.setItems(githubUserList);
    }

    @Override
    public void showError(String message) {
        textViewErrorMessage.setVisibility(View.VISIBLE);
        recyclerViewUsers.setVisibility(View.GONE);
        textViewErrorMessage.setText(message);
    }

    @Override
    public void showLoading() {
        progressBar.setVisibility(View.VISIBLE);
        recyclerViewUsers.setVisibility(View.GONE);
        textViewErrorMessage.setVisibility(View.GONE);
    }

    @Override
    public void hideLoading() {
        progressBar.setVisibility(View.GONE);
        recyclerViewUsers.setVisibility(View.VISIBLE);
        textViewErrorMessage.setVisibility(View.GONE);

    }
```
我们基本上只是反转视图的可见性并给`userAdapter`设置网络服务返回的新数据。

译者注：如果遇到Toolbar相关的报错`This Activity already has an action bar supplied by the window decor. Do not request Window.FEATURE_SUPPORT_ACTION_BAR and set windowActionBar to false in your theme to use a Toolbar instead.`，应该是Theme的设置没有关闭默认的action bar。Theme的代码如下：
```
<resources>

    <!-- Base application theme. -->
    <style name="MyTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
    </style>

</resources>
```
译者注：记得添加网络访问权限
```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="za.co.riggaroo.gus">

    <uses-permission android:name="android.permission.INTERNET" />
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/MyTheme">

        <activity android:name=".presentation.search.UserSearchActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

8 . 现在你可以跑一下这个App了。你应该可以搜索一个Github的用户名并看到结果。
![github_user_search](http://img.blog.csdn.net/20161130151623382)

Yay！我们有了一个能工作的App了。这篇博客的代码可以在 [这里](https://github.com/riggaroo/GithubUsersSearchApp/tree/testing-tutorial-part5-complete) 找到。在下一篇，我们将会介绍如何编写UI的测试。


