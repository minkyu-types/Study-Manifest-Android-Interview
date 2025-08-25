## 📚 Q45. 안드로이드의 Bitmap이란 무엇이며, 큰 Bitmap을 효율적으로 처리하는 방법은 무엇인가요?

### 🎯 개요
Bitmap은 메모리 내에 픽셀 데이터를 보관하는 이미지 표현입니다. 리소스/파일/네트워크 등에서 가져온 이미지를 화면에 렌더링할 때 사용하며, 고해상도 이미지의 경우 메모리 사용량이 매우 커 OutOfMemoryError가 쉽게 발생할 수 있습니다.

#### ⚠️ 큰 Bitmap의 문제
- **과도한 메모리 소비**: 필요 이상 해상도의 이미지를 전부 로드하면 byteCount가 커짐
- **성능 저하**: 디코딩·GC 부담 증가로 프레임 드랍
- **크래시 위험**: 메모리 압박으로 OOM 발생 가능

### ⚙️ 메모리를 할당하지 않고 Bitmap 크기 읽기 (inJustDecodeBounds)
이미지를 전부 로드하기 전에 메타데이터(가로·세로·MIME)만 확인해 필요성/목표 크기를 판단합니다.

```kotlin
val options = BitmapFactory.Options().apply {
    inJustDecodeBounds = true
}
BitmapFactory.decodeResource(resources, R.drawable.my_image, options)

val imageWidth = options.outWidth
val imageHeight = options.outHeight
val imageType = options.outMimeType
```

### ⚙️ inSampleSize로 축소 디코딩
목표 크기(reqWidth, reqHeight)에 맞춰 2의 배수로 서브샘플링하여 메모리를 절감합니다.

```kotlin
fun calculateInSampleSize(
    options: BitmapFactory.Options,
    reqWidth: Int,
    reqHeight: Int
): Int {
    val (height, width) = options.run { outHeight to outWidth }
    var inSampleSize = 1

    if (height > reqHeight || width > reqWidth) {
        var halfHeight = height / 2
        var halfWidth = width / 2
        while ((halfHeight / inSampleSize) >= reqHeight &&
               (halfWidth / inSampleSize) >= reqWidth) {
            inSampleSize *= 2
        }
    }
    return inSampleSize
}
```

#### 서브샘플링을 사용한 전체 디코딩  
calculateInSampleSize를 사용하여 두 단계로 비트맵을 디코딩할 수 있습니다.  
1. 경계만 디코딩합니다.  
2. 계산된 inSampleSize를 설정하고 축소된 비트맵을 디코딩합니다.  

```kotlin
fun decodeSampledBitmapFromResource(
    res: Resources,
    resId: Int,
    reqWidth: Int,
    reqHeight: Int
): Bitmap? {
    return BitmapFactory.Options().run {
    inJustDecodeBounds = true
    BitmapFactory.decodeResource(res, resId, this)

    // inSampleSize 계산
    inSampleSize = calculateInSampleSize(this, reqWidth, reqHeight)
    // inSampleSize를 설정하고 비트맵 디코딩
    inJustDecodeBounds = false
    BitmapFactory.decodeResource(res, resId, this)
}
```

사용 예시
```kotlin
val bitmap = decodeSampledBitmapFromResource(
    resources,
    R.drawable.my_image,
    reqWidth = 100,
    reqHeight = 100
)
imageView.setImageBitmap(bitmap)
```

### ❓ 실전 질문
#### Q) 큰 Bitmap을 메모리에 로드하는 위험성과 효율적 처리 방법은?
- **위험성**: 대량 메모리 점유 → GC 부하/프레임 드랍 → OOM 위험
- **처리**: `inJustDecodeBounds`로 크기 확인 → `inSampleSize`로 축소 디코딩 → 메모리 LruCache + 디스크 캐시 단계화 → 백그라운드 처리 및 플레이스홀더 적용

#### 💡 Pro Tips for Mastery : 커스텀 이미지 로딩 시스템에서 큰 비트맵 캐싱을 어떻게 구현하나요?

### 🗂 메모리 캐싱: LruCache 활용
최근 사용 항목을 우선 보존하고 오래된 항목부터 제거하여 중복 디코딩을 줄입니다. 일반적으로 앱 가용 메모리의 1/8 수준을 캐시로 할당합니다.

```kotlin
object LruCacheManager {
    private val maxMemoryKb = (Runtime.getRuntime().maxMemory() / 1024).toInt()
    private val cacheSizeKb = maxMemoryKb / 8

    val memoryCache = object : LruCache<String, Bitmap>(cacheSizeKb) {
        override fun sizeOf(key: String, value: Bitmap): Int = value.byteCount / 1024
    }
}

fun loadBitmap(@DrawableRes resId: Int, imageView: ImageView) {
    val key = resId.toString()
    LruCacheManager.memoryCache.get(key)?.let {
        imageView.setImageBitmap(it)
        return
    }

    imageView.setImageResource(R.drawable.image_placeholder)
    val decoded = decodeSampledBitmapFromResource(
        imageView.resources, resId, reqWidth = 100, reqHeight = 100
    )
    if (decoded != null) {
        LruCacheManager.memoryCache.put(key, decoded)
        imageView.setImageBitmap(decoded)
    }
}
```

### 💾 디스크 캐싱: DiskLruCache 병행
앱 재시작 후에도 재사용이 필요하면 디스크 캐시를 추가합니다. 메모리 캐시 조회 → 디스크 캐시 조회 → 네트워크/디코딩 순으로 단계화합니다. 구현에는 [DiskLruCache 저장소](https://github.com/JakeWharton/DiskLruCache)를 참고하세요.

### 💡 추가 팁
- **백그라운드 디코딩**: WorkManager/Coroutine으로 메인 스레드 차단 방지
- **플레이스홀더/오류 이미지**: 사용자 경험 개선
- **글라이드/코일/피카소**: 검증된 라이브러리 사용 시 메모리·디스크 캐시와 썸네일·프리로드 등 고급 기능을 쉽게 활용

### 🧠 요약
- Bitmap은 픽셀 단위 데이터로 메모리 사용량이 크므로 목적 크기에 맞춘 **샘플링 디코딩**이 핵심
- `inJustDecodeBounds`로 메타데이터만 선확인 → `inSampleSize`로 축소 디코딩 → 필요 시 `inPreferredConfig`로 추가 절감
- **LruCache(메모리)** + **DiskLruCache(디스크)**로 중복 디코딩/재다운로드 최소화
- 디코딩·I/O는 **백그라운드**에서 처리해 UI를 부드럽게 유지

## 📚 Q46. 애니메이션을 어떻게 구현하나요?

### 🎯 개요
애니메이션은 전환을 부드럽게 하고, 변화에 주목을 끌며, 시각적 피드백을 제공해 UX를 향상합니다. 안드로이드는 속성 기반 애니메이션부터 레이아웃·상태 전환까지 다양한 API를 제공합니다.

### 🧩 주요 애니메이션 방법

#### 1) View Property Animations
간단한 **View 속성**(alpha, translationX/Y, rotation, scaleX/Y 등)을 손쉽게 애니메이션화합니다.

```kotlin
val view: View = findViewById(R.id.my_view)
view.animate()
    .alpha(0.5f)
    .translationX(100f)
    .setDuration(500)
    .setInterpolator(AccelerateDecelerateInterpolator())
    .start()
```

#### 2) ObjectAnimator
setter가 존재하는 **임의 객체의 속성**까지 애니메이션화할 수 있어 유연합니다.

```kotlin
val animator = ObjectAnimator.ofFloat(view, "translationY", 0f, 300f)
animator.duration = 500
animator.interpolator = OvershootInterpolator()
animator.start()
```

#### 3) AnimatorSet
여러 애니메이션을 **순차/동시**로 조합해 복잡한 시나리오를 구성합니다.

```kotlin
val fade = ObjectAnimator.ofFloat(view, "alpha", 1f, 0f)
val move = ObjectAnimator.ofFloat(view, "translationX", 0f, 200f)

val set = AnimatorSet()
set.playSequentially(fade, move) // 또는 set.playTogether(fade, move)
set.duration = 1000
set.start()
```

#### 4) ValueAnimator
임의의 값 보간으로 **커스텀 애니메이션**을 구현합니다.

```kotlin
val valueAnimator = ValueAnimator.ofInt(0, 100)
valueAnimator.duration = 500
valueAnimator.addUpdateListener { anim ->
    val v = anim.animatedValue as Int
    val params = binding.progressbar.layoutParams
    params.width = ((screenSize / 100f) * v).toInt()
    binding.progressbar.layoutParams = params
}
valueAnimator.start()
```

### 📄 XML 기반 View 애니메이션
리소스에서 정의해 단순하고 재사용이 쉽습니다.

res/anim/slide_in.xml
```xml
<translate xmlns:android="http://schemas.android.com/apk/res/android"
    android:fromXDelta="-100%"
    android:toXDelta="0%"
    android:duration="500"
    android:interpolator="@android:anim/accelerate_decelerate_interpolator"/>
```

사용법
```kotlin
val animation = AnimationUtils.loadAnimation(this, R.anim.slide_in)
view.startAnimation(animation)
```

### 🎛 MotionLayout (상태·레이아웃 전환)
ConstraintLayout 기반으로 **상태 간 전환**을 정밀하게 정의합니다.

res/xml/motion_scene.xml
```xml
<MotionScene xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <ConstraintSet android:id="@+id/start">
        <Constraint android:id="@id/box">
            <Layout
                android:layout_width="100dp"
                android:layout_height="100dp"
                app:layout_constraintTop_toTopOf="parent"
                app:layout_constraintStart_toStartOf="parent"/>
        </Constraint>
    </ConstraintSet>

    <ConstraintSet android:id="@+id/end">
        <Constraint android:id="@id/box">
            <Layout
                android:layout_width="100dp"
                android:layout_height="100dp"
                app:layout_constraintBottom_toBottomOf="parent"
                app:layout_constraintEnd_toEndOf="parent"/>
        </Constraint>
    </ConstraintSet>

    <Transition
        app:constraintSetStart="@id/start"
        app:constraintSetEnd="@id/end"
        app:duration="500">
        <OnSwipe
            app:touchAnchorId="@id/box"
            app:touchAnchorSide="top"
            app:dragDirection="dragDown"/>
    </Transition>
</MotionScene>
```

레이아웃 사용
```xml
<androidx.constraintlayout.motion.widget.MotionLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layoutDescription="@xml/motion_scene">

    <View
        android:id="@+id/box"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:background="@color/blue"/>

</androidx.constraintlayout.motion.widget.MotionLayout>
```

### 🎬 Drawable 애니메이션 (프레임 애니메이션)
`AnimationDrawable`로 프레임 전환을 구현합니다.

res/drawable/animation_list.xml
```xml
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:oneshot="false">
    <item android:drawable="@drawable/frame1" android:duration="100"/>
    <item android:drawable="@drawable/frame2" android:duration="100"/>
</animation-list>
```

사용법
```kotlin
val imageView: ImageView = findViewById(R.id.animated_image)
imageView.setBackgroundResource(R.drawable.animation_list)
val anim = imageView.background as AnimationDrawable
anim.start()
```

### 🧪 물리 기반 애니메이션 (DynamicAnimation)
자연스러운 모션을 위한 스프링/플링 기반 API.

```kotlin
val spring = SpringAnimation(view, DynamicAnimation.TRANSLATION_Y, 0f)
spring.spring = SpringForce()
    .setFinalPosition(0f)
    .setStiffness(SpringForce.STIFFNESS_LOW)
    .setDampingRatio(SpringForce.DAMPING_RATIO_HIGH_BOUNCY)
spring.start()
```

#### 💡 Pro Tips for Mastery : 인터폴레이터는 애니메이션과 어떻게 작동하나요?
- **Linear**: 등속
- **Accelerate / Decelerate / AccelerateDecelerate**: 가속·감속 곡선
- **Bounce / Overshoot**: 튕김, 오버슈트 효과

적용 예
```kotlin
val a = ObjectAnimator.ofFloat(view, "translationY", 0f, 500f)
a.duration = 1000
a.interpolator = OvershootInterpolator()
a.start()
```

커스텀 인터폴레이터
```kotlin
class CustomInterpolator : Interpolator {
    override fun getInterpolation(input: Float): Float = input * input
}

animator.interpolator = CustomInterpolator()
```

### 🧠 요약
크기 조절이나 이동과 같은 간단한 변화에는 View Property Animations, ObjectAnimator  
더 복잡한 시나리오의 경우 AnimatorSet  
임의의 값을 애니메이션화하는것은 ValueAnimator

### ❓ 실전 질문
#### Q) 클릭 시 확장/축소되는 버튼에 부드러운 애니메이션을 추가하려면?
- `ValueAnimator`로 너비/높이 값을 보간하거나, `ObjectAnimator`로 `scaleX/scaleY`를 애니메이션. `OvershootInterpolator`로 피드백 강화.

#### Q) 전통적 View 애니메이션 대신 MotionLayout을 사용할 때와 장점은?
- **상태 기반 전환/복합 제약 변경**이 필요한 경우 활용. 타임라인·키프레임·제약 변경을 XML로 선언적으로 관리하며, 제스처 연동(OnSwipe/OnClick)과 미세 제어가 용이.

## 📚 Q47. Window란 무엇인가요?

### 🎯 개요
`Window`는 Activity/Dialog/Toast 등 화면에 표시되는 **모든 View 계층의 최상위 컨테이너**입니다. 앱 UI와 디스플레이 사이의 다리로서, 레이아웃 파라미터, 입력 분배, 전환/애니메이션, 시스템 UI 제어를 담당합니다. 각 Activity는 기본적으로 하나의 앱 Window를 가집니다.

### 🧩 주요 특징
- **DecorView**: Window의 루트 뷰. 상태바/내비게이션 바 공간과 앱 콘텐츠 영역을 포함
- **레이아웃 파라미터**: 크기, 위치, 가시성 등 표시 방식 제어
- **입력 처리**: 터치·키 이벤트를 올바른 View로 라우팅
- **애니메이션/전환**: 열기/닫기/전환 애니메이션 적용 가능
- **시스템 UI 제어**: 상태바/내비게이션 바 표시/숨김, 플래그 설정

### ⚙️ Activity Window 커스텀 예시
```kotlin
// 전체 화면 모드
window.decorView.systemUiVisibility = View.SYSTEM_UI_FLAG_FULLSCREEN
// 배경 색상 변경
window.setBackgroundDrawable(ColorDrawable(Color.BLACK))
```

### 🧱 Window 관리: WindowManager
`WindowManager`는 윈도우의 추가/제거/업데이트를 담당하는 시스템 서비스입니다. 앱 창, 시스템 대화상자, 오버레이 등 다양한 창이 **레이어(z-order)**와 포커스, 입력, 애니메이션 규칙 하에 공존하도록 관리합니다.

#### 오버레이 예시 (권한 필요)
```kotlin
val wm = context.getSystemService(Context.WINDOW_SERVICE) as WindowManager
val floating = LayoutInflater.from(context).inflate(R.layout.floating_view, null)

val type = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O)
    WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY
else WindowManager.LayoutParams.TYPE_PHONE

val params = WindowManager.LayoutParams(
    WindowManager.LayoutParams.WRAP_CONTENT,
    WindowManager.LayoutParams.WRAP_CONTENT,
    type,
    WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE,
    PixelFormat.TRANSLUCENT
).apply {
    gravity = Gravity.TOP or Gravity.START
    x = 0; y = 0
}

wm.addView(floating, params)
// ...
// wm.removeView(floating)
```

#### 💡 Pro Tips for Mastery : PopupWindow
`PopupWindow`는 별도 클래스로, 내부적으로 `WindowManager`를 이용해 콘텐츠를 떠 있게 표시합니다. 임시 메뉴/툴팁 등 상황별 UI에 적합하며 포커스/외부터치/애니메이션을 유연하게 제어할 수 있습니다.

### 🧠 요약
- `Window`는 View 트리의 최상위 컨테이너이자 시스템과의 인터페이스
- `DecorView`를 통해 시스템 UI 공간 + 앱 콘텐츠를 포괄
- `WindowManager`가 창의 수명주기/레이어/입력/애니메이션을 관리
- 오버레이/팝업 등 고급 시나리오에선 권한·UX를 신중히 고려

### ❓ 실전 질문
#### Q) 단순한 레이아웃의 Activity가 표시될 때 Window는 몇 개인가요?
- 일반적으로 **앱 Window 1개**가 존재합니다. (시스템 상태바/내비게이션 바는 별도 시스템 Window로 취급될 수 있으나, 앱 관점에서는 1개 Window에서 `DecorView` 하위에 UI가 구성됩니다.)

## 📚 Q48. 웹 페이지를 어떻게 렌더링하나요?

### 🎯 개요
`WebView`는 앱 내에서 웹 콘텐츠를 렌더링하고 상호작용할 수 있게 하는 컴포넌트입니다. 최신 기능/호환성 확보를 위해 `AndroidX WebKit` 사용을 권장합니다.

### ⚙️ 초기화 및 로드
레이아웃에 추가하거나 코드로 생성합니다.

```xml
<!-- activity_main.xml -->
<WebView
    android:id="@+id/webView"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>
```

```kotlin
val webView: WebView = findViewById(R.id.webView)
webView.loadUrl("https://www.example.com")
```

권한
```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

### ⚙️ JavaScript 활성화
```kotlin
val settings = webView.settings
settings.javaScriptEnabled = true
```

### 🧭 내비게이션 가로채기 (외부 브라우저 열림 방지)
```kotlin
webView.webViewClient = object : WebViewClient() {
    // API < 24
    override fun shouldOverrideUrlLoading(view: WebView?, url: String?): Boolean {
        url?.let { view?.loadUrl(it) }
        return true
    }
    // API >= 24
    override fun shouldOverrideUrlLoading(view: WebView?, req: WebResourceRequest?): Boolean {
        req?.url?.toString()?.let { view?.loadUrl(it) }
        return true
    }
}
```

### ⬇️ 다운로드 처리
```kotlin
webView.setDownloadListener { url, userAgent, contentDisposition, mimeType, _ ->
    val request = DownloadManager.Request(Uri.parse(url))
    val dm = getSystemService(Context.DOWNLOAD_SERVICE) as DownloadManager
    dm.enqueue(request)
}
```

### 🧪 JavaScript 실행/브릿지
```kotlin
// 실행 결과 콜백 (API 19+)
webView.evaluateJavascript("document.body.style.background='red'\u003B") { result ->
    Log.d("WebView", "JS result: $result")
}

// JS ↔ Android 브릿지
class WebAppInterface(private val context: Context) {
    @JavascriptInterface fun showToast(message: String) {
        Toast.makeText(context, message, Toast.LENGTH_SHORT).show()
    }
}
webView.settings.javaScriptEnabled = true
webView.addJavascriptInterface(WebAppInterface(this), "Android")
```

### 🔒 보안 고려사항
- 불필요한 JavaScript 비활성화
- 파일 접근 허용 옵션 신중히 사용
- URL/입력 검증으로 XSS/스푸핑 방지
- `@JavascriptInterface`로 노출된 메서드 최소화·검증

### 🧠 요약
- `WebView`는 앱 내 미니 브라우저로, 로드/내비게이션/JS/다운로드를 커스텀 가능
- `WebViewClient`로 인앱 내비게이션 유지, `evaluateJavascript`와 브릿지로 상호작용
- 보안·성능 영향 고려, 필요 시 `AndroidX WebKit`로 최신 기능 활용

### ❓ 실전 질문
- 인앱 결제/리디렉션이 많은 환경에서 외부 브라우저로 튀지 않게 하려면 어떻게 구성하나요?
- `addJavascriptInterface` 사용 시 어떤 보안 수칙을 지켜야 하나요?

답변
- 외부 브라우저 방지: 반드시 `WebViewClient`를 설정하고 `shouldOverrideUrlLoading`에서 `view.loadUrl(...)`로 인앱 처리. 결제 스킴/딥링크는 화이트리스트 기반으로 `Intent` 처리 분기
- JS 브릿지 보안: 필요한 기능만 최소한으로 노출, 신뢰된 도메인만 로드, 입력 검증/XSS 방지, 파일 접근 제한, `evaluateJavascript` 사용, HTTPS 강제

# 카테고리 2 : Jetpack 라이브러리  

Jetpack은 안드로이드 개발자가 애플리케이션을 더 효율적이고 유지관리 가능하게 구축하는 데  
도움이 되도록 구글에서 제공하는 라이브러리 및 툴 모음  

Jetpack은 모듈식, 즉 라이브러리 형태로 제공되므로 개발자는 프로젝트에 필요한  
특정 라이브러리만 포함하도록 선택할 수 있습니다.  
ViewModel, Navigation, Room등 유연하게 필요와 상황에 따라서 통합할 수 있습니다.  

## Q49. AppCompat 라이브러리란 무엇인가요?

### 🎯 개요
AppCompat 라이브러리는 개발자가 하위 버전의 안드로이드와의 호환성을 유지하는 데  
도움이 되도록 설계된 Android Jetpack 제품군의 일부입니다.  
하위 안드로이드 버전과의 하위 호환성을 보장하면서 최신 기능을 앱에 사용할 수 있습니다.

#### 🛠️ 주요 기능
- UI 컴포넌트 하위 호환성  
  AppCompat 라이브러리는 FragmentActivity를 확장하고 하위 버전의 안드로이드와의 호환성을 보장하는 AppCompatActivity와 같은 최신 UI 컴포넌트를 제공합니다.  
  이를 통해 개발자는 하위 안드로이드 버전을 실행하는 기기에서 액션 바와 같은 기능을 사용할 수 있습니다.
- Material Design 지원  
  AppCompat을 사용하면 개발자는 하위 안드로이드 버전을 실행하는 기기에  
  Material Design 원칙을 통합할 수 있습니다.  
  AppCompatButton, AppCompatTextView등 기기의 API 레벨에 따라 모양이나 동작을 자동으로 조정하는 위젯이 제공됩니다.  
- 테마 및 스타일링 지원  
  AppCompat을 사용하면 Theme.AppCompat과 같은 테마를 사용하여  
  모든 API 레벨에서 일관된 UI를 보장할 수 있습니다.  
- 동적 기능 지원  
  AppCompat 라이브러리는 동적 리소스 로딩 및 벡터 드로어블 지원을 제공하여  
  하위 호환성을 유지하면서 최신 디자인 요소를 효율적으로 구현하기 쉽게 만듭니다.  

#### ⚙️ AppCompat을 사용하는 이유  

AppCompat 라이브러리를 사용하는 주된 이유는 최신 안드로이드 기능과  
UI 컴포넌트가 지원되는 모든 API 레벨에서 일관되게 작동하도록 보장하는 것입니다.  

#### ⚙️ AppCompat 사용 예제

```kotlin
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
```

#### 🧠 요약
AppCompat 라이브러리는 광범위한 기기 및 API 레벨과 호환되는 안드로이드 애플리케이션을  
구축하는 데 유용합니다.  

#### ❓ 실전 질문
#### AppCompat 라이브러리는 하위 안드로이드 버전에서 Material Design 지원을 어떻게 가능하게 하며, 이와 같은 동작을 기반으로 하는 주요 UI 컴포넌트에는 무엇이 있나요?

Theme.AppCompat 테마를 통해서 리플 효과, 그림자, 색상 팔레트 등을  
하위 API에서도 동일하게 표현합니다.  
ApppCompatButton, AppCompatTextView, AppCompatEditText등을 지원합니다.  

## 📚 Q50. Material Design Components란 무엇인가요?

### 🎯 개요
Material Design Components는 Google의 Material Design 가이드라인을 기반으로  
하는 커스텀 가능한 UI 위젯 및 컴포넌트 집합입니다.  
해당 컴포넌트는 일관되고 사용자 친화적인 인터페이스를 제공하도록 설계되었습니다.  
이러한 원칙이 효과적으로 구현되도록 보장하는 Material Components for Android 라이브러리를 제공합니다. 

#### 🛠️ Material Design Components의 주요 특징
1. Material Theming  
  MDC는 Material Theming을 통해 테마 설정을 지원하여  
  개발자가 타이포그래피, 모양 및 색상을 전역적으로 또는 컴포넌트 수준에서 커스텀할 수 있도록 합니다.
2. 미리 빌드된 UI 컴포넌트  
  MDC는 버튼, 카드, 앱 바, 내비게이션 드로어, 칩 등과 같이 즉시 사용할 수 있는  
  광범위한 UI 컴포넌트를 제공합니다. 이러한 컴포넌트는 접근성, 성능 및 반응성에 최적화되어 있습니다.
3. 애니메이션 지원  
  모션 및 전환을 강조합니다.  
  공유 요소 전환, 리플 효과 및 시각적 피드백과 같은 애니메이션에 대한 내장 지원이 포함되어 사용자 상호작용을 향상시킵니다.  
4. 다크 모드 지원  
5. 접근성

#### ⚙️ Material Button 사용 예제
```xml
<com.google.android.material.button.MaterialButton
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Click Me"
    app:cornerRadius="8dp"
    app:icon="@drawable/ic_example"
    app:iconGravity="start"
    app:iconPadding="8dp" />
```

#### 🧠 요약  
Material Design Components를 사용하여 개발자는  
가이드라인을 준수하고 현대적이고 일관된 사용자 인터페이스를 만들 수 있습니다.  

## 📚 Q51. ViewBinding을 사용하면 어떤 장점이 있나요?

### 🎯 개요
ViewBinding은 레이아웃의 뷰와 상호 작용하는 프로세스를 단순화하기 위해 안드로이드에서 도입된 기능입니다.  
수동으로 findViewById()를 호출하지 않아도 되고, 뷰에 접근하는 타입-세이프 방식을 제공하여 보일러 플레이트 코드를 줄이고 잠재적인 런타임 오류를 최소화합니다.  

#### 🛠️ ViewBinding 작동 방식
프로젝트에서 ViewBinding을 활성화하면 안드로이드는 각 XML 레이아웃 파일에 대한 바인딩 클래스를 생성합니다.  
생성된 바인딩 클래스의 이름은 레이아웃 파일 이름에서 파생되며  
각 밑줄은 카멜 케이스로 변환되고 이름 끝에 Binding이 추가됩니다.  
예) activity_main.xml -> ActivityMainBinding

```kotlin
// Activity에서의 사용 예제
class MainActivity : AppCompatActivity() {
    // 바인딩 클래스 인스턴스 선언
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // 레이아웃 인플레이트 및 바인딩 클래스 초기화
        binding = ActivityMainBinding.inflate(layoutInflater)
        // 루트 뷰를 액티비티의 콘텐츠 뷰로 설정
        setContentView(binding.root)
        // 바인딩 객체를 통해 뷰에 직접 접근
        binding.textView.text = "Hello, ViewBinding!"
        binding.button.setOnClickListener { /* 클릭 리스너 로직 */ }
    }
}
```  
inflate는 바인딩 클래스의 인스턴스를 생성하는 데 사용되고 binding.root는 레이아웃을 설정하기 위해 setContentView()에 전달됩니다.  

#### ⚙️ ViewBinding의 장점  
- 타입 안정성  
  캐스팅할 필요 없이 뷰에 직접 접근하여 타입 불일치로 인한 런타임 오류를 제거합니다.  
- 더 깔끔한 코드  
  findViewById()를 호출할 필요가 없습니다.  
- Null 안정성  
  nullable 타입의 뷰를 자동으로 처리하여 더 안전한 코드를 보장합니다.  
- 성능
  DataBinding과 달리 ViewBinding은 바인딩 표현식이나 추가 XML 파싱을 사용하지 않으므로 런타임 오버헤드가 최소화됩니다.  

#### 🧠 요약
ViewBinding은 안드로이드 앱에서 뷰와 상호작용하는 가볍고 타입-세이프한 방법으로  
보일러 플레이트 코드를 줄이고 코드 안전성을 향상시킵니다.