---
title: react-native 0.42 整合极光推送
date: 2019-08-29 14:44:20
tags: react-native
---
# [极光推送官网](https://www.jiguang.cn)注册应用

<!-- more -->

# 配置jpush-react-native

github地址：[https://github.com/jpush/jpush-react-native](https://link.jianshu.com/?t=https://github.com/jpush/jpush-react-native)
同样的，打开终端在项目根目录下输入：

```
npm install jcore-react-native@1.0.0 --save
npm install jpush-react-native@1.5.0 --save
```

下载完成后，按1、2、3的顺序修改如下文件：

![操作顺序图](https://raw.githubusercontent.com/luckybearbear/img/master/hexo/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_15670615598134.png)

## 在1中修改如下：

```js
include ':jpush-react-native'
project(':jpush-react-native').projectDir = new File(rootProject.projectDir, '../node_modules/jpush-react-native/android')
include ':jcore-react-native'
project(':jcore-react-native').projectDir = new File(rootProject.projectDir, '../node_modules/jcore-react-native/android')
```

## 在2中修改如下：

```
android {
    defaultConfig {
        applicationId "yourApplicationId" //就是在极光注册的包名
        ...
        manifestPlaceholders = [
                JPUSH_APPKEY: "yourAppKey", //在此替换你的APPKey
                APP_CHANNEL: "developer-default"    //应用渠道号
        ]
    }
}
...
dependencies {
    compile fileTree(dir: "libs", include: ["*.jar"])
    compile project(':jpush-react-native')
    compile project(':jcore-react-native')
    compile "com.facebook.react:react-native:+"  // From node_modules
}
```

## 在3中修改如下：

```
 <!--添加通知权限，${ApplicationID}替换成你的applicationID!-->
    <premission 
        android:name="${ApplicationID}.permission.JPUSH_MESSAGE"
        android:protectionLevel="signature"/>

    <application
        ...
        >
        ...
        <!-- Required . Enable it you can get statistics data with channel -->
        <meta-data android:name="JPUSH_CHANNEL" android:value="${APP_CHANNEL}"/>
        <meta-data android:name="JPUSH_APPKEY" android:value="${JPUSH_APPKEY}"/>
    </application>
```

## 打开MainApplication.java，修改如下：

```
@Override
        protected List<ReactPackage> getPackages() {
            return Arrays.<ReactPackage>asList(
                    new MainReactPackage(),
                    new JPushPackage(false,false)
            );
        }
```

