## Q52) DataBinding의 동작 원리에 대해서 설명해주세요
- DataBinding은 XML 레이아웃의 UI 컴포넌트를 앱의 데이터 소스에 직접 바인딩할 수 있는 안드로이드 라이브러리
- 보일러 플레이트 코드를 줄이고 UI <> 데이터 모델 간의 실시간 업데이트를 허용하여 UI 디자인에 선언적 프로그래밍을 부분적으로 적용시킴
- 또한 이 개념은 UI 로직과 비즈니스 로직 분리를 위한 디자인 패턴으로, MVVM 아키텍처에서 중심점 역할을 함
- <layout> 태그를 사용하는 각 XML 레이아웃에 대한 바인딩 클래스를 생성하며, 이를 통해 뷰에 직접적으로 접근하고 표현식을 사용하여 데이터를 바인딩할 수 있음

### DataBinding의 특징
(1) 양방향 데이터 바인딩
- UI와 기본 데이터 모델 간의 데이터 자동 동기화를 가능하게 함
- 입력 필드 값을 업데이트하는 등에 특히 유용함
(2) 바인딩 표현식
- 문자열 연결/조건문 같은 간단한 로직을 XML에서 직접 사용할 수 있음
(3) 생명주기 인식
- 생명주기가 적절한 상태일 때만 UI를 자동으로 업데이트함

### DataBinding의 장점
(1) 보일러플레이트 코드 감소: `findViewById()` 및 명시적인 UI 업데이트가 필요 없어짐
(2) 실시간 UI 업데이트: 데이터 변경 사항을 UI에 자동으로 반영함
(3) 선언적 UI: 로직을 XML로 이동하여 잘 사용하면 복잡한 레이아웃을 단순화할 수 있음
(4) 테스트 용이성 향상: UI와 코드를 분리하여 둘 다 독립적으로 테스트하기 쉽게 만듬

### DataBinding의 단점
- 성능 오버헤드: ViewBinding에 비해 더 많은 런타임 오버헤드가 발생함
- 복잡성: 작거나 간단한 프로젝트에는 불필요한 복잡성을 유발할 수 있음
- 러닝 커브: 바인딩 표현식 및 생명주기 관리에 대한 러닝 커브가 요구됨


## Q53) LiveData에 대해서 설명해 주세요
- LiveData는 AAC에서 제공하는 관찰 가능한 데이터 홀더 클래스
- 생명주기를 인식하므로 Activity/Fragment/View와 같이 연관된 안드로이드 컴포넌트의 생명주기에 따라 동작이 달라짐
- 주요 목적은 UI 컴포넌트가 데이터 변경 사항을 관찰하고 해당 데이터가 변경될 때마다 UI를 반응형으로 업데이트할 수 있도록 하는 것

### LiveData의 장점
(1) 생명주기 인식: LiveData는 컴포넌트의 생명주기를 관찰하고 컴포넌트가 활성 상태일 때만 데이터를 업데이트하여 크래시 및 메모리 누수 위험을 줄임
(2) 자동 정리: 컴포넌트에 연결된 관찰자는 주어진 생명주기가 소멸될 때 자동으로 제거되고 정리됨
(3) 관찰자 패턴: UI 컴포넌트는 관찰자를 활용하여 LiveData의 데이터가 변경될 때 자동으로 업데이트됨
(4) 스레드 안전성: LiveData는 스레드 안전하도록 설계되어 백그라운드 스레드에서 업데이트할 수 있음

### MutableLiveData와 LiveData의 차이
- MutableLiveData: `setValue()`, `postValue()`를 통해 데이터 수정을 허용함
  외부에서의 직접적인 수정을 방지하기 위해 ViewModel 내에서 프라이빗하게 유지함
- LiveData: 외부 컴포넌트가 데이터를 수정하는 것을 방지하는 읽기 전용 LiveData로, 더 나은 캡슐화를 보장함

### LiveData 사용 사례
(1) UI 상태 관리: 네트워크 응답, 데이터베이스와 같은 소스의 데이터를 담는 컨테이너 역할
(2) 옵저버 패턴 구현: LiveData는 발행자 역할을 하고 Observer 인터페잉스 구현이 구독자 역할을 하는 옵저버 패턴을 따름. LiveData 값이 변경될 될 때마다 구독자에게 실시간 업데이트를 용이하게 함
(3) 일회성 이벤트: 토스트, 스낵바나 화면 이동 같은 일화성 이벤트에도 사용될 수 있으나 이 때는 SingleLiveEvent 또는 유사하게 커스텀해서 처리해야 함

### Pro Tips for Mastery: LiveData에서 `setValue()`와 `postValue()` 메서드의 차이점은 무엇인가요?
- `setValue()`와 `postValue()`는 LiveData 객체가 보유한 데이터를 업데이트하는 데 사용되지만, 특히 스레딩 및 동기화 측면에서 다른 사용 사례와 동작을 가짐

#### `setValue()`
- 데이터를 동기적으로 업데이트하며 메인 스레드에서만 호출할 수 있음
- 값을 즉시 업데이트하고 변경 사항이 (동일한 프레임) 동안 관찰자에게 반영되도록 해야 할 때 사용됨
- 백그라운드 스레드에서 호출하면 예외가 발생함

#### `postValue()`
- 데이터를 비동기적으로 업데이트하는 데 사용됨
- 백그라운드 스레드에서 UI를 업데이트해야 하는 경우에 적합함
- 호출하면 메인 스레드에서 업데이트 발생하도록 예약하여 현재 스레드를 차단하지 않고 스레드 안전성을 보장함

```
protected void postValue(T value) {
    boolean postTask;
    synchronized (mDataLock) {
        postTask = mPendingData == NOT_SET; // 이전에 전달된 작업이 없는지 확인
        mPendingData = value; // 보류 중인 데이터 업데이트
    }
    if (!postTask) {
        return; // 이미 전달된 작업이 있으면 바환
    }
    // 메인스레드 실행자에 Runnable 실행
    ArchTaskExecutor.getInstance().postToMainThread(mPostValueRunnable);
}
```
(1) mDataLock에서 동기화하고 mPendingData를 업데이트하여 작업을 실행해야 하는지 확인
(2) 값이 이미 보류 중이면 중복 실행을 피함(return)
(3) 그렇지 않다면 mPostValueRunnable을 ArchTaskExecutor를 통해 메인스레드에서 실행되도록 예약하여 스레드 안전한 업데이트를 보장함


## Q54) Jetpack ViewModel에 대해 설명해 주세요
- Jetpack ViewModel은 생명주기를 인식하는 방식으로 UI 관련 데이터를 저장하고 관리하도록 설계된 AAC의 핵심 구성 요소
- 화면 회전과 같은 구성 변경 시에도 데이터가 유지되도록 보장하면서 UI 로직과 비즈니스 로직을 분리하여 개발자가 견고하고 유지 관리 가능한 앱을 만드는 데 도움을 줌
- 주요 목적은 구성 변경 중에 UI 관련 데이터를 보존하는 것

### ViewModel의 특징
(1) 생명주기 인식: ViewModel은 Activity/Fragment의 생명주기에 범위가 지정되어 해당 UI 컴포넌트가 더 이상 사용되지 않을 때 자동으로 소멸됨
(2) 구성 변경 간 지속성: 구성 변경 중에 소멸되고 다시 생성되는 Acctivity/Fragment와 달리 ViewModel은 상태를 유지하여 데이터 손실을 방지하고 데이터의 반복적인 재로드를 피함
(3) 관심사 분리: UI 로직과 비즈니스 로직을 분리하여 더 깔끔하고 유지 관리하기 쉬운 코드를 설계하는 데 도움이 됨

### ViewModel 생명주기
- ViewModel 인스턴스는 ViewModel 인스턴스의 생명주기를 관리하기 위한 메커니즘 역할을 하는 ViewModelStoreOwner에 스코프가 지정됨
- ViewModelStoreOwner는 Activity, Fragment, Navigation 그래프 또는 그래프 내 대상 등이 될 수 있음

### Pro Tips for Mastery: ViewModel의 생명주기는 어떻게 되나요?
- ViewModel의 생명주기는 ViewModelStoreOwner에 연결됨
- ViewModel은 ViewModelStoreOwner의 범위 내에서 존재하며, 화면 회전과 같은 구성 변경 시에도 데이터와 상태가 유지되도록 보장함

- ViewModelStoreOwner가 처음 생성될 때 ViewModel 인스턴스가 초기화됨
- ViewModelStoreOwner가 메모리에 남아 있는 한 동일한 ViewModel 인스턴스가 유지됨
- 기기 회전과 같은 구성 변경이 발생하면 ViewModelStoreOwner는 다시 생성되지만 기존 ViewModel 인스턴스가 재사용되어 데이터를 유지함
  -> 새로운 ViewModelStoreOwner가 생성될 때, 이전 ViewModelStoreOwner로부터 기존 ViewModel 객체의 정보를 가져오기 때문에 가능함
- `onDestroy()`가 구성 변경으로 인해 트리거되지 않은경우 ViewModelStore의 `clear()` 메서드가 호출됨

### Pro Tips for Mastery: AAC ViewModel과 MVVM ViewModel의 차이점에 대해서 설명해주세요
(1) AAC ViewModel
- 비즈니스 로직 또는 UI 상태 홀더 역항르 하도록 설계된 생명주기를 인식하는 컴포넌트
- 관련 비즈니스 로직을 캡슐화하면서 상태를 관리하고 UI에 제공함
- 상태를 보유하고 화면 회전과 같은 구성 변경 시에도 유지하는 장점이 있음

(2) MVVM ViewModel
- View와 Model 간의 다리 역할
- View가 상태 변경에 반응할 수 있도록 하는 바인딩 메커니즘을 강조하여 더 선언적이고 모듈화된 설계를 용이하게 함


## Q55) Jetpack Navigation 라이브러리란 무엇인가요?



## Q56) Dagger2와 Hilt의 동작원리 및 차이점에 대해서 설명해주세요



## Q57) Jetpack Paging 라이브러리는 어떤 메커니즘으로 동작하나요?



## Q58) Baseline Profile은 앱의 성능에 어떤 이점을 가져다주나요?


