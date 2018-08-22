Google I/O 2014 发布了Material Design。希望统一 Android平台设计语言规范。然而再国内的很多产品和设计师并不吃这一套，还是各种仿IOS的UI。作为一个Google粉当然要学会Android Material控件的使用。而且这些控件使用起来非常方便。以下是Android Material常用控件的整理。
**请注意：介绍了多个控件、多图预警，流量党珍惜下流量。**
>介绍的控件：  
1、Toolbar和Menu。
2、基于CoordinatorLayout的联动。
3、NavigationView。
4、CardView。
5、TextInputLayout。
6、RecyclerView。
7、TabLayout。
8、SnackBar
9、FloatingActionButton

github地址：https://github.com/AxeChen/MaterialDesignSimple  

![整理](http://upload-images.jianshu.io/upload_images/1930161-b40e0c632c405767.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 1、Toolbar
ToolBar是Android 5.0推出的一个新的导航控件用于取代之前的ActionBar。

![Toolbar](http://upload-images.jianshu.io/upload_images/1930161-ff1ec62ab2591dc8.gif?imageMogr2/auto-orient/strip)
#### 1.1、基本使用
##### 导入依赖
```
compile 'com.android.support:appcompat-v7:25.3.1'
```
##### 修改主题
我们需要使用Toolbar代替Actionbar，就必须将需要使用Toolbar的Activity的Theme设为NoActionBar。
```
    <style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>
```
由于示例代码中只有一个Activity，所以我将整个AppTheme继承了Theme.AppCompat.Light.NoActionBar。
如果实际情况只有·部分Activity使用Toolbar，可以单独为这些Activity设置没有ActionBar的主题。

#### 布局文件中定义Toolbar
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.mg.axe.toolbarsimple.MainActivity">

    <android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/actionBarSize"
        android:background="@color/colorPrimary"
        app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
        app:theme="@style/ThemeOverlay.AppCompat.ActionBar" />
</RelativeLayout>
```
#### setSupportActionBar
这个方法是 AppCompatActivity 中的方法，所以使用的 Activity 必须继承 AppCompatActivity。
```
   private Toolbar toolbar;
   private void initToolbar() {
        toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
    }
```
通过以上三步，Toolbar就能展示了。
#### 1.2、修改toolbar的属性
##### 通过调用Toolbar提供的方法。
可以直接调用Toolbar的一些方法来改变样式：
我写的示例代码中调用的Toolbar方法。
```
    private void setToolbarProperty() {
        // 设置正标题
        toolbar.setTitle("正标题");
        // 设置副标题
        toolbar.setSubtitle("副标题");
        // 设置左边按钮图片
        toolbar.setNavigationIcon(R.mipmap.ic_launcher_round);
        // 设置(Log)标题与左边按钮之间图标
        toolbar.setLogo(R.mipmap.ic_launcher);
    }
```
实际上Toolbar提供修改Toolbar属性的方法非常多，远不止以上提供的方法。
#####  在xml文件里修改
先导入命名空间:
```
 xmlns:app="http://schemas.android.com/apk/res-auto"
```
![在xml文件里修改](http://upload-images.jianshu.io/upload_images/1930161-dd233d84cb139e14.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

属性太多就不整理了。**对修改Toolbar的属性，这篇简书整理得不错，可以直接拿来用:**http://www.jianshu.com/p/2c21676033db



#####  通过Actionbar来修改
通过Actionbar的一些方法也能够修改Toolbar的一些样式（因为里面用了代理模式，具体还在研究中）。
![通过Actionbar来修改](http://upload-images.jianshu.io/upload_images/1930161-17d051a26b037ce8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这篇简书也有提到 ： http://www.jianshu.com/p/7b5c99e1cfa3 
部分方法的作用：http://blog.csdn.net/andygo_520/article/details/51439688

#### 1.3、Toolbar点击的回调
Toolbar的点击回调主要是左边的按钮点击回调和toolbar上面的Menu点击回调。
**对于Menu的点击回调更习惯重写onOptionsItemSelected方法来监听回调，后面的Menu会提到。**
```
   // 点击左侧按钮监听
  toolbar.setNavigationOnClickListener(new View.OnClickListener() {
      @Override
      public void onClick(View v) {
          Toast.makeText(MainActivity.this, "点击了Navigation按钮", Toast.LENGTH_SHORT).show();
      }
  });
   // toolbar的Menu回调
  toolbar.setOnMenuItemClickListener(new Toolbar.OnMenuItemClickListener() {
    @Override
    public boolean onMenuItemClick(MenuItem item) {
        return false;
    }
});
```

#### 1.4、Toolbar添加Menu
Menu是Toolbar非常重要的部分，可以将一些功能放在Menu上来实现，常见的有发送，分享等等。

##### 添加Menu
首先必须在Activity重写onCreateOptionsMenu方法，添加Menu的操作都在这个方法中执行。
```
@Override
    public boolean onCreateOptionsMenu(Menu menu) { 
            return super.onCreateOptionsMenu(menu);
    }
```
添加Menu有两种方式：
* 通过加载Menu资源文件
* 代码添加

如果通过资源文件添加，先要在res/menu下添加一个menu的资源文件。
```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/action_settings"
        android:icon="@android:drawable/ic_menu_set_as"
        android:orderInCategory="100"
        android:title="设置"
        app:showAsAction="never" />
    <item
        android:id="@+id/action_share"
        android:icon="@android:drawable/ic_menu_share"
        android:orderInCategory="100"
        android:title="分享"
        app:showAsAction="always" />
    <item
        android:id="@+id/action_edit"
        android:icon="@android:drawable/ic_menu_edit"
        android:orderInCategory="100"
        android:title="编辑"
        app:showAsAction="ifRoom" />
</menu>
```
如果是通过代码添加，onCreateOptionsMenu方法中调用menu.add()方法。
```
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        //使用代码创建menu
        //menu.add(0, 0, 0, "搜索").setShowAsAction(MenuItem.SHOW_AS_ACTION_ALWAYS);
        menu.add(0, 0, 0, "搜索").setIcon(android.support.v7.appcompat.R.drawable.abc_ic_search_api_material).setShowAsAction(MenuItem.SHOW_AS_ACTION_ALWAYS);
        //使用加载xml文件的方式创建
        getMenuInflater().inflate(R.menu.test_menu, menu);
        //添加子菜单
        menu.addSubMenu(0, 1, 0, "submenu").setIcon(R.mipmap.ic_launcher).addSubMenu(0, 2, 0, "submenu1");
        return super.onCreateOptionsMenu(menu);
    }
```
这里有个重要的属性showAsAction，选项和作用如下：
never：永远不显示的ToolBar上，只放在OverFlow按钮中（最右边三个点的按钮）。只有OverFlow按钮才会弹出。
always ：总会展示在Toolbar上。
ifRoom ：如Toolbar还有足够的空间，则显示在Toolbar上。否则就放在OverFlow按钮中。
withText，collapseActionView没用过，看下这篇博客的说明吧：http://www.jianshu.com/p/4106e1414d64


#### 3.5、监听Menu点击
首先重写onOptionsItemSelected方法。
根据MenuItem.getItemId()的方法即可找点击的Menu。
如果是加载Menu资源文件，那么ItemId就是给Item设置的id。
```
 <item
        android:id="@+id/action_settings"
        android:icon="@android:drawable/ic_menu_set_as"
        android:orderInCategory="100"
        android:title="设置"
        app:showAsAction="never" />
```
如果通过代码添加的Menu，那么ItemId就是调用add方法传入的itemid。看下add方法的源码就可以找到。
```
public MenuItem add(int groupId, int itemId, int order, CharSequence title);
```
 监听点击的代码。
```
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case 0:
                Toast.makeText(this, "点击搜索", Toast.LENGTH_SHORT).show();
                break;
            case 1:
                Toast.makeText(this, "点击submenu", Toast.LENGTH_SHORT).show();
                break;
            case 2:
                Toast.makeText(this, "submenu1", Toast.LENGTH_SHORT).show();
                break;
            case R.id.action_settings:
                Toast.makeText(this, "点击设置", Toast.LENGTH_SHORT).show();
                break;
            case R.id.action_edit:
                Toast.makeText(this, "点击编辑", Toast.LENGTH_SHORT).show();
                break;
            case R.id.action_share:
                Toast.makeText(this, "点击分享", Toast.LENGTH_SHORT).show();
                break;
        }
        return super.onOptionsItemSelected(item);
    }
```
附上Fragment中使用Menu的博客：http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2013/0104/777.html

### 2、基于CoordinatorLayout的联动
CoordinatorLayout是Android Material Design的重要组件，实现协调其他组件实现联动的效果。
#### 2.1、 CollapsingToolbarLayout
效果图：
![CollapsingToolbarLayout
](http://upload-images.jianshu.io/upload_images/1930161-11f74f963543d580.gif?imageMogr2/auto-orient/strip)

##### 添加依赖
```
 compile 'com.android.support:design:25.3.1'
 compile 'com.android.support:appcompat-v7:25.3.1'
```

##### 布局结构  
```
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:context="com.mg.axe.collapsingtoolbarlayout.MainActivity">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/app_bar"
        android:layout_width="match_parent"
        android:layout_height="@dimen/app_bar_height"
        android:fitsSystemWindows="true"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/toolbar_layout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorPrimary"
            app:expandedTitleMarginEnd="64dp"
            app:expandedTitleMarginStart="48dp"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">

            <ImageView
                android:id="@+id/backdrop"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:fitsSystemWindows="true"
                android:scaleType="centerCrop"
                android:src="@mipmap/yangmi"
                app:layout_collapseMode="parallax" />

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin"
                app:popupTheme="@style/AppTheme.PopupOverlay" />

        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>

    <android.support.v4.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">
    </android.support.v4.widget.NestedScrollView>
</android.support.design.widget.CoordinatorLayout>
```
既然是基于CoordinatorLayout，所以控件的父布局就是CoordinatorLayout。
重要的属性(这些没有设置是无法实现效果的)：
* CollapsingToolbarLayout中的属性
```app:layout_scrollFlags="scroll|exitUntilCollapsed“```
决定CollapsingToolbarLayout的滑动方式，这些flag的作用：
>**enterAlways:** 一旦向上滚动这个view就可见。
>**enterAlwaysCollapsed:** 当你定义了一个minHeight，那么view将在到达这个最小高度的时候开始显示，并且从这个时候开始慢慢展开，当滚动到顶部的时候展开完。
>**exitUntilCollapsed:** 当你定义了一个minHeight，这个view将在滚动到达这个最小高度的时候消失。

**当然这些flag最好是实际使用看效果。**

* NestedScrollView中的属性
```app:layout_behavior="@string/appbar_scrolling_view_behavior”```
这个属性用于通知AppBarLayout 。View（这里是NestedScrollView
）何时发生了滚动事件。

##### 1.2、向上滑动隐藏Toolbar
效果图：
![向上滑动隐藏Toolbar](http://upload-images.jianshu.io/upload_images/1930161-ed20f7cc1336f4ea.gif?imageMogr2/auto-orient/strip)

##### 添加依赖
```
 compile 'com.android.support:design:25.3.1'
 compile 'com.android.support:appcompat-v7:25.3.1'
```

##### 布局结构
```
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:context="com.mg.axe.coordinatorlayoutscrollsimple.MainActivity">

    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar">
        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?android:attr/actionBarSize"
            android:background="@color/colorPrimary"
            app:layout_scrollFlags="scroll|enterAlways"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
            app:theme="@style/ThemeOverlay.AppCompat.ActionBar" />

    </android.support.design.widget.AppBarLayout>
 <android.support.v4.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">
    </android.support.v4.widget.NestedScrollView>

</android.support.design.widget.CoordinatorLayout>
```
这里同样需要设置这两个属性：
``` app:layout_scrollFlags="scroll|enterAlways"```
```app:layout_behavior="@string/appbar_scrolling_view_behavior"```


##### 1.3、其他的一些问题
**通过以上的两个例子可以总结出基于CoordinatorLayout联动的两点规律：**
1、父布局肯定是CoordinatorLayout
2、一定会设置```app:layout_scrollFlags```和```app:layout_behavior```两个属性。
**滑动的View：**滑动的view官方建议使用NestedScrollView或者RecyclerView。ListView在5.0以下就没有效果了。
**高阶的使用：**高阶使用当然是自定义behavior了。http://blog.csdn.net/gdutxiaoxu/article/details/53453958

### 3、NavigationView
NavigationView用来实现侧滑导航的布局。
效果图：
![NavigationView](http://upload-images.jianshu.io/upload_images/1930161-f2d1f401069d62c4.gif?imageMogr2/auto-orient/strip)
如果使用Android studio新建Android项目，在选择Activity时会有Navigation Drawer Activity。选择它会生成一个带有侧滑导航栏的Activity。
![新建Navigation Drawer Activity](http://upload-images.jianshu.io/upload_images/1930161-dd676e990858ccfa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 添加依赖
```
    compile 'com.android.support:appcompat-v7:25.3.1'
    compile 'com.android.support:design:25.3.1'
```

##### 布局结构

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:openDrawer="start">

    <include
        layout="@layout/app_bar_main"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

    <android.support.design.widget.NavigationView
        android:id="@+id/nav_view"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:fitsSystemWindows="true"
        app:headerLayout="@layout/nav_header_main"
        app:menu="@menu/activity_main_drawer" />
</android.support.v4.widget.DrawerLayout>
```
NavigationView中，主要的两个属性：
```
 app:headerLayout="@layout/nav_header_main"
 app:menu="@menu/activity_main_drawer" 
```
如下图所示，NavigationView分为两部分，headerView 和 menu：
![QQ截图20170827155506.png](http://upload-images.jianshu.io/upload_images/1930161-1e1eecb95b5a2021.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
headview：就是普通的布局。
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">

    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:layout_marginTop="40dp"
        android:src="@mipmap/ic_launcher_round" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:layout_marginTop="20dp"
        android:text="AxeChen"
        android:textColor="@android:color/secondary_text_light"
        android:textSize="18sp" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:layout_marginTop="10dp"
        android:text="不务正业的码农！" />

</LinearLayout>
```
menu:需要在res/menu下定义：
```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">

    <group android:checkableBehavior="single">
        <item android:id="@+id/nav_camera" android:icon="@drawable/ic_menu_camera"
            android:title="Import" />
        <item android:id="@+id/nav_gallery" android:icon="@drawable/ic_menu_gallery"
            android:title="Gallery" />
        <item android:id="@+id/nav_slideshow" android:icon="@drawable/ic_menu_slideshow"
            android:title="Slideshow" />
        <item android:id="@+id/nav_manage" android:icon="@drawable/ic_menu_manage"
            android:title="Tools" />
    </group>
    <item android:title="Communicate">
        <menu>
            <item android:id="@+id/nav_share" android:icon="@drawable/ic_menu_share"
                android:title="Share" />
            <item android:id="@+id/nav_send" android:icon="@drawable/ic_menu_send"
                android:title="Send" />
        </menu>
    </item>
</menu>
```
#### 3.1 修改样式
* **用代码修改 ：** NavigationView提供了很多setXXX（）的方法。
*  **xml文件中修改：**
![xml文件中修改](http://upload-images.jianshu.io/upload_images/1930161-0158284920ad9171.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**记得添加命名空间**
 

#### 3.1、其他的问题
**menu图片为灰色问题**
在使用的时候，会发现所有的图片多会展示成灰色，不管你的图片是否为彩色的图片。
如下方法解决：
```
NavigationView navigationView = (NavigationView) findViewById(R.id.navigation_view);  
navigationView.setItemIconTintList(null);  
```

### 4、CardView
效果图：
![CardView](http://upload-images.jianshu.io/upload_images/1930161-8395dde4f8f7c2b9.gif?imageMogr2/auto-orient/strip)
CardView并不属于design包，所以使用CardView必须先添加依赖：
```
    compile 'com.android.support:cardview-v7:25.3.1'
```
CardView是一个继承FrameLayout的View。用于布局的圆角、阴影等效果。基本使用只要知道CardView一些常用属性即可。

| 属性 | 作用|
|:----------------------------:|:------------------------------------:|
| cardElevation | 阴影的大小 |
| cardElevation | 阴影最大高度 |
| cardElevation | 卡片的背景色 |
| cardCornerRadius | 卡片的圆角大小 |
| contentPadding | 卡片内容于边距的间隔 |
| contentPaddingBottom| 卡片内容与底部的边距 |
| contentPaddingTop| 卡片内容与顶部的边距 |
| contentPaddingLeft| 卡片内容与左边的边距 |
| contentPaddingRight| 卡片内容与右边的边距 |
| contentPaddingStart| 卡片内容于边距的间隔起始 |
| contentPaddingEnd | 卡片内容于边距的间隔终止 |
| cardUseCompatPadding | 设置内边距，V21+的版本和之前的版本仍旧具有一样的计算方式 |
| cardPreventConrerOverlap| 在V20和之前的版本中添加内边距，这个属性为了防止内容和边角的重叠 |

如果直接给CardView添加```android:foreground="?attr/selectableItemBackground"```就会加上水波纹点击反馈。

** 遇到的问题：**如果你在RecyclerView中使用CardView，如果你用的context是getApplicationContext()，可能会出现展示出夜间模式的效果。我当时使用getApplicationContext()是为了防止内存溢出。具体原因暂时还不清楚。
出现问题的代码。
```
    public RecommendAdapter(Context context, List<ParkBean> parkBeanList) {
        this.context = context.getApplicationContext();
        this.parkBeanList = parkBeanList;
    }
```
解决方案：不使用  context.getApplicationContext(); 
```
  public RecommendAdapter(Context context, List<ParkBean> parkBeanList) {
        //this.context = context.getApplicationContext();
        this.context = context;
        this.parkBeanList = parkBeanList;
    }
```


### 5、TextInputLayout
#### 5.1 基本使用
使用TextInputLayout控件的效果，需要将EditText作为TextInputLayout的子控件使用，**而且只能有一个EditText。**
##### 添加依赖
```
  compile 'com.android.support:design:25.3.1'
```
##### 布局结构
```
    <android.support.design.widget.TextInputLayout
        android:id="@+id/tilayoutUserName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="20dp"
        android:layout_marginRight="20dp"
        android:layout_marginTop="30dp"
        >
        <EditText
            android:id="@+id/et_name"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:hint="请输入用户名" />
    </android.support.design.widget.TextInputLayout>
```
效果图：
![TextInputLayout](http://upload-images.jianshu.io/upload_images/1930161-5dce5a6957be5874.gif?imageMogr2/auto-orient/strip)

##### 5.2 错误提示
效果图：
![验证输入的字符](http://upload-images.jianshu.io/upload_images/1930161-1d9430e116b6d310.gif?imageMogr2/auto-orient/strip)
TextInputLayout提供了非常方便的错误提示：
* 当需要错误提示时，调用TextInputLayout的```setError("错误提示文字")```。
* 如果要清空错误提示时，调用TextInputLayout的```setError(null)```。

以效果图中的用户名为例：
实现Editext的addTextChangedListener方法，监听输入字符。然后根据字符判断输入的字符串是否合理。
```
    private void initListener() {
        edName.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {
            }
            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {
            }
            @Override
            public void afterTextChanged(Editable s) {
                checkName(s.toString());
            }
        });
    }

    private void checkName(String s) {
        if (s.length() < 10) {
            nameInput.setError("名字长度太短");
        } else {
            nameInput.setError(null);
        }
    }
```


### 6、RecyclerView
**RecyclerView是Android程序员必学控件，如果现在还不会RecyclerView就赶紧学起来！**
RecyclerView可以实现GirdView，ListView和瀑布流的效果。可以设置滑动的方向。
###### 添加依赖
```
    compile 'com.android.support:appcompat-v7:25.3.1'
```

##### 6.1、LayoutManager
决定RecyclerView以一种怎样的形式展示由LayoutManager决定。
设置GridLayoutManager，实现GridView的效果。
设置LinearLayoutManager，实现ListView的效果。
设置StaggeredGridLayoutManager，实现瀑布流的的效果。
![LayoutManager](http://upload-images.jianshu.io/upload_images/1930161-30dfcf2f9d3294e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 6.2 RecyclerView.Adapter、 RecyclerView.ViewHolder
实现数据的展示需要用到Adapter和ViewHolder
##### 继承RecyclerView.Adapter的Adapter

必须重写的三个方法：   
**onCreateViewHolder：** 创建ViewHolder并返回，负责为给Item创建视图。
**onBindViewHolder：** 负责将数据绑定到Item的视图上。
**getItemCount：** 返回总数据条数。
```
    private class MyAdapter extends RecyclerView.Adapter {
        @Override
        public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
            return new ItemViewHolder(LayoutInflater.from(RecyclerListViewActivity.this).inflate(R.layout.item_list, parent, false));
        }
        @Override
        public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
            bindItemViewHolder((ItemViewHolder) holder, position);
        }
        @Override
        public int getItemCount() {
            return items.size();
        }
    }
```

##### 继承RecyclerView.ViewHolder的ViewHolder
和使用Listview的Viewholder差不多。里面是一些初始化控件的操作。
```
public class ItemViewHolder extends RecyclerView.ViewHolder {

        private ImageView imageView;
        private TextView tvTitle;
        private TextView tvDescription;

        public ItemViewHolder(View itemView) {
            super(itemView);
            imageView = (ImageView) itemView.findViewById(R.id.ivImg);
            tvTitle = (TextView) itemView.findViewById(R.id.tvTitle);
            tvDescription = (TextView) itemView.findViewById(R.id.tvDescription);
        }
}
```


#### 6.3 实现ListView、Gridview的效果
前面已经提到过设置不同LayoutManager会展示不同的效果。
实现Listview效果：
```
 recyclerView.setLayoutManager(new LinearLayoutManager(this, LinearLayoutManager.VERTICAL, false));
```
**注意：**LinearLayoutManager.VERTICAL为垂直滑动，如果设置成LinearLayoutManager.HORIZONTAL则会水平滑动。
效果图：
![ListView的效果](http://upload-images.jianshu.io/upload_images/1930161-e616f7ebe333230f.gif?imageMogr2/auto-orient/strip)

实现GridView效果：
```
recyclerView.setLayoutManager(new GridLayoutManager(this, 3, LinearLayoutManager.VERTICAL, false));
```
![GridView的效果](http://upload-images.jianshu.io/upload_images/1930161-358da735beec84b3.gif?imageMogr2/auto-orient/strip)

**至于瀑布流，由于平时使用比较少，这里就不做详细的解释了。**

### 7、TabLayout
效果图：
![基本的使用](http://upload-images.jianshu.io/upload_images/1930161-cafdd553841e3560.gif?imageMogr2/auto-orient/strip)
##### 添加依赖
```
  compile 'com.android.support:design:26.0.0-alpha1'
```

#### 7.1、如何添加Tab。

![添加Tab](http://upload-images.jianshu.io/upload_images/1930161-307066f209beb222.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上图可看出，TabLayout提供了addTab()的方法来添加Tab。初始化Tab之后可以调用```setXXX()```来决定是展示文字、图片或者自定义布局。

#### 7.2、TabMode
**TabLayout.MODE_SCROLLABLE**：当元素过多时会超出父布局，并可以滑动Tab，Tab的宽度为Tab的实际宽度。
**TabLayout.MODE_FIXED**：无论界面由多少元素都会充满父布局。并且平均分配Tab的宽度。

#### 7.3、监听Tab的选择状态
```
tabLayoutOne.addOnTabSelectedListener(new TabLayout.OnTabSelectedListener() {
            @Override
            public void onTabSelected(TabLayout.Tab tab) {
            }
            @Override
            public void onTabUnselected(TabLayout.Tab tab) {
            }
            @Override
            public void onTabReselected(TabLayout.Tab tab) {
            }
        });
```
#### 7.4、修改TabLayou的样式
*  **在布局文件中设置**  
在使用自定义属性时，记得添加命名空间。  
 ![布局中设置属性](http://upload-images.jianshu.io/upload_images/1930161-638164f0d5273c0f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*  **在代码中设置**
调用Tablayout的setXXX()的方法。  

* **在styles中修改**
继承**Widget.Design.TabLayout**修改TabLayout的样式。
继承**TextAppearance.Design.Tab**修改Tab的样式。
```
<style name="MyCustomTabLayout" parent="Widget.Design.TabLayout">
    <item name="tabMaxWidth">@dimen/tab_max_width</item>
    <item name="tabIndicatorColor">?attr/colorAccent</item>
    <item name="tabIndicatorHeight">2dp</item>
    <item name="tabPaddingStart">12dp</item>
    <item name="tabPaddingEnd">12dp</item>
    <item name="tabBackground">?attr/selectableItemBackground</item>
    <item name="tabTextAppearance">@style/MyCustomTabTextAppearance</item>
    <item name="tabSelectedTextColor">?android:textColorPrimary</item>
</style>
<style name="MyCustomTabTextAppearance" parent="TextAppearance.Design.Tab">
    <item name="android:textSize">14sp</item>
    <item name="android:textColor">?android:textColorSecondary</item>
    <item name="textAllCaps">true</item>
</style>
```
修改完毕之后记得将styles设置到TabLayout控件上。
#### 7.5、与Viewpager的联动
效果图：  ![与viewpager的联动](http://upload-images.jianshu.io/upload_images/1930161-1ffaaf9d034e1a1b.gif?imageMogr2/auto-orient/strip)
Viewpager+fragment+导航栏的这种UI结构。已经应用得非常广泛了，几乎是随处可见。TabLayout提供了和Viewpager联动方法，实现简单联动的方法也比较简单。  
**注：**这里代码只考虑了Tab展示Text的情况。如果Tab展示的是图片或者自定义View，会稍微麻烦点。
```
  viewPager.setAdapter(new PagerAdapter(getSupportFragmentManager()));
  tabLayout.setupWithViewPager(viewPager);
```
viewPager设置完PagerAdapter之后，调用 tabLayout.setupWithViewPager(viewPager);即可。
PagerAdapter重写getPageTitle方法，根据Viewpager页面的position来展示不同Tab的Text。
```
 @Override
  public CharSequence getPageTitle(int position) {
            return "第" + (position + 1) + "页";
   }
```

**不过这里还是埋了坑，有时候会出现Tab无法展示的情况（写过的都懂）。**
这个链接可以解决这个问题：http://blog.csdn.net/sundy_tu/article/details/52682246




### 8、SnackBar
##### 添加依赖
```
 compile 'com.android.support:design:26.0.0-alpha1'
``` 

效果图：
![SnackBar](http://upload-images.jianshu.io/upload_images/1930161-9b4f7bc121e0cc01.gif?imageMogr2/auto-orient/strip)
SnackBar和Toast的用法差不多，都是通过show方法展示。和Toast不同之处是能和用户交互，同时提供了一些展示和消失的回调，修改SnackBar的文字颜色也比较简单。
#### 8.1 SnackBar的基本使用
* **仅仅只提示文字：**
弹出提示文字之后一段时间后会消失。
```
Snackbar.make(button, "第一次使用SnackBar", Snackbar.LENGTH_SHORT).show();
```
* **提供一个可以点击的按钮：**
添加一个可以点击的按钮。
```
Snackbar.make(button, "第一次使用SnackBar", Snackbar.LENGTH_SHORT).setAction("确定", new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {

                    }
                }).show();
```
* **必须和用户交互之后才消失：**
Snackbar.make（）设置时间的参数改为：Snackbar.LENGTH_INDEFINITE。如果不设置这个参数，Snackbar都会在一段时间后消失。
```
 Snackbar.make(button, "第一次使用SnackBar", Snackbar.LENGTH_INDEFINITE).setAction("确定", new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {

                    }
                }).show();
```
*  **添加状态回调：**
监听Snackbar的展示和消失。
```
Snackbar.make(button, "第一次使用SnackBar", Snackbar.LENGTH_INDEFINITE).setAction("确定", new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {

                    }
                }).addCallback(new Snackbar.Callback() {
                    @Override
                    public void onShown(Snackbar sb) {
                        super.onShown(sb);
                        Toast.makeText(MainActivity.this, "弹出了", Toast.LENGTH_SHORT).show();
                    }

                    @Override
                    public void onDismissed(Snackbar transientBottomBar, int event) {
                        super.onDismissed(transientBottomBar, event);
                        Toast.makeText(MainActivity.this, "消失了", Toast.LENGTH_SHORT).show();
                    }
                }).show();
```
#### 8.2、修改Snackbar的样式
Snackbar只提供了一个修改按钮文字的方法：**snackbar.setActionTextColor(Color.GREEN)**
如需修改提示文字，背景的颜色等。可以通过**snackbar.getView()**获取到Snackbar对应的View进行修改。获取到Snackbar的View之后通过findById就能获取其他的一些控件。（Snackbar对应的布局文件在源码中可以找到。）
**注意：**这种方式还是存在风险。如果哪天Snackbar更新，Snackbar开发者修改了这些控件的Id，那就无法获取了。
```
 Snackbar snackbar = Snackbar.make(button, "第一次使用SnackBar", Snackbar.LENGTH_INDEFINITE).setAction("确定", new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {

                    }
                });
                snackbar.setActionTextColor(Color.GREEN);

  //修改背景
  snackbar.getView().setBackgroundResource(R.color.colorPrimaryDark);
  /* snackbar 并没有提供修改提示文字颜色的方法。不过可以通过找到
      snackbar的布局design_layout_snackbar_include通过布局可以找到textview的id。
     再通过snackbar.getView().findViewById(R.id.snackbar_text);
   */
  TextView textView = (TextView) snackbar.getView().findViewById(R.id.snackbar_text);
  textView.setTextColor(getResources().getColor(R.color.colorAccent));
  snackbar.show();
```
### 9、FloatingActionButton
FloatingActionButton：界面浮动的标签，一般用于页面关键功能入口。FloatingActionButton非常简单，知道了解FloatingActionButton的一些属性和点击回调即可。
效果图：

![FloatingActionButton](http://upload-images.jianshu.io/upload_images/1930161-0946a66b70c8dafe.gif?imageMogr2/auto-orient/strip)

常用属性：
**先加入命名空间！**

|属性|作用|
|:-------------:|:----:|
| fabSize | 定义FloatingActionButton的大小。auto（大） mini（小） normal（中）|
| elevation | 普通状态下的阴影深度 |
| pressedTranslationZ | 按下时的阴影深度 |
| backgroundTint | 默认展示的背景颜色|
| rippleColor | 按下时的颜色（5.0以后为水波纹的颜色）|
| layout_anchor | 定位其他控件，和其他控件边界相交|
| layout_anchorGravity | 和layout_anchor属性联用，在其他控件的相对位置 |
| useCompatPadding  | 设置内边距 |
| borderWidth | 边框宽度（使用的比较少，效果也不好看）|



点击回调：
```
 FloatingActionButton fabOne = (FloatingActionButton) findViewById(R.id.fabOne);
 fabOne.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Snackbar.make(v, "正在打开地图", Snackbar.LENGTH_SHORT).show();
            }
  });
```




### 最后
感谢各位程序员老爷看完这个又臭又长的博客！
源码地址：https://github.com/AxeChen/MaterialDesignSimple （欢迎star）
