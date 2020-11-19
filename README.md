# 05游戏集成sdk开发包

## as在线导入方式
### 导入方式
将SDK存储库添加到您的构建文件中(项目根目录下build.gradle文件)
allprojects {
    repositories {
        ...
        maven { url "https://raw.githubusercontent.com/05youxi/05sdk/main" }
    }
}
在模块build.gradle文件加入dependencies
dependencies {
    implementation '05you.lib:aar:1.0.1'
}

## as离线导入
下载线上项目05you/lib/aar/版本号/aar-版本号.arr的文件，导入项目lib目录
在模块build.gradle文件加入dependencies
dependencies {
    implementation fileTree(includes: ['*.jar','*.aar'], dir: 'libs')
}

## eclipse导入参考以下链接
https://www.cnblogs.com/shortboy/p/4424944.html

## 混淆规则
如项目中已开启混淆，则添加混淆规则
-keep class com.bun.miitmdid.core.** {*;}

## 配置AndroidManifest.xml
### 在集成app模块的AndroidManifest.xml中加入一下代码
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="应用的包名">

    <uses-permission android:name="android.permission.CALL_PHONE"/>
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    <uses-permission android:name="android.permission.GET_TASKS"/>
    <uses-permission android:name="com.sdp.permission.WALLET_PAY" />
    <uses-permission android:name="android.permission.BROADCAST_PACKAGE_INSTALL" />
    <uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES" />
    <uses-permission android:name="com.asus.msa.SupplementaryDID.ACCESS" />
    <uses-permission android:name="android.permission.READ_PRIVILEGED_PHONE_STATE"/>
    <application
        android:allowBackup="true"
        andoird:name="自定义application"
        android:label="@string/app_name"
        android:theme="@style/AppTheme">
        <--配置渠道号,value为分配给合作游戏产品的APPID，作为唯一标示符-->
        <meta-data
            android:name="05YOUXI_APPID"
            android:value="1"/>
        <meta-data
            android:name="debug_type"
            android:value="1"/>

        <activity
            android:name="com.yyjia.sdk.PayActivity"
            android:configChanges="orientation|keyboardHidden|navigation"
            android:screenOrientation="behind"
            android:theme="@android:style/Theme.Light.NoTitleBar"/>
        <activity
            android:name="com.yyjia.sdk.HeePayActivity"
            android:configChanges="orientation|keyboardHidden|navigation"
            android:theme="@android:style/Theme.Light.NoTitleBar"/>
        <activity
            android:name="com.alipay.sdk.app.H5PayActivity"
            android:configChanges="orientation|keyboardHidden|navigation"
            android:exported="false"
            android:screenOrientation="behind"/>
        <activity
            android:name="com.alipay.sdk.auth.AuthActivity"
            android:configChanges="orientation|keyboardHidden|navigation"
            android:exported="false"
            android:screenOrientation="behind"/>
        <activity
            android:name="com.heepay.plugin.activity.WeChatNotityActivity"
            android:configChanges="orientation|keyboardHidden|screenSize"
            android:screenOrientation="behind"
            android:theme="@android:style/Theme.NoDisplay"/>
        <activity
            android:name="com.yyjia.sdk.LoginActivity"
            android:configChanges="orientation|keyboardHidden|navigation|screenSize"
            android:screenOrientation="behind"
            android:theme="@style/game_sdk_activity_transparent"/>

        <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="应用包名.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true" >
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/filepaths" />
        </provider>

        <service android:name="com.yyjia.sdk.util.FloatViewService"/>
        <service
            android:name="com.yyjia.sdk.util.CxAccessService"
            android:enabled="true"
            android:exported="true"
            android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">
            <intent-filter>
                <action android:name="android.accessibilityservice.AccessibilityService"/>
            </intent-filter>

            <meta-data
                android:name="android.accessibilityservice"
                android:resource="@xml/game_sdk_accessible_service_config"/>
        </service>
    </application>

</manifest>

在res/xml目录下加入filepaths.xml,game_sdk_accessible_service_config.xml两个文件（xml目录不存在则创建）

## sdk工作流程
### 游戏启动-初始化sdk-获取sdk实例,设置登录注册监听-用户注册登录-启动统计服务-回传用户信息
SDK工作流程说明：
当游戏启动时，初始化SDK
游戏内用户信息发生变化时，如用户登录成功、注册成功、切换账号登录后，并回传玩家信息给SDK。

## 初始化sdk
### 在自定义application的onCreate下调用初始化sdk的方法
```
public void onCreate() {
    super.onCreate();
    SDKUtils.init(this);
}
```

## sdk使用

### 在使用sdk的地方获取sdk实例
```
mCenter = GMcenter.getInstance(Context context);
mCenter.onCreate(Activity activity);
```
### 登录、登出、关闭登录窗口、SDK初始化 事件监听
```
Center.setLoginListener(LoginListener loginListener);
mCenter.setLoginListener(new LoginListener() {
    // 登录监听方法
    @Override
    public void loginSuccessed(String code) {
        if (code == Information.LOGIN_SUSECCEDS) {
            // SDK登录成功 游戏(服务端)请求 登录验证
            ToastUtil.showShortToast(MainActivity.this, "登录成功");
            mCenter.showFloatingView(MainActivity.this);
        } else {
            ToastUtil.showShortToast(MainActivity.this, "登录失败");
        }
    }

    // 登出监听方法
    public void logoutSuccessed(String code) {
        if (code == Information.LOGOUT_SUSECCED) {
            // 账号 退出 游戏需要重启到 登录界面
            ToastUtil.showShortToast(MainActivity.this, "退出游戏成功");
        } else {
            ToastUtil.showShortToast(MainActivity.this, "退出游戏失败);
        }
    }

    @Override
    public void logcancelSuccessed(String code) {
        if (code == Information.LOGCANCEL_SUSECCED) {
            ToastUtil.showShortToast(MainActivity.this, "取消登录);
        }
    }
});

```

```
mCenter.setInitListener(new InitListener() {

    @Override
    public void onSuccess(int code) {
        mCenter.checkLogin();
    }

    @Override
    public void onFailure(int code, String errMsg) {
        if (code == Information.CODE_GET_APP_ID_ERROR) {
            // 获取AppId失败
        } else if (code == Information.CODE_NET_TIME_OUT) {
            // 网络连接超时
        } else if (code == Information.CODE_SERVER_RETURN_DATA_ERROR) {
            // 服务端返回数据错误
        }
        ToastUtil.showShortToast(MainActivity.this, errMsg);
    }
});

// 游戏退出监听方法(当调用exitDialog()方法，必须设置此监听)
mCenter.setExitGameListener(new ExitGameListener() {
    @Override
    public void onSuccess() {
        ToastUtil.showShortToast(MainActivity.this, "Exit Game Success");
    }

    @Override
    public void onFailure() {
        ToastUtil.showShortToast(MainActivity.this, "Exit Game Failure");
    }

    @Override
    public void onCancel() {
        ToastUtil.showShortToast(MainActivity.this, "Exit Game Cancel");
    }
});

```

### 登录接口
```
mCenter.checkLogin();  
```

### 切换账号或者注销
```
mCenter.logout();
```

### 充值接口
参数说明：
Float money=0.01f;
String productname=”商品名称”;
String serverId =”服务器编号”;
String charId = ”角色编号”;
String cporderId = ”CP订单号”;
String callbackInfo = ”扩展信息 当充值成功会回传给游戏服务器”;

#### PayListener支付监听事件
```
mCenter.pay(Context context, float rmb, String productname,
                String serverId, String charId, String cporderid,
                String callbackInfo, PayListener payListener);

mCenter.pay(MainActivity.this, 0.01f, "test", "1", "1", System.currentTimeMillis() + "", "123456", new PayListener() {
    @Override
    public void paySuccessed(String code, String cporderid) {
    if (code == Information.PAY_SUSECCED) {
        ToastUtil.showShortToast(MainActivity.this, "PAY SUCCESS");
    } else {
        ToastUtil.showShortToast(MainActivity.this, "PAY FAIL");
    }
}

    @Override
    public void payGoBack() {
        Utils.E("payGoBack");
        ToastUtil.showShortToast(MainActivity.this, "PAY BACK");
    }

});
```

### 取得sessionid 与uid
```
mCenter.getSid();//sid(token)
mCenter.getUid();//用户uid
```
### 提交用户角色信息
参数说明：
String serverId=”服务器ID”;
String serverName=”服务器名”;
String roleId=”角色ID”;
String roleName=”角色名”;
String roleLevel=”角色等级”;
String roleCTime=”角色创建时间 时间戳”;
String power=”战斗力”; 
String sign=”签名”; 

sign签名规则:
sign=MD5(power=**&roleid=**&rolelevel=**&rolename=**&serverid=**+loginkey)
注意：
① 签名中的loginkey (即登录KEY) 由我方提供，为SDK参数之一;
② 若不存在某项参数，请务必传空字符，不可不传，且请勿随便传值. 
```
// 当角色信息变动的时候调用这个接口（例如角色创建、升级等情况）
mCenter.submitRoleInfo(String serverId, String serverName,
tring roleId, String roleName, String roleLevel, 			String roleCTime,String power,String sign);
```
### 退出游戏
```
// 如果游戏没有自己的退出对话框
mCenter.exitDialog(MainActivity.this);
// 只要游戏能确保正常退出的时候能走onDestroy()方法,就在onDestroy()的方法里的super.onDestrot()之前调用，不能的话再游戏确认退出的时候调用
mCenter.exitGame(MainActivity.this);
```
### 隐藏显示悬浮窗（登录成功的时候也需要调用）
```
mCenter.hideFloatingView(MainActivity.this);
mCenter.showFloatingView(MainActivity.this);

// 在onResume里调用
@Override
protected void onResume() {
    super.onResume();
    if (mCenter != null) {
        mCenter.onResume(this);
        mCenter.showFloatingView(MainActivity.this);
    }
}
// 在onPause里调用
@Override
protected void onPause() {
    super.onPause();
    if (mCenter != null) {
        mCenter.hideFloatingView(this);
    }
}

// 在onStop里调用
@Override
protected void onStop() {
    super.onStop();
    if (mCenter != null) {
        mCenter.onStop(MainActivity.this);
    }
}
```

### 在以下对应的声明周期中的实现以下方法
```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mCenter.onCreate(MainActivity.this);
}
@Override
protected void onStart() {
    super.onStart();
    mCenter.onStart(MainActivity.this);
}

@Override
protected void onResume() {
    super.onResume();
    if (mCenter != null) {
        mCenter.onResume(MainActivity.this);
    }
}

@Override
protected void onPause() {
    super.onPause();
    if (mCenter != null) {
        mCenter.onPause(MainActivity.this);
        mCenter.hideFloatingView(this);
    }
}

@Override
protected void onStop() {
    super.onStop();
    Utils.E("onStop");
    if (mCenter != null) {
        mCenter.onStop(MainActivity.this);
    }
}

@Override
protected void onRestart() {
    super.onRestart();
    mCenter.onRestart(MainActivity.this);
}

@Override
protected void onDestroy() {
    if (mCenter != null) {  // 必须在super.onDestroy()之前调用
        mCenter.onDestroy(this);
    }
    super.onDestroy();
}

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (mCenter != null) {
        mCenter.onActivityResult(this, requestCode, resultCode, data);
    }
}
```
### 可添加动态申请权限
如果游戏的targetSdkVersion的版本号大于等于23(即Android 6.0或者以上),要动态去申请sdk所使用的敏感权限(电话权限和存储权限),在MainActivity添加该方法(如图),如游戏本身的敏感权限则由游戏自己去动态申请。
```
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions,@NonNull int[] grantResults) {
    super.onRequestPermissionsResult(requestCode,permissions,grantResults);
   mCenter.onRequestPermissionsResult(requestCode,permissions,grantResults);
}
```
如果targetSdkVersion版本小于23,即不用动态申请权限,就没必要调用此方法。

### 根据需求自行获取是否实名和生日信息
```
mCenter.isAuthentication();// 获取实名认证返回值
mCenter.getBirthday();// 获取生日信息
```

建议：
若游戏的Activity没作横竖屏切换判断的，请在AndroidManifest文件对应的Activity添加上这行代码：
android:configChanges="orientation|keyboardHidden|navigation|screenSize"





