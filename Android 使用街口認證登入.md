# Android 使用街口認證登入

JKO Auth Login SDK 提供開發者使用「街口認證登入」 來登入您的 Android 應用程式。此份文件介紹了如何整合 JKO Auth Login SDK 於您現有的 Android 應用程式中。 

# 1. 在您的專案中引入 `aar` 檔案


新增 JKO OAuth Login SDK  arr 檔案於您專案中 module-level 的 `build.gradle` 。

***build.gradle (AppName.app)***

```groovy
dependencies {
    ...
        implementation files('libs/jkos-login-core.aar')
    ...
}
```

# 2. 設定 Resource 以及 Manifest

新增下列的字串元素：

***/app/res/values/strings.xml***

```xml
<string name="jkos_client_id">[CLIENT_ID]</string> 
<string name="jkos_oauth_login_scheme">jk[CLIENT_ID]</string>
```

您必須將字串元素的 ID（name） `jkos_client_id` and `jkos_oauth_login_scheme` 設定與上述設定完全一至，因為 SDK 直接使用了相同的 ID。

新增下列的 meta-data 元素：

***/app/manifest/AndroidManifest.xml***

```xml
<meta-data 
    android:name="com.jkos.sdk.login.sdk.ClientId" 
    android:value="@string/jkos_client_id"/> 
<meta-data 
    android:name="com.jkos.sdk.login.sdk.Scheme" 
    android:value="@string/jkos_oauth_login_scheme"/> 
```

# 3. 使用 JkoLoginClient

根據下列三步驟，您能成功使用 JkoLoginClient 物件以完成街口認證登入服務：

1. 在使用 `JkoLoginClient` 前您必須對其初始化。您可於初始化的建構子中設定  `OnJkoLoginCallbackListener` 來監聽您的登入結果，又或者於 `JkoLoginClient` 初始化後使用 setter 方式注入。監聽結果參數詳情請參考「參數」介紹。
2. 使用 `JkoLoginClient` 物件中的函式 `getJkoLoginIntent()`  取得客製化的 Intent，並將其帶入 Android 原生的函式 `startActivityForResult()` (Androidx version 1.30 前) 或 `registerForActivityResult.launch()` (Androidx version 1.30 以後) 以啟動街口認證登入服務。
    
    `getJkoLoginIntent()`  函式需將欲申請之服務帶入參數 `scopesApply` 中，若欲需申請 binding service 則需將欲連結之開發者端 User Id 帶入參數 `isvUserId` 。欲輸入之參數詳情請參考「參數」部分。
    
3. 最後，您需要在  `onActivityResult` (Androidx version 1.30 前) 或`ActivityResultCallback` (Androidx version 1.30 以後) 呼叫  `JkoLoginClient.onActivityResult()` 函式，將登入結果傳送至 `JkoLoginClient` 。

* 請確保步驟 1 與步驟 2 （依據版本不同可能為步驟 3 ） 中所設定的 `REQUEST_CODE`保持一至。

以下呈現了如何啟動街口認證登入服務以及處理登入結果。

### Before Androidx Version 1.3.0

```kotlin
companion object {
    const val REQUEST_CODE = 1
}

private lateinit var jkoLoginClient: JkoLoginClient

override fun onCreate(savedInstanceState: Bundle?) {
    ...

    // Step 1. initialize JkoLoginClient berfore you use it
    jkoLoginClient = JkoLoginClient(REQUEST_CODE, object : JkoLoginClient.OnJkoLoginCallbackListener{
        
    override fun onSuccess(
                    authCode: String,
                    scopesUserGranted: List<String>,
                    jkosUserId: String?,
                    isvUserId: String?
        ) {
        // authCode: to get access token
        // scopesUserGranted: scopes granted by the user
        // jkosUserId: JKOS’s user ID, it is nonnull when binding service is activated
        // isvUserId: SDK Developer's user ID, it is nonnull when binding service is activated
    }

        override fun onUserCancel() {

        }

        override fun onError(errorStatus: JkoErrorStatus, errorCode: String, errorMessage: String) {
            when(errorStatus) {
                    JkoErrorStatus.CIPHER_ERROR -> {}
                    JkoErrorStatus.CLIENT_PARAMETER_ERROR -> {}
                    ...
                    JkoErrorStatus.UNKNOWN_ERROR -> {}
            }
        }
    })

    // Step 2. get the specific intent when you want to start JKO Auth Login Service
    findViewById<Button>(R.id.button_login)findViewById<Button>(R.id.button_login).setOnClickListener { view ->
        val intent = jkoLoginClient.getJkoLoginIntent(
                context = view.context,
                scopesApply = listOf("binding", "pointtransmit"),
                isvUserId = "developer_user_id"
            )
        startActivityForResult(intent, REQUEST_CODE)
    }

}

// Step 3. register the function `onActivityResult` of JkoLoginClient
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    jkoLoginClient.onActivityResult(requestCode, resultCode, data)
    super.onActivityResult(requestCode, resultCode, data)
}
```

### From Androidx Version 1.3.0

```kotlin
companion object {
    const val REQUEST_CODE = 1
}

private lateinit var jkoLoginClient: JkoLoginClient

override fun onCreate(savedInstanceState: Bundle?) {
    ...
    // Step 1. initialize JkoLoginClient berfore you use it
    jkoLoginClient = JkoLoginClient(REQUEST_CODE, object : JkoLoginClient.OnJkoLoginCallbackListener{
        override fun onSuccess(
                    authCode: String,
                    scopesUserGranted: List<String>,
                    jkosUserId: String?,
                    isvUserId: String?
        ) {
          // authCode: to get access token
          // scopesUserGranted: scopes granted by the user
          // jkosUserId: JKOS’s user ID, it is nonnull when binding service is activated
          // isvUserId: SDK Developer's user ID, it is nonnull when binding service is activated
    }

        override fun onUserCancel() {
        
        }

        override fun onError(errorStatus: JkoErrorStatus, errorCode: String, errorMessage: String) {
                when(errorStatus) {
                    JkoErrorStatus.CIPHER_ERROR -> {}
                    JkoErrorStatus.CLIENT_PARAMETER_ERROR -> {}
                                        ...
                    JkoErrorStatus.UNKNOWN_ERROR -> {}
            }               
    }
    })

    // Step 2. get the specific intent when you want to start JKO Auth Login Service
    findViewById<Button>(R.id.button_login).setOnClickListener { view ->
        val intent = jkoLoginClient.getJkoLoginIntent(
                context = view.context,
                scopesApply = listOf("binding", "pointtransmit"),
                isvUserId = "developer_user_id"
            )
        resultLauncher.launch(intent)
    }
}

// Step 3. register the function `onActivityResult` of JkoLoginClient
private val resultLauncher = registerForActivityResult(ActivityResultContracts.StartActivityForResult()){result ->
    if(result == null){
        return@registerForActivityResult
    }
    jkoLoginClient.onActivityResult(REQUEST_CODE, result.resultCode, result.data)
}
```

# 參數

## JkoLoginClient.getJkoLoginIntent()

### input params
| Name | Description | 
| --- | --- |
| context | Activity 或 Fragment 的 context。 | 
| scopesApply | 開發者欲申請之授權項目。授權項目詳情見下方`SCOPE` 列表。| 
| isvUserId | 若開發者欲申請之授權項目中，需將街口 User Id 與開發者 User Id 進行連結，需於此填入欲連結之開發者 User Id 。 |

### SCOPE
| Name | Description | 
| --- | --- |
| binding | 用戶同意是否允許第三方平台用戶與街口用戶的建立關聯，獲得授權綁定關係後，後續部分業務行為會進行綁定關係的驗證。若欲申請此項 SCOPE 需於 `input params` 中輸入參數 `isvUserId` 。| 
| pointtransmit | 用戶同意從第三方帳戶補儲值至個別用戶的街口帳戶，於補儲值的過程中，會驗證用戶綁定狀態、資金方帳戶、收款方狀態等業務邏輯。| 


## JkoLoginClient.OnJkoLoginCallbackListener

### onSuccess
| Name | Description | 
| --- | --- |
| authCode | 使用者同意授權後獲得，用於後續 Server 端換取 access token。| 
| scopesUserGranted | 使用者同意之授權項目。| 
| jkosUserId | 若使用者同意之授權項目中，已將街口 User Id 與開發者 User Id 進行連結，此為回傳成功連接之街口 User Id。 |
| isvUserId | 若使用者同意之授權項目中，已將街口 User Id 與開發者 User Id 進行連結，此為回傳成功連接之開發者 User Id。|

### onError
| Name | Description | 
| --- | --- |
| errorStatus | 使用者錯誤列表，詳情見下方 `JkoErrorStatus` 列表。 | 
| errorCode | 使用者錯誤代碼，除錯時使用。| 
| errorMessage | 使用者錯誤訊息，除錯時使用。|

### JkoErrorStatus
| Name | Description | 
| --- | --- |
| CIPHER_ERROR | 安全性加解密錯誤。| 
| CLIENT_PARAMETER_ERROR | SDK 使用方輸入參數設定錯誤。| 
| SDK_INFO_ERROR | 街口資訊校驗錯誤，詳情見下方 SDK_INFO_ERROR 列表。 |
| AUTHORIZATION_ERROR | 街口登入驗證錯誤，詳情見下方 AUTHORIZATION_ERROR 列表。 |
| JKO_API_ERROR | 街口內部 API 錯誤。 |
| UNKNOWN_ERROR | 未知錯誤。|

### SDK_INFO_ERROR
| errorCode | Description | 
| --- | --- |
| 3_AL_1200 | 無法取得 meta data。| 
| 3_AL_1201 | meta data `Schema` 不可為 null 或空。| 
| 3_AL_1202 | meta data `ClientId` 不可為 null 或空。。 |
| 3_AL_1203 | 使用者的街口 app 版本過低導致無法使用此服務。|
| 3_AL_1204 | 開發者的 JKO Auth Login SDK 版本過低導致無法使用此服務。 |


### AUTHORIZATION_ERROR
| errorCode | Description | 
| --- | --- |
| 206 | 無效的 `ClientId`。| 
| 207 | 無效的 `scopesApply`。| 
| 208 | 該開發者並無申請之 `scopesApply` 使用權限。|

# 可選設定

## 取消 Bundle ID 驗證

JKO Auth Login SDK 於使用時會校驗使用方之 Bundle ID 是否與當初約定相同，開發過程中開發者可於啟動街口認證登入服務前開啟 Debug Mode 規避此項檢查。（此設定僅`BuildConfig.DEBUG` 時生效，正式版本此項設定無效。 ）

```kotlin
JkoCore.setDebugMode(true)
```

## 引導使用者開啟 Google Play 升級街口 app

若發生使用者的街口 app 版本過低導致無法使用此服務（SDK_INFO_ERROR, 3_AL_1203）情況，開發者可透過 JKO Auth Login SDK 取得客製化可直接開啟 Google Play 街口支付 app 的 `Intent` ，以引導使用者更新升級街口 app 以順利完成此服務。

```kotlin
val intent = JkoCore.getJkoGooglePlayIntent()
startActivity(intent)
```
