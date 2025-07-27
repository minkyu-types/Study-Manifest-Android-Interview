## 📚 Q17. Bundle의 사용 목적에 대해서 설명해주세요

### 🎯 개요
Bundle은 Activity, Fragment, Service와 같은 **컴포넌트 간에** 데이터를 전달하는 데 사용되는 **키-값 쌍 데이터 구조**  
작은 용량의 데이터를 효율적으로 전송되는 데 사용됩니다.  
가볍고 쉽게 관리하고 전송할 수 있는 형식으로 직렬화하도록 설계되었습니다.  

### 🛠️ 사용 사례  
1. **Activity간 데이터 전달** : Intent에 Bundle을 담아서 전달  
2. **Fragment간 데이터 전달** : setArguments() 및 getArguments()와 함께 전달되어서 Fragment 간에 데이터 전달  
3. **인스턴스 상태 및 복원** : Bundle은 onSaveInstanceState() 및 onRestoreInstasnceState()와 같은 생명주기 메서드에서 상태를 저장하고 복원하는 데 사용  
4. **Service에 데이터 전달** : Service를 시작하거나 바인딩된 Service에 데이터를 전달할 때 Bundle을 통해 데이터를 운반할 수 있음

### ⚙️ 작동 방식  
키-값 구조로 직렬화하여 작동합니다.  
키는 문자열이며 값은 기본 유형, Serializable, Parcelable 객체 또는 다른 Bundle일 수 있습니다.  
이를 통해 데이터를 효율적으로 저장하고 전송할 수 있습니다.  

내부적으로 mMap 이라는 ArrayMap<String, Object>에 저장함
intent.putExtra()에서는 새 번들을 만들어서 사용중
getExtra()는 번들을 반환

```kotlin
val intent = Intent(this, ActivityB::class.java).apply {
    putExtra("user_name,"John Doe")
    putExtra("user_age", 25)
}
startActivity(intent)

val name = intent.getStringExtra("user_name") // not-null?
val age = intent.getIntExtra("user_age",-1)
```

```kotlin
val fragment = MyFragment().apply {
    arguments = Bundle().apply {
        putString("user_name, "Jane Doe")
        putInt("user_age", 30)
    }
}

val name = arguments?.getString("user_name")
val age = arguments?.getInt("user_age")
```

```kotlin
override fun onSaveInstanceState(outState: Bundle) {
    super.onSaveInstanceState(outState)
    outState.putString("user_input", editText.text.toString())
}

override fun onRestoreInstanceState(savedInstanceState: Bundle) {
    super.onRestoreInstanceState(savedInstanceState)
    val userInput = savedInstanceState.getString("user_input")
    editText.setText(userInput)
}
```

### 🧠 요약
Bundle은 컴포넌트 및 생명주기 이벤트 간에 데이터를 **효율적으로 전달하고 보존하기 위한 안드로이드의 중요한 구성 요소입니다**.  
가볍고 유연한 구조로 인해 애플리케이션 **상태 및 데이터 전송 관리에 필수적인 도구입니다**.

### ❓ 실전 질문 
### 구성 변경 중 onSaveInstanceState()는 UI 상태를 보존하기 위해 Bundle을 어떻게 활용하며, Bundle에 어떤 유형의 데이터를 담을 수 있나요?
- Activity나 Fragment에서 onSaveInstanceState 메서드를 오버라이드하여, UI의 현재 상태를 outState에 저장합니다
- 시스템이 구성 변경으로 인해 Activity/Fragment를 재생성할 때, 저장된 Bundle이 onCreate() 또는 onRestoreInstanceState()로 전달되어 이전 상태를 복원할 수 있습니다.

Bundle은 기본 데이터 타입과 String, Serializable, Parcelable을 지원합니다.

## 📚 Q18. Activity 또는 Fragment 간에 데이터를 어떻게 전달하나요?

### 🎯 Activity 간 데이터 전달
일반적인 매커니즘은 Intent입니다.  
데이터는 키-값 쌍의 형태로 Intent에 추가되고, 수신하는 Activity는 getIntent()를 사용하여 가져옵니다.

```kotlin
val intent = Intent().apply {
    putExtra("USER_NAME", "John Doe")
}

class SecondActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // view 설정

        val userName = intent.getStringExtra("USER_NAME")
    }
}
```

### 🎯 Fragment 간 데이터 전달

```kotlin
val fragment = MyFragment().apply {
    arguments = Bundle().apply {
        putString("user_name, "Jane Doe")
        putInt("user_age", 30)
    }
}

val name = arguments?.getString("user_name")
val age = arguments?.getInt("user_age")
```

### 🎯 Jetpack Navigation 라이브러리로 Fragment 간 데이터 전달
Jetpack Navigation 라이브러리가 지원하는 Safe Args 플러그인을 사용하면  
대상 간 타입-세이프 네비게이션과 인수 전달을 가능하게 하는 direction 및 argument 클래스가  
컴파일 타임에 자동적으로 생성되어, 런타임 시 값을 안전하게 가져올 수 있습니다.

1. 네비게이션 그래프에서 인수 정의하기  
nav_graph.xml 에서 username이란 String 타입의 인수를 정의한다.  
```xml
<fragment
    android:id="@+id/secondFragment"
    android:name="com.example.SecondFragment"
    <argument
        android:name="username"
        app:argType="string" />
</fragment>
```

2. 소스 프래그먼트에서 데이터 전달하기  
Safe Args 플러그인은 컴파일 타임에 대상 객체와 빌더 클래스를 생성하고  
아래 예시와 같이 인수를 안전하고 명시적으로 전달할 수 있도록 한다  
```kotlin
val action = FirstFragmentDirections
    .actionFirstFragmentToSecondFragment(username = "skydoves")
.findNavController().navigate(action)
```

navigate에서 Directions 객체를 받는다
![navDirection](/hegunhee/images/nav_direction.png)  
3. 대상 프래그먼트에서 데이터 검색하기  
아래 코드와 같이 전달된 인수를 통해 데이터를 가져올 수 있음  
```kotlin
val username = arguments?.let {
    SecondFragmentArgs.fromBundle(it).username  
}
```

Safe Args를 사용하여 정의된 인수를 **컴파일 타임**에 검사하여 정적인 코드로 만들고  
런타임에 해당 인수 값을 안전하게 가져옴으로써 **런타임 오류**를 줄일 수 있습니다.  
클래스 및 메서드를 생성해 주기 때문에, Fragment 간의 데이터 핸들링 관련 코드의 가독성을 향상시킬 수 있습니다.  

### 🎯 Shared ViewModel 사용하기
동일한 Activity 내에서 Fragment 끼리 통신해야 하는 경우 **Shared ViewModel**을 고려해 볼 수 있습니다.  
동잃한 Activity 내의 여러 Fragment 간에 공유되는 ViewModel 인스턴스를 의미합니다.  
activityViewModels() 메서드를 사용하여 구현할 수 있으며  
해당 메서드는 ViewModel의 범위를 Activity로 지정하여 Fragment간 동일한 ViewModel 인스턴스에 접근하고  
공유할 수 있도록 합니다.  
이 방법을 통해 Fragment끼리 의존성이 생기는 것을 피하고  
각 Fragment의 생명주기에 따라 안전하게 데이터를 수신할 수 있습니다.  

```kotlin
class SharedViewModel : ViewModel() {
    private val _userData = MutableStateFlow<User?>(null)
    val userData: StateFlow<User?> = _userData.asStateFlow()

    fun setUserData(user: User) {
        _userData.value = user
    }
}

// Fragment A (데이터를 전송하는 Fragment)
class FirstFragment : Fragment() {
    private val sharedViewModel: SharedViewModel by activityViewModels()

    fun updateUser(user: User) {
        sharedViewModel.setUserData(user)
    }
}

// Fragment B (데이터를 수신하는 Fragment)
class SecondFragment : Fragment() {
    private val sharedViewModel: SharedViewModel by activityViewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        lifecycleScope.launch {
            viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.RESUMED) {
                sharedViewModel.userData.collectLatest { user ->
                    // 사용자 데이터 처리
                }
            }
        }
    }
}

// Activity (Activity에서 데이터 수신하기)
class MainActivity : ComponentActivity() {
    private val sharedViewModel: SharedViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        lifecycleScope.launch {
            lifecycle.repeatOnLifecycle(Lifecycle.State.RESUMED) {
                sharedViewModel.userData.collectLatest { user ->
                    // 사용자 데이터 처리
                }
            }
        }
    }
}
```
### ❓ 실전 질문 
### 동일한 Activity 내에서 Fragment 간 데이터를 주고받을 때 어떤 방법이 효과적인가요?

직접적인 Fragment 트랜잭션을 사용한다면 강한 결합도 문제가 발생합니다.

#### Bundle 사용이 적합한 경우:
- 단순한 일회성 데이터 전달
- Fragment 간 복잡한 의존성이 없는 경우
- 작은 크기의 데이터

#### ViewModel 사용이 적합한 경우:
- 지속적인 데이터 공유가 필요한 경우
- 여러 Fragment에서 동일한 데이터를 관찰해야 하는 경우
- 복잡한 UI 상태 관리
- 구성 변경 시 데이터 보존이 필요한 경우

### 💡 Pro Tips for Mastery: Fragment Result API
상황에 따라 Fragment에서 다른 Fragment 혹은 Activity 간에 **일회성으로 값**을 전달해야 합니다  
Fragment 버전 1.3.0 이상에서는 각 FragmentManager가 FragmentResultOwner를 구현하여  
Fragment가 서로 직접 참조할 필요 없이 결과 리스너를 통해 통신할 수 있습니다.  
데이터 전달을 **단순화하면서 느슨한 결합**을 유지합니다.  

Fragment B(송신자) 에서 Fragment A(수신자) 로 데이터를 전달하기 위해  
아래와 같은 방법을 사용할 수 있습니다. 

1. Fragment A(결과를 받는 프래그먼트)에서 결과 리스너 설정하기
2. Fragment B에서 동일한 requestKey를 사용하여 결과 전달하기  

#### 🛠️ Fragment A에서 결과 리스너 설정하기  
Fragment A는 setFragmentResultListener()를 사용하여 리스너를 등록해야 하며  
**STARTED 상태가 될 때 결과를 받도록 보장합니다.  **

```kotlin
class FragmentA : Fragment() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
    
    // 데이터를 수신하기 위해 리스너 등록
    parentFragmentManager.setFragmentResultListener("requestKey") { requestKey, bundle ->
        val result = bundle.getString("bundleKey")
        // 수신한 결과 값 처리
        }
    }
}
```
고유한 키 값을 사용하여 리스너를 등록  
모든 콜백은 Fragment가 STARTED 상태에 들어갈 때 실행됩니다.  

#### 🛠️ Fragment B에서 결과 보내기  
Fragment B는 setFragmentResult()를 사용하여 결과를 전달할 수 있으며  
Fragment A가 활성화될 때 데이터를 받아볼 수 있도록 보장합니다  

```kotlin
class FragmentB : Fragment() {

    private lateinit var button: Button

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        button = view.findViewById(R.id.button)
        button.setOnClickListener {
            val result = "result"
            // 값을 Fragment A로 전달
            parentFragmentManager.setFragmentResult("requestKey", bundleOf("bundleKey" to result))
        }
    }
}
```
고유한 키 값을 사용하여 FragmentManager에 전달할 값을 저장합니다  
Fragment A가 활성 상태가 아니라면, Fragment A가 재개되고 리스너를 등록할 때까지  
값은 계속 저장되어 있습니다.  

#### ⚙️ 프래그먼트 결과의 동작  
- **키-리스너의 1:1 관계** : 각 키는 한 번에 **하나의 리스너**와 **하나의 결과**만 가질 수 있습니다  
- **보류 중인 결과는 덮어쓰여짐**: 리스너가 활성화되기 전에 여러 결과가 설정되면 **최신 결과**만 저장됨  
- **결과는 소비 후 삭제됨** : Fragment가 결과를 수신하고 처리하면 결과는 FragmentManager에서 제거됩니다  
- **백 스택의 프래그먼트는 결과를 받지 못함** : 결과를 받으려면 **백 스택에서 팝되고 STARTED 상태**여야 합니다
- **STARTED 상태의 리스너는 즉시 트리거됨**: 수신자에서 결과를 설정할 때 수신자가 이미 활성 상태면 리스너는 **즉시 실행**됩니다.  

## 📚 Q19. 화면 회전과 같은 구성 변경이 발생하면 Activity에 어떤 변화가 생기나요?

### 🎯 개요
시스템은 새 구성을 적용하기 위해 현재 Activity를 종료하고 다시 실행하게 됩니다  
이러한 동작은 앱의 리소스가 변경된 구성을 새롭게 반영하고 앱이 다시 로드되도록 보장합니다  

### 🛠️ 구성 변경 중 기본 동작  
1. Activity 종료 및 재시작  
  구성 변경이 발생하면 Activity가 종료된 다음 다시 시작됩니다  
    - 시스템은 현재 실행중인 Activity의 onPause(), onStop(), onDestroy() 메서드를 순차적으로 호출합니다.  
    - 구성을 변경하면 Activity는 다시 시작되고, onCreate() 메서드가 호출됩니다.  
2. 리소스 다시 로드하기  
  시스템은 새 구성에 따라 리소스를 다시 로드하여 앱이 화면방향, 테마 또는 언어와 같은 변경사항이 반영될 수 있도록 합니다  
3. 데이터 손실 방지  
  개발자는 재생성 중 데이터 손실을 방지하기 위해 onSaveInstanceState() 및 onRestoreInstanceState()  
  메서드를 사용하거나 ViewModel을 활용하여 인스턴스 상태를 저장하고 복원할 수 있습니다  

### 🛠️ 재생성을 유발하는 구성 변경
1. 화면 회전
2. 다크/라이트 테마 변경
3. 글꼴 크기 변경
4. 언어 변경

### 🛠️ Activity 재생성 피하기
Activity를 다시 시작하지 않고 구성 변경을 처리하려면 매니페스트 파일에서 android:configChanges 속성을 추가하면 됩니다.  
이 방식은 변경 사항을 개발자가 수동적으로 처리하는 형태로 책임을 개발자에게 위임합니다.  

```xml
<activity
    android:name=".MainActivity"
    android:configChanges="orientation￨screenSize￨keyboardHidden" />
```
이렇게 되면 시스템은 Activity를 소멸시키지 않고 다시 생성하지 않습니다.  
대신 onConfigurationChanged() 메서드가 호출되어 개발자가 변경 사항을 수동으로 처리할 수 있게 됩니다.
```kotlin
override fun onConfigurationChanged(newConfig: Configuration) {
    super.onConfigurationChanged(newConfig)

    if (newConfig.orientation == Configuration.ORIENTATION_LANDSCAPE) {
        // 화면이 가로로 변경된 경우 로직 처리
    } else if (newConfig.orientation == Configuration.ORIENTATION_PORTRAIT) {
        // 화면이 세로로 변경된 경우 로직 처리
    }
}
```

### 🧠 요약
구성 변경이 발생하면 기본 동작은 Activity를 소멸시키고 다시 생성하여 리소스를 다시 로드하는것입니다.  
onSaveInstance()나 ViewModel을 사용하여 일시적으로 UI 상태를 보존할 수 있습니다.  
매니페스트 파일에서 android:configChanges 속성을 추가하면  
소멸-재생성 과정을 생략하고 수동적으로 구성 변경에 대한 대응을 할 수 있습니다.  


## 📚 Q20. ActivityManager란 무엇인가요?

### 🎯 개요
ActivityManager는 기기에서 실행 중인 Activity, 태스크, 프로세스에 대한 정보를 제공하고  
관리하는 안드로이드 시스템 서비스 입니다  
안드로이드 프레임워크의 일부로, 개발자가 앱 생명주기, 메모리 사용량 및 태스크 관리 측면과 상호 작용하고  
제어할 수 있도록 합니다  

### 🔧 ActivityManager의 주요 기능
1. 태스크 및 Activity 정보  
  - 실행 중인 태스크, Activity 및 해당 스택 상세애 대한 세부 정보를 추적할 수 있습니다  
  - 앱 동작 및 시스템 리소스 사용량을 모니터링 하는데 도움이 됩니다
2. 메모리 관리  
  - 앱의 메모리 소비 및 시스템 전체 메모리 상태를 포함하여 시스템 전체의 메모리 사용량에 대한 정보를 제공합니다.  
  - 앱 성능을 최적화하고 메모리 부족 상태를 처리할 수 있습니다  
3. 앱 프로세스 관리  
  - 실행중인 앱 프로세스 및 Service에 대한 세부 정보를 쿼리할 수 있습니다
  - 앱 상태를 감지하거나 프로세스 수준의 변화에 응답할 수 있습니다  
4. 디버깅 및 진단  
  - 힙 덤프 생성 또는 앱 프로파일링과 같이 디버깅을 위한 도구를 제공합니다  
  - 성능 병목 현상이나 메모리 누수를 식별하는 데 도움이 될 수 있습니다  

### ⚙️ ActivityManager에서 제공하는 메서드

#### **getRunningAppProcesses()**
- 기기에서 현재 실행 중인 프로세스 목록을 반환합니다

#### **getMemoryInfo(ActivityManager.MemoryInfo memoryInfo)**
- 사용 가능한 메모리, 임계 메모리, 기기가 메모리 부족 상태인지 여부 등 시스템에 대한 자세한 메모리 정보를 검색합니다
- 메모리 부족 상태 동안 앱 동작을 최적화하는 데 유용합니다

#### **killBackgroundProcesses(String packageName)**
- 시스템 리소스를 확보하기 위해 지정된 앱의 백그라운드 프로세스를 종료합니다
- 리소스 집약적인 앱을 테스트하거나 관리하는 데 유용합니다

#### **isLowRamDevice()**
- 기기가 저사양 RAM으로 분류되는지 확인하여 앱이 저메모리 기기에 대한 리소스 사용량을 최적화하는 데 도움을 줍니다

#### **appNotResponding(String message)**
- 테스트 목적으로 ANR(App Not Responding) 이벤트를 시뮬레이션합니다
- 디버깅 중에 앱이 ANR 상황에서 어떻게 동작하거나 응답하는지 이해하는 데 사용할 수 있습니다

#### **clearApplicationUserData()**
- 파일, 데이터베이스 및 SharedPreferences를 포함하여 애플리케이션과 관련된 모든 사용자별 데이터를 지웁니다
- 공장 초기화나 앱을 기본 상태로 재설정하는 경우에 종종 사용됩니다

### 🛠️ 사용 예제
아래 코드는 ActivityManager를 사용하여 메모리 정보를 가져오는 방법을 보여줍니다.

```kotlin
val activityManager = getSystemService(Context.ACTIVITY_SERVICE) as ActivityManager
val memoryInfo = ActivityManager.MemoryInfo()
activityManager.getMemoryInfo(memoryInfo)

Log.d(TAG, "Low memory state: ${memoryInfo.lowMemory}")
Log.d(TAG, "Available memory: ${memoryInfo.availMem / (1024 * 1024)} MB")
Log.d(TAG, "Threshold memory: ${memoryInfo.threshold / (1024 * 1024)} MB")

val processes = activityManager.runningAppProcesses
if (processes != null && processes.isNotEmpty()) {
    Log.d(TAG, "Process name: ${processes.first().processName}")
}

// 테스트 목적으로 ANR(App Not Responding) 이벤트를 시뮬레이션합니다. (주의해서 사용)
// activityManager.appNotResponding("Pokedex is not responding")

// 애플리케이션이 디스크에서 할당 중인 데이터를 지우도록 허용 (주의해서 사용)
// activityManager.clearApplicationUserData()
```

### 💡 Pro Tips for Mastery: LeakCanary에서의 ActivityManager 활용 사례
**LeakCanary**는 Block이라는 기업에서 관리하는 안드로이드 애플리케이션용 오픈 소스 메모리 누수 감지 라이브러리입니다.  
개발 중에 앱의 메모리 누수를 자동으로 모니터링하고 감지하여, 개발자가 메모리 누수를 효율적으로 고치는 데 도움이 될만한 분석을 제공합니다.  
LeakCanary는 내부적으로 메모리 상태 및 정보 추적을 위해 **ActivityManager를 활용**합니다.

### 🧠 요약
ActivityManager는 시스템 수준 관리, 성능 튜닝 및 앱 동작 모니터링을 위한 것입니다.  
최신 안드로이드에서는 해당 기능들이 부분적으로 더 고도화된 API로 대체되었지만,  
안드로이드 애플리케이션에서 리소스 사용을 관리하고 최적화하기 위한 도구로 여전히 활용할 수 있습니다.  
적절한 상황에서 효율적으로 사용하면 개발자는 의도하지 않은 시스템 성능 저하를 피하기 위한 목적으로 활용할 수 있습니다.

### ❓ 실전 질문
**ActivityManager.getMemoryInfo()를 어떻게 앱 성능 최적화에 활용할 수 있으며, 시스템이 메모리 부족 상태에 들어가면 개발자는 어떤 조치를 취해야 하나요?**

#### 🎯 **getMemoryInfo() 활용 방법**

##### **메모리 상태 모니터링**
```kotlin
val activityManager = getSystemService(Context.ACTIVITY_SERVICE) as ActivityManager
val memoryInfo = ActivityManager.MemoryInfo()
activityManager.getMemoryInfo(memoryInfo)

// 메모리 부족 상태 확인
if (memoryInfo.lowMemory) {
    // 메모리 부족 시 최적화 로직 실행
    optimizeMemoryUsage()
}
```

##### **동적 리소스 관리**
```kotlin
private fun optimizeMemoryUsage() {
    // 캐시 크기 줄이기
    imageCache.evictAll()
    
    // 불필요한 데이터 정리
    clearUnusedResources()
    
    // 이미지 품질 낮추기
    imageLoader.setQuality(ImageQuality.LOW)
}
```

#### 🛠️ **메모리 부족 시 개발자 조치**

##### **즉시 대응 조치**
- **캐시 정리**: 이미지 캐시, 데이터베이스 캐시 등 즉시 정리
- **백그라운드 작업 중단**: 불필요한 백그라운드 작업 일시 중단
- **UI 요소 최적화**: 복잡한 애니메이션이나 무거운 UI 요소 비활성화

##### **앱 구조 최적화**
- **메모리 누수 방지**: WeakReference, SoftReference 활용
- **이미지 최적화**: Glide, Picasso 등 이미지 라이브러리 사용
- **데이터베이스 최적화**: 쿼리 최적화 및 인덱스 활용

##### **사용자 경험 개선**
- **로딩 상태 표시**: 메모리 정리 중 사용자에게 피드백 제공
- **점진적 로딩**: 대용량 데이터를 단계적으로 로드
- **메모리 사용량 표시**: 개발자 옵션에서 메모리 사용량 모니터링

#### 💡 **실제 구현 예시**
```kotlin
class MemoryOptimizer(private val context: Context) {
    private val activityManager = context.getSystemService(Context.ACTIVITY_SERVICE) as ActivityManager
    
    fun checkMemoryStatus() {
        val memoryInfo = ActivityManager.MemoryInfo()
        activityManager.getMemoryInfo(memoryInfo)
        
        when {
            memoryInfo.lowMemory -> {
                // 긴급 메모리 정리
                emergencyMemoryCleanup()
            }
            memoryInfo.availMem < memoryInfo.threshold * 1.5 -> {
                // 예방적 메모리 정리
                preventiveMemoryCleanup()
            }
        }
    }
    
    private fun emergencyMemoryCleanup() {
        // 모든 캐시 정리
        // 백그라운드 작업 중단
        // 사용자에게 알림
    }
}
```

#### 🧠 **요약**
**getMemoryInfo()**를 활용하여 실시간으로 메모리 상태를 모니터링하고,  
메모리 부족 시 즉시 대응하는 로직을 구현하여 앱의 안정성과 성능을 보장할 수 있습니다.  
이는 특히 메모리 제약이 있는 기기에서 사용자 경험을 크게 향상시킬 수 있습니다.

## 📚 Q21. SparseArray 무엇인가요?  

### 🎯 개요
SparseArray는 HashMap과 유사하게 정수 키를 객체 값에 매핑하는 안드로이드에 최적화된 데이터 구조입니다.  
정수인 키와 함께 사용하도록 최적화되어 있어 정수 기반 키를 사용할 때 Map이나 HashMap보다  
메모리 관리 측면에서 효율이 좋고 상황에 따라 성능이 더 좋습니다  
(because it avoids auto-boxing keys and its data structure doesn't rely on an extra entry object for each mapping.)  

### 🔧 SparseArray의 주요 특징
1. 메모리 효율성  
  - SparseArray는 오토박싱을 피하고 Entry 객체와 같은 추가 데이터 구조에 의존하지 않습니다
  - 훨씬 적은 메모리를 소비합니다.
2. 성능  
  - 메모리 최적화 덕분에 중간 크기의 데이터 셋에서 더 나은 성능을 제공합니다  
3. Null 키 값 사용 불가
  - 키 값으로 기본 정수를 사용하므로 키 값에 null을 허용하지 않습니다  

```kotlin
import android.util.SparseArray

val sparseArray = SparseArray<String>()
sparseArray.put(1, "One")
sparseArray.put(2, "Two")

// 요소 접근
val value = sparseArray[1] // "One"

// 요소 제거
sparseArray.remove(2)

// 요소 순회
for (i in 0 until sparseArray.size()) {
    val key = sparseArray.keyAt(i)
    val value = sparseArray.valueAt(i)
    println("Key: $key, Value: $value")
}
```

### 🛠️ SparseArray 사용의 이점
1. 오토박싱 방지  
  - HashMap<Integer, Object>에서는 키가 Integer 객체로 박싱 및 언박싱 비용으로 오버헤드가 발생합니다
  - SparseArray는 int 키로 직접 작동하여 메모리와 계산 작업을 절약합니다.
2. 메모리 절약 
  - 키와 값을 저장하기 위해 내부적으로 기본 재열을 사용하여 Entry와 같은 여러 객체를 생성하는  
    HashMap 구현에 비해 메모리 차지 공간을 줄입니다.
3. 컴팩트한 데이터 저장에 효율적  
  - 적은 수의 키-값 쌍이 있는 밀도가 낮은 데이터 셋이나 키가 넓은 정수 범위에 걸쳐  
    드문드문 분포된 데이터 셋에 적합합니다.
4. 안드로이드 특화  
  - 제한된 리소스를 처리하기 위해 안드로이드에 특화된 구조로 설계, View ID를 객체에 매핑하는 등의 시나리오에 효과적

### 🛠️ SparseArray의 한계
메모리 효율적이지만 모든 사용 사례에 항상 적합한 선택은 아닙니다.
1. 성능 트레이드오프
  - 요소 접근은 키 조회를 위해 이진 탐색을 사용하기 때문에 매우 큰 데이터 셋의 경우 HashMap보다 느립니다.
2. 정수 키만 사용 가능
  - 정수 키로 제한되어 다른 유형의 키가 필요한 사용 사례에는 적합하지 않습니다


#### 🧠 **요약**
안드로이드에서 메모리 효율성을 위해 최적화된, 정수 키를 객체를 매핑하는 특수한 자료구조  
오토박싱을 피하고 메모리 사용량을 줄이므로 HashMap보다 상당한 이점이 있음  
정수 키를 사용하는 데이터 셋에 적합함  
메모리 절약을 위해 일부 성능을 절충할 수 있지만, 리소스가 제한적인 사례에 유용함

## 📚 Q22. 런타임 권한을 어떻게 처리하나요?

### 🎯 개요
안드로이드 6.0(API 23)부터는 앱은 설치 시 자동으로 권한을 획득하는 대신  
런타임에 위험 권한을 명시적으로 요청해야 합니다  

### 🔧 권한 선언 및 확인
권한을 요청하기 전에 앱은 AndroidManifest.xml 파일에 해당 권한을 선언해야 합니다.  
런타임 시에는 사용하가 해당 권한이 필요한 기능과 상호 작용할 때만 권한을 요청해야 합니다.  

사용자에게 요청하기 전에 ContextCompat.checkSelfPermission()을 사용하여  
권한이 이미 부여되었는지 확인하는 것이 중요합니다.  
권한이 부여되지 않았으면 사용자에게 권한을 올바르게 요청해야 합니다.  

```kotlin
when {
    ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) == PackageManager.PERMISSION_GRANTED -> {
        // 권한 부여됨, 동작 이어서 진행
    }
    ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.CAMERA) -> {
        // 사용자가 권한 요청을 거부한 경우
        showPermissionRationale()
    }
    else -> {
        // 사용자 권한 요청
        requestPermissionLauncher.launch(Manifest.permission.CAMERA)
    }
}
```

### 🔧 권한 요청하기  
권한 요청에 권장되는 방법은 ActivityResultLauncher API를 사용하는 것입니다.  
그러면 시스템은 사용자에게 권한 요청을 허용하거나 거부하도록 안내합니다.  
```kotlin
val requestPermissionLauncher =
    registerForActivityResult(ActivityResultContracts.RequestPermission()) { isGranted ->
        if (isGranted) {
            // 권한 부여됨, 동작 이어서 진행
        } else {
            // 권한 거부됨
        }
    }
```

### 🔧 권한 요청 근거 제공하기
경우에 따라 시스템은 권한이 왜 필요한지 설명해야 합니다  
이는 사용자 경험을 개선하고 권한 획득 가능성을 높입니다  

```kotlin
fun showPermissionRationale() {
    AlertDialog.Builder(this)
        .setTitle("권한 필요")
        .setMessage("이 기능이 제대로 작동하기 위해 카메라 접근 권한이 필요합니다.")
        .setPositiveButton("확인") { _, _ ->
            requestPermissionLauncher.launch(Manifest.permission.CAMERA)
        }
        .setNegativeButton("취소", null)
        .show()
}
```

### 🔧권한 거부 처리하기  
사용자가 권한을 여러 번 거부하면 안드로이드는 이를 영구 거부로 처리하여 앱이 다시 요청할 수 없게 됩니다  
앱은 사용자에게 기능제한에 대해 알리고 필요한 경우 시스템 설정으로 안내해야 합니다.  

```kotlin
if (!ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.CAMERA) &&
    ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
    // 사용자가 영구적으로 권한을 거부함 ('다시 묻지 않음' 선택)
    showSettingsDialog()
}

fun showSettingsDialog() {
    val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS).apply {
        data = Uri.parse("package:$packageName")
    }
    startActivity(intent)
}
```

### 🔧 위치 권한 처리하기  
위치 권한은 포그라운드와 백그라운드 접근으로 분류됩니다  
포그라운드에는 ACCESS_FINE_LOCALE, ACCESS_COARSE_LOCATION  
백그라운드에는 ACCESS_BACKGROUND_LOCATION 권한이 필요합니다.  

#### 🧠 **요약**
런타임 권한을 적절하게 처리하면 보안, 규정 준수 및 사용자 신뢰를 보장할 수 있습니다.  
모범 사례를 따르면 원활하고 개인 정보를 존중하는 사용자 경험을 만들 수 있습니다.  

## 📚 Q23. Looper, Handler, HandlerThread의 역할은 무엇인가요?

### 🎯 개요  
Looper, Handler, HandlerThread는 안드로이드에서 **스레드 간 비동기 작업과 메시지 처리**를 위해 함께 동작하는 핵심 컴포넌트입니다.  
이들은 백그라운드 작업을 효율적으로 처리하면서, UI 스레드와 안전하게 통신할 수 있도록 설계되었습니다.

---

### 🛠️ 주요 역할 및 사용법

#### 1. **Looper**
- **정의**: 스레드를 살아있게 유지하며, 메시지 큐를 순차적으로 처리하는 역할
- **용도**: 메인(UI) 스레드에는 기본적으로 Looper가 존재하며, 워커 스레드에서는 직접 생성해야 함
- **초기화 예시**:
    ```kotlin
    val thread = Thread {
        Looper.prepare() // Looper 연결
        val handler = Handler(Looper.myLooper()!!) // Handler 생성
        Looper.loop() // 메시지 루프 시작
    }
    thread.start()
    ```

#### 2. **Handler**
- **정의**: 메시지나 Runnable을 Looper가 있는 스레드의 메시지 큐에 전달하고 처리하는 역할
- **용도**: 한 스레드에서 다른 스레드(특히 UI 스레드)로 작업을 전달할 때 사용
- **예시**:
    ```kotlin
    val handler = Handler(Looper.getMainLooper()) // 메인 스레드용 Handler
    handler.post {
        // UI 업데이트 코드
        textView.text = "Updated from background thread"
    }
    ```

#### 3. **HandlerThread**
- **정의**: Looper가 내장된 특수한 Thread로, 별도의 워커 스레드에서 메시지 큐를 처리할 수 있게 함
- **용도**: 백그라운드에서 순차적으로 작업을 처리해야 할 때 사용
- **예시**:
    ```kotlin
    val handlerThread = HandlerThread("WorkerThread")
    handlerThread.start()
    val workerHandler = Handler(handlerThread.looper)
    workerHandler.post {
        // 백그라운드 작업
        Thread.sleep(1000)
        Log.d("HandlerThread", "Task completed")
    }
    // 스레드 종료
    handlerThread.quitSafely()
    ```

---

### ⚙️ 관계 및 차이점 요약
- **Looper**: 메시지 큐를 관리하며, 스레드를 계속 실행 상태로 유지
- **Handler**: Looper와 연결되어 메시지/작업을 큐에 넣고 처리
- **HandlerThread**: Looper가 내장된 워커 스레드로, 백그라운드 작업에 적합

---

### 🧠 요약
- Looper, Handler, HandlerThread는 **안드로이드의 비동기 처리와 스레드 간 통신의 핵심**입니다.
- Looper는 메시지 큐를 관리, Handler는 메시지 전달 및 실행, HandlerThread는 Looper가 내장된 백그라운드 스레드 제공
- 이 구조를 활용하면 **UI 스레드와 워커 스레드 간 안전하고 효율적인 작업 분리**가 가능합니다.