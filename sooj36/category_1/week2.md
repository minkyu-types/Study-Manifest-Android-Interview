##    ***Q) 3. Serializable과 Parcelable의 차이점은 무엇인가요?***

       **실전 질문:**
        안드로이드에서 Serializable과 Parcelable의 차이점은 무엇이며, 일반적으로 컴포넌트 간 데이터 전달에 Parcelable이 선호되는 이유는 무엇인가요?

            **- Serializable**

                - 자바 표준 인터페이스(java.io.Serializable 구현만 필요)

                - 리플렉션 기반의 느린 속도 (런타임 시, 객체 그래프를 분석하고 직렬화)

                - 많은 임시 객체 생성 (GC 부담 증가, 메모리 오버헤드 발생)

                - 성능이 중요하지 않은 작업 처리 시, 컴포넌트 간에 데이터 교환 목적이 아닌 용도에 사용

  

            **- Parcelable**

                - 안드로이드 특화 인터페이스

                - 높은 성능과 효율성 (리플렉션 의존 없이 명시적 직렬화 수행 (Serializable 대비 10대 빠름))

                - 적은 메모리 사용량 (최소한의 임시 객체만을 생성하여 GC 부담 감소)

                - 성능이 중요한 데이터 전달에 선호

  
 #### @ **Pro Tips for Mastery: Parcel과 Parcelable이란 무엇인가요?**

        - Parcel

            - 프로세스 간 통신(IPC)을 위한 고성능 컨테이너 클래스

                - 즉, Parcelable(포장 설명서)에 따라 잘 포장된 데이터(물건)를 담아 운반하는 택배 상자/트럭

            - 프로세스 경계를 넘어 다양한 데이터 타입(문자열, 배열, 객체 등)을 효율적으로 직렬화하여 전송

        - (Q). 왜 영구 저장용으로 사용하면 안되는지 -?

            -(A). 내부 구현이 변경될 수 있어, 이전 데이터를 읽을 수 없게 만들 수 있음

        - (Q2). 데이터 크기가 1MB로 제한

            -(A2).

            Parcelable 로 직렬화된 데이터는 바인더 트랜잭션 버퍼라는 정해진 크기의 공간을 통해 전달되는데, 해당 버퍼의 크기는 약 1MB로 제한

            공식 Docs : [https://developer.android.com/reference/android/os/TransactionTooLargeException]

            "The Binder transaction buffer has a limited fixed size, currently 1Mb, which is shared by all transactions in progress for the process"

            "바인더 트랜잭션 버퍼는 제한된 고정 크기를 가지며, 이는 프로세스에서 진행 중인 모든 트랜잭션이 공유합니다"

  
  -----------------------

## *   ***Q) 4. Context란 무엇이며 어떤 유형의 Context가 있나요?***

| **Application Context** | **Activity Context**      |
| ------------------- | --------------------- |
| **Service Context<br>** | **Broadcast Context<br>** |
- `(1). Application Context`
	- 앱 전체 생명주기와 일치하는 컨텍스트
	- 싱글톤이며 어디서든 호출이 가능하고, UI 작업에는 적합하지 않음
	  
- `(2). Activity Context`
	- 해당 Activity의 생명주기와 연결된 컨텍스트, UI 관련 작업에 적합하며 완료 시, 함께 소멸
	  
- `(3). Service Context`
	- Service 의 생명주기와 연결된 컨텍스트
	- 네트워크 작업, 음악 재생처럼 백그라운드에서 실행되는 작업에 주로 사용
	  
- `(4). Broadcast Context`
	- `BroadcastReceiver`가 호출될 때 제공되는 컨텍스트
	- 수명이 짧기 때문에 특정 브로드캐스트에 응답하는 용도로만 사용해야 하며, 오래 걸리는 작업을 수행해서는 안됨
		- `BroadcastReceiver`의 `onReceive()` 메서드는 메인 스레드에서 실행되는데, 오래 걸리는 작업 수행 시 ANR 발생 
		  (해당 작업은 Service / WorkManager같은 백그라운드 전용 컴포넌트에서)

    *   `**실전 질문:** 안드로이드 애플리케이션에서 올바른 유형의 Context를 사용하는 것이 왜 중요하며, Activity Context에 대해 오랜 참조를 유지하는 것은 잠재적으로 어떤 문제를 발생시킬 수 있나요?`
	    *  1. 해당 생명주기보다 오래 유지하여, GC이 Context 또는 관련 리소스에 대한 메모리를 회수할 수 없게 하므로 메모리 누수가 생기게 됨.
	    * 2. UI 요소를 Application Context 로 생성 시, 테마 정보가 없어 화면이 의도와 다르게 나타나는 예상치 못한 동작이 일어남.

  #### @ **Pro Tips for Mastery: ContextWrapper란 무엇인가요?**
	- 기존 Context 객체를 한번 감싸서(Wrapping), 원본을 직접 수정하지 않고, 기능을 변경하거나 확장할 수 있도록 하는 포장지 같은 클래스

	- 기존 Context 가 가진 특정 동작을 재정의하거나, 새로운 기능을 추가하고 싶을 때 사용하며 이를 통해 기존 코드와의 호환성을 유지하면서 필요한 부분만 커스터마이징 할 수 있음

	- 사용 사례
		- 1. 커스텀 테마 적용 
		- 2. 동적 리소스 처리 : ex 상황에 따라 다른 언어의 문자열 같은 리소소를 제공할 수 있음
		- 3. 의존성 주입(DI)
	- 이점
		- 1. 재사용성 : 커스텀 로직을 래퍼 클래스로 캡슐화하여 여러 컴포넌트에서 재사용할 수 있음
		- 2. 캡슐화 : 원본 코드를 변경하지 않고, 필요한 기능만 안전하게 재정의


  #### @ **Pro Tips for Mastery: Activity에서 this와 baseContext 인스턴스의 차이점은 무엇인가요?
	  - 모두 Context를 반환하지만, 서로 다른 계층과 목적을 가짐
	  - Activity 'this'
		  - (특징) ContextWrapper 의 하위 클래스이므로, this는 생명주기 관리나 UI 상호작용과 같은 Activity 고유 기능이 포함된 고수준 Context
		  - (사용시점) 다른 Activity를 시작하거나, Dialog를 띄우는 등 일반적인 UI 및 생명주기 관련 작업에 사용
	- baseContext
		- Activity가 상속하는 ContextWrapper 클래스의 일부로, Activity가 구축되는 기반이 되는 기본(Basic) Context 를 나타냄
		- (특징) Context 메서드 핵심 구현을 제공하는 ContextImpl 인스턴스
		- (사용시점) 일반적으로 직접 사용하는 경우는 드물며, ContextWrapper를 구현하는 등에 사용
	- (차이점)
		- (범위) this는 현재 Activity 인스턴스와 그 생명주기를 나타내지만, baseContext는 Activity가 구축된 저수준의 Context 를 참조
		- (사용법)  this는 UI 관련 작업에, baseContext 는 주로 커스텀 ContextWrapper 구현 시 핵심 기능과 성호작용 할때 사용
  
  -------------

## ***Q) 5. Application 클래스란 무엇인가요?***

1. 주요 역할
	- **전역 리소스 관리** 
	  : DB, 네트워크 클라이언트처럼 앱 어디서든 공통으로 사용되어야 하는 객체들을 생성하고 관리
	  
	- **컴포넌트 초기화** 
	  : Firebase Analytics나 Timber 같은 분석 및 로깅 라이브러리들은 앱이 시작될 때 한 번만 초기화해주면 되는데, 
	  Application 클래스의 `onCreate()`에서 적용
	  
	- **의존성 주입** : Dagger 나 Hilt 같은 프레임워크를 초기화하여 앱 전체에 의존성 제공
2.  주의 사항
	-  (1) onCreate() 에서 무거운 작업을 하지 않는 것
		- → Application.onCreate()는 메인 스레드에서 실행 
		  고로, 무거운 작업을 수행하게 되면 앱의 첫 화면이 뜰 때까지 시간이 길어지게 되어 사용자 경험에 부정적으로 작용
		  
	- (2) 만능 클래스로 쓰지 않는 것
		- → 단일 책임 원칙(SRP)의 관점에서 볼 때, 
		  Application 클래스의 책임은 '앱의 전역 상태 관리 및 초기화' 하는 것 하나.
		- 결과적으로 다른 부분과 얽혀 결합도가 높아지면 유지보수나 테스트에 부정적인 영향을 끼치게 됨
		  
	- (3) 스레드 안정성 보장할 것
		- → 관리하는 공유 리소스는 앱의 생명주기 동안 계속 존재하며, 여러 스레드에서 동시에 접근이 가능
		- ex. 백그라운드 스레드가 서버에서 사용자 정보를 받아와 업데이트를 하는 동시에, 
		  메인 스레드는 화면을 그리기 위해 해당 정보를 읽으려 할 수 있음.
		  -> 이럴 때, 안전장치가 없다면 데이터가 손상되거나 앱이 종료될 수 있음

#### @ **실전 질문 : Application 클래스의 목적은 무엇이고, (주요 역할 답변) 생명주기 및 리소스 관리 측면에서 Activity와는 어떻게 다른가요?**

	 -  **생명주기**
		 - Application 클래스 : 앱 프로세스 전체와 동일
		 - Activity : 생명주기가 하나의 화면에 묶여있어서, 상호작용에 따라 수시로 생성되고 소멸
	- **리소스 관리**
		- Application 클래스 : 어떤 화면에서든 접근해야 하는 전역 리소스(DB 연결 객체, 네트워크 클라이언트) 관리
		- Activity : 해당 화면에서만 사용하는 지역적 리소스(특정 화면의 애니메이션 등) 관리
				- onStop() : 화면이 보이지 않을 때, 리소스 해제
				- onDestroy() : 화면이 소멸될 때, 완전히 정리


  -------------------

##    ***Q) 6. AndroidManifest 파일의 목적은 무엇인가요?***

> - 안드로이드 운영체제가 앱을 실행하고 관리하는 데, 필요한 모든 필수 정보(패키지명, 버전, API 레벨 등)를 담고 있는 설정 파일

| 주요 컴포넌트 등록   |              |
| ------------ | ------------ |
| < activity > | 사용자 인터페이스 화면 |
| < service >  | 백그라운드 처리 작업  |
| < receiver > | 브로드캐스트 리시버   |
| < provider > | 콘텐츠 프로바이더    |
- 핵심 컴포넌트를 등록하지 않으면, 시스템은 존재를 알지 못해 실행 불가

| 권한 및 정보 선언          |               |
| ------------------- | ------------- |
| < uses-permission > | 앱 권한 요청       |
| < uses-feature >    | 필요한 하드웨어 / 기능 |
| < intent-filter >   | 컴포넌트가 처리할 인텐트 |
| < meta-data >       | 추가 정보 제공      |
- 인터넷 사용, 위치 정보 접근 등 앱이 동작하는 데, 필요한 모든 권한도 이 곳에 선언해야 함

> [!실전 질문]
> AndroidManifest의 인텐트 필터는 앱 상호 작용을 어떻게 가능하게 하고, 액티비티 클래스가 AndroidManifest에 등록되어있지 않으면 어떻게 되나요?

	- (1) 인텐트 필터는, 특정 컴포넌트가 어떤 종류의 인텐트에 응답할 수 있는지 명시하는 역할
		- EX) 특정 Activity에 '링크 열기' or '콘텐츠 공유'가 가능한 인텐트 필터를 설정해두면, 
		  다른 앱이나 안드로이드 시스템이 해당 링크나 콘텐츠를 처리해야 할 때
		  우리의 앱을 후보로 올려주거나 직접 실행시켜 "어떤 요청에 응답할 수 있는지 미리 알려주는 방식"으로 앱 간의 상호작용을 가능하게 함
	
	- (2) 액티비티 클래스가 AndroidManifest에 등록되지 않으면, 안드로이드 시스템은 해당 Activity의 존재 자체를 알지 못함.
		  고로, 해당 Activity를 시작하려 시도해도, 시스템은 Activity를 찾을 수 없는 ActivityNotFoundException 을 발생시켜 앱이 비정상적으로 종료. 

  -----------------

##    ***Q) 7. Activity 생명주기를 설명해주세요***
![[Pasted image 20250714155336.png]]
![[Pasted image 20250714155100.png]]

- onCreate()
	- setContentView() 로 UI 레이아웃을 설정하고, 뷰와 데이터를 바인딩하며, 
	  saveInstanceState가 있다면, 상태를 복원하는 등 Activity의 모든 일회성 초기화를 수행하는 곳
- onStart()
	- Activity의 UI가 사용자에게 가시적인 상태가 되지만, 포커스를 받지 못해 상호작용은 불가능
- onResume()
	- Activity가 foreground 상태가 되어 사용자와 상호작용 가능
	- Running 상태

(화면 전환 또는 다른 앱 실행 상황)
- onPause()
	- Activity가 포커스를 잃었을 때, 호출
	- 다이얼로그가 위에 뜨는 것처럼 Activity가 부분적으로 보이기는 하지만, 사용자와 상호작용 불가능한 상태
- onStop()
	- 만약 다른 Activity가 화면 전체를 덮어 UI가 완전히 보이지 않게 될 때 호출
	- 이 상태에서 불필요한 리소스를 해제하거나 비교적 무거운 정리 작업 수행
	  
(다시 앱으로 돌아온 상황)
- onStop() 상태에 있던 Activity로 사용자가 다시 돌아오면 onRestart()가 가장 먼저 호출되고 -> onStart() 와 onResume() 을 거쳐 다시 'Running' 상태로 복귀

(Activity 종료 상황)
	- 사용자가 뒤로 가기로 Activity를 종료하거나, 시스템이 Activity를 소멸시킬 때 onDestroy()가 호출
	- 모든 리소스가 최종적으로 해제
  
> [!실전 질문]
>     onPause()와 onStop()의 차이점은 무엇인지 설명하고, 리소스 점유율이 높은 작업을 처리하는 경우 해당 메서드들을 어떤 시나리오에서 사용해야 하나요?
-  두 메서드의 차이점 "가시성" 여부 (위에 답변)
-  리소스 점유율 높은 작업 처리 시
	- onPause() :
		-  애니메이션, 센서 업데이트 중지 같은 비교적 가벼운 작업을 일시 중지하는데 사용
		  
	- onStop() :
		-  Activity가 화면에 보이지 않는 동안 필요 없는 무거운 리소스를 해제하거나, 백그라운드 작업을 중단하는 등, 리소스 점유율이 높은 작업처리에 사용


#### @ Pro Tips for Mastery: 액티비티 간의 생명주기 변화 심층적으로 살펴보기

https://claude.ai/public/artifacts/0ebb466f-7585-4206-a5a3-5d26dfd36ca4

#### @ Pro Tips for Mastery: Activity의 lifecycle 인스턴스란 무엇인가요?

- Activity의 현재 상태를 실시간으로 알려주는 자동 알림 시스템
- 기존 : GPS를 on/off 하는 로직
	- Activity의 onStart()에서 직접 GPS on 코드를 넣고, onStop()에서 off 코드를 넣어야 함
	- 모든 생명주기 관련 코드가 Activity안에 다 들어가서 코드가 지저분해짐

-  lifecycle 인스턴스 방식
	- GPS 관련 로직을 별도의 클래스로 만들고, lifecycle 인스턴스에 lifecycle.addObserver() 구독신청 함

- 리소스 생명주기를 깔끔하게 관리해주어 메모리 누수 방지


  -----------------

##    ***Q) 8. Fragment 생명주기를 설명해주세요***
![[Pasted image 20250714174857.png]]


> [!실전 질문]
> **:** onCreateView()와 onDestroyView()의 목적은 무엇이며, 해당 메서드에서 뷰 관련 리소스를 올바르게 처리하는 것이 왜 중요한가요?

	- onCreateView() 는
		Fragment UI가 처음 그려질 때 호출되며, 이 메서드에서 레이아웃 XML 파일을 화면에 표시될 객체로 만드는 작업을 하고, 이 View에 반환
	- onDestroyView() 는
		Fragment 의 View 계층 구조가 메모리에서 제거될 때 호출
	-(2). 메모리 누수 방지 위해
		onDestroyView()가 호출되면 View는 파괴되지만, Fragment 객체 자체는 아직 살아있을 수 있음
		이때 View에 대한 참조(어댑터, 뷰 바인딩 객체 등)를 깨끗하게 정리하지 않으면, GC이 파괴된 메모리에서 회수하지 못해 메모리 누수로 이어짐
		
#### @ **Pro Tips for Mastery** : fragmentManager와 childFragmentManager의 차이점은 무엇인가요?
> / **fragmentManager :**
> 	Activity 수준에서 Fragment 관리하는 역할
> 	Activity에 직접 연결된 메인 Fragment들을 추가, 교체 제거하는데 사용
> 
> / **childFragmentManager :**
> 	하나의 Fragment 내에서 그 자식 Fragment들을 관리
> 	이를 통해 ViewPager 안에 여러 Fragment를 넣는 것처럼, Fragment안에 또 다른 Fragment를 중첩시키는 복잡한 UI구조 만들 수 있음


  -------------------

##    ***Q) 9. Service란 무엇인가요?***

> [!실전 질문]
> Started 서비스와 Bound 서비스의 차이점은 무엇이며, 각각 언제 사용해야 하나요?

> - **Started Service** 는 
> 	- 앱에서 startService() 를 호출할 때 시작
> 	- stopSelf() 를 사용하여 스스로 작업을 중지하거나 stopService()를 사용하여 명시적으로 중지될 때 까지 지속적으로 실행
> 	- ex) 백그라운드 음악 재생, 파일 업로드/다운로드 **(독립적으로 작업을 수행하는가 ?)**
> 	  
> - **Bound Service** 는
> 	- 컴포넌트가 bindService() 를 사용하여 서비스에 바인딩할 수 있도록 함
> 	- 바인딩된 클라이언트가 있는 동안 활성 상태를 유지하며, 모든 클라이언트의 연결이 끊어지면 자동으로 중지
> 	- ex) 원격 서버에서 데이터 가져오기, 백그라운드 블루투스 연결 관리 **(다른 컴포넌트와 상호작용하며 결과를 주고받는가?)**

#### @ Pro Tips for Mastery: 포그라운드 서비스(foreground service)를 어떻게 처리하나요?

> - **포그라운드 서비스** : 사용자가 **인지할 수 있는 작업을 수행**하는 특별한 유형의 서비스이며, 지속적인 알림을 반드시 표시해야 함
> 	- 일반 서비스와의 가장 큰 차이는 startForeground() 를 통해 호출하고, **시작 즉시 알림을 표시해야 한다는 것**

```
import android.app.*
import android.content.Intent
import android.os.Build
import android.os.IBinder
import androidx.core.app.NotificationCompat

class ForegroundService : Service() {

    override fun onCreate() {
        super.onCreate()
        // 리소스 초기화
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        val notification = createNotification()
        startForeground(1, notification) // 서비스를 포그라운드로 시작
        
        // 실제 작업 수행
        // ...

        return START_STICKY
    }

    private fun createNotification(): Notification {
        val notificationChannelId = "ForegroundServiceChannel"
        
        // Android 8.0 (API 26) 이상에서는 알림 채널 생성 필수
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                notificationChannelId,
                "Foreground Service",
                NotificationManager.IMPORTANCE_DEFAULT
            )
            getSystemService(NotificationManager::class.java).createNotificationChannel(channel)
        }

        return NotificationCompat.Builder(this, notificationChannelId)
            .setContentTitle("Foreground Service")
            .setContentText("Running in the foreground")
            .setSmallIcon(R.drawable.ic_notification)
            .build()
    }

    override fun onDestroy() {
        super.onDestroy()
        // 리소스 정리
    }

    // Bound Service가 아니므로 null 반환
    override fun onBind(intent: Intent?): IBinder? = null
}
```

- 포그라운드 서비스는 사용자에게 작업이 진행 중임을 **'지속적인 알림'**으로 명확히 보여주는 특별한 서비스

###### **핵심 로직: `onStartCommand()`**

서비스가 시작될 때 호출되는 이 메서드에서 다음 순서로 작업을 처리합니다.

1. **알림 생성 (`createNotification()`)**
    
    - 가장 먼저 사용자에게 표시할 `Notification` 객체를 생성. 
    - 포그라운드 서비스는 알림 표시가 의무이기 때문.
        
2. **포그라운드 상태 전환 (`startForeground()`)**
    
    - 생성한 알림과 함께 `startForeground()`를 호출. 
    - 이 메서드가 일반 서비스를 포그라운드 서비스로 전환시키고 알림을 즉시 표시하는 핵심적인 역할
        
3. **실제 작업 수행**
    
    - 포그라운드 상태로 전환된 후, 음악 재생이나 위치 추적 같은 실제 백그라운드 작업을 수행하는 로직을 실행
        
4. **서비스 재시작 정책 반환 (`return START_STICKY`)**
    
    - 메모리 부족 등으로 시스템이 서비스를 강제 종료했을 때, 나중에 시스템 자원이 확보되면 서비스를 다시 생성하라고 알려줌
        

###### **(중요) 알림 생성 상세: `createNotification()`**

알림을 생성할 때, 가장 중요한 것은 **Android 8.0 (API 26) 이상** 버전을 위한 처리

- **알림 채널(NotificationChannel) 생성**: 이 버전부터는 알림 채널을 만들고 시스템에 등록하지 않으면 **알림 표시 X

###### **전체 생명주기 관리**

핵심 로직 외에 전체적인 서비스 처리를 위해 다음 생명주기 메서드도 함께 관리.

- **초기화 (`onCreate`)**
    
    - 서비스가 처음 생성될 때 딱 한 번 호출되며, 재사용될 리소스를 미리 초기화하기에 적합.
        
- **정리 (`onDestroy`)**
    
    - 서비스가 소멸될 때 호출되며, 사용했던 모든 리소스를 깨끗하게 정리하고 해제해야함.

#### @ Pro Tips for Mastery: 서비스(service)의 생명주기는 어떻게 되나요?

> 
![[Pasted image 20250714183045.png]]

> - **Started Service 생명주기**
> 	- (시작) startService() 호출
> 		- 다른 컴포넌트가 startService() 를 호출하는 것에서 시작
> 		
> 	- (생성) onCreate() 
> 		- 서비스가 처음 생성될 때 단 한 번만 호출
> 		- 이 단계에서 서비스에 필요한 리소스를 초기화 하는 작업을 수행
> 		
> 	- (작업 실행)  onStartCommand()
> 		- startService()가 호출될 때마다 트리거되어 실제 백그라운드 작업을 처리하는 곳
> 		- 작업 처리 후, 서비스가 시스템에 의해 강제 종료되었을 때, 어떻게 동작할 지 결정하는 정책(START_STICKY등)을 반환
> 	- (종료) `stopSelf()` 또는 `stopService()`
> 		-  Started Service 는 스스로 stopSelf()를 호출하거나, 다른 컴포넌트가 stopService() 를 호출하여 명시적으로 중지시키기 전까지 계속 실행
> 	- (소멸) onDestroy()
> 		- 서비스가 중지될 때 호출되는 마지막 단계
> 		- 사용했던 모든 리소스를 깨끗하게 해제하는 정리 작업 수행

> - **Bound Service 생명주기**
> 	- **(시작) `bindService()` 호출**
> 	    - 다른 컴포넌트가 `bindService()`를 호출하여 서비스에 연결(바인딩)을 요청하는 것에서 시작.
>         
> 	- **(생성) `onCreate()`**
> 	    - 서비스가 처음 생성될 때 단 한 번만 호출
> 	    - 이 단계에서 서비스에 필요한 리소스를 초기화하는 작업을 수행.
>         
> 	- **(연결) `onBind()`**
> 	    - 클라이언트가 서비스에 성공적으로 바인딩될 때 호출
> 	    - 서비스와 통신하는 데 사용할 인터페이스(`IBinder`)를 반드시 반환해야 함
>         
> 	- **(연결 해제) `onUnbind()`**
> 	    - 서비스에 연결된 모든 클라이언트의 연결이 끊어질 때 호출
> 	    - 바인딩된 클라이언트에 특정한 리소스를 정리하는 작업을 수행할 수 있음
>         
> 	- **(소멸) `onDestroy()`**
> 	    - 서비스가 종료될 때 호출되는 마지막 단계
> 	    - 모든 클라이언트의 연결이 끊어져 서비스가 소멸될 때 호출되며, 사용했던 모든 리소스를 정리
--------