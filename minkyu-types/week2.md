# Q3) Serializable과 Parcelable의 차이점은 무엇인가요?

## Serializable
- JAVA 표준 인터페이스
- 객체를 바이트 스트림으로 변환
- 리플렉션 API 기반으로 동작하기 때문에 런타임에 클래스와 필드를 동적으로 검사하여 직렬화
- 직렬화 과정 중에 많은 임시 객체를 생성하여 메모리 오버헤드가 증가

## Parcelable
- 안드로이드 전용 인터페이스
- 각 프로퍼티를 Parcel 버퍼에 읽고 쓰는 방식
- 리플렉션 API를 사용하지 않으므로 Serializable보다 빠름
- 임시 객체 또한 많이 생성하지 않아 GC 최소화
- 컴포넌트 간 데이터 전달에 사용
- Parcelize 플러그인과 함께 사용하면 보일러플레이트 코드 감소

## 대부분의 경우
- 안드로이드 개발에서 Serializable을 사용할 일은 없다

## Pro tips for mastery - Parcelable과 Parcel이란?
### Parcel
- 컴포넌트 간의 고성능 프로세스 간 통신(IPC)를 가능하게 하는 컨테이너 클래스
- 데이터를 마샬링(flatten)/언마샬링(unflattening)gkdu IPC 경계를 넘어 전달할 수 있게 함
- 내부 구현이 변경될 경우 이전 데이터를 읽지 못할 수 있음(영구 저장 불가)

<br></br>

# Q4) Context란 무엇이며 어떤 유형의 Context가 있나요?

## Context란
- 애플리케이션의 환경/상태를 나타내는 객체
- 애플리케이션 리소스/클래스에 대한 접근을 제공
- 앱과 안드로이드 시스템 사이의 인터페이스
- 컴포넌트가 리소스/DB/시스템 서비스 등에 접근할 수 있도록 함

## Appliation Contezt
- 애플리케이션 라이프 사이클과 연결되어 있는 객체
- Activity/Fragment보다 오래 지속되는 Context가 필요할 때 사용
### 사용 예시
1. SharedPreference/DataStore/Room과 같은 앱 전역 리소스 접근 시
2. 전체 앱 라이프 사이클 동안 지속되어야 하는 BroadcastReceiver를 등록하는 경우
3. 앱 라이프 사이클 동안 유지되는 라이브러리나 컴포넌트를 초기화할 때

## Activity Context
- Activity의 this 인스턴스
- Activity의 라이프 사이클과 연결되어 있음
- Activity 관련 리소스 접근, 다른 Activity 시작, 레이아웃 인플레이션에 사용
### 사용 예시
1. UI 컴포넌트를 생성/업데이트할 때
2. 다른 Activity를 시작할 때
3. 현재 Activity 범위 안의 리소스/테마에 접근하는 경우

## Service Context
- Service 라이프 사이클과 연결되어 있음
- 네트워크 작업 수행, 음악 재생 등 백그라운드에서 실행되는 작업에 사용
- Service에 필요한 시스템 서비스에 대한 접근을 제공

## Broadcast Context
- BroadcastReceiver가 호출될 때 제공
- 일반적으로 특정 Broadcast에 응답할 때 사용
- 장기적인 테스크를 수행하면 안됨

## Pro tips for mastery - Context 사용 시 주의할 점은 무엇인가요?
- 메모리 누수, 크래시, 비효율적인 리소스 처리 같은 문제를 일으킬 수 있음
- Activity Context에 대한 참조를 해당 컴포넌트 라이프 사이클보다 길게 유지하는 경우,
  해당 Context가 강하게 참조하는 요소들 또한 메모리에서 회수되지 못해 메모리 누수로 이어짐
  -> 이 경우 대부분 메모리에 상주하는 스태틱 객체가 Context를 참조하는데,
  WeakReference로 감싸서 약하게 참조해야 한다.
- UI 컴포넌트 생성 시 Application Context를 사용하면 테마가 올바르게 적용되지 않음
  -> Activity Context를 사용해서 테마를 적용해야 함
- 컴포넌트가 파괴된 후에 Context를 참조하지 않아야 함
    - 해당 Context에 연결된 리소스가 존재하지 않을 수 있으므로 예상치 못한 동작이 발생할 수 있음
- 백그라운드 스레드에서 Context 참조하지 않기
    - 메인 스레드용으로 설계된 Context
    - 코루틴 사용 시 메인 스레드로 전환해서 사용해야 함

## Pro tips for mastery - ContextWrapper란 무엇인가요?
- ContextWrapper는 Contet를 상속받는 클래스로, Context 객체를 감싸서 래핑된 Context에
  대한 호출을 위임하는 기능을 제공함
- 원본 Context의 동작을 수정/확장하기 위한 중간 계층 역할
- ContextWrapper를 사용하면 Context와 직접 접근하지 않고 특정 기능을 커스텀할 수 있음
- 기존 Context의 특정 동작을 개선/오버라이드 해야 할 때 사용
### 사용 예시
- Dagger/Hilt는 DI를 위해 커스텀 ContextWrapper를 사용하고, 컴포넌트에 해당 ContextWrapper를
  Context 타입으로 제공함

## Pro tips for mastery - Activity의 this 인스턴스와 baseContext 인스턴스의 차이점은 무엇인가요?
- 둘 다 Context를 반환함

### Activity의 this 인스턴스
- Activity의 현재 인스턴스를 참조하며, ContextWrapper의 하위 클래스이므로
  this는 라이프 사이클 관리 및 UI 상호작용 갘은 추가 기능을 포함해서
  Activity와 상호작용할 수 있는 API를 호출할 수 있음
- 또한 Activity의 현재 Context를 참조하므로, 해당 Activity의 메서드를 호출할 수 있음

### baseContext 인스턴스
- Activity가 구축되는 기반 또는 기본 Context를 나타냄
- Activity가 상속받는 ContextWrapper 클래스의 "일부"
- Context 메서드에 대한 핵심 구현부를 제공하는 ContextImpl 인스턴스

### this와 baseContext의 주요 차이점
- 범위
    - this는 Activity 인스터스와 라이프사이클을 나타내지만,
    - baseContext는 저수준 Context를 참조
- 사용법
    - this는 Activity 라이프사이클이나 UI 관련 작업에 사용
    - baseContext는 커스텀 ContextWrapper를 구현할 때, Context의 핵심 구현체와 상호작용할 때 사용
- 계층
    - baseContext는 Activity의 기반 Context
    - baseContext에 접근하면 Activity가 ContextWrapper에 제공하는 Api에 대해 우회 접근이 가능

### 요약
- this는 Activity 자체를 참조하지만,
- baseContext는 원본이 수정되지 않은 Context를 참조

# Q5) Application 클래스란 무엇인가요?
- 전역 애플리케이션 상태와 라이프사이클을 유지하기 위한 역할
- 애플리케이션의 프로세스 진입점
- 앱 전역에 걸쳐 공유되는 리소스/인스턴스를 초기화하기 좋음

### 주요 메서드
1. onCreate()
    - 앱 프로세스가 생성될 때 호출됨
    - DB 인스턴스, 네트워크 라이브러리, Firebase 애널리틱스 등을 초기화
    - 앱 라이프사이클 전체 중 단 한 번만 호출됨
2. onTerminate()
    - 애플리케이션이 종료될 때 호출됨
    - 안드로이드가 이 메서드의 호출을 보장하지 않음
    - 따라서 실제 기기의 프로덕션 환경에서는 호출되지 않음
3. onLowMemory()
    - 시스템이 메모리 부족 상태를 감지할 때 호출됨

### 사용사례
1. 전역 리소스 관리
2. 컴포넌트 초기화
3. 의존성 주입

### 주의사항
- 초기 앱 실행 지연을 방지하기 위해 onCreate()에서 무거운 테스크를 실행하지 말 것
- 전역 초기화 및 리소스 관리 로직만 작성할 것
- 앱 전반에 걸쳐 사용되는 공유 리소스에 대해서는 스레드 안전성을 보장해야 함

# Q6) AndroidManifest 파일의 목적은 무엇인가요?
- 안드로이드 OS에 애플리케이션에 대한 필수 정보들을 정의하는 구성 파일
- 애플리케이션과 OS 사이의 브릿지 역할
- 애플리케이션

### 주요 기능
1. 애플리케이션 컴포넌트 선언
- 필수 컴포넌트를 등록해서 안드로이드 시스템이 이를 시작하거나 상호작용할 수 있게 함
2. 권한 설정
- 앱에 필요한 권한을 선언해서 사용자가 이를 허용/거부 가능
3. 하드웨어 및 소프트웨어 요구 사항
- 앱이 의존하는 기능을 명시 -> 이를 충족하지 않는 기기를 PlayStore가 필터링할 수 있도록 함
4. 앱 메타데이터
- 앱의 패키지 명, 버전, 최소/타겟 API 레벨, 테마, 스타일 정보 제공
5. 인텐트 필터
- 컴포넌트에 대한 Intent Filter를 정의해서 링크를 열거나 공유하는 Intent 종류를 명시
- 다른 앱이 개발자의 앱과 상호 작용할 수 있도록 함
6. 앱 구성 및 설정
- 메인 런처 Activity 정의, 테마 지정 등

# Q7) Activity 생명주기를 설명해주세요
- Activity가 생성부터 소멸까지 거치는 다양한 상태를 나타냄
- 리소스를 효과적으로 관리할 수 있음

1. onCreate
2. onStart
3. onRestart
3. onResume
4. onPause
5. onStop
6. onDestroy

### onPause()와 onStop()의 차이점은 무엇인지 설명하고, 리소스 점유율이 높은 작업을 처리하는 경우 해당 메서드들을 어떤 시나리오에서 활용해야 하나요?

## Pro tips for mastery - 액티비티 간의 생명주기 변화 심층적으로 살펴보기
- 

## Pro tips for mastery - 액티비티의 라이프사이클 인스턴스란 무엇인가요?

# Q8) Fragment 생명주기를 설명해주세요
- 각 Fragment 인스턴스는 연결된 부모 액티비티의 생명주기와는 별도로 자체적인 생명주기를 가진다
- 


