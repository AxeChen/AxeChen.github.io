---
layout:     post
title:      intent-filter的action，category，data匹配规则
subtitle:   intent-filter的action，category，data匹配规则
date:       2016-09-27
author:     陈再峰
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - iOS
    - Activity
    - intent-filter
---

我们知道有两种方式来启动Activity，显示调用和隐式调用。当使用隐式调用时，又会涉及到IntentFilter的匹配规则。我确信大多数开发者很少关注隐式调用，因为平时开发中用到大多数是显示调用。例如：用Intent直接打开一个Activity，或者用Intent通过包名等其他信息打开另外一个应用等。而隐式调用则使用的比较少，当然也不是完全不使用。例如：当我们需要打开浏览器访问某个链接时，手机上可能存在多个浏览器，我们也无法拿到某一个浏览器的包名，那么一般情况下我们会写如下代码：  
```
  Intent intent = new Intent(); 
  intent.setAction("android.intent.action.View"); 
  intent.setDate(Uri.parser("链接地址"));
  startActivity(intent); 
```

执行完这段代码后，系统将会弹框提示选择哪个浏览器打开。只要这个Intent中的action(通过setAction()方法配置)能和Activity配置的过滤规则中的任何一个action相同即可匹配成功(这里后面会详细分析)。这里就说明Intent中action匹配到了多个Activity，所以系统会将所有能打开这个链接的应用展示出来供用户选择。这种通过action匹配activity的方式就是一种典型的隐式调用。
首先我们先分析显示调用和隐式调用的原理：
### 1、Activity的调用模式
  
#### a、显示调用
>**显示调用需要明确的指出被启动的对象的组件信息、包括包名和类名**

示例1：通过包名打开一个应用
```
  Intent intent = new Intent(Intent.ACTION_MAIN);
  intent.addCategory(Intent.CATEGORY_LAUNCHER);
  ComponentName cn = new ComponentName("com.mg.axe.testappa","com.mg.axe.testappa.MainActivity");
  intent.setComponent(cn);startActivity(intent);  
```

示例2： 打开一个Activity 
```
  Intent intent = new Intent(ActivityA.this,ActivityB.class); 
  startActivity(intent);
```

**(这种方式从严格上讲不算显示调用,因为显示调用的意义是 ：需要明确的指出被启动的对象的组件信息,包括包名和类名，这里并没有指出包名 。)**

#### b、隐式调用
>**需要Intent能匹配目标组件的IntentFilter中所设置的过滤信息.如果不匹配将无法启动目标Activity**

示例1：通过action方式匹配对应的Activity

```
  Intent intent = new Intent(); 
  intent.setAction("android.intent.action.View"); 
  startActivity(intent); 
```

运行结果会像这样：

![图一](http://upload-images.jianshu.io/upload_images/1930161-cb26dac7e7c5015a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  

为什么会匹配到这么多应用的Activity？ 因为在这些Activity的IntentFilter匹配规则中有如下规则：  

![图二](http://upload-images.jianshu.io/upload_images/1930161-a199fe4e6053159f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  

（**由于我们这里没有匹配其他的条件，所以会匹配到很多应用的Activity**，我们可以添加其他的匹配条件，比如入“category”，“data”的匹配来更加精确的匹配到所需要的Activity）

通过上面的实例，大概了解了显示调用和隐式调用的方式。接下来我们将重点放在隐式调用的IntentFilter的匹配中。
### 2.Action的匹配规则
>**Intent中的Action必须能够和Activity过滤规则中的Action匹配.(这里的匹配是完全相等). 一个过滤规则中有多个action,那么只要Intent中的action能够和Activity过滤规则中的任何一个action相同即可匹配成功。***

在AndroidManifest中添加AActivity的action的匹配规则**<action android:name="com.axe.mg.what" />**
```
<activity
 android:name=".AActivity"
 android:label="@string/app_name"
 android:theme="@style/AppTheme.NoActionBar">
 <intent-filter>
 <category android:name = "android.intent.category.DEFAULT" />
  <action android:name="com.axe.mg.what" />
 </intent-filter>
</activity>
```

然后调用以下方法即可匹配到AActivity:
```
public void match(){
 Intent intent = new Intent();
 intent.setAction("com.axe.mg.what");
 startActivity(intent);
}
```
注意：
1、系统定义了一些Action。当然我们也可以自己定义action。比如**<action android:name="com.axe.mg.what" />**    

2、在Activity中定义的Action匹配规则可能有多个，只要Intent中的action能够和Activity过滤规则中的任何一个action相同即可匹配成功。例如：
```
<activity
 android:name=".AActivity"
 android:label="@string/app_name"
 android:theme="@style/AppTheme.NoActionBar">
 <intent-filter>
 <category android:name = "android.intent.category.DEFAULT" />
  <action android:name="com.axe.mg.what" />
  <action android:name="com.axe.mg.how"/>
 </intent-filter>
</activity>
```
```
public void match() {   
  Intent intent = new Intent();    
  //只设置一个action。依旧能够成功。
  intent.setAction("com.axe.mg.what");    
  startActivity(intent);
}
```

### 3.category的匹配规则
>**如果Intent中的存在category那么所有的category都必须和Activity过滤规则中的category相同。才能和这个Activity匹配。Intent中的category数量可能少于Activity中配置的category数量，但是Intent中的这category必须和Activity中配置的category相同才能匹配。**

我们在Activity中配置category匹配规则：

```
<activity
 android:name=".AActivity"
 android:label="@string/app_name"
 android:theme="@style/AppTheme.NoActionBar">
 <intent-filter>
 <category android:name = "android.intent.category.DEFAULT" />
 <category android:name="aaa.bb.cc"/>
 <action android:name="com.axe.mg.what" />
 </intent-filter>
</activity>
```
运行以下代码可以匹配到AActivity：
```
public void match(){
 Intent intent = new Intent();
 intent.addCategory("aaa.bb.cc");
 intent.setAction("com.axe.mg.what");
 startActivity(intent);
}
```

注意：1、只通过category匹配是无法匹配到AActivity的。因为category属性是一个执行Action的附加信息。
所以只靠category是无法匹配的。像如下代码：

```
//没有setAction()无法匹配
public void match(){
 Intent intent = new Intent();
 intent.addCategory(Intent.CATEGORY_DEFAULT);
 intent.addCategory("aaa.bb.cc");
 startActivity(intent);
}
```

### 4.data的匹配规则
>**类似于action匹配，但是data有更复杂的结构**

**a.data的结构**
```
<data android:scheme="axe"
 android:host="axe"
 android:port="axe"
 android:path="axe"
 android:pathPattern="axe"
 android:pathPrefix="axe"
 android:mimeType="axe"/>
```

data 由两部分组成
**mineType** 和 **URI**

**mineType**： 指媒体类型 例如: image/jpeg vided/* ...

**URl** 可配置更多信息，类似于url。我们可以看下URI的结构
><scheme>://<host>:<port>/[<path>|<pathPrefix>|<pathPattern>]
content://com.axe.mg:100/fold/subfolder/etc
http://www.axe.com:500/profile/info

我们看下URL的属性：
>**Scheme：**URI的模式。如果URI中没有指定Scheme.那么整个URI无效。默认值为content 和 file。
**Host：**URI的host。比如www.axe.com。如果**指定了scheme和port，path等其他参数，但是host未指定，那么整个URI无效；**如果**只指定了scheme，没有指定host和其他参数，URI是有效的。**可以这样理解：一个完整的URI ：http://www.axe.com:500/profile/info 我将后面的prot 和path“:500/profile/info ”去掉，这个URI任然有效。如果我单独将www.axe.com 那这个URI就无效了。
**Port：**URI端口，当URI指定了scheme 和 host 参数时port参数才有意义。
**path：**用来匹配完整的路径，如：http://example.com/blog/abc.html，这里将 path 设置为 /blog/abc.html 才能够进行匹配；
**pathPrefix：** 用来匹配路径的开头部分，拿上面的 Uri 来说，这里将 pathPrefix 设置为 /blog 就能进行匹配了；
**pathPattern：** 用表达式来匹配整个路径。

**b.一些实例**
（1）**只匹配scheme**
```
<activity
 android:name=".CActivity"
 android:label="@string/app_name"
 android:theme="@style/AppTheme.NoActionBar">
 <intent-filter>
 <action android:name="test" />
 <category android:name="android.intent.category.DEFAULT" />
 <data android:scheme="axe" />
 </intent-filter>
 </activity>
</application>
```
匹配该Activity只需要data中scheme为axe就能匹配到
```
public void match(){
 Intent intent=new Intent();
 //只设置Intent的Data属性
 intent.setData(Uri.parse("axe://haha"));
 startActivity(intent);
}
```
（2）**匹配 scheme host port**
```
<activity
 android:name=".CActivity"
 android:label="@string/app_name"
 android:theme="@style/AppTheme.NoActionBar">
 <intent-filter>
 <action android:name="test" />
 <category android:name="android.intent.category.DEFAULT" />
 <data
 android:host="www.axe.com"
 android:port="8888"
 android:scheme="axe" />
 </intent-filter>
</activity>
```
匹配这个Activity需要 scheme 为 axe ，host 为 www.axe.com， port为8888才能匹配。 只要有一个不正确都无法匹配
```
public void match(View view){
 Intent intent=new Intent();
 //只设置Intent的Data属性
 intent.setData(Uri.parse("axe://www.axe.com:8888/mypath"));
 startActivity(intent);
}
```
（3）**匹配 scheme host path**
```
<activity
 android:name=".CActivity"
 android:label="@string/app_name"
 android:theme="@style/AppTheme.NoActionBar">
 <intent-filter>
 <action android:name="xx" />
 <category android:name="android.intent.category.DEFAULT" />
 <data
   android:host="www.axe.com"
   android:path="/mypath"
   android:scheme="axe" />
 </intent-filter>
</activity>
```
匹配这个Activity 必须 scheme为 axe， host 为 www.axe.com， path 为 mypath
才能匹配
```
public void match(View view) {
 Intent intent = new Intent();
 intent.setData(Uri.parse("axe://www.axe.com:4545/mypath"));
  //port不写任然能匹配. data中没有要求做匹配
  //intent.setData(Uri.parse("axe://www.axe.com/mypath"));
 startActivity(intent);
}
```
（4） **匹配 scheme host port path**
```
<activity
 android:name=".CActivity"
 android:label="@string/app_name"
 android:theme="@style/AppTheme.NoActionBar">
 <intent-filter>
 <action android:name="xx" />
 <category android:name="android.intent.category.DEFAULT" />
 <data
 android:host="www.axe.com"
 android:path="/mypath"
 android:port="8888"
 android:scheme="axe" />
 </intent-filter>
</activity>
```
匹配这个Activity 必须 scheme为 axe,，host 为 www.axe.com， path 为 mypath，
port 为8888 才能匹配
```
public void match(V) {
 Intent intent = new Intent();
 intent.setData(Uri.parse("axe://www.axe.com:8888/mypath"));
 startActivity(intent);
}
```
（5）**mintype匹配**
```
<activity
 android:name=".CActivity"
 android:label="@string/app_name"
 android:theme="@style/AppTheme.NoActionBar">
 <intent-filter>
 <action android:name="xx" />

 <category android:name="android.intent.category.DEFAULT" />
 <data
 android:mimeType="axe/abc"
 android:host="www.axe.com"
 android:path="/mypath"
 android:port="8888"
 android:scheme="axe" />
 </intent-filter>
</activity>
```
可以看到我们添加了mimeType。这种匹配方法我们需要做改变。我们不能使用
setType 和 setData ， 需要使用setDataAndType()。
从源码可以看出：setType() 会将URL设为null； setData()会将mineType设为null；以下为源码：
```
public Intent setType(String type) {
 mData = null;
 mType = type;
 return this;
}

public Intent setData(Uri data) {
 mData = data;
 mType = null;
 return this;
}
```
匹配这个Activity 必须 scheme为 axe， host 为 www.axe.com， path 为 mypath，
port 为8888， mineType 为 axe/abc才能匹配。
```
public void match() {
 Intent intent = new Intent();
//注意这个方法
 intent.setDataAndType(Uri.parse("axe://www.axe.com:8888/mypath"),"axe/abc");
 startActivity(intent);
}
```
（6）**Scheme的默认值content 和 file。**
```
<activity
 android:name=".CActivity"
 android:label="@string/app_name"
 android:theme="@style/AppTheme.NoActionBar">
 <intent-filter>
 <action android:name="xx" />
 <category android:name="android.intent.category.DEFAULT" />
 <data android:mimeType="image/*" />
 </intent-filter>
</activity>
```
上面的配置中我们并没有指定scheme。我们可以通过默认值content 和 file匹配。
```
public void match(){
 Intent intent=new Intent();
//content也可匹配
 intent.setDataAndType(Uri.parse("file://axe"),"image/png");
 startActivity(intent);
}
```

（7）**存在多个data的匹配**
一个Activity只要能匹配任何一组data,并且每个data都指定了完整的属性(有时候匹配不上, 这个规律还未找到)
```
<activity
 android:name=".CActivity"
 android:label="@string/app_name"
 android:theme="@style/AppTheme.NoActionBar">
 <intent-filter>
 <action android:name="xx" />

 <category android:name="android.intent.category.DEFAULT" />
 <!-- 只要Intent的Data属性的scheme是lee，即可启动该Activity -->
 <data
 android:mimeType="axe/abc"
 android:host="www.axe.com"
 android:path="/mypath"
 android:port="8888"
 android:scheme="axe" />

 <data
 android:mimeType="axe/ddd"
 android:host="www.axe.com"
 android:path="/mypath"
 android:port="8888"
 android:scheme="axee" />
 </intent-filter>
</activity>
```
以下两种方式都可以匹配.但是有时候会匹配不到.暂时不知道规律.建议只使用一个data.如果有多个规则需要匹配. 那就添加intent-filter
```
public void match() {
 Intent intent = new Intent();
 intent.setDataAndType(Uri.parse("axe://www.axe.com:8888/mypath"),"axe/abc");
 startActivity(intent);
}

public void match() {
 Intent intent = new Intent();
intent.setDataAndType(Uri.parse("axee://www.axe.com:8888/mypath"),"axe/ddd");
startActivity(intent);
}
```
### 4.其他的技巧
**a.无法匹配时的crash log**
```
//类似于这样。
Caused by: android.content.ActivityNotFoundException: No Activity found to handle Intent { cat=[android.intent.category.DEFAULT,aaa.bb.cc] }
at android.app.Instrumentation.checkStartActivityResult(Instrumentation.java:1781)
at android.app.Instrumentation.execStartActivity(Instrumentation.java:1501)
at android.app.Activity.startActivityForResult(Activity.java:3804)
at android.app.Activity.startActivityForResult(Activity.java:3765)
```
**b.一个Activity只要能匹配任何一组intent-filter，即可成功启动对应的Activity**
```
<activity
 android:name=".CActivity"
 android:label="@string/app_name"
 android:theme="@style/AppTheme.NoActionBar">
 <intent-filter>
 <action android:name="xx" />
 <category android:name="android.intent.category.DEFAULT" />
 <data
 android:mimeType="axe/abc"
 android:host="www.axe.com"
 android:path="/mypath"
 android:port="8888"
 android:scheme="axe" />
 </intent-filter>

 <intent-filter>
 <action android:name="xx" />
 <category android:name="android.intent.category.DEFAULT" />
 <data
 android:mimeType="axe/ddd"
 android:host="www.axe.com"
 android:path="/mypath"
 android:port="8888"
 android:scheme="axee" />
 </intent-filter>
</activity>
```
以下两种方式都可以匹配
```
public void match() {
 Intent intent = new Intent();
 intent.setDataAndType(Uri.parse("axe://www.axe.com:8888/mypath"),"axe/abc");
 startActivity(intent);
}

public void match() {
 Intent intent = new Intent();
 intent.setDataAndType(Uri.parse("axee://www.axe.com:8888/mypath"),"axe/ddd");
 startActivity(intent);
}
```
有参考来自：
http://www.cnblogs.com/wolipengbo/p/3427574.html
http://blog.csdn.net/eyishion/article/details/51113094
非常感谢你们的blog！给我很大的帮助。
