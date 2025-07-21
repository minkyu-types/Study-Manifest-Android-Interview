# Week3. Q10 ~ Q16

## Q) 10. BroadcastReceiver란 무엇인가요?
안드로이드 운영 체제 전체의 브로드캐스트 메시지나  
앱 특정 브로드캐스트를 수신하고 응답할 수 있도록 하는 컴포넌트입니다.  
배터리 상태 변경, 네트워크 연결 혹은 커스텀 인텐트와 같은 다양한 이벤트를 알립니다.  

동적인 시스템 또는 앱 수준 이벤트에 반응하는 응답성 있는 애플리케이션을 구축하는데 유용합니다.  

### BroadcastReceiver의 목적
Activity나 Service의 생명주기에 직접적으로 연결되지 않을 수 있는 이벤트를 처리하는 데 사용됩니다.  
백그라운드에서 계속 실행되지 않고도 변경 사항에 반응할 수 있도록 하는 메시징 시스템 역할을 하여 리소스를 절약합니다.  

### 브로드캐스트 유형
1. 시스템 브로드캐스트  
  안드로이드 운영 체제에서 배터리 잔량 변경, 시간대 업데이트 또는 네트워크 연결 변경과 같은 시스템 이벤트를 앱에 알리기 위해 보냅니다
2. 커스텀  
  앱 내부 또는 앱간에 특정 정보나 이벤트를 전달하기 위해 보냅니다.

### 커스텀 BroadcastReceiver 선언하기
BroadcastReceiver를 생성하려면 BroadcastReceiver 클래스를 상속받고  
브로드캐스트 처리 로직을 정의하는 onReceiver 메서드를 재정의해야 합니다.  

```Kotlin
class MyBroadcastReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        val action = intent.action
        if (action == Intent.ACTION_BATTERY_LOW) {
            // 배터리 부족 이벤트 처리
        }
    }
}
```

### 커스텀 BroadcastReceiver 등록하기
BroadcastReceiver는 두 가지 방법으로 등록할 수 있습니다.
1. 매니페스트 파일을 통한 정적 등록 
  앱이 실행 중이지 않을 때도 처리해야 하는 이벤트에 사용합니다.  
  AndroidManifest.xml 파일에 intent-filter를 추가합니다.  
```xml
<receiver android:name=".MyBroadcastReceiver"  
          android:exported="false">
    <intent-filter>
        <action android:name="android.intent.action.BATTERY_LOW"/>
    </intent-filter>
</receiver>
````

2. 코드를 통한 동적 등록  
  앱이 활성 상태이거나 특정 상태일 때만 처리해야 하는 이벤트에 사용됩니다.

```kotlin
val receiver = MyBroadcastReceiver()
val intentFilter = IntentFilter(Intent.ACTION_BATTREY_LOW)

val flags = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
    Context.RECEIVER_NOT_EXPORTED
} else {
    0 // 플래그 없음
}
registerReceiver(receiver, intentFilter, flags)

override fun onDestroy() {
    unregisterReceiver(receiver)
    super.onDestory()
}
```

### 유의 사항
- 생명주기 관리 : 동적 등록을 하는 경우 메모리 누수를 방지하기 위해 unregisterReceiver를 사용하여 리시버 등록 해제해야 합니다.
- 백그라운드 실행 제한 : 안드로이드 8.0부터 백그라운드 앱은 Implicit broadcast exceptions를 제외하고 브로드캐스트 수신에 제한을 받습니다.  
  Context.registerReceiver 또는 JobSchedular를 사용해야 합니다.
- 보안 : 민감한 정보가 포함된 브로드캐스트는 무단 접근을 방지하기 위해 권한으로 보호해야 합니다.

### BroadcastReceiver 사용 사례
- 네트워크 연결 변동 모니터링
- SMS 또는 통화 이벤트에 응답
- 충전 상태와 같은 시스템 이벤트에 대한 UI 업데이트
- 커스텀 브로드캐스트로 작업 또는 알람 예약

### 요약
브로드캐스트는 OS 시스템과 상호 작용하는 반응형 애플리케이션을 구축하는 데 필수적인 컴포넌트 입니다.  
시스템 또는 앱 이벤트를 효율적으로 수신하고 응답할 수 있도록 합니다.  

### 실전 질문
### 브로드캐스트의 유형에는 어떤 것이 있으며, 기능 및 사용 측면에서 시스템 브로드캐스트와 커스텀 브로드캐스트는 어떤 차이가 있나요?
시스템 브로드캐스트는 운영체제에서 시스템 차원의 이벤트가 발생할 때 자동으로 발송되는 브로드캐스트이고  
커스텀 브로드캐스트는 개발자가 직접 정의하고 앱 내부, 앱 간에 특정 이벤트나 데이터를 전달하기 위해 사용하는 브로드캐스트  

시스템 브로드캐스트는 앱이 직접 발생시키지 않아도 OS가 알아서 이벤트를 앱에 알려줍니다.  
예) 배터리 부족, 네트워크 연결 상태 변경, 기기 부팅 완료

커스텀 브로드캐스트는 앱 내부에서 특정 상황에 맞춰 직접 브로드캐스트를 발송합니다.  
예) com.exmaple.ACTION_SYNC_FINISHED와 같이 개발자가 임의로 액션을 정의, 여러 컴포넌트간의 느슨한 결합을 위해 사용  
앱 내부에서 동적으로 등록해서 사용하며, 외부 노출이 필요한 경우 권한 설정 및 보안에 신경 써야 합니다.

## Q) 11. ContentProvider의 목적은 무엇이며, 애플리케이션 간의 안전한 공유를 어떻게 용이하게 하나요?
ContentProvider는 구조화된 데이터에 대한 접근을 관리하고 애플리케이션 간 데이터 공유를 위한 인터페이스를 제공하는 컴포넌트  
다른 앱과의 중앙 저장소 역할을 하여 앱 간의 안전하고 일관된 데이터 공유를 보장합니다.  

여러 앱이 동일한 데이터에 접근해야 하거나, 데이터베이스 또는 내부 저장소 구조를 노출하지 않고 다른 앱에 데이터를 제공하는 경우 특히 유용합니다.  

### ContentProvider의 목적
데이터 접근 로직을 캡슐화하여 앱 간 데이터 공유를 더 쉽고 안전하게 만드는 것입니다.  
SQLite, 파일 시스템, 네트워크 기반 데이터같은 기본 데이터 소스를 추상화하고 데이터와 상호작용 하기 위한 통합 인터페이스를 제공합니다.  

### ContentProvider의 주요 구성 요소
데이터 접근 주소로 URI를 사용합니다. URI는 다음으로 구성됩니다.  
1. 권한 : ContentProvider를 식별합니다 (특정 앱의 provider)
2. Path : 데이터 유형을 지정합니다 (/users, /products)
3. ID (optional) : 데이터 셋 내의 특정 항목을 참조합니다.

### ContentProvider 구현하기
ContentProvider를 상속받고 다음 메서드를 구현해야 합니다.
- onCreate() : 초기화
- query() : 데이터를 검색
- insert() : 새 데이터를 추가
- update() : 기존 데이터를 수정합니다.
- delete() : 데이터를 제거합니다.
- getType() : 데이터의 MIME 유형을 반환합니다.

### ContentProvider 등록하기
다른 앱에서 ContentProvider에 접근할 수 있도록 하려면  
AndroidManifest.xml 파일에 선언해야 합니다. authority 속성은
ContentProvider를 고유하게 식별합니다

```xml
<provider
  android:name=".MyContentProvider"
  android:authorities="com.example.myapp.provider"
  android:exported="true"
  android:grantUriPermissions="true" />
```

### ContentProvider에서 데이터 접근하기
다른 앱에서 ContentProvider와 상호 작용하려면 ContentResolver 클래스를 사용해야 합니다.  
ContentResolver는 데이터를 쿼리, 삽입, 업데이트 또는 삭제하는 메서드를 제공합니다.  

```kotlin
val contentResolver = context.contentResolver

// 데이터 쿼리
val cursor = contentResolver.query(
  Uri.parse("content://com.example.myapp.provider/users"),
  null,
  null,
  null,
  null
)

// 데이터 삽입
val values = ContentValues().apply {
  put("name", "John Doe")
  put("email", "johndoe@example.com")
}
contentResolver.insert(Uri.parse("content://com.example.myapp.providers/users"), values)
```

### ContentProvider 사용 사례
- 다른 애플리케이션 간 데이터 공유
- 앱 시작 시 컴포넌트 또는 리소스 초기화
- 연락처, 미디어 파일 또는 앱별 데이터와 같은 구조화된 데이터에 대한 접근 제공
- 연락처 앱이나 파일 선택기와 같은 안드로이드 시스템 기능과의 통합 활성화
- 세분화된 보안 제어를 통한 데이터 접근 허용

### 실전 질문
### Q) ContentProvider URI의 주요 구성 요소는 무엇이며, ContentResolver는 데이터 쿼리하거나 수정하기위해 어떻게 상호 작용하나요?
1. 권한 : ContentProvider를 식별합니다 (특정 앱의 provider)
2. Path : 데이터 유형을 지정합니다 (/users, /products)
3. ID (optional) : 데이터 셋 내의 특정 항목을 참조합니다.

ContentResolver는 앱에서 ContentProvider에 접근할 때 사용하는 안드로이드 시스템 제공 클래스입니다.  
ContentResolver를 통해 쿼리, 삽입, 수정, 삭제 등 다양한 데이터 조작이 가능합니다.
- query()
- insert()
- update()
- delete()

### Pro Tips for Mastery : 앱 시작 시 리소스나 초기 셋업을 위해 ContentProvider를 사용하는 사례
일반적으로 리소스나 라이브러리 초기화는 Application 클래스에서 하지만  
관심사를 더 잘 분리하기 위해 초기화 관련 로직을 별도의 ContentProvider에 캡슐화할 수 있습니다.  
커스텀 ContentProvider를 생성하고 AndroidManifest.xml에 등록하면 초기화 작업을 효율적으로 위임할 수 있습니다.  

ContentProvider의 onCreate() 메서드는 Application.onCreate() 메서드보다 먼저 호출되므로  
초기화를 위한 훌륭한 진입점 역할을 합니다.  
Firebase 초기화 작업같은것도 충분히 ContentProvider에서 할 수 있습니다.  

## Q) 12. 구성 변경(configuration changes)을 어떻게 처리하나요?
구성 변경을 올바르게 처리하는 것은 화면 회전, 언어 변경, 다크/라이트 모드 전환이 되었을 때  
원활한 사용자 경험을 유지하는 데 중요한 역할을 합니다.  
기본적으로 안드로이드 시스템은 구성 변경이 발생할 때 Activity를 다시 시작하며  
이로 인해 일시적으로 UI의 상태가 손실될 수 있습니다.  
따라서 구성 변경에 적절하게 대응하려면 아래와 같은 전략을 고려해 볼 수 있습니다.

1. UI 상태 저장 및 복원  
  onSaveInstanceState() 및 onRestoreInstanceState() 메서드를 구현하여  
  Activity 재생성 중 UI 상태를 보존하고 복원합니다.
2. Jetpack ViewModel  
  ViewModel 클래스를 활용하여 구성 변경에도 유지되어야 하는 UI 관련 데이터를 저장합니다.  
  ViewModel 재시작 범위를 넘어서 존재하도록 설계되었으므로  
  구성 변경이 발생했을 때 데이터를 보존하고 다시 복원하는 데 이상적입니다.
3. 구성 변경 수동으로 처리하기  
  애플리케이션이 특정 구성 변경중에 리소스를 업데이트할 필요가 없고 Activity 재시작을 피하고 싶다면  
  AndroidManifest.xml에 Activity가 처리할 구성 변경 사항을 android:configChanges 속성을 사용하여 선언할 수 있습니다.
4. Jetpack Compose에서 rememberSaveable 활용  
  rememberSaveable을 사용하여 구성 변경이 발생했을 때 UI 상태를 보존할 수 있습니다.

### 추가 팁
- 내비게이션 및 백 스택 보존  
  Navigation 컴포넌트를 사용하면 구성 변경 시 내비게이션 백 스택이 보존됩니다 
- 앱 구성에 의존적인 데이터 피하기  
  앱 구성에 의존적인 값을 UI 레이어에 직접 저장하지 않는 것이 좋습니다.  
  ViewModel과 같은 대안을 고려할 수 있습니다.

### 실전 질문
### Q) 구성 변경을 처리하기 위한 전략과 ViewModel은 데이터를 어떻게 보존하는가
onSaveInstanceState(), onRestoreInstaceState() 메서드를 구현  
구성 변경 수동으로 처리하기, rememberSaveable 활용  
ViewModel은 Activity보다 생명주기가 길기 때문에 구성 변경이 발생했을 때 데이터를 보존하고 다시 복원

### Q) Manifest 파일에서 android:config 속성은 Activity 생명주기와 동작에 어떤 영향을 미치는가
기본적으로 안드로이드는 구성 변경이 발생하면 현재 Activity를 종료하고 새로 생성합니다.  
android:configChanges 속성을 AndroidManifest.xml의 activity 태그에 지정하면  
특정 구성 변경이 발생해도 시스템이 Activity를 재시작 하지 않고 대신 onConfigurationChanged() 메서드만 호출됩니다  
```xml
<activity
  android:name=".MyActivity"
  android:configChanges="orientation|screenSize|keyboardHidden" />

````
언제 onConfigurationChanged()를 직접 사용하는게 좋을까
- Activity 재시작이 성능상 부담이 되거나, 복잡한 상태 복원이 필요 없는 경우
- 특정 구성 변경에 대해 UI 일부만 동적으로 갱신하면 충분한 경우
- 실시간 미디어 재생, 카메라 미리보기, 게임 등에서 Activity 재시작이 끊김을 유발하는 경우

- 동영상/음악 플레이어 앱
  - 동영상 재생 중 화면 회전이 발생할 때 Activity가 재시작되면 재생이 끊기고 사용자 경험이 나빠집니다
  - 이 경우 android:configChanges="orientation|screenSize"를 선언하고 onConfigurationChanged()에서 레이어웃만  
    동적으로 조정하면 재생이 끊기지 않고 부드럽게 화면 전환이 가능합니다.

## Q) 13. 안드로이드는 메모리를 어떻게 효율적으로 관리하며, 메모리 누수를 어떻게 방지하는지 설명해주세요
안드로이드는 사용되지 않는 메모리를 자동으로 회수하여 활성 중인 애플리케이션 및 서비스에게  
효율적인 메모리 할당을 보장하는 **가비지 컬렉션** 메커니즘을 통해 메모리를 관리합니다.  
관리형 메모리 환경에 의존하므로 수동으로 할당하고 해제할 필요가 없습니다.  
메모리량을 모니터링하고 더 이상 참조되지 않는 객체를 정리하여  
과도한 메모리 소비를 방지합니다.  

https://developer.android.com/guide/components/activities/process-lifecycle?hl=ko  
시스템 메모리가 부족할 때 포그라운드 애플리케이션의 원활한 작동을 우선시하며  
백그라운드 프로세스를 종료하기 위해 low-memory killer를 사용합니다  

### 안드로이드에서 메모리 누수의 원인
애플리케이션이 더 이상 필요하지 않은 객체에 대한 참조를 유지하여 가비지 컬렉터가 메모리를 회수하지 못하게 할 때 발생합니다.  
일반적인 원인으로는 부적절한 생명주기 관리, 정적 참조 또는 Context에 대한 장기 참조 유지 등이 원인입니다.  

### 메모리 누수를 피하기 위한 모범 사례
1. 생명주기를 인지하는 컴포넌트 사용  
  ViewModel, collectAsStateWithLifecycle, Flow, LiveData  
2. Context에 대한 오랜 참조 피하기  
  정적 필드나 싱글턴과 같은 오래 지속되는 객체에서 Context에 대한 참조를 유지하지 않아야 합니다
3. 리스너 및 콜백 등록 올바르게 해제하기
4. 중요하지 않은 객체는 WeakReference 사용하기
5. 누수 감지 툴 사용
6. View에 대한 정적 참조 피하기
7. 리소스 닫기
8. Fragment와 Activity 현명하게 사용하기

## Q) 14. ANR이란 무엇인지, ANR이 발생하는 주요 원인은 무엇이며, 어떻게 예방할 수 있는지 설명해주세요
ANR(Application Not Responding)은 앱의 메인 스레드가 너무 오랫동안, 통상적으로 5초 이상 차단될 때 발생하는  
안드로이드 시스템 오류입니다  
ANR이 발생하면 안드로이드는 사용자에게 앱을 닫거나 응답을 기다리도록 안내합니다.

### ANR이 발생하는 원인
- 메인 스레드에서 5초 이상 걸리는 무거운 작업
- 장시간 실행되는 네트워크 또는 데이터베이스 등의 I/O 작업
- UI 스레드 차단 작업(UI 스레드에서의 동기 작업)

### ANR 예방 방법
ANR을 예방하려면 무겁거나 시간이 많이 걸리는 작업을 오프로드하여 메인 스레드의 응답성을 유지하는 것이 중요합니다
1. 무거운 작업을 메인 스레드 밖으로 이동  
  I/O, 네트워크 요청, 데이터베이스 쿼리와 같은 무거운 작업을 백그라운드 스레드로(Coroutines)
2. WorkManager 사용하기
  백그라운드에서 실행되어야 하는 장기적인 작업은 WorkManager를 사용, WorkManager는 개발자가 필요한 작업을  
  사전에 스케쥴링하고, 메인 스레드 외부에서 실행되도록 보장
3. 데이터 불러오기 최적화(Paging)
4. 구성 변경 시 UI 작업 최소화  
  ViewModel을 활용하여 작업을 분리하고 UI 관련 데이터를 유지
5. Android Studio로 모니터링 및 프로파일링
6. 블로킹 호출 피하기
7. 가벼운 지연 작업에 Handler 사용

### 실전 질문
### ANR을 진단하고 앱 성능과 유저 경험을 개선해본적이 있는가?
데이터베이스 쿼리 작업을 메인스레드에서 진행해서 앱이 중단된 적이 있다

## Q) 15. 딥 링크를 어떻게 처리하는지 설멍해주세요
딥 링크는 사용자가 URL이나 알림과 같은 외부 소스에서 앱 내의 특정 화면이나 기능으로 직접 이동할 수 있도록 합니다.  
AndroidManifest.xml에서 이를 정의하고 해당 Activity나 Fragment에 들어오는 Intent를 처리해야 합니다.

### 1. 매니페스트에서 딥 링크 정의하기
딥 링크를 활성화 하려면 딥 링크를 처리해야 하는 Activity에 대해 AndroidManifest.xml에서 intent filter를 선언해야 합니다  
```xml
<activity android:name=".MyDeepLinkActivity
    android:exported="true">
  <intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <!-- 딥 링크 URI 정의 -->
    <data
      android:scheme="https"
      android:host="example.com"
      android:pathPrefix="/deepLink" />
  </intent-filter>
</activity>
```
- android:scheme: URL 스키마(가령, https)를 지정합니다.
- android:host: 도메인(example.com)을 지정합니다.
- android:pathPrefix: URL의 경로(가령, /deepLink)를 정의합니다.

이 설정을 통해 https://example.com/deepLink와 같은 URL이 MyDeepLinkActivity를 열도록 허락합니다.

### 2. Activity에서 딥 링크 처리하기
Activity 내부에서 들어오는 Intent 데이터를 검색하고 처리하여 적절한 화면으로 이동하거나 작업을 수행합니다.

```kotlin
class MyDeepLinkActivity : AppCompatActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_my_deep_link)

    // Intent 가져오기
    val intentData: Uri? = intent?.data
    if(intentData != null) {
      val id = intentData.getQueryParameter("id")
      // 특정 화면으로 이동
    }
  }
}
```

### 3. 딥링크 테스트하기
abd 명령어를 사용할 수 있습니다.
```
adb shell am start ‑a android.intent.action.VIEW \
‑d "https://example.com/deepLink?id=123" \
com.example.myapp # 앱 패키지 이름
```

### 추가 고려 사항
- 커스텀 스키마  
  앱 내부적으로 실행하는 경우 딥링크에 대한 커스텀 스키마를 사용할 수 있지만 경우에 따라 더 넓은 호환성을 위해 HTTP URL을 선호
- 내비게이션
  딥 링크 데이터를 기반으로 앱 내의 다른 Activity나 Fragment로 이동하기 위해 Intent를 사용합니다.
- 폴백 처리
  앱이 딥 링크 데이터가 유효하지 않거나 불완전한 경우를 처리하여 더 나은 사용자 경험을 제공할 수 있음
- App Links
  딥 링크가 브라우저 대신 앱에서 직접 열리도록 하려면 App Links를 설정해야 합니다.

### 실전 질문
### 딥 링크를 어떻게 테스트하고 디버깅 기법이 있나요?
abd 명령어 사용, 실제 기기에서 URL 클릭  
로그 활용, 인텐트 필터 매칭

## Q) 16. 태스크와 백 스택이란 무엇인가요?
- 태스트 : 사용자가 특정 목적을 달성하기 위해 사용하는 Activity의 집합
- 백스택 : 후입선출 구조로 이뤄져있음

### 태스트
태스크는 일반적으로 런처나 Intent를 통해 Activity가 실행될 때 시작됩니다  
태스크는 Intent및 Activity 런치 모드가 어떻게 구성되었는지에 따라 여러 애플리케이션과 해당 Activity에 속해있을 수 있습니다  
태스크는 연관된 Activity가 소멸될 때가지 활 상태를 유지합니다  

### 백 스택
태스크 내의 Activity의 기록을 유지됩니다  
말 그대로 액티비티를 저장해놓는 스택  
직관적인 탐색과 사용자 워크플로우의 연속성을 보장함

태스크와 백스택은 Activity 런치 모드와 인텐트 플래그의 영향을 받습니다.  
런치 모드와 Intent 플래그는 태스크 및 백 스택 내 Activity의 동작을 제어하는 데 사용되는 매커니즘 입니다.  

런치 모드는 Activity가 어떻게 인스턴스화되고 백 스택에서 처리되는 지를 결정합니다

1. standard : 기본 런치 모드, 인스턴스가 존재하더라도 Activity가 시작될 때마다 새 인스턴스가 생성되어 백스택에 추가 
2. singleTop : Activity의 인스턴스가 이미 백 스택의 맨 위에 있는 경우 새 인스턴스가 생성되지 않습니다  
  onNewIntent()에서 Intent를 처리합니다.
3. singleTask : 태스크 내의 Activity의 인스턴스가 하나만 존재, onNewIntent()가 호출됨
4. singleInstance : singleTask와 유사하지만, Activity가 다른 Activity와 분리된 고유한 태스크에 배치됩니다.

### 인텐트 플래그
Intent가 전송될 때 Activity가 시작되는 방식이나 백스택의 동작을 수정하는 데 사용됩니다
- FLAG_ACTIVITY_NEW_TASK  
  태스크가 이미 존재하는 경우 해당 태스크를 맨 앞으로 가져옵니다
- FLAG_ACTIVITY_CLEAR_TOP  
  Activity가 이미 백 스택에 있는 경우, 그 위에 있는 모든 Activity가 날아가고 기존 인스턴스가 Intent를 처리
- FLAG_ACTIVITY_SINGLE_TOP  
  Activity가 백 스택의 맨 위에 있는 경우 새 인스턴스가 생성되지 않도록 보장합니다
- FLAG_ACTIVITY_NO_HISTORY  
  Activity가 백 스택에 추가되는 것을 방지하여 종료 후에도 유지되지 않도록 합니다

### 사용 사례
- 런치 모드 : AndroidManifest.xml 에 설정할 수 있으며 Activity의 기본 동작을 설정할 수 있도록 함
- 인텐트 플래그 : Intent를 생성할 때 개발자가 유동적으로 플래그를 설정할 수 있는 방식, 유연성을 제공