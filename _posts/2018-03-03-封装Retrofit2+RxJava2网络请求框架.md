---
tags:
    - Android
	- Retrofit
	- RxJava
---



因为Retrofit方便使用和支持RxJava，所以Retrofit已经成为了非常流行的网络框架。学会封装和使用Retrofit网络请求框架练练手是提升自己架构水平一个非常好的示例。而且当成功封装第一个组件时，再次遇到需要封装组件这样的任务也会变得得心应手。


#### 1、封装的主要逻辑
![主要的逻辑](http://upload-images.jianshu.io/upload_images/1930161-1cdaf0da3450005a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



根据这个逻辑图一步一步封装网络框架吧。
##### 1.1 导入依赖
```
    compile "io.reactivex.rxjava2:rxjava:2.1.0" // 必要rxjava2依赖
    compile "io.reactivex.rxjava2:rxandroid:2.0.1" // 必要rxandrroid依赖，切线程时需要用到
    compile 'com.squareup.retrofit2:retrofit:2.3.0' // 必要retrofit依赖
    compile 'com.squareup.retrofit2:adapter-rxjava2:2.3.0' // 必要依赖，和Rxjava结合必须用到，下面会提到
    compile 'com.squareup.retrofit2:converter-gson:2.3.0' // 必要依赖，解析json字符所用
    compile 'com.squareup.okhttp3:logging-interceptor:3.8.1' //非必要依赖， log依赖，如果需要打印OkHttpLog需要添加
```

##### 1.2 新建Manger类，管理所需要的API
这里是一个千篇一律的套路，如果封装全局使用的框架，并且需要和整个软件有一样长的生命周期。那就有两个特点：  
* 在Application里面初始化。  
* 使用单例模式。  
* 在一个"管理类"中初始化所需要的所有参数。

```
/**
 * Created by Zaifeng on 2018/2/28.
 * API初始化类
 */
public class NetWorkManager {

    private static NetWorkManager mInstance;
    private static Retrofit retrofit;
    private static volatile Request request = null;

    public static NetWorkManager getInstance() {
        if (mInstance == null) {
            synchronized (NetWorkManager.class) {
                if (mInstance == null) {
                    mInstance = new NetWorkManager();
                }
            }
        }
        return mInstance;
    }

    /**
     * 初始化必要对象和参数
     */
    public void init() {
        // 初始化okhttp
        OkHttpClient client = new OkHttpClient.Builder()
                .build();

        // 初始化Retrofit
        retrofit = new Retrofit.Builder()
                .client(client)
                .baseUrl(Request.HOST)
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .addConverterFactory(GsonConverterFactory.create())
                .build();
    }

    public static Request getRequest() {
        if (request == null) {
            synchronized (Request.class) {
                request = retrofit.create(Request.class);
            }
        }
        return request;
    }
}
```
这里新建了NetWorkManager 在init方法中初始化了OkHttp和Retrofit。两者的初始化都是构造者模式。

**初始化OKHttp的拓展**
上面的代码只是添加了必要的属性，还可以对OkHttp和Retrofit添加更多的拓展。比如：需要对OkHttp添加Log可以如下写。
```
HttpLoggingInterceptor logging = new HttpLoggingInterceptor();
logging.setLevel(HttpLoggingInterceptor.Level.BASIC);
OkHttpClient client = new OkHttpClient.Builder()
                    .addInterceptor(logging)
                    .build();
```
```addInterceptor()```还有很多的用法，比如：封装一些公共的参数等等。参考如下博客：[http://blog.csdn.net/jdsjlzx/article/details/52063950](http://blog.csdn.net/jdsjlzx/article/details/52063950)
这里就不多说明。

**初始化Retrofit的时候的两个必要配置：**
* 第一个配置：```.addCallAdapterFactory(RxJava2CallAdapterFactory.create())```
这个是用来决定你的返回值是Observable还是Call。
```
    // 使用call的情况
    Call<String> login();  

    // 使用Observable的情况
    Observable<String> login();  
```
如果返回为Call那么可以不添加这个配置。如果使用Observable那就必须添加这个配置。否则就会请求的时候就会报错！
对于这个配置，详细可以看下这篇博客。
http://blog.csdn.net/new_abc/article/details/53021387

* 第二个配置：```.addConverterFactory(GsonConverterFactory.create())```
这个配置是将服务器返回的json字符串转化为对象。这个是可以自定义Converter来应对服务器返回的不同的数据的。具体如何去自定义，这里也不做说明了（因为内容比较多），可以看下如下的博客。
[https://www.jianshu.com/p/5b8b1062866b](https://www.jianshu.com/p/5b8b1062866b)

**初始化Request**
这里也初始化了Request。如果有Retrofit基础看到```retrofit.create```这个方法，就应该知道Request是定义的请求的服务器的API封装类。里面通过注解的方式声明所需要请求的接口，这里下面会讲到。

##### 1.3 约定Response
在解析服务器返回的数据时，需要和服务器统一返回Json格式，这个在开发中经常遇到。一般服务器也不会随便返回Json格式给前端，否则前端解析时将会非常麻烦。(所以这里的数据结构需要前端和服务端约定一个固定的格式)
这里的固定json格式是：
```
{ret:0,data:"",msg:""}
```
所以这里定义的固定的Response为：
```
public class Response<T> {

    private int ret; // 返回的code
    private T data; // 具体的数据结果
    private String msg; // message 可用来返回接口的说明

    public int getCode() {
        return ret;
    }

    public void setCode(int code) {
        this.ret = code;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}
```
##### 1.4 定义Request
在前面NetWorkManager中已经提到了，这个是定义的请求服务器的API的接口。
```
public interface Request {

    // 填上需要访问的服务器地址
    public static String HOST = "https://www.xxx.com/app_v5/";
    
    @POST("?service=sser.getList")
    Observable<Response<List<javaBean>>> getList(@Query("id") String id);
}
```
这里用@Post注解了一个Post请求。

##### 1.5 定义ResponseTransformer处理数据和异常
这个也是自己定义的类，不使用这个自定义的类也是可以的。但是为了封装之后使用的便捷性，还是建议封装。ResponseTransformer实际上是Rxjava中的“变换”的封装。
没使用ResponseTransformer时的情况。
```
 model.getCarList("xxxxx")
                .compose(schedulerProvider.applySchedulers())
                .subscribe(new Consumer<Response<List<JavaBean>>>() {
                    @Override
                    public void accept(Response<List<JavaBean>> listResponse) throws Exception {
                        if(listResponse.getCode() == 200){
                            List<JavaBean> javaBeans = listResponse.getData();
                        }else{
                            // 异常处理
                        }
                    }
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(Throwable throwable) throws Exception {
                            // 异常处理
                    }
                });
```
使用了ResponseTransformer的情况：
```
 model.getCarList("xxxxx")
                .compose(ResponseTransformer.handleResult())
                .compose(schedulerProvider.applySchedulers())
                .subscribe(new Consumer<List<JavaBean>>() {
                               @Override
                               public void accept(List<JavaBean> carBeans) throws Exception {
                                   // 处理数据 直接获取到List<JavaBean> carBeans
                               }
                           }, new Consumer<Throwable>() {
                               @Override
                               public void accept(Throwable throwable) throws Exception {
                                   // 处理异常
                               }
                           }

                );
```
没有使用ResponseTransformer，需要先判断Response的code，然后将数据从Response中取出来。比较繁琐的就是判断code==200的代码，每次请求接口都需要去写一遍。
使用了ResponseTransformer将解析的Response数据进一步处理（将判断code==200的代码进行封装），直接将需要使用的数据处理提取出来，并且将Exception统一处理！这个就是RxJava中“变换”的妙用，同时在使用时会和“compose”联用，减少重复代码。
具体的ResponseTransformer代码。
```
public class ResponseTransformer {

    public static <T> ObservableTransformer<Response<T>, T> handleResult() {
        return upstream -> upstream
                .onErrorResumeNext(new ErrorResumeFunction<>())
                .flatMap(new ResponseFunction<>());
    }


    /**
     * 非服务器产生的异常，比如本地无无网络请求，Json数据解析错误等等。
     *
     * @param <T>
     */
    private static class ErrorResumeFunction<T> implements Function<Throwable, ObservableSource<? extends Response<T>>> {

        @Override
        public ObservableSource<? extends Response<T>> apply(Throwable throwable) throws Exception {
            return Observable.error(CustomException.handleException(throwable));
        }
    }

    /**
     * 服务其返回的数据解析
     * 正常服务器返回数据和服务器可能返回的exception
     *
     * @param <T>
     */
    private static class ResponseFunction<T> implements Function<Response<T>, ObservableSource<T>> {

        @Override
        public ObservableSource<T> apply(Response<T> tResponse) throws Exception {
            int code = tResponse.getCode();
            String message = tResponse.getMsg();
            if (code == 200) {
                return Observable.just(tResponse.getData());
            } else {
                return Observable.error(new ApiException(code, message));
            }
        }
    }
}

```
##### 1.6、 处理Exception
这里的Exception分为两部分：
* 自己本地产生的Exception，比如解析错误，网络链接错误等等。
* 服务器产生的Excption，比如404，503等等服务器返回的Excption。

定义ApiException统一处理：
```
public class ApiException extends Exception {
    private int code;
    private String displayMessage;

    public ApiException(int code, String displayMessage) {
        this.code = code;
        this.displayMessage = displayMessage;
    }

    public ApiException(int code, String message, String displayMessage) {
        super(message);
        this.code = code;
        this.displayMessage = displayMessage;
    }

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public String getDisplayMessage() {
        return displayMessage;
    }

    public void setDisplayMessage(String displayMessage) {
        this.displayMessage = displayMessage;
    }
}
```
对于本地产生的Exception如下处理：
```
public class CustomException {

    /**
     * 未知错误
     */
    public static final int UNKNOWN = 1000;

    /**
     * 解析错误
     */
    public static final int PARSE_ERROR = 1001;

    /**
     * 网络错误
     */
    public static final int NETWORK_ERROR = 1002;

    /**
     * 协议错误
     */
    public static final int HTTP_ERROR = 1003;

    public static ApiException handleException(Throwable e) {
        ApiException ex;
        if (e instanceof JsonParseException
                || e instanceof JSONException
                || e instanceof ParseException) {
            //解析错误
            ex = new ApiException(PARSE_ERROR, e.getMessage());
            return ex;
        } else if (e instanceof ConnectException) {
            //网络错误
            ex = new ApiException(NETWORK_ERROR, e.getMessage());
            return ex;
        } else if (e instanceof UnknownHostException || e instanceof SocketTimeoutException) {
            //连接错误
            ex = new ApiException(NETWORK_ERROR, e.getMessage());
            return ex;
        } else {
            //未知错误
            ex = new ApiException(UNKNOWN, e.getMessage());
            return ex;
        }
    }
}
```
这些异常如何抛出呢 ？ 那就看下上面提到的 ResponseTransformer 中处理异常的代码吧。两种异常都通过Observable.error发射出去。
```
// 本地异常处理
Observable.error(CustomException.handleException(throwable));

// 对服务器放回的code进行判断
 int code = tResponse.getCode();
 String message = tResponse.getMsg();
 if (code == 200) {       
     return Observable.just(tResponse.getData());
 } else {
     return Observable.error(new ApiException(code, message));
  }
```

##### 1.7、线程切换
使用过Rxjava对Rxjava线程切换应该比较熟悉，这里将线程切换单独封装成一个类再结合compose使用。
```
public class SchedulerProvider implements BaseSchedulerProvider {

    @Nullable
    private static SchedulerProvider INSTANCE;

    // Prevent direct instantiation.
    private SchedulerProvider() {
    }

    public static synchronized SchedulerProvider getInstance() {
        if (INSTANCE == null) {
            INSTANCE = new SchedulerProvider();
        }
        return INSTANCE;
    }

    @Override
    @NonNull
    public Scheduler computation() {
        return Schedulers.computation();
    }

    @Override
    @NonNull
    public Scheduler io() {
        return Schedulers.io();
    }

    @Override
    @NonNull
    public Scheduler ui() {
        return AndroidSchedulers.mainThread();
    }

    @NonNull
    @Override
    public <T> ObservableTransformer<T, T> applySchedulers() {
        return observable -> observable.subscribeOn(io())
                .observeOn(ui());
    }
}
```

#### 2、实战
封装好之后，接下来就是实际使用了。不过在实际开发中可能是先实现逻辑再去封装，很多东西需要在代码写完之后才知道要去封装起来的。

##### 2.1、初始化
BaseApplication中初始化。
```
public class BaseApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        NetWorkManager.getInstance().init();
    }
}
```

##### 2.2、搭建好MVP结构，在Presenter中使用。

```
    Disposable disposable = model.getCarList("xxxxxx")
                .compose(ResponseTransformer.handleResult())
                .compose(schedulerProvider.applySchedulers())
                .subscribe(carBeans -> {
                    // 处理数据 直接获取到List<JavaBean> carBeans
                    view.getDataSuccess();
                }, throwable -> {
                    // 处理异常
                    view.getDataFail();
                });

        mDisposable.add(disposable);
```
两个回调，回调之后View层做出相应的UI变化即可。
这里用了compose的使用就不多做解释。这里compose结合线程的切换封装和变换封装，减少了冗余代码。
关于compose的链接：[https://www.jianshu.com/p/3d0bd54834b0](https://www.jianshu.com/p/3d0bd54834b0)
最后的目录结构：   
![目录结构](http://upload-images.jianshu.io/upload_images/1930161-d8fc4636916bc187.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 3、需要注意的地方
1、很多时候需要实现效果才能再去进行封装。           
2、在封装框架的时候封装的是逻辑层，不要在逻辑层中写业务层的代码。     

源码地址：[https://github.com/AxeChen/retrofit2_rxjava2](https://github.com/AxeChen/retrofit2_rxjava2)
