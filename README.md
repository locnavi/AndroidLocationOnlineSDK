# AndroidLocationOnlineSDK

AndroidLocationOnlineSDK 是一个室内定位SDK，通过扫描周边ibeacon，将ibeacon数据传输到后端计算获得室内位置。

## AndroidX工程的Library引用
通过jitpack将github上的aar引入到工程中。
在setting.gradle中添加
```bash
    maven { url 'https://jitpack.io' }
```

在app的build.gradle中添加
```bash
    // use jitpack from github
    implementation 'com.github.locnavi:AndroidLocationOnlineSDK:0.1.14'
    implementation 'com.squareup.okhttp3:okhttp:4.9.2'
    implementation 'org.altbeacon:android-beacon-library:2.19.4'
```

## support工程的Library引用
Android support的项目调用AndroidX提供的aar可能会有问题。我们可以将aar转成支持android support。通过jetifier-standalone工具转化。参考官方链接(https://developer.android.google.cn/studio/command-line/jetifier)
转换成功后即可用本地aar引入使用

## 加入权限
在AndroidMainfest.xxml中添加
```bash
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    //检测到未打开蓝牙时能自动打开蓝牙
    <uses-permission android:name="android.permission.BLUETOOTH"/>
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
```

## SDK的使用

### 初始化
在Application的onCreate方法中添加
```java
        //初始化SDK
        LocNaviClient client = LocNaviClient.getInstanceForApplication(this);
        client.setBaseUri("http://192.168.2.16:8086");
        //在App获取到用户信息之后调用
        client.setUserInfo("pda", "123456");
        //额外设定蓝牙定位上传的接口url
        // client.setUploadApi("http://192.168.2.16:3939/tagLocation");
        //需要跟我们确认采集方式，默认使用BEACON_MODE_IBEACON
        //client.setBeaconMode(LocNaviConstants.BEACON_MODE_BEACON);
```

定位权限及蓝牙功能检测
在开启定位之前先确认蓝牙及定位的权限已经开启
```java
        verifyBluetooth();
        verifyLocation();
        requestPermissions();
```


```java
    private static final int PERMISSION_REQUEST_FINE_LOCATION = 1;
    private static final int PERMISSION_REQUEST_BACKGROUND_LOCATION = 2;

    private void verifyBluetooth() {
        try {
            if (!LocNaviClient.getInstanceForApplication(this).checkAvailability()) {
                BluetoothAdapter bluetoothAdapter = BluetoothAdapter.getDefaultAdapter();
                boolean isEnabled = bluetoothAdapter.isEnabled();
                if (!isEnabled) {
                    //打开蓝牙
                    bluetoothAdapter.enable();
                }
                // new AlertDialog.Builder(this)
                //     .setTitle("蓝牙未开启")
                //     .setMessage("室内定位基于蓝牙扫描，请打开蓝牙使用")
                //     .setPositiveButton(android.R.string.ok, null)
                //     .show();
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
    
        //判断手机定位功能是否开启
    private void verifyLocation() {
        if (!this.isLocationEnabled()) {
            AlertDialog.Builder builder = new AlertDialog.Builder(this)
                    .setTitle("定位功能未开启")
                    .setMessage("室内定位需要定位功能开启后使用")
                    .setNegativeButton(android.R.string.cancel, null)
                    .setPositiveButton(android.R.string.ok, new DialogInterface.OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialog, int which) {
                            //openGPS(MainActivity.this);
                            //跳转GPS设置界面
                            Intent intent = new Intent(Settings.ACTION_LOCATION_SOURCE_SETTINGS);
                            MainActivity.this.startActivity(intent);
                        }
                    });
            builder.setCancelable(true);
            builder.show();
        }
    }
    
        //判断定位功能是否开启
    private boolean isLocationEnabled() {
        int locationMode = 0;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            try {
                locationMode = Settings.Secure.getInt(getContentResolver(), Settings.Secure.LOCATION_MODE);
            } catch (Settings.SettingNotFoundException e) {
                e.printStackTrace();
                return false;
            }
            return locationMode != Settings.Secure.LOCATION_MODE_OFF;
        } else {
            String locationProviders = Settings.Secure.getString(getContentResolver(), Settings.Secure.LOCATION_PROVIDERS_ALLOWED);
            return !TextUtils.isEmpty(locationProviders);
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
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
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
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
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
```java
  //默认同时开启LocNaviConstants.LOCATION_MODE_AUTO模式，目前是同时打开蓝牙和GPS，以后应该是在室内用蓝牙，室外用GPS。若只用蓝牙传LOCATION_MODE_ONLY_BEACON，只用GPS传LOCATION_MODE_ONLY_GPS
  client.start();
```

### 定位监听
```java
    LocNaviLocationListener listener = new LocNaviLocationListener() {
        @Override
        public void onReceiveLocation(LocNaviLocation location) {

        }
    };
    client.registerLocationListener(listener);
```

### 触发事件

事件的字段需要提前沟通、后台根据传入的字段做解析
#### 1、登录事件
```java
    //传入用户名、用户id即可
    client.setUserInfo("pda", "123456");
```
#### 2、登出事件
```java
    client.setUserInfo(null, null);
```
#### 3、开始配送事件
```java
        //尽量将有用的配送单的信息都放传入
        Map properties = new HashMap();
        properties.put("DELIVERY_CODE", "12344");
        properties.put("DEPT_STORE_ID", "568");
        LocNaviClient client = LocNaviClient.getInstanceForApplication(this);
        client.track(LocNaviConstants.EVENT_START_DELIVERY, properties);
```
#### 4、结束配送事件
```java
        //可只传入配送单id
        Map properties = new HashMap();
        properties.put("DELIVERY_CODE", "12344");
        properties.put("DEPT_STORE_ID", "568");
        LocNaviClient client = LocNaviClient.getInstanceForApplication(this);
        client.track(LocNaviConstants.EVENT_END_DELIVERY, properties);
```
#### 5、取消配送事件
```java
        //可只传入配送单id
        Map properties = new HashMap();
        properties.put("DELIVERY_CODE", "12344");
        properties.put("DEPT_STORE_ID", "568");
        LocNaviClient client = LocNaviClient.getInstanceForApplication(this);
        client.track(LocNaviConstants.EVENT_CANCEL_DELIVERY, properties);
```


## 使用Intent触发消息

### setBaseUri
        Intent intent = new Intent();
        intent.setAction("com.locanavi.locationonline.broadcast");
        intent.putExtra("method", "setBaseUri");
        HashMap params = new HashMap();
        params.put("uri", "http://192.168.1.1:8000");
        intent.putExtra("params", params);
        sendBroadcast(intent);

### 登录、登出
        Intent intent = new Intent();
        intent.setAction("com.locanavi.locationonline.broadcast");
        intent.putExtra("method", "setUserInfo");
        HashMap params = new HashMap();
        //登录
        params.put("name", "pda");
        params.put("id", "12345");
        //登出
        //params.put("name", null);
        //params.put("id", null);
        intent.putExtra("params", params);
        sendBroadcast(intent);

### 开始定位
        Intent intent = new Intent();
        intent.setAction("com.locanavi.locationonline.broadcast");
        intent.putExtra("method", "start");
        HashMap params = new HashMap();
        params.put("mode", LocNaviConstants.LOCATION_MODE_AUTO);
        intent.putExtra("params", params);
        sendBroadcast(intent);

### 停止定位
        Intent intent = new Intent();
        intent.setAction("com.locanavi.locationonline.broadcast");
        intent.putExtra("method", "stop");
        HashMap params = new HashMap();
        params.put("mode", LocNaviConstants.LOCATION_MODE_AUTO);
        intent.putExtra("params", params);
        sendBroadcast(intent);


### 配送事件
        Intent intent = new Intent();
        intent.setAction("com.locanavi.locationonline.broadcast");
        intent.putExtra("method", "track");

        HashMap params = new HashMap();
        //其他的配送事件只需要替换如下内容
        params.put("event", LocNaviConstants.EVENT_CANCEL_DELIVERY);
        Map properties = new HashMap();
        properties.put("DELIVERY_CODE", "12344");
        properties.put("DEPT_STORE_ID", "568");
        params.put("properties", properties);

        intent.putExtra("params", params);
        sendBroadcast(intent);
