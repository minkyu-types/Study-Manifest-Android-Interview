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


### Q) 2. Jetpack Compose가 선언적 UI 프레임워크라고 불리는 이유는 무엇인가요?


### Q) 3. recomposition이란 무엇이며 언제 발생하나요? 또한 앱 성능과 어떤 관련이 있나요?


### Q) 4. Composable 함수는 내부적으로 어떻게 작동하나요?


### Q) 5. Jetpack Compose의 안정성이란 무엇이며, 성능과 어떤 관련이 있나요?


### Q) 6. 안전성 개선을 통해 Compose 성능을 최적화한 경험이 있나요?
