# Pangle Android SDK の導入マニュアル

- [Pangle Android SDK の導入マニュアル](#integrate-manual)
    - [インストールガイド](#integrate-manual)
        - [SDKのインストール](#install)
            - [動作環境](#envir)
            - [インストール](#manual-install)
        - [Android Manifestへの追加設定](#manifest-setting)
        - [難読化について](#obfuscation)
    - [AppIDとSlotIDの作成](#appid-slotid)
    - [広告の実装](#adsdk-setting)
        - [SDKの初期化](#adsdk-init)
            - [インターフェイス及パラメータの説明](#adsdk-if)
        - [広告のロード](#ad-load)
            - [TTAdManagerの初期化及びインターフェイスの説明](#ttadmanager-init)
            - [TTAdNativeによる広告のロード](#ttadnative-init)
            - [AdSlotによるロード広告情報の設定](#adslot-init)
            - [Native広告](#native-ad)
              - [Native広告のロードと表示](#native-ad-load)
              - [Native広告インターフェイスの説明](#native-ad-load)
            - [Reward広告](#reward-ad)
              - [Reward広告のロードと表示](#reward-ad-load)
              - [サーバー間の検証コールバック](#reward-ad-server)
              - [Reward広告インターフェイスの説明](#reward-ad-load)
            - [FullScreen広告](#fullscreen-ad)
              - [FullScreen広告のロードと表示](#fullscreen-ad-load)
              - [FullScreen広告インターフェイスの説明](#fullscreen-ad-load)
    - [PangleのAdmobメディエーション](#admob)
    - [付録](#appendix)
        - [SDKエラーコード](#sdk-error)
        - [FAQ](#faq)


<a name="integrate-manual"></a>
## インストールガイド

<a name="install"></a>
### SDKのインストール

<a name="envir"></a>
#### 動作環境
動作する Android のAPIレベルは 14 (Android 4.0)以上 になります。

<a name="manual-install"></a>
#### インストール
Pangle管理画面からダウンロードしたSDKのzipファイルを解凍後に、`open_ad_sdk.aar`をApplication Module/libs(ない場合は新規作成してください)にコピーし, appのbuild.gradleに下記のコードを追加してください。

```groovy
repositories {
    flatDir {
        dirs 'libs'
    }
}
depedencies {
    compile(name: 'open_ad_sdk', ext: ‘aar')
}
```


<a name="manifest-setting"></a>
### AndroidManifestへの追加設定

`AndroidManifest.xml`に次の内容を記述ください。

```xml
<!--Required  permissions-->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

<!-- If there is a video ad and it is played with textureView, please be sure to add this, otherwise a black screen will appear -->
<uses-permission android:name="android.permission.WAKE_LOCK" />

<application>
    <!--  Both single-process and multi-process should be configured.-->
    <provider
    android:name="com.bytedance.sdk.openadsdk.multipro.TTMultiProvider"
    android:authorities="${applicationId}.TTMultiProvider"
    android:exported="false" />
</application>
```

* targetSdkVersion を23以降に設定する場合, SDKで利用する必要なパーミッションの承認が必要です。

SDKで使用されるsoファイルは `x86`、`x86_64`、`armeabi`、`armeabi-v7a`、`arm64-v8a`をサポートしています。
それ以外をサポートする場合は、`build.gradle`にabiFilters設定をしてください。例：
```gradle
ndk {
  // 対象のアーキテクチャに変更
  abiFilters ‘armeabi-v7a’, ‘arm64-v8a’, ‘x86’, ‘x86_64’, ‘armeabi’
}
```


<a name="obfuscation"></a>
### 難読化について

コードの難読化をしている場合は、SDKを除外する必要があります。`proguard.cfg`に以下を追加してください。

```
-keep class com.bytedance.sdk.openadsdk.** { *; }
```


<a name="appid-slotid"></a>
## AppIDとSlotIDを作成する
Pangleの管理画面に`AppID`と`PlacementID`を作成してください。ご不明点がある場合は、担当者へお問い合わせください。


<a name="adsdk-setting"></a>
## 広告の実装

<a name="adsdk-init"></a>
### SDKの初期化

広告を取得する前に SDK を初期化する必要があります。

その時にAppIDを入れる必要があります。まだ取得していない場合は、
[AppIDとSlotIDを作成する](#appid-slotid)を参考ください。

特別な理由が無い限り、アプリの起動時に行ってください。

```java
public class DemoApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();

        // It is strongly recommended to call in Application #onCreate () method to avoid content as null
        TTAdSdk.init(context,
                            new TTAdConfig.Builder()
                                    .appId("YOUR_APP_ID")
                                    .useTextureView(false)
                                    // Use TextureView to play the video. The default setting is SurfaceView, when the context is in conflict with SurfaceView, you can use TextureView
                                    .appName("APP Name")
                                    .titleBarTheme(TTAdConstant.TITLE_BAR_THEME_DARK)
                                    .allowShowPageWhenScreenLock(true)
                                    // Allow or deny permission to display the landing page ad in the lock screen
                                    .debug(true)
                                    // Turn it on during the testing phase, you can troubleshoot with the log, remove it after launching the app
                                    .supportMultiProcess(false)
                                    // Whether to support multi-process, true indicates support
                                    .coppa(0)
                                    // Fields to indicate whether you are a child or an adult ，0:adult ，1:child
                                    .setGDPR(0)
                                    //Fields to indicate whether you are protected by GDPR,  the value of GDPR : 0 close GDRP Privacy protection ，1: open GDRP Privacy protection
                                    .build());
    }
}
```


<a name="adsdk-if"></a>
#### インターフェイス及パラメータの説明

```java
/**
 * Entrance of Pangle SDK initialization
 *
 * @param context Must be application context
 * @param config Initialize configuration and required parameters
 * @return TTAdManager instance
 */
public static TTAdManager init (Context context, TTAdConfig config);
```

```java
public Static Class TTAdConfig.Builder {
    private String mAppId; // Required parameter, set the app's AppId
    private String mAppName; // Required parameter, set the app name
    private boolean mIsPaid = false ; // Optional parameter, set whether it is a paid user: true indicates paid user, false indicates non-paid user. Default setting is false, non-paid user
    private int mTitleBarTheme = TTAdConstant.TITLE_BAR_THEME_LIGHT ; // Optional parameter, set the landing page theme, the default setting is TTAdConstant#TITLE_BAR_THEME_LIGHT
    private boolean mIsDebug = false ; // Optional parameter, whether to enable debug information output: true indicates enable, false indicates disable. The default setting is false, disable
    private boolean mIsUseTextureView = false ; // Optional parameter, set whether to use texture to play the video: true indicates use, false indicates do not use. The default setting is false, do not use (the default setting is to use surface)
    private boolean mIsSupportMultiProcess = false ; // Optional parameter, set whether to support multi-process: true indicates support, false indicates do not support. The default setting is false, do not support
    private IHttpStack mHttpStack;// Optional parameter, developers can customize external network request libraries ，the SDK default HttpUrlConnection
    private boolean mIsAsyncInit = false;// Whether initialize the SDK asynchronously or not
}
```



<a name="ad-load"></a>
### 広告のロード


<a name="ttadmanager-init"></a>
#### TTAdManagerの初期化及びインターフェイスの説明
`TTAdManager`は広告ロード用オブジェクトの生成、権限のリクエストとSDKバージョンの取得などで使用されます。

必ず`TTAdSdk.init`を実行後に対象オブジェクトを取得してください。
```java
//Must be called after initialization, otherwise it will be null
TTAdManager ttAdManager = TTAdSdk.getAdManager();
```

`TTAdManager`のインタフェースは以下のようになります。

```java
public interface TTAdManager {
    // Must use activity to create TTADNative object
    TTAdNative createAdNative (Context context);

    /**
     * Get the Pangle SDK version number
     *
     * @return Version number
     */
    String getSDKVersion ();

    /**
    * Fields to indicate whether you are protected by GDPR,
    * the value of GDPR :  0 : close GDRP Privacy protection
    *                      1 : open GDRP Privacy protection
    * @return
    */
    int getGdpr();

    /**
    * show GDPR Privacy Protection Dialog
    */
    void showPrivacyProtection();
    }
}
```

<a name="ttadnative-init"></a>
#### TTAdNativeによる広告のロード

`TTAdNative`は各フォマートの広告をロードできます。ロードのコールバックによりロード結果が通知されます。新規時の引数にActivityContextを利用してください。


```Java
TTAdNative mTTAdNative = TTAdSdk.getAdManager().createAdNative(baseContext);//baseContext is suggested to be activity
```

`TTAdNative`のインタフェースは以下のようになります。

```Java
public interface TTAdNative {
    /**
     * Asynchronously load rewarded video ad, the result will be callback via RewardVideoAdListener
     * @param adSlot
     * @param listener
     */
    void loadRewardVideoAd (AdSlot adSlot, @NonNull RewardVideoAdListener listener);

    /**
     * Asynchronously load full-screen video ad, the result will be callback through FullScreenVideoAdListener
     * @param adSlot
     * @param Listener
     */
    void loadFullScreenVideoAd (AdSlot adSlot, @NonNull FullScreenVideoAdListener Listener);

    /**
     * @param adSlot, Requests configuration information.
     * @param listener, Calls back loading results.
     * Feed ads are loaded asynchronously, and the results will be called back via FeedAdListener.
     */
     void loadNativeAd(AdSlot adSlot, @NonNull NativeAdListener listener);


     /**
     * Rewarded video ad loading Listener
     */
     Interface RewardVideoAdListener {
        /**
         * Callback of failed loading
         *
         * @param code
         * @param message
         */
        @MainThread
        void onError ( int code, String message);

        /**
         * Callback of completed ad loading, the developers can render during callback
         *
         * @param ad Rewarded video ad interface
         */
        @MainThread
        void onRewardVideoAdLoad (TTRewardVideoAd ad);

        /**
         * Callback of completed local ad loading, the developers can render during callback
         */
        void onRewardVideoCached ();
    }

     /**
     * Full-screen video ad loading listener
     */
     Interface FullScreenVideoAdListener {
        /**
         * Callback of failed loading
         *
         * @param code
         * @param message
         */
        @MainThread
        void onError ( int code, String message);

        /**
         * Callback of completed ad loading, the developers can render during this callback
         *
         * @param ad full-screen video ad interface
         */
        @MainThread
        void onFullScreenVideoAdLoad (TTFullScreenVideoAd ad);

        /**
         * Callback of completed local ad loading, the developers can render during callback
         */
        void onFullScreenVideoCached ();
}

    /**
     * Loading listener for native ads.
     */
    interface NativeAdListener {

        /**
         * Callback for failed loading.
         *
         * @param code
         * @param message
         */
        @MainThread
        void onError(int code, String message);

        /**
         * * Callback for successful ad loading. Rendering is available in this callback.
         * @param ads. Returned ad list.
         */
        @MainThread
        void onNativeAdLoad(List<TTNativeAd> ads);

    }
}
```

<a name="adslot-init"></a>
#### AdSlotによるロード広告情報の設定

ロードする広告の情報を`AdSlot`のオブジェクトに設定してください。


```java
AdSlot adSlot = new AdSlot.Builder ()
    // Required parameter, set your CodeId
    .setCodeId (" 900486272 ")
    // Required parameter, set the maximum size of the ad image and the desired aspect ratio of the image, in units of Px
    // Note: If you select a native ad on the Pangle, the returned image size may differ significantly from the size you expect
    .setImageAcceptedSize (640, 320)
    // For template ads, please set the size of the customized template ads (in dp). If your code bit belongs to a customized template ad, please check it on the media platform.    .setExpressViewAcceptedSize(expressViewWidth, expressViewHeight)
    // Optional parameter, allow or deny permission to support deeplink
    .setSupportDeepLink (true)
    // Optional parameter, set the number of ads returned per request for in-feed ads, up to 3
    .setAdCount ( 1 )
    // Required parameter for native ad requests, choose TYPE_BANNER or TYPE_INTERACTION_AD
    .setNativeAdType (AdSlot.TYPE_BANNER )
    // Parameter for rewarded video ad requests, name of the reward
    .setRewardName ( "gold coin" )
    // The number of rewards in rewarded video ad
    .setRewardAmount ( 3 )
    // User ID, a required parameter for rewarded video ads
    // It is developer's unique identifier for users; sdk pass-through is not necessary if the server is not in callback mode, it can be set to an empty string
    .setUserID ( "user123" )
    // Set how you wish the video ad to be displayed, choose from TTAdConstant.HORIZONTAL or TTAdConstant.VERTICAL
    .setOrientation (orientation)
     // Pass-through parameters and strings of rewards in rewarded video ad, if you use json object, you must use serialization as String type, it can be empty
    .setMediaExtra ( "media_extra")
    .build ();

```

<a name="native-ad"></a>
#### Native広告

<a name="native-ad-load"></a>
##### Native広告のロードと表示
`TTAdNative.loadNativeAd(AdSlot adSlot, @NonNull NativeAdListener listener)`でNative広告をロードします。ロード結果は`NativeAdListener`に通知されます。

ロード成功時に`addView`で表示します。

```java
//create the ad request params as AdSlot and pay attention to the setNativeAdtype method. For specific meaning of params, refer to the document.
    final AdSlot adSlot = new AdSlot.Builder()
                .setCodeId(codeId)
                .setSupportDeepLink(true)
                .setImageAcceptedSize(600, 257)
                .setNativeAdType(AdSlot.TYPE_BANNER) //You must call this method when request the native ad, and set the params as TYPE_BANNER or TYPE_INTERACTION_AD
                .setAdCount(1)
                .build();

//request ads and render the returned ad.       
    mTTAdNative.loadNativeAd(adSlot, new TTAdNative.NativeAdListener() {
            @Override
            public void onError(int code, String message) {
                TToast.show(NativeBannerActivity.this, "load error : " + code + ", " + message);
            }

            @Override
            public void onNativeAdLoad(List<TTNativeAd> ads) {
                if (ads.get(0) == null) {
                    return;
                }
                View bannerView = LayoutInflater.from(mContext).inflate(R.layout.native_ad, mBannerContainer, false);
                if (bannerView == null) {
                    return;
                }
                mBannerContainer.removeAllViews();
                mBannerContainer.addView(bannerView);
                // Demo NativeBannerActivity
                setAdData(bannerView, ads.get(0));
            }
        });
```

<a name="native-ad-if"></a>
##### Native広告インターフェイスの説明

```java
public interface TTNativeAd {

    /**
     * Get Pangle logo. Image size: 80*80
     *
     * @return bitmap
     */
    Bitmap getAdLogo();

    /**
     * Ads title.
     *
     * @return
     */
    String getTitle();

    /**
     * Ads description.
     *
     * @return
     */
    String getDescription();

    /**
     * Ads source.
     *
     * @return
     */
    String getSource();

    /**
     * Ad icon Image
     *
     * @return
     */
    TTImage getIcon();

    /**
     * Ad Image list
     *
     * @return
     */
    List<TTImage> getImageList();

    /**
    * Return Native Ad Interaction Type.
    *
    * @return 2:open the ad page in browser，3:open the page in app webview，4:download the app，5:phone call，others: Unidentified.     
    */
    int getInteractionType();

    /**
     * Return Native Ad Image Type.
     *
     * @return 3 large image;2 small image;4 group image;5 video; others Unidentified.
     */
    int getImageMode();

    /**
     * Return dislike dialog
     *
     * @param activity  It is recommended to pass in the current activity, otherwise the dislike dialog may not pop up.
     * @return
     */
    TTAdDislike getDislikeDialog(Activity activity);

    /**
     * custom dislike dialog
     *
     * @param dialog customized dialog, obtained from an external source.
     * @return
     */
    TTAdDislike getDislikeDialog(TTDislikeDialogAbstract dialog);


    /**
     * After you register a clickable View, click/show will be completed internally.
     *
     * @param container: Renders the outermost ViewGroup of the ad.
     * @param clickView : Clickable View
     */
    void registerViewForInteraction(@NonNull ViewGroup container, @NonNull View clickView, AdInteractionListener listener);

    /**
     * After you register a clickable View, click/show will be completed internally.
     *
     * @param container     Renders the outermost ViewGroup of the ad.
     * @param clickViews    List of clickable views.
     * @param creativeViews  Views for downloading or making phone calls.
     */
    void registerViewForInteraction(@NonNull ViewGroup container, @NonNull List<View> clickViews, @Nullable List<View> creativeViews, AdInteractionListener listener);

    /**
     * After you register a clickable View, click/show will be completed internally.     
    *
    * @param container     Renders the outermost ViewGroup of the ad.
    * @param clickViews    List of clickable views.
    * @param creativeViews   Views for downloading or making phone calls.
    * @param dislikeView   dislike icon
    * @param listener      Click callback     
    */
    void registerViewForInteraction(@NonNull ViewGroup container, @NonNull List<View> clickViews, @Nullable List<View> creativeViews, @Nullable View dislikeView, AdInteractionListener listener);

    View getAdView();

    /**
     * Callback Interface of the feed ad interaction.
     */
    interface AdInteractionListener {

        /**
         * The callback of the ad.
         *
         * @param ad
         */
        void onAdClicked(View view, TTNativeAd ad);

        /**
         * Callback when user clicks the creative area.
         *
         * @param view
         * @param ad
         */
        void onAdCreativeClick(View view, TTNativeAd ad);

        /**
        * Callback when the ad is displayed, only once for each ad.
         *
         * @param ad
         */
        void onAdShow(TTNativeAd ad);
    }
}

```

<a name="reward-ad"></a>
#### Reward広告

<a name="reward-ad-load"></a>
##### Reward広告のロードと表示

`TTAdNative.loadRewardVideoAd(AdSlot var1, @NonNull TTAdNative.RewardVideoAdListener var2)`でNative広告をロードします。ロード結果は`RewardVideoAdListener`に通知されます。

ロード成功時に`TTRewardVideoAd.showRewardVideoAd(Activit activity);
`で表示します。

```java
AdSlot adSlot = new AdSlot.Builder()
                .setCodeId("901121430")
                .setSupportDeepLink(true)
                .setAdCount(2)
                .setImageAcceptedSize(1080, 1920)
                .setRewardName("gold coin") //name of the reward
                .setRewardAmount(3) // number of rewards
                // It is developer's unique identifier for users; sdk pass-through is not necessary if the server is not in callback mode               
                // can be set to an empty string
                .setUserID("user123")
                .setOrientation(orientation) // Set how you wish the video ad to be displayed, choose from TTAdConstant.HORIZONTAL or TTAdConstant.VERTICAL
                .setMediaExtra("media_extra") // pass-through user information, not mandatory
                .build();

      mTTAdNative.loadRewardVideoAd(adSlot, new TTAdNative.RewardVideoAdListener() {
            @Override
            public void onError(int code, String message) {
                Toast.makeText(RewardVideoActivity.this, message, Toast.LENGTH_SHORT).show();
            }

            //The loaded video file is cached to the local callback
            @Override
            public void onRewardVideoCached() {
                Toast.makeText(RewardVideoActivity.this, "rewardVideoAd video cached", Toast.LENGTH_SHORT).show();
            }

            //Video creatives are loaded into, such as title, video url, etc., excluding video files
            @Override
            public void onRewardVideoAdLoad(TTRewardVideoAd ad) {
                Toast.makeText(RewardVideoActivity.this, "rewardVideoAd loaded", Toast.LENGTH_SHORT).show();
                mttRewardVideoAd = ad;
                //mttRewardVideoAd.setShowDownLoadBar(false);
                mttRewardVideoAd.setRewardAdInteractionListener(new TTRewardVideoAd.RewardAdInteractionListener() {
                    @Override
                    public void onAdShow() {
                        Toast.makeText(RewardVideoActivity.this, "rewardVideoAd show", Toast.LENGTH_SHORT).show();
                    }

                    @Override
                    public void onAdVideoBarClick() {
                        Toast.makeText(RewardVideoActivity.this, "rewardVideoAd bar click", Toast.LENGTH_SHORT).show();
                    }

                    @Override
                    public void onAdClose() {
                        Toast.makeText(RewardVideoActivity.this, "rewardVideoAd close", Toast.LENGTH_SHORT).show();
                    }

                    @Override
                    public void onVideoComplete() {
                        Toast.makeText(RewardVideoActivity.this, "rewardVideoAd complete", Toast.LENGTH_SHORT).show();
                    }

                    @Override
                    public void onVideoError() {
                        Toast.makeText(RewardVideoActivity.this, "rewardVideoAd onVideoError", Toast.LENGTH_SHORT).show();
                    }

                    @Override
                    public void onRewardVerify(boolean rewardVerify, int rewardAmount, String rewardName) {
                        Toast.makeText(RewardVideoActivity.this, "verify:" + rewardVerify + " amount:" + rewardAmount +
                                " name:" + rewardName, Toast.LENGTH_SHORT).show();
                    }
                });
            }
        });

        mttRewardVideoAd.showRewardVideoAd(RewardVideoActivity.this);

```

<a name="reward-ad-server"></a>
##### サーバー間の検証コールバック
サーバ間のコールバックにより、リワード広告を見るユーザに報酬を提供するか否かを判定できます。ユーザが広告を最後まで見た場合は、Pangleサーバからお客様自身のサーバへコールバックURLを割り当て、ユーザの操作完了を伝えることができます。

<a name="callback-manual"></a>
###### コールバック方法の説明
PangleのサーバはGET方式で第三者のサービスのコールバックURLをリクエストし、以下のパラメータを返します。
`user_id=%s&trans_id=%s&reward_name=%s&reward_amount=%d&extra=%s&sign=%s`


|フィールドの定義  |  フィールド名  |  フィールドタイプ  |  注釈 |
| --- | --- | --- | --- |
|sign   | サイン  |  string  |  サイン |
|user_id   | ユーザid  |  string |   SDKから設定され、アプリケーション側がユーザを識別ためのID|
|trans_id  |  取引id  |  string |   最後まで見た場合の唯一の取引ID|
|reward_amount   | リワード数 |   int  |  Pangleから割り当てられるかSDKから設定される|
|reward_name  |  リワード名  |  string  |  Pangleから割り当てられるかSDKから設定される|
|extra  |  Extra  |  string   | SDKから設定される。必要ない場合は`null`|

<a name="sign"></a>
###### サイン作成方法
appSecurityKey: Pangleでのリワード動画の作成で生成されたtransId：取引id sign = sha256(appSecurityKey:transId)

Pythonの例：

```python
import hashlib

if __name__ == "__main__":
trans_id = "6FEB23ACB0374985A2A52D282EDD5361u6643"
app_security_key = "7ca31ab0a59d69a42dd8abc7cf2d8fbd"
check_sign_raw = "%s:%s" % (app_security_key, trans_id)
sign = hashlib.sha256(check_sign_raw).hexdigest()
```

<a name="response"></a>
##### レスポンス
jsonデータを返します。フィールドは以下の通り。

|フィールドの定義  |  フィールド名  |  フィールドタイプ  |  注釈|
| --- | --- | --- | --- |
|isValid   | 校正結果 |   bool  |  判定結果、リワードを支給するか。|
例：

```json
{
 "isValid": true
}
```

<a name="reward-ad-if"></a>
##### Reward広告インターフェイスの説明

```Java
public interface TTRewardVideoAd {
    /**
     * Set video interaction callback
     */
    void setRewardAdInteractionListener(RewardAdInteractionListener listener);


    /**
     * Returns the interaction type of the video ad, the current interaction type is downloading
     */
    int getInteractionType ();

    /**
     * Show video ad
     */
    @MainThread
    void showRewardVideoAd(Activity activity);

    /**
     * Video interactive callback interface
     */
    public interface RewardAdInteractionListener {
        //Video ad display callback
        void onAdhow ();

        //Ad download bar click callback
        void onAdVideoBarClick ();

        //Video ad close callback
        void onAdClose ();

        //Video ad play callback
        void onVideoComplete ();

        //video ad play error
        void onVideoError();

        //Call back of video ad completion and reward validation. The parameters are valid, the number of rewards, and the name of the reward.
        void onRewardVerify(boolean rewardVerify, int rewardAmount, String rewardName);
    }
}
```


<a name="fullscreen-ad"></a>
#### FullScreen広告

<a name="fullscreen-ad-load"></a>
##### FullScreen広告のロードと表示
`TTAdNative.loadFullScreenVideoAd(AdSlot adSlot, @NonNull FullScreenVideoAdListener Listener)`でNative広告をロードします。ロード結果は`FullScreenVideoAdListener`に通知されます。

ロード成功時に`TTFullScreenVideoAd.showFullScreenVideoAd(Activit activity);
`で表示します。

```java

// Set the ad parameters
AdSlot adSlot = new AdSlot.Builder()
               .setCodeId(codeId)
               .setSupportDeepLink(true)
               .setImageAcceptedSize(1080, 1920)
               .setOrientation(orientation)
               .build();
       // Load full-screen video ad
       mTTAdNative.loadFullScreenVideoAd(adSlot, new TTAdNative.FullScreenVideoAdListener() {
           @Override
           public void onError(int code, String message) {
               TToast.show(FullScreenVideoActivity.this, message);
           }

           @Override
           public void onFullScreenVideoAdLoad(TTFullScreenVideoAd ad) {
               TToast.show(FullScreenVideoActivity.this, "FullVideoAd loaded");
               mttFullVideoAd = ad;
               mttFullVideoAd.setFullScreenVideoAdInteractionListener(new TTFullScreenVideoAd.FullScreenVideoAdInteractionListener() {

                   @Override
                   public void onAdShow() {
                       TToast.show(FullScreenVideoActivity.this, "FullVideoAd show");
                   }

                   @Override
                   public void onAdVideoBarClick() {
                       TToast.show(FullScreenVideoActivity.this, "FullVideoAd bar click");
                   }

                   @Override
                   public void onAdClose() {
                       TToast.show(FullScreenVideoActivity.this, "FullVideoAd close");
                   }

                   @Override
                   public void onVideoComplete() {
                       TToast.show(FullScreenVideoActivity.this, "FullVideoAd complete");
                   }

                   @Override
                   public void onSkippedVideo() {
                       TToast.show(FullScreenVideoActivity.this, "FullVideoAd skipped");

                   }

               });
           }

           @Override
           public void onFullScreenVideoCached() {
               TToast.show(FullScreenVideoActivity.this, "FullVideoAd video cached");
           }
       });

       mttFullVideoAd.showFullScreenVideoAd(FullScreenVideoActivity.this);
```

<a name="fullscreen-ad-if"></a>
##### FullScreen広告インターフェイスの説明
```Java
public interface TTFullScreenVideoAd {
    /**
     * Register interstitial ad interactive callback
     *
     * @param listener
     */
    void setFullScreenVideoAdInteractionListener(FullScreenVideoAdInteractionListener listener);

    /**
     * Get the interaction type for the ad
     *
     * @return 2 open in the browser (normal type) 3 landing page (normal type), 4: app download, 5: telephone dialing - 1 unknown type
     */
    int getInteractionType();

    /**
     * @param activity
     * display ad
     */
    @MainThread
    void showFullScreenVideoAd(Activity activity);

    void setShowDownLoadBar(boolean showDownLoadBar);

    /**
     * Full-screen video ad interactive listener
     */
    interface FullScreenVideoAdInteractionListener {
        /**
         * Ad display callback once per ad
         */
        void onAdShow();

        /**
         * Ad download bar click callback
         */
        void onAdVideoBarClick();

        /**
         * Ad closed callback
         */
        void onAdClose();

        /**
         * Callback after the video has finished playing
         */
        void onVideoComplete();

        /**
         * Skip video
         */
        void onSkippedVideo();

    }
}
```



<a name="admob"></a>
### PangleのAdmobメディエーション

PangleではAdmobメディエーション用のCustom Event Adapterを提供しています。そのAdapterを利用することで、AdmobメディエーションでPangleのリワード広告を利用することが可能になります。

以下の点にご注意ください。

+ **Custom Eventを設定するときは、Class Nameと実現するAdapterクラス名を一致させる必要があります。一致していない場合はadapterの呼び出しができません。**
+ **SimulatorはデフォルトではEnable test deviceタイプのデバイスに設定されており、Google Test Adsしか取得できず、Pangleのテスト広告は取得できません。Pangleの広告をテストしたい場合はiOSの実機を使用し、AdMob TestDevicesに追加しないでください。**



<a name="appendix"></a>
## 付録

<a name="sdk-error"></a>
### SDKエラーコード
データ取得における異常は主にコールバック方法で処理されます。
以下は各種のerror codeの値です。

|Error Code  |  Description
|---------|-------------------------------------|
|20000 |   Success
|40000  |  http content type error
|40001  |  http request pb error
|40002  |  source_type='app', request app can't be empty
|40003  |  source_type='wap', request wap cannot be empty
|40004  |  Ad slot cannot be empty
|40005  |  Ad slot size cannot be empty
|40006   | Illegal ad ID
|40007   | Incorrect number of ads
|40008   | Image size error
|40009   | Media ID is illegal
|40010   | Media type is illegal
|40011   | Ad type is illegal
|40012   | Media access type is illegal and has been deprecated
|40013   | Code bit id is less than 900 million, but adType is not splash ad
|40014  |  The redirect parameter is incorrect
|40015   | Media rectification exceeds deadline, request illegal
|40016  |  The relationship between slot_id and app_id is invalid.
|40017 |   Media access type is not legal API/SDK
|40018  |  Media package name is inconsistent with entry
|40019  |  Media configuration ad type is inconsistent with request
|40020  |  The ad space registered by developers exceeds daily request limit
|40021  |  Apk signature sha1 value is inconsistent with media platform entry
|40022  |  Whether the media request material is inconsistent with the media platform entry
|40023  |  OS field is incorrect
|40024  |  SDK version is too low to return ads
|40025   | SDK package is incompletely installed. It is recommended that you verify the SDK package integrity or contact our technical support
|40029  |  Wrong methods are used for template ads. Please see the FAQ description below
|50001 |   Server Error
|60001  |  Show event processing error
|60002  |  Click event processing error
|60007  |  Server abnormity or failure of rewarded video ad rewards verification
|-1   | Data parsing failed
|-2   | Network Error
|-3   | Parsing data without ad
|-4   |Return data is missing the necessary fields
|-5    |bannerAd image failed to load
|-6    |Interstitial ad image failed to load
|-7   | Splash ad image failed to load
|-8   | Frequent request
|-9   | Request entity is empty
|-10  |  Cache parsing failed
|-11  |  Cache expired
|-12  |  No splash ad in the cache
|101  |  Failed to parse the rendering result data
|102  |  Invalid main template
|103  |  Invalid template difference
|104  |  Abnormal material data
|105  |  Template data parsing error
|106  |  Rendering error
|107  |  Rendering has timed out and not called back

<a name="faq"></a>
### FAQ

**エラー40029について**  
1. SDKのバージョンが`2.5.0.0`以下時に発生します。SDKを更新してください。
2. ロードする広告フォマートがサポートされていないときに発生します。現在`Express`テンプレートがサポートしていないので、カスタムテンプレートの広告フォマートを呼ぶように変更してください。
