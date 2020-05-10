# Android APP开发入门


首先要明白Android系统的四大组件：活动Activity，服务Service，广播接收器Broadcast Receiver，内容提供器Context Provider。活动是应用程序的门面，就是你在手机屏幕上看到的东西；服务在后台运行；广播接收器是在手机内接收或发送广播消息；内容提供器在app之间共享数据。  

## 工具准备:android studio

开发Android程序需要JDK和Android SDK,Android Studio是官方的IDE工具,集成好了所有需要用到的工具,去Android官网下载即可。
## 开始创建（此时我的版本为3.6）
### HelloWorld
#### 开始
在欢迎界面点击'Start a new Android Studio project',选择设备是手机，其他设备APP在搭建时是接口不同，读者可自行学习，进入创建活动界面，里面有很多的活动模板，我们先选择‘Empty Activity’，点next，第一行是应用名称，像“QQ”、“微信”，当然这里目前只能是英文字符。第二行表示项目的包名，Android系统就是通过包名来区分不同程序，因此包名一定要具有唯一性，个人开发者填com.example即可,反之填公司域名。第三行是项目代码存放位置。第四行是所用语言，我选的是Java，值得一提的是Kotlin虽然晚于Java但是目前谷歌公司想将其推广为AndroidAPP的主要语言，读者可自行学习。第五行选择的是该应用的最低兼容版本，统计结果显示Android4.0以上的系统占据了超过98/%的Android市场，因此选择API 15即可，之前因某些配置问题，我个人选择的是最低版本是API 22。最后是是否使用原版的Android支持库，这里暂时不作考虑。此时点击Finish即可。  
因为搭建过程中随时可能要下载其他插件，建议联网。初次搭建过程要下载Gradle插件，当然也可以导入本地已下载好的，通过导航栏里File——Settings——BuildExecutionDeployment——Gradle更改即可，该插件用于编译过程，所以不能跳过，但因为是外网下载，所以速度较慢，可以考虑使用国内阿里云镜像，配置更改方法下面有简单介绍。点击Finish按钮之后需耐心等待一会儿，即需build过程完成，点击左下角的build，当显示successful即可，这里顺提一下，3.6版本较3.5版本的改进就是，第一次之后的搭建过程build的速率明显提高。此时软件就为我们自动生成了许多东西。接下来我们就需要一个Android模拟器来测试完成的程序，当然你也直接可以用你的安卓手机通过usb线与电脑连接，在右上角有一个AVD Manager，点击，再点击‘Create Virtual Device’，类型选择是手机，我选择的是Nexus 5，想设计特有的点击‘New Hardware Profile’，后面那个可以导入别人设计好的，或者自己设备的条件。点击next，选择该虚拟设备搭载的安卓系统，我选的是Pie，点击Download即可，下载完成后点next，通常手机开机都是竖屏的，默认的是Portrait，之后点Finish。  
回到原界面，直接点击横着的绿三角就开始运行程序了，选择搭建好的安卓模拟器，就可以看到屏幕上出现的手机模拟屏幕自动打开了你的app，然后上面显示的是“Hallo World！”。这是Android Studio自动帮你生成的，当然如果开始时你选的是‘Add no Activity’就不会这样了。现在回到Studio中，在左面文件列表的左上角，我们将Android模式切换为Project模式，这是真实的目录格式，我只讲基础的一部分，别的感兴趣可以自行百度。  
#### 简单介绍
一、app：项目与资源等内容位置。  libs：存放第三方jar包。  java：放置java代码。  res：drawable目录下放图片，layout目录下放布局文件，values目录下放字符串，样式，颜色等配置，mipmap目录下放应用图标。  AndroidManifest.xml:整个项目的配置文件,程序中定义的组件都需要在这里注册,另外在这里添加应用程序的权限声明  build.gradle:app模块的gradle构建脚本。  proguard-rules.pro:生成安装包文件时将代码混淆防止他人破解。  
二、gradle：包含了gradle wrapper的配置文件。  
三、.gitignore:将指定的目录或文件排除在版本控制之外。  build.gradle:项目全局的gradle构建脚本。  gradle.properties:全局的gradle构建脚本。  gradlew和gradlew.bat:用于在命令行界面执行gradle命令,前者用于Linux或Mac,后者用于Windows。  项目名.iml:用于标识这是一个IntelliJ IDEA项目(Studio基于其开发)。  local.properties:指定本机中Android SDK路径。  settings.gradle:指定项目中所有引入模块,通常情况下会自动引入。  
#### 分析具体文件
1.AndroidManifest.xml：  
```XML
<application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher" //指定应用图标
        android:label="@string/app_name"   //指定应用名称
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                 //表示HelloWorld活动是该项目的主活动
                <category android:name="android.intent.category.LAUNCHER" />
                 //表示在手机上点击应用图标后首先启动的活动
            </intent-filter>
        </activity>
    </application>
```  
2.目录src/main/java/com.example.hello MainActivity:  
Android程序设计讲究逻辑与视图分离,通常在布局文件中编写界面,然后在活动中引入进来.  
```java
public class MainActivity extends AppCompatActivity { 
//所有活动必须继承它或它的子类才能拥有活动的特性,这里AppCompatActivity是Activity的子类

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main); //引入布局文件
    }
}
```  
3.目录main/res/layout activity_main.xml:  
在右上角可以在code,split与design三中种编辑模式里切换,简单的UI界面可在code中编码完成,复杂的推荐练习design模式(方便快捷),split就是左边是Text，右边是预览图。现在切换到code模式。  
```XML
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView //布局中显示文字
        android:layout_width="wrap_content" //文字框的宽度是合适宽度,
        android:layout_height="wrap_content"
        android:text="Hello World!"//内容是XXX
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```  
4.目录res/valuse strings.xml:  
```XML
<resources>
    <string name="app_name">HelloWorl</string> //定义了一个应用程序名的字符串
</resources>
```  
引用它有两种方式:一是在代码中用R.sting.app_name;二是XML中用@sring/app_name。当然string还可以被替换成drawable、mipmap、layout。  
5.最外层build.gradle:(Gradle使用了基于Groovy的领域特定语言DSL来声明项目配置)  
```java
buildscript {
    repositories {
        google()
        jcenter() //代码托管仓库,声明后可轻松引用任何其上的开源项目
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.6.3' //声明所用的Gradle插件
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}
allprojects {
    repositories {
        google()
        jcenter()
    }
}
task clean(type: Delete) {
    delete rootProject.buildDir
}
```  
想使用阿里云镜像的话,将google()与jcenter()分别换为maven{url 'https://maven.aliyun.com/repository/google/' }与maven{url 'https://maven.aliyun.com/repository/jcenter/' }即可,若build失败,请检查网址是否发生改变。  
6.app目录下的build.gradle:  
```java
apply plugin: 'com.android.application' //表明这是应用程序模块,还有一个就是库模块com.android.library,它只能只能依附前者运行
android {
    compileSdkVersion 29 //指定项目编译版本
    buildToolsVersion "29.0.3" //指定项目构建工具的版本
    defaultConfig {
        applicationId "com.example.helloworld" //指定项目的包名
        minSdkVersion 23 //最低兼容版本
        targetSdkVersion 29 //测试目标版本
        versionCode 1 //指定项目版本号
        versionName "1.0" //指定项目版本名
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release { //指定生成正式版安装文件的配置
            minifyEnabled false //是否混淆代码
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro' 
            //指定混淆规则的文件,前者是默认的,后者可自己编写特定的混淆规则
        }
    }
}
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar']) //本地依赖声明
    implementation 'androidx.appcompat:appcompat:1.1.0' //远程依赖声明
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    //还有一种声明是库依赖声明,格式是implementation project(':库的名字')
    testImplementation 'junit:junit:4.12' //声明测试用例库
    androidTestImplementation 'androidx.test.ext:junit:1.1.1'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.2.0'
}
```  
### 日志工具Log
类是android.util.Log,有Log.v()、Log.d()、Log.i()、Log.w()、Log.e(),对应级别越来越高,v是琐碎信息，对应verbose级别；d是调试信息，对应debug级；i是较重要信息如有关用户的行为数据，对应info级；w为警告信息，对应warn级；e为错误信息，对应error级。Log.d()方法中传入两个参数:前者是tag,一般为当前类名,用于对打印信息进行过滤;后者是msg,为想打印的内容。打印信息在底部工具栏中的Logcat中即可看到。可在第一行第三个列表中选择过滤信息的级别。  
### 探究活动
#### 创建活动
右键单击目录src/main/java下的com.example.helloworld,点击new,找到Activity,再找到Empty Activity,出现活动的创建界面。第一行是活动的名字，勾选Generate Layout File会自动为其生成一个布局文件，并让你设置布局文件的名字，勾选Launch Activity会将其设置为当前项目的主活动，建议这两个先不勾选，语言任选java，点击finish。创建活动的流程就是这样,新活动先不用管。
#### 创建布局
对着layout目录右键——New——Layout resource file，新窗口中第一行为布局文件名，第二行为根元素，我们不妨设为LinearLayout。此刻我们不妨做个登录界面，代码如下：  
```XML
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" //布局为竖向,自上向下,horizontal为横向
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="60dp">
        <TextView
            android:layout_width="90dp"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical"
            android:textSize="18sp" //文字大小
            android:text="账户:" /> 
        <EditText  //可编辑文本框,用于用户输入东西
            android:id="@+id/account"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1" //权重占比为1,相当于于下面的密码框五五开,因为它的权重也为1
            android:layout_gravity="center_vertical" /> //居中放置
    </LinearLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="60dp"
        android:orientation="horizontal">
        <TextView
            android:layout_width="90dp"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical"
            android:textSize="18sp"
            android:text="密码 :" />
        <EditText
            android:id="@+id/password"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:layout_gravity="center_vertical"
            android:inputType="textPassword" />
    </LinearLayout>

    <Button //添加按钮
        android:id="@+id/login" //元素唯一标识符
        android:layout_width="match_parent" //宽度与父元素一样
        android:layout_height="wrap_content" //高度刚好包含里面的内容
        android:text="登录" /> //按钮上显示的文字

</LinearLayout>
```  
记得要在老活动中onCreat(){}里用setContentView()方法来加载这个布局啊!
####使用Toast
在setContentView()方法后加入下面代码：  
```java
Button button = (Button) findViewById(R.id.login); //找到布局文件中定义元素,但返回的是View对象,再将其转成Button对象
       button.setOnClickListener(new View.OnClickListener(){ //为按钮注册一个监听器,点击按钮时执行onclick方法
           @Override
           public  void onClick(View v) {
               Toast.makeText(context,"Text",Toast.LENGTH_LONG).show();
             //通过静态方法makeText创建出一个Toast对象,再用show()显示出来
             //第一个是Context参数,是Toast要求的上下文,直接传入活动本身即可
             //第二个参数就是要显示的文本内容
             //第三个是显示时长,有LONG和SHORT
           }
       });
       //你也可以将Toast...换成finish()来销毁当前活动
```  
那么"Text"就会像QQ消息提醒那样从手机屏幕的低端弹出,用作提醒。

## 参考书籍：  
1.《第一行代码Android》（第二版）郭霖著。  
2.《第一行代码Android》（第三版）郭霖著。  
3.《Android移动应用开发基础篇》Shane Conder著。  
