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
1 . 首先，在`za.co.riggaroo.gus.presentation.base`包中创建基本接口`MvpVIew`和`MvpPresenter`。所有的MVP功能类都将继承这两个接口。
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

