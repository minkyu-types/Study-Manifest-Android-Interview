### 기본 개념
Jetpack composesms 시스템적으로 Compose Compiler, Compose Runtime, Compose UI의 세 가지 계층 구조로 구성되어 있음
개발자들이 사용하는 대부분의 API들은 Runtime, UI 계층에서 제공됨

### Q) 0. Jetpack Compose의 동작 구조는 어떻게 이루어져 있나요?
Jetpakc Compose는 선언적 접근 방식을 사용하여 애플리케이션을 구축하기 위한 최신 UI 툴킷
Compiler, Runtime, UI의 세 가지 주요 계층으로 이루어져 있으며, 각 계층은 UI 코드를 상호 작용 가능한 애플리케이션으로 변환하는 데 중요한 역할을 함

#### Compose Compiler
- Kotlin으로 작성된 선언적 UI 코드를 Jetpack Compose가 실행할 수 있는 최적화된 코드로 변환하는 역할
- 컴파일 타임에 @Composable 함수를 처리하고, 필요한 UI 업데이트 및 recomposition 로직을 생성함
- 컴파일러는 Kotlin 컴파일러로 구현되어 효율적인 코드 생성을 보장하고 상태 관리/코드 최적화/더 나은 성능을 위한 람다 리프팅과 같은 기능을 지원함
- KAPT/KSP와 같은 기존 어노테이션 처리 도구와 달리, Compiler 플러그인은 FIR에서 직접 작동함(Frontend Intermediate Representation)
  - 그러므로 컴파일 타임에 정적 코드에 대해 더 자세하게 접근할 수 있으며,
  - 개발자가 작성한 Kotlin 소스 코드를 동적으로 변환하고
  - 최적화된 Java 바이트코드를 생성할 수 있음
- @Composable과 같은 Compose 라이브러리의 어노테이션은 코드 생성/recomposition 관리/성능 최적화와 같은 작업을 조율하는 Compose Compiler의 내부 메커니즘을 통해 작동함
  - 개발 효율성과 런타임 성능 향상

#### Compose Runtime
- recomposition 및 상태 관리를 지원하는 데 필요한 핵심 기능을 제공
  - 변경 가능한 상태(mutable state)를 처리하고,
  - 스냅샷(snapshots)을 관리하며,
  - 애플리케이션 상태가 변경될 때마다 UI 업데이트를 트리거
- Compose의 반응형 UI 시스템을 구동하는 핵심적인 역할
  - 상태 변경에 따라 올바른 UI 컴포넌트가 동적으로 업데이트되도록 보장함
- Gap buffer 자료 구조에서 영감을 받은 slot table을 사용하여 컴포지션 상태를 메모이징하는 방식으로 작동하며, 
  내부적으로는 반응형 UI 구축에 필수적인 중요한 작업을 수행함
  - 사이드 이펙트 관리
  - remember를 사용한 상태 보존
  - 상태 변경 시 recomposition 트리거
  - CompositionLocal을 사용한 컨텍스트별 데이터 저장
  - UI 계층 구조를 효율적으로 생성하기 위한 Compose 레이아웃 노드 구축
- 현재는 안드로이드 팀이 Gap buffer -> Link table 자료 구조로 마이그레이션 하는 중
  - 요소의 효율적인 삽입/삭제/재배열이 가능함

#### Compose UI
- 애플리케이션 구축을 위한 고수준 컴포넌트 및 UI 위젯을 제공함
- 텍스트/버튼/레이아웃 컨테이너와 같은 기본 요소와 커스텀 UI 컴포넌트 구축을 위한 더 상위호환의 API를 포함
- 내부적으로 Compose UI 모듈은 근본적으로 안드로이드의 전통적인 UI 시스템 위에서 동작하는 한 편,
  그 위에서 동작하는 Compose의 메커니즘이 따로 있기 때문에 생각보다 복잡한 관계를 이루고 있어 기존의 안드로이드 UI 시스템과 완전히 분리된 개념이라고 생각하기는 어려움
- Compose UI 라이브러리는 Compose Runtime에 의해 처리되는 Compose 레이아웃 트리 구축을 단순화하도록 설계된 광범위한 컴포넌트를 제공함
- KMP 지원을 통해 Jetbrains는 최근 CMP를 적극적으로 발전시켜 여러 플랫폼에서 동일한 Compose UI 라이브러리로 일관된 UI를 제공할 수 있도록 생태계를 만들고 있음

#### 실전 질문
Q) Compose Compiler의 역할은 무엇이고, KAPT 또는 KSP와 같은 전통적인 어노테이션 프로세서와 어떻게 다른가요?

Q) Compose Runtime은 recomposition과 상태를 어떻게 관리하며, 내부적으로 어떤 자료 구조를 사용하나요?


### Q) 1. Compose phase에 대해 설명해 주세요
Jetpack compose는 UI를 그릴 때 Composition -> Layout -> Drawing 의 3가지 주요 단계로 이루어진 렌더링 파이프라인을 순차적으로 따름

#### Composition
Composition 단계는 @Composable 함수를 실행하고 UI 트리를 구축하여 컴포저블 함수에 대한 설명을 생성하는 역할을 함
이 단계에서 Compose는 초기 UI 구조를 구축하고 Slot Table이라는 데이터 구조에 컴포저블 간의 관계를 기록함
상태 변경이 발생하면 Composition 단계는 영향을 받는 UI에 대해 다시 계산하고, 필요한 경우 recomposition을 트리거함

- 주요 작업
  - @Composable 함수 실행
  - UI 트리 생성 및 업데이트
  - recomposition을 위한 변경 사항 추적

#### Layout
Layout 단계는 Composition 단계 바로 직후에 수행됨
제공된 제약 조건에 따라 각 UI 컴포넌트의 크기/위치를 결정함
각 Composable은 자식 요소를 측정하고, 크기를 결정하며, 부모에 대한 상대적 위치를 설정함
- 주요 작업
  - UI 컴포넌트 측정
  - 너비,높이 및 위치 정의
  - 부모 컨테이너 내 자식 배치

#### Drawing
Drawing 단계는 앞선 Composition과 Layout 단계를 마친 UI 컴포넌트가 화면에 렌더링되는 절차
Compose는 안드로이드에서 UI들을 렌더링하기 위해 Skia 그래픽 엔진을 사용하여 하드웨어 가속 기반의 부드러운 렌더링을 제공함
- 주요 작업
  - 시각적 요소 렌더링
  - 화면에 UI 컴포넌트 그리기
  - 커스텀 드로잉 작업 적용

### Q) 2. Jetpack Compose가 선언적 UI 프레임워크라고 불리는 이유는 무엇인가요?
Jetpack Compose는 개발자가 상태 변경 시 UI를 어떻게 업데이트할 지를 나타내는 것이 아닌,
특정 상태에서 UI가 어떻게 보여야 하는지를 설명하는 선언적 UI 프레임워크의 특성을 가지고 있음
이는 개발자가 뷰를 업데이트하고 UI 일관성을 유지하기 위해 수동으로 UI를 업데이트하는 전통적인 명령형 UI 접근 방식과 비교됨

#### 선언적 UI의 주요 특징
- 상태 주도 UI
선언적 UI 프레임워크에서는 상태 관리 시스템이 라이브러리 자체에 내장되어 있음
시스템은 각 컴포넌트의 상태를 추적하고, 상태가 변경될 때 UI를 자동으로 업데이트함
상태가 변경될 때마다 프레임워크는 recomposition을 트리거하여 영향을 받는 UI 컴포넌트만 업데이트하고 최신 데이터를 반영하여 뷰를 수동적으로 관리할 필요가 없도록 함

- 컴포넌트를 함수/클래스로 정의
UI 컴포넌트를 함수/클래스로 표현되는 모듈식 컴포넌트로 정의되도록 함
이러한 컴포넌트는 UI 레이아웃과 동작을 모두 설명하여 XML 같은 마크업 언어와 Kotlin/Java 간의 간극을 줄임
Composable은 현재 상태를 기반으로 UI를 설명하고 다른 함수와 결합하여 모듈식 + 확장 가능한 구조를 만들 수 있음

- 직접적인 데이터 바인딩
개발자가 모델 데이터를 UI 컴포넌트에 직접 바인딩하여 수동적으로 데이터를 동기화할 필요가 없음
이런 접근 방식은 더 깔끔하고 유지 관리하기 쉬운 코드를 작성할 수 있음

- 컴포넌트 멱등성
선언적 프레임워크의 가장 핵심적인 특징은 멱등성임
개발자가 상태 조건을 Kotlin 내에 직접 포함시켜 논리적으로 UI 코드를 작성할 수 있음
이런 접근 방식은 상태 변경에 대해 UI가 자동으로 업데이트되도록 보장하고, 상태 관리/코드 가독성 모두를 단순화함

#### Jetpack Compose가 선언적 UI 프레임워크로 분류되는 이유
- 함수로 UI 정의
@Composable 어노테이션이 달린 함수는 Compose Compiler에 의해 해석 및 변환되어 선언적 UI 생성을 가능하게 함
- 상태 관리
Compose Runtime에서 제공하는 remember와 같은 함수는 컴포저블의 상태와 생명주기를 효율적으로 관리함
- 직접 데이터 바인딩
Composable 함수의 파라미터는 UI에 직접 바인딩되어 데이터가 UI 컴포넌트에 원활하게 연결될 수 있음
- 컴포넌트 멱등성
Composable은 동일한 입력 값에 대해 일관되게 동일한 UI 출력을 생성하여 예측 가능한 동작을 보장함


### Q) 3. recomposition이란 무엇이며 언제 발생하나요? 또한 앱 성능과 어떤 관련이 있나요?
Jetpack Compose는 이미 렌더링된 UI 레이아웃을 업데이트하기 위해 3가지 주요 단계를 통해 상태 변경이 발생할 때마다 UI를 다시 그리는 메커니즘을 사용하는데,
이러한 프로세스를 recomposition 이라고 함
recomposition이 발생하면 Compose는 Composition 단계부터 새롭게 시작하며, 여기서 컴포저블 노드는 UI 변경사항을 Compose Runtime에 알리고, 업데이트된 UI가 최신 상태를 반영하도록 보장함

#### Recomposition이 발생하는 조건
대부분의 모바일 애플리케이션에서는 앱 내 데이터 모델을 메모리에 담아 표현하는 '상태'를 유지함
상태가 변경됨에 따라 UI도 함께 동기화되도록 하기 위해 Jetpack Compose는 두 가지 메인 메커니즘을 통해 recomposition을 트리거함

(1) 매개변수에 변경이 발생했을 때(Input changes)
컴포저블 함수는 입력 매개변수가 변경될 때 recomposition을 트리거함
Compose Runtime은 equals() 함수를 사용하여 새 매개변수 값을 이전 매개변수 값과 비교함
비교 결과가 false면 런타임은 변경사항을 인지하고 recomposition을 트리거하여 동기화가 필요한 부분에 한해서만 UI를 업데이트함
(2) 상태 변경이 관찰되었을 때(Observing State Changes)
Jetpack Compose는 일반적으로 remember 함수와 State API를 함께 사용하여 상태 변경을 모니터링함
이러한 접근 방식은 상태 객체를 메모리에 보존하고 recomposition이 발생했을 때에도
메모리에 저장된 값을 복원하여 UI에 최신 상태를 일관되게 반영하도록 보장함

#### Recomposition과 성능
Jetpack Compose에서 recomposition은 UI가 상태 변화를 자동으로 감지하고 동기화될 수 있도록 하는 반응형 특성을 담당하는 핵심 기능이지만,
과도하거나 불필요한 recomposition은 앱 성능을 저하시킬 수 있음
따라서, recomposition 작동 방식과 이를 효과적으로 추적하는 방법을 이해하는 것은 Compose 애플리케이션 최적화에 필수적임
recomposition을 최적화하고 앱 성능을 향상시키려면
Layout Inspector를 사용하여 실제 앱 내에서 발생하는 recomposition 횟수를 추적함으로써 불필요한 recomposition을 식별하는 것이 효과적

Layout Inspector를 사용하면 에뮬레이터/실제 기기에서 실행중인 앱의 Compose 레이아웃을 트래킹할 수 있음
컴포저블이 얼마나 자주 recompose 되거나 recomposition을 건너뛰는지 모니터링하여 잠재적인 성능 문제를 식별하는 데 도움이 됨

프로젝트에서 recomposition 추적을 시험해볼 수 있는 또 다른 도구는 Composition Tracing임
이 도구는 성능 문제를 진단하는 데 유용한 도구로, 시스템 추적을 통해 오버헤드가 낮은 측정을 제공하고 메서드 추적을 통해
세부적인 함수 호출 추적을 제공하지만, 앱 성능에는 직접적으로 영향을 미치지 않음
시스템 추적은 일반적으로 개별적인 컴포저블 함수를 포함하지는 않는다는 점이 아쉬움

### Q) 4. Composable 함수는 내부적으로 어떻게 작동하나요?
Jetpack Compose는 일반적인 함수를 선언적 UI로 동작시키기 위해 @Composable 어노테이션을 사용함
해당 어노테이션이 붙은 함수는 컴파일 타임 시 Compose Compiler Plugin에 의해 개발자가 작성한 Kotlin 함수를
Jetpack Compose의 내부 메커니즘에 맞는 상태 기반 UI 코드로 변환됨

#### 컴파일러 변환
함수에 @Composable 어노테이션을 추가하면 해당 Compose Compiler Plugin이 Kotlin 컴파일 프로세스를 가로챔
컴파일러는 컴포저블 함수를 표준 Kotlin 함수로 취급하지 않고, Compose의 반응형 시스템을 추가하기 위해
기존 함수에 추가적인 매개변수와 각종 로직을 주입함
가장 중요한 매개변수 중 하나는 컴포지션 상태를 추적하고 UI 상태가 변경될 때 
recomposition을 처리하는 Composer 매개변수

Composer 매개변순는 컴파일 타임에 컴파일러가 주입하는 객체이기 때문에
개발자는 해당 존재의 유무 자체를 알지 못해도 되고, 최종적으로 사용되는 API의 형태는 굉장히 단순함
```
@Composable
fun MyComposable() {
  Text("Hell, Compose!")
}
```
내부적으로 이 메서드는 Composer 객체와 상태 관리를 위한 기타 메타데이터를 주입하여 완전히 새로운 버전의 함수로 변형함

#### 컴포지션 및 리컴포지션
Compose Runtime은 @Composable 함수의 생명주기를 관리함
Compose Phaes의 3단계 중 첫번째 단계인 Composition 단계에서 런타임은 컴포저블 함수를 실행하고 UI 트리를 구축함
해당 트리는 Slot Table이라는 데이터 구조에 저장되어 Compose가 UI를 효율적으로 관리하고 업데이트하는 역할을 담당함

만약 상태가 변경되면 Recomposition이 트리거됨
업데이트해야 하는 UI 트리를 다시 빌드하는 대신 Compose는 슬롯 테이블을 사용하여 UI의 어떤 부분을 업데이트해야 하는지 결정하고
해당 컴포저블 함수만 선택적으로 다시 실행함

#### Remember 및 상태 관리
상태 관리를 위해 Compose는 remember/State와 같은 API를 제공함
이러한 메커니즘은 런타임 및 컴파일러와 긴밀하게 작용하여,
recomposition으로부터 상태를 보존하여 UI가 앱의 데이터 모델과 일관성을 유지하도록 보장함

#### Pro Tips for Mastery: Compose Compiler와 Composer


### Q) 5. Jetpack Compose의 안정성이란 무엇이며, 성능과 어떤 관련이 있나요?



### Q) 6. 안전성 개선을 통해 Compose 성능을 최적화한 경험이 있나요?























