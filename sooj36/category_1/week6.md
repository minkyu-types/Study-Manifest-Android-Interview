**안드로이드 면접 질문 [250811]**

  

## Q) 31. 애플리케이션 용량을 어떻게 줄이나요?

  

| 번호 | 최적화 항목 |
|---|---|
| 1 | 사용하지 않는 리소스 제거하기 |
| 2 | R8로 코드 축소 활성화하기 |
| 3 | 리소스 최적화 사용하기 |
| 4 | AAB(Android App Bundle) 사용하기 |
| 5 | 불필요한 의존성 제거하기 |
| 6 | 네이티브 라이브러리 최적화하기 |
| 7 | Proguard 규칙을 구성하여 디버그 정보 줄이기 |
| 8 | 동적 기능(Dynamic Features) 사용하기 |
| 9 | 앱 내 대용량 애셋 피하기 |


  

**전략1**. 사용하지 않는 리소스 제거

    - 이미지, 레이아웃, 문자열 등 앱에서 사용되지 않는 리소스는 불필요하게 APK 또는 AAB의 크기를 증가시킨다.

      Android Studio에 내장된 Lint 도구를 사용하면 이러한 리소스를 쉽게 식별하고 제거할 수 있다.

      또한, build.gradle 파일에서 shrinkResources를 true로 설정하면 빌드 과정에서 사용되지 않는 리소스를 자동으로 제거해준다.

  
  

```

android {

    buildTypes {

  

        debug {

            minifyEnabled false

            shrinkResources false

        }

  

        release {

            minifyEnabled true // R8 활성화(코드 축소)

            shrinkResources true // 리소스 축소 (minifyEnabled 필요)

            }

                }

                    }

  
  

```

  

**전략2.**

2. R8 이용한 코드 축소 및 최적화

    - 안드로이드의 기본 코드 축소 및 최적화 도구로, 사용되지 않는 클래스, 메서드, 필드를 제거하고 코드를 난독화하여 앱의 크기를 줄인다.

  

    minifyEnabled true 로 설정해서 R8를 활성화할 수 있다

  

--> (+) shrinkResources 는 minifyEnabled가 활성화된 상태에서만 제대로 작동할 수 있다.

    리소스 사용 여부를 판단하기 위해 먼저 코드 분석(R8의 역할)이 필요하기 때문이다.

  

/

  

**전략3**. 리소스 최적화 사용하기

- 1. (이미지) PNG/JPEG 대신 Vector Drawable or WebP 형식

- 2. (도구) TinyPNG 또는 ImageMagick 로 압축

  

```

android {

    defaultConfig {

        vectorDrawables.useSupportLibrary = true

    }

}

```

  

**전략4**. Android App Bundles (AAB) 사용

- AAB : Google Play가 기기별 최적화 APK 제공 (화면 밀도, CPU 아키텍처, 언어 분리).

  

```

android {

    bundle {

        density { enableSplit = true }

        abi { enableSplit = true }

        language { enableSplit = true }

    }

}

```

  

**전략5-6**. 불필요한 의존성 및 네이티브 라이브러리 최적화

- Gradle Dependency Analyzer로 중복 라이브러리 제거.

- 필요한 ABI만 포함 (armeabi-v7a, arm64-v8a)

  

```

android {

    defaultConfig {

        ndk {

            abiFilters "armeabi-v7a", "arm64-v8a"

        }

    }

}

  

```

  

**전략7**. Proguard 규칙을 구성하여 디버그 정보 줄이기

  

**전략8**. 동적 기능(Dynamic Features) 사용하기

- 자주 사용되지 않는 기능을 주문형 모듈로 분리하여 앱을 모듈화하여 초기 다운로드 용량을 줄인다

  

**전략9**. 앱 내 대용량 애셋 피하기

- 비디오나 고해상도 이미지 같은 대용량 에셋은 앱에 번들링하지 않고, 콘텐츠 전송 네트워크(CDN)에 호스팅한 후 런타임에 동적으로 로드

- 미디어 콘텐츠의 경우 스트리밍을 사용

  

#### 업데이트 :

- 2025년 11월 1일부터 Android 15 이상을 타겟하는 앱 중, 네이티브 코드(NDK)를 사용하는 앱은 16KB 페이지 크기를 필수로 지원해야 한다.

- 이 정책의 주 목적 역시 메모리 관리 효율을 높여 앱 실행 속도, 배터리 수명 및 전반적인 시스템 성능을 향상시키기 위함

  

---

  
  

## Q) 32. 안드로이드 애플리케이션의 프로세스(process)란 무엇이며, 안드로이드 운영 체제는 이를 어떻게 관리하나요?

  

##### (정의)

- 안드로이드에서 프로세스는 애플리케이션이 실행되는 환경으로,

각 앱은 다른 앱과 격리된 자체 프로세스에서 단일 실행 스레드(메인 스레드)로 작동

  

/

##### (작동방식)

- 리눅스 fork() 시스템 호출로 새 프로세스를 생성 →

- 각 프로세스는 Dalvik 또는 ART 가상 머신의 고유 인스턴스에서 실행을 보장 →

- 각 프로세스에 고유한 UID를 할당으로 엄격한 보안 경계 적용

  

/

##### (컴포넌트 - 프로세스 연결)

- 기본적으로 모든 컴포넌트는 동일한 프로세스에서 실행

- 개발자는 AndroidManifest.xml의 android:process 속성으로 커스텀 설정이 가능

    - (개별 컴포넌트 설정): <activity>, <service>, <receiver>, <provider> 각각에 독립적으로 적용

    - (전역 설정): <application> 요소에 적용하여 모든 컴포넌트의 기본 프로세스 정의

    - 앱의 특정 기능을 별도의 프로세스에서 실행시킬 수 있다.

        안드로이드는 **우선순위 기반**으로 프로세스를 동적 관리하며, 메모리 상황과 앱의 현재 상태에 따라 낮은 우선순위 프로세스부터 순차적으로 종료시켜 시스템 성능을 최적화한다.

/

##### (우선순위 계층)

  

| 프로세스 종류 | 정의 | 우선순위 | 특징 |
|---|---|---|---|
| 포그라운드 프로세스 | 사용자와 활발하게 상호작용하며 실행 중인 프로세스 | 최고 | 거의 종료되지 않음 |
| 보이는 프로세스 | 사용자에게 보이지만 활발하게 상호작용하지 않는 프로세스 | 높음 | 다이얼로그 뒤의 Activity 등 |
| 서비스 프로세스 | 백그라운드 Service를 실행하는 프로세스 | 중간 | 데이터 동기화, 음악 재생 등 |
| 캐시된 프로세스 | 빠른 재실행을 위해 메모리에 유지되는 유휴 프로세스 | 최저 | 메모리 부족 시 가장 먼저 종료 |


  

/

##### (보안 및 권한)

- 각 안드로이드 프로세스는

리눅스 보안 모델로 샌드박스 처리되어 명시적 권한 부여 없이는 다른 프로세스 데이터에 접근할 수 없다.

- 이 격리 시스템은 멀티태스킹 환경에서 시스템 안정성과 데이터 개인정보 보호를 동시에 보장하는 안드로이드의 핵심 보안 기반이다.

  

/

##### (실무TIP) :

- 크래시 위험이 높은 기능을 별도 프로세스로 분리하는 것.

- 특히 광고 SDK나 실험적 기능들을 격리시켜두면, 그쪽에서 문제 생겨도 메인 앱은 안전히 돌아감

  

- 프로세스 분리는 양날의 검이다.

  

- 장점 => 격리성

- 단점 => 프로세스 간 통신의 오버헤드

    - 각 프로세스마다 별도의 메모리를 사용하기 때문에, 전체적으로 메모리 사용량이 늘어남

  
  

  /

### 실전 질문

-

  

**Q) 안드로이드는 메모리가 부족할 때 어떤 프로세스를 종료할지 결정하기 위해 우선순위 기반 프로세스 관리 시스템을 사용합니다. 시스템이 프로세스 우선순위를 어떻게 정하는지, 그리고 중요한 프로세스가 종료되는 것을 방지하기 위해 개발자가 따라야 할 전략은 무엇인지 설명해 주세요.**

  

##### (개발자 전략)

- 1. 핵심 백그라운드 작업에 Foreground Service 사용

: 음악재생이나 네비게이션처럼 중단되면 안되는 중요한 작업에 "포그라운드 서비스"를 사용.

    - 포그라운드 서비스를 사용하면 사용자에게 상태바 알림이 항상 표시되고,

    시스템은 이 프로세스를 '보이는 프로세스'나 '포그라운드 프로세스'에 준하는 중요한 작업으로 인지

  

- 2. 컴포넌트 생명주기

: 중요한 데이터 처리나 다운로드 같은 작업을 Activity(화면)에 직접 연결하면, 사용자가 다른 앱으로 전환하는 순간 해당 Activity는 '보이지 않는' 상태가 되어 프로세스 우선순위가 낮아짐.

따라서 이러한 작업은 화면과 독립적으로 동작하는 Service 컴포넌트로 옮겨서 관리.

  

- 3. 모든 백그라운드 작업에 항상 Service가 필요한 것은 아님.

    간단한 데이터 동기화나 주기적인 작업은 안드로이드이가 제공하는 WorkManager 같은 API를 사용하는 것이 좋음

  

### Pro Tips for Mastery:

안드로이드의 4대 주요 컴포넌트라고 불리는 이유는 무엇인가요?

- **프로세스 독립성**

  

- 안드로이드 앱이 시스템/다른 앱과 상호작용하는 공식 진입점이자 실행 단위이기 때문.

- 각 컴포넌트는 생명주기, 권한, IPC 모델을 통해 앱 동작을 정의하고, OS가 프로세스를 관리, 기동, 종료하는 근거가 됨

- 트리거 시 시스템이 프로세스를 새로 시작할 수 있으며, android:process로 멀티프로세스 구성도 가능.

  

- ##### 프로세스와의 연결 요지

    - [Activity]: 화면 진입 시 프로세스가 없으면 생성, 프로세스 종료 시 Activity도 소멸.

    - [Service]: UI 없이 백그라운드 작업, 동일/별도 프로세스 선택 가능.

    - [BroadcastReceiver]: 앱이 꺼져 있어도 브로드캐스트로 프로세스 기동 트리거.

    - [ContentProvider]: IPC를 통한 데이터 공유 지점, 접근 시 프로세스 기동 가능.

  

---

---

  

  

## Q) ==33. View 생명주기를 설명해주세요==


  - **6단계 : 생성 → 측정 & 배치 → 그리기 → 이벤트 처리 → 분리 → 소멸**

1. View 생성(onAttachedToWindow)
	- View가 화면에 들어갈 준비를 하며, 리스너 연결, 애니메이션 준비 같은 '초기화' 작업
<br>
2.  측정 및 배치(onMeasure / onLayout)
	- "얼마나 넓고 높아야 할지"를 계산하여 부모 레이아웃의 규칙을 보고 자신의 가로/세로를 정함
	- "화면에서 정확히 어디에 둘지"를 결정하며, ViewGroup이라면 자식 View들도 각각 배치
<br>
3. 그리기(onDraw)
	- 실제 화면에 색, 텍스트, 이미지 등을 그리는데 여기서 Canvas를 사용
<br>
4. 이벤트 처리(onTouchEvent/onClick)
	- 터치, 클릭 같은 사용자 입력에 반응
<br>
5. 분리(onDetachedFromWindow)
	- 화면에서 사라질 때 리소스를 정리
	- ex: 애니메이션 멈추기, 리스너 해제, 코루틴/쓰레드 취소 등
<br>
6. 소멸(GC)
	- 더 이상 쓰지 않는 메모리를 정리
	- 이 단계에서 누수를 막으려면 참조/리스너를 해제해야 함

<br>
<br>

#####  Q33. 실전 질문
<br>

**Q) 이미지 로딩이나 애니메이션 설정과 같이 비용이 많이 드는 기능을 포함하는 커스텀 View를 만든다고 가정해 봅시다. 
View 생명주기의 어느 시점에서 이러한 리소스 및 기능을 초기화해야 하며, 메모리 누수를 방지하기 어떻게 방지할 수 있나요?**
- 1) `리소스 초기화와 해제는 View가 윈도우에 연결되고 분리되는 시점`을 활용하는 것이 가장 이상적
	- // 리소스 및 기능 초기화 시점: `onAttachedToWindow()`
  - 비용이 많이 드는 리소스(이미지 로딩, 애니메이션)는 View가 화면에 실제로 연결될 준비를 마쳤을 때 초기화
  
<br>

- 2) `메모리 누수 방지 : View가 화면에서 제거될 때 사용했던 모든 리소스를 반드시 해제 해야함`
	
  
<br>

**Q) 애플리케이션에 성능 문제가 발생하는 동적으로 생성된 View를 포함하는 복잡한 레이아웃이 있습니다. 
적절한 응답성을 유지하면서 렌더링 효율성을 향상시키기 위해 onMeasure() 및 onLayout() 메서드를 어떻게 최적화할 수 있을까요?**

```
- 겹겹이 레이아웃을 줄여 화면 구조를 단순하게 만들고, 필요시에만 뷰를 만들고 
→ (뷰 계층 단순화 및 지연 로딩)

- 크기 재기(onMeasure)은 같은 조건이면 결과를 저장하여 재사용하고, 복잡한 계산은 한 번만
→ (측정 결과 캐싱(Caching))

- 모양만 바뀌면 다시 그리기(invalidate)만, 크기까지 바뀌면 레이아웃 다시 하기(requestLayout)
→ (적절한 갱신 요청)

- 스크롤 목록은 RecyclerView로 재사용하고, 그릴 때는 새 객체를 만들지 말고 미리 만든 도구(Paint)를 계속 쓸 것
→ (재사용 및 객체 할당 방지)

```


### Pro Tips for Mastery:

View의 findViewTreeLifecycleOwner() 함수는 어떤 역할을 하나요?

```
= 이 View가 속한 트리에서 가장 가까운 LifecycleOwner가 누군지 찾아주는 함수. 못 찾으면 null 부여

= LiveData 구독, 코루틴 수집, Observer 등록 같은 일을 "Activity/Fragment 생명주기에 맞춰 시작/정지 하기 위해. 그러면 화면이 닫힐 때 자동으로 정리돼서 메모리 누수를 막을 수 있음

= View가  액티비티/프래그먼트에 직접 의존하지 않고도 생명주기에 맞춰 안전하게 동작하나, 가까운 Activity/Fragment가 없는 화면 구조라면 null이 올 수 있으므로, 사용 전 null 체크 해야함

```

---
---


## Q) 34. View와 ViewGroup의 차이점은 무엇인가요?

```
(1) View 란 : 화면에 보이는 '한 조각'으로 버튼, 텍스트, 이미지 같은 단일 요소를 뜻함

* Additional Tip
- View는 안드로이드 UI 최소 단위이며, 렌더링/이벤트/업데이트를 모두 담당하는 무거운 객체로
코드 한두줄로 만들 수 있지만 내부적으로는 큰 오버헤드가 있을 수 있다

- View.java가 수만 줄 규모인 것처럼, 뷰 인스턴스를 많이 만들수록 메모리 사용과 측정/레이아웃/드로잉 비용이 커져 프레임 드랍을 유발

- 조건부 UI는 지연생성(ViewStub/동적 인플레이트)으로 초기 렌더링 부하를 낮춤

- 결론으론 **적게, 단순하게, 필요할 때만** 뷰를 만들면 전체 렌더링이 가벼워지고 응답성이 좋아진다.

```

```

(2) ViewGroup 이란 : 여러 View 조각을 담는 컨테이너로
- 세로 쌓기(LinearLayout), 복잡한 배치(ConstraintLayout) 같은 배치 규칙을 가지고, 자식들의 크기와 위치를 정한다.

- 독립적인 뷰 하나보다 더 많은 작업을 하기 때문에, 너무 많이 겹겹이 쓰게되면 화면이 느려질 수 있다

- 역할)
	1. 자식 뷰들의 크기/위치/그리는 순서를 관리 (ViewParent)
	2. 자식을 동적으로 추가/삭제할 수 있게 해줌 (ViewManager)



```

| 구분 | View | ViewGroup |
|---|---|---|
| 계층 | 리프 노드(leaf node)(자식 없음) | 브랜치 노드(branch node)(자식 View/ViewGroup 포함) |
| 레이아웃 동작 | 자신의 크기·위치만 가짐 | 레이아웃 규칙(예: Linear/Constraint)로 자식의 크기·위치 결정 |
| 상호작용 처리 | 터치·키 이벤트 직접 처리 | onInterceptTouchEvent 등으로 자식 이벤트 가로채 관리 가능 |
| 성능 관점 | 상대적으로 단순 | 중첩이 많을수록 측정/배치/그리기 비용 증가 → 지연 유발 가능 |


  -

  

### 실전 질문


**Q) View 생명주기에서 requestLayout(), invalidate(), postInvalidate()가 어떻게 작동하는지 설명하고 각각 언제 사용해야 하나요?**


```
(1) requestLayout()
	- 동작 : "크기/배치가 달라졌을 수 있다"라고 알려 레이아웃 패스를 다시 돌림
	(onMeasure → onLayout 재실행, 그 후 필요 시 그리기)

	- 사용 : 뷰의 크기나 위치에 영향을 주는 속성 변경 시(패딩, 텍스트 크기, 레이아웃 파라미터 변경 등)

(2) invalidate()
	- 동작 : "모양이 바뀌었다"라고 알려 다시 그리기만 요청
	(onDraw 재호출, 측정/레이아웃은 건드리지 않음)

	- 사용 : 색상, 텍스트 내용, 프로그레스 값 등 크기/배치에 영향 없는 화면 내용 변경 시

(3) postInvalidate()
	- 동작 :invalidate를 UI스레드에서 안전하게 수행하도록 예약
	(백그라운드 스레드에서 호출해도 메인 스레드에서 그리기 재요청)

	- 사용 : 워커 스레드에서 화면 갱신이 필요할 때(코루틴/IO 콜백 등)

=> 
	- 크기 영향 X ▷ invalidate
	- 크기/배치 영향 있음 → requestLayout(필요 시 invalidate 병행).
	
```
  

**Q) View 생명주기는 Activity 생명주기와 어떻게 다르며, 효율적인 UI 렌더링을 위해 둘 다 이해하는 것이 왜 중요한가요?**

  
```
1. 범위가 다르다
2. 책임이 다르다

	1. 범위
		- Activity 생명주기 : 화면(페이지) 단위의 **시작 - 일시중지 - 재개 - 종료**를 관리
		(onCreate/onStart/onResume/onPause/onStop/onDestroy)

		- View 생명주기 : 화면 조각(버튼/텍스트 등) 단위의 **부착 - 측정 - 배치 - 그리기 - 분리**를 관리 
		(onAttachedToWindow/onMeasure/onLayout/onDraw/onDetachedFromWindow)


	2. 책임
		- Activity : 리소스 준비·권한·네비게이션·상태 보존 등 “화면 전체”를 통제
		- View : 크기 계산과 렌더링, 사용자 입력 처리 등 “UI 요소 자체”를 통제.

	3. 두 생명주기의 이해 필요성
		1. 효율적 렌더링 
			: 무거운 초기화는 View의 onAttachedToWindow에서, 
				크기 의존 계산은 onSizeChanged에서,
				정리는 onDetachedFromWindow에서 수행해 
				불필요한 re-measure/re-layout을 줄인다.

				화면 전역 리소스는 Activity 생명주기에 맞춰 준비/해제해 낭비를 막는다.

		2. 버그/누수 방지
			: Activity가 사라질 때 View가 등록한 리스너·코루틴·애니메이션을 해제하지 않으면 누수와 잦은 레이아웃 패스 유발. 

			// * 레이아웃 패스 : onMeasure + onLayout
			
			반대로 View 변화가 Activity 상태와 맞물리면 타이밍 이슈(예: 아직 attach 전)로 크래시 가능

		3. 성능 최적화
			: 크기·배치가 바뀌면 requestLayout, 
			모양만 바뀌면 invalidate처럼 View 규칙을 지키고, 
			화면 전환/백그라운드 시점은 Activity 규칙을 지켜 작업을 중단해 프레임 드랍을 예방.


```
  

---
---


## Q) 35. ViewStub이란 무엇이고, 이를 사용하여 UI 성능을 최적화해 본 경험이 있나요?

  ```
  ViewStub : 보이지 않는 아주 가벼운 '플레이스홀더' 뷰로, 필요할 때까지 실제 레이아웃 inflate를 미룸

(특징)
1. 메모리 공간 최소화되어 가벼움
2. 인플레이션 지연 
3. 일회성 

(사용 사례)
1. 조건부 레이아웃
2. 초기 렌더링 시간 절감
3. 동적 UI

(장점)
1. 처음에 바로 만들지 않아, 초기 로딩이 빠름
2. 조건부 UI를 필요할 때만 만들게 하여 관리가 쉽고 코드가 단순

(한계)
1. 한 번 inflate하면 ViewStub 자체가 사라지고 레이아웃으로 바뀌므로 다시 ViewStub으로 되돌릴 수 없고, 이미 만들어진 뒤에 또 inflate를 부르면 예외가 날 수 있기에 만들어진 뷰를 찾아 쓰면 된다.

  ```

  -

  

### 실전 질문

**Q) ViewStub이 인플레이션될 때 어떤 일이 발생하며, 레이아웃 성능 및 메모리 사용량 측면에서 뷰 계층 구조에 어떤 영향을 미치나요?**
  
```
- inflate()를 호출하면 보이지 않던 ViewStub 자리에 지정한 실제 레이아웃이 생성되어 교체되며,
	새로 생긴 자식 뷰들이 계층에 편입된다

- '레이아웃 성능'에 미치는 영향
	: 초기 렌더링 향상 / 필요 시점 비용 전가 / 전체 패스 감수

- '메모리 사용'에 미치는 영향
	: 초기 메모리 절감 / 총량 최적화

```


---

---


## Q) 36. 커스텀 뷰(custom views)는 어떻게 구현하나요?

```
-  안드로이드에서 기본 제공하는 UI 컴포넌트로는 구현할 수 없는 독특한 디자인이나 기능이 필요할 때, 직접 만드는 UI 컴포넌트

- (1). 커스텀 View 클래스 생성

	- View(또는 ImageView/TextView 등)를 상속해서 새 클래스 생성
	- 생성자에서 기본값을 준비하고, 그릴 도구(Paint 등)를 만들어 둠

class CustomCircleView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyle: Int = 0
) : View(context, attrs, defStyle) {

    private val paint = Paint(Paint.ANTI_ALIAS_FLAG).apply { // Anti-aliasing 추가
        color = Color.RED
        style = Paint.Style.FILL
    }

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        // 중앙에 빨간색 원 그리기
        canvas.drawCircle(
            width / 2f,
            height / 2f,
            min(width, height) / 4f,
            paint
        ) // 반지름 수정
    }
}


..

- (2). XML 레이아웃에서 커스텀 View 사용하기

<com.example.myapp.ui.CustomCircleView
    android:id="@+id/customCircleView"
    android:layout_width="100dp"
    android:layout_height="100dp"
    android:layout_gravity="center" />

- 레이아웃XML에서 커스텀 뷰를 사용할 때는 클래스의 전체 패키지 경로를 태그로 지정해야 하며,
	일반 뷰처럼 너비/높이 등 속성을 그대로 줄 수 있으며, 정의해둔 커스텀 속성도 함께 전달할 수 있음

- (3). 커스텀 속성 추가하기 (선택 사항)

	- 1. 커스텀 속성 선언(attrs.xml)
		=> 내 뷰만의 설정값(색, 반지름)을 커스텀 속성으로 선언언

<resources>
    <declare-styleable name="CustomCircleView">
        <attr name="circleColor" format="color" />
        <attr name="circleRadius" format="dimension" />
    </declare-styleable>
</resources>

	- 2. 커스텀 속성 읽기 + + TypedArray recycle
	(obtainStyledAttributes로 값을 읽고, TypedArray는 finally에서 recycle)

	class CustomCircleView @JvmOverloads constructor(
    context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {

    var circleColor: Int = Color.RED
    var circleRadius: Float = 50f

    init {
        attrs?.let {
            val ta = context.obtainStyledAttributes(it, R.styleable.CustomCircleView, defStyleAttr, 0)
            try {
                circleColor = ta.getColor(R.styleable.CustomCircleView_circleColor, Color.RED)
                circleRadius = ta.getDimension(R.styleable.CustomCircleView_circleRadius, 50f)
            } finally {
                ta.recycle() // 반드시 해제
            }
        }
    }
}
	- 3. onDraw와(필요 시) onMeasure에서 속성 반영
	=> onDraw에서 속성으로 그리기, 크기 규칙이 필요하면 onMeasure에서 계산

	private val paint = Paint(Paint.ANTI_ALIAS_FLAG).apply {
    style = Paint.Style.FILL
}

override fun onDraw(canvas: Canvas) {
    super.onDraw(canvas)
    paint.color = circleColor
    canvas.drawCircle(width / 2f, height / 2f, circleRadius, paint)
}

override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
    val desiredW = (circleRadius * 2 + paddingLeft + paddingRight).toInt()
    val desiredH = (circleRadius * 2 + paddingTop + paddingBottom).toInt()
    val w = resolveSize(desiredW, widthMeasureSpec)
    val h = resolveSize(desiredH, heightMeasureSpec)
    setMeasuredDimension(w, h)
}


	- 4. 런타임 변경 시 requestLayout/invalidate 적절히 호출
	=> 크기 영향 있으면 requestLayout, 모양만 바뀌면 invalidate
	
fun setCircleProperties(color: Int, radius: Float) {
    circleColor = color
    circleRadius = radius
    // 크기 변화 가능성이 있으면 레이아웃 재요청
    requestLayout()
    // 모양 변화는 다시 그리기
    invalidate()
}

	- 5. XML에서 커스텀 뷰 사용 + 커스텀 속성 지정
	=> 풀 패키지 이름으로 태그를 쓰고, app 네임스페이스로 커스텀 속성 전달

<com.example.myapp.ui.CustomCircleView
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/customCircleView"
    android:layout_width="100dp"
    android:layout_height="100dp"
    app:circleColor="@color/blue"
    app:circleRadius="30dp" />


- (4). 레이아웃 측정 처리하기 (선택 사항)
	=> 커스텀 뷰가 크기를 측정하는 방식을 수동으로 처리하고 싶고, 특히 표준적인 뷰와 다르게 동작해야 하는 경우 onMeasure() 메서드를 재정의하여 구현할 수 있음

override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
    // 원하는 기본 크기 설정
    val desiredWidth = (paddingLeft + paddingRight + suggestedMinimumWidth)
        .coerceAtLeast(200) // 예시 기본 너비
    val desiredHeight = (paddingTop + paddingBottom + suggestedMinimumHeight)
        .coerceAtLeast(200) // 예시 기본 높이

    // MeasureSpec 모드와 크기를 기반으로 최종 크기 결정
    val width = resolveSize(desiredWidth, widthMeasureSpec)
    val height = resolveSize(desiredHeight, heightMeasureSpec)

    // 최종 측정된 크기 설정
    setMeasuredDimension(width, height)
}


```

### Pro Tips for Mastery: 커스텀 뷰의 기본 생성자에서 @JvmOverloads를 사용할 때 왜 주의해야 하나요?

```
커스텀 뷰를 만들 때 @JvmOverloads를 쓰면, 기본 디자인(스타일)이 끊겨 모양이 달라질 수 있음

따라서, "3가지 정보를 받는 생성자"에서 그 뷰가 써야 하는 기본 스타일을 함께 적어줘야 한다

예를 들어, 글쓰기칸(editText)계열이면, EditText용 기본 스타일을 넣어줘야 테마대로 상속이 보장

```

---
---

  

  

## Q) 37. Canvas란 무엇이며 어떻게 활용하나요?

  

-  Canvas는 2D 드로잉 표면의 인터페이스로, Paint 클래스와 긴밀하게 협업해 픽셀 레벨의 정밀한 제어를 가공.

- 게임이나 데이터를 시각화에서는 표준 View 컴포넌트로 불가능한 복잡한 그래픽을 구현할 때 필수적

  

```

class CustomView(context : Context) : View(context) {

    private val paint = Paint().apply {

        color = Color.BLUE

        style = Paint.Style.FILL

    }

  

    override fun onDraw(canvas : Canvas) {

        super.onDraw(canvas)

        // View 중앙에 파란색 원 그리기

        canvas.drawCircle(width / 2f, height / 2f, 100f, paint)

    }

}

```

## **🎨 Canvas 핵심 기능 요약**

  

### ** 기본 드로잉 작업**

| 기능 | 메서드 | 용도 |
|---|---|---|
| 도형 | `drawCircle()`, `drawRect()`, `drawLine()` | 원, 사각형, 선과 같은 도형 |
| 텍스트 | `drawText()` | 지정 위치에 텍스트 렌더링 |
| 이미지 | `drawBitmap()` | 비트맵 이미지 렌더링 |
| 복잡한 모양 | `Path` 객체 + `drawPath()` | 커스텀 모양 그리기 |


  

### **좌표계 변환**

| 변환 타입 | 메서드 | 효과 |
|---|---|---|
| 이동 | `canvas.translate(dx, dy)` | 원점 위치 변경 |
| 크기 | `canvas.scale(sx, sy)` | 그리기 크기 조절 |
| 회전 | `canvas.rotate(degrees)` | 지정 각도로 회전 |


  

- 캔버스의 좌표계를 수정하여 복잡한 장면을 더 쉽게 그릴 수 있다.

  

### **중요 특징**

- **누적 적용**: 변환은 호출할 때마다 **누적**됨

- **전역 영향**: 이후 모든 드로잉 작업에 **지속적 영향**

  

**실무 팁**: 변환 사용 시 `save()`/`restore()`로 상태 관리 필수

- 특정 변환(회전, 크기 조절 등)을 일부 그래픽에만 적용하고 싶을 때,

canvas.save() 로 현재 상태를 저장하고 변환 및 드로잉 작업 수행 후 canvas.restore()를 호출하여 이전 상태로 되돌리는 패턴

  

-

  

### 실전 질문

  

**Q)

AndroidX 라이브러리에서 지원하지 않는 복잡한 모양이나 UI 요소를 렌더링 하는 커스텀 뷰를 어떻게 만들 수 있을까요?

예를 들어, 화면에 로딩 중 상태를 표현하는 커스텀 스피너를 직접 그린다면,

어떤 Canvas 메서드와 API를 활용할 수 있을까요?**

  

- View를 상속하는 커스텀 클래스를 만들고, onDraw() 메서드 안에서 Canvas API를 활용해 시간에 따라 모양이나 각도가 변하는 도형을 그리는 것.

  

1. 원의 일부인 호(arc)를 그리고 계속 회전시키거나, 호의 길이(sweep)를 늘렸다 줄이며 생동감 부여

2. 애니메이션은 ValueAnimaor / 프레임 콜백(postInvalidateOnAnimation)을 통해 부드럽게 구현

  

##### 구현

- **(1) 클래스 생성**

= View를 상속받는 커스텀 뷰 클래스 정의

```

class CustomSpinner(context: Context, attrs: AttributeSet? = null) : View(context, attrs) {

    // ... 구현 내용

}

  

```

  

/

- **(2) Paint 및 상태 변수 초기화**

= 스피너 모양을 결정할 Paint 객체와 애니메이션에 사용될 상태 변수들을 선언

  

```

private val paint = Paint(Paint.ANTI_ALIAS_FLAG).apply {

    style = Paint.Style.STROKE

    strokeCap = Paint.Cap.ROUND

    strokeWidth = 12f // 예시 두께

}

private var rotationAngle = 0f

private var sweepAngle = 120f

  

```

  

/

- **(3). 크기 계산(onSizeChanged)**

```
    - 뷰의 크기가 결정되면, 스피너를 그릴 실제 영역(RectF)과 중심점을 계산

// (패딩과 선 두께(strokeWidth)를 모두 반영하여 경계를 설정해야 스피너가 잘리지 않음)

```
  

/

- **(4). 뷰 측정 (onMeasure)**

```
    - 레이아웃에서 wrap_content가 사용되었을 때를 대비해 합리적인 기본 크기를 지정

    - resolveSize()를 사용하여 MeasureSpec에 따라 최종 크기를 결정하고 setMeasuredDimension()으로 설정

    // 즉, 부모 뷰의 요구사항(MeasureSpec)과 뷰가 제안하는 최소.권장 사이즈를 절충해 실제 최종 크기 확정

```
  

/

- **(5). 그리기 (onDraw)**

```
계산된 값들을 이용해 Canvas에 스피너를 그림

canvas.save()와 canvas.restore()를 사용해 캔버스의 상태(특히 회전)가 다른 드로잉에 영향을 주지 않도록 격리.

canvas.rotate()로 캔버스 자체를 회전시킨 후, canvas.drawArc()로 호를 그림
```

/

- **(6). 애니메이션 설정**

```
ValueAnimator를 사용해 회전과 길이 변화를 만듬

회전 애니메이터: 0°에서 360°까지 무한 반복하며, 회전 각도(rotationAngle)를 갱신

길이 애니메이터: 최소 각도와 최대 각도 사이를 왕복하며 호의 길이(sweepAngle)를 갱신

각 프레임이 갱신될 때마다 postInvalidateOnAnimation()을 호출하여 onDraw를 다시 실행시킴.
```

/

- **(7). 생명주기 관리**

뷰의 생명주기에 맞춰 애니메이션을 안전하게 시작하고 정지

onAttachedToWindow(): 뷰가 화면에 붙을 때 애니메이터를 시작

onDetachedFromWindow(): 뷰가 화면에서 떨어질 때 애니메이터를 취소하여 메모리 누수를 방지.


---
---