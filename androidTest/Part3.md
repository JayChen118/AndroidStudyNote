安卓自动化测试入门-3-网络请求的单元测试
===========

>本文翻译自Riggaroo的[《Introduction to Android Testing – Part 3》](https://riggaroo.co.za/introduction-android-testing-part3/) 
>注意：以下的测试特指“程序员编写的自动化代码测试” 
>水平有限，欢迎指教。如有错漏，多多包涵。
>作者的项目地址：
>https://github.com/riggaroo/GithubUsersSearchApp。
>请注意：每个分支对应这一系列博客的每一篇文章。

在前面两篇文章中，我介绍了如何配置编写测试的先决条件并创建了一个范例APP。接下来我们会在这篇博客中继续开发并编写单元测试。如果你还没看前两篇博客，我建议你还是阅读一下：[part 1](http://blog.csdn.net/jaychen2011/article/details/52712130) 和 [part 2](http://blog.csdn.net/jaychen2011/article/details/52723025)。

>译者注：原文的链接，[Part 1](https://riggaroo.co.za/introduction-automated-android-testing/) 和 [Part 2](https://riggaroo.co.za/automated-android-testing-part-2-setup/)

在这篇博客中，我们将会从Github API获取一个用户列表，并为这个功能编写测试代码。我们将会从[这个repo的这个节点开始](https://github.com/riggaroo/GithubUsersSearchApp/tree/testing-tutorial-part2-complete)。

创建网络服务访问
--------

我们将使用Retrofit和RxJava来访问Github API。我不准备在这个系列介绍它们。如果你不熟悉RxJava，我建议阅读[这些文章](http://blog.danlew.net/2014/09/15/grokking-rxjava-part-1/)。如果之前你没用过Retrofit，我建议阅读[这篇](http://square.github.io/retrofit/)。

为了根据搜索条件获取用户列表，我们需要使用下面的访问点（我将每页数量设置为2是因为API访问有限制）：

	https://api.github.com/search/users?per_page=2&q=rebecca

To get more user information (such as a user’s bio and location), we need to make a subsequent call:
为了获取用户更详细的信息（例如用户的简历和地理位置），我们需要在获取列表的请求后面进行另一个请求：

	https://api.github.com/users/riggaroo

1. 在访问服务之前，我们需要在项目里面创建服务返回的JSON对象。我一般使用这个[线上工具](http://www.jsonschema2pojo.org/)生成。让我们生成下面的两个类：`User` 类和 `UserList` 类。

	    package za.co.riggaroo.gus.data.remote.model;
	    import com.google.gson.annotations.Expose;
	    import com.google.gson.annotations.SerializedName;
    
	    public class User {
            @SerializedName("login")
            @Expose
            private String login;
            @SerializedName("id")
            @Expose
            private Integer id;
            @SerializedName("avatar_url")
            @Expose
            private String avatarUrl;
            @SerializedName("gravatar_id")
            @Expose
            private String gravatarId;
            @SerializedName("url")
            @Expose
            private String url;
            @SerializedName("html_url")
            @Expose
            private String htmlUrl;
            @SerializedName("followers_url")
            @Expose
            private String followersUrl;
            @SerializedName("following_url")
            @Expose
            private String followingUrl;
            @SerializedName("gists_url")
            @Expose
            private String gistsUrl;
            @SerializedName("starred_url")
            @Expose
            private String starredUrl;
            @SerializedName("subscriptions_url")
            @Expose
            private String subscriptionsUrl;
            @SerializedName("organizations_url")
            @Expose
            private String organizationsUrl;
            @SerializedName("repos_url")
            @Expose
            private String reposUrl;
            @SerializedName("events_url")
            @Expose
            private String eventsUrl;
            @SerializedName("received_events_url")
            @Expose
            private String receivedEventsUrl;
            @SerializedName("type")
            @Expose
            private String type;
            @SerializedName("site_admin")
            @Expose
            private Boolean siteAdmin;
            @SerializedName("name")
            @Expose
            private String name;
            @SerializedName("company")
            @Expose
            private Object company;
            @SerializedName("blog")
            @Expose
            private String blog;
            @SerializedName("location")
            @Expose
            private String location;
            @SerializedName("email")
            @Expose
            private Object email;
            @SerializedName("hireable")
            @Expose
            private Object hireable;
            @SerializedName("bio")
            @Expose
            private String bio;
            @SerializedName("public_repos")
            @Expose
            private Integer publicRepos;
            @SerializedName("public_gists")
            @Expose
            private Integer publicGists;
            @SerializedName("followers")
            @Expose
            private Integer followers;
            @SerializedName("following")
            @Expose
            private Integer following;
            @SerializedName("created_at")
            @Expose
            private String createdAt;
            @SerializedName("updated_at")
            @Expose
            private String updatedAt;
        ...
	    }
[点击查看User完整代码](https://github.com/riggaroo/GithubUsersSearchApp/blob/testing-tutorial-part3-complete/app/src/main/java/za/co/riggaroo/gus/data/remote/model/User.java)

        package za.co.riggaroo.gus.data.remote.model;
        import com.google.gson.annotations.Expose;
        import com.google.gson.annotations.SerializedName;
        
        import java.util.ArrayList;
        import java.util.List;
        
        public class UsersList {
        
            @SerializedName("total_count")
            @Expose
            private Integer totalCount;
            @SerializedName("incomplete_results")
            @Expose
            private Boolean incompleteResults;
            @SerializedName("items")
            @Expose
            private List<User> items = new ArrayList<User>();
        
            public UsersList(final List<User> githubUsers) {
                this.items = githubUsers;
            }
        
            /**
             * @return The totalCount
             */
            public Integer getTotalCount() {
                return totalCount;
            }
        
        
            /**
             * @return The incompleteResults
             */
            public Boolean getIncompleteResults() {
                return incompleteResults;
            }
        
            /**
             * @return The items
             */
            public List<User> getItems() {
                return items;
            }
        
        
        }
>译者注：作者在remote包下面新建了model包，原本的顶层model包基本不再使用。

	当这些model被创建之后，导航到`za.co.riggaroo.gus.data.remote`的`GithubUserRestService`（如果没有请新建，后面一样）。这里就是我们创建Retrofit访问的地方。

        package za.co.riggaroo.gus.data.remote;
        import retrofit2.http.GET;
        import retrofit2.http.Path;
        import retrofit2.http.Query;
        import rx.Observable;
        import za.co.riggaroo.gus.data.remote.model.User;
        import za.co.riggaroo.gus.data.remote.model.UsersList;
        
        public interface GithubUserRestService {
        
            @GET("/search/users?per_page=2")
            Observable<UsersList> searchGithubUsers(@Query("q") String searchTerm);
        
            @GET("/users/{username}")
            Observable<User> getUser(@Path("username") String username);
        }
第一个接口会进行用户列表的搜索，第二个接口可以拿到更详细的用户信息。

2. 导航到`za.co.riggaroo.gus.data`的`UserRepositoryImpl`。（译者注：作者貌似忘了`UserRepositoryImpl`实现的接口`UserRepository`，可能还是觉得没必要贴源码，毕竟从实现类可以看出被实现的接口，请记得自己补上。）我们在这里将两个网络接口访问接口融合在一起，并将接口的返回转换成前台View需要使用的数据。先用RxJava根据搜索条件获取一个用户列表，再给列表里的每一个用户发送获取详细信息的接口（译者注：这样就是`用户数+1`个网络请求。）（如果你自己实现了这个API的调用，我会尝试只调用一个能获取所有数据的网络访问请求 - 正如我另一篇文章《[关于减少移动数据使用率](https://www.youtube.com/watch?v=tnj2uoTwevg)》里面说的那样）。

	`UserRepository`
        
        package za.co.riggaroo.gus.data;
        
        import java.util.List;
        
        import rx.Observable;
        import za.co.riggaroo.gus.data.remote.model.User;
        
        public interface UserRepository {
        
            Observable<List<User>> searchUsers(final String searchTerm);
        }

	`UserRepositoryImpl`

		package za.co.riggaroo.gus.data;
        
        import java.io.IOException;
        import java.util.List;
        
        import rx.Observable;
        import za.co.riggaroo.gus.data.remote.GithubUserRestService;
        import za.co.riggaroo.gus.data.remote.model.User;
        
        public class UserRepositoryImpl implements UserRepository {
        
            private GithubUserRestService githubUserRestService;
        
            public UserRepositoryImpl(GithubUserRestService githubUserRestService) {
                this.githubUserRestService = githubUserRestService;
            }
        
            @Override
            public Observable<List<User>> searchUsers(final String searchTerm) {
                return Observable.defer(() -> githubUserRestService.searchGithubUsers(searchTerm).concatMap(
                        usersList -> Observable.from(usersList.getItems()).concatMap(
                                user -> githubUserRestService.getUser(user.getLogin())).toList()))
                        .retryWhen(observable -> observable.flatMap(o -> {
                            if (o instanceof IOException) {
                                return Observable.just(null);
                            }
                            return Observable.error(o);
                        }));
            }
        }
>译者注：如果项目没有添加Java8支持，会提示不支持Lambda表达式。请在build.gradle里面添加jack支持以及java8支持。在`android{ defaultConfig{}}`里面添加`jackOptions {enabled true}`。在`android`节点里面添加
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
   如果对上面的代码阅读困难，请看下面的解释，或者大致理解为网络接口的调用即可，不必深究细节，因为本系列的重点是测试。

	在上面的代码，我用`Observable.defer()`创建了一个Observable，这意味着这些observables代码只在当它有了subscriber才会运行（不像`Observable.create()`，这个方法会在创建之后立即运行）。正如下面的已修正的评论所说的，`Observable.create()`是不安全的RxJava API，不应该使用它。（译者注：[原文](https://riggaroo.co.za/introduction-android-testing-part3/)的评论讨论了这个问题）
	
	当Subscriber存在时，`githubUserRestService`会以`searchTerm`为参数被调用。在那里，我用`concatMap`获取用户列表，并给每一个User新建一个Observable，每个Observable调用`githubUserRestService.getUser()`。最后的observable会合并所有返回，并转换成一张用户列表。
	
	在这些网络访问中，我添加了重试的机制。当`IOException`抛出时，`retryWhen()`将会重新执行observable。当用户**没有可用网络**时，Retrofit会抛出`IOException`（也许你想给重试机制加上终止条件，例如重试了一定的次数之后就停止。）。
	
	你可能注意到我使用了lambda表达式，你可以在项目添加Jack编译链的支持后使用它。阅读[这里](https://developer.android.com/preview/j8-jack.html)了解在Android中激活Java 8的支持。
	现在我们有了一个repository和获取用户列表的两个网络请求。我们应该为刚刚写好的代码编写单元测试了！

单元测试 - Mockito是什么？
------------------

为了给repository对象写单元测试，我们准备使用Mockito。Mockito是什么？Mockito是MIT许可下的面向Java的一个开源测试框架。这个框架允许我们在自动化单元测试中创建很多的测试对象（mock 对象）。（译者注：就是一些桩对象，可以给这些对象设置它们的行为，例如传入某些参数到某个方法，就可以得到对应的返回值。详细请看后面的代码。）

Mockito可以让你控制方法调用并检查对象之间的交互行为。

当我们编写单元测试时，我们需要思考如何在一个隔离的环境下去测试一个确定的组件。我们不应该去测试超出被测试类职责的功能。Mockito帮助我们做到这种隔离。

Okay，让我们开始写测试吧！

给UserRepositoryImple写单元测试
-------------------------

1. 选中`UserRepositoryImpl`类，按下“ALT+ENTER”。在弹出菜单中选择“Create Test”。这时会弹出一个对话框：![Create-Test-Dialog](http://img.blog.csdn.net/20161005143503653)

2. 你可以选择生成方法，不过我一般会让选项留空。接下来会问你要将测试代码放到哪个目录下面。选择“app/src/test”目录，因为我们在编写的是JUnit测试，这种测试不需要Android [Context](https://developer.android.com/reference/android/content/Context.html)。![Select-Test-Directory-Automated-Testing-Android](http://img.blog.csdn.net/20161005144352594)

3. 现在我们完成编写单元测试的准备工作。新建一个`UserRepository`成员字段。我们还需要创建一个mock的`GithubUserRestService`实例，这是因为在这个测试中我们不准备直接调用API。这个测试只是为了确保UserRepository能正确地完成数据转换工作。以下代码是单元测试的配置：

        @Mock
        GithubUserRestService githubUserRestService;
    
        private UserRepository userRepository;
    
        @Before
        public void setUp() throws Exception {
            MockitoAnnotations.initMocks(this);
            userRepository = new UserRepositoryImpl(githubUserRestService);
        }

	导包

        import org.junit.Before;
        import org.mockito.Mock;
        import org.mockito.MockitoAnnotations;
        
        import za.co.riggaroo.gus.data.remote.GithubUserRestService;
        
        import static org.junit.Assert.*;
	这个被`@Before`注解的方法会在任何单元测试之前运行，它确保Mock对象在被使用之前就配置好。我们在`setUp()`中调用`MockitoAnnotations.initMocks()`方法，这样就使用mock出来的github服务创建了一个`UserRepository`实例。
	
4. 我们将要写的第一个测试是用来测试`GithubUserRestService`被使用正确的参数调用。同时可以测试它是否返回预期的结果。下面是我编写的例子：

        @Test
        public void searchUsers_200OkResponse_InvokesCorrectApiCalls() {
            //Given
            when(githubUserRestService.searchGithubUsers(anyString())).thenReturn(Observable.just(githubUserList()));
            when(githubUserRestService.getUser(anyString()))
                    .thenReturn(Observable.just(user1FullDetails()), Observable.just(user2FullDetails()));
    
            //When
            TestSubscriber<List<User>> subscriber = new TestSubscriber<>();
            userRepository.searchUsers(USER_LOGIN_RIGGAROO).subscribe(subscriber);
    
            //Then
            subscriber.awaitTerminalEvent();
            subscriber.assertNoErrors();
    
            List<List<User>> onNextEvents = subscriber.getOnNextEvents();
            List<User> users = onNextEvents.get(0);
            Assert.assertEquals(USER_LOGIN_RIGGAROO, users.get(0).getLogin());
            Assert.assertEquals(USER_LOGIN_2_REBECCA, users.get(1).getLogin());
            verify(githubUserRestService).searchGithubUsers(USER_LOGIN_RIGGAROO);
            verify(githubUserRestService).getUser(USER_LOGIN_RIGGAROO);
            verify(githubUserRestService).getUser(USER_LOGIN_2_REBECCA);
        }
    
        private UsersList githubUserList() {
            User user = new User();
            user.setLogin(USER_LOGIN_RIGGAROO);
    
            User user2 = new User();
            user2.setLogin(USER_LOGIN_2_REBECCA);
    
            List<User> githubUsers = new ArrayList<>();
            githubUsers.add(user);
            githubUsers.add(user2);
            UsersList usersList = new UsersList();
            usersList.setItems(githubUsers);
            return usersList;
        }
    
        private User user1FullDetails() {
            User user = new User();
            user.setLogin(USER_LOGIN_RIGGAROO);
            user.setName("Rigs Franks");
            user.setAvatarUrl("avatar_url");
            user.setBio("Bio1");
            return user;
        }
    
        private User user2FullDetails() {
            User user = new User();
            user.setLogin(USER_LOGIN_2_REBECCA);
            user.setName("Rebecca Franks");
            user.setAvatarUrl("avatar_url2");
            user.setBio("Bio2");
            return user;
        }
	
	这个测试被分割成三个部分：given，when，then。我把我的测试切分成这样是因为这样可以确保写出来的测试是结构化的，并且可以促使你去思考你正在测试的确切功能。在这个测试，我在进行这些功能的测试：给予Github Service返回的确切用户列表，当我搜索用户时，应该返回之前给予的用户列表，并且格式应该正确。
	
	我发现测试的命名一样非常重要。我喜欢的命名规则如下：
	
	>[Name of method under test]\_[Conditions of test case]_[Expected Result]
	
	所以在这个测试，这个方法的名字是`searchUsers_200OkResponse_InvokesCorrectApiCalls()`.在这个测试里面，一个`TestSubscriber`被添加为搜索查询的observable的订阅者。Assertions（断言）是作用在`TestSubscriber`上面，来确定它拥有期待的结果。

	译者补充：
	
        private static final String USER_LOGIN_RIGGAROO = "riggaroo";
        private static final String USER_LOGIN_2_REBECCA = "rebecca";

	导包：
	
        import org.junit.Assert;
        import org.junit.Before;
        import org.junit.Test;
        import org.mockito.Mock;
        import org.mockito.MockitoAnnotations;
        
        import java.util.ArrayList;
        import java.util.List;
        
        import rx.Observable;
        import rx.observers.TestSubscriber;
        import za.co.riggaroo.gus.data.remote.GithubUserRestService;
        import za.co.riggaroo.gus.data.remote.model.User;
        import za.co.riggaroo.gus.data.remote.model.UsersList;
        
        import static org.mockito.Matchers.anyString;
        import static org.mockito.Mockito.verify;
        import static org.mockito.Mockito.when;
>译者注：如果还有别的编译错误，请自行补充构造函数与set方法。

5. 下一个单元测试将会测试如果有IOException被搜索service抛出，那么网络请求将重试。

        @Test
        public void searchUsers_IOExceptionThenSuccess_SearchUsersRetried() {
    
            // Given
            when(githubUserRestService.searchGithubUsers(anyString())).thenReturn(getIOExceptionError(), Observable.just(githubUserList()));
            when(githubUserRestService.getUser(anyString())).thenReturn(Observable.just(user1FullDetails()), Observable.just(user2FullDetails()));
    
            // When
            TestSubscriber<List<User>> subscriber = new TestSubscriber<>();
            userRepository.searchUsers(USER_LOGIN_RIGGAROO).subscribe(subscriber);
    
            // Then
            subscriber.awaitTerminalEvent();
            subscriber.assertNoErrors();
    
            verify(githubUserRestService, times(2)).searchGithubUsers(USER_LOGIN_RIGGAROO);
    
            verify(githubUserRestService).getUser(USER_LOGIN_RIGGAROO);
            verify(githubUserRestService).getUser(USER_LOGIN_2_REBECCA);
    
        }
        
        private Observable getIOExceptionError() {
            return Observable.error(new IOException());
        }
在这个测试中，我们断言`githubUserRestService.searchGithubUsers()`被调用两次，而其它的网络请求只调用一次。我们还断言subscriber没有遇到终止性的错误。

UserRepositoryImpl的最终单元测试代码
---------------------------

我还添加了一些上面没有提到的测试。它们用来测试不同的情况，但是它们也遵循上面提到的相同的理念。下面是完整的UserRepositoryImpl的测试代码：

    package za.co.riggaroo.gus.data;
    
    import org.junit.Assert;
    import org.junit.Before;
    import org.junit.Test;
    import org.mockito.Mock;
    import org.mockito.MockitoAnnotations;
    
    import java.io.IOException;
    import java.util.ArrayList;
    import java.util.List;
    
    import okhttp3.MediaType;
    import okhttp3.ResponseBody;
    import retrofit2.Response;
    import retrofit2.adapter.rxjava.HttpException;
    import rx.Observable;
    import rx.observers.TestSubscriber;
    import za.co.riggaroo.gus.data.remote.GithubUserRestService;
    import za.co.riggaroo.gus.data.remote.model.User;
    import za.co.riggaroo.gus.data.remote.model.UsersList;
    
    import static org.mockito.Matchers.anyString;
    import static org.mockito.Mockito.never;
    import static org.mockito.Mockito.times;
    import static org.mockito.Mockito.verify;
    import static org.mockito.Mockito.when;
    
    public class UserRepositoryImplTest {
    
        private static final String USER_LOGIN_RIGGAROO = "riggaroo";
        private static final String USER_LOGIN_2_REBECCA = "rebecca";
        @Mock
        GithubUserRestService githubUserRestService;
    
        private UserRepository userRepository;
    
        @Before
        public void setUp() throws Exception {
            MockitoAnnotations.initMocks(this);
            userRepository = new UserRepositoryImpl(githubUserRestService);
        }
    
        @Test
        public void searchUsers_2000kResponse_InvokesCorrectApiCalls() {
    
            // Given
            when(githubUserRestService.searchGithubUsers(anyString())).thenReturn(Observable.just(githubUserList()));
            when(githubUserRestService.getUser(anyString())).thenReturn(Observable.just(user1FullDetails()), Observable.just(user2FullDetails()));
    
            // When
            TestSubscriber<List<User>> subscriber = new TestSubscriber<>();
            userRepository.searchUsers(USER_LOGIN_RIGGAROO).subscribe(subscriber);
    
            // Then
            subscriber.awaitTerminalEvent();
            subscriber.assertNoErrors();
    
            List<List<User>> onNextEvents = subscriber.getOnNextEvents();
            List<User> users = onNextEvents.get(0);
            Assert.assertEquals(USER_LOGIN_RIGGAROO, users.get(0).getLogin());
            Assert.assertEquals(USER_LOGIN_2_REBECCA, users.get(1).getLogin());
            verify(githubUserRestService).searchGithubUsers(USER_LOGIN_RIGGAROO);
            verify(githubUserRestService).getUser(USER_LOGIN_RIGGAROO);
            verify(githubUserRestService).getUser(USER_LOGIN_2_REBECCA);
        }
    
        @Test
        public void searchUsers_IOExceptionThenSuccess_SearchUsersRetried() {
    
            // Given
            when(githubUserRestService.searchGithubUsers(anyString())).thenReturn(getIOExceptionError(), Observable.just(githubUserList()));
            when(githubUserRestService.getUser(anyString())).thenReturn(Observable.just(user1FullDetails()), Observable.just(user2FullDetails()));
    
            // When
            TestSubscriber<List<User>> subscriber = new TestSubscriber<>();
            userRepository.searchUsers(USER_LOGIN_RIGGAROO).subscribe(subscriber);
    
            // Then
            subscriber.awaitTerminalEvent();
            subscriber.assertNoErrors();
    
            verify(githubUserRestService, times(2)).searchGithubUsers(USER_LOGIN_RIGGAROO);
    
            verify(githubUserRestService).getUser(USER_LOGIN_RIGGAROO);
            verify(githubUserRestService).getUser(USER_LOGIN_2_REBECCA);
    
        }
    
        @Test
        public void searchUsers_GetUserIOExceptionThenSuccess_SearchUsersRetried() {
            // Given
            when(githubUserRestService.searchGithubUsers(anyString())).thenReturn(Observable.just(githubUserList()));
            when(githubUserRestService.getUser(anyString())).thenReturn(getIOExceptionError(), Observable.just(user1FullDetails()), Observable.just(user2FullDetails()));
    
            // When
            TestSubscriber<List<User>> subscriber = new TestSubscriber<>();
            userRepository.searchUsers(USER_LOGIN_RIGGAROO).subscribe(subscriber);
    
            // Then
            subscriber.awaitTerminalEvent();
            subscriber.assertNoErrors();
    
            verify(githubUserRestService, times(2)).searchGithubUsers(USER_LOGIN_RIGGAROO);
    
            verify(githubUserRestService, times(2)).getUser(USER_LOGIN_RIGGAROO);
            verify(githubUserRestService).getUser(USER_LOGIN_2_REBECCA);
        }
    
        @Test
        public void searchUsers_OtherHttpError_SearchTerminatedWithError() {
            // Given
            when(githubUserRestService.searchGithubUsers(anyString())).thenReturn(get403ForbiddenError());
    
            // When
            TestSubscriber<List<User>> subscriber = new TestSubscriber<>();
            userRepository.searchUsers(USER_LOGIN_RIGGAROO).subscribe(subscriber);
    
            // Then
            subscriber.awaitTerminalEvent();
            subscriber.assertError(HttpException.class);
    
            verify(githubUserRestService).searchGithubUsers(USER_LOGIN_RIGGAROO);
    
            verify(githubUserRestService, never()).getUser(USER_LOGIN_RIGGAROO);
            verify(githubUserRestService, never()).getUser(USER_LOGIN_2_REBECCA);
    
        }
    
        private Observable get403ForbiddenError() {
            return Observable.error(new HttpException(
                    Response.error(403, ResponseBody.create(MediaType.parse("application/json"), "Forbidden"))));
        }
    
        private Observable getIOExceptionError() {
            return Observable.error(new IOException());
        }
    
        private UsersList githubUserList() {
            User user = new User();
            user.setLogin(USER_LOGIN_RIGGAROO);
    
            User user2 = new User();
            user2.setLogin(USER_LOGIN_2_REBECCA);
    
            List<User> githubUsers = new ArrayList<>();
            githubUsers.add(user);
            githubUsers.add(user2);
            UsersList usersList = new UsersList();
            usersList.setItems(githubUsers);
            return usersList;
        }
    
    
        private User user1FullDetails() {
            User user = new User();
            user.setLogin(USER_LOGIN_RIGGAROO);
            user.setName("Rigs Franks");
            user.setAvatarUrl("avatar_url");
            user.setBio("Bio1");
            return user;
        }
    
        private User user2FullDetails() {
            User user = new User();
            user.setLogin(USER_LOGIN_2_REBECCA);
            user.setName("Rebecca Franks");
            user.setAvatarUrl("avatar_url2");
            user.setBio("Bio2");
            return user;
        }
    }


运行单元测试
------

在编写完测试代码之后，我们需要来运行它们，看看它们是否通过并有多少的代码覆盖率。

1. 想运行测试，你可以右键点击测试类的名字并在弹出菜单选择“Run `UserRepositoryImplTest` with Coverage”。![Run-unit-tests-with-coverage](http://img.blog.csdn.net/20161005170528985)

2. 你将会看到测试结果出现在Android Studio的右手边。![Code-Coverage-Report-Unit-test-Android-Studio](http://img.blog.csdn.net/20161005170730721)

我们在UserRepositoryImpl这个类上面的代码覆盖率达到了100%~！

在下一篇博客，我们对UI进行实现，将搜索结果展现出来并为它编写更多的测试。
