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

定位权限及蓝牙功能检测
在开启定位之前先确认蓝牙及定位的权限已经开启
```bash
        verifyBluetooth();
        requestPermissions();
```


```bash
    private static final int PERMISSION_REQUEST_FINE_LOCATION = 1;
    private static final int PERMISSION_REQUEST_BACKGROUND_LOCATION = 2;

    private void verifyBluetooth() {
        try {
            if (!LocNaviClient.getInstanceForApplication(this).checkAvailability()) {
                new AlertDialog.Builder(this)
                    .setTitle("蓝牙未开启")
                    .setMessage("室内定位基于蓝牙扫描，请打开蓝牙使用")
                    .setPositiveButton(android.R.string.ok, null)
                    .show();
            }
        }
        catch (RuntimeException e) {
            new AlertDialog.Builder(this)
                    .setTitle("蓝牙功能不可用")
                    .setMessage("该手机不支持蓝牙，无法使用室内定位功能！")
                    .setPositiveButton(android.R.string.ok, null)
                    .show();
        }
    }

    private void requestPermissions() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            if (this.checkSelfPermission(Manifest.permission.ACCESS_FINE_LOCATION)
                    == PackageManager.PERMISSION_GRANTED) {
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
                    if (this.checkSelfPermission(Manifest.permission.ACCESS_BACKGROUND_LOCATION)
                            != PackageManager.PERMISSION_GRANTED) {
                        //请求背景定位
                        requestPermissions(new String[]{Manifest.permission.ACCESS_BACKGROUND_LOCATION},
                                PERMISSION_REQUEST_BACKGROUND_LOCATION);
                    }
                }
            } else {
                //未授权或者拒绝的都直接申请，其中拒绝且点击不再提醒的会直接调用回调
                requestPermissions(new String[]{Manifest.permission.ACCESS_FINE_LOCATION,
                                Manifest.permission.ACCESS_BACKGROUND_LOCATION},
                        PERMISSION_REQUEST_FINE_LOCATION);
            }
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode,
                                           String permissions[], int[] grantResults) {
        switch (requestCode) {
            case PERMISSION_REQUEST_FINE_LOCATION: {
                if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    Log.d(TAG, "定位授权成功");
                } else {
                    new AlertDialog.Builder(this)
                            .setTitle("提示")
                            .setMessage("请在设置->应用->开启定位权限，才能室内定位")
                            .setPositiveButton(android.R.string.ok, null)
                            .show();
                }
                return;
            }
            case PERMISSION_REQUEST_BACKGROUND_LOCATION: {
                if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    Log.d(TAG, "背景定位授权成功");
                } else {
                    new AlertDialog.Builder(this)
                            .setTitle("提示")
                            .setMessage("请在设置->应用->开启背景定位权限，才能做背景室内定位")
                            .setPositiveButton(android.R.string.ok, null)
                            .show();
                }
                return;
            }
        }
    }
```

开启定位
```bash
  client.start();
```

触发事件
