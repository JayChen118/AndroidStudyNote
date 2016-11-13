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

创建Presenter
-----------
1 . 首先，在`za.co.riggaroo.gus.presentation.base`包中创建基本接口`MvpView`和`MvpPresenter`。所有的MVP功能类都将继承这两个接口。
```
public interface MvpView {
}

public interface MvpPresenter<V extends MvpView> {
 
    void attachView(V mvpView);
 
    void detachView();
}
```

2 . 创建一个`BasePresenter`。在这个类中，我们检查当前的Presenter是否已经依附了一个View，并提供管理RxJava订阅者的方法。
```
public class BasePresenter<T extends MvpView> implements MvpPresenter<T> {
 
    private T view;
 
    private CompositeSubscription compositeSubscription = new CompositeSubscription();
 
    @Override
    public void attachView(T mvpView) {
        view = mvpView;
    }
 
    @Override
    public void detachView() {
        compositeSubscription.clear();
        view = null;
    }
 
    public T getView() {
        return view;
    }
 
    public void checkViewAttached() {
        if (!isViewAttached()) {
            throw new MvpViewNotAttachedException();
        }
    }
 
    private boolean isViewAttached() {
        return view != null;
    }
 
    protected void addSubscription(Subscription subscription) {
        this.compositeSubscription.add(subscription);
    }
 
    protected static class MvpViewNotAttachedException extends RuntimeException {
        public MvpViewNotAttachedException() {
            super("Please call Presenter.attachView(MvpView) before" + " requesting data to the Presenter");
        }
    }
}
```
正如你在上面看到的，这个presenter定义了一个`CompositeSubscription`。这个对象将会保存一组RxJava的Subscription（订阅）。在`detachView()`方法中调用了`compositionSubscription.clear()`方法，这个方法将会取消所有的订阅，从而防止内存泄露和View造成的崩溃（当View被销毁，它就不会被订阅，相关的代码也不会运行）。当继承于这个类的presenter中有subscription被创建时，我们调用`addSubscription()`。

3 . 创建一个`UserSearchContract`接口来表示View和Presenter之间的Contract（约定？交互关系？自己理解就好，翻译不出来了）。在这个接口中，分别为View和Presenter创建一个接口。
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
在View接口中，有个四个方法：`showSearchResults()`,`showError()`,`showLoading()`,`hideLoading()`。在Presenter中，只有一个`search()`方法。

一个**Presenter**既不在意一个**View**如何去展示获得的数据，也不在意如何展示错误信息。相似的，一个**View**也不关心一个**Presenter**如何去搜索，只需要**Presenter**会调用回调方法，具体的实现无关紧要。

分离View和Presenter之间的逻辑是件简单的事。从如何将Presenter重用到另一种类型的UI的角度考虑，你就会明白代码应该放到哪里。例如，当你必须使用Java Swing作为UI实现工具，你的Presenter可以保持不变的话，就仅仅需要改变你的View实现了。这意味着当你考虑逻辑代码应该放在哪里时，仅仅需要问自己：当我有了另一套不同的UI时，Presenter里面的逻辑还有意义吗？

4 . 现在我们已经定义好View跟Presenter之间的约定。创建或导航到`UserSearchPresenter`。在这里，我们添加对`UserRepository`的订阅，这就是我们调用Github API的地方。
```
class UserSearchPresenter extends BasePresenter<UserSearchContract.View> implements UserSearchContract.Presenter {
    private final Scheduler mainScheduler, ioScheduler;
    private UserRepository userRepository;

    UserSearchPresenter(UserRepository userRepository, Scheduler ioScheduler, Scheduler mainScheduler) {
        this.userRepository = userRepository;
        this.ioScheduler = ioScheduler;
        this.mainScheduler = mainScheduler;
    }

    @Override
    public void search(String term) {

    }
}
```
这个Presenter继承了`BasePresenter`并且实现了第3步定义的`UserSearchContract.Presenter`接口。我们将在这个类里面实现`Search()`方法的具体逻辑（先放一个空方法）。

使用[Constructor injection](https://en.wikipedia.org/wiki/Dependency_injection#Constructor_injection)（构造注入？）可以在需要做单元测试时轻松地仿造(mock) UserRepository。两个Scheduler也是通过构造器注入，在单元测试时，我们会一直用`Schedulers.immediate()`（即立即执行的策略），而在View层调用时，我们会使用不同的线程（即一个主线程，一个IO线程）。

5 . 以下是`search()`的实现：
```
    @Override
    public void search(String term) {
        checkViewAttached();
        getView().showLoading();
        addSubscription(userRepository.searchUsers(term).subscribeOn(ioScheduler).observeOn(mainScheduler).subscribe(new Subscriber<List<User>>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {
                getView().hideLoading();
                getView().showError(e.getMessage()); //TODO You probably don't want this error to show to users - Might want to show a friendlier message :)
            }

            @Override
            public void onNext(List<User> users) {
                getView().hideLoading();
                getView().showSearchResults(users);
            }
        }));
    }
```

首先，调用`checkViewAttached()`，如果当前没有View依附在Presenter上的话，会抛出异常。接着通过调用`showLoading()`告诉View，它应该开始加载了。给`userRepository.searchUsers()`创建一个Subscription（订阅）。设置`subscribeOn()`的参数为`ioScheduler`，因为我们希望网络调用发生在IO线程上。设置`observeOn()`的参数为`mainScheduler`，因为我们希望这个Subscription的结果可以在主线程观察到（应该是在主线程运行的意思）。最后通过调用`addSubscription()`，将Subscription添加到我们的Subscription组里面。

在`onNext()`里面，通过调用`hideLoading()`和`showSearchResults()`方法处理API返回的用户列表。在`onError()`里面，停止加载并调用`showError()`显示错误信息。

以下是`UserSearchPresenter`的全部代码：
```
package za.co.riggaroo.gus.presentation.search;


import java.util.List;

import rx.Scheduler;
import rx.Subscriber;
import za.co.riggaroo.gus.data.UserRepository;
import za.co.riggaroo.gus.data.remote.model.User;
import za.co.riggaroo.gus.presentation.base.BasePresenter;

class UserSearchPresenter extends BasePresenter<UserSearchContract.View> implements UserSearchContract.Presenter {
    private final Scheduler mainScheduler, ioScheduler;
    private UserRepository userRepository;

    UserSearchPresenter(UserRepository userRepository, Scheduler ioScheduler, Scheduler mainScheduler) {
        this.userRepository = userRepository;
        this.ioScheduler = ioScheduler;
        this.mainScheduler = mainScheduler;
    }

    @Override
    public void search(String term) {
        checkViewAttached();
        getView().showLoading();
        addSubscription(userRepository.searchUsers(term).subscribeOn(ioScheduler).observeOn(mainScheduler).subscribe(new Subscriber<List<User>>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {
                getView().hideLoading();
                getView().showError(e.getMessage()); //TODO You probably don't want this error to show to users - Might want to show a friendlier message :)
            }

            @Override
            public void onNext(List<User> users) {
                getView().hideLoading();
                getView().showSearchResults(users);
            }
        }));
    }
}
```


为 UserSearchPresenter 编写单元测试
----------------------------

现在我们已经定义好presenter了，开始为它写一些单元测试吧！

1 . 选中`UserSearchPresenter`的类名，按下“ALT + Enter”键，选中“Create Test”。选择“app/src/**test**/java”目录，因为这是不需要Android依赖的单元测试。测试代码的最终存放路径为:`app/src/test/java/za/co/riggaroo/gus/presentation/search`。

2 . 在`UserSearchPresenterTest`里面，创建setup方法以及定义我们在测试中需要用到的变量。
```
public class UserSearchPresenterTest {

    @Mock
    UserRepository userRepository;
    @Mock
    UserSearchContract.View view;

    UserSearchPresenter userSearchPresenter;

    @Before
    public void setUp() throws Exception {
        MockitoAnnotations.initMocks(this);
        userSearchPresenter = new UserSearchPresenter(userRepository, Schedulers.immediate(), Schedulers.immediate());
        userSearchPresenter.attachView(view);
    }
}
```

通过仿造 `UserRepository` 和 `UserSearchContract.View` ，我们可以确保只测试 `UserSearchPresenter` 。在`setup()`方法中，我们调用`MockitoAnnotations.initMocks()`来初始化仿造的变量。接着用仿造的对象和即时计划（immediate schedules）创建presenter。调用`attachView()`将仿造的View依附到Presenter上面。

3 . 第一个测试的目标是一个有效的查询条件会有正确的回调：
```
    private static final String USER_LOGIN_RIGGAROO = "riggaroo";
    private static final String USER_LOGIN_2_REBECCA = "rebecca";
   
    @Test
    public void search_ValidSearchTerm_ReturnsResults() {
        UsersList userList = getDummyUserList();
        when(userRepository.searchUsers(anyString())).thenReturn(Observable.<List<User>>just(userList.getItems()));

        userSearchPresenter.search("riggaroo");

        verify(view).showLoading();
        verify(view).hideLoading();
        verify(view).showSearchResults(userList.getItems());
        verify(view, never()).showError(anyString());
    }

    UsersList getDummyUserList() {
        List<User> githubUsers = new ArrayList<>();
        githubUsers.add(user1FullDetails());
        githubUsers.add(user2FullDetails());
        return new UsersList(githubUsers);
    }

    User user1FullDetails() {
        return new User(USER_LOGIN_RIGGAROO, "Rigs Franks", "avatar_url", "Bio1");
    }

    User user2FullDetails() {
        return new User(USER_LOGIN_2_REBECCA, "Rebecca Franks", "avatar_url2", "Bio2");
    }
```

这个测试断定：**设定** user repository 会返回一组用户，**当**在presenter上调用 `search()`，**最后**View的 `showLoading()`，`hideLoading()` 和`showSearchResult()`被调用。这个测试也断定`showError()`方法不会被调用。

4 . 第二个测试的目标是当UserRepository抛出异常后会出现错误页面：
```
    @Test
    public void search_UserRepositoryError_ErrorMsg() {
        String errorMsg = "No internet";
        when(userRepository.searchUsers(anyString())).thenReturn(Observable.error(new IOException(errorMsg)));

        userSearchPresenter.search("bookdash");

        verify(view).showLoading();
        verify(view).hideLoading();
        verify(view, never()).showSearchResults(anyList());
        verify(view).showError(errorMsg);
    }
```

这个测试是这样进行的：**设定** userRepository 会返回一个异常，**当**调用 `search()`时，**最后**会调用`showError()`。

5 . 最后的测试的目标是在没有View依附时，会抛出异常：
```
    @Test(expected = BasePresenter.MvpViewNotAttachedException.class)
    public void search_NotAttached_ThrowsMvpException() {
        userSearchPresenter.detachView();

        userSearchPresenter.search("test");

        verify(view, never()).showLoading();
        verify(view, never()).showSearchResults(anyList());
    }
```

译者注：如果MvpViewNotAttachedException报错，将访问限制改为public。

6 . 让我们运行这些测试吧！看看我们能有多少覆盖率。右键点击测试类名，选择“Run tests with coverage”。

![test_result](http://img.blog.csdn.net/20161113133303160)

Yay！我们获得了100%的覆盖率。

下一篇博客将会涉及创建UI并编写UI测试。