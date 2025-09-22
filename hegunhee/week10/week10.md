# 카테고리 3: 비지니스 로직  
비즈니스 로직은 일반적으로 데이터 계층 및 도메인 계층 내에 존재하며  
UI와 관련성이 적은 작업에 중점을 둡니다.  
네트워크 요청, 데이터베이스 쿼리, 로컬 저장소에 값 읽고 쓰기 등 백엔드 서버와의 통신 등이 포함됩니다.  
비즈니스 논리와 관련된 작업은 대부분 선택이 아닌 필수인 경우가 많습니다.  
따라서 해당 로직들을 올바르게 이해하고, 효율적으로 사용하는 것이 매우 중요합니다.  

## 📚 Q59. 장기적으로 실행되는 백그라운드 작업을 어떻게 관리하나요?  

### 🎯 개요  
안드로이드는 최적의 리소스 사용과 최신 OS 제한 사항 준수를 보장하면서 장기적으로 실행 가능한 백그라운드 작업을  
처리하기 위한 여러 매커니즘을 제공합니다.  

#### ⚙️ 조건부 작업에 적합한 WorkManager (즉시, 반복, 지연)  
앱이 닫히거나 기기 재부팅 후에도 실행되어야 하는 작업의 경우 WorkManager가 공식적으로 권장되는 솔루션입니다.  
백그라운드 작업을 관리하고 네트워크 가용성 또는 충전 상태와 같은 제약 조건 하에서  
작업이 실행되도록 보장합니다.  
예) 로그 업로드, 데이터 동기화, 비디오와 같은 영상 파일 업로드  

```kotlin
class UploadWorker(appContext: Context, workerParams: WorkerParameters) : Worker(appContext, workerParams) {
    override fun doWork(): Result {
        // 여기서 백그라운드 작업 수행
        uploadData()
        return Result.success()
    }
}

// 작업 예약
val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.CONNECTED) // 네트워크 연결 필요 제약 조건
    .build()

val workRequest = OneTimeWorkRequestBuilder<UploadWorker>()
    .setConstraints(constraints)
    .build()

WorkManager.getInstance(context).enqueue(workRequest)
````
https://developer.android.com/develop/background-work/background-tasks/persistent?hl=ko


#### ⚙️ 오랜 작업에 적합한 Service  
음악 재생이나 위치 추적과 같이 지속적이고 오랜 실행이 필요한 작업에는 서비스가 이상적입니다.  
서비스는 UI와 독립적으로 실행되며 앱이 백그라운드에 있을 때도 계속 실행될 수 있습니다.  
작업이 노티피케이션과 함께 사용자가 인지할 수 있는 상황에서 실행되어야 하는 경우 Foreground Service를 사용합니다.  

```kotlin
class MyForegroundService : Service() {
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        // 장기 실행 작업 수행
        startForeground(NOTIFICATION_ID, createNotification())
        // START_STICKY, START_REDELIVER_INTENT 등 반환 값 고려
        return START_NOT_STICKY
    }

    // 알림 생성 로직 (Notification Channel 포함)
    private fun createNotification(): Notification {
        return NotificationCompat.Builder(this, CHANNEL_ID).build()
    }
}
```
#### ⚙️ Kotlin Coroutines 및 Dispatchers 사용하기  
앱 생명주기에 연결된 작업의 경우 Kotlin Coroutines는 Kotlin 언어 수준에서 깔끔하고 구조화된 접근 방식을 제공합니다.  
무거운 작업을 오프로드하려면 Dispatchers.IO를 사용하고 CPU 집약적인 계산에는 Dispatchers.Default를 사용합니다.  
해당 접근 방식은 앱이 닫힌 후에도 유지될 필요가 없는 작업에 이상적입니다.  

```kotlin
class MyViewModel : ViewModel() {
    fun fetchData() {
        viewModelScope.launch(Dispatchers.IO) { // IO 디스패처에서 네트워크 작업 실행
            val data = fetchFromNetwork()
            // 결과를 메인 스레드로 전환하여 UI 업데이트
            withContext(Dispatchers.Main) {
                updateUI(data)
            }
        }
    }
}
```

#### 💡 Additional Tips CPU 집약적이란 말의 뜻  
CPU 집약적이란 말은 굉장히 무겁고 CPU를 많이 사용해야 할 것 같은 작업이라고 느낄 수 있지만  
CPU 집약적인 연산은 **연산 능력을 요구하며** 사용 가능한 CPU 코어 수에 의해 제약을 받습니다.  
즉 디바이스에서 사용 가능한 코어의 수만큼만 쓰레드를 생성합니다.  
암호화, 이미지 처리, 비디오 인코딩 같은 작업에 쓰입니다.  
이러한 작업은 전적으로 CPU에 의존하기 떄문에 사용 가능한 코어 수 보다 많은 스레드를 추가하면 성능 향상보다는  
스레드 경합이 발생하는 경우가 많습니다.  

I/O와 관련한 작업은 CPU 코어 수보다 많은 스레드를 사용함으로써, 여러 I/O 작업이 CPU 경합을 일으키지 않고  
동시에 실행될 수 있게 합니다.

#### ⚙️ 시스템 수준 작업에 적합한 JobScheduler  
작업이 기기 전체 작업과 관련되고 특정 조건이 필요한 경우 JobScheduler를 사용할 수 있습니다.  
즉시 실행이 필요하지 않은 작업에 적합합니다.  
(일반적인 상황에서는 WorkManager 권장)  

```kotlin
val jobScheduler = context.getSystemService(JobScheduler::class.java)
val jobInfo = JobInfo.Builder(JOB_ID, ComponentName(context, MyJobService::class.java))
    .setRequiredNetworkType(JobInfo.NETWORK_TYPE_UNMETERED) // 네트워크에 연결 중인 상황에
    .setRequiresCharging(true) // 충전 중인 상황에만 실행
    .setPeriodic(TimeUnit.MINUTES.toMillis(15)) // 작업 주기 설정
    .build()

jobScheduler.schedule(jobInfo)
````

#### 🧠 요약
- WorkManager
  - 신뢰할 수 있는 조건부 영구 작업에 적합
- Services
  - 미디어 재생이나 위치 추적과 같은 연속적이고 오랜 작업에 적합
- Kotlin Coroutines
  - 생명주기에 바운딩된 가벼운 작업에 적합
- JobScheduler  
  - 시스템 수준 작업에 적합  

#### ❓ 실전 질문
#### 안드로이드 앱에서 백엔드 서버로부터 대용량파일을 다운로드하는 기능을 구현해야 합니다.
#### 다운로드는 앱이 닫혀도 계속 되어야 하며, 성능 및 네트워크 조건 측면에서 효율적이어야 합니다. 어떤것을 선택해야 할까요?  
Foreground Service  
장시간 실행 가능하며, 앱 종료 후에도 계속 실행, 사용자에게 진행상황을 알릴 수 있고  
시스템에 의한 강제 종료 가능성이 낮으므로 Foreground Service가 적합합니다.  

## 📚 Q60. Json 형식을 어떻게 객체로 직렬화하나요?  

### 🎯 개요  
최신 안드로이드 앱은 원격 서버와 자주 상호작용하기 때문에 JSON을 객체로 직렬화합니다.  
지속적으로 변동성이 있는 사용자 데이터, 환경 설정 또는 콘텐츠를  
리모트 서버에서 가져오기 위해 앱은 Kotlin 객체를 JSON으로 직렬화하여  
데이터를 백엔드로 보내 필요한 정보를 요청하고, 서버의 JSON 응답을 받으면 다시 객체로 역직렬화 합니다.  

#### 🛠️ 직렬하와 역직렬화란?  
- 직렬화(Serialization)  
  - 객체나 데이터 구조를 나중에 쉽게 저장, 전송 또는 재구성할 수 있는 형식으로 변환하는 프로세스  
  - 안드로이드와 백엔드 통신에서는 종종 객체를 JSON 문자열이나 유사항 구조화된 형식으로 변환하는 것을 의미  
- 역직렬화(Deserialization)  
  - 직렬화된 데이터를 가져와 애플리케이션에서 작업할 수 있는 메모리 내 객체로 다시 재구성하는 역 프로세스입니다.  

#### ⚙️ 수동 직렬화 및 역직렬화  
외부의 솔루션을 사용하지 않고도 직접 수동적으로 문자열 조작 및 파싱 기술을 통해  
객체를 JSON 문자열로 변환하고, 그 반대로 변환하여 직, 역직렬화할 수 있습니다.  

data class User(val name: String, val age: Int)

```kotlin
fun serializeUser(user: User): String {
    // 각 속성을 JSON 형식에 맞게 문자열로 변환
    return """{
        "name": "${user.name}",
        "age": ${user.age}
    }""".trimIndent() // 들여쓰기 제거
}

// 사용 예제
val user = User("John", 30)
val jsonString = serializeUser(user)
// 출력: {"name":"John","age":30}
```
반면에 수동 역직렬화는 JSON 문자열을 파싱하고 값을 추출하여 객체를 재구성하는 것을 동반합니다.  
```kotlin
fun deserializeUser(json: String): User {
    // 정규 표현식을 사용하여 값 추출 (간단한 예시)
    val nameRegex = """"name"\s*:\s*"([^"]*)"""".toRegex()
    val ageRegex = """"age"\s*:\s*(\d+)""".toRegex()

    val name = nameRegex.find(json)?.groups?.get(1)?.value ?: ""
    val age = ageRegex.find(json)?.groups?.get(1)?.value?.toIntOrNull() ?: 0

    return User(name, age)
}

// 사용 예제
val jsonString = """{"name":"John","age":30}"""
val user = deserializeUser(jsonString)
// 출력: User(name="John", age=30)
```
이 방법은 권장되지는 않지만 학습 목적이나 의존성을 최소화하는 가벼운 경우에는 유용할 수 있습니다.  
kotlinx.serialization, Moshi, Gson과 같은 라이브러리를 통해 해당 프로세스를 훨씬 더 효과적으로 구현하여  
JSON 문자열을 Kotlin 또는 Java 객체로 또는 그 반대로 역변환할 수 있습니다.  

#### ⚙️ kotlinx.serialization  
Jetbrains에서 개발한 kotlinx.serialization은 kotlin과 직접적으로 통합되어  
Kotlin의 언어적 기능을 활용하도록 설계되었습니다.  
어노테이션을 사용하여 직렬화 동작을 정의하고 JSON뿐만 아니라 ProtoBuf와 같은 다른 형식과도 원활하게 동작합니다.  

```kotlin
import kotlinx.serialization.*
import kotlinx.serialization.json.*

@Serializable // 직렬화 가능 클래스임을 명시
data class User(val name: String, val age: Int)

val json = """{"name": "John", "age": 30}"""
// JSON 문자열을 User 객체로 역직렬화
val user: User = Json.decodeFromString<User>(json)
// User 객체를 JSON 문자열로 직렬화
val serializedJson: String = Json.encodeToString(user)
```
kotlinx.serialization은 Kotlin 컴파일러 플러그인을 사용하여  
Kotlin 객체를 JSON 문자열로 변환하고 JSON 문자열을 다시 Kotlin 객체로 파싱하는 타입 안정성을 보장하며  
내부적으로 리플렉션을 사용하지 않는 메커니즘을 제공하므로  
최신 안드로이드 및 Kotlin 개발에서 가장 선호되는 방법 중 하나입니다.  

```kotlin
data class User(val name: String, val age: Int) {
    companion object {
        val serializer = object : KSerializer<User> {
            override fun serialize(encoder: Encoder, value: User) {
                encoder.encodeString(value.name)
                encoder.encodeInt(value.age)
            }
            
            override fun deserialize(decoder: Decoder): User {
                val name = decoder.decodeString()
                val age = decoder.decodeInt()
                return User(name, age)
            }
        }
    }
}
```

내부적으로 Kotlin은 Moshi의 리플렉션 모드 및 Gson과 달리 무거운 런타임 리플렉션 없이  
효율적인 직렬화를 수행하기 위해 컴파일러가 생성하는 코드에 의존합니다.  

#### ⚙️ Moshi  
Square에서 개발한 Moshi는 타입 안정성과 Kotlin 지원을 강조하는 최신 JSON 라이브러리  
Gson과 달리 Moshi는 Kotlin의 **null 가능성** 및 **기본 매개변수**를 기본적으로 지원합니다.  
```kotlin
data class User(val name: String, val age: Int)

// Moshi 인스턴스 생성 (Kotlin 지원 추가 필요)
val moshi = Moshi.Builder()
    .add(KotlinJsonAdapterFactory()) // Kotlin 지원 추가
    .build()

// User 클래스에 대한 어댑터 가져오기
val adapter: JsonAdapter<User> = moshi.adapter(User::class.java)

val json = """{"name": "John", "age": 30}"""

// JSON을 객체로 역직렬화
val user: User? = adapter.fromJson(json)

// 객체를 JSON으로 직렬화
val serializedJson: String? = user?.let { adapter.toJson(it) }
```

Moshi에서는 JSON 직렬화 및 역직렬화를 처리하는 두 가지 주요 접근 방식이 있는데  
리플렉션에 기반한 동작 방식과 컴파일 타임에 코드를 생성하는 방식 두 가지입니다.  

- 리플렉션 기반 Moshi  
  - 리플렉션 기반 Moshi는 Java 리플렉션을 사용하여 런타임에 동적으로 JSON 어댑터를 생성  
  - 추가 설정 필요 없지만 런타임 오버헤드가 발생
- 코드 생성 기반  
  - 어노테이션 프로세스 기법을 통해 컴파일 타임에 JSON 어댑터를 생성하여, 더 빠른 런타임  
    성능과 컴파일 타임 오류 검사를 제공합니다.  

Moshi의 리플렉션 기반 어댑터는 빠른 셋업 환경을 제공하지만  
런타임에 성능 오버헤드가 발생하기 때문에 권장되는 솔루션이 아닙니다.  
Moshi의 코드 생성 기반 플러그인은 컴파일 타임에 최적화된 어댑터를 생성하여 더 나은 성능을 제공하므로  
일반적인 상황에서 더 선호됩니다.  

#### ⚙️ Gson  
Google에서 개발한 Gson은 널리 사용되는 JSON 라이브러리입니다.  
Java 객체를 JSON으로 직렬화하고 JSON을 다시 Java 객체로 역직렬화할 수 있습니다.  
간단한 API와 통합 용이성 덕분에 인기 있는 라이브러리입니다.  

```kotlin
data class User(val name: String, val age: Int)

val gson = Gson()
val json = """{"name": "John", "age": 30}"""
val user = gson.fromJson(json, User::class.java) // JSON을 객체로 역직렬화
val serializedJson = gson.toJson(user) // 객체를 JSON으로 직렬화
```

#### ⚙️ Gson보다 kotlinx.serialization, Moshi를 사용해야 하는 이유  
1. 더 나은 Kotlin 지원  
  - Gson은 Java용 Kotlin의 다양한 기능(기본 매개변수, val/var 차이, null 허용성)을 자연스럽게 처리하지 못함  
2. 성능 및 효율성  
  - kotlinx.serialization 및 Moshi는 런타임에 리플렉션에 크게 의존하는 Gson보다 빠르고 메모리 효율적입니다.  
3. 멀티플랫폼 호환성  
  - kotlinx.serialization은 KMP를 환벽하게 지원  
4. 컴파일 타임 안정성  
  - kotlinx.serialization및 Moshi는 컴파일 타임에 대부분 오류를 감지  

#### 🧠 요약  
kotlinx.serialization은 Kotlin 우선 및 KMP 프로젝트에 가장 적합한 선택입니다.  
Moshi는 빠른 성능과 더 안전한 JSON 처리가 필요한 안드로이드 중심 앱에 이상적입니다.  
Gson은 JVM 기반의 프로젝트에 통합하기 쉽지만 성능, Kotlin 지원 및 현대 앱 개발의 모범 사례에서는 뒤쳐집니다.

## 📚 Q61. 원격 데이터를 가져오기 위해 네트워크 요청을 어떻게 처리하나요?  

### 🎯 개요  
일반적으로 Retrofit과 OkHttp는 안드로이드 개발에서 백엔드로부터 네트워크 요청을 하는 데  
흔히 사용되는 라이브러리 입니다.  

Retrofit은 선언적 인터페이스를 제공하여 API 상호 작용을 단순화하는 반면  
OkHttp는 기본적인 HTTP 클라이언트 역할을 하여 연결 풀링, 캐싱 및 효율적인 통신을 제공합니다.  

#### 🛠️ Retrofit을 사용한 네트워크 요청  
Retrofit은 HTTP 요청을 깔끔하고 타입 세이프한 API 인터페이스로 추상화합니다.  
JSON 응답을 Kotlin 또는 Java 객체로 변환하기 위해 Gson 또는 Moshi와 같은 직렬화 라이브러리와 원활하게 작동합니다.  

Retrofit을 사용하여 백엔드 데이터를 가져오려면 다음 단계를 따릅니다.  
1. API 인터페이스 정의  
어노테이션을 사용하여 API 엔드포인트와 HTTP 메서드를 선언합니다.  

```Kotlin
interface ApiService {
    @GET("data")
    // 코루틴 지원을 위한 suspend 함수 또는 Call<DataModel> 반환
    suspend fun fetchData(): Response<DataModel>
}
```

2. Retrofit 인스턴스 설정  
베이스 URL과 JSON 직렬화를 위해 ConverterFactory를 아래와 같이 하여 Retrofit을 구성합니다.  

```kotlin
1 // Kotlinx Serialization Converter Factory 예시
2 val retrofit = Retrofit.Builder()
3.baseUrl("https://api.example.com/")
4.addConverterFactory(Json{ ignoreUnknownKeys = true } // 알 수 없는 키 무시 설정 추가
5.asConverterFactory("application/json"
.toMediaType()))
6.build()
7
8 // API 서비스 인스턴스 생성
9 val apiService = retrofit.create(ApiService::class.java)
```

3. 네트워크 호출하기  
코루틴을 사용하여 API를 비동기적으로 호출합니다.  

```kotlin
viewModelScope.launch { // ViewModel 스코프 또는 다른 코루틴 스코프 사용
    try {
        val response = apiService.fetchData()
        if (response.isSuccessful) {
            // 성공적인 응답 처리
            val data = response.body()
            if (data != null) { // Nullable일 수 있으므로 확인 필요
                Log.d(TAG, "Data fetched: $it")
                // UI 업데이트 또는 데이터 처리
            } else {
                Log.e(TAG, "Response body is null")
            }
        } else {
            // 오류 응답 처리
            Log.e(TAG, "Error: ${response.code()}- ${response.message()}")
            // 오류 본문 파싱 (response.errorBody()) 등
        }
    } catch (e: Exception) { // 네트워크 오류 또는 기타 예외 처리
        Log.e(TAG, "Network request failed", e)
    }
}
```

#### 🛠️ OkHttp를 사용한 커스텀 HTTP 요청  
OkHttp는 헤더, 캐싱 등을 세밀하게 제어할 수 있도록 HTTP 요청을  
관리하기 위해 더 직접적인 접근 방식을 보여줍니다.  

```kotlin
val client = OkHttpClient()

val request = Request.Builder()
    .url("https://api.example.com/data")
    .build()

// 비동기 요청 실행
client.newCall(request).enqueue(object : Callback {
    override fun onFailure(call: Call, e: IOException) {
        // 네트워크 오류 처리
        e.printStackTrace()
    }

    override fun onResponse(call: Call, response: Response) {
        // 응답 스레드는 백그라운드 스레드일 수 있으므로 UI 업데이트 시 주의
        response.use { // response.body().close() 자동 호출
            if (response.isSuccessful) {
                val responseBody = response.body?.string() // 응답 본문 읽기 (한 번만 가능)
                Log.d(TAG, "Response: $responseBody")
                // UI 업데이트는 메인 스레드에서 수행
                // runOnUiThread { /* UI 업데이트 */ }
            } else {
                Log.e(TAG, "Error: ${response.code}")
            }
        }
    }
})
```
#### 🛠️ OkHttp와 Retrofit 통합하기  
Retrofit은 내부적으로 OkHttp를 HTTP 클라이언트로 사용합니다.  
인터셉터를 추가하여 로깅, 인증 또는 캐싱과 같은 OkHttp의 동작을 커스텀할 수 있습니다.  

```kotlin
val okHttpClient = OkHttpClient.Builder()
    // 로깅 인터셉터 추가 (예시)
    .addInterceptor(HttpLoggingInterceptor().apply { level = HttpLoggingInterceptor.Level.BODY })
    // 인증 헤더 추가 인터셉터
    .addInterceptor { chain ->
        val originalRequest = chain.request()
        val newRequest = originalRequest.newBuilder()
            .header("Authorization", "Bearer your_token") // 실제 토큰 사용
            .build()
        chain.proceed(newRequest)
    }
    .connectTimeout(30, TimeUnit.SECONDS) // 타임아웃 설정 (선택 사항)
    .readTimeout(30, TimeUnit.SECONDS)
    .build()

val retrofit = Retrofit.Builder()
    .baseUrl("https://api.example.com/")
    .client(okHttpClient) // 커스텀 OkHttpClient 설정
    .addConverterFactory(GsonConverterFactory.create()) // 또는 다른 Converter Factory
    .build()
```

#### 🧠 요약  
Retrofit은 내부적으로 복잡한 처리를 하고 있어도 API 표면은 개발자들이 사용하기 쉽게 설계되었습니다.  
OkHttp는 네트워크 동작을 사용자가 쉽게 커스텀할 수 있는 유연성을 제공합니다.  

#### ⚙️ Ktor란? (추가 조사)  
JetBrains가 만든 비동기 HTTP 클라이언트/서버 프레임워크  
코틀린 친화적이며 suspend 함수를 지원  
멀티 플랫폼 지원이 가능  

```kotlin
val client = HttpClient(Android) {
    install(ContentNegotiation) {
        json() // kotlinx.serialization 사용
    }
    install(Logging) {
        level = LogLevel.ALL
    }
    install(Auth) {
        bearer {
            loadTokens { BearerTokens("your_token", "") }
        }
    }
}

suspend fun fetchData(): MyData {
    return client.get("https://api.example.com/data").body()
}
```

#### 특징  
- 인터셉터 개념이 유연해서 요청/응답 처리 커스터마이징 가능
- HTTP/2, WebSocket등 최신 프로토콜 지원  

#### ❓ 실전 질문
#### 앱에서 동시에 여러 API를 요청하고 수행하고 UI를 업데이트하기 전에 결과를 결합해야 한다고 가정했을떄  
#### Retrofit과 코루틴을 사용하여 이를 효율적으로 구현하려면 어떻게 해야 하나요?  

보통 API를 요청하고 결합하는것은 Repository단에서 작업함  
그러므로 여러개의 api를 요청하는것을 async로 감싸서 작업하면  
네트워크 요청 시간을 줄일 수 있음  

```kotlin
suspend fun getUserData() {
  supervisorScope {
    val a = async { apiService.getData1() }
    val b = async { apiService.getData2() } 
    val c = apiService.getData3() 
    User(
      a = a.await(),
      b = b.await(),
      c = c
    )
  }
}
```

#### ❓ 실전 질문  
#### API 응답 실패 시 어떻게 처리하고, 재시도 매커니즘은 어떻게 구현하나요?  

보통 멀티모듈환경에서 작업하므로 HttpException을 커스텀 런타임 Exception으로 변환해서 Presentation Layer에 던져준다.  
401일 경우 authenticator나 Interceptor를 통해서 재시도한다  

#### 💡 Pro Tips for Mastery : OkHttp Authenticator 및 Interceptor를 사용하여 OAuth 토큰 갱신하기  
OAuth로 보호되는 API로 작업할 때 토큰 만료 및 갱신 시나리오를 처리하는 것이 일반적입니다.  
OkHttp는 토큰을 가로채고 새로 고치는 두 가지 주요 매커니즘을 Authenticator와 Interceptor를 제공합니다.  
둘 다 다른 목적을 가지고 있으며 애플리케이션의 특정 요구사항에 따라 선택적으로 사용할 수 있습니다.  

#### ⚙️ OkHttp Authenticator  
OkHttp의 Authenticator 인터페이스는 토큰 만료와 같은 인증 문제를 처리하도록 따로 설계되었습니다.  
서버가 401 Unauthorized 상태 코드를 응답하면 Authenticator가 호출되어 업데이트된 인증 자격 증명이 포함된 새 요청을 제공합니다.  

```kotlin
class TokenAuthenticator(
    private val tokenProvider: TokenProvider // 토큰 제공 및 새로고침 로직 캡슐화
) : Authenticator {

    override fun authenticate(route: Route?, response: Response): Request? {
        // 이전 요청이 이미 인증 헤더를 가지고 있었는지 확인 (무한 루프 방지)
        val previousToken = response.request.header("Authorization")

        // 토큰 동기화 및 새로고침 (한 번만 시도하도록 제어)
        synchronized(this) {
            // 현재 토큰과 이전 요청의 토큰 비교
            val currentToken = tokenProvider.getToken() // 현재 저장된 토큰 가져오기

            // 토큰이 변경되지 않았거나 새로고침 실패 시 null 반환 (더 이상 재시도 안 함)
            if (previousToken != null && previousToken == "Bearer $currentToken") {
                val newToken = tokenProvider.refreshToken() // 동기적으로 토큰 새로고침 시
                if (newToken == null) {
                    // 새로고침 실패 시 처리 (예: 로그아웃)
                    tokenProvider.clearToken() // 토큰 제거
                    return null // 인증 실패
                }
            }

            // 새로고침된 토큰 또는 현재 토큰으로 새 요청 생성
            val refreshedToken = tokenProvider.getToken()
            if (refreshedToken != null) {
                return response.request.newBuilder()
                    .header("Authorization", "Bearer $refreshedToken")
                    .build()
            }
        }
        return null // 토큰 없으면 인증 불가
    }
}

TokenProvider 인터페이스/클래스 (예시)
  interface TokenProvider {
  fun getToken(): String?
  fun refreshToken(): String? // 동기 방식으로 구현 필요
  fun clearToken()
   ... (토큰 저장/관리 로직)
}
```

TokenProvider는 일반적으로 새로 고침 엔드포인트에 동기 네트워크  
호출을 하여 토큰을 새로 고치는 역할을 하는 커스텀 클래스입니다.  

Authenticator를 사용하려면 OkHttpClient를 생성할 때 아래와 같이 설정이 필요합니다.  

```kotlin
val okHttpClient = OkHttpClient.Builder()
    .authenticator(TokenAuthenticator(tokenProvider))
    // ... 다른 설정 ...
    .build()
```

```kotlin
#### ⚙️ OkHttp Interceptor 사용하기  
Interceptor는 토큰 추가 및 갱신 로직을 처리할 수 있는 더 유연한 접근 방식입니다.  
Authenticator와 달리 Interceptor는 요청 또는 응답이 처리되기 전에 가로채서 수정할 수 있습니다.  

class TokenInterceptor(
    private val tokenProvider: TokenProvider
) : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        // 현재 토큰 가져오기
        val token = tokenProvider.getToken()
        // 요청에 토큰 추가 (토큰이 있는 경우)
        val originalRequest = chain.request()
        val requestWithToken = if (token != null) {
            originalRequest.newBuilder()
                .header("Authorization", "Bearer $token")
                .build()
        } else {
            originalRequest // 토큰 없으면 원본 request 사용
        }

        // 토큰 요청
        var response = chain.proceed(requestWithToken)

        // 토큰 만료 확인 (401 응답)
        if (response.code == 401) {
            // 동기화 블록으로 토큰 새로고침 중복 방지
            synchronized(this) {
                val newToken = tokenProvider.refreshToken() // 동기적으로 토큰 새로고침
                if (newToken != null) {
                    // 새 토큰으로 request 다시 만들기
                    val newRequest = requestWithToken.newBuilder()
                        .header("Authorization", "Bearer $newToken")
                        .build()
                    // 이전 응답 닫기 (리소스 누수 방지)
                    response.close()
                    // 새롭게 만들어진 request로 재시도
                    response = chain.proceed(newRequest)
                } else {
                    // 갱신 실패 시 처리 (로그아웃 등)
                    tokenProvider.clearToken()
                }
            }
        }
        return response
    }
}
```

#### ⚙️ Authenticator와 Interceptor의 주요 차이점  
1. 목적  
  - Authenticator : 401 응답에 의해 트리거되는 인증 문제를 처리하도록 설계되었습니다.
  - Interceptor : 요청 및 응답 처리 모두에 더 세분화된 제어를 처리할 수 있습니다.  
2. 자동적으로 트리거  
  - Authenticator : 401 응답에 대해 자동으로 호출  
  - Interceptor : 특정 시나리오를 감지하고 처리하기 위한 수동 로직이 필요  
3. 사례  
  - Authenticator : 간단한 인증 문제
  - Interceptor : 복잡한 요청/응답 처리가 필요한 시나리오


#### 💡 Pro Tips for Mastery : Retrofit CallAdapter란?  
개발자가 Retrofit API 메서드의 반환 타입을 사용자 정의할 수 있도록 하는 추상화 API  
기본적으로는 Call<T> 객체를 반환하지만  
CallAdapter를 사용함녀 이 기본 반환 타입을 LiveData, Flow, RxJava 또는  
커스텀 타입과 같은 다른 타입으로 반환할 수 있습니다.  

#### 🛠️ CallAdapter 작동 방식  
Retrofit은 CallAdapter 인스턴스를 생성하기 위해 CallAdapter.Factory를 사용  
CallAdapter는 런타임에 Call<T> 객체를 원하는 타입으로 변환하는 역할을 합니다.  
이 프로세스는 Retrofit이 API 인터페이스에 대한 프록시를 생성할 때 사용됩니다.  

#### 🛠️ Retrofit의 기본 CallAdapter  
다른 타입을 원하면 적절한 라이브러리를 사용하거나, 커스텀 CallAdapter를 작성해야 합니다.  

Retrofit은 내부적으로 Kotlin Coroutines Adapter를 사용하여  
원래 기본 타입인 Call<List<User>>에서 List<User>를 반환하는 suspend 함수로 변환합니다.  

#### 🛠️ 커스텀 CallAdapter  

```kotlin
import androidx.lifecycle.LiveData
import retrofit2.*
import java.lang.reflect.ParameterizedType
import java.lang.reflect.Type
import java.util.concurrent.atomic.AtomicBoolean

// LiveData를 반환하는 CallAdapter 구현
class LiveDataCallAdapter<R>(private val responseType: Type) : CallAdapter<R, LiveData<R>> {

    override fun responseType(): Type = responseType

    override fun adapt(call: Call<R>): LiveData<R> {
        return object : LiveData<R>() {
            private var started = AtomicBoolean(false)

            override fun onActive() {
                super.onActive()
                if (started.compareAndSet(false, true)) { // 한 번만 실행되도록 보장
                    call.enqueue(object : Callback<R> {
                        override fun onResponse(call: Call<R>, response: Response<R>) {
                            postValue(response.body()) // postValue 사용 (백그라운드 스레드 실행 가능)
                        }

                        override fun onFailure(call: Call<R>, t: Throwable) {
                            // 오류 처리: null 또는 특정 오류 상태 전달
                            postValue(null)
                        }
                    })
                }
            }
        }
    }
}

// LiveDataCallAdapter 인스턴스를 생성하는 Factory
class LiveDataCallAdapterFactory : CallAdapter.Factory() {
    override fun get(
        returnType: Type,
        annotations: Array<Annotation>,
        retrofit: Retrofit
    ): CallAdapter<*, *>? {
        // 반환 타입이 LiveData인지 확인
        if (getRawType(returnType) != LiveData::class.java) {
            return null
        }

        // LiveData의 제네릭 타입 추출 (LiveData<List<User>> 에서 List<User> 추출)
        val observableType = getParameterUpperBound(0, returnType as ParameterizedType)
        val rawObservableType = getRawType(observableType)

        // 제네릭 타입이 Response인 경우 처리 (선택 사항)
        if (rawObservableType == Response::class.java) {
            if (observableType !is ParameterizedType) {
                throw IllegalArgumentException("Response must be parameterized as Response<Foo> or Response<? extends Foo>")
            }
            val bodyType = getParameterUpperBound(0, observableType)
            return LiveDataCallAdapter<Any>(bodyType) // Response<T>의 T 타입 사용
        }

        // 일반 LiveData<T> 반환 타입 처리
        return LiveDataCallAdapter<Any>(observableType)
    }
}

// Retrofit 생성 시 커스텀 어댑터 사용 예시
val retrofit = Retrofit.Builder()
    .baseUrl("https://example.com/")
    .addConverterFactory(GsonConverterFactory.create()) // Converter Factory 먼저 추가
    .addCallAdapterFactory(LiveDataCallAdapterFactory()) // CallAdapter Factory 추가
    .build()
```

## 📚 Q62. 대규모 데이터 셋을 효율적으로 로드하는데 왜 페이징 기법이 필요하고  
## RecyclerView로 구현해 본 경험이 있나요?

### 🎯 개요  
페이징 시스템은 대규모 데이터 셋을 처리할 때 데이터 로드 및 화면에  
렌더링 하는 방식을 최적화합니다.

데이터를 더 작은 페이지로 로드하면 메모리 사용량이 크게 줄어들어  
잠재적인 메모리 부족 문제를 방지할 수 있습니다.  
또한 한 번에 많은 데이터를 로드하는 것이 아니라  
현재 화면에 보여지는데 필요한 데이터만 가져와 렌더링 하므로 초기 로드 시간이 단축됩니다.  
네트워크 사용량이 최소화되어 특히 대역폭이 제한된 시나리오에서 리소스를 효율적으로 사용할 수 있습니다.

#### 🛠️ 페이징 시스템 직접 구현  

1. RecyclerView.Adapter 및 ViewHolder 생성  
첫 번째 단계는 해당 ViewHolder와 함께 RecyclerView.Adapter 또는 ListAdapter를 생성하는 것입니다.  
해당 구성 요소는 데이터 셋을 RecyclerView에 관리하고 바인딩하기 위해 필수입니다.  
Adapter는 데이터 소스를 처리하고 ViewHolder는 데이터 셋의 아이템이 개별적으로 렌더링 되는 방식을 정의합니다.  

```kotlin
class PokedexAdapter : ListAdapter<Pokemon, PokedexAdapter.PokedexViewHolder>(diffUtil) {

    override fun onCreateViewHolder(
        parent: ViewGroup,
        viewType: Int
    ): PokedexViewHolder {
        val binding = ItemPokemonBinding.inflate(LayoutInflater.from(parent.context), parent, false) // parent 전달
        return PokedexViewHolder(binding)
    }

    override fun onBindViewHolder(holder: PokedexViewHolder, position: Int) {
        // getItem()으로 안전하게 아이템 가져오기
        holder.bind(getItem(position))
    }

    inner class PokedexViewHolder(
        private val binding: ItemPokemonBinding
    ) : RecyclerView.ViewHolder(binding.root) {
        fun bind(pokemon: Pokemon) {
            // 데이터 바인딩 로직
        }
    }

    companion object {
        private val diffUtil = object : DiffUtil.ItemCallback<Pokemon>() {
            override fun areItemsTheSame(oldItem: Pokemon, newItem: Pokemon): Boolean =
                oldItem.name == newItem.name

            override fun areContentsTheSame(oldItem: Pokemon, newItem: Pokemon): Boolean =
                oldItem == newItem
        }
    }
}
```

2. RecyclerView에 addOnScrollListener 추가  
RecyclerView에 addOnScrollListener를 구현하여 스크롤 상태를 트래킹합니다.  
화면에서 마지막으로 보이는 항목이 데이터 셋 끝에 가까워지면 네트워크 또는 데이터베이스에서 다음 데이터 셋에 대한 로드를 트리거합니다.  

```kotlin
recyclerView.addOnScrollListener(object : RecyclerView.OnScrollListener() {
    override fun onScrolled(recyclerView: RecyclerView, dx: Int, dy: Int) {
        super.onScrolled(recyclerView, dx, dy)
        // 레이아웃 매니저 가져오기 (LinearLayoutManager 또는 GridLayoutManager)
        val layoutManager = recyclerView.layoutManager ?: return // Null 체크

        val lastVisiblePosition = when (layoutManager) {
            is LinearLayoutManager -> layoutManager.findLastVisibleItemPosition()
            is GridLayoutManager -> layoutManager.findLastVisibleItemPosition()
            else -> return // 다른 레이아웃 매니저는 처리하지 않음
        }

        val totalItemCount = layoutManager.itemCount
        val threshold = 4 // 미리 로드할 아이템 개수 (임계값)

        // 마지막 아이템에 가까워지고 로딩 중이 아닐 때 다음 페이지 미리 로드
        if (lastVisiblePosition + threshold >= totalItemCount && totalItemCount > 0 && !viewModel.isLoading.value) {
            viewModel.loadNextPage()
        }
    }
})
```

3. RecyclerView.Adapter에 새 데이터 셋 추가  
성공적으로 트리거 되면 RecyclerView.Adapter의 기존 데이터에 추가합니다.  


#### 🧠 요약  
데이터를 페이지로 분할하고, 스크롤 이벤트를 관찰하고  
어댑터를 동적으로 업데이트하여 직접 페이징 시스템을 만들 수 있습니다.  
이를 구현하는 다양한 방법이 있지만 Jetpack Paging 라이브러리를 활용하면 또 다른 접근 방식으로 페이징 시스템을 구현할 수 있습니다.

## 📚 Q63. 네트워크에서 이미지를 어떻게 가져오고 렌더링하나요?  

### 🎯 개요  
이미지 로딩은 네트워크에서 가져온 콘텐츠를 표시하는 작업과 같이  
애플리케이션 개발에 있어서 흔히 요구되는 작업입니다.  
네트워크 요청, 이미지 크기 조절, 캐싱, 렌더링 및 효율적인 메모리 관리와 같은 복잡한 기능을 구현해야 합니다.  

Glide, Coil, Fresco와 같은 라이브러리를 활용할 수 있습니다.  

#### ⚙️ Glide  
Glide는 오랫동안 인기를 끌어 온 빠르고 효율적인 이미지 로딩 라이브러리입니다.  
캐싱, 플레이스홀더, 이미지 및 이미지 변환과 같은 복잡한 시나리오를 처리하는 데 이상적입니다.  

```kotlin
Glide.with(context)
    .load("https://example.com/image.jpg") // 이미지 URL
    .placeholder(R.drawable.placeholder) // 로딩 중 표시할 이미지
    .error(R.drawable.error_image) // 오류 시 표시할 이미지
    .circleCrop() // 원형 자르기 변환 (선택 사항)
    .transition(DrawableTransitionOptions.withCrossFade()) // 페이드 효과 (선택 사항)
    .into(imageView) // 대상 ImageView
```

Glide는 자동으로 이미지를 캐시하여 네트워크 호출을 최적화하고 성능을 향상시킵니다.  
애니메이션 GIF 지원, 플레이스홀더, 변환, 캐싱, 리소스 재사용과 같은 유용한 기능을 제공합니다.

#### ⚙️ Coil  
Coil은 Kotlin Multiplatform으로 설계된 100% Kotlin 기반의 이미지 로딩 라이브러리입니다.  
내부적으로 코루틴을 활용하고 Jetpack Compose와 같은 최신 기능을 지원합니다  
Coil은 내부적으로 OkHttp및 코루틴과 같이 안드로이드 프로젝트에서 이미 널리 사용되는  
라이브러리를 사용하기 때문에 기존 프로젝트에 원활하게 통합될 수 있습니다.  

```kotlin
import coil.load
import coil.transform.CircleCropTransformation

imageView.load("https://example.com/image.jpg") {
    crossfade(true) // 크로스페이드 효과
    placeholder(R.drawable.placeholder)
    error(R.drawable.error_image)
    transformations(CircleCropTransformation()) // 원형 자르기 변환
    // size(Size.ORIGINAL) // 원본 크기 로드 (선택 사항)
    // memoryCachePolicy(CachePolicy.ENABLED) // 메모리 캐시 정책 설정
    // diskCachePolicy(CachePolicy.ENABLED) // 디스크 캐시 정책 설정
}
```

Coil은 Kotlin 및 Jetpack Compose와 원활하게 통합되며  
이미지 변환, 애니메이션 GIF 지원, SVG 렌더링, 비디오 프레임 추출과 같은 유용한 기능을 제공합니다.  
가벼운 특성 덕분에 최신 안드로이드 프로젝트에 훌륭한 선택입니다.  

Coil은 Jetpack Compose, Kotlin Multiplatform 및 기타 최신 안드로이드 솔루션에 대한 다양한 지원을 제공합니다.  

#### ⚙️ Fresco  
Fresco는 Meta에서 개발한 이미지 로딩 라이브러리로  
조금 더 복잡한 사용 사례를 위해 설계되었습니다.  
이미지를 디코딩하고 표시하기 위해 자체 파이프라인을 사용하는 접근 방식을 사용합니다.  

```kotlin
// Fresco 초기화 (Application 클래스에서)
// Fresco.initialize(this);

// 레이아웃에서 SimpleDraweeView 사용
val draweeView: SimpleDraweeView = findViewById(R.id.drawee_view)
val uri = Uri.parse("https://example.com/image.jpg")
draweeView.setImageURI(uri)

// 또는 Controller를 사용하여 더 세밀하게 제어
// val controller = Fresco.newDraweeControllerBuilder()
//     .setUri(uri)
//     .setOldController(draweeView.controller)
//     .build()
// draweeView.controller = controller
```

Fresco는 자체적으로 SimpleDraweeView라는 커스텀 View를 제공합니다.  

```xml
<com.facebook.drawee.view.SimpleDraweeView
    xmlns:fresco="http://schemas.android.com/apk/res-auto" <!-- 네임스페이스 추가 -->
    android:id="@+id/drawee_view"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    fresco:placeholderImage="@drawable/placeholder" <!-- 플레이스홀더 설정 -->
    fresco:failureImage="@drawable/error_image" <!-- 오류 이미지 설정 -->
    fresco:progressBarImage="@drawable/loading_spinner" <!-- 로딩 스피너 설정 -->
/>
```
Fresco는 이미지 파이프라인, 드로위, 최적화된 메모리 관리, 고급 로딩 매커니즘, 스트리밍 및 애니메이션과 같은 괜찮은 기능을 제공합니다.  

#### 🧠 요약  
- Glide : 유연하고 네트워크 이미지를 쉽게 처리하는 데 널리 사용됩니다.  
  Jetpack Compose를 지원하지만 몇 년 동안 베타 상태로 남아있습니다.  
- Coil : Kotlin 중심적이고 가벼우며 최신 안드로이드 프로젝트에 원활하게 통합됩니다.  
  Jetpack Compose 및 Kotlin Multiplatform을 지원합니다.  
- Fresco : 메모리 집약적인 시나리오에 적합하며 점진적 이미지 로딩, 이미지 파이프라인 및 더 복잡한 작업과 같은 고급 기능을 처리하는 데 효율적입니다.  

## 📚 Q64. 로컬 디바이스에서 데이터를 저장하고 복원하는 방벙베 대해서 설명해주세요  

### 🎯 개요  
안드로이드는 경량화된 키-값 기반의 데이터 저장  
구조화된 데이터베이스 구축 및 쿼리 또는 로컬 파일 처리 등 각 시나리오에 적합한 데이터 저장 메커니즘을 제공합니다.  

#### ⚙️ SharedPreferences  
SharedPreferences는 앱 내 설정이나 사용자 환경 설정과 같은  
가벼운 값 가장 적합한 키-값 쌍 형태의 데이터 저장 메커니즘입니다.  
원시타입을 저장하고 앱 재시작 시에도 유지할 수 있습니다.  

SharedPreferences는 동기적으로 메인 스레드를 차단하는 문제가 있고  
최근 비동기 처리 함수 등을 제공하는 DataStore의 등장으로 인해 최신 애플리케이션에는 덜 선호되고 있습니다.  

```kotlin
import androidx.core.content.edit // KTX 확장 함수 사용

val sharedPreferences = context.getSharedPreferences("app_prefs", Context.MODE_PRIVATE)
// KTX 확장 함수를 사용하여 간단하게 데이터 수정 후 반영
sharedPreferences.edit {
    putString("user_name", "skydoves")
    putInt("user_score", 100)
    // apply()는 비동기, commit()은 동기
    // KTX edit {} 블록은 자동으로 apply() 호출
}

// 값 읽기
val userName = sharedPreferences.getString("user_name", null) // 기본값 지정
```

#### ⚙️ DataStore  
Jetpack DataStore는 SharedPreferences를 대체하는 더 모던하고 효율적인 방법입니다.  
키-값 저장을 위한 PreferencesDataStore와 구조화된 객체 값 데이터를 저장하기 위한 ProtoDataStore의 두 유형을 제공합니다.  
SharedPreferences와 달리 DataStore는 비동기식이므로 메인 스레드를 차단하는 잠재적인 문제를 방지합니다.  

```kotlin
import androidx.datastore.preferences.core.edit
import androidx.datastore.preferences.core.stringPreferencesKey
import androidx.datastore.preferences.preferencesDataStore
import kotlinx.coroutines.flow.first
import kotlinx.coroutines.flow.map
import kotlinx.coroutines.runBlocking

// Context의 확장 함수로 DataStore 인스턴스 생성 (싱글톤 권장)
val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "settings")

// 데이터 저장을 위한 키 정의
val USER_NAME_KEY = stringPreferencesKey("user_name")

// 데이터 쓰기 (코루틴 내에서 수행)
suspend fun saveUserName(context: Context, name: String) {
    context.dataStore.edit { settings ->
        settings[USER_NAME_KEY] = name
    }
}

// 데이터 읽기 (Flow 사용)
fun getUserNameFlow(context: Context): Flow<String?> {
    return context.dataStore.data
        .map { preferences ->
            preferences[USER_NAME_KEY] // 키에 해당하는 값 반환 (없으면 null)
        }
}

// 값 저장 및 복원 예시
fun exampleUsage(context: Context) {
    viewModelScope.launch {
        saveUserName(context, "John Doe")
        val name = getUserNameFlow(context).first() // Flow에서 첫 번째 값 가져오기
        Log.d("DataStore", "User name: $name")
    }
}
```

#### ⚙️ Room Database  

Room Database는 구조화되고 관계형 데이터를 처리하도록 설계된 SQLite를 수준 높게 추상화한 솔루션입니다.  
어노테이션, 컴파일 타임 검사, 반응형 프로그래밍을 위한 LiveData 또는 Flow 지원을 통해 데이터베이스 관리를 매우 단순화합니다.  
Room은 복잡한 쿼리나 대량의 구조화된 데이터 저장이 필요한 앱에 이상적입니다.  

```kotlin
// 데이터 엔티티 정의
@Entity(tableName = "users") // 테이블 이름 지정 (선택 사항)
data class User(
    @PrimaryKey val id: Int,
    @ColumnInfo(name = "user_name") val name: String // 컬럼 이름 지정 (선택 사항)
)

// 데이터 접근 객체(DAO) 정의
@Dao
interface UserDao {
    // suspend 함수로 비동기 처리
    @Insert(onConflict = OnConflictStrategy.REPLACE) // 크래시 발생 시 대체 전략
    suspend fun insertUser(user: User)

    @Query("SELECT * FROM users WHERE id = :userId") // 명확한 파라미터 이름 사용
    suspend fun getUserById(userId: Int): User? // Nullable 반환 타입

    @Query("SELECT * FROM users ORDER BY user_name ASC")
    fun getAllUsers(): Flow<List<User>> // Flow를 사용하여 데이터 변경 관찰
}

// 데이터베이스 클래스 정의
@Database(entities = [User::class], version = 1, exportSchema = false) // 스키마 내보내기 비활성화 (선택 사항)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao

    // 싱글톤 인스턴스 제공 (권장)
    companion object {
        @Volatile
        private var INSTANCE: AppDatabase? = null

        fun getInstance(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "app_database" // 데이터베이스 파일 이름
                )
                    // .addMigrations(MIGRATION_1_2) // 마이그레이션 추가 (필요시)
                    .build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

#### ⚙️ File Storage  

바이너리 또는 커스텀 데이터의 경우 안드로이드 내부 또는 외부 저장소에 파일을 저장할 수 있도록 합니다.  
외부 저장소는 다른 앱과 공유할 수 있습니다.  
파일 I/O 작업은 이미지, 비디오 또는 커스텀 직렬화된 데이터 저장과 같은 작업에 사용할 수 있습니다.  

```kotlin
// 내부 저장소의 앱별 디렉토리에 파일 생성
val file = File(context.filesDir, "user_data.txt")
try {
    FileOutputStream(file).use { fos ->
        fos.write("Sample user data".toByteArray())
    }
    // 또는 BufferedWriter 사용
    // file.bufferedWriter().use { out ->
    //     out.write("Sample user data")
    // }
} catch (e: IOException) {
    e.printStackTrace()
    // 오류 처리
}

// 파일 읽기 (예시)
try {
    val content = file.readText()
    Log.d("FileStorage", "File content: $content")
} catch (e: IOException) {
    e.printStackTrace()
}
```

#### 🧠 요약  
SharedPreferences 또는 DataStore는 사용자 설정이나 기능 플래그와 같은 가벼운 키-값 데이터를 저장하는 데 이상적입니다.  
Room은 구조화된 쿼리를 사용하여 복잡한 관계형 데이터를 관리하는 데 적합합니다.  
File Storage는 바이너리 파일이나 대규모 커스텀 데이터셋을 처리하는 데 가장 적합합니다.  

#### ❓ 실전 질문
#### 오프라인 접근을 위해 네트워크 API에서 받은 대용량 JSON 응답을 저장해야 하는 시나리오에서는 어떤 매커니즘이 적합한가?  
만약 단순히 저장/복원 할 경우에는 Fire Storage가 괜찮지만  
Room이 적합하다  
Json을 파싱해서 엔티티로 저장하면 쿼리/페이징을 지원하므로 오프라인 조회에 적합하다  

## 📚 Q65. 오프라인 우선 아키텍처를 어떻게 설계하실 건가요?  

### 🎯 개요  
오프라인 우선 디자인은 애플리케이션이 로컬에 캐시되거나 저장된 데이터에 의존하여  
활성 네트워크 없이도 앱이 원활하게 작동하도록 보장합니다.  
인터넷이 끊기는 시나리오더라도 데이터를 로컬에 캐시하거나 저장하고 연결이 복원되면 원격 서버와 동기화하여  
매끄러운 경험을 제공합니다.  

#### ⚙️ 오프라인 우선 아키텍처의 핵심 개념  

1. 로컬 데이터 지속성  
  - 신뢰할 수 있는 오프라인 우선 전략은 로컬 데이터 저장소에서 시작됩니다.
  - Room Database는 구조화된 로컬 데이터를 관리하기 위해 권장되는 솔루션입니다.
2. 데이터 동기화
  - 로컬 데이터와 원격 데이터 간의 동기화는 일관성을 보장합니다.
  - WorkManager는 이를 위한 훌륭한 선택 중 하나로  
    네트워크 연결과 같은 조건이 충족될 때 지연된 동기화 작업이 실행되도록 합니다.
3. 캐시 및 가져오기 정책(Cache and Fetch Politices)  
  - 아래와 같이 데이터 캐싱 및 가져오기에 대한 명확한 정책을 정의해야 합니다.
  - 캐싱 데이터 읽기 : 앱이 먼저 로컬 저장소에서 데이터를 가져오고 필요할 때만 네트워크에 새로운 데이터를 요청합니다.
  - 캐싱 데이터 쓰기 : 업데이트가 로컬에 기록되고 백그라운드에서 서버와 동기화됩니다.
4. 충돌 해결
  - 로컬 소스와 원격 소스 간에 데이터를 동기화할 때 충돌 해결 전략을 구현해야 합니다.
  - 최신 데이터 우선 : 가장 최근 변경 사항을 우선시합니다.
  - 사용자 정의 : 사용자가 수동으로 충돌을 해결하거나 도메인별 규칙을 적용하도록 허용합니다.

```kotlin
// 데이터 엔티티 (동기화 상태 플래그 포함)
@Entity
data class Article(
    @PrimaryKey val id: Int,
    val title: String,
    val content: String,
    val isSynced: Boolean = false // 서버와 동기화되었는지 여부
)

// DAO 인터페이스
@Dao
interface ArticleDao {
    @Query("SELECT * FROM Article")
    fun getAllArticles(): Flow<List<Article>> // Flow로 변경 사항 관찰

    // 로컬에만 있는 미동기화된 Article 조회
    @Query("SELECT * FROM Article WHERE isSynced = 0")
    suspend fun getUnsyncedArticles(): List<Article>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertArticle(article: Article)

    // 동기화 완료 후 상태 업데이트
    @Query("UPDATE Article SET isSynced = 1 WHERE id = :articleId")
    suspend fun markArticleAsSynced(articleId: Int)
}

// 동기화 Worker
class SyncWorker(appContext: Context, params: WorkerParameters) : CoroutineWorker(appContext, params) {
    override suspend fun doWork(): Result {
        // 데이터베이스 및 DAO 인스턴스 가져오기 (의존성 주입 권장)
        val articleDao = AppDatabase.getInstance(applicationContext).articleDao()

        return try {
            // 미동기화된 데이터 가져오기
            val unsyncedArticles = articleDao.getUnsyncedArticles()

            if (unsyncedArticles.isNotEmpty()) {
                // 서버와 동기화 시도
                if (syncToServer(unsyncedArticles)) {
                    // 성공 시 로컬 상태 업데이트
                    unsyncedArticles.forEach {
                        articleDao.markArticleAsSynced(it.id)
                    }
                    Log.d("SyncWorker", "Sync successful for ${unsyncedArticles.size} articles.")
                } else {
                    Log.e("SyncWorker", "Sync failed.")
                    // 재시도 로직은 WorkManager가 처리하거나 여기서 명시적으로 Result.retry() 반환
                    return Result.retry()
                }
            } else {
                Log.d("SyncWorker", "No articles to sync.")
            }
            Result.success()
        } catch (e: Exception) {
            Log.e("SyncWorker", "Sync failed with error", e)
            Result.failure() // 실패로 간주
        }
    }

    // 서버와 동기화하는 실제 로직 (네트워크 호출 등)
    private suspend fun syncToServer(articles: List<Article>): Boolean {
        // 실제 네트워크 요청 및 결과 처리 로직 구현
        Log.d("SyncWorker", "Attempting to sync ${articles.size} articles...")
        // 아래 예시에서는 가짜로 지연시키고 성공을 반환하고 있는데,
        // 실제 구현에서는 네트워크 요청 로직이 들어감
        kotlinx.coroutines.delay(2000)
        return true // 실제 구현에서는 API 호출 결과에 따라 반환
    }
}
````

요약
1. 백그라운드 동기화 관리를 위해 WorkManager를 사용합니다.
2. 로컬 데이터 저장을 위해 Room을 활용합니다.
3. 효율적인 데이터 업데이트를 위해 명확한 캐시 정책을 정의합니다.  
4. 데이터 일관성을 보장하기 위해 충돌 해결 매커니즘을 구현합니다.

#### 🧠 요약  
안드로이드의 오프라인 우선 접근 방식은 사용자가 인터넷 연결 상태에  
관계없이 원활한 기능을 보장할 수 있도록 합니다.  
Room, WorkManager 및 적절한 캐싱 전략과 같은 도구를 활용하여 일관된 사용자 경험을 유지할 수 있습니다.  

## 📚 Q66. 초기 데이터 로딩을 위한 작업을 Compose의 LaunchedEffect와 ViewModel.init()중 어디에서 하는 것이 이상적인가요?

### 🎯 개요  
안드로이드 개발에서 자주 논의되는 주제는 초기 데이터를 Composable의 LaunchedEffect에서 로드할지  
아니면 ViewModel의 init() 블록 내에서 로드할지 여부입니다.  

공식 안드로이드 문서 및 architecture-samples Github의 예제는 일반적으로 구성 변경 시 더 나은 생명주기 관리  
및 데이터 지속성을 위해 ViewModel.init() 내에서 데이터를 로드할 것을 예시로 보여주고 있습니다.  

설문 조사 결과 62.6 : 37.4로 ViewModel.init()이 우세합니다.  

#### ⚙️ ViewModel.init()이 더 낫다  
어떤 안드로이드 개발자는 다음과 같이 설명했습니다.  
Jetpack Compose UI를 애플리케이션의 상태 또는 데이터의 시각적 표현으로 간주한다면  
앱에 무엇을 해야 할지 지시하기 위해 UI에 의존한다는 것을 사실 설계 결합으로 볼 수 있습니다.  
그래서 LaunchedEffect를 이용해서 UI에서 데이터를 가져오는 것 보다  
ViewModel.init()을 사용하는 것이 더 나은 관심사 분리를 제공하여 비즈니스 로직과 UI 상태 관리가 분리되도록 하는게 낫습니다.  

#### ⚙️ 유연하게 사용하자  
또 다른 안드로이드 개발자는 다음과 같은 관점을 공유했습니다.  
ViewModel.init에만 의존하면 특정 동작이 트리거 되는 시점에 대한  
제어가 어려워지고 유닛 테스트가 복잡해질 수 있다는 것을 명심해야 합니다.  
대신 ViewModel 내에서 이벤트 기반 흐름을 관찰하여 트리거되고 지연될 수 있는  
독립적인 함수를 정의하는 것을 선호합니다.  
해당 접근 방식은 더 큰 유연성과 제어권을 제공하여 LaunchedEffect 또는 이벤트 트리거 작업과 같은  
메서드가 데이터 로딩을 더 효과적으로 관리할 수 있다고 합니다.  
ViewModel.init 블록에 전적으로 의존하는 대신  
사용자 상호 작용이나 특정 이벤트를 기반으로 데이터를 다시 가져올 때 효율성이 향상된다는 것입니다.  

#### ⚙️ 둘 다 안티패턴이다 Lazy Observation을 사용하자  
두 솔루션 모두 주목할 만한 단점이 있습니다.  
Google의 Android Toolkit 팀의 Ian Lake는 두 접근 방식 모두 안드로이드 개발에서 안티패턴으로  
간주된다고 지적하며 초기 데이터 로딩에 대한 다른 대안이 필요함을 시사했습니다.  

ViewModel.init()에서 초기 데이터를 로드하면 ViewModel 생성 중에 의도하지 않은  
사이드 이펙트가 발생하고, UI 상태 관리라는 의도된 역할에서 벗어나 생명주기 처리를 복잡하게 만들 수 있습니다.  

마찬가지로 LaunchedEffect 내에서는 데이터를 초기화하면 매 첫 컴포지션마다 반복적으로 트리거 될 위험이 있습니다.  
이는 ViewModel의 생명주기가 일반적으로 Composable 함수의 생명주기 보다 길기 때문에  
Composable 함수가 새로 컴포지션에 진입할 때마다 동일한 ViewModel 인스턴스에 대해서 동일한 비즈니스 로직을 반복적으로  
트리거할 가능성이 있습니다.  
따라서, 생명주기 불일치로 예기치 않은 동작이 발생하고 의도된 흐름을 방해할 수 있습니다.  

이러한 우려를 해결하기 위해서 지연 초기화를 위해 cold flows 사용을 권장합니다.  
이 접근 방식에서 Flow는 수집되기 시작할 때만 네트워크 요청이나 데이터베이스 쿼리와 같은 비즈니스 로직을 실행합니다.  
UI 레이어에서 구독자가 있을 때까지 Flow는 비활성 상태로 유지되어 불필요한 작업이 수행되지 않도록 보장합니다.

#### 🛠️ 지연 관찰 모범 사례  
```kotlin
val pokemon : StateFlow<Pokemon?> = savedStateHandle.getStateFlow("pokemon", null)

val pokemonInfo: StateFlow<PokemonInfo?> = pokemon.filterNotNull().flatMapLatest {
    pokemon -> 

    detailsRepository.fetchPokemonInfo(
        name = pokemon.name,
        onComplete = { isLoading = false },
        onError = { errorMessage = it }
    )
}.stateIn(
    scope = viewModelScope,
    started = SharingStarted.WhileSubscribed(5_000),
    initialValue = null
)
```
위 코드에서 업스트림 호출(detailsRepository.fetchPokemonInfo())은 첫 번째 구독자가  
Flow 수집을 시작할 때만 트리거 됩니다. 그런 다음 결과는 캐시 되어 stateIn 메서드를  
사용하여 상태로 변환되어 효율적인 데이터 관리와 중복 작업 최소화를 보장합니다.  
stateIn은 콜드 플로우를 핫 플로우인 StateFlow로 변환하여, 일부 상태 값에 대한 업데이트를 제공합니다.  

#### 🧠 요약  

안드로이드에서 초기 데이터를 로드하는 방법에는 LaunchedEffect, ViewModel.init(), 콜드 플로우를 통한 지연 관찰 등 여러 가지가 있습니다.  
이 내용의 디스커션은 효율성을 높이고 부작용을 피하기 위해 콜드 플로우 활용을 제안하는 것으로 끝났습니다.  
그러나 늘 그렇듯 모든 상황에서 완벽한 해결책이라는 것은 없습니다.  

#### ❓ 실전 질문
#### ViewModel.init() 또는 LaunchedEffect에서 초기 데이터를 로드하는 것의 장단점은 무엇이며  
#### 언제 어떤 접근 방식을 사용하시나요? 만약 다른 접근법을 선호한다면 어떤 것이 있나요?  

저는 주로 ViewModel.init()을 사용합니다.  
ViewModel은 UI의 상태를 보유하고 있기 때문에  
상태를 초기화하는 작업은 ViewModel.init()이 적합하다고 생각합니다.  

장점은 UI와 상태를 분리해서 다룰 수 있어서 좋습니다.  
단점은 ViewModel과 상태 초기화가 엮여있어서 부작용이 발생할 수 있다는 것입니다.  
UI를 개선해야 할 때 원치않은 정보가 ViewModel.init()에서 변경된다면  
ViewModel까지 수정 범위를 넓혀야합니다.  

LaunchedEffect의 장점은 UI에서 어떤 정보들을 사용하는지 직관적으로 볼 수 있고  
단점은 상태를 UI에서 직접 변경하기 때문에 UI와 상태를 변경하는 로직이 분리되지 못합니다.  
LaunchedEffect는 주로 특정 상세정보나 정보가 ViewModel에 전달되어야 할때 사용됩니다.  
LaunchedEffect(userId) {
    viewModel.fetchUserData(userId)
}

#### ❓ 실전 질문
#### 두 방식과 비교했을 때 콜드 플로우를 사용한 지연 관찰은 초기 데이터를 로드할 때 효율성을 어떻게 개선하나요?  
콜드 플로우를 사용한 지연 관찰은 데이터가 필요할 때 해당 로직을 실행할 수 있기 때문에  
부하를 지연해서 발생시킬 수 있습니다.  
기존 방식들은 모두 컴포저블이 처음 호출될 때 실행되지만  
위 방식은 지연되기 때문에 예측되는 부작용을 줄일 수 있습니다.  
