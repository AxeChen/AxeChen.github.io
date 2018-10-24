很多人在面试的时候回答MVP的优点会提出：“有利于单元测试”。但是很多程序员没有写单元测试的习惯，特别是小型的创业公司，由于大量的编码工作使程序员将测试的任务全部交给了测试部门。实际上单元测试能够减少逻辑上的错误和bug量。

#### 1、Presenter中的逻辑测试
这里只对Presenter的测试进行说明，Presenter的测试相对于Mode和View的测试更加重要。因为主要的逻辑代码写在这里。

##### 1.1、Presenter测试的逻辑原理

在测试代码中编写一些测试案例，断言Model层可能出现的结果来测试功能代码。在MVP架构中，将View和Model分开了，用Presenter进行View和Model通信。用断言的方式来列举出Model可能出现的数据，验证Presenter中的逻辑判断，然后检查View层执行的方法。
其中：测试的过程控制、测试逻辑、测试案例等用到Junit框架；用于断言和检查方法执行用到Mockito框架。

##### 1.2、单元测试依赖的测试框架
Presenter为逻辑测试，主要依赖Junit和mokito框架。
```
    // 测试相关
    testImplementation "junit:junit:4.12"
    testImplementation "org.mockito:mockito-core:1.10.19"
```
在进行单元测试前，必须了解Junit和Mockito提供的注解和方法，否则就无法进行测试。
比如：
![Junit和Mockito](https://upload-images.jianshu.io/upload_images/1930161-a4116c53e060f6b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
以上图片的内容来自文章：[https://www.jianshu.com/p/5c8cde7ab54e](https://www.jianshu.com/p/5c8cde7ab54e)
如果不知道Junit和mokito，建议先点过去看下。

#### 2、在MVP框架中的单元测试实战

假如现在在做一个登录的功能，登录的需求如下。
* 1、登录成功。（当服务返回的code为0。）
* 2、登录失败。（当服务其返回的code不为0。）
* 3、登录失败，因为用户名为空。
* 4、登录失败，因为密码为空。

##### 2.1、非TTD测试驱动开发模式的单元测试
（什么是TTD测试驱动开发？后面会提到）
按照需求，会去创建MVP结构来做这个模块。（**注意：**1、这里展示的是最简单的MVP结构。2、网络请求框架我用的是已经封装好的Rxjava+Retorfit。3、api用的是WanAndroid的api）

![简单的mvp结构](https://upload-images.jianshu.io/upload_images/1930161-a61fc3e561b1b499.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在LoginContract中根据需求定义接口：
```
public interface LoginContract {
    public interface View {
        // 登录成功。（当服务返回的code为0。）
        void loginSuccess();

        // 登录失败。（当服务其返回的code不为0。）
        void loginFail();

        // 登录失败，因为用户名错误
        void loginFailCauseByErrorUserName();

        // 登录失败，因为用户名错误
        void loginFailCauseByErrorPassword();
    }

    public interface Model {
        Observable<Response<JSONObject>> login(String userName, String userPwd);
    }

    public interface Presenter {
        void login(String userName, String userPwd);
    }
}
```
Presenter中的代码：
```
/**
 * 登录Presenter层
 */
public class LoginPresenter implements LoginContract.Presenter {

    private LoginContract.Model model;
    private LoginContract.View view;
    private BaseSchedulerProvider schedulerProvider;
    private CompositeDisposable mDisposable;

    public LoginPresenter(LoginContract.Model model,LoginContract.View view){
        this.view = view;
        this.model = model;
        mDisposable = new CompositeDisposable();
        schedulerProvider = SchedulerProvider.getInstance();
    }

    /**
     * 这个是为单元测试建的构造函数，原因是因为Rxjava线程切换，必须设置为立即执行才能测试通过
     * @param model
     * @param view
     * @param schedulerProvider
     */
    public LoginPresenter(LoginContract.Model model,LoginContract.View view,SchedulerTestProvider schedulerProvider){
        this.view = view;
        this.model = model;
        mDisposable = new CompositeDisposable();
        this.schedulerProvider = schedulerProvider;
    }

    @Override
    public void login(String userName, String userPwd) {
        if (TextUtils.isEmpty(userName)) {
            view.loginFailCauseByErrorUserName();
            return;
        }
        if (TextUtils.isEmpty(userPwd)) {
            view.loginFailCauseByErrorPassword();
            return;
        }
        Disposable disposable = model.login(userName, userPwd).
                compose(ResponseTransformer.handleResult()).
                compose(schedulerProvider.applySchedulers())
                .subscribe(jsonObject -> {
                            view.loginSuccess();
                        },
                        throwable -> {
                            view.loginFail();
                        });
        mDisposable.add(disposable);
    }
}
```
这里要进行测试的就是login(String userName, String userPwd)方法中的逻辑。测试的时候会断言服务器返回的数据，会列举一些登录成功和登录失败的测试案例，如果这些测试案例能够正常运行，那么就代表这些代码的逻辑运行正常。
* 新建一个测试类去测试Presenter的逻辑。
  ![新建测试类](https://upload-images.jianshu.io/upload_images/1930161-d9ee0b08f386a391.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 根据需求写出测试案例初始化Presenter使用的必要类
```
/**
 * 登录Presenter的测试类
 */
public class LoginPresenterTest {

    @Mock
    private LoginContract.Model model;

    @Mock
    private LoginContract.View view;

    private LoginPresenter presenter;

    @Mock
    private SchedulerTestProvider schedulerProvider;

    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
        schedulerProvider = new SchedulerTestProvider();
        presenter = new LoginPresenter(model, view,schedulerProvider);
    }

    /**
     * 登陆成功
     * @throws Exception
     */
    @Test
    public void loginSuccess() throws Exception {
    }

    /**
     * 登录失败，服务器返回错误的code
     * @throws Exception
     */
    @Test
    public void loginFailByServer() throws Exception{
    }

    /**
     * 登录失败，错误的用户名
     * @throws Exception
     */
    @Test
    public void loginFailByErrorUserName() throws Exception{
    }

    /**
     * 登录失败，错误的密码
     * @throws Exception
     */
    @Test
    public void loginFailByErrorPwd() throws Exception{
    }

}
```
这里写出登录成功loginsuccess()的测试代码：
```
    @Test
    public void loginSuccess() throws Exception {
        // 1、断言model.login的方法返回正确的Response数据
        when(model.login("123321","123321")).thenReturn(Observable.just(new Response<>(0,new JSONObject(),"")));
        // 2、Presenter执行登录逻辑
        presenter.login("123321","123321");
        // 3、预测回调给view层的方法是否被调用
        verify(view).loginSuccess();
    }
```
这里的测试逻辑为：    
1、断言model返回正确的数据。       
2、presenter去调用登录的方法。     
3、检测view最后会调用的方法。    

然后单独测试这个方法。
![点这里测试](https://upload-images.jianshu.io/upload_images/1930161-a9860c4576283d0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果绿了就成功了。（什么玩意？ 绿..绿了？）
![测试成功](https://upload-images.jianshu.io/upload_images/1930161-6d497763fc0bf349.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

那这里对返回的参数进行修改，将code改为非0，这个逻辑在网络请求框架中的ResponseTransformer类中定义的，非0则请求失败。
```
    @Test
    public void loginSuccess() throws Exception {
        // 1、断言model.login的方法返回正确的Response数据
        when(model.login("123321","123321")).thenReturn(Observable.just(new Response<>(1,new JSONObject(),"")));
        // 2、Presenter执行登录逻辑
        presenter.login("123321","123321");
        // 3、预测回调给view层的方法是否被调用
        verify(view).loginSuccess();
    }
```
按照实际逻辑，如果为非0就是登录失败，view层的loginSuccess()方法一定不会被调用，那么测试肯定无法通过。那么运行一下。
![测试失败](https://upload-images.jianshu.io/upload_images/1930161-2b9b6b5d45348cf9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
好的红了，看下这个提示信息**Wanted but not invoked**。这个意思为：**想要使用但是不被调用。**


以上的例子就是单元测试的一个简单的例子了。

##### 2.1、TTD测试驱动开发模式的单元测试
这里提到了TTD测试驱动开发，TTD测试驱动开发到底是个什么东东呢？

> 在开发之前，先编写测试代码，通过测试案例去编写功能代码。一开始测试案例由于没有功能代码都会无法运行通过，然后通过修改功能代码使测案例一个一个通过测试。

如果这个功能用TTD测试驱动开发去做如何做呢？     
* 首先Presenter中的login()方法不要写功能代码，让他是一个空的方法。
* 然后新建Presenter测试类将测试案例都列举出来。
* 编写测试案例的测试代码。
* 根据测试代码去编写功能代码，使测试案例一个一个通过测试。

#### 3、可能会遇到的问题

##### 3.1、Android的api无法直接使用需要特殊处理
在Presenter中一般只做逻辑操作，不做界面处理，所以很少会用到Android的API。但是这个不是绝对的。比如TextUtils做非空判断。如果不对这个做特殊处理，那么会报无法调用的错误。    
**解决方案：**在TextUtils中新建一个包名一样的TextUtils类即可。

![新建TextUtils](https://upload-images.jianshu.io/upload_images/1930161-945268ac8b351044.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 3.2、Rxjava的线程测试处理，这里用到Rxjava可能会遇到
这个需要将线程的改为```Schedulers.trampoline() ```强制立即进行当前的任务。
具体看下这篇文章吧：
https://www.jianshu.com/p/22384556bd22

##### 3.3、静态类方法无法被Mock
具体可以看看这个。https://blog.csdn.net/hongchangfirst/article/details/46453677


#### 4、其他的测试
单元测试不仅仅只Presenter测试，还有view层的测试和model层的测试。这里就不做说明啦，主要是没去做过view层和model层的测试，知道Presenter层的测试，其他的测试应该大同小异。如果以后做到了这块，再更新一些播客吧。
![手动滑稽](https://upload-images.jianshu.io/upload_images/1930161-aee57be922bc3c13.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



代码地址：[https://github.com/AxeChen/MvpUnitTest](https://github.com/AxeChen/MvpUnitTest)

**如果你对这篇文章感兴趣，还请点个赞。**


参考的博客：     
[Junit和Mokito](https://www.jianshu.com/p/5c8cde7ab54e)    
[静态方法无法被Mock](https://blog.csdn.net/hongchangfirst/article/details/46453677)
[强制线程立即执行](https://www.jianshu.com/p/22384556bd22)