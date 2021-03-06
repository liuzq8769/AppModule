# ApplyModule
助力打造一个搭建android应用框架的工具模块组，包含RxHttp(RxJava2 + Retrofit2 + OkHttp3)网络请求、Multistate Layout多状态布局（error、empty、loading、content）、Bottom Bar一个自定义的视图组件，模仿新材料设计底部导航模式、类似商城加减控件、流式布局单选多选、搜索控件、指纹识别、非常实用的Utils类。

 ### 特点
   * 1.基于RxJava2和Retrofit2重构，便捷使用，例子是采用MVP模式，Activity和Fragment均进行封装处理。
   * 2.重写FrameLayout，构建多状态布局。
   * 3.重写LinearLayout，构建Bottom Bar底部导航模式。
   * 4.重写LinearLayout，构建类似商城加减控件。
   * 5.继承ViewGroup，自定义FlowLayout构建实用单选、多选组件。
   * 6.重写LinearLayout，构建常用搜索功能控件。
   * 7.适配大部分机型指纹识别功能（三星、魅族厂商提供专有SDK）。
   * 8.本着省事的原则，将平时用到的Utils类进行归类。

# 如何使用它

## Step 1.先在 build.gradle(Project:XXXX) 的 repositories 添加:

```gradle
allprojects {
	repositories {
		...
		maven { url "https://jitpack.io" }
	}
}
```

## Step 2. 然后在 build.gradle(Module:app) 的 dependencies 添加:

```gradle
dependencies {
	//基础工具库（必选）
 	implementation 'com.github.Lion7k.ApplyModule:commlibrary:v1.0.2'

	//rxhttp网络请求库（可选）
	implementation 'com.github.Lion7k.ApplyModule:rxhttp:v1.0.2'

	//多状态布局库（可选）
	implementation 'com.github.Lion7k.ApplyModule:statusview:v1.0.2'

	//定义的视图组件，模仿新材料设计底部导航模式（可选）
 	implementation 'com.github.Lion7k.ApplyModule:bottombar:v1.0.2'
	
	//定义的视图组件，类似商城加减控件（可选）
 	implementation 'com.github.Lion7k.ApplyModule:addsubtractview:v1.0.2'
	
	//定义的视图组件，流式布局构建单选、多选功能（可选）
 	implementation 'com.github.Lion7k.ApplyModule:flowlayout:v1.0.2'
	
	//定义的视图组件，搜索功能控件（可选）
 	implementation 'com.github.Lion7k.ApplyModule:searchbox:v1.0.2'
	
	//适配大部分机型指纹识别（可选）
 	implementation 'com.github.Lion7k.ApplyModule:fingerprintidentify:v1.0.2'
}
```
## Step 3. 在application类里边进行初始化配置
> ##### 在自己的Application的onCreate方法中进行初始化配置
```
public class App extends BaseApplication {

    @Override
    public void onCreate() {
        super.onCreate();
        ToastUtils.init(this);
        PreferencesUtils.initPrefs(this);
    }
}
```

## 以下详细描述各个模块使用方式:
### 一、rxhttp网络请求库
#### 1.在自己的Application的onCreate方法中进行初始化配置
```
   /**
     * 全局请求的统一配置（以下配置根据自身情况选择性的配置即可）
     */
    private void initRxHttpUtils() {

        //一个项目多url的配置方法
        RxUrlManager.getInstance().setMultipleUrl(AppUrlConfig.getAllUrl());

        RxHttpUtils
                .getInstance()
                .init(this)
                .config()
                //自定义factory的用法
                //.setCallAdapterFactory(RxJava2CallAdapterFactory.create())
                //.setConverterFactory(ScalarsConverterFactory.create(),GsonConverterFactory.create(GsonAdapter.buildGson()))
                //配置全局baseUrl
                .setBaseUrl("https://www.wanandroid.com/")
                //开启全局配置
                .setOkClient(createOkHttp());

//        TODO: 2018/5/31 如果以上OkHttpClient的配置满足不了你，传入自己的 OkHttpClient 自行设置
//        OkHttpClient.Builder builder = new OkHttpClient.Builder();
//
//        builder
//                .addInterceptor(log_interceptor)
//                .readTimeout(10, TimeUnit.SECONDS)
//                .writeTimeout(10, TimeUnit.SECONDS)
//                .connectTimeout(10, TimeUnit.SECONDS);
//
//        RxHttpUtils
//                .getInstance()
//                .init(this)
//                .config()
//                .setBaseUrl(BuildConfig.BASE_URL)
//                .setOkClient(builder.build());

    }

    private OkHttpClient createOkHttp() {
        //        获取证书
//        InputStream cerInputStream = null;
//        InputStream bksInputStream = null;
//        try {
//            cerInputStream = getAssets().open("YourSSL.cer");
//            bksInputStream = getAssets().open("your.bks");
//        } catch (IOException e) {
//            e.printStackTrace();
//        }

        OkHttpClient okHttpClient = new OkHttpConfig
                .Builder(this)
                //添加公共请求头
                .setHeaders(new BuildHeadersListener() {
                    @Override
                    public Map<String, String> buildHeaders() {
                        HashMap<String, String> hashMap = new HashMap<>();
                        hashMap.put("appVersion", BuildConfig.VERSION_NAME);
                        hashMap.put("client", "android");
                        hashMap.put("token", "your_token");
                        hashMap.put("other_header", URLEncoder.encode("中文需要转码"));
                        return hashMap;
                    }
                })
                //添加自定义拦截器
                //.setAddInterceptor()
                //开启缓存策略(默认false)
                //1、在有网络的时候，先去读缓存，缓存时间到了，再去访问网络获取数据；
                //2、在没有网络的时候，去读缓存中的数据。
                .setCache(true)
                .setHasNetCacheTime(10)//默认有网络时候缓存60秒
                //全局持久话cookie,保存到内存（new MemoryCookieStore()）或者保存到本地（new SPCookieStore(this)）
                //不设置的话，默认不对cookie做处理
                .setCookieType(new SPCookieStore(this))
                //可以添加自己的拦截器(比如使用自己熟悉三方的缓存库等等)
                //.setAddInterceptor(null)
                //全局ssl证书认证
                //1、信任所有证书,不安全有风险（默认信任所有证书）
                //.setSslSocketFactory()
                //2、使用预埋证书，校验服务端证书（自签名证书）
                //.setSslSocketFactory(cerInputStream)
                //3、使用bks证书和密码管理客户端证书（双向认证），使用预埋证书，校验服务端证书（自签名证书）
                //.setSslSocketFactory(bksInputStream,"123456",cerInputStream)
                //设置Hostname校验规则，默认实现返回true，需要时候传入相应校验规则即可
                //.setHostnameVerifier(null)
                //全局超时配置
                .setReadTimeout(10)
                //全局超时配置
                .setWriteTimeout(10)
                //全局超时配置
                .setConnectTimeout(10)
                //全局是否打开请求log日志
                .setDebug(BuildConfig.DEBUG)
                .build();

        return okHttpClient;
    }
```
#### 2.默认已实现三种数据格式
* 1、CommonObserver （直接写自己的实体类即可，不用继承任何base）
* 2、StringObserver (直接String接收数据)
* 3、DataObserver (适合{"code":200,"msg":"描述",data:{}}这样的格式，需要使用BaseData<T> ,其中T为data中的数据模型
> ##### 如果以上三种不能满足你的需要，可以分别继承对应的baseObserver方法实现自己的逻辑
	
### 二、statusview多状态布局库
#### 1.主要功能
* 1、可在 Activity、Fragment 、XML 中使用，可作用于 XML 的根布局 View 或其子 View
* 2、默认支持 Loading、Empty、Error 三种状态布局，可进行常规配置
* 3、可自定义状态布局，并提供对应接口来完成需要的配置
* 4、状态布局懒加载，仅在初次显示时初始化
#### 2.初始化
##### 可以直接在 XML 中初始化：
```
<com.shehuan.statusview.StatusView
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!--your layout-->

</com.shehuan.library.StatusView>
```
##### 也可以在 Activity、Fragment中初始化：
```
// 作用于 Activity 根布局 View
statusView = StatusView.init(Activity activity);
// 作用于 Activity 布局文件中指定的 View
statusView = StatusView.init(Activity activity, @IdRes int viewId);
// 作用于 Fragment 布局文件中指定的 View
statusView = StatusView.init(Fragment fragment, @IdRes int viewId);
```
##### 注意事项：
* 1、当 Fragment 布局文件的根 View 使用 StatusView 时，为避免出现的异常问题，建议在 XML 中初始化！
* 2、当直接在 Fragment 中使用时，init()方法需要在onCreateView()之后的生命周期方法中执行！
#### 3.配置
##### 如果使用默认的状态布局，可以通过如下方式配置布局：
```
statusView.config(new StatusViewBuilder.Builder()
                .setLoadingTip() // loading 提示信息
                .setEmptyip() // empty 提示信息
                .setErrorTip() // error 提示信息
                .setTipColor() // 提示信息颜色
                .setTipSize() // 提示信息字体大小
                .setEmptyIcon() // empty 图标
                .setErrorIcon() // error 图标
                .showEmptyRetry() // 是否显示 empty 重试按钮
                .showErrorRetry() // 是否显示 error 重试按钮
                .setEmptyRetryText() // empty 重试按钮文字
                .setErrorRetryText() // error 重试按钮文字
                .setRetryColor() // 重试按钮文字颜色
                .setRetrySize() // 重试按钮字体大小
                .setRetryDrawable() // 重试按钮 drawable 背景
                .setOnEmptyRetryClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View view) {
                        // empty 重试按钮点击事件
                    }
                })
                .setOnErrorRetryClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        // error 重试按钮点击事件
                    }
                })
                .build());
```
##### 如果需要使用自定义状态布局，可以通过如下方式设置：
```
statusView.setLoadingView(@LayoutRes int layoutId);
statusView.setEmptyView(@LayoutRes int layoutId);
statusView.setErrorView(@LayoutRes int layoutId);	
```
##### 或者在 XML 通过自定义属性配置，自定义属性的声明如下
```
<declare-styleable name="StatusView">
    <attr name="sv_loading_view" format="reference" />
    <attr name="sv_empty_view" format="reference" />
    <attr name="sv_error_view" format="reference" />
</declare-styleable>	
```
##### 使用自定义状态布局时，如果需要进行一些配置可通过
###### statusView.setOnXXXViewConvertListener()系列方法来完成，例如：
```
statusView.setOnErrorViewConvertListener(new StatusViewConvertListener() {
    @Override
    public void onConvert(ViewHolder viewHolder) {

    }
});
```
#### 4.切换状态布局
```
statusView.showLoadingView();
statusView.showEmptyView();
statusView.showErrorView();
statusView.showContentView(); // 即原始的页面内容布局
```
#### 5.更自由的用法
##### 如果不想局限于 Loading、Empty、Error 三种状态，那么下面的用法会更适合你：
```
// 添加指定索引对应的状态布局
statusView.setStatusView(int index, @LayoutRes int layoutId)
// 为指定索引的状态布局设置初次显示的监听事件，用来进行状态布局的相关初始化
statusView.setOnStatusViewConvertListener(int index, StatusViewConvertListener listener)
// 显示指定索引的状态布局
statusView.showStatusView(int index)
```

##### 更多使用细节可参考demo！
