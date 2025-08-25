### Q45. 안드로이드의 Bitmap이란 무엇이며, 큰 Bitmap을 효율적으로 처리하는 방법은 무엇인가요?
- Bitmap은 메모리 내의 이미지 표현 
- 픽셀 데이터를 저장하며 리소스/파일/원격 소스에서 가져온 이미지를 화면에 렌더링하기 위해 사용됨
- 고해상도 이미지의 경우, 대량의 픽셀 데이터를 보유하므로 부적절하게 처리하면 메모리 고갈 및 OOM가 쉽게 발생할 수 있음

#### 큰 Bitmap의 문제점
- 카메라나 인터넷에서 다운로드와 이미지와 같은 이미지들은 이를 표시하는 UI 컴포넌트에서 요구하는 크기보다 훨씬 큼
- 이런 전체 해상도 이미지를 불필요하게 로드하면 문제가 발생할 수 있음
  - 과도한 메모리 소비
  - 성능 오버헤드 유발
  - 메모리 압박으로 인해 크래시 발생 가능
 
#### 메모리를 할당하지 않고 Bitmap 크기 읽기
- Bitmap을 로드하기 전에 전체 로드가 정당한지 결정하기 위해 크기를 검사하는 것이 중요함
- BitmapFactory.Options 클래스를 사용하면 `inJustDecodeBounds = true`로 설정하여 픽셀 데이터에 대한 메모리 할당을 피하면서 이미지 메타데이터를 디코딩할 수 있음
- 이를 통해 이미지 크기가 표시 요구 사항과 일치하는지 평가하여 불필요한 메모리 할당을 방지할 수 있음

#### 샘플링을 사용하여 축소된 Bitmap 로드하기
- 크기를 알면 inSampleSize 옵션을 사용하여 비트맵을 대상 크기에 맞게 축소할 수 있음
- 이는 이미지를 2, 4 등의 배수로 서브샘플링하여 메모리 사용량을 줄임
- 이미지 품질과 메모리 효율성의 균형을 맞추는 데 도움이 됨

#### 서브샘플링을 사용한 전체 디코딩 프로세스
- calculateInSampleSize를 사용하여 두 단계로 비트맵을 디코딩할 수 있음
(1) 경계만 디코딩
(2) 계산된 inSampleSize를 설정하고 축소된 비트맵을 디코딩
- UI가 최적화된 메모리 사용량으로 적절한 크기의 이미지를 얻도록 보장함

#### 실전 질문
##### Q) 큰 Bitmap을 메모리에 로드하는 것은 어떤 위험성이 있으며, 어떻게 효율적으로 처리할 수 있나요?

#### Pro Tips for Mastery: 커스텀 이미지 로딩 시스템에서 큰 비트맵 캐싱을 어떻게 구현하나요?
- 이미지 리스트, 그리드, 캐러셀을 처리할 때 원활하고 메모리 안전한 안드로이드 애플리케이션을 구축하려면 큰 비트맵을 효율적으로 관리하는 것이 필수적임
- 안드로이드는 2가지 효과적인 전략을 제공하는데, LruCache를 사용한 인메모리 캐싱과 DiskLruCache를 사용한 디스크 기반 캐싱

##### LruCache를 사용한 메모리 내 캐싱
- LruCache는 Bitmap을 사용할 때 선호되는 메모리 내 캐싱 솔루션
- 최근에 사용된 항목에 대한 강한 참조를 유지하고, 메모리가 부족할 때 가장 오랫동안 사용되지 않은 항목을 자동으로 제거함
- 메모리 오버플로우 위험을 방지하기 위해 사용 가능한 메모리의 약 1/8을 할당함
- 이를 통해 메모리에 빠르게 접근하고 중복 디코딩을 방지할 수 있음

```
object LruCacheManager {
    // 사용 가능한 최대 메모리 (KB)
    val maxMemory = (Runtime.getRuntime().maxMemory() / 1024).toInt()
    // 캐시 크기를 최대 메모리의 1/8로 설정
    val cacheSize = maxMemory / 8
    val memoryCache = object : LruCache<String, Bitmap>(cacheSize) {
            // 캐시 항목 크기를 KB 단위로 정의
            override fun sizeOf(key: String, bitmap: Bitmap): Int {
            // Bitmap의 바이트 수를 KB로 변환
            return bitmap.byteCount / 1024
        }
    }
}

fun loadBitmap(imageId: int, imageView: ImageView) {
    val key = imageId.toString()

    // 메모리 캐시에서 먼저 확인
    LruCacheManager.memoryCache.get(key)?.let {
        imageView.setImageBitmap(it)
    } ?: run {
        // 캐시에 없으면 플레이스홀더 설정 및 백그라운드 작업 요청
        imageView.setImageResource(R.drawable.image_placeholder)

        val workRequest = OneTimeWorkRequestBuilder<BitmapDecodeWorker>()
            .setInputData(workDataOf("imageId" to imageId))
            .build()
        WorkManager.getInstance(context).enqueue(workRequest)
    }
}

class BitmapDecodeWorker(
    context: Context,
    workerParams: WorkerParameters
): CoroutineWorker(context, workerParams) {

    override suspend fun doWork(): Result {
        val imageId = inputData.getInt("imageId", -1)
        if (imageId == -1) return Result.failure()

        val bitmap = decodeSampleBitmapFromResource(
            res = applicationContext.resources,
            resId = imageId,
            reqWidth = 100,
            reqHeight = 100
        )

        bitmap?.let {
            LruCacheManager.memoryCache.put(imageId.toString(), it)
            return Result.success()
        }

        return Result.failure()
    }
}
```
- 메모리 핸들링 로직에 SoftReference, WeakReference를 사용하지 않은 이유는, 공격적인 GC로 인해 캐싱값 자체를 더 이상 신뢰할 수 없는 상황이 발생할 수 있기 때문에

##### SoftReference, WeakReference를 사용했을 때 발생할 수 있는 문제점


##### DiskLruCache를 사용한 디스크 내 캐싱
- 안드로이드에서 메모리는 제한적이고 휘발적이라는 특성을 가짐
- Bitmap이 앱 세션 간에 지속되고 재계산을 피하도록 보장하려면 DiskLruCache를 사용하여 Bitmap을 디스크에 저장할 수 있음
- 이는 특히 리소스 집약적인 이미지나 스크롤 가능한 이미지 목록을 처리할 때 유용함

(1) 디코딩된 Bitmap을 유지하기 위해 안전한 해싱 및 I/O 로직으로 DiskLruCache를 래핑하는 DiskCacheManager를 작성할 수 있음
```
class DiskCacheManager(
    context: Context,
    cacheDirName: String = "images",
    cacheSize: Long = 10 * 1024 * 1024
) {
    private var diskLruCache: DiskLruCache? = null
    private val lock = Any()

    init {
        val cacheDir = getDiskCacheDir(context, cacheDirName)
        if (!cacheDir.exists()) {
            cacheDir.mkdirs()
        }
        try {
            // DiskLruCache 열기
            diskLruCache = DiskLrucache.open(cacheDir, 1, 1, cacheSize)
        } catch(e: IOException) {
            e.printStackTrace()
        }
    }

    private fun getDiskCacheDir(context: Context, uniqueName: String): File {
        val cachePath = context.cacheDir.path
        return File(cachePath + File.seperator + uniqueName)
    }

    private fun filenameForKey(key: String): String {
        return try {
            val messageDigest = MessageDigest.getInstance("SHA-1")
            messageDigest.update(key.toByteArray())
            byteToHexString(messageDigest.digest())
        } catch(e: NoSuchAlgorithmException) {
            key.hashCode().toString()
        }
    }

    private fun bytesToHexString(bytes: ByteArray): String {
        val sb = StringBuilder()
        for (b in bytes) {
            val hex = Integer.toHexString(0xFF and b.toInt())
            if (hex.length) == 1) {
                sp.append('0')
            }
            sb.append(hex)
        }
        return sb.toString()
    }

    // 디스크 캐시에서 비트맵 가져오기
    fun get(key: String): Bitmap? {
        synchronized(lock) {
            val safeKey = filenameForKey(key)
            var snapshot: DiskLruCache.Snapshot? = null
    
            return try {
                snapshot = diskLruCache?.get(safeKey)
                snapshot?.getInputStream(0)?.use { inputStream ‐>
                    BitmapFactory.decodeStream(inputStream)
                }
            } catch (e: IOException) {
                e.printStackTrace()
                null
            } finally {
                snapshot?.close()
            }
        }
    }

    fun set(key: String, bitmap: Bitmap) {
        synchronized(lock) {
            val safeKey = filenameForKey(key)
            var editor: DiskLruCache.Editor? = null
            try {
                editor = diskLruCache?.edit(safeKey)
                if (editor != null) {
                    editor.newOutputStream(0).use { outputStream ->
                        bitmap.compress(Bitmap.CompressFormat.JPEG, 100, outputStream)
                        editor.commit()
                    }
                } else {
                    diskLruCache?.flush()
                }
            } catch(e: IOException) {
                e.printStackTrace()
                try {
                    editor?.abort()
                } catch(ignored: IOException) {
                    
                }
            }
        }
    }
}
```
- 위 클래스는 다음을 보장함
(1) 디스크-안전한 SHA-1 기반 파일 이름 생성
(2) 안전한 I/O 작업
(3) 디스크 캐시에 중복으로 데이터를 쓰는 행위 방지

### Q46. 애니메이션을 어떻게 구현하나요?
- 애니메이션은 부드러운 전환을 만들고, UI 변화에 대해서 사용자들의 이목을 집중시키며, 시각적 피드백을 제공하여 사용자 경험을 향상시킬 수 있음

#### View Property Animations
- alpha, translationX, translationY, rotation, scaleX와 같은 View 객체의 속성을 애니메이션화할 수 있음
- 간단한 뷰에 간단한 변화를 줄 때 사용함

#### ObjectAnimator
- View 객체뿐만 아니라 setter 메서드가 있는 모든 객체의 속성을 애니메이션화할 수 있음
- 커스텀 속성을 애니메이션화하는 데 더 큰 유연성을 제공함

#### AnimatorSet
- 여러 애니메이션을 순차적으로 또는 동시에 실행하도록 결합하여 복잡한 애니메이션을 조정하는 데 적합

#### ValueAnimator
- 임의의 값 사이를 애니메이션화하는 방법을 제공하여 커스텀 가능하고 유연한 애니메이션을 구현할 수 있음
- interpolator로 애니메이션 진행을 제어함으로써 너비/높이/알파 또는 기타 속성을 애니메이션화하는 등 광범위하게 사용할 수 있음
- 특정 요구 사항에 맞는 정밀하고 동적인 애니메이션을 구현하는 데 적합함

#### XML 기반 View 애니메이션
- 단순성과 재사용성을 위해 리소스 파일에서 애니메이션 동작을 정의함
- 위치, 크기 조절, 회전, 투명도 조절에 사용할 수 있음

#### MotionLayout
- 복잡한 모션 및 레이아웃 애니메이션을 만들기 위한 안드로이드 전용 컴포넌트
- ConstraintLayout 위에 구축되었으며, XML을 사용하여 상태 간의 애니메이션 및 전환을 정의할 수 있음
- 전환 및 상태에 대한 정밀한 제어로 정교한 애니메이션을 만드는 데 적합함

#### Drawable 애니메이션
- AnimationDrawable을 사용하여 프레임별 전환을 포함하며, 로딩 스피너와 같은 간단한 애니메이션을 만드는 데 적합함

#### 물리 기반 Animations
- 실제 역학을 시뮬레이션
- 자연스럽고 동적인 모션 효과를 만들기 위해 SpringAnimation 및 FlingAnimation API를 제공함

#### 실전 질문
(1) 클릭 시 확장 및축소되는버튼에부드러운애니메이션을추가하려고 합니다. 어떻게 구현해 볼 수 있을까요
(2) 전통적인 View 애니메이션 대신 MotionLayout을 사용하는 사례는 무엇이고, 그 장점은 무엇인가요?

#### Pro Tips for Mastery: 인터폴레이터는 애니메이션과 어떻게 작동하나요?
- Interpolator는 애니메이션 값 변경 속도를 수정하여 애니메이션이 시간 경고에 따라 어떻게 진행되는지 정의함
- 애니메이션의 가속, 감속, 등속 운동을 제어하여 더 자연스럽거나 시각적으로 매력적으로 보이게 만듬
- Interpolator는 시작 값과 끝 값 사이의 애니메이션 동작을 정의하는 데 사용됨
- 애니메이션이 느리게 시작하여 속도를 높인 다음 멈추기 전에 느려지도록 만들 수 있음
- 이는 선형 진행을 넘어 애니메이션 실행 방식을 유연하게 제어할 수 있도록 함

(1) LinearInterpolator : 가속이나 감속 없이 일정한 속도로 애니메이션 
(2) AccelerateInterpolator : 느리게 시작하여 점진적으로 속도를 높임
(3) DecelerateInterpolator : 빠르게 시작하여 끝으로 갈수록 느려짐
(4) AccelerateDecelerateInterpolator : 부드러운 효과를 위해 가속과 감속을 모두 결합
(5) BounceInterpolator : 스프링 애니메이션을 모방하여 애니메이션이 튕기는 것처럼 보이게 함
(6) OvershootInterpolator : 최종 값을 초과하여 애니메이션한 후 다시 정착함

- ObjectAnimator, ValueAnimator, ViewPropertyAnimator와 같은 애니메이션 객체에 인터폴레이터를 적용할 수 있음(setInterpolator() 메서드로)

### Q47. Window란 무엇인가요?
- Window는 화면에 표시되는 Activity 또는 다른 UI 컴포넌트의 모든 뷰를 담는 컨테이너를 나타냄
- View 계층 구조의 최상위 요소이며 애플리케이션 UI와 디스플레이 간의 다리 역할을 함
- 모든 Activity, Dialog, Toast는 Window 객체에 연결되어 있으며, 이는 포함된 뷰의 레이아웃 매개변수, 애니메이션 및 전환 기능 등을 제공함

#### Window의 주요 특징
(1) DecorView
- Window는 계층 구조의 루트 뷰 역할을 하는 DecorView를 포함
- 일반적으로 상태 표시줄, 네비게이션 바, 앱의 콘텐츠 영역을 포함함
(2) 레이아웃 매개변수
- Window는 크기, 위치, 가시성과 같은 레이아웃 매개변수를 사용하여 뷰가 어떻게 정렬되고 표시되는지 정의함
- 이는 프로그래밍 방식으로 커스텀할 수 있음
(3) 입력 처리
- Window는 입력 이벤트를 처리하고 이를 적절한 뷰로 전달함
(4) 애니메이션 및 전환
- Window는 화면 열기, 닫기, 화면 간 전환을 위한 애니메이션을 지원함
(5) 시스템 UI 처리
- Window는 상태 표시줄 및 네비게이션 바와 같은 시스템 UI 요소를 표시하거나 숨길 수 있음

#### Window 관리
- Window는 윈도우 추가/제거 또는 업데이트를 담당하는 시스템 서비스인 WindowManager에 의해 관리됨
- 이를 통해 다양한 윈도우(앱 윈도우, 시스템 다이얼로그, 알림)가 기기에서 올바르게 공존하고 상호작용할 수 있음

#### Window 사용 사례
(1) Activity Window 커스텀 : `getWindow()` 메서드를 사용하여 Activity의 윈도우 동작을 수정할 수 있음. 상태 표시줄을 숨기거나 배경 색상을 변경하는 등
(2) Dialog 생성 : Dialog는 새로운 윈도우를 사용하여 그 위에 구현되므로 다른 UI 요소 위에 떠있을 수 있음
(3) 오버레이 사용 : TYPE_APPLICATION_OVERLAY를 통해 시스템 수준 기능이나 헤드업 알림과 같은 오버레이를 만드는 데 Window를 사용할 수 있음
(4) 멀티 윈도우 모드 처리 : 안드로이드는 분할 화면이나 PIP 모드와 같은 기능을 활성화하기 위해 멀티 윈도우를 지원함

#### 실전 질문
(1) 단순한 레이아웃을 가진 Activity가 화면에 표시될 때 몇 개의 Window가 존재하며, 어느 부분에 필요한가요?

#### Pro Tips for Mastery: WindowManager란 무엇인가요?
- WindowManager는 화면에서 윈도우의 배치, 크기 및 모양을 관리하는 안드로이드 시스템 서비스
- 애플리케이션과 안드로이드 시스템 간의 윈도우 관리 인터페이스 역할을 하며, 앱이 윈도우를 생성, 수정, 제거할 수 있도록 함
- 안드로이드의 윈도우는 전체 화면 Activity부터 플로팅 오버레이까지 무엇이든 될 수 있음

##### WindowManager의 주요 책임
- WindowManager는 시스템의 윈도우 계층 구조를 관리하는 역할을 함
- 윈도우가 Z-Index(레이어링) 및 다른 시스템 윈도우와의 상호 작용에 따라 올바르게 표시되도록 보장함
- 예를 들어 윈도우의 포커스 변경, 터치 이벤트 및 애니메이션을 처리함

##### 일반적인 사용 사례
(1) 커스텀 View 추가: 개발자는 WindowManager를 사용하여 플로팅 위젯이나 시스템 오버레이와 같이 앱의 표준 Activity 외부에 커스텀 뷰를 표시할 수 있음
(2) 기존 Window 수정: 애플리케이션은 크기 조절, 위치 변경 또는 투명도 변경과 같이 기존 윈도우의 속성을 업데이트할 수 있음
(3) Window 제거: `removeView()` 메서드를 사용하여 프로그래밍 방식으로 윈도우를 제거할 수 있음

- TYPE_APPLICATION_OVERLAY는 뷰가 다른 앱 위에 표시되도록 허용함(권한 필요)
- FLAG_NOT_FOCUSABLE과 같은 플래그는 달리 지정하지 않는 한 윈도우가 사용자 입력과 상호작용하지 않도록 함

##### 제한 사항 및 권한
- 시스템 오버레이와 같은 특정 유형의 윈도우에는 SYSTEM_ALERT_WINDOW와 같은 특별한 사용자 권한이 필요함
- 안드로이드 8.0(API 28)부터 시스템은 보안상의 이유로 오버레이 윈도우에 대해 더 엄격한 제한을 부과함

#### Pro Tips for Mastery: PopupWindow란 무엇인가요?
- PopupWindow는 기존 레이아웃 위에 떠 있는 팝업 뷰를 표시하는 데 사용되는 UI 컴포넌트
- 일반적으로 화면을 덮고 해제하기 위해 사용자 상호작용이 필요한 Dialog와 달리, PopupWindow는 더 유연하며 화면의 특정 위치에 배치될 수 있음
- 메뉴나 툴팁과 같은 임시/상황별 UI 요소에 자주 사용됨

(1) 레이아웃 위에 존재하는 어떠한 뷰든 해당 뷰 위에 콘텐츠를 표시할 수 있음
(2) 팝업 외 영역 화면을 어둡게 하거나, 사용자 상호작용을 차단할 필요가 없는 시나리오에서 팝업 뒤의 다른 UI 컴포넌트와 상호작용할 수 있음
(3) 커스텀 레이아웃, 애니메이션 및 해제 동작을 구현할 수 있음
(4) 원활한 사용자 경험을 위해 터치 기반 해제 및 포커스 제어를 지원함

### Q48. 웹 페이지를 어떻게 렌더링하나요?
- WebView는 앱 내에서 직접 웹 컨텐츠를 표시하고 상호 작용할 수 있는 안드로이드 컴포넌트
- 애플리케이션에 내장된 미니 브라우저 역할을 하여 개발자가 웹 페이지르 렌더링하고, HTML 콘텐츠를 로드하거나, JavaScript를 직접적으로 실행할 수 있도록 함
- 앱이 실행중인 기기에서 최신 WebView 기능을 안전하게 활용하려면 AndroidX Webkit 라이브러리를 사용해야 함
- 이 라이브러리는 이전 버전과 호환되는 API를 제공하여 기기의 안드로이드 버전에 관계 없이 최신 기능에 접근할 수 있도록 보장함

(1) WebView 초기화하기
(2) 웹 페이지 로드하기
- WebView 인스턴스에서 `loadUrl()` 메서드를 사용함
- 페이지가 인터넷 접근을 요구하는 경우 Manifest 파일에서 필요한 권한을 활성화해야함
(3) JavaScript 활성화하기
- WebSettings를 수정하여 활성화
(4) WebView 동작 커스텀하기
- 페이지 네비게이션 가로채기: 외부 브라우저에서 열지 않고 WebView 내에서 페이지 네비게이션을 처리하려면 WebViewClient를 사용해야 함
- 다운로드 처리하기: WebView를 통해 다운로드되는 파일을 관리하려면 DownloadListener를 활용할 수 있음
- evaluateJavaScript 또는 `loadUrl()`을 사용하여 JavaScript 코드를 주입함

#### 보안 고려 사항
(1) 보안 위험에 앱을 노출시킬 수 있으르 필요하지 않는 한 JavaScript를 활성화하지 않는 것이 좋음
(2) 파일에 대한 무단 접근을 방지하기 위해 `setAllowFileAccess()` 및 `setAllowFileAccessFromFileUrls(`)를 신중하게 사용하는 것이 좋음
(3) 교차 사이트 스크립팅(XSS), URL 스푸핑과 같은 보안 취약점을 유발하지 않는지 확인해야 함
(4) @JavascriptInterface를 통해 노출된 메서드가 보안 취약점을 유발하지 않는지 확인해야 함

#### 실전 질문
(1) 외부 링크를 클릭할 때 사용자가 앱을 벗어나는 것을 방지하기 위해 WebView 내비게이션을 효과적으로 처리하는 방법에는 무엇이 있는지 설명해 주세요

### Q49. AppCompat 라이브러리란 무엇인가요?
- AppCompat 라이브러리는 개발자가 하위 버전의 안드로이드와의 호환성을 유지하는 데 도움이 되도록 설계된 Android Jetpack 제품군의 일부
- 하위 안드로이드 버전과의 하위 호환성을 보장하면서 최신 기능을 앱에 사용할 수 있음

(1) UI 컴포넌트 하위 호환성
- AppCompat 라이브러리는 FragmentActivity를 확장하고 하위 버전의 안드로이드와의 호환성을 보장하는 AppCompatActivity와 같은 최신 UI 컴포넌트를 제공함
(2) Material Design 지원
- AppCompat을 사용하면 개발자는 하위 안드로이드 버전을 실행하는 기기에 Material Design 원칙을 통합할 수 있음
(3) 테마 및 스타일링 지원
- Theme.AppCompat과 같은 테마를 사용하여 모든 API 레벨에서 일관된 UI를 보장할 수 있음
- 이런 테마는 벡터 드로어블 지원과 같은 최신 스타일링 기능을 하위 안드로이드 버전에 제공함
(4) 동적 기능 지원
- 동적 리소스 로딩 및 벡터 드로어블 지원을 제공하여 하위 호환성을 유지하면서 최신 디자인 요소를 효율적으로 구현하기 쉽게 만듬

#### AppCompat을 사용하는 이유
- AppCompat 라이브러리를 사용하는 주된 이유는 최신 안드로이드 기능과 UI 컴포넌트가 지원되는 모든 API 레벨에서 일관되게 작동하도록 보장하는 것

### Q50. Material Design Components(MDC)란 무엇인가요?
- Google의 Material Design 가이드라인을 기반으로 하는 커스텀 가능한 UI 위젯 및 컴포넌트 집합
- 해당 컴포넌트는 개발자가 앱의 브랜딩 및 디자인 요구사항에 맞게 모양과 동작을 커스텀할 수 있도록 하면서 일관되고 사용자 친화적인 인터페이스를 제공하도록 설계되었음

#### Material Design Components의 주요 특징
(1) Material Theming
- Material Theming을 통해 테마 설정을 지원하여 개발자가 타이포그래피, 모양, 색상을 전역적으로 또는 컴포넌트 수준에서 커스텀할 수 있도록 함
- 이를 통해 앱 전체에서 일관성을 유지하면서 UI를 브랜드 아이덴티티에 맞게 쉽게 맞출 수 있음
(2) 미리 빌드된 UI 컴포넌트
- 버튼, 카드, 앱 바, 네비게이션 드로어, 칩 등과 같이 즉시 사용할 수 있는 광범위한 UI 컴포넌트를 제공함
- 이러한 컴포넌트들은 접근성, 성능, 반응성에 최적화되어 있음
(3) 애니메이션 지원
- Material Design은 모션 및 전환을 강조함
- 공유 요소 전환, 리플 효과 및 시각적 피드백과 같은 애니메이션에 대한 내장 지원이 포함되어 사용자 상호 작용을 향상시킴
(4) 다크 모드 지원
- 다크 모드를 쉽게 구현할 수 있는 API가 포함되어 있어 개발자가 시각적 일관성을 보장하면서 라이트/다크 모드에 대한 테마를 적용할 수 있음
(5) 접근성
- MDC는 더 큰 터치 대상, 접근성을 위한 sementic 레이블 및 적절한 포커스 관리와 같은 기능을 제공하여 접근성 표준을 준수하며, 모든 사용자를 위한 포괄적인 UI를 보장함

#### 실전 질문
(1) MDC의 Material Theming은 앱 전체에서 디자인 일관성을 유지하는 데 어떻게 도움이 되나요?

### Q51. ViewBinding을 사용하면 어떤 장점이 있나요?
- ViewBinding은 레이아웃의 View와 상호작용하는 프로세스를 단순화하기 위해 안드로이드에서 도입된 기능
- 수동으로 `findViewById()`를 호출하지 않아도 되고, View에 접근하는 타입-안전한 방식을 제공하여 보일러플레이트 코드를 줄이고 잠재적인 런타임 오류를 최소화함

#### ViewBinding 작동 방식
- 프로젝트에서 ViewBinding을 활성화하면 안드로이드는 각 XML 레이앙웃 파일에 대한 바인딩 클래스를 생성함
- 바인딩 클래스에는 레이아웃의 모든 View에 대한 참조가 포함되어 있어, 캐스팅하거나 `findViewById()`를 호출할 필요 없이 직접 접근할 수 있음

#### ViewBinding의 장점
(1) 타입 안전성 : 캐스팅할 필요 없이 직접 접근하여 타입 불일치로 인한 런타임 오류를 방지함
(2) 더 깔끔한 코드 : `findViewById()`를 호출할 필요가 없어지고 보일러 플레이트 코드가 줄어듬
(3) Null 안전성 : nullable 타입의 View를 자동으로 처리하여 선택적 UI 컴포넌트와 상호작용할 때 더 안전한 코드를 보장함
(4) 성능 : DataBinding과 달리 ViewBinding은 바인딩 표현식이나 추가 XML 파싱을 사용하지 않으므로 런타임 오버헤드가 최소화됨

#### DataBinding과의 비교
- DataBinding은 바인딩 표현식 및 양방향 데이터 바인딩과 같은 더 많은 기능을 제공하지만, 더 복잡하고 런타임 오버헤드를 유발함
- 반면, ViewBinding은 순수하게 View 상호 작용 단순화에 중점을 두며 성능 면에서 더 가벼움
- LiveData나 Flow 등을 바인딩하여 데이터를 직접적으로 결합하는 기능이 필요하지 않은 경우 이상적

#### 실전 질문
(1)  ViewBinding은 findViewById()와 비교하여 타입 안전성과 null 안전성을 어떻게 개선하며, 해당 접근 방식의 이점은 무엇인가요?









