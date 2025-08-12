## 📚 Q31. 애플리케이션 용량을 어떻게 줄이나요?

### 🎯 개요
저장 공간이 제한적이거나 인터넷 연결이 느린 사용지의 사용 경험을 개선하는 데  
애플리케이션 용량을 최적화하는것은 필수적입니다.  

### 🛠️ 사용하지 않는 리소스 제거하기
사용되지 않는 리소스는 불필요하게 APK 또는 AAB 크기를 증가시킵니다.  
Android Studio의 Lint와 같은 도구는 이러한 리소스를 식별하는 데 도움이 될 수 있습니다.  
build.gradle 파일에서 shrinkResources를 활성화 하여 빌드 프로세스 중에 사용되지 않는 리소스를 자동으로 제거하도록 합니다.  

```groovy
android {
    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
        }
    }
}
```

#### ⚙️ R8로 코드 축소 활성화 하기
기본 코드 축소기 및 최적화 도구인 R8은 사용되지 않는 클래스와 메서드를 제거  
코드를 난독화하여 더 컴팩트하게 만듭니다.  
ProGuard를 적절하게 사용하면 중요한 코드나 리플렉션 기반 라이브러는 난독화를 생략하고  
오동작하지 않도록 보장합니다.  

#### ⚙️ 리소스 최적화 사용하기
이미지 및 XML 파일과 같은 리소스를 최적화하면 앱 용량을 크게 줄일 수 있습니다.  
- 벡터 드로어블  
  PNG, JPEG 대신에 용량을 덜 차지하는 벡터 드로어블로 대체합니다.  
  SVG 사용  
- 이미지 압축  
  TinyPNG 또는 ImageMagick과 같은 도구를 사용하여  
  눈에 띄는 품질 손상 없이 래스터 이미지를 압축합니다.  
  TinyPNG는 손실압축과 무손실 압축을 혼합해서 사용중
  ImageMagick은 PNG -> SVG 변환 자동화 가능
- WebP 형식  
  이미지를 PNG 또는 JPEG보다 압축률이 좋은 WebP 형식으로 변환합니다.

#### ⚙️ Android App Bundles 사용하기  
특정 구성에 필요한 리소스와 코드만 포함하여 앱 용량을 줄임

### 🛠️ 불필요한 의존성 줄이기  
Android Studio의 Gradle Dependency Analyzer를 사용하여  
무거운 라이브러리 및 전의 의존성을 식별할 수 있습니다.  

#### ⚙️ 네이티브 라이브러리 최적화하기
- 사용하지 않는 아키텍처 제외  
  build.gradle 파일의 abiFilters 옵션을 사용하여 필요한 ABI만 포함합니다. 
- 디버그 심볼 제외  
  stripDebugSymbols를 사용하여 네이티브 라이브러리에서 디버깅 심볼을 제거합니다.

#### ⚙️roguard 규칙을 구성하여 디버깅 정보 줄이기  
디버깅 메타데이터는 최종 APK 또는 AAB에 불필요한 무게를 더합니다.  
proguard-rules.pro 파일을 구성하여 이런 정보를 제거합니다.  
-dontwarn com.example.unusedlibrary.**  
-keep class com.example.imortant.** { *; }

#### ⚙️ 동적 기능 사용하기  
동적 기능 모듈을 사용하면 자주 사용하지 않는 기능은 주문형 모듈로 분리하여  
앱을 모듈화할 수 있습니다.  

#### ⚙️ 앱 내 대용량 에셋 피하기
- 비디오나 고해상도 이미지와 같은 대용량 에셋은 콘텐츠 전송 네트워크에 호스팅하고 런타임에 동적으로 로드합니다.
- 미디어 콘텐츠는 앱과 함께 번들링하는 대신 스트리밍을 사용합니다.  

#### 🧠 요약
R8, 리소스 최적화, App Bundle과 같은 최신 형식 활용 등 다양한 전략이 있습니다.   
의존성 검토, 네이티브 라이브러리 최적화, 기능 모듈화는 앱 용량을 최소화하는데 도움을 줍니다.  

#### ❓ 실전 질문
#### 시각적 품질을 유지하면서 이미지 리소스를 어떻게 최적화하고, 최대 효율성을 위해 어떤 이미지 포맷을 사용할 수 있을까요?  
- WebP를 사용하면 JPEG나 PNG보다 2~30% 정도 파일 크기가 작아집니다.(무손실, 손실압축 모두 지원)  
- SVG 파일을 사용해서 벡터 드로어블에 사용하기 쉽게 만듦  

#### ❓ 실전 질문
#### 사용자가 자주 사용하지 않는 기능을 필요할 때부터 사용할 수 있도록 하여 초기 앱 용량을 줄이는 방법은 무엇이 있을까요?

동적 기능을 사용하면 주문형 모듈로 분리하여 앱을 모듈화 할 수 있음

## 📚 Q32. 안드로이드 애플리케이션의 프로세스란 무엇이며, 안드로이드 운영체제는 이를 어떻게 관리하나요?  

### 🎯 개요
안드로이드에서 프로레스는 애플리케이션이 실행되는 한경입니다.  
다른 앱과 격리된 자체 프로세스에서 단일 실행 스레드로 작동하여  
시스템 보안, 메모리 관리 및 내결함성을 보장합니다.  
리눅스 커널을 사용하여 운영체제에 의해 관리되며 엄격한 생명주기 규칙을 따릅니다.

#### ⚙️ 안드로이드에서 프로세스 동작 방식  
안드로이드 애플리케이션이 시작되면 운영 체제는 리눅스 fork() 시스템 함수를  
호출하여 해당 앱을 위한 새 프로세스를 생성합니다.  

각 프로세스는 Dalvik 또는 ART 가상 머신의 고유 인스턴스에서 실행되어  
안전하고 독립적인 실행을 보장합니다.  
각 프로세스는 고유한 리눅스 사용자 ID(UID)를 할당하여  
권한 제어 및 파일 시스템 격리를 포함한 엄격한 보안 경계를 적용합니다.  

#### ⚙️ 애플리케이션 컴포넌트와 프로세스 연결  
애플리케이션의 모든 컴포넌트는 동일한 프로세스 내에서 실행되며  
대부분의 애플리케이션은 이 표준을 따릅니다.  

하지만 개발자는 AndroidManifest.xml 파일의 android:process 속성을 사용하여  
프로세스 할당을 커스텀 할 수 있습니다.  
4대 컴포넌트에 적용될 수 있습니다.  
하나의 앱에서 생성된 별도의 프로세스는 pid(Process ID)는 다르지만  
동일한 UID(user ID)를 가지기 때문에 권한 문제 없이 앱의 파일과 리소스에 접근할 수 있습니다.

분리가 필요한 경우는  
액티비티 자체가 메모리를 많이 사용하는 경우 입니다.  

다른 애플리케이션의 컴포넌트가 동일한 리눅스 사용자 ID를 가지고 동일한 인증서로 서명된 경우  
동일한 프로세스를 공유할 수 있습니다.  

안드로이드는 시스템 리소스 요구에 따라 프로세스를 동적으로 관리하며  
필요할 때 우선순위가 낮은 프로세스를 종료합니다.  
활성중인 액티비티 > 보이지 않는 액티비티  

#### ⚙️ 프로세스와 앱 생명주기  
안드로이드는 시스템 메모리와 앱의 현재 상태를 기반으로 프로세스 및 앱 생명주기를 관리하며  
엄격한 우선순위 계층을 따릅니다

https://developer.android.com/guide/components/activities/process-lifecycle?hl=ko

1. 포그라운드 프로세스  
  - 사용자와 활발히 상호 작용하며 실행 중인 프로세스.  
    가장 높은 우선순위이며 거의 종료되지 않습니다.
2. 보이는 프로세스  
  - 사용자에게 보이지만 활발하게 상호 작용하지 않는 프로세스  
  - Dialog 뒤의 Activity
3. 서비스 프로세스  
  - 데이터 동기화나 음악 재생과 같은 작업을 수행하는 백그라운드 서비스를 실행하는 프로세스 
4. 캐시된 프로세스
  - 더 빠른 재실행을 위해 메모리에 유지되는 유후 프로세스 
  - 메모리가 부족할 때 가장 먼저 종료됩니다.  

안드로이드 시스템은 메모리를 확보하고 시스템 안정성을 유지하기 위해  
우선순위가 낮은 프로세스를 자동으로 종료합니다.  

#### ⚙️ 보안 및 권한  
각 안드로이드 프로세스는 리눅스 보안 모델을 사용하여  
샌드박스 처리되어 엄격한 권한 기반 접근 제어를 시행합니다.  
명시적으로 권한이 부여되지 않는 한 애플리케이션이 다른 프로세스의  
데이터에 접근할 수 없도록 보장합니다.  

#### 🧠 요약
안드로이드의 프로세스는 애플리케이션 컴포넌트를 위한 실행 환경 역할을 하여  
격리되고 안전하며 효율적인 앱 작동을 보장합니다.  

#### ❓ 서로 다른 안드로이드 컴포넌트들을 별도의 프로세스에서 실행해야 하는 애플리케이션을 개발 중이라고 해봅시다. AndroidManifest에서 이를 어떻게 구성하며, 여러 프로세스를 사용할 때의 잠재적인 단점은 무엇인가요?

각 컴포넌트마다 android:process 설정으로 구성할 수 있습니다.  
잠재적인 단점은 메모리 오버헤드 증가, 개발 복잡성 증가의 단점이 존재할 수 있습니다.

#### ❓ 시스템이 프로세스 우선 순위를 어떻게 정하는지, 중요한 프로세스가 종료되는 것을 방지하기 위해서 개발자가 따라야 할 전략은 무엇인가요?

포그라운드 프로세스, 가려진 프로세스, 서비스 프로세스, 캐시된 프로세스로 우선순위가 나뉩니다.  
프로세스는 메모리가 부족할때 우선순위대로 종료되므로  
너무 많은 메모리를 사용하지 않게 방지해야 합니다.  
onTerminate와 같은 함수에서 메모리를 정리해주거나  
설령 프로세스가 종료되더라도 중요 정보는 DB나 서버에 저장해야 합니다.  

#### 💡 Pro Tips for Mastery : 4대 컴포넌트가 붙여진 이유

Activity, Service, BroadcastReceiver, ContentProvider가 4대 주요 컴포넌트라고 불리는데는 프로세스와 밀접한 관련이 있기 때문입니다.

#### ⚙️ 각 컴포넌트가 안드로이드 프로세스와 어떤 관련이 있는가
1. Activities  
  - 사용자의 상호 작용의 진입점이며 안드로이드 프로세스 생명주기와 밀접하게 관련이 있습니다.
  - 사용자가 앱을 열면 시스템은 앱의 프로세스에서 Activity를 시작합니다.
  - 프로세스가 종료되면 Activity가 소멸됩니다.
2. Services  
  - Service는 사용자 인터페이스 없이 백그라운드 작업을 수행합니다.
  - 애플리케이션이 보이지 않을 때도 실행될 수 있습니다.
  - android:process 속성에 따라 앱과 동일한 프로세스 혹은 별도의 프로세스에서 실행될 수 있습니다.
3. Broadcast Receivers
  - 앱이 실행중이 아니더라도 트리거되어 필요한 경우 안드로이드 시스템이 해당 프로세스를 시작하도록 합니다.
4. Content Providers
  - 공유 애플리케이션 데이터를 관리하여 데이터베이스에 읽거나 쓸 수 있도록 합니다.
  - 프로세스 간 통신을 허용하므로 다른 애플리케이션 간에 데이터를 공유하는 데 사용될 수 있으며 프로세스를 안전하고 효율적으로 관리하도록 요구합니다.

#### ⚙️ 안드로이드 프로세스와의 연결
컴포넌트가 트리거되면 안드로이드 시스템은 아직 실행중이 아닌  
앱의 프로세스를 새롭게 시작할 수 있습니다.  
각 컴포넌트는 매니페스트 파일의 android:process 속성을 사용하여  
자체 프로세스를 할당받을 수도 있어 리소스 집약적인 작업에 더 많은 유연성을 제공합니다.

각 컴포넌트는 자체 전용 프로세스를 가질 수 있습니다.  
다른 안드로이드 컴포넌트에 비해 더 강력하고 독립적으로 작동 가능합니다.  
이 설계는 백그라운드 실행, IPC 및 시스템 수준 상호작용을 가능하게 하여  
안드로이드 앱이 복잡한 다중 프로세스 작업을 효율적으로 처리할 수 있도록 보장합니다.  

#### 🧠 요약
필수적인 애플리케이션 기능, 사용자와 상호작용, 앱 간 통신을 가능하게 하기 때문에 4대 컴포넌트로 불립니다.  
안드로이드 프로세스 모델과의 긴밀한 관계를 통해 효율적인 프로세스 관리  
최적의 리소스 활용 및 시스템 수준 작업 조정을 보장하여 안드로이드 앱 개발의 근간을 형성합니다.  

## 안드로이드 UI - Views
UI 시스템은 사용자가 앱에 대한 첫 인상을 결정하게 되고 의미 있는 사용자 상호작용을 촉진하므로 중요한 역할을 합니다.  
요즘은 Jetpack Compose를 많이 사용하지만  
현업에서 View 시스템을 사용하는경우도 많으니 Views에 대해 공부해보도록 합시다  

## 📚 Q33. View 생명주기를 설명해주세요  

### 🎯 개요
View 생명주기는 View가 생성되고 Activity나 Fragment에 연력되고, 화면에 표시되고  
최종적으로 소멸되거나 분리되는 동안 거치는 생명주기 이벤트를 나타냅니다.

![view-lifecycle](/hegunhee/images/view_lifecycle.png) 

1. View 생성 
  View가 하드 코딩 방식으로 인스턴스화 되거나 XML 레이아웃에서 인플레이션되는 단계입니다.  
  리스너 설정 및 데이터 바인딩과 같은 초기 설정 작업이 여기서 수행됩니다.  
  onAttachedToWindow() 메서드는 View가 부모 뷰에 추가되고  
  화면 렌더링을 할 준비를 마쳤을 때 트리거 됩니다. 
2. Layout 단계(onMeasure, onLayout)  
  View의 크기와 위치를 측정합니다.  
  onMeasure() 메서드는 레이아웃 매개변수와 부모 제약 조건에 따라 View의 너비와 높이를 결정합니다.  
  측정된 후 onLayout() 메서드는 View를 부모 내에 배치하여 화면에 표시될 위치를 최종 결정합니다.  
3. Drawing 단계 (onDraw)  
  크기와 위치가 최종 결정된 후 onDraw() 메서드는 텍스트나 이미지와 같은  
  View의 내용을 Canvas에 렌더링 합니다.  
  커스텀 뷰는 해당 메서드를 재정의하여 커스텀 드로잉 로직을 구현할 수 있습니다.  
4. Event 처리 (onTouchEvent, onClick)  
  상호 작용하는 View는 이 단계에서 터치 이벤트, 제스처와 같은 사용자 상호 작용을 처리합니다.  
5. View 분리(onDetachedFromWindow)  
  View가 화면과 부모 ViewGroup에서 제거될 때(Activity 또는 Fragment의 소멸)  
  onDetachedFromWindow() 메서드가 호출됨  
  이 단계는 리소스를 정리하거나 리스너를 분리하는 데 이상적  
6. View 소멸  
  View가 더 이상 사용되지 않으면 가비지 컬렉션 처리됨
  리스너나 백그라운드 작업과 같은 모든 리소스가 적절하게 해소되었는지 확인을 통해 메모리 누수를 방지  

#### 🧠 요약
View의 샘명주기는 생성, 측정, 레이아웃, 드로잉, 이벤트 처리 및 최종 분리를 포함하며  
안드로이드 애플리케이션 내에서 표시되고 사용되는 동안 거치는 단계를 반영합니다.  


#### ❓ 이미지 로딩이나 애니메이션 설정과 같이 비용이 많이 드는 기능을 포함하는 커스텀 View는 뷰의 생명주기 어느 시점에 이러한 리소스 및 기능을 초기화해야 하며, 어떻게 메모리 누수를 방지해야 하나요?

onAttachedToWindow에서 리소스 초기화하는것이 적절해 보입니다.  
View가 Window에 연결될 때 호출되는 시점으로  
비용이 많이 드는 리소스들을 초기화하기에 적합합니다.

onDestroyFromWindow 에서 리소스 정리하는것이 적절해보입니다.  
View가 Window에서 분리되는 시점으로 이때 리소스를 정리해야 합니다.  

onWindowVisibilityChanged()으로 가시성 관리를 합니다.  
Window의 가시성 변화에 따라 애니메이션을 일시정지/재개합니다.

#### ❓ 성능 문제가 발생하는 동적으로 생성된 View를 포함하는 복잡한 레이아웃이 있을 때 적절한 응답성을 유지하면서 렌더링 효율을 향상시키기 위해 onMeasure() 및 onLayout() 메서드를 어떻게 최적화할 수 있을까요?

##### onMeasure() 최적화 전략

측정 결과 캐싱  
동일한 MeasureSpec에 대한 측정 결과를 캐싱하여 중복 계산을 방지합니다.

```kotlin 
class OptimizedViewGroup @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : ViewGroup(context, attrs, defStyleAttr) {

    // 측정 결과 캐시
    private val measureCache = mutableMapOf<Long, Pair<Int, Int>>()
    private var lastMeasureSpecWidth = 0
    private var lastMeasureSpecHeight = 0

    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        val specKey = (widthMeasureSpec.toLong() shl 32) or heightMeasureSpec.toLong()
        
        // 캐시에서 확인
        measureCache[specKey]?.let { cachedResult ->
            setMeasuredDimension(cachedResult.first, cachedResult.second)
            return
        }

        // MeasureSpec이 변경된 경우만 자식 View 측정
        val specChanged = lastMeasureSpecWidth != widthMeasureSpec || 
                         lastMeasureSpecHeight != heightMeasureSpec

        if (specChanged) {
            measureChildren(widthMeasureSpec, heightMeasureSpec)
            lastMeasureSpecWidth = widthMeasureSpec
            lastMeasureSpecHeight = heightMeasureSpec
        }

        val measuredWidth = calculateOptimalWidth(widthMeasureSpec)
        val measuredHeight = calculateOptimalHeight(heightMeasureSpec)

        // 결과 캐싱
        measureCache[specKey] = Pair(measuredWidth, measuredHeight)
        
        // 캐시 크기 제한
        if (measureCache.size > MAX_CACHE_SIZE) {
            measureCache.clear()
        }

        setMeasuredDimension(measuredWidth, measuredHeight)
    }

    companion object {
        private const val MAX_CACHE_SIZE = 50
    }
}
```

##### 조건부 자식 View 측정
변경이 필요한 자식 View만 선택적으로 측정합니다.

```kotlin
override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
    var totalWidth = 0
    var totalHeight = 0
    
    for (i in 0 until childCount) {
        val child = getChildAt(i)
        
        // 가시성 확인으로 불필요한 측정 방지
        if (child.visibility == View.GONE) {
            continue
        }
        
        val layoutParams = child.layoutParams as CustomLayoutParams
        
        // 크기가 변경된 자식만 재측정
        if (layoutParams.needsRemeasure || isFirstMeasure) {
            measureChild(child, widthMeasureSpec, heightMeasureSpec)
            layoutParams.needsRemeasure = false
        }
        
        totalWidth += child.measuredWidth + layoutParams.leftMargin + layoutParams.rightMargin
        totalHeight = maxOf(totalHeight, child.measuredHeight + layoutParams.topMargin + layoutParams.bottomMargin)
    }
    
    setMeasuredDimension(
        resolveSize(totalWidth, widthMeasureSpec),
        resolveSize(totalHeight, heightMeasureSpec)
    )
}
```

##### onLayout() 최적화 전략

##### 배치 정보 캐싱 및 증분 업데이트
이전 배치 정보를 캐싱하고 변경된 부분만 업데이트합니다.  

```kotlin
class OptimizedViewGroup : ViewGroup {
    
    // 배치 정보 캐싱
    private val childBounds = mutableMapOf<View, Rect>()
    private var lastLayoutWidth = 0
    private var lastLayoutHeight = 0

    override fun onLayout(changed: Boolean, l: Int, t: Int, r: Int, b: Int) {
        val currentWidth = r - l
        val currentHeight = b - t
        
        // 크기가 변경되지 않은 경우 기존 배치 유지
        if (!changed && 
            currentWidth == lastLayoutWidth && 
            currentHeight == lastLayoutHeight &&
            !hasChildLayoutChanged()) {
            return
        }

        layoutChildrenOptimized(currentWidth, currentHeight)
        
        lastLayoutWidth = currentWidth
        lastLayoutHeight = currentHeight
    }

    private fun layoutChildrenOptimized(parentWidth: Int, parentHeight: Int) {
        var currentX = paddingLeft
        var currentY = paddingTop
        
        for (i in 0 until childCount) {
            val child = getChildAt(i)
            
            if (child.visibility == View.GONE) {
                continue
            }
            
            val layoutParams = child.layoutParams as CustomLayoutParams
            val childWidth = child.measuredWidth
            val childHeight = child.measuredHeight
            
            // 새로운 배치 위치 계산
            val newBounds = Rect(
                currentX + layoutParams.leftMargin,
                currentY + layoutParams.topMargin,
                currentX + layoutParams.leftMargin + childWidth,
                currentY + layoutParams.topMargin + childHeight
            )
            
            // 이전 배치와 비교하여 변경된 경우만 레이아웃
            val oldBounds = childBounds[child]
            if (oldBounds == null || oldBounds != newBounds) {
                child.layout(newBounds.left, newBounds.top, newBounds.right, newBounds.bottom)
                childBounds[child] = newBounds
            }
            
            currentX += childWidth + layoutParams.leftMargin + layoutParams.rightMargin
            
            // 줄 바꿈 처리
            if (currentX + getNextChildWidth(i + 1) > parentWidth - paddingRight) {
                currentX = paddingLeft
                currentY += childHeight + layoutParams.topMargin + layoutParams.bottomMargin
            }
        }
    }
}
```


#### 💡 Pro Tips for Mastery : findViewTreeLifecycleOwner() 함수는 어떤 역할을 하나요? 

findViewTreeLifecycleOwner() 함수는 View 클래스의 확장 함수  
LifecycleOwner를 구현하는 커스텀 컴포넌트와 같은 호스팅 컴포넌트 생명주기의 범위를 나타냅니다.  
View 트리에 연결된 가장 가까운 LifecycleOwner를 찾아 반환  

#### findViewTreeLifecycleOwner()를 사용하는 이유  
이 함수는 LiveData, ViewModel 또는 LifecycleOserver와 같은 생명주기 인식 요소와 상호 작용해야 하는 커스텀 View나 서드파티 컴포넌트로 작업할 때 특히 유용합니다.  

- 생명주기를 인식하는 컴포넌트가 올바른 생명주기에 제대로 바인딩됩니다.
- 생명주기가 끝나면 관찰자가 정리되도록 하여 메모리 누수를 방지합니다.

```kotlin
class CustomLifecycleAwareView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null
) : LinearLayout(context, attrs) {

    private var lifecycleObserver: LifecycleObserver? = null

    fun bindObserver(observer: LifecycleObserver) {
        lifecycleObserver?.let { removeObserver(it) }

        val lifecycleOwner = findViewTreeLifecycleOwner()

        lifecycleOwner?.lifecycle?.addObserver(observer) ?: run {
            Log.e("CustomView", "No LifecycleOwner found for the View")
        }
        lifecycleObserver = observer
    }

    override fun onDetachedFromWindow() {
        super.onDetachedFromWindow()
        lifecycleObserver?.let { removeObserver(it) }
    }

    private fun removeObserver(observer: LifecycleObserver) {
        findViewTreeLifecycleOwner()?.lifecycle?.removeObserver(observer)
        lifecycleObserver = null
    }
}
```

커스텀 뷰, 서드파티 라이브러리, 로직 결합도 분리에 사용됩니다.

#### 한계  
해당 함수는 유용한 기능을 제공하지만 View 트리에 LifecycleOwner가 존재해야 합니다.  
LifecycleOwner를 제공하는 소유자가 없으면 null을 반환하므로 크래시나 예상치 못한 동작이 발생할 수 있기 때문에 신중하게 사용해야 합니다.  

#### 요약
해당 함수는 View트리에서 가장 가까운 LifecycleOwner를 획득하는 데 유용한 기능을 제공합니다

## 📚 Q34. View와 ViewGroup의 차이점은 무엇인가요?

### 🎯 개요
View와 ViewGroup은 UI 컴포넌트를 구현하기 위한 기본적 개념입니다.  
둘 다 android.view 패키지의 일부이지만 UI 계층 구조에서 다른 목적을 가집니다.  

#### ⚙️ View란?
View는 화면에 표시되는 직사각형 형태의 UI를 구성하는 최소 단위의 단일 컴포넌트 입니다.  
- Button, TextView, EditText

##### Additional Tips  
View는 34,000 이상의 코드로 구현되어 있습니다.  
이는 Viww 인스턴스를 생성하고 관리하는 것이 의외로 상당한 오버헤드가 수반됨을 나타냅니다.  
그러므로 성능을 최적화하려면 가능한 불필요한 View 인스턴스를 피해야 합니다.

#### ⚙️ ViewGroup이란?
ViewGroup은 여러 View 또는 다른 ViewGroup 요소를 포함하는 일종의 컨테이너입니다.  
이는 LinearLayout, RelativeLayout, ConstraintLayout, FrameLayout과 같은 레이아웃의 기본이되는 클래스입니다.  
ViewGroup은 자식 뷰의 레이아웃과 위치를 관리하며, 화면에서 자식 View들의 사이즈가 측정되고 그려지는 방식을 정의합니다.  

##### Additional Tips  
ViewGroup 클래스는 View를 확장하고 ViewParent 및 ViewManager 인터페이스를 모두 구현합니다.  
ViewGroup은 다른 View 객체를 위한 컨테이너 역할을 하므로 독립적인 View보다 본질적으로 더 복잡하고 리소스 집약적입니다.

#### ⚙️ View와 ViewGroup의 주요 차이점
1. 목적
  - View : 콘텐츠를 표시하거나 사용자와 상호 작용하도록 설계된 단일 UI
  - ViewGroup : 여러 자식 뷰를 구성하고 관리하기 위한 컨테이너
2. 계층
  - View : UI 계층 구조의 리프노드, 다른 뷰를 포함할 수 없음
  - ViewGroup : 여러 자식 뷰 또는 다른 ViewGroup 요소를 포함 가능한 브랜치 노드
3. 레이아웃 동작
  - View : 레이아웃 매개변수에 의해 정의된 자체적인 크기와 위치를 가짐
  - ViewGroup : 정의된 레이아웃 규칙을 사용하여 자식 뷰의 크기와 위치를 결정
4. 상호작용 핸들링
  - View : 터치 및 키 이벤트를 처리 가능함
  - ViwwGroup : onInterceptTouchEvent와 같은 메서드를 사용하여 자식의 이벤트를 가로채고 관리 가능
5. 성능
  - ViewGroup : 계층 구조로 인해 렌더링에 복잡성을 더함  
  과도하게 중첩될 경우 렌더링 시간 증가, UI 업데이트 지연과 같은 성능 문제가 발생 가능

#### 🧠 요약
View는 모든 UI 요소의 근간이 되며  
ViewGroup은 여러 View 객체를 구성하고 관리하기 위한 컨테이너

#### ❓ View 생명주기에서 requestLayout(), invalidate(), postInvalidate()가 어떻게 작동하는지 설명하고 각각 언제 사용해야 하나요?

requestLayout()은 크기 측정부터 다시 해야할 경우에 작동되며
measure부터 다시 시작합니다. 

invalidate()는 onDraw()만 다시 실행하기 위해서 작동합니다.  

Ui Thread가 아닌 경우 invalidate대신 postInvalidate()를 사용해야 합니다.  

#### ❓ View 생명주기는 Activity 생명주기와 어떻게 다르며 효율적인 UI 렌더링을 위해 둘 다 이 해하는 것이 왜 중요한가요?

View 생명주기는 개별 UI 컴포넌트 단위의 작은 범위이므로  
Activity 내에서 동적으로 View가 추가/제거될 때 개별적으로 발생합니다.  

Activity는 사용자의 앱 전환, 화면 회전 등에 의해 트리거되며  
View는 Activity 생명주기 내에서 더 세밀하게 발생합니다.

## 📚 Q35. ViewStub이란 무엇이고, 이를 사용하여 UI 성능을 최적화해 본 경험이 있나요?

### 🎯 개요
ViewStub은 명시적으로 필요할 때까지 레이아웃의 인플레이션을 지연시키는 데 사용되는 가볍고 보이지 않는 플레이스홀더 뷰 입니다.  
당장 필요하지 않은 뷰를 필요한 시기에 적절하게 인플레이션하여 오버헤드를 피함으로써 성능을 개선하는 데 사용됩니다.

### 🛠️ ViewStub의 주요 특징
1. 가벼움  
  - 인플레이션될 때까지 레이아웃 공간을 차지하거나 리소스를 소비하지 않으므로 메모리 공간이 최소화된 매우 가벼운 뷰입니다.
2. 인플레이션 지연  
  - 실제 레이아웃은 inflate() 메서드가 호출되거나 ViewStub이 보이게 될 때만 인플레이션 됩니다.
3. 일회성  
  - 한번 인플레이션되면 ViewStub은 뷰 계층 구조에서 인플레이션된 레이아웃으로 대체되며 재사용할 수 없습니다.

### ⚙️ ViewStub의 일반적인 사용 사례
1. 조건부 레이아웃
  - ViewStub은 오류 메시지, 진행률 표시 또는 선택적 UI 요소와 같이 조건부로 표시되는 레이아웃에 이상적입니다.
2. 초기 렌더링 시간 절감  
  - 복잡하거나 리소스 집약적인 뷰의 인플레이션을 지연시키므로 Activity 또는 Fragment의 초기 렌더링 시간을 개선하는 데 도움이 됩니다.
  - by lazy {}와 동일한 이익을 얻을 수 있지않을까 싶음
3. 동적 UI
  - 필요할 때만 화면에 동적으로 콘텐츠를 렌더링하는 데 사용될 수 있어 메모리 사용량을 최적화함

![view-stub](/hegunhee/images/viewstub.png)


### 🛠️ ViewStub 사용 방법
ViewStub은 인플레이션할 레이아웃 속성과 함께 XML 레이아웃에 정의할 수 있습니다.

```xml
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <!-- Regular Views -->
    <TextView
        android:id="@+id/title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Main Content" />

    <!-- Placeholder ViewStub -->
    <ViewStub
        android:id="@+id/viewStub"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout="@layout/optional_content" />
        
</LinearLayout>
```

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val viewStub = findViewById<ViewStub>(R.id.viewStub)

        // 필요할 때 레이아웃 인플레이션
        val inflatedView: View? = try {
            viewStub.inflate() // 성공 시 inflatedView 반환, 실패 시 null 또는 예외
        } catch (e: IllegalStateException) {
            // 이미 인플레이트된 경우 findViewById 사용
            findViewById(viewStub.inflatedId)
        }

        // 인플레이션된 레이아웃의 뷰에 접근 (null 체크 필요)
        inflatedView?.let {
            val optionalTextView = it.findViewById<TextView>(R.id.optionalText)
            optionalTextView.text = "Inflated Content"
        }
    }
}
```

### ⚙️ ViewStub의 장점
1. 최적화된 성능
  늦게 초기화해도 되는 뷰 생성을 지연시켜 메모리 사용량을 줄이고 초기 렌더링 성능을 개선
2. 쉬운 레이아웃 관리  
  뷰를 수동으로 추가하거나 제거하지 않고도 UI 요소를 선택적으로 렌더링함으로써 쉽게 관리할 수 있음
3. 쉬운 사용성
  API 사용이 간단하고 XML 통합이 쉬워 개발자가 쉽게 활용할 수 있음

### ⚙️ ViewStub의 한계
1. 일회성  
  일단 인플레이션되면 ViewStub은 뷰 계층 구조에서 제거되며 재사용할 수 없습니다.
2. 제한된 컨트롤
  플레이스홀더이므로 인플레이션될 대까지 사용자 상호 작용을 처리하거나 복잡한 작업을 수행할 수 없습니다.

#### 🧠 요약
필요할 때까지 레이아웃 인플레이션을 지연시켜  
성능을 최적화하는 유용한 UI 컴포넌트입니다.  
조건부 레이아웃이나 지금 당장 렌더링이 필요하지 않은 뷰에 특히 유용하며  
메모리 사용량을 줄이고 앱 응답성을 개선하는 데 도움이 됩니다.  

#### ❓ ViewStub이 인플레이션될 때 어떤 일이 일어나며, 레이아웃 성능 및 메모리 사용량 측면에서 뷰 계층 구조에 어떤 영향을 미치나요?

인플레이션이 실행되면 ViewStub은 인플레이트 되고  
infaltedId를 가진 layout 파일로 대체됩니다.  
lazy include 방식입니다.  

ViewStub은 뷰 계층구조에서 완전히 제거됩니다.  
앱 시작 시 불필요한 뷰 생성을 건너뛰며 UI 렌더링이 빨리지며  
실제 필요한 시점에만 뷰를 생성하여 사용자 경험을 개선합니다.

그렇기때문에 초기 메모리 사용량은 감소합니다.  

https://medium.com/@lunay0ung/android-viewstub-light-weight-view-30f4985dfab4

## 📚 Q36. 커스텀 뷰는 어떻게 구성하나요?

### 🎯 개요
커스텀 뷰 구현은 여러 화면에서 재사용해야 하는 특정 스펙과 동작을 가진  
UI 컴포넌트를 사용자 정의해야 할 때 필수적입니다.  
커스텀 뷰를 생성하면 복잡한 UI 로직을 캡슐화하고 재사용성을 높이며  
프로젝트 내 다른 레이어의 구조를 단순화할 수 있습니다.  

### 커스텀 뷰 생성 과정

1. 커스텀 View 클래스 생성하기
기본 뷰 클래스를 확장하는 새 클래스를 정의합니다.  
필요에 따라 onDraw(), onMeasure(), onLayout()과 같은 필요한 생성자 및 메서드를 재정의합니다.  
아래 예시는 onDraw() 메서드를 오버라이드하여 캔버스에 직접 빨간색 원을 그리는 커스텀 뷰 입니다.

```kotlin
class CustomCircleView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyle: Int = 0
): View(context, attrs, defStyle) {

    private val paint = Paint(Paint.ANTI_ALIAS_FLAG).apply { // Anti-aliasing 추가
        color = Color.RED
        style = Paint.Style.FILL
    }

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        // 중앙에 빨간색 원 그리기
        canvas.drawCircle(width / 2f, height / 2f, min(width, height) / 4f, paint)
    }
}
```

2. XML 레이아웃에서 커스텀 View 사용하기  
커스텀 뷰 클래스를 생성한 후 XML 파일에서 직접 참조할 수 있습니다.  
커스텀 클래스의 전체 패키지 이름을 정확하게 사용해야 합니다.

```xml
<com.example.myapp.ui.CustomCircleView
    android:id="@+id/customCircleView"
    android:layout_width="100dp"
    android:layout_height="100dp"
    android:layout_gravity="center" />
```

3. 커스텀 속성 추가하기 (선택 사항)  
뷰의 속성을 커스텀할 수 있습니다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="CustomCircleView">
        <attr name="circleColor" format="color"/>
        <attr name="circleRadius" format="dimension"/>
    </declare-styleable>
</resources>
```
커스텀 뷰 클래스에서 context.obtainStyledAttributes()를 사용하여 커스텀 속성 값을 가져올 수 있습니다.  

```kotlin
class CustomCircleView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0 // defStyleAttr로 이름 변경 권장
): View(context, attrs, defStyleAttr) {
    var circleColor: Int = Color.RED
    var circleRadius: Float = 50f

    private val paint = Paint(Paint.ANTI_ALIAS_FLAG).apply {
        style = Paint.Style.FILL
    }

    init {
        attrs?.let {
            getAttrs(it, defStyleAttr)
        }
        paint.color = circleColor // 페인트 색상 초기화
    }
    private fun getAttrs(attrs: AttributeSet, defStyleAttr: Int) {
        val typedArray = context.obtainStyledAttributes(
            attrs, R.styleable.CustomCircleView, defStyleAttr, 0 // 기본 스타일 리소스 0
        )
        try {
            setTypeArray(typedArray)
        } finally {
            typedArray.recycle()
        }
    }
    private fun setTypeArray(typedArray: TypedArray) {
        circleColor = typedArray.getColor(R.styleable.CustomCircleView_circleColor, Color.RED)
        circleRadius = typedArray.getDimension(R.styleable.CustomCircleView_circleRadius, 50f)
    }

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        // onDraw에서 사용자 정의 된 반지름 값 사용
        canvas.drawCircle(width / 2f, height / 2f, circleRadius, paint)
    }

    // 필요하다면 onMeasure 재정의
    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        val desiredWidth = (circleRadius * 2 + paddingLeft + paddingRight).toInt()
        val desiredHeight = (circleRadius * 2 + paddingTop + paddingBottom).toInt()
        val width = resolveSize(desiredWidth, widthMeasureSpec)
        val height = resolveSize(desiredHeight, heightMeasureSpec)
        setMeasuredDimension(width, height)
    }
    // circleColor 또는 circleRadius 변경 시 뷰를 다시 그리도록 하는 메서드
    fun setCircleProperties(color: Int, radius: Float) {
        this.circleColor = color
        this.circleRadius = radius
        paint.color = color
        requestLayout() // 뷰의 크기 변경 가능성이 있으므로 requestLayout 호출
        invalidate() // 다시 그리기 요청
    }
}
```

4. 레이아웃 측정 처리하기 (선택 사항)  
커스텀 뷰가 크기를 측정하는 방식을 수동적으로 처리하고 싶고,  
특히 표준적인 뷰와 다르게 동작해야 하는 경우 onMeasure() 메서드를 재정의하여 구현할 수 있습니다.

#### 🧠 요약
커스텀 뷰를 잘 구현하면 UI 디자인에 유연성을 부여할 수 있습니다.

#### ❓ XML 레이아웃에서 이전 버전과의 호환성을 보장하면서 커스텀 뷰에 커스텀 속성을 효율적으로 적용하려면 어떻게 해야 하나요?

이전 버전이라는것이 어떤것인지 모르겠으나  
android API 버전에 대해서는 분기처리를 해서 처리하거나  
View의 생성자를 모두 구현하는 방식을 택할것같습니다.  

#### 💡 Pro Tips for Mastery : 커스텀 뷰의 기본 생성자에서 @JvmOverloads를 사용할 때 왜 주의해야 하나요?

@JvmOverloads 어노테이션은 Kotlin 함수 또는 클래스에 대해  
여러 오버로드된 메서드 또는 생성자를 자동으로 생성하여 Kotlin과 Java 간의 상호 운용성을 단순화하는 기능입니다.  

코틀린 기본 생성자에 사용할 경우 자바코드에서 정의한 모든 생성자에 대응할 수 있게 됩니다  

하지만 자바 생성자에서 의도한 바와 다르게 작동할 수 있으므로 주의해야 합니다.  
특정 값을 입력하지 않은경우 자동으로 메서드를 불러오거나 함수를 불러와서 사용하는 경우가 누락되는 경우가 발생할 수 있습니다.


## 📚 Q37. Canvus란 무엇이며 어떻게 활용하나요?

### 🎯 개요
커스텀 드로잉을 위한 핵심 구성 요소입니다.  
화면이나 Bitmap과 같은 다른 드로잉이 가능한 표면에 직접 그래픽을 렌더링하기 위한 인터페이스를 제공합니다.  
커스텀 뷰, 애니메이션 및 시각 효과를 만드는 데 다방면으로 활용됩니다.  

### 🛠️ Canvus 작동 방식  
Canvus 클래스는 도형, 텍스트, 이미지 및 기타 콘텐츠를 그릴 수 있는  
2D 드로잉 표면을 나타냅니다.  
Paint 클래스와 긴밀하게 상호 작용합니다.  
커스텀 View의 onDraw() 메서드를 재정의하면  
Canvus 객체가 전달되어 무엇을 그릴지 정의할 수 있습니다.

```kotlin
class CustomView(context: Context) : View(context) {
    private val paint = Paint().apply {
        color = Color.BLUE
        style = Paint.Style.FILL
    }
    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        // 커스텀 그리기 로직
    }
}
```

위의 onDraw() 메서드는 Canvus 객체를 사용하여 커스텀 뷰의 중앙에 파란색 원을 그립니다.  

### ⚙️ Canvus의 일반적인 작업  
Canvus는 다음과 같은 다양한 드로잉 작업을 허용합니다.  
- 도형  
  drawCircle(), drawRect(), drawLine() 같은 메서드를 사용하여 도형을그릴 수 있습니다.  
- 텍스트  
  drawText() 메서드는 지정된 좌표와 모양으로 텍스트를 렌더링합니다.  
- 이미지  
  drawBitmap()을 사용하여 이미지를 렌더링합니다.  
- 커스텀 패스  
  path 객체와 drawPath()를 결합하여 복잡한 모양을 그릴 수 있습니다.

### ⚙️ 변환  
크기 조절, 회전, 이동과 같은 변환을 지원합니다.  

- 이동  
  canvus.translate(dx, dy)를 사용하여 캔버스 원점을 새 위치로 이동합니다.
- 크기 조절  
  canvus.scale(sx, sy)를 사용하여 드로잉 크기를 배율로 조절합니다
- 회전  
  canvus.rotate(degrees)를 사용하여 지정된 각도로 캔버스를 회전합니다.

### ⚙️ 사용 사례  
Canvus는 다음과 같이 고급 커스텀 그래픽이 필요한 시나리오에서 특히 유용합니다.

1. Custom Views
2. Games
3. Charts and Diagrams
4. Image Processing

#### 🧠 요약
Canvus는 화면에 커스텀 그래픽을 렌더링하는 유연하고 유용한 방법을 제공합니다  
도형, 텍스트, 이미지, 변환을 활용하여 개발자는 풍부한 시각적 및 커스텀 경험을 만들 수 있습니다

#### ❓ AndroidX 라이브러리에서 지원하지 않는 복잡한 모양이나 UI 렌더링 하는 커스텀 뷰를 어떻게 만들 수 있을까요?

기존의 도형, 변환을 통해서 구현 가능합니다.  
회전하는 원형의 경우 drawCircle + rotate()  
페이드 효과는 Paint의 alpha 조절

