
### 一、概述
![](https://main.qcloudimg.com/raw/c282b9e34d4cb018ded862281f438f2c.png)
1. 首先按照如下指引进行 iOS 及安卓手机的兼容性适配。
2. 兼容性验证：从如下手机列表中选择至少 10 款手机进行兼容性验证，需覆盖安卓系统版本 4.0~8.0。

### 二、兼容性配置
**iOS 接入**
iPhone 的兼容性适配，需在配置里加上摄像头和麦克风的使用权限。App 的 info.plist 中加入：
`.NSMicrophoneUsageDescription `  
`.NSCameraUsageDescription `

**Android 接入**
由于 android 机器碎片化严重，用系统 WebView 调起系统摄像头完成视频录制有很多兼容性，如部分机器出现调不起摄像头、调起摄像头无法录制视频等。因此整理了接入指引。

请合作方务必按照如下步骤顺序实现兼容性的处理：
1. 引入工具类
将 WBH5FaceVerifySDK.java 文件拷贝到项目中。
2. 申请权限
a. 在 Manifest.xml 文件中增加申请以下权限
```
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.RECORD_AUDIO" />
```
b. 动态申请权限
 - 如果第三方编译的 targetSdkVersion >= 23，则需要动态申请权限。
 - 如果第三方编译的 targetSdkVersion < 23，则不需要动态申请权限。
3. WebSettings 的设置
调用 WebView.loadUrl(String url) 前一行添加如下代码设置 WebSettings。
```
/**
  * 对 WebSettings 进行设置：添加 ua 字段和适配 h5 页面布局等
  * @param mWebView  第三方的 WebView 对象
  * @param context  第三方上下文
  */
WBH5FaceVerifySDK.getInstance().setWebViewSettings(mWebView,getApplicationContext()); 
```
4. WebChromeClient 的重写
调用 WebView.loadUrl(String url) 前，WebView 必须调用 setWebChormeClient(WebChromeClient webChormeClient)，并重写 WebChromeClient 的如下三个函数：
```
	/**
		* android端接收H5端发来的请求
	For Android >= 3.0
		*/
			public void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType) {
					if(WBH5FaceVerifySDK.getInstance().recordVideoForApiBelow21(uploadMsg, acceptType,activity))
							return;
		// TODO: 第三方有调用系统相机处理其他业务的话，将相关逻辑代码放在下面	

			}
	/**
		* android端接收H5端发来的请求
	For Android >= 4.1
		*/
			public void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType, String capture) {
					if(WBH5FaceVerifySDK.getInstance().recordVideoForApiBelow21(uploadMsg, acceptType,activity))
							return;
		// TODO: 第三方有调用系统相机处理其他业务的话，将相关逻辑代码放在下面

	}
	/**
		* android端接收H5端发来的请求
	For Lollipop 5.0+ Devices
		*/
			@Override
			public boolean onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback, FileChooserParams fileChooserParams) {
					if(WBH5FaceVerifySDK.getInstance().recordVideoForApi21(webView, filePathCallback,activity, fileChooserParams)){
							return true;
					}
			 // TODO: 第三方有调用系统相机处理其他业务的话，将相关逻辑代码放在下面

					return true;
			} 
```
> **注意：**
> - 如果第三方已重写以上函数，只要将如上述所示的函数体内容添加至第三方的对应函数体首行即可
> - 如果第三方没有重写以上函数，则直接按上述所示重写。

5. Activity 的重写
WebView 所属的 Activity 必须重写如下函数：
```
/**
  *返回到WebView所属的Activity的回调
  */
@Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (WBH5FaceVerifySDK.getInstance().receiveH5FaceVerifyResult(requestCode,resultCode,data))
            return;
// TODO: 第三方有其他请求的返回结果要处理的话，将相关逻辑代码放在下面
		
    }
```
>**注意：**
> - 如果第三方 WebView 所属的 Activity 已重写以上函数，则将如上图所示的函数体内容添加至第三方的对应函数体首行即可。
> - 如果第三方 WebView 所属的 Activity 没有重写以上函数，则直接按上图所示重写。
 
### 三、兼容性验证
为了验证合作方 Android 端已经兼容了摄像头的处理，建议合作方做如下兼容性测试。
-  从下面的机型列表中选择覆盖 Android4.0-8.0 版本号的机型进行测试。
-  每个系统版本号选取至少 2 款不同手机。
 
|手机品牌	|手机型号	|安卓系统版本|
|-|-|-|
|华为	|Mate9pro|	8.0.0|
|华为	|Mate10	 |8.0.0|
|华为|	华为荣耀 8	|7.0.0|
|华为	|华为 P9 Plus	|7.0.0|
|华为|	华为 Nexus 6P|	6.0.1|
|华为	|华为 Mate 7|	6.0.0|
|华为|	华为荣耀 7i	|5.1.1|
|华为	|P8 标配双 4G 版	|5.0.1|
|华为	 |Mate 7 青春版	|4.4.4|
|华为|	华为 G660-L075|	4.3.0|
|oppo|	OPPO   R11 Plus	|7.1.1
|oppo|	OPPO R11	|7.1.1|
|oppo	|OPPO   R9s Plus|	6.0.1|
|oppo|	OPPO   A57	|6.0.1|
|oppo	|OPPO A53|	5.1.1|
|oppo	|OPPO R7 Plus|	5.0.0|
|oppo|	OPPO   R7	|4.4.4|
|oppo	|OPPO   R815T	|4.2.1|
|vivo|	VIVO   X20A|	7.1.1|
|vivo|	Xplay6|	7.1.0|
|vivo|	VIVO   Y55A	|6.0.1|
|vivo|	X9Plus	|6.0.0|
|vivo|	VIVO Xplay5A|	5.1.1|
|vivo	|VIVO Y35A	|5.0.2|
|vivo	|VIVO X5	|4.4.4|
|vivo	|VIVO Y613	|4.2.2|
|小米|	小米 MI 6	|8.0.0|
|小米|	MiNote 3	|7.1.1|
|小米	|MIX 2	|7.1.0|
|小米|	小米 MI 4|	6.0.1|
|小米	|MI 4W	|6.0.0|
|小米	|小米 4C|	5.1.1|
|小米|	MI NOTE Pro	|5.0.2|
|小米	|MI 3W	|4.4.4|
|小米|	MI 2	|4.1.1|
|三星	|三星 Note 8	|7.1.1|
|三星|	三星 S8	|7.0.0|
|三星	|GALAXY A8	|6.0.1|
|三星|	GALAXY S5|	6.0.1|
|三星|	GALAXY On7(全网通)	|5.1.1|
|三星|	GALAXY S4（I9508）|	5.0.0|
|三星	|GALAXY A3	|4.4.4|
|三星	|SM-G3819D	|4.1.2|
|魅族	|魅族 PRO 6s|	7.1.1|
|魅族	|魅族 PRO 6 Plus|	7.0.0|
|魅族|	U20	|6.0.0|
|魅族	|M5	|6.0.0|
|魅族|	魅蓝 Note	|5.1.0|
|魅族	|MX5	|5.1.0|
|魅族	|魅蓝	|4.4.4|
|魅族	|魅族 MX3	|4.2.1|
|酷派	|大神 Note3|	5.1.0|
|酷派	|锋尚 Pro|	4.4.4|
|酷派|	酷派   8729	|4.3.0|
|红米	|小米 Redmi 4X	|7.1.2|
|红米	|Redmi Note 5A|	7.1.0|
|红米	|Redmi 4A	|6.0.1|
|红米	|Redmi Note 4X|	6.0.0
|红米|	红米 3	|5.1.1|
|红米	|红米 Note 3	|5.0.2|
|红米	|红米 note	|4.4.4|
|红米|	红米 1S	|4.4.0|
|HTC|	One A9|	7.0.0|
|HTC	|10（国际版/双 4G）|	6.0.1|
|HTC	|HTC A9	|6.0.0|
|HTC	|HTC D728w	|5.1.0|
|HTC	|HTC M9w|	5.0.2|
|HTC|	 Desire 820s	|4.4.4|
|HTC|	HTC X920e	|4.1.1|
