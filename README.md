# AndroidLocationOnlineSDK

AndroidLocationOnlineSDK 是一个室内定位SDK，通过扫描周边ibeacon，将ibeacon数据传输到后端计算获得室内位置。

## Library引用
通过jitpack将github上的aar引入到工程中。
在setting.gradle中添加
```bash
    maven { url 'https://jitpack.io' }
```

在app的build.gradle中添加
```bash
    // use jitpack from github
    implementation 'com.github.locnavi:AndroidLocationOnlineSDK:0.0.3'
    implementation("com.squareup.okhttp3:okhttp:4.9.2")
    implementation 'org.altbeacon:android-beacon-library:2+'
```

## 加入权限
在AndroidMainfest.xxml中添加
```bash
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

## SDK的使用

### 初始化
在Application的onCreate方法中添加
```bash
        //初始化SDK
        LocNaviClient client = LocNaviClient.getInstanceForApplication(this);
        client.setBaseUri("http://192.168.2.16:8086");
        //在App获取到用户信息之后调用
        client.setUserInfo("pda", "123456");
```

定位权限及蓝牙功能检测（稍后添加）

开启定位
```bash
  client.start();
```

触发事件
