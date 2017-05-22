## 요약

이 항목에서는 adjust의 Android SDK에 대해 설명합니다. adjust에 대한 자세한 내용은 [adjust.com]을 참조하십시오.

web views를 쓰는 앱이며 Javascript 코드에서 Adjust 추적을 사용하고자 할 경우 안드로이드 [web views SDK 설명서](web_views.md)를 참조해 주십시오.

## 목차

* [앱 예제](#example-app)
* [기본 연동](#basic-integration)
   * [SDK 받기](#sdk-get)
   * [Adjust 모듈 들여오기](#sdk-import)
   * [프로젝트에 SDK 추가](#sdk-add)
   * [Google Play 서비스 추가](#google-play-services)
   * [권한 추가](#sdk-permissions)
   * [Proguard 설정](#sdk-proguard)
   * [Adjust 브로드캐스트 수신기](#sdk-broadcast-receiver)
   * [앱에 SDK 연동](#sdk-integrate)
   * [기본 설정](#basic-setup)
   * [세선 추적](#session-tracking)
      * [API 레벨 14 이상](#session-tracking-api14) 
      * [API 레벨 9-13](#session-tracking-api9) 
   * [Adjust 로그 기록(logging)](#adjust-logging)
   * [앱 빌드](#build-the-app)
* [부가 기능](#additional-features)
   * [이벤트 추적](#event-tracking)
      * [수익 추적](#revenue-tracking)
      * [수익 중복 제거](#revenue-deduplication)
      * [인앱 구매 검증](#iap-verification)
      * [콜백 파라미터](#callback-parameters)
      * [파트너 파라미터](#partner-parameters)
    * [세션 파라미터](#session-parameters)
      * [세션 콜백 파라미터](#session-callback-parameters)
      * [세션 파트너 파라미터](#session-partner-parameters)
      * [예약 시작(delay start)](#delay-start)
    * [속성 콜백](#attribution-callback)
    * [세션 및 이벤트 콜백](#session-event-callbacks)
    * [추적 사용 중지](#disable-tracking)
    * [오프라인 모드](#offline-mode)
    * [이벤트 버퍼링(buffering)](#event-buffering)
    * [배경 추적](#background-tracking)
    * [기기 ID](#device-ids)
      * [Google Play 서비스 광고 식별자](#di-gps-adid)
      * [Adjust 기기 식별자](#di-adid)
    * [사용자 속성](#user-attribution)
    * [푸시 토큰(push token)](#push-token)
    * [사전 설치 트래커(pre-installed trackers)](#pre-installed-trackers)
    * [딥링크](#deeplinking)
        * [기본 딥링크](#deeplinking-standard)
        * [거치(deferred) 딥링크](#deeplinking-deferred)
        * [딥링크를 통한 재어트리뷰션](#deeplinking-reattribution)
* [문제해결](#troubleshooting)
    * ["Session failed (Ignoring too frequent session. ...)" 오류가 나타납니다.](#ts-session-failed)
    * [브로드캐스트 수신기가 설치 참조 페이지를 제대로 캡처하고 있나요?](#ts-broadcast-receiver)
    * [응용 프로그램 런칭 시 이벤트를 촉발할 수 있나요?](#ts-event-at-launch) 
* [라이선스](#license)

## <a id="example-app"></a>앱 예제

[`example` 디렉토리][example]에 앱 예제가 있습니다. 안드로이드 프로젝트를 열어 adjust SDK 연동 방법의 예를 확인할 수 있습니다.

## <a id="basic-integration"></a>기본 연동

다음은 Adjust SDK를 안드로이드 프로젝트와 연동하기 위해 최소한으로 수행해야 하는 절차입니다. 여기서는 Android Studio를 안드로이드 개발에 사용하고 API 레벨 9(Gingerbread) 이상을 대상으로 한다고 가정합니다.

[Maven Repository][maven]를 사용하는 경우에는 [세 번째 단계](#sdk-add)부터 시작해도 됩니다.

### <a id="sdk-get"></a>SDK 받기

[릴리스 페이지][releases]에서 최신 버전을 다운로드합니다. 압축 파일을 선택한 폴더에 풉니다.

### <a id="sdk-import"></a>Adjust 모듈 들여오기

Android Studio 메뉴에서 `File → Import Module...`을 선택합니다.

![][import_module]

`Source directory` 필드에서 압축을 푼 폴더를 찾습니다.
`./android_sdk/Adjust/adjust` 폴더를 선택합니다. 작업을 완료하기 전에 모듈 이름인 `:adjust`가 표시되는지 확인합니다.

![][select_module]

`adjust` 모듈을 나중에 Android Studio 프로젝트로 가져옵니다.

![][imported_module]

### <a id="sdk-add"></a>프로젝트에 SDK 추가

앱의 `build.gradle` 파일을 열고 `dependencies` 블록을 찾은 후 다음 라인을 추가합니다.

```
compile project(":adjust")
```

![][gradle_adjust]

Maven을 사용하는 경우 다음 라인을 대신 추가합니다.

```
compile 'com.adjust.sdk:adjust-android:4.7.0'
```

### <a id="google-play-services"></a>Google Play 서비스 추가

2014년 8월 1일 자로 Google Play Store의 앱은 [Google 광고 ID]
[google_ad_id]를 사용하여 장치를 고유하게 식별해야 합니다. adjust SDK에서 Google 광고 ID를 사용할 수 있게 하려면 [Google Play 서비스][google_play_services]를 연동해야 합니다. 이 작업을 아직 수행하지 않은 경우 다음 단계를 수행하십시오.

1. 앱의 `build.gradle` 파일을 열고 `dependencies` 블록을 찾은 후 다음 라인을 추가합니다.

    ```
    compile 'com.google.android.gms:play-services-analytics:8.4.0'
    ```

    ![][gradle_gps]

2. **Google Play 서비스 버전 7 이상을 사용 중인 경우 이 단계를 건너뜁니다.** Package Explorer에서 Android 프로젝트의 `AndroidManifest.xml`을 엽니다. `<application>` 요소에 다음 `meta-data` 태그를 추가합니다.


    ```xml
    <meta-data android:name="com.google.android.gms.version"
               android:value="@integer/google_play_services_version" />
    ```

    ![][manifest_gps]

### <a id="sdk-permissions"></a>권한 추가

Package Explorer에서 Android 프로젝트의 `AndroidManifest.xml`을 엽니다. `INTERNET`에 대한 `uses-permission` 태그가 아직 없는 경우 이 태그를 추가합니다.

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

Google Play Store가 대상이 **아닌** 경우 다음 두 권한을 모두 추가합니다.

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
```

![][manifest_permissions]

### <a id="sdk-proguard"></a>Proguard 설정

Proguard를 사용 중인 경우 다음 행을 Proguard 파일에 추가합니다.

```
-keep public class com.adjust.sdk.** { *; }
-keep class com.google.android.gms.common.ConnectionResult {
    int SUCCESS;
}
-keep class com.google.android.gms.ads.identifier.AdvertisingIdClient {
    com.google.android.gms.ads.identifier.AdvertisingIdClient$Info getAdvertisingIdInfo(android.content.Context);
}
-keep class com.google.android.gms.ads.identifier.AdvertisingIdClient$Info {
    java.lang.String getId();
    boolean isLimitAdTrackingEnabled();
}
-keep class dalvik.system.VMRuntime {
    java.lang.String getRuntime();
}
-keep class android.os.Build {
    java.lang.String[] SUPPORTED_ABIS;
    java.lang.String CPU_ABI;
}
-keep class android.content.res.Configuration {
    android.os.LocaledList getLocales();
    java.util.Locale locale;
}
-keep class android.os.LocaledList {
    java.util.Locale get(int);
}
```

**Google Play Store가 대상이 아닌** 경우 `com.google.android.gms` 규칙을 제거할 수 있습니다.

### <a id="sdk-broadcast-receiver"></a>Adjust 브로드캐스트 수신기 추가

`INSTALL_REFERRER` intent로 다른 브로드캐스트 수신기를 사용하고 있지 않다면 `AndroidManifest.xml`에 있는 `application` 태그에 다음 `receiver` 태그를 추가합니다.

```xml
<receiver
    android:name="com.adjust.sdk.AdjustReferrerReceiver"
    android:exported="true" >
    <intent-filter>
        <action android:name="com.android.vending.INSTALL_REFERRER" />
    </intent-filter>
</receiver>
```

![][receiver]

이 브로드캐스트 수신기는 전환 트래킹을 개선하기 위해 설치 참조 페이지 검색에 사용합니다.

다른 브로드캐스트 수신기를 `INSTALL_REFERRER` intent로 이미
사용 중이라면 [이 지침][referrer]에 따라 Adjust 수신기를
추가하십시오.

### <a id="sdk-integrate"></a>앱에 SDK 연동

먼저 기본 세션 트래킹을 설정합니다.

#### <a id="basic-setup"></a>기본 설정

전역 android [응용 프로그램][android_application] 클래스를 사용하여 SDK를 초기화하는 것이 좋습니다. 앱에 이 클래스가 아직 없는 경우 다음 단계를 수행하십시오.

1. `Application`을 확장하는 클래스를 만듭니다.
    ![][application_class]

2. 앱의 `AndroidManifest.xml` 파일을 열고 `<application>` 요소를 찾습니다.
3. `android:name` 특성을 추가하고 새 응용 프로그램 클래스 이름으로 설정한 후 이름 앞에 점을 추가합니다.

    앱 예제에서는 이름이 `GlobalApplication`인 `Application` 클래스를 사용하므로, 매니페스트 파일은 다음과 같이 구성됩니다.

    ```xml
     <application
       android:name=".GlobalApplication"
       ... >
         ...
    </application>
    ```

    ![][manifest_application]

4. `Application` 클래스에서 `onCreate` 메서드를 추가하거나 만들고 다음 코드를 추가하여 adjust SDK를 초기화합니다.

    ```java
    import com.adjust.sdk.Adjust;
    import com.adjust.sdk.AdjustConfig;

    public class GlobalApplication extends Application {
        @Override
        public void onCreate() {
            super.onCreate();

            String appToken = "{YourAppToken}";
            String environment = AdjustConfig.ENVIRONMENT_SANDBOX;
            AdjustConfig config = new AdjustConfig(this, appToken, environment);
            Adjust.onCreate(config);
        }
    }
    ```

    ![][application_config]

    `{YourAppToken}`을 앱 토큰으로 대체합니다. 앱 토큰은 [대시보드][adjust.com]에서 찾을
    수 있습니다.

    앱을 테스트에 사용할지 아니면 프로덕션에 사용할 지에 따라 `environment`를 다음 값 중 하나로 설정해야 합니다.

    ```java
    String environment = AdjustConfig.ENVIRONMENT_SANDBOX;
    String environment = AdjustConfig.ENVIRONMENT_PRODUCTION;
    ```

    **중요:** 이 값은 앱을 테스트하는 경우에만
    `AdjustConfig.ENVIRONMENT_SANDBOX`로 설정해야 합니다. 앱을 게시하기 전에 environment를 `AdjustConfig.ENVIRONMENT_PRODUCTION`으로 설정해야 합니다. 개발 및 테스트를 다시 시작할 경우에는 `AdjustConfig.ENVIRONMENT_SANDBOX`로 다시 설정하십시오.

    이 environment는 실제 트래픽과 테스트 장치의 테스트 트래픽을 구별하기 위해 사용합니다. 이 값을 항상 유의미하게 유지해야 합니다! 수익을 추적하는 경우에 특히 중요합니다.

#### <a id="session-tracking"></a>세션 추적

**주의:** 이 단계는 **매우 중요**하므로 **앱에 제대로 구현했는지 반드시 확인하세요**. 구현하게 되면 Adjust SDK가 제공하는 세션 추적을 앱에서 올바르게 활성화할 수 있습니다.

#### <a id="session-tracking-api14"></a>API 레벨 14 이상

  1. `ActivityLifecycleCallbacks` 인터페이스를 구현하는 비공개 클래스를 추가합니다. 이 인터페이스에 액세스할 권한이 없다면, 앱의 대상 Android api 레벨이 14보다 낮기 때문입니다. 이 [지침](#session-tracking-api9) 따라 각 작업을 수동으로 업데이트해야 합니다. 이전에 앱의 각 작업에 대한 `Adjust.onResume` 및 `Adjust.onPause` 호출이 있었을 경우 각 호출을 제거해야 합니다.

    ![][activity_lifecycle_class]

  2. `onActivityResumed(Activity activity)` 메소드를 편집하고 `Adjust.onResume()`에 호출을 추가합니다.
`onActivityPaused(Activity activity)` 메서드를 편집하고 `Adjust.onPause()`에 호출을 추가합니다.

    ![][activity_lifecycle_methods]

  3. adjust SDK가 구성된 `onCreate()` 메소드를 추가하고 `registerActivityLifecycleCallbacks` 호출을 이전에 만든 `ActivityLifecycleCallbacks` 클래스의 인스턴스와 함께 추가합니다.

    ```
    import com.adjust.sdk.Adjust;
    import com.adjust.sdk.AdjustConfig;

    public class GlobalApplication extends Application {
        @Override
        public void onCreate() {
            super.onCreate();

            String appToken = "{YourAppToken}";
            String environment = AdjustConfig.ENVIRONMENT_SANDBOX;
            AdjustConfig config = new AdjustConfig(this, appToken, environment);
            Adjust.onCreate(config);

            registerActivityLifecycleCallbacks(new AdjustLifecycleCallbacks());

            //...
        }
    }
    private static final class AdjustLifecycleCallbacks implements ActivityLifecycleCallbacks {
        @Override
        public void onActivityResumed(Activity activity) {
            Adjust.onResume();
        }

        @Override
        public void onActivityPaused(Activity activity) {
            Adjust.onPause();
        }
        //...
    }
    ```

    ![][activity_lifecycle_register]

#### <a id="session-tracking-api9"></a>API 레벨 9-13

앱 `minSdkVersion`이 `9`와 `13` 사이일 경우, 장기 연동 절차를 간소화하기 위해 `14` 이상으로 업그레이드하는 걸 고려해 주십시오. 주요 버전의 최신 시장 점유율을 확인하려면 안드로이드 [대시보드][adjust.com]를 참조하십시오.

올바른 세션 추적을 위해서는 작업 재개 혹은 중지가 있을 때마다 Adjust SDK 메소드 몇 가지를 호출해야 합니다. 그렇게 하지 않으면 SDK가 세션 시작이나 종결을 놓칠 수 있습니다. 이 작업을 위해서는 **앱 내 각 작업 시 다음 단계를 수행해야 합니다**.

  1. 작업 소스 파일을 엽니다.
  2. 파일 맨 위에 `import` 문을 추가합니다.
  3. 작업 `onResume` 메소드에서 `Adjust.onResume` 메소드를 호출합니다. 필요 시에는 이 메소드를 만듭니다.
  4. 작업 `onPause` 메소드에서 `Adjust.onPause` 메소드를 호출합니다. 필요 시에는 이 메소드를 만듭니다.   

단계 수행 후 작업이 다음과 같이 보여야 합니다.

```java
    import com.adjust.sdk.Adjust;
// ...
public class YourActivity extends Activity {
    protected void onResume() {
        super.onResume();
        Adjust.onResume();
    }
    protected void onPause() {
        super.onPause();
        Adjust.onPause();
    }
    // ...
}
```

![](https://camo.githubusercontent.com/f23435fcdc1cbae9f18bcc8aa6ec382151bc5fd0/68747470733a2f2f7261772e6769746875622e636f6d2f61646a7573742f73646b732f6d61737465722f5265736f75726365732f616e64726f69642f76342f31345f61637469766974792e706e67)

### <a id="adjust-logging"></a>Adjust 로그 기록

다음 매개변수 중 하나를 사용하여 `AdjustConfig` 인스턴스에서 `setLogLevel`을 호출하면 테스트에 표시되는 로그의 양을 늘리거나 줄일 수 있습니다.

```java
config.setLogLevel(LogLevel.VERBOSE);   // 모든 로그 활성화
config.setLogLevel(LogLevel.DEBUG);     // 더 많은 로그 활성화
config.setLogLevel(LogLevel.INFO);      // 기본값
config.setLogLevel(LogLevel.WARN);      // info 로그 비활성화
config.setLogLevel(LogLevel.ERROR);     // 경고 역시 비활성화
config.setLogLevel(LogLevel.ASSERT);    // 오류 역시 비활성화
config.setLogLevel(LogLevel.SUPRESS);    // 모든 로그 비활성화
```

로그 출력 전부를 비활성화하고 싶다면, 로그 레벨을 supress로 설정하는 동시에 `AdjustConfig` 개체에서 생성자를 사용해야 합니다. 이 개체는 supress 로그를 지원할 지 아닐지를 가리키는 boolean 파라미터를 받습니다.  

```java
String appToken = "{YourAppToken}";
String environment = AdjustConfig.ENVIRONMENT_SANDBOX;

AdjustConfig config = new AdjustConfig(this, appToken, environment, true);
config.setLogLevel(LogLevel.SUPRESS);

Adjust.onCreate(config);
```

### <a id="build-the-app"></a>앱 빌드

안드로이드 앱을 작성하고 실행합니다. `LogCat` 뷰어에서 `tag:Adjust` 필터를 설정하여 다른 모든 로그를 숨길 수 있습니다. 앱이 시작된 후에 다음 `Install tracked` adjust 로그가 표시됩니다. 

![][log_message]

## <a id="additional-features"></a>추가 기능

Adjust SDK를 프로젝트와 연동한 후에는 다음 기능을 사용할 수
있습니다.

### <a id="event-tracking">이벤트 추적

Adjust를 사용하여 앱의 모든 이벤트를 추적할 수 있습니다. 버튼의 모든 탭을 추적하려면 [대시보드][adjust.com]에서 새 이벤트 토큰을 만들어야
합니다. 예를 들어 이벤트 토큰이 `abc123`일 경우, 버튼의 `onClick`
메소드에 다음 라인을 추가하여 클릭을 추적할 수 있습니다.

```java
AdjustEvent event = new AdjustEvent("abc123");
Adjust.trackEvent(event);
```

### <a id="revenue-tracking">수익 추적

사용자가 광고를 누르거나 인앱 구매를 하여 수익이 발생할 수 있는 경우 이벤트를 사용하여 해당 매출을 추적할 수 있습니다. 예를 들어 한 번 누를 때 0.01 유로의 수익이 발생할 경우 수익 이벤트를 다음과 같이 추적할 수 있습니다.

```cs
AdjustEvent adjustEvent = new AdjustEvent("abc123");
adjustEvent.setRevenue(0.01, "EUR");
Adjust.trackEvent(adjustEvent);
```

물론 콜백 파라미터와도 연결할 수 있습니다.

통화 토큰을 설정할 경우, Adjust가 자동으로 들어오는 수익을 미리 지정한 보고용 통화로 전환해 줍니다. 통화 전환에 관한 자세한 내용은 [여기][currency-conversion]에서 확인하세요.

수익 및 이벤트 추적에 관한 자세한 내용은 [이벤트 추적 설명서][event-tracking]에서 확인하세요.

추적 전 이벤트 환경 설정에 이벤트 인스턴스를 사용할 수 있습니다.

### <a id="revenue-deduplication">수익 중복 제거

거래 ID를 선택 사항으로 추가하여 수익 중복 추적을 피할 수 있습니다. 가장 최근에 사용한 거래 ID 10개를 기억하며, 중복 거래 ID로 이루어진 수익 이벤트는 집계하지 않습니다. 인앱 구매 추적 시 특히 유용합니다. 사용 예는 아래에 나와 있습니다.

인앱 구매를 추적할 경우, 거래가 완료되고 아이템을 구매했을 때만 `trackEvent`를 호출해야 한다는 사실을 기억하십시오. 그렇게 해야 실제로 발생하지 않은 수익을 추적하는 일을 피할 수 있습니다.

```java
AdjustEvent adjustEvent = new AdjustEvent("abc123");

adjustEvent.setRevenue(0.01, "EUR");
adjustEvent.setTransactionId("transactionId");

Adjust.trackEvent(adjustEvent);
```

### <a id="iap-verification">인앱 구매 검증

Adjust의 서버 측 수신 확인 도구인 구매 검증(Purchase Verification)을 사용하여 앱에서 이루어지는 구매의 유효성을 확인하려면 안드로이드 구매 SDK를 확인하십시오. 자세한 내용은 [여기][android-purchase-verification]에서 확인할 수 있습니다.

### <a id="callback-parameters">콜백 파라미터

[대시보드][adjust.com]에서 이벤트 콜백 URL을 등록할 수 있습니다. 이벤트를 추적할 때마다 GET 요청이 해당 URL로 전송됩니다. 이벤트를 추적하기 전에 이벤트 인스턴스에서 `addCallbackParameter`를 호출하여 콜백 파라미터를 해당 이벤트에 추가할 수 있습니다. 그러면 해당 파라미터가 콜백 URL에 추가됩니다.

예를 들어 URL `http://www.adjust.com/callback`을 등록한 경우
이벤트를 다음과 같이 추적할 수 있습니다.

```java
AdjustEvent event = new AdjustEvent("abc123");

event.addCallbackParameter("key", "value");
event.addCallbackParameter("foo", "bar");

Adjust.trackEvent(event);
```

이 경우에는 이벤트를 추적하고 다음 주소로 요청이 전송됩니다.

```
http://www.adjust.com/callback?key=value&foo=bar
```

파라미터 값으로 사용할 수 있는 `{gps_adid}`와 같은 다양한 자리 표시자(placeholder)를 지원합니다. 결과로 생성한 콜백에서 이 자리 표시자는 현재 장치의 Google Play 서비스 ID로 대체됩니다. 사용자 지정 파라미터는 저장되지 않으며 콜백에만 추가됩니다. 이벤트에 대한 콜백을 등록하지 않은 경우 해당 파라미터는 읽을 수 없습니다.

사용 가능한 값의 전체 목록을 포함한 URL 콜백 사용에 대한 자세한 내용은 [콜백 설명서][callbacks-guide]를 참조하십시오.

### <a id="partner-parameters">파트너 파라미터

Adjust 대시보드에서 활성화된 연동에 대해 네트워크 파트너로 전송할
매개변수도 추가할 수 있습니다.

위에서 설명한 콜백 매개변수의 경우와 비슷하지만,
`AdjustEvent` 인스턴스에서 `addPartnerParameter` 메소드를 호출해야 추가할 수 있습니다.

```java
AdjustEvent event = new AdjustEvent("abc123");

event.addPartnerParameter("key", "value");
event.addPartnerParameter("foo", "bar");

Adjust.trackEvent(event);
```

특별 파트너와 그 연동에 대한 자세한 내용은 [특별 파트너설명서][special-partners]를 참조하십시오.

### <a id="session-parameters">세션 파라미터 설정

일부 파라미터는 Adjust SDK 이벤트 및 세션 발생시마다 전송을 위해 저장합니다. 어느 파라미터든 한 번 저장하면 로컬에 바로 저장되므로 매번 새로 추가할 필요가 없습니다. 같은 파라미터를 두 번 저장해도 효력이 없습니다.

이 세션 파라미터는 설치 시에도 전송할 수 있도록 Adjust SDK 런칭 전에도 호출할 수 있습니다. 설치 시에 전송하지만 필요한 값은 런칭 후에야 들어갈 수 있게 하고 싶다면 Adjust SDK 런칭 시 [예약 시작](#delay-start)을 걸 수 있습니다. 

### <a id="session-callback-parameters">세션 콜백 파라미터

[이벤트](#callback-parameters)에 등록한 콜백 파라미터는 Adjust SDK 전체 이벤트 및 세션 시 전송할 목적으로 저장할 수 있습니다.

세션 콜백 파라미터는 이벤트 콜백 파라마터와 비슷한 인터페이스를 지녔지만, 이벤트에 키, 값을 추가하는 대신 `Adjust` 인스턴스에 있는 `addSessionCallbackParameter` 메소드를 호출하여 추가합니다.

```
Adjust.addSessionCallbackParameter("foo", "bar");
```

세션 콜백 파라미터는 이벤트에 추가된 콜백 파라미터와 합쳐지며, 이벤트에 추가된 콜백 파라미터가 우선권을 지닙니다. 그러나 세션에서와 같은 키로 이벤트에 콜백 파라미터를 추가한 경우 새로 추가한 콜백 파라미터가 우선권을 가집니다.

원하는 키를 `Adjust` 인스턴스의 `removeSessionCallbackParameter` 메소드로 전달하여 특정 세션 콜백 파라미터를 제거할 수 있습니다.

```
Adjust.removeSessionCallbackParameter("foo");
```

세션 콜백 파라미터의 키와 값을 전부 없애고 싶다면 `Adjust` 인스턴스의 `resetSessionCallbackParameters` 메소드로 재설정하면 됩니다.

```
Adjust.resetSessionCallbackParameters();
```

### <a id="session-partner-parameters">세션 파트너 파라미터

Adjust SDK 내 모든 이벤트 및 세션에서 전송되는 [세션 콜백 파라미터](#session-callback-parameters)가 있는 것처럼, 세션 파트너 파라미터도 있습니다.

이들 파라미터는 Adjust [대시보드][adjust.com]에서 연동 및 활성화된 네트워크 파트너에게 전송할 수 있습니다.

세션 파트너 파라미터는 이벤트 파트너 파라미터와 인터페이스가 비슷하지만, 이벤트에 키와 값을 추가하는 대신 `Adjust` 인스턴스에서 `addSessionPartnerParameter` 메소드를 호출하여 추가합니다.

```
Adjust.addSessionPartnerParameter("foo", "bar");
```

세션 파트너 파라미터는 이벤트에 추가한 파트너 파라미터와 합쳐지며, 이벤트에 추가된 파트너 파라미터가 우선순위를 지닙니다. 그러나 세션에서와 같은 키로 이벤트에 파트너 파라미터를 추가한 경우, 새로 추가한 파트너 파라미터가 우선권을 가집니다.

원하는 키를 `Adjust` 인스턴스의 `removeSessionPartnerParameter` 메소드로 전달하여 특정 세션 파트너 파라미터를 제거할 수 있습니다.

```
Adjust.removeSessionPartnerParameter("foo");
```

세션 파트너 파라미터의 키와 값을 전부 없애고 싶다면 `Adjust` 인스턴스의 `resetSessionPartnerParameters` 메소드로 재설정하면 됩니다.

```
Adjust.resetSessionPartnerParameters();
```

### <a id="delay-start">예약 시작

Adjust SDK에 예약 시작을 걸면 앱이 고유 식별자 등의 세션 파라미터를 얻어 인스톨 시에 전송할 시간을 벌 수 있습니다.

`AdjustConfig` 인스턴스의 `setDelayStart` 메소드에서 예약 시작 시각을 초 단위로 설정하세요.

```
config.setDelayStart(5.5);
```

이 경우 Adjust SDK는 최초 인스톨 세션 및 생성된 이벤트를 5.5초간 기다렸다가 전송합니다. 이 시간이 지난 후, 또는 그 사이에 `Adjust.sendFirstPackages()`을 호출했을 경우 모든 세션 파라미터가 지연된 인스톨 세션 및 이벤트에 추가되며 Adjust SDK는 원래대로 돌아옵니다.

Adjust SDK의 최대 지연 예약 시작 시간은 10초입니다.

### <a id="attribution-callback"></a>속성 콜백

트래커 속성 변경 알림을 받기 위해 콜백을 등록할 수 있습니다. 속성에서 고려하는 소스가 각각 다르기 때문에 이 정보는 동시간에 제공할 수 없습니다. 가장 간단한 방법은 익명의 단일 수신기를 생성하는 것입니다.

[해당 속성 데이터 정책][attribution-data]을 반드시 고려하세요.

SDK를 시작하기 전에 `AdjustConfig`로 익명 수신기를 추가합니다.


```java
AdjustConfig config = new AdjustConfig(this, appToken, environment);

config.setOnAttributionChangedListener(new OnAttributionChangedListener() {
    @Override
    public void onAttributionChanged(AdjustAttribution attribution) {
    }
});

Adjust.onCreate(config);
```

또는 `Application` 클래스에서 `OnAttributionChangedListener` 인터페이스를 실행하여 수신기로 설정해도 됩니다. 

```java
AdjustConfig config = new AdjustConfig(this, appToken, environment);
config.setOnAttributionChangedListener(this);
Adjust.onCreate(config);
```

수신기 함수는 SDK가 최종 속성 데이터를 접수한 수 호출됩니다. 수신기 함수에서는 `attribution` 파라미터에 액세스할 수 있습니다. 해당 파라미터에 대한 개요는 다음과 같습니다.

- `string trackerToken` 현재 설치된 트래커 토큰.
- `string trackerName` 현재 설치된 트래커 이름.
- `string network` 현재 설치된 네트워크 그룹화 기준.
- `string campaign` 현재 설치된 캠페인 그룹화 기준.
- `string adgroup` 현재 설치된 ad group 그룹화 기준.
- `string creative` 현재 설치된 크리에이티브 그룹화 기준.
- `string clickLabel` 현재 설치된 클릭 레이블.
- `string adid` Adjust 장치 식별자.

### <a id="session-event-callbacks"></a>세션 및 이벤트 콜백

수신기를 등록하여 이벤트나 세션 추적 시 알림을 받을 수 있습니다. 수신기에는 4가지가 있습니다. 이벤트 성공 추적, 이벤트 실패 추적, 세션 성공 추적, 그리고 세션 실패 추적입니다. `AdjustConfig` 개체를 생성한 후 수신기 수를 추가할 수 있습니다. 

```java
AdjustConfig config = new AdjustConfig(this, appToken, environment);

// Set event success tracking delegate.
config.setOnEventTrackingSucceededListener(new OnEventTrackingSucceededListener() {
    @Override
    public void onFinishedEventTrackingSucceeded(AdjustEventSuccess eventSuccessResponseData) {
        // ...
    }
});

// Set event failure tracking delegate.
config.setOnEventTrackingFailedListener(new OnEventTrackingFailedListener() {
    @Override
    public void onFinishedEventTrackingFailed(AdjustEventFailure eventFailureResponseData) {
        // ...
    }
});

// Set session success tracking delegate.
config.setOnSessionTrackingSucceededListener(new OnSessionTrackingSucceededListener() {
    @Override
    public void onFinishedSessionTrackingSucceeded(AdjustSessionSuccess sessionSuccessResponseData) {
        // ...
    }
});

// Set session failure tracking delegate.
config.setOnSessionTrackingFailedListener(new OnSessionTrackingFailedListener() {
    @Override
    public void onFinishedSessionTrackingFailed(AdjustSessionFailure sessionFailureResponseData) {
        // ...
    }
});

Adjust.onCreate(config);
```

수신기 함수는 SDK가 서버에 패키지 전송을 시도한 후에 호출됩니다. 수신기 함수에서 수신기에 대한 특정 응답 데이터 개체에 액세스할 수 있습니다. 세션 성공 시 응답 데이터 개체 필드 개요는 다음과 같습니다.

- `String message` 서버에서 전송한 메시지 또는 SDK가 기록한 오류
- `String timestamp` 서버에서 전송한 데이터의 타임스탬프
- `String adid` Adjust가 제공하는 고유 장치 식별자
- `JSONObject jsonResponse` 서버로부터의 응답이 있는 JSON 개체

이벤트 응답 데이터 개체 두 가지에는 다음 정보가 포함됩니다.

- `String eventToken` 추적 패키지가 이벤트인 경우 이벤트 토큰

그리고 이벤트 및 세션 실패 개체에는 다음 정보도 포함됩니다.

- `boolean willRetry` 나중에 패키지 재전송 시도가 있을 것임을 나타냅니다.

### <a id="disable-tracking"></a>추적 사용 중지

`setEnabled`를 `false` 파라미터로 설정한 상태로 호출하면 Adjust SDK에서 현재 장치의 모든 작업 추적을 중지할 수 있습니다. **이 설정은 세션 간에 기억됩니다**.

```java
Adjust.setEnabled(false);
```

`isEnabled` 함수를 호출하여 Adjust SDK가 현재 사용 가능한지 확인할 수 있습니다. 매개변수가 `true`로 설정된 `setEnabled`를 호출하면 Adjust SDK를 언제든지 활성화할 수 있습니다.

### <a id="offline-mode"></a>오프라인 모드

Adjust SDK를 오프라인 모드로 전환하여 서버로 전송하는 작업을 일시 중단하고 추적 데이터를 보관하여 나중에 보낼 수 있습니다. 오프라인 모드일 때는 모든 정보가 파일에 저장되므로, 너무 많은 이벤트가 촉발되지 않도록 주의하십시오.

`setOfflineMode`를 `true` 매개변수로 설정한 상태로 호출하면 오프라인 모드를 활성화할 수 있습니다.

```java
Adjust.setOfflineMode(true);
```

또는 `setOfflineMode`를 `false`로 설정한 상태로 호출하면 오프라인 모드를 비활성화할 수 있습니다.
adjust SDK를 다시 온라인 모드로 전환하면 저장된 정보가 모두 올바른 시간 정보와 함께 adjust 서버로 전송됩니다.

추적 사용 중지와 달리 이 설정은 세션 간에 **기억되지
않습니다.** 따라서 앱을 오프라인 모드에서 종료한 경우에도 SDK는
항상 온라인 모드로 시작됩니다.

### <a id="event-buffering"></a>이벤트 버퍼링

앱이 이벤트 추적을 많이 사용하는 경우, 매 분마다 배치 하나씩만 보내도록 하기 위해 일부 HTTP 요청을 지연시키고자 할 경우가 있을 수 있습니다. `AdjustConfig` 인스턴스로 이벤트 버퍼링을 적용할 수 있습니다.

```java
AdjustConfig adjustConfig = new AdjustConfig("{YourAppToken", "{YourEnvironment}");

adjustConfig.setEventBufferingEnabled(true);

Adjust.start(adjustConfig);
```

### <a id="background-tracking"></a>배경 추적

Adjust SDK 기본값 행위는 **앱이 배경에 있을 동안에는 HTTP 요청 전송을 잠시 중지**하는 것입니다. `AdjustConfig` 인스턴스에서 `setSendInBackground` 메소드를 호출하면 이를 바꿀 수 있습니다.

```cppAdjustConfig adjustConfig = new AdjustConfig("{YourAppToken", "{YourEnvironment}");

adjustConfig.setSendInBackground(true);

Adjust.start(adjustConfig);
```

### <a id="device-ids"></a>장치 ID

Adjust SDK로 장치 식별자 몇 가지를 획득할 수 있습니다.

### <a id="di-gps-adid"></a>Google Play 서비스 광고 식별자

Google Analytics와 같은 서비스를 사용하려면 중복 보고가 발생하지 않도록 장치 ID와 클라이언트 ID를 조정해야 합니다.

Google 광고 ID가 필요한 경우 제한 사항으로 인해 배경 스레드에서만 ID를 읽을 수 있습니다. `getGoogleAdId` 함수를 컨텍스트와 `OnDeviceIdsRead` 인스턴스와 함께 호출하면 상황에 관계 없이 작동합니다.

```java
Adjust.getGoogleAdId(this, new OnDeviceIdsRead() {
    @Override
    public void onGoogleAdIdRead(String googleAdId) {
        // ...
    }
});
```

`OnDeviceIdsRead` 인스턴스의 `onGoogleAdIdRead` 메소드를 통해 Google 광고 ID에 `googleAdId` 변수로 액세스할 수 있습니다.

### <a id="di-adid"></a>Adjust 장치 식별자

Adjust 백엔드는 앱을 인스톨한 장치에서 고유한 **Adjust 장치 식별자** (**adid**)를 생성합니다. 이 식별자를 얻으려면 `Adjust` 인스턴스에서 다음 메소드를 호출하면 됩니다.

```cpp
String adid = Adjust.getAdid();
```

**주의**: **adid** 관련 정보는 Adjust 백엔드가 앱 인스톨을 추적한 후에만 얻을 수 있습니다. 그 순간부터 Adjust SDK는 장치 **adid** 정보를 갖게 되며 이 메소드로 억세스할 수 있습니다. 따라서 SDK가 초기화되고 앱 인스톨 추적이 성공적으로 이루어지기 전에는 **adid** 억세스가 **불가능합니다**.

### <a id="user-attribution"></a>사용자 속성

[속성 콜백 섹션](#attribution-callback)에서 설명한 바와 같이, 이 콜백은 변동이 있을 때마다 새로운 속성 관련 정보를 전달할 목적으로 촉발됩니다. 사용자의 현재 속성 값 관련 정보를 언제든 억세스하고 싶다면, `Adjust` 인스턴스의 다음 메소드를 호출하면 됩니다.

```cpp
AdjustAttribution attribution = Adjust.getAttribution();
```

**주의**: 유저의 현재 속성 값 관련 정보는 Adjust 백엔드가 앱 인스톨을 추적하여 최초 속성 콜백이 촉발된 후에만 얻을 수 있습니다. 그 순간부터 Adjus SDK는 유저 속성 값 정보를 갖게 되며 이 메소드로 억세스할 수 있습니다. 따라서 SDK가 초기화되고 최초 속성 콜백이 촉발되기 전에는 유저 속성 값 억세스가 **불가능합니다**. 

### <a id="push-token"></a>푸시 토큰

푸시 알림 토큰을 전송하려면 앱에서 토큰을 받거나 값 변화가 있을 때마다 아래와 같이 Adjust에 대한 호출을 추가하세요.

```java
Adjust.setPushToken(pushNotificationsToken);
```

### <a id="pre-installed-trackers">사전 설치 트래커

Adjust SDK를 사용하여 앱이 사전 설치된 장치를 지닌 사용자를 인식하고 싶다면 다음 절차를 따르세요.

1. [대시보드][adjust.com]에 새 트래커를 생성합니다.
2. `AdjustConfig` 인스턴스의 기본값 트래커를 다음과 같이 설정합니다.

    ```objc
    AdjustConfig config = new AdjustConfig(this, appToken, environment);
    config.setDefaultTracker("{TrackerToken}");
    Adjust.onCreate(config);
    ```

  `{TrackerToken}`을 2에서 생성한 트래커 토큰으로 대체합니다. 대시보드에서는 (`http://app.adjust.com/`을 포함하는) 트래커 URL을 표시한다는 사실을 명심하세요. 소스코드에서는 전체 URL을 표시할 수 없으며 6자로 이루어진 토큰만을 명시해야 합니다.

3. 앱 빌드를 실행하세요. 앱 로그 출력 시 다음과 같은 라인을 볼 수 있을 것입니다.

    ```
    Default tracker: 'abc123'
    ```

### <a id="deeplinking"></a>딥링크

URL에서 앱으로 딥링크를 거는 옵션이 있는 Adjust 트래커 URL을 사용하고 있다면, 딥링크 URL과 그 내용 관련 정보를 얻을 가능성이 있습니다. 해당 URL 클릭 시 사용자가 이미 앱을 설치한 상태(기본 딥링크)일 수도, 앱을 설치하지 않은 상태(거치 딥링크)일 수도 있습니다. 기본 딥링크 상황에서 안드로이드는 딥링크 내용에 관한 정보 인출을 기본 지원합니다. 안드로이드는 거치 딥링크를 기본 지원하지 않지만, Adjust SDK는 거치 딥링크 정보를 인출하는 메커니즘을 제공합니다.

#### <a id="deeplinking-standard">기본 딥링크

사용자가 앱을 설치하고 `deep_link` 파라미터가 들어간 Adjust 트래커 URL을 클릭 시 런칭하도록 하려 할 경우, 앱에 딥링크를 활성화해야 합니다. 이를 위해서는 원하는 **특정 고유 스킴명 (scheme name)**을 선택하여 사용자가 링크를 클릭하고 앱이 열릴 때 런칭할 작업을 배정하여 이루어집니다. 이 과정은 `AndroidManifest.xml`에 설정되어 있습니다. `intent-filter` 섹션을 매니페스트 파일 내 원하는 작업 정의에 추가하고 `android:scheme` 속성값을 원하는 스킴명과 함께 배정하면 됩니다. 

```xml
    <activity
    android:name=".MainActivity"
    android:configChanges="orientation|keyboardHidden"
    android:label="@string/app_name"
    android:screenOrientation="portrait">

    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>

    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="adjustExample" />
    </intent-filter>
    </activity>
```
이 설정이 끝나고, 트래커 URL 클릭 시 앱이 런칭되도록 하려면 배정한 스킴명을 Adjust 트래커 URL의 `deep_link` 파라미터에 사용해야 합니다. 딥링크에 정보가 추가되지 않은 트래커 URL은 아래와 같이 보일 것입니다.

```
https://app.adjust.com/abc123?deep_link=adjustExample%3A%2F%2F
```
  
URL 내 `deep_link` 파라미터 값은 **URL 인코딩이 되어야 한다**는 사실을 기억해 주세요.

앱이 위와 같이 설정된 상황에서 이 트래커 URL을 클릭하면, `MainActivity` 인텐트와 함께 앱이 런칭됩니다. `MainActivity` 클래스에서 `deep_link` 파라미터 내용이 자동으로 제공됩니다. URL에서 인코딩이 된 상태라 해도 내용이 전달된 후에는 **인코딩이 이루어지지 않습니다**.

`AndroidManifest.xml` 파일 내 `android:launchMode` 작업 설정에 따라 `deep_link` 파라미터 내용 정보가 작업 파일 내 적절한 위치로 전달됩니다. `android:launchMode` 속성 값에 대한 더 자세한 정보는 [안드로이드 문서](https://developer.android.com/guide/topics/manifest/activity-element.html)에서 확인하세요.  

딥링크 내용 정보는 `Intent` 객체를 통해 원하는 작업 내 `onCreate` 메소드 또는 `onNewIntent` 메소드로 전달됩니다. 앱을 런칭하고 두 개 메소드 중 하나가 촉발되면 실제 딥링크가 클릭한 URL 내 `deep_link` 파라미터로 전달되게 할 수 있습니다. 이렇게 하면 이 정보를 사용하여 앱에서 추가적 로직을 수행할 수 있게 됩니다.    

딥링크 내용은 이들 두 개 메소드에서 다음과 같이 추출할 수 있습니다.

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    Intent intent = getIntent();
    Uri data = intent.getData();

    // data.toString() -> This is your deep_link parameter value.
}
```

```java
@Override
protected void onNewIntent(Intent intent) {
    super.onNewIntent(intent);

    Uri data = intent.getData();

    // data.toString() -> This is your deep_link parameter value.
}
```

#### <a id="deeplinking-deferred">거치 딥링크

거치 딥링크는 사용자가 `deep_link` 파라미터가 들어 있는 Adjust 트래커 URL을 클릭했으나, 그 시점에 장치에 앱을 설치하지 않은 경우 발생합니다. 클릭 후 사용자는 Play Store로 재이동하여 앱을 다운로드하게 됩니다. 링크를 처음 연 후 `deep_link` 파라미터 내용이 앱으로 전달됩니다. 

거치 딥링크에서 `deep_link` 파라미터 내용 정보를 얻으려면 `AdjustConfig` 개체에 수신기를 설치해야 합니다. Adjust SDK가 백엔드에서 딥링크 정보를 얻으면 수신기가 촉발됩니다.

```
AdjustConfig config = new AdjustConfig(this, appToken, environment);

// Evaluate the deeplink to be launched.
config.setOnDeeplinkResponseListener(new OnDeeplinkResponseListener() {
    @Override
    public boolean launchReceivedDeeplink(Uri deeplink) {
        // ...
        if (shouldAdjustSdkLaunchTheDeeplink(deeplink)) {
            return true;
        } else {
            return false;
        }
    }
});

Adjust.onCreate(config);
```

Adjust SDK가 백엔드에서 딥링크 내용 정보를 수신하면 그 내용이 수신기에 전달되며 `boolean` 리턴값을 요청합니다. 리턴값은 (기본 딥링크에서와 마찬가지로) Adjust SDK가 스킴명을 배정한 작업을 딥링크에서 런칭할 것인지 여부를 표시합니다.    

리턴값을 `true`로 설정하면 작업이 런칭되어 [기본 딥링크](#deeplinking-standard)에서 설명한 것과 같은 결과를 구현합니다. SDK가 작업을 런칭하기를 원하지 않는다면, 수신기에서 `false`를 리턴하여 딥링크 내용을 토대로 앱에서 다음 작업을 어떻게 실행할 지 스스로 정할 수 있습니다.

#### <a id="deeplinking-reattribution">딥링크를 통한 재어트리뷰션

Adjust는 딥링크를 사용하여 광고 캠페인 리인게이지먼트(re-engagement)를 수행할 수 있게 해줍니다. 이에 대한 자세한 정보는 [관련 문서](https://docs.adjust.com/en/deeplinking/#manually-appending-attribution-data-to-a-deep-link)를 참조하세요. 

이 기능을 사용 중이라면, 사용자를 올바로 리어트리뷰트하기 위해 앱에서 호출을 하나 더 수행해야 합니다.

앱에서 딥링크 내용을 수신했다면, `Adjust.appWillOpenUrl` 메소드 호출을 추가하세요. 이 호출이 이루어지면 Adjust SDK는 딥링크 내에 새로운 어트리뷰션 정보가 있는지 확인하고, 새 정보가 있으면 Adjust 백엔드로 송신합니다. 딥링크 정보가 담긴 Adjust 트래커 URL을 클릭한 유저를 리어트리뷰트해야 할 경우 앱에서 해당 사용자의 새 속성 정보로 [속성 콜백](#attribution-callback)이 촉발되는 것을 확인할 수 있습니다.    

`Adjust.appWillOpenUrl` 호출은 다음과 같이 이루어집니다.

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    Intent intent = getIntent();
    Uri data = intent.getData();

    Adjust.appWillOpenUrl(data);
}
```

```java
@Override
protected void onNewIntent(Intent intent) {
    super.onNewIntent(intent);

    Uri data = intent.getData();

    Adjust.appWillOpenUrl(data);
}
```

## <a id="troubleshooting">문제 해결

### <a id="ts-session-failed">"Session failed (Ignoring too frequent session. ...)" 오류가 나타납니다.

이 오류는 일반적으로 설치를 테스트할 때 발생합니다. 앱을 제거하고 다시 설치해도 새 설치를 촉발할 수 없습니다. 서버에서는 SDK가 로컬에서 집계한 세션 데이터를
유실했다고 판단하며 서버에 제공된 장치 관련 정보에 따라 오류 메시지를 무시합니다.

이 동작은 테스트 중 불편을 초래할 수도 있지만, sandbox 동작이 프로덕션 환경과 최대한
일치하도록 하기 위해 필요합니다.

장치의 세션 데이터를 Adjust 서버에서 재설정할 수 있습니다. 로그에서 다음 오류 메시지를 확인합니다.

```
Session failed (Ignoring too frequent session. Last session: YYYY-MM-DDTHH:mm:ss, this session: YYYY-MM-DDTHH:mm:ss, interval: XXs, min interval: 20m) (app_token: {yourAppToken}, adid: {adidValue})
```

아래에 `{yourAppToken}` 및 `{adidValue}/{gps_adidValue}/{androidIDValue}` 값을 입력하고 다음 링크 중 하나를 엽니다.

```
http://app.adjust.com/forget_device?app_token={yourAppToken}&adid={adidValue}
```


```
http://app.adjust.com/forget_device?app_token={yourAppToken}&gps_adid={gps_adidValue}
```

```
http://app.adjust.com/forget_device?app_token={yourAppToken}&android_id={androidIDValue}
```

장치가 메모리에서 삭제되면 링크에서 `Forgot device`만 반환됩니다. 장치가 이미 메모리에서 삭제되었거나 값이 올바르지 않으면 `Device not found` 메시지가 반환됩니다.

### <a id="ts-broadcast-receiver">브로드캐스트 수신기가 설치 참조 페이지를 제대로 캡처하고 있나요?

[설명서](#broadcast-receiver) 지침을 따랐다면 브로드캐스트 수신기는 설치 참조 페이지를 Adjust SDK와 Adjust 서버로 보내도록 구성됩니다.

테스트 설치 참조 페이지를 수동으로 촉발시켜 이 구성을 테스트할 수 있습니다. `com.your.appid`를 앱 ID로 대체하고 Android Studio와 함께 제공되는 [adb](http://developer.android.com/tools/help/adb.html) 도구를 사용하여 다음 명령을 실행합니다.

```
adb shell am broadcast -a com.android.vending.INSTALL_REFERRER -n com.your.appid/com.adjust.sdk.AdjustReferrerReceiver --es "referrer" "adjust_reftag%3Dabc1234%26tracking_id%3D123456789%26utm_source%3Dnetwork%26utm_medium%3Dbanner%26utm_campaign%3Dcampaign"
```

이미 다른 브로드캐스트 수신기를 `INSTALL_REFERRER` intent로 사용 중이고 이 [설명서][referrer]의 지침을 따른 경우, `com.adjust.sdk.AdjustReferrerReceiver`를 브로드캐스트 수신기로 대체합니다.

`-n com.your.appid/com.adjust.sdk.AdjustReferrerReceiver` 파라미터를 제거하여 장치의 모든 앱에서 `INSTALL_REFERRER` intent를 수신하도록 할 수도 있습니다.

로그 레벨을 `verbose`로 설정하면 아래 참조 페이지,

```
V/Adjust: Reading query string (adjust_reftag=abc1234&tracking_id=123456789&utm_source=network&utm_medium=banner&utm_campaign=campaign) from reftag
```

그리고 SDK 패키지 처리기에 추가된 다음 클릭 패키지를 읽어서 로그를 볼 수 있습니다.

```
V/Adjust: Path:      /sdk_click
    ClientSdk: android4.6.0
    Parameters:
        app_token        abc123abc123
        click_time       yyyy-MM-dd'T'HH:mm:ss.SSS'Z'Z
        created_at       yyyy-MM-dd'T'HH:mm:ss.SSS'Z'Z
        environment      sandbox
        gps_adid         12345678-0abc-de12-3456-7890abcdef12
        needs_attribution_data 1
        referrer         adjust_reftag=abc1234&tracking_id=123456789&utm_source=network&utm_medium=banner&utm_campaign=campaign
        reftag           abc1234
        source           reftag
        tracking_enabled 1
```

앱 시작 전에 이 테스트를 실시하면 전송할 패키지가 보이지 않습니다. 패키지는 앱이 시작된 후에 전송됩니다.

### <a id="ts-event-at-launch">응용 프로그램 런칭 시 이벤트를 촉발할 수 있나요?

직관적으로 생각하는 것과는 다를 수 있습니다. 전역 `Application` 클래스의 `onCreate` 메소드는 응용 프로그램 시작 시 뿐만 아니라, 시스템 또는 응용 프로그램 이벤트가 앱에 의해 캡처될 때도 호출됩니다.

Adjust SDK는 이 때 초기화가 준비되지만 실제로 시작되지는 않습니다. 작업 시작 시, 즉 사용자가 실제로 앱을 시작할 경우에만 실제로 시작됩니다.

따라서 이 때 이벤트를 촉발해도 원하는 작업이 수행되지 않습니다. 이런 호출은 사용자가 앱을 시작하지 않은 경우에도 외부 요인에 따라 결정되는 시간에 맞춰 Adjust SDK를 시작하고 이벤트를 보냅니다.

따라서 응용 프로그램 시작 시 이벤트를 촉발시키면 추적하는 설치 및 세션 수가 부정확해집니다.

설치 후에 이벤트를 촉발시키려면 [속성 콜백](#attribution-callback)를 사용하십시오.

앱이 시작될 때 이벤트를 촉발시키려면 시작된 작업의 `onCreate` 메소드를 사용하십시오.

[dashboard]:                      http://adjust.com
[adjust.com]:                     http://adjust.com

[maven]:                          http://maven.org
[example]:                        https://github.com/adjust/android_sdk/tree/master/Adjust/example
[releases]:                       https://github.com/adjust/adjust_android_sdk/releases
[referrer]:                       https://github.com/adjust/sdks/blob/master/doc/ko-sdks/referrer.md
[google_ad_id]:                   https://support.google.com/googleplay/android-developer/answer/6048248?hl=en
[event-tracking]:                 https://docs.adjust.com/ko/event-tracking
[callbacks-guide]:                https://docs.adjust.com/ko/callbacks
[application_name]:               http://developer.android.com/guide/topics/manifest/application-element.html#nm
[special-partners]:               https://docs.adjust.com/ko/special-partners
[attribution-data]:               https://github.com/adjust/sdks/blob/master/doc/attribution-data.md
[android-dashboard]:              http://developer.android.com/about/dashboards/index.html
[currency-conversion]:            https://docs.adjust.com/ko/event-tracking/#part-7
[android_application]:            http://developer.android.com/reference/android/app/Application.html
[android-launch-modes]:           https://developer.android.com/guide/topics/manifest/activity-element.html
[google_play_services]:           http://developer.android.com/google/play-services/setup.html
[activity_resume_pause]:          doc/activity_resume_pause.md
[reattribution-with-deeplinks]:   https://docs.adjust.com/ko/deeplinking/#part-6-1
[android-purchase-verification]:  https://github.com/adjust/android_purchase_sdk

[activity]:                     https://raw.github.com/adjust/sdks/master/Resources/android/v4/14_activity.png
[proguard]:                     https://raw.github.com/adjust/sdks/master/Resources/android/v4/08_proguard_new.png
[receiver]:                     https://raw.github.com/adjust/sdks/master/Resources/android/v4/09_receiver.png
[gradle_gps]:                   https://raw.github.com/adjust/sdks/master/Resources/android/v4/05_gradle_gps.png
[log_message]:                  https://raw.github.com/adjust/sdks/master/Resources/android/v4/15_log_message.png
[manifest_gps]:                 https://raw.github.com/adjust/sdks/master/Resources/android/v4/06_manifest_gps.png
[gradle_adjust]:                https://raw.github.com/adjust/sdks/master/Resources/android/v4/04_gradle_adjust.png
[import_module]:                https://raw.github.com/adjust/sdks/master/Resources/android/v4/01_import_module.png
[select_module]:                https://raw.github.com/adjust/sdks/master/Resources/android/v4/02_select_module.png
[imported_module]:              https://raw.github.com/adjust/sdks/master/Resources/android/v4/03_imported_module.png
[application_class]:            https://raw.github.com/adjust/sdks/master/Resources/android/v4/11_application_class.png
[application_config]:           https://raw.github.com/adjust/sdks/master/Resources/android/v4/13_application_config.png
[manifest_permissions]:         https://raw.github.com/adjust/sdks/master/Resources/android/v4/07_manifest_permissions.png
[manifest_application]:         https://raw.github.com/adjust/sdks/master/Resources/android/v4/12_manifest_application.png
[activity_lifecycle_class]:     https://raw.github.com/adjust/sdks/master/Resources/android/v4/16_activity_lifecycle_class.png
[activity_lifecycle_methods]:   https://raw.github.com/adjust/sdks/master/Resources/android/v4/17_activity_lifecycle_methods.png
[activity_lifecycle_register]:  https://raw.github.com/adjust/sdks/master/Resources/android/v4/18_activity_lifecycle_register.png

## <a id="license"></a>라이선스

adjust SDK는 MIT 라이선스에 따라 사용이 허가되었습니다.

Copyright (c) 2012-2017 adjust GmbH,
http://www.adjust.com

이로써 본 소프트웨어와 관련 문서 파일(이하 "소프트웨어")의 복사본을 받는 사람에게는 아래 조건에 따라 소프트웨어를 제한 없이 다룰 수 있는 권한이 무료로 부여됩니다. 이 권한에는 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여할 권리와 소프트웨어를 제공 받은 사람이 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여하는 것을 허가할 수 있는 권리가 제한 없이 포함됩니다.

위 저작권 고지문과 본 권한 고지문은 소프트웨어의 모든 복사본이나 주요 부분에 포함되어야 합니다.

소프트웨어는 상품성, 특정 용도에 대한 적합성 및 비침해에 대한 보증 등을 비롯한 어떤 종류의 명시적이거나 암묵적인 보증 없이 "있는 그대로" 제공됩니다. 어떤 경우에도 저작자나 저작권 보유자는 소프트웨어와 소프트웨어의 사용 또는 기타 취급에서 비롯되거나 그에 기인하거나 그와 관련하여 발생하는 계약 이행 또는 불법 행위 등에 관한 배상 청구, 피해 또는 기타 채무에 대해 책임지지 않습니다.
--END--
