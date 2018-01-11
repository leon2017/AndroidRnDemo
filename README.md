# AndroidRnDemo
Android项目集成ReactNative


# setp 1: 
```
npm install
```

# Android已有项目集成ReactNative
我们知道从0开始搭建ReactNative项目的话,参照官方的[Getting Started](https://facebook.github.io/react-native/docs/getting-started.html)来的话基本是没有问题的。但是如果我们想把现有的项目集成RN的话,虽然有官方的[Integration with Existing Apps](https://facebook.github.io/react-native/docs/integration-with-existing-apps.html?from=timeline)集成步骤，但是有个别的地方说的还是不清不楚，可能对于初学者埋下了坑点。本人也是第一次集成，现在讲集成的点点滴滴写出来，本文基本思路还是引用官方的集成步骤来的。

## 开发环境搭建
- 新建Android Studio工程为AndroidRnDemo,项目SDK版本等信息
    
```
    //最低版本
    minSdkVersion 19
    //目标版本
    targetSdkVersion 26
    
    // support v7
    implementation 'com.android.support:appcompat-v7:26.1.0'
    
    //gradle
    classpath 'com.android.tools.build:gradle:3.0.1'
```
- 配置ReactNative的js环境
> 我们进入到AndroidRnDemo的项目根目录，接下来我们开始在终端输入：

```
npm init
```
我们可以看到init让你生成一个package.json文件，当然这里里面的配置信息就是让你刚刚init填写的配置参数：

```
//init命令提示输入的信息
name: (AndroidRnDemo) androidrndemo
version: (1.0.0) 
description: 
entry point: (index.js) 
test command: 
git repository: 
keywords: 
author: loen
license: (ISC) 

//package.json
{
  "name": "androidrndemo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "loen",
  "license": "ISC"
}

```
## 添加ReactNative到项目 
接下来我们将React、ReactNative添加到项目中：

```
npm install --save react react-native
```
安装完成之后，我们再去配置.flowconfig，这个是对js代码做约束的，建议配置（也可以不配置）。

```
curl -o .flowconfig  https://raw.githubusercontent.com/facebook/react-native/master/.flowconfig
```
当然你也可以在项目根目录新建.flowconfig，然后把这个[链接](https://raw.githubusercontent.com/facebook/react-native/master/.flowconfig)的文件内容复制到.flowconfig中。

添加start到package.json文件   
> 这样我们就可以时时的调用本地调试服务进行热加载了

```
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node node_modules/react-native/local-cli/cli.js start"
},
```


> 好了，现在看下我们的项目目录吧  

![基本目录](http://oxp7ffm1s.bkt.clouddn.com/1515642113043.jpg)

## 添加ReactNative到Android项目
#### 添加ReactNative依赖到Android项目
> 配置你的maven  

添加ReactNative的依赖到你的app目录下的build.gradle中：

```
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support:appcompat-v7:26.1.0'
    ...
     // From node_modules.
    api "com.facebook.react:react-native:+"
}
```
配置项目根目录下的build.gradle：

这里有个小插曲：
```
//官方配置
allprojects {
    repositories {
        ...
        maven {
            // All of React Native (JS, Android binaries) is installed from npm
            url "$rootDir/../node_modules/react-native/android"
        }
    }
    //...
}
// 由于官方的android项目是放在android/目录下，所以他的路径是这样的
"$rootDir/../node_modules/react-native/android"
//而我们为了方便AS编译，直接放在项目根目录配置RN的，所以这里的路径要改一下
 url "$rootDir/../node_modules/react-native/android"
 
 //我们的目录正确配置
allprojects {
    repositories {
        maven {
            url "$rootDir/node_modules/react-native/android"
        }
        jcenter()
    }
}
```
> 为了防止个别机型.so库和findbugsbug问题，建议在app/build.gradle中增加下面的代码：

```
android {

    defaultConfig {
       ndk {
            //选择要添加的对应cpu类型的.so库。
            abiFilters 'armeabi', "armeabi-v7a","armeabi-v7a","x86"
        }
    }
    
    //...
    configurations.all {
        resolutionStrategy.force 'com.google.code.findbugs:jsr305:3.0.0'
    }
}
```


#### AndroidManifest.xml
添加权限：

```
<uses-permission android:name="android.permission.INTERNET"/>
/**设置调试 的权限**/
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
<uses-permission android:name="android.permission.SYSTEM_OVERLAY_WINDOW" />
```
添加debug模式Activity(正式打包注释掉就好了)

```
<activity android:name="com.facebook.react.devsupport.DevSettingsActivity"/>
```
> debug模式下需要悬浮窗的权限，这个需要手动设置，每部手机姿势不一样，具体请百度调整姿势。如果没有设置的话，个别手机在debug模式下reload会出现异常   

```
android.view.WindowManager$BadTokenException: Unable to add window android.view.ViewRootImpl$W@fc0db15 -- permission denied for this window type
```
## 添加ReactNative界面
#### 添加index.js
首先我们在项目根目录添加index.js,这个文件作为RN的入口文件。

```
import React from 'react';
import {AppRegistry, StyleSheet, Text, View} from 'react-native';

class HelloWorld extends React.Component {
  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.hello}>Hello, World</Text>
      </View>
    );
  }
}
var styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
  },
  hello: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
});

AppRegistry.registerComponent('AndroidRnDemoApp', () => HelloWorld);
```
注意 AppRegistry.registerComponent('AndroidRnDemoApp', () => HelloWorld);里面的AndroidRnDemoApp,这个作为接下来我们要绑定的activity的入口通道名。

#### 配置Android动态权限--->（非必须，针对targetSdk>=23版本（android 6.0））
我们的项目目标sdk是26，所以我们需要去代码请求窗口浮层的权限，为了方便起见呢，我们将在MainActivity里面配置这个权限：

```
public class MainActivity extends AppCompatActivity {

    static final int OVERLAY_PERMISSION_REQ_CODE = 1000;
    private AppCompatButton mBtnRn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initEvent();
    }

    private void initEvent() {
        initView();
        checkAppPermission();
    }

    private void initView() {
        mBtnRn = (AppCompatButton) findViewById(R.id.btn_rn);
        mBtnRn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //跳转RN页面
            }
        });
    }

    private void checkAppPermission() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            if (!Settings.canDrawOverlays(this)) {
                Intent intent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION,
                        Uri.parse("package:" + getPackageName()));
                startActivityForResult(intent, OVERLAY_PERMISSION_REQ_CODE);
            }
        }
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (requestCode == OVERLAY_PERMISSION_REQ_CODE) {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                if (!Settings.canDrawOverlays(this)) {
                    //SYSTEM_ALERT_WINDOW被拒绝
                }
            }
        }
    }
}
```

#### 添加RN Activity界面
我们新建一个Activity作为RN界面展示的容器。这里姑且叫BaseRnActivity吧。

```
public class BaseRnActivity extends Activity implements DefaultHardwareBackBtnHandler {

    private ReactRootView mReactRootView;
    private ReactInstanceManager mReactInstanceManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mReactRootView = new ReactRootView(this);
        mReactInstanceManager = ReactInstanceManager.builder()
                .setApplication(getApplication())
                .setBundleAssetName("index.android.bundle")
                .setJSMainModulePath("index")
                .addPackage(new MainReactPackage())
                .setUseDeveloperSupport(BuildConfig.DEBUG)
                .setInitialLifecycleState(LifecycleState.RESUMED)
                .build();

        //这里的AndroidRnDemoApp必须对应“index.js”中的“AppRegistry.registerComponent()”的第一个参数
        mReactRootView.startReactApplication(mReactInstanceManager, "AndroidRnDemoApp", null);
        //加载ReactRootView到布局中
        setContentView(mReactRootView);
    }

    @Override
    public void invokeDefaultOnBackPressed() {
        super.onBackPressed();
    }


    /**
     * ReactInstanceManager生命周期同activity
     */

    @Override
    protected void onPause() {
        super.onPause();

        if (mReactInstanceManager != null) {
            mReactInstanceManager.onHostPause(this);
        }
    }

    @Override
    protected void onResume() {
        super.onResume();

        if (mReactInstanceManager != null) {
            mReactInstanceManager.onHostResume(this, this);
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        if (mReactInstanceManager != null) {
            mReactInstanceManager.onHostDestroy(this);
        }
    }

    @Override
    public void onBackPressed() {
        if (mReactInstanceManager != null) {
            mReactInstanceManager.onBackPressed();
        } else {
            super.onBackPressed();
        }
    }

    @Override
    public boolean onKeyUp(int keyCode, KeyEvent event) {
        if (keyCode == KeyEvent.KEYCODE_MENU && mReactInstanceManager != null) {
            mReactInstanceManager.showDevOptionsDialog();
            return true;
        }
        return super.onKeyUp(keyCode, event);
    }
}
```

最后不要忘了在AndroidManifest中添加：


```
<activity
  android:name=".BaseRnActivity"
  android:theme="@style/Theme.AppCompat.Light.NoActionBar">
</activity>
```

## RN打离线包到Android
由于我们在BaseRnActivity的ReactInstanceManager中setBundleAssetName("index.android.bundle")了android离线包，所以在运行之前我们app之前先打个离线包JSBundle到android项目中。

> 首先在项目app/src/main下面必须要创建一个assets目录，然后我们就开始打离线包啦：


```
//注意路径
react-native bundle --platform android --dev false --entry-file index.js --bundle-output app/src/main/assets/index.android.bundle --assets-dest app/src/main/res/

```


```
# 控制台这样输出的话 ，表示打包成功
Loading dependency graph, done.
bundle: start
bundle: finish
bundle: Writing bundle output to: app/src/main/assets/index.android.bundle
bundle: Done writing bundle output
```


##  运行ReactNative

由于我们是已有项目集成RN,所以我们就不可以使用命令react-native run-android了。没关系我们手动编译呗。


```
//先开启本地react native服务
adb reverse tcp:8081 tcp:8081
npm start
```
这里需要注意的是连接Debug server的话，需要我们手机连接pc的代理，[具体请参考](https://facebook.github.io/react-native/docs/running-on-device.html#docsNav)

ok 运行成功

![运行成功](http://oxp7ffm1s.bkt.clouddn.com/www.jpg)




## 遇到的坑

1. 未添加NDK的so库

```
java.lang.UnsatisfiedLinkError: could find DSO to load: libreactnativejni.so
java.lang.UnsatisfiedLinkError: dlopen failed: "xxx/libgnustl_shared.so" is 32-bit instead of 64-bit
```
这个错误前面在我集成的项目中已经添加了 ，如果你没有添加，那就报错了。

```
android {

    defaultConfig {
       ndk {
            //选择要添加的对应cpu类型的.so库。
            abiFilters 'armeabi', "armeabi-v7a","armeabi-v7a","x86"
        }
    }
    
    //...
    configurations.all {
        resolutionStrategy.force 'com.google.code.findbugs:jsr305:3.0.0'
    }
}
```

2.依赖错误：

```
Error:Execution failed for task ':app:prepareDebugAndroidTestDependencies'.
```
在app/build.gradle中添加

```
configurations.all {
    resolutionStrategy.force 'com.google.code.findbugs:jsr305:3.0.0'
}
```
3.待补充...

## Demo地址

[github](https://github.com/leon2017/AndroidRnDemo)
