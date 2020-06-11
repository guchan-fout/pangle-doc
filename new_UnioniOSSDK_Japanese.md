# Pangle iOS SDK の導入マニュアル

- [Pangle iOS SDK の導入マニュアル](#integrate-manual)
    - [インストールガイド](#install-guide)
        - [SDKのインストール](#install)
            - [動作環境](#envir)
            - [CocoaPods](#cocoapods)
            - [手動インストール](#manual-install)
        - [Xcodeコンパイルオプションの設定](#xcode-setting)
            - [App Transport Security](#ats)
            - [Other Linker Flags](#flag)
            - [依存ライブラリの追加](#add-library)  
    - [AppIDとSlotIDの作成](#appid-slotid)
    - [広告の実装](#ads-setting)
        - [SDKの初期化](#buadsdkmanager)
            - [BUAdSDKManagerでのSDK初期化](#buadsdkmanager-use)
            - [インターフェイスの説明](#buadsdkmanager-if)
        - [リワード動画(BURewardedVideoAd)](#burewardedvideoad)
            - [BURewardedVideoAdインターフェイスの説明](#burewardedvideoad-if)
            - [BURewardedVideoAdコールバックの説明](#burewardedvideoad-callback)
            - [リワード動画の初期化とロード](#burewardedvideoad-instance)
            - [BURewardedVideoModel](#burewardedvideo-model)
            - [サーバー間の検証コールバック](#server-callback)
                - [コールバック方法の説明](#callback-manual)
                - [サイン作成方法](#sign)
                - [レスポンス](#response)
            - [PangleのAdmobメディエーション](#admob)
        - [フルスクリーン動画(BUFullscreenVideoAd)](#bufullscreenvideoad)
            - [BUFullscreenVideoAdインターフェイスの説明](#bufullscreenvideoad-if)
            - [BUFullscreenVideoAdコールバックの説明](#bufullscreenvideoad-callback)
            - [フルスクリーン動画の初期化とロード](#bufullscreenvideoad-instance)
    - [付録](#appendix)
        - [SDKエラーコード](#sdk-error)
        - [FAQ](#faq)


<a name="integrate-manual"></a>
## インストールガイド

<a name="install-guide"></a>
### SDKのインストール

<a name="envir"></a>
### 動作環境
SDKをアプリにインストールするには、
+ iOS 9.X 以上をターゲットに設定していること
+ Xcode 10.0 以降を使用していること
+ 対応アーキテクチャ：i386, x86-64, armv7, armv7s, arm64

<a name="cocoapods"></a>
#### CocoaPods
SDK 2.0.0.0 以降はpod方式によるインストールをサポートしています。
Podfileに以下のように記入し `pod install` することで SDK がご利用いただけます。

```pod
# also can use " pod 'Bytedance-UnionAD', '~> X.X.X.X' " to switch the version
pod 'Bytedance-UnionAD'
```

pod方式の接続について更に詳しくお知りになりたい方は[GitHub](https://github.com/bytedance/Bytedance-UnionAD)をご参照ください


<a name="manual-install"></a>
#### 手動インストール
Pangle管理画面からダウンロードしたSDKのzipファイルを解凍後に、Frameworksフォルダにある`BUAdSDK.framework`, `BUFoundation.framework`, `BUAdSDK.bundle`をプロジェクトにドラッグ&ドロップすれば完了です。

*SDKをアップデートする際に、`BUAdSDK.framework`, `BUFoundation.framework`, `BUAdSDK.bundle`とも更新してください。*

ドラッグ&ドロップするときは、以下の方法で選択してください。

![image](http://sf1-ttcdn-tos.pstatp.com/img/union-platform/9537cbbf7a663781539ae6b07f2e646b.png~0x0_q100.webp)

ドラッグ&ドロップ完了後は、Copy Bundle Resourcesに`BUAdSDK.bundle`が含まれていることを確認してください。含まれていないと、アイコン画像をロードできないことがあります。

![image](http://sf1-ttcdn-tos.pstatp.com/img/union-platform/f20b43b4fbed075820aa738ea1416bd4.png~0x0_q100.webp)


<a name="manual-install"></a>
### Xcodeコンパイルオプションの設定

<a name="ats"></a>
#### App Transport Security

App Transport Security Settingsを追加するには、`Info.plist` ファイルにAllow Arbitrary Loadsオプションを追加して、値をYESに変更してください。SDK APIは全HTTPSをサポートしていますが、広告主のクリエイティブが非HTTPSの場合もあります。

```
<key>NSAppTransportSecurity</key>
<dict>
<key>NSAllowsArbitraryLoads</key>
<true/>
</dict>
```
詳しい操作は図の通りです。

![image](http://sf1-ttcdn-tos.pstatp.com/img/union-platform/1944c1aad1895d2c6ab2ca7a259658d5.png~0x0_q100.webp)

<a name="flag"></a>
#### Other Linker Flags

Build SettingsのOther Linker Flags に **-ObjC** を追加してください。
詳しい操作は図の通りです。

![image](http://sf1-ttcdn-tos.pstatp.com/img/union-platform/e7723fa701c3ab9d9d7a787add33fdad.png~0x0_q100.webp)


<a name="add-library"></a>
#### 依存ライブラリの追加
プロジェクトはTARGETS -> Build PhasesでLink Binary With Librariesを検索し、「+」をクリック、下記依存ライブラリを追加する必要があります。

+ StoreKit.framework
+ MobileCoreServices.framework
+ WebKit.framework
+ MediaPlayer.framework
+ CoreMedia.framework
+ AVFoundation.framework
+ CoreTelephony.framework
+ CoreLocation.framework
+ SystemConfiguration.framework
+ AdSupport.framework
+ CoreMotion.framework
+ libresolv.9.tbd
+ libc++.tbd
+ libz.tbd
+ Add the imageio.framework if the above dependency library is still reporting errors.
詳しい操作は図の通りです。

![image](http://sf1-ttcdn-tos.pstatp.com/img/union-platform/9729c0facdcba2a6aec2c23378e9eee7.png~0x0_q100.webp)

<a name="appid-slotid"></a>
## AppIDとSlotIDを作成する
Pangleの管理画面にAppIDとPlacementIDを作成してください。ご不明点がある場合は、担当者へお問い合わせください。


<a name="ads-setting"></a>
## 広告の実装

<a name="buadsdkmanager"></a>
### SDKの初期化
`BUAdSDKManager`はSDKの初期化、およびグローバル情報の設定などの機能を提供するクラスです。

<a name="buadsdkmanager-use"></a>
#### 初期化
広告を取得する前に SDK を初期化する必要があります。

その時にAppIDを入れる必要があります。まだ取得していない場合は、
[AppIDとSlotIDを作成する](#appid-slotid)を参考ください。

特別な理由が無い限り、アプリの起動時に行ってください。

```Objective-C:AppDelegate.m
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    [BUAdSDKManager setAppID:@"Your APP ID"];

    return YES;
}
```


<a name="buadsdkmanager-if"></a>
#### インターフェイスの説明
現在、以下のクラスメソッドを提供しています。

```Objective-C
/**
 Register the App key that’s already been applied before requesting an ad from Pangle.
 @param appID : the unique identifier of the App
 */
+ (void)setAppID:(NSString *)appID;

/**
 Configure development mode.
 @param level : default BUAdSDKLogLevelNone
 */
+ (void)setLoglevel:(BUAdSDKLogLevel)level;

/// Set the gender of the user.
+ (void)setUserGender:(BUUserGender)userGender;

/// Set the age of the user.
+ (void)setUserAge:(NSUInteger)userAge;

/// Set the user's keywords, such as interests and hobbies, etc.
+ (void)setUserKeywords:(NSString *)keywords;

/// set additional user information.
+ (void)setUserExtData:(NSString *)data;

/// Set whether the app is a paid app, the default is a non-paid app
+ (void)setIsPaidApp:(BOOL)isPaidApp;

+ (NSString *)appID;

+ (BOOL)isPaidApp;
```


使用方法について更に詳しくお知りになりたい方は、[GitHub上のDemoプロジェクト](https://github.com/bytedance/Bytedance-UnionAD) をご参照ください。


<a name="burewardedvideoad"></a>
### リワード動画(BURewardedVideoAd)
動画リワード広告は、ユーザが動画広告を見ることでアプリ内で使用可能な報酬を手に入れる広告形式です。広告の長さは15～30秒で、スキップできません。また、広告の終了画面にてユーザを次の操作へ誘導することが可能です。


<a name="burewardedvideoad-if"></a>
#### BURewardedVideoAdインターフェイスの説明


```Objective-C
@interface BURewardedVideoAd : NSObject
@property (nonatomic, strong) BURewardedVideoModel *rewardedVideoModel;
@property (nonatomic, weak, nullable) id<BURewardedVideoAdDelegate> delegate;

/**
Whether material is effective.
Setted to YES when data is not empty and has not been displayed.
Repeated display is not billed.
*/
@property (nonatomic, getter=isAdValid, readonly) BOOL adValid;

- (instancetype)initWithSlotID:(NSString *)slotID rewardedVideoModel:(BURewardedVideoModel *)model;
- (void)loadAdData;
- (BOOL)showAdFromRootViewController:(UIViewController *)rootViewController;

@end
```

<a name="burewardedvideoad-callback"></a>
#### BURewardedVideoAdコールバックの説明
```Objective-C
@protocol BURewardedVideoAdDelegate <NSObject>

@optional

/**
This method is called when video ad material loaded successfully.
*/
- (void)rewardedVideoAdDidLoad:(BURewardedVideoAd *)rewardedVideoAd;

/**
This method is called when video ad materia failed to load.
@param error : the reason of error
*/
- (void)rewardedVideoAd:(BURewardedVideoAd *)rewardedVideoAd didFailWithError:(NSError *)error;

/**
This method is called when video cached successfully.
*/
- (void)rewardedVideoAdVideoDidLoad:(BURewardedVideoAd *)rewardedVideoAd;

/**
This method is called when video ad slot will be showing.
*/
- (void)rewardedVideoAdWillVisible:(BURewardedVideoAd *)rewardedVideoAd;

/**
This method is called when video ad slot has been shown.
*/
- (void)rewardedVideoAdDidVisible:(BURewardedVideoAd *)rewardedVideoAd;

/**
This method is called when video ad is about to close.
*/
- (void)rewardedVideoAdWillClose:(BURewardedVideoAd *)rewardedVideoAd;

/**
This method is called when video ad is closed.
*/
- (void)rewardedVideoAdDidClose:(BURewardedVideoAd *)rewardedVideoAd;

/**
This method is called when video ad is clicked.
*/
- (void)rewardedVideoAdDidClick:(BURewardedVideoAd *)rewardedVideoAd;


/**
This method is called when video ad play completed or an error occurred.
@param error : the reason of error
*/
- (void)rewardedVideoAdDidPlayFinish:(BURewardedVideoAd *)rewardedVideoAd didFailWithError:(NSError *)error;

/**
Server verification which is requested asynchronously is succeeded.
@param verify :return YES when return value is 2000.
*/
- (void)rewardedVideoAdServerRewardDidSucceed:(BURewardedVideoAd *)rewardedVideoAd verify:(BOOL)verify;

/**
Server verification which is requested asynchronously is failed.
Return value is not 2000.
*/
- (void)rewardedVideoAdServerRewardDidFail:(BURewardedVideoAd *)rewardedVideoAd;
/**
This method is called when the user clicked skip button.
*/
- (void)rewardedVideoAdDidClickSkip:(BURewardedVideoAd *)rewardedVideoAd;
@end

```

<a name="burewardedvideoad-instance"></a>
#### リワード動画のロード

**毎回新たに`BURewardedVideoAd`オブジェクトを生成し、`loadAdData`を呼び出す方法で新しいリワード動画をリクエストする必要があります。リワード広告を一度表示すると、そのオブジェクトを繰り返して使用しないようにご注意ください。**


```Objective-C
BURewardedVideoModel *model = [[BURewardedVideoModel alloc] init];
model.userId = @"123";
self.rewardedVideoAd = [[BURewardedVideoAd alloc] initWithSlotID:self.viewModel.slotID rewardedVideoModel:model];
self.rewardedVideoAd.delegate = self;
[self.rewardedVideoAd loadAdData];
```

<a name="burewardedvideoad-model"></a>
#### BURewardedVideoModel

```Objective-C
@interface BURewardedVideoModel : NSObject

/**
required.
Third-party game user_id identity.
Mainly used in the reward issuance, it is the callback pass-through parameter from server-to-server.
It is the unique identifier of each user.
In the non-server callback mode, it will also be pass-through when the video is finished playing.
Only the string can be passed in this case, not nil.
*/
@property (nonatomic, copy) NSString *userId;

//optional. reward name.
@property (nonatomic, copy) NSString *rewardName;

//optional. number of rewards.
@property (nonatomic, assign) NSInteger rewardAmount;

//optional. serialized string.
@property (nonatomic, copy) NSString *extra;

@end

```

<a name="server-callback"></a>
#### サーバー間の検証コールバック
サーバ間のコールバックにより、リワード広告を見るユーザに報酬を提供するか否かを判定できます。ユーザが広告を最後まで見た場合は、Pangleサーバからお客様自身のサーバへコールバックURLを割り当て、ユーザの操作完了を伝えることができます。

<a name="callback-manual"></a>
##### コールバック方法の説明
PangleのサーバはGET方式で第三者のサービスのコールバックURLをリクエストし、以下のパラメータを返します。
user_id=%s&trans_id=%s&reward_name=%s&reward_amount=%d&extra=%s&sign=%s


|フィールドの定義  |  フィールド名  |  フィールドタイプ  |  注釈 |
| --- | --- | --- | --- |
|sign   | サイン  |  string  |  サイン |
|user_id   | ユーザid  |  string |   SDKから設定され、アプリケーション側がユーザを識別ためのID|
|trans_id  |  取引id  |  string |   最後まで見た場合の唯一の取引ID|
|reward_amount   | リワード数 |   int  |  Pangleから割り当てられるかSDKから設定される|
|reward_name  |  リワード名  |  string  |  Pangleから割り当てられるかSDKから設定される|
|extra  |  Extra  |  string   | SDKから設定される。必要ない場合は`null`|

<a name="sign"></a>
##### サイン作成方法
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

<a name="admob"></a>
#### PangleのAdmobメディエーション

PangleではAdmobメディエーション用のCustom Event Adapterを提供しています。そのAdapterを利用することで、AdmobメディエーションでPangleのリワード広告を利用することが可能になります。

以下の点にご注意ください。

+ **Custom Eventを設定するときは、Class Nameと実現するAdapterクラス名を一致させる必要があります。一致していない場合はadapterの呼び出しができません。**
+ **iOS simulatorはデフォルトではEnable test deviceタイプのデバイスに設定されており、Google Test Adsしか取得できず、Pangleのテスト広告は取得できません。Pangleの広告をテストしたい場合はiOSの実機を使用し、AdMob TestDevicesに追加しないでください。**


<a name="fullscreen"></a>
### フルスクリーン動画(BUFullscreenVideoAd)
フルスクリーン動画はフルスクリーンで動画広告を展示する広告形式です。この種の広告の長さは15～30秒で、スキップできます。また、エンドカード終了画面が表示され、ユーザに次の操作に移るよう誘導できます。

<a name="fullscreen-if"></a>
#### BUFullscreenVideoAdインターフェイスの説明

```Objective-C
@interface BUFullscreenVideoAd : NSObject

@property (nonatomic, weak, nullable) id<BUFullscreenVideoAdDelegate> delegate;
@property (nonatomic, getter=isAdValid, readonly) BOOL adValid;

/**
Initializes video ad with slot id.
@param slotID : the unique identifier of video ad.
@return BUFullscreenVideoAd
*/
- (instancetype)initWithSlotID:(NSString *)slotID;

/**
Load video ad datas.
*/
- (void)loadAdData;

/**
Display video ad.
@param rootViewController : root view controller for displaying ad.
@return : whether it is successfully displayed.
*/
- (BOOL)showAdFromRootViewController:(UIViewController *)rootViewController;

@end
```

<a name="fullscreen-callback"></a>
#### BUFullscreenVideoAdコールバックの説明

```Objective-C
@protocol BUFullscreenVideoAdDelegate <NSObject>

@optional

/**
This method is called when video ad material loaded successfully.
*/
- (void)fullscreenVideoMaterialMetaAdDidLoad:(BUFullscreenVideoAd *)fullscreenVideoAd;

/**
This method is called when video ad materia failed to load.
@param error : the reason of error
*/
- (void)fullscreenVideoAd:(BUFullscreenVideoAd *)fullscreenVideoAd didFailWithError:(NSError *)error;

/**
This method is called when video cached successfully.
*/
- (void)fullscreenVideoAdVideoDataDidLoad:(BUFullscreenVideoAd *)fullscreenVideoAd;

/**
This method is called when video ad slot will be showing.
*/
- (void)fullscreenVideoAdWillVisible:(BUFullscreenVideoAd *)fullscreenVideoAd;

/**
This method is called when video ad slot has been shown.
*/
- (void)fullscreenVideoAdDidVisible:(BUFullscreenVideoAd *)fullscreenVideoAd;

/**
This method is called when video ad is clicked.
*/
- (void)fullscreenVideoAdDidClick:(BUFullscreenVideoAd *)fullscreenVideoAd;

/**
This method is called when video ad is about to close.
*/
- (void)fullscreenVideoAdWillClose:(BUFullscreenVideoAd *)fullscreenVideoAd;

/**
This method is called when video ad is closed.
*/
- (void)fullscreenVideoAdDidClose:(BUFullscreenVideoAd *)fullscreenVideoAd;


/**
This method is called when video ad play completed or an error occurred.
@param error : the reason of error
*/
- (void)fullscreenVideoAdDidPlayFinish:(BUFullscreenVideoAd *)fullscreenVideoAd didFailWithError:(NSError *)error;

/**
This method is called when the user clicked skip button.
*/
- (void)fullscreenVideoAdDidClickSkip:(BUFullscreenVideoAd *)fullscreenVideoAd;

@end
```

<a name="fullscreen-instance"></a>
#### フルスクリーン動画のロード
**毎回新たに`BUFullscreenVideoAd`オブジェクトを生成し、`loadAdData`を呼び出す方法で新しいフルスクリーン動画をリクエストする必要があります。動画を一度表示すると、そのオブジェクトを繰り返して使用しないようにご注意ください。**


```Objective-C
- (void)viewDidLoad {
	[super viewDidLoad];
	// Do any additional setup after loading the view.
	#warning----- Every time the data is requested, a new one BUFullscreenVideoAd needs to be initialized. Duplicate request data by the same full screen video ad is not allowed.
	self.fullscreenVideoAd = [[BUFullscreenVideoAd alloc] initWithSlotID:self.viewModel.slotID];
	self.fullscreenVideoAd.delegate = self;
	[self.fullscreenVideoAd loadAdData];
	[self.view addSubview:self.button];
}

- (UIButton *)button {
	if (!_button) {
		CGSize size = [UIScreen mainScreen].bounds.size;
		_button = [[BUDNormalButton alloc] initWithFrame:CGRectMake(0, size.height*0.75, 0, 0)];
		[_button setTitle:[NSString localizedStringForKey:ShowFullScreenVideo] forState:UIControlStateNormal];
		[_button addTarget:self action:@selector(buttonTapped:) forControlEvents:UIControlEventTouchUpInside];
	}
	return _button;
}

- (void)buttonTapped:(id)sender {
	/**Return YES when material is effective,data is not empty and has not been displayed.
	Repeated display is not charged.
	*/
	[self.fullscreenVideoAd showAdFromRootViewController:self.navigationController];
}

```

<a name="appendix"></a>
## 付録

<a name="sdk-error"></a>
### SDKエラーコード
データ取得における異常は主にコールバック方法で処理されます。

```Objective-C
- (void)rewardedVideoAd:(BURewardedVideoAd *)rewardedVideoAd didFailWithError:(NSError *)error;
- (void)fullscreenVideoAd:(BUFullscreenVideoAd *)fullscreenVideoAd didFailWithError:(NSError *)error;
```

以下は各種のerror codeの値です。

```Objective-C
typedef NS_ENUM(NSInteger, BUErrorCode) {
    BUErrorCodeTempError        = -6,       // native template is invalid
    BUErrorCodeTempAddationError= -5,       // native template addation is invalid
    BUErrorCodeOpenAPPStoreFail = -4,       // failed to open appstore
    BUErrorCodeNOAdError        = -3,       // parsed data has no ads
    BUErrorCodeNetError         = -2,       // network request failed
    BUErrorCodeParseError       = -1,       // parsing failed

    BUErrorCodeNERenderResultError= 101,    // native Express ad, render result parse fail
    BUErrorCodeNETempError        = 102,    // native Express ad, template is invalid
    BUErrorCodeNETempPluginError  = 103,    // native Express ad, template plugin is invalid
    BUErrorCodeNEDataError        = 104,    // native Express ad, data is invalid
    BUErrorCodeNEParseError       = 105,    // native Express ad, parse fail
    BUErrorCodeNERenderError      = 106,    // native Express ad, render fail
    BUErrorCodeNERenderTimoutError= 107,    // native Express ad, render timeout

    BUErrorCodeSDKStop          = 1000,     // SDK stop forcely

    BUErrorCodeParamError       = 10001,    // parameter error
    BUErrorCodeTimeout          = 10002,

    BUErrorCodeSuccess          = 20000,
    BUErrorCodeNOAD             = 20001,    // no ads

    BUErrorCodeContentType      = 40000,    // http conent_type error
    BUErrorCodeRequestPBError   = 40001,    // http request pb error
    BUErrorCodeAppEmpty         = 40002,    // request app can't be empty
    BUErrorCodeWapEMpty         = 40003,    // request wap can't be empty
    BUErrorCodeAdSlotEmpty      = 40004,    // missing ad slot description
    BUErrorCodeAdSlotSizeEmpty  = 40005,    // the ad slot size is invalid
    BUErrorCodeAdSlotIDError    = 40006,    // the ad slot ID is invalid
    BUErrorCodeAdCountError     = 40007,    // request the wrong number of ads
    BUUnionAdImageSizeError     = 40008,    // wrong image size
    BUUnionAdSiteIdError        = 40009,    // Media ID is illegal
    BUUnionAdSiteMeiaTypeError  = 40010,    // Media type is illegal
    BUUnionAdSiteAdTypeError    = 40011,    // Ad type is illegal
    BUUnionAdSiteAccessMethodError  = 40012,// Media access type is illegal and has been deprecated
    BUUnionSplashAdTypeError    = 40013,    // Code bit id is less than 900 million, but adType is not splash ad
    BUUnionRedirectError        = 40014,    // The redirect parameter is incorrect
    BUUnionRequestInvalidError  = 40015,    // Media rectification exceeds deadline, request illegal
    BUUnionAppSiteRelError      = 40016,    // The relationship between slot_id and app_id is invalid.
    BUUnionAccessMethodError    = 40017,    // Media access type is not legal API/SDK
    BUUnionPackageNameError     = 40018,    // Media package name is inconsistent with entry
    BUUnionConfigurationError   = 40019,    // Media configuration ad type is inconsistent with request
    BUUnionRequestLimitError    = 40020,    // The ad space registered by developers exceeds daily request limit
    BUUnionSignatureError       = 40021,    // Apk signature sha1 value is inconsistent with media platform entry
    BUUnionIncompleteError      = 40022,    // Whether the media request material is inconsistent with the media platform entry
    BUUnionOSError              = 40023,    // The OS field is incorrectly filled
    BUUnionLowVersion           = 40024,    // The SDK version is too low to return ads
    BUErrorCodeAdPackageIncomplete  = 40025,// the SDK package is incomplete. It is recommended to verify the integrity of SDK package or contact technical support.
    BUUnionMedialCheckError     = 40026,    // Non-international account request for overseas delivery system

    BUErrorCodeSysError         = 50001     // ad server error
};

```

<a name="faq"></a>
### FAQ

1.iOSの広告画面をappで開くと、閉じたり、戻ったりすることができません。

答え：お使いのrootViewControllerでNavigationBarを非表示にしている場合、戻ることができません。


2.リワード動画を再生するとき、orientationは設定できますか。

答え：orientationはsdkにより現在のスクリーンの状態を読み取ります。開発者による設定は不要です。

3.userIdとは何ですか。

答え：ゲームユーザーの識別し、主にリワードが有効となるかを判定する過程で、サーバからサーバへのコールバックされるパラメータで、ゲームがユーザを識別する唯一のものです。非サーバコールバックモードは動画再生終了後のコールバックにおいても、ゲームアプリケーションにパススルーされますが、この時、空の文字列がパススルーされます。

４.パッケージのサイズはどれくらいですか？

答え：demoパッケージをもとに計算すると約580kです。しかし、具体的なサイズは導入した機能により異なりますので、実際のサイズはまとめた後のパッケージサイズに準じます。

５.リワード動画とフルスクリーン動画における素材ロードのコールバック成功と広告動画におけるクリエイティブキャッシュのコールバック成功にはどのような違いがあるのでしょうか。

答え：素材ロードの成功とは広告素材のクリエイティブのロード完了を言い、広告を展示することができます。但し、動画はシングルスレッドでロードされるため、この時の動画データはキャッシュができていません。そのため、インターネットの接続状況が悪い中で動画類を再生する際は、リアルタイムでデータをロードするため、動画が止まることがあります。スムーズに再生するためには、広告動画のクリエイティブのキャッシュができる時に広告を展示されることをお勧めします。
