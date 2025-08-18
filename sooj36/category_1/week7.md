#### Q38~Q44

## 🤔 Q38. View 시스템의 무효화(invalidation)란 무엇인가요?

### 무효화(Invalidation)
- View를 다시 그려야 함을 표시하는 프로세스
- 변경 사항이 발생할 때 UI를 업데이트하기 위한 기본 메커니즘

### 무효화 작동 방식
- `invalidate()` 또는 `postInvalidate()` 메서드 호출로 트리거
- 시스템은 View를 "더티(dirty)"로 플래그 지정
- 다음 프레임 동안 무효화된 View를 드로잉 패스에 포함시켜 시각적 표현을 업데이트


### 주요 메서드
1. **invalidate()**:
- 단일 View를 무효화, UI 스레드에서 사용합니다.
- View를 더티(변경이 필요한 부분)로 표시하여 
시스템이 다음 레이아웃 패스 중에 다시 그리도록 신호를 보냅니다.
- View를 다시 그리는 것이 아니라 다음 프레임을 위해 예약합니다

2. **invalidate(Rect dirty)**:
- View 내 특정 직사각형 영역만 무효화하여 더 작은 부분으로
다시 그리기하여 성능을 최적화 합니다.
3. **postInvalidate()**:
- UI 스레드가 아닌 스레드에서 View 무효화를 하며,
무효화 요청을 메인 스레드에 게시하여 스레드 안전성을 보장합니다.


**Q) invalidate() 메서드는 어떻게 작동하며 
    postInvalidate()와 어떻게 다른가요? 
    각각이 적합한 실제 사용 사례를 제시해주세요.**

```
**invalidate()** : 
- UI 스레드에서 즉시 호출, 
- 다음 draw cycle에서 onDraw() 실행 → 메인 스레드 전용

**postInvalidate()** :
- 백그라운드 스레드에서 안전하게 호출, 
- Handler를 통해 UI 스레드로 메시지 전달 후 invalidate() 실행

**사용 사례** :
- invalidate()는 터치/클릭 이벤트, 
- postInvalidate()는 네트워크/DB 작업 완료 후 UI 업데이트
```

---

## 🤔 Q39. ConstraintLayout이란 무엇인가요?

- 여러 레이아웃을 중첩하지 않고 복잡한 UI 생성 가능한 레이아웃
- 다른 뷰나 부모 컨테이너에 상대적인 제약 조건(constraints)을 사용하여 뷰 위치/크기 정의
- 플랫 뷰 계층 구조로 렌더링 속도를 높이고 앱 성능 향상

### 주요 특징
1. **제약 조건 기반한 위치 지정**
   - 뷰는 정렬, 중앙 배치 및 앵커링을 위한 제약 조건을 사용하여 
      형제 뷰 또는 부모 레이아웃에 상대적으로 위치 지정 가능

2. **유연한 크기 제어** 
   - match_constraint, wrap_content 및 고정 크기 같은 옵션을 제공하여
반응형 레이아웃 쉽게 디자인 가능

3. **체인(Chain) 및 가이드라인(Guideline) 지원**
   - 체인 사용 시 : 뷰를 동일한 간격으로 가로/세로로 그룹화 가능
   - 가이드라인 사용 시 : 고정 또는 백분율 기반 위치에 정렬 가능

4. **배리어(Barrier) 및 그룹핑(Grouping)**
   - 배리어 : 참조된 뷰의 크기에 따라 동적으로 조정
   - 그룹핑 : 여러 뷰의 가시성 변경을 단순화


### 장점
- 플랫 뷰 계층 구조로 렌더링 성능 향상 & 레이아웃 관리 단순화
- 반응형 디자인 지원
- Android Studio Layout Editor 지원

**Q) ConstraintLayout은 중첩된 
LinearLayout 및 RelativeLayout과 비교하여 성능을 어떻게 향상시키나요? 
ConstraintLayout 사용이 더 효율적인 시나리오를 말씀해주세요.**

1. ConstraintLayout은 플랫한 뷰 계층구조로 
중첩 레이아웃을 제거해 measure/layout 패스 횟수를 줄이고, 
단일 컨테이너에서 복잡한 관계 정의가 가능해 렌더링 성능이 향상

2. 복잡한 UI에서 LinearLayout 3-4단계 중첩이 필요한 경우나, 
화면 비율 기반 레이아웃, 체인/배리어가 필요한 반응형 디자인에서 더 유리

**Q) ConstraintLayout에서 match_constraint (0dp) 동작이 어떻게
작동하는지 설명해주세요. wrap_content 및 match_parent와 어떻게
다르며, 어떤 상황에서 사용해야 하나요?**

| 크기 설정 | 크기 결정 | 동작 방식 |
|---|---|---|
| **wrap_content** | 내용물 크기 | 텍스트나 이미지 등 내용물의 실제 크기에 맞춰 뷰 크기 조절 |
| **match_parent** | 부모 크기 | 부모 컨테이너의 전체 크기를 채움 |
| **match_constraint (0dp)** | Constraint 관계 | 설정된 constraint 사이의 공간을 모두 채우도록 확장 |

---

## 🤔 Q40. SurfaceView 대신 TextureView는 언제 사용해야 하나요?

### SurfaceView
- 별도 스레드에서 렌더링 처리
- 전용 드로잉 표면 제공
- 고성능 렌더링에 적합 (게임, 비디오 재생)
- 크기 조절이나 회전과 같은 변환이나 애니메이션에 제한

표면은 SurfaceHolder 콜백 메서드를 통해 생성 및 관리되며, 필요에 따라 렌더링을 시작하고 중지할 수 있습니다.
저수준 API를 사용하여 비디오를 재생하거나 게임 루프에서 그래픽을 계속해서 그리는데 SurfaceView를 사용할 수 있습니다.


```
class CustomSurfaceView(context: Context) : SurfaceView(context), SurfaceHolder.Callback {
    init {
        holder.addCallback(this)
    }

    override fun surfaceCreated(holder: SurfaceHolder) {
        // 여기서 렌더링 또는 드로잉 시작
    }

    override fun surfaceChanged(holder: SurfaceHolder, format: Int, width: Int, height: Int) {
        // 표면 변경 처리
    }

    override fun surfaceDestroyed(holder: SurfaceHolder) {
        // 여기서 렌더링 중지 또는 리소스 해제
    }
}
```


### TextureView 
- UI 계층 구조에 원활하게 통합되고
- 회전, 크기 조절, 알파 블렌딩 등 변환 작업이나 애니메이션화 가능
- 메인 스레드에서 작동 (따라 성능적으로 덜 효율적)
- 동적 상호작용이 필요한 UI에 적합


```
class CustomTextureView(context: Context, attrs: AttributeSet? = null) : TextureView(context, attrs), TextureView.SurfaceTextureListener {
    init {
        surfaceTextureListener = this
    }

    override fun onSurfaceTextureAvailable(surface: SurfaceTexture, width: Int, height: Int) {
        // 렌더링 시작 또는 SurfaceTexture 사용
    }

    override fun onSurfaceTextureSizeChanged(surface: SurfaceTexture, width: Int, height: Int) {
        // 표면 크기 변경 처리
    }

    override fun onSurfaceTextureDestroyed(surface: SurfaceTexture): Boolean {
        // 리소스 해제 또는 렌더링 중지
        return true // SurfaceTexture가 앱 프로세스에 의해 해제되었음을 나타냄
    }

    override fun onSurfaceTextureUpdated(surface: SurfaceTexture) {
        // 표면 텍스처 업데이트 처리 (프레임 업데이트 등)
    }
}

```

#### 차이점 : 각 컴포넌트가 렌더링 및 UI 통합을 처리하는 방식

### 선택 기준
- **SurfaceView**: 성능이 최우선인 경우 (비디오나 게임같은 연속 렌더링)
- **TextureView**: UI 통합 및 시각적 변환이 필요한 경우 (카메라 미리보기, 비디오 애니메이션)


---

## 🤔 Q41. RecyclerView는 내부적으로 어떻게 작동하나요?

### 핵심 메커니즘
1. **뷰 재활용(Recycling Views)**
- RecyclerView는 데이터 셋의 모든 항목에 대해 새 뷰를 생성하는 대신 기존 뷰를 재사용합니다. 뷰가 보이는 영역 밖으로 스크롤되면 소멸되는 대신 뷰 풀(RecyclerView.RecycledViewPool)에 추가됩니다. 
- 새 항목이 뷰에 들어오면 RecyclerView는 가능한 이 풀에서 기존 뷰를 검색하여 인플레이션 오버헤드를 피합니다.

/
2. **ViewHolder 패턴** 
- 항목 레이아웃 내 뷰에 대한 참조를 저장하기 위해 ViewHolder를 사용합니다. 
이는 바인딩 중 반복적인 findViewById() 호출을 방지하여 
레이아웃 순회 및 뷰조회를 줄여 성능을 향상시킵니다.

/
3. **Adapter의 역할**
- RecyclerView.Adapter는 데이터 소스와 RecyclerView를 연결하는 다리 역할을 합니다. 
어댑터의 onBindViewHolder() 메서드는 뷰가 재사용될 때 
데이터를 뷰에 바인딩하여 보이는 항목만 업데이트되도록 보장합니다

/
4. **RecycledViewPool**
   - 사용되지 않는 뷰를 저장하는 객체 풀 역할을 합니다.
   이를 통해 RecyclerView는 유사한 뷰 유형을 가진 여러 목록 또는 섹션에서 뷰를 재사용하여 메모리 사용량을 더욱 최적화 할 수 있습니다


### 재활용 메커니즘
1. **스크롤 및 항목 가시성**
: 사용자가 스크롤하면 뷰 밖으로 나가는 항목은 RecyclerView에서 분리되지만 소멸되지 않습니다. 대신 RecycledViewPool에 추가됩니다.

2. **재활용된 뷰에 데이터 리바인딩** : 새 항목이 뷰에 들어오면 RecyclerView는 먼저 RecycledViewPool에서 필요한 유형의 사용 가능한 뷰를 확인합니다. 
일치하는 항목이 발견되면 onBindViewHolder()를 사용하여 새 데이터로 리바인딩하여 뷰를
재사용합니다.

3. **사용 가능한 뷰가 없는 경우 인플레이션** : 풀에 적합한 뷰가 없는
경우 RecyclerView는 onCreateViewHolder()를 사용하여 새 뷰를
인플레이션합니다.

4. **효율적인 메모리 사용**: 뷰를 재활용함으로써 RecyclerView는 메
모리 할당 및 GC을 최소화하여 대규모 데이터 셋이나
빈번한 스크롤이 포함된 시나리오에서 발생할 수 있는 성능 문제를 줄입니다.

```
class MyAdapter(private val dataList: List<String>) : RecyclerView.Adapter<MyAdapter.MyViewHolder>() {

    // ViewHolder 클래스: 아이템 뷰의 참조를 저장
    class MyViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val textView: TextView = itemView.findViewById(R.id.textView)
    }

    // ViewHolder 생성: 새 뷰 인플레이션
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyViewHolder {
        val view = LayoutInflater.from(parent.context).inflate(R.layout.item_layout, parent, false)
        return MyViewHolder(view)
    }

    // ViewHolder에 데이터 바인딩
    override fun onBindViewHolder(holder: MyViewHolder, position: Int) {
        holder.textView.text = dataList[position]
    }

    // 데이터 셋 크기 반환
    override fun getItemCount(): Int = dataList.size
}

```

### 주요 구성요소
- **ViewHolder**: 뷰 참조 저장, findViewById() 호출 최소화
- **Adapter**: 데이터와 RecyclerView 연결
- **RecycledViewPool**: 재사용 가능한 뷰 저장소

### 다중 아이템 타입 구현
- `getItemViewType()` 오버라이드
- 각 타입별 ViewHolder 클래스 생성
- 뷰 타입에 따른 적절한 레이아웃 인플레이션


### **[실전 질문]**
**Q-1) RecyclerView의 ViewHolder 패턴은 ListView와 비교하여 성능
향상 측면에서 어떤 이점이 있나요?**

- ListView는
1. findViewById() 반복 호출 : 스크롤할 때마다 뷰 탐색
2. O(n) 복잡도 : 뷰 트리 전체 탐색
3. 메모리 비효율
4. notifyDataSetChanged()로 전체 리스트를 갱신

- ViewHolder는
1. 한 번만 탐색 : 생성 시점에 뷰 참조 저장
2. O(1) 접근 : 캐시된 참조로 바로 접근 가능
3. 객체 재사용  : RecycledViewPool 활용
4. DiffUtil을 통해 변경된 아이템만 효율적으로 업데이트 가능

- 1. 탐색 효율성 : O(n) -> O(1)
- 2. 메모리 효율성 : 객체 재사용

/
**Q-2)RecyclerView에서 ViewHolder의 생성부터 재활용까지의 생명주기
를 설명해주세요.**

1. 생성 (onCreateViewHolder)
- 새로운 ViewHolder 객체 생성
- 레이아웃 인플레이션
- findViewById로 뷰 참조 저장

2. 바인딩 (onBindViewHolder)
- ViewHolder에 데이터 연결
- 텍스트, 이미지 등 실제 내용 설정

3. 화면 표시(사용자가 실제로 보는 상태)
- 터치 이벤트 등 상호작용 가능

4. 화면 이탈(스크롤)
- 사용자가 스크롤하여 ViewHolder가 화면 밖으로 이동
- Scrap Heap 이동

5. 임시 저장
- RecycledViewPool 이동 + 데이터 초기화

6. 재사용
- 풀에서 기존 ViewHolder 꺼내고
- 새로운 데이터로 다시 바인딩 (2번으로 복귀)
핵심: ViewHolder 객체는 재생성하지 않음!

7. 최종 해제(RecyclerView가 완전히 파괴될 때) 
- 메모리에서 ViewHolder 객체 제거

/
**Q-3) RecycledViewPool이란 무엇이며, 뷰 아이템 렌더링을 최적화하는
데 어떻게 사용할 수 있나요?**

- 사용하지 않는 ViewHolder들을 뷰 타입별로 저장해두는 객체 풀입니다.
- 여러 RecyclerView가 동일한 Pool을 공유하거나, Pool 크기를 조정하여 
ViewHolder 생성 비용을 줄이고 메모리 효율성을 높입니다.


/
**💡 Pro Tips for Mastery: 동일한 RecyclerView에서 다른 유형의
아이템을 어떻게 구현하나요?**

- 동일한 목록에서 여러 아이템 유형을 지원합니다.
- 핵심은 아이템 유형을 구별하고 올바르게 바인딩하는 것입니다.

- 1. **아이템 유형 정의** : 
각 아이템 유형은 고유 식별자(일반적으로 상수)로 표시됩니다. 
이러한 식별자를 통해 어댑터는 뷰 생성 및 바인딩 중에 아이템 유형을 구별할 수 있습니다.

- 2. **getItemViewType() 재정의**: 
어댑터에서 getItemViewType() 메서드를 재정의하여 데이터 셋의 
각 아이템에 대해 적절한 유형을 반환합니다. 
이 메서드는 RecyclerView가 인플레이션할 레이아웃 유형을결정하는 데 도움이 됩니다.

- 3. **여러 ViewHolder 처리**: 
각 아이템 유형에 대해 별도의 ViewHolder 클래스를 생성합니다. 
각 ViewHolder는 해당 레이아웃에 데이터를 바인딩하는 역할을 합니다.

- 4. **뷰 유형에 따라 레이아웃 인플레이션**: 
onCreateViewHolder() 메서드에서 getItemViewType()에서 반환된 
뷰 유형에 따라 적절한 레이아웃을 인플레이션합니다.

- 5. **적절하게 데이터 바인딩** :
onBindViewHolder() 메서드에서 아이템 유형을 확인하고 
해당 ViewHolder를 사용하여 데이터를 바인딩합니다.


```
class MultiTypeAdapter(private val items: List<ListItem>) : RecyclerView.Adapter<RecyclerView.ViewHolder>() {

    // 아이템 유형 상수 정의
    companion object {
        const val TYPE_HEADER = 0
        const val TYPE_CONTENT = 1
    }

    // 위치에 따른 아이템 유형 반환
    override fun getItemViewType(position: Int): Int {
        return when (items[position]) {
            is ListItem.Header -> TYPE_HEADER
            is ListItem.Content -> TYPE_CONTENT
        }
    }

    // 뷰 유형에 따라 ViewHolder 생성
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        return when (viewType) {
            TYPE_HEADER -> {
                val view = LayoutInflater.from(parent.context).inflate(R.layout.item_header, parent, false)
                HeaderViewHolder(view)
            }
            TYPE_CONTENT -> {
                val view = LayoutInflater.from(parent.context).inflate(R.layout.item_content, parent, false)
                ContentViewHolder(view)
            }
            else -> throw IllegalArgumentException("Invalid view type")
        }
    }

    // ViewHolder에 데이터 바인딩
    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        when (holder) {
            is HeaderViewHolder -> holder.bind(items[position] as ListItem.Header)
            is ContentViewHolder -> holder.bind(items[position] as ListItem.Content)
        }
    }

    override fun getItemCount(): Int = items.size

    // 헤더 유형 ViewHolder
    class HeaderViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        private val title: TextView = itemView.findViewById(R.id.headerTitle)

        fun bind(item: ListItem.Header) {
            title.text = item.title
        }
    }

    // 콘텐츠 유형 ViewHolder
    class ContentViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        private val content: TextView = itemView.findViewById(R.id.contentText)

        fun bind(item: ListItem.Content) {
            content.text = item.text
        }
    }
}

// 다양한 아이템 유형을 나타내는 데이터 클래스 (Sealed Class 사용)
sealed class ListItem {
    data class Header(val title: String) : ListItem()
    data class Content(val text: String) : ListItem()
}

```

/
**💡 Pro Tips for Mastery: RecyclerView의 성능을 어떻게
향상시키나요?**
- ListAdapter와 DiffUtil을 활용하여 RecyclerView의 성능을 향상시킬 수 있습니다.

- DiffUtil은 
   - 두 목록 간의 차이를 계산하고 RecyclerView 어댑터를 
효율적으로 업데이트하는 안드로이드 유틸리티 클래스

   - 목록의 모든 항목을 비효율적으로 다시 렌더링할 수 있는 
notifyDataSetChanged()를 불필요하게 호출하는 것을 피할 수 있습니다. 


- DiffUtil 사용 단계
1. **DiffUtil 콜백 생성**:
DiffUtil.ItemCallback을 구현하거나 DiffUtil.Callback을 상속받습니다. 
(이전 목록과 새 목록 간의 차이 계산을 어떻게 할지 정의)

2. **어댑터에 목록 업데이트 제공**: 
(새 데이터가 도착하면 어댑터에 전달하고 
DiffUtil을 사용하여 차이를 계산합니다.)

3. **RecyclerView 어댑터와 DiffUtil 바인딩**: 
(DiffUtil을 어댑터에 통합하여 업데이트를 자동으로 처리합니다.)


---

## 🤔 Q42. Dp와 Sp의 차이점은 무엇인가요? 

### Dp (Density-independent Pixels)
- 패딩, 마진, 너비 등 UI 요소 측정 단위로
- 화면 밀도에 따라 자동 조절
- 1dp = 160 DPI(인치당 도트 수) 화면의 물리적 픽셀 1개 

### Sp (Scale-independent Pixels)
- 텍스트 크기 전용 단위
- 화면 밀도 + 사용자 글꼴 크기 환경설정 모두 고려
- 접근성 지원 (사용자 글꼴 크기 설정 반영)

### 주요 차이점
1. **목적**: Dp는 크기(버튼 크기, 패딩) / Sp는 텍스트 크기
2. **사용자 환경설정**: Sp는 글꼴 크기 설정 반영, Dp는 반영하지 않음
3. Sp는 접근성 지원, Dp는 일관된 레이아웃 유지


**💡 Pro Tips for Mastery: Sp 단위를 사용할 때 화면 깨짐을 어떻게
처리하나요?**

1. `wrap_content` 적절히 사용
2. `minLines`, `maxLines`, `ellipsize` 활용
3. 중요한 UI 컴포넌트는 고정 크기(dp) 고려
4. 극단적인 글꼴 크기로 테스트
5. ConstraintLayout으로 동적 크기 조절
6. Sp 대신 Dp 크기 사용



---

## 🤔 Q43. 나인패치(nine-patch) 이미지의 용도는 무엇인가요?

### 나인패치 이미지
- 시각적 품질 손실 없이 늘리거나 크기 조절 가능한 PNG 이미지
- 버튼, 배경, 컨테이너 등과 같은 다양한 화면 크기 및 콘텐츠 크기에 맞게 동적으로 크기 조절해야 하는 요소에 사용
- 파일명에 `.9.png` 포함 (android:background="@drawable/button_background.9.png" <)

### 주요 기능
1. **늘어나는 영역 정의(Stretchable Areas)**: 
이미지 가장자리 1픽셀 테두리의 검은색 선으로 정의

2. **콘텐츠 영역 정의(Content Area Definition)**: 
텍스트나 UI 요소의 적절한 정렬 보장

3. **동적 크기 조절(Dynamic Resizing)**: 다양한 화면 크기에서 모양 유지하도록 보장

### 한계
- 수동 생성 및 테스트 필요
- 직사각형/정사각형 형태에 가장 적합
- 복잡한 모양에는 덜 효과적

**Q) 나인패치 이미지는 일반 PNG 이미지와 어떻게 다르며, 
어떤 시나리오에서 나인패치 이미지를 사용해야 하나요?**

- 차이점
일반 PNG는 늘어나면 전체가 왜곡되지만, 
나인패치(.9.png)는 가장자리 검은 선으로 지정한 특정 영역만 늘어나서 
모서리와 테두리가 깨지지 않습니다.

- 사용 시나리오
버튼 배경, 채팅 말풍선, 다이얼로그 배경처럼 
텍스트 길이나 화면 크기에 따라 크기가 동적으로 변해야 하는 UI 요소에 사용합니다.
---

## 🤔 Q44. Drawable이란 무엇이며, UI 개발에서 어떻게 사용되나요?

### Drawable
- 화면에 그릴 수 있는 모든 것에 대한 추상화 개념
- 이미지, 벡터 그래픽, 모양 등 다양한 그래픽 콘텐츠의 기본 클래스 역할

### 주요 Drawable 타입

#### 1. BitmapDrawable (래스터 이미지)
- PNG, JPG, GIF 등 래스터 이미지를 표시하는데 사용
- 크기 조절, 타일링, 필터링 지원

```
<bitmap xmlns:android="http://schemas.android.com/apk/res/android"
    android:src="@drawable/sample_image"
    android:tileMode="repeat"/>

일반적으로 ImageView 컴포넌트에서 이미지 표시 또는 배경으로 사용됩니다.
```

#### 2. VectorDrawable (확장 가능한 벡터 그래픽)
- XML 경로 기반 확장 가능한 벡터 그래픽(SVG 유사)을 나타냄
- 아이콘, 로고 및 확장 가능한 UI 요소에 이상적
- 어떤 해상도에서도 품질 유지,다양한 화면 밀도에서 픽셀화 문제를 방지

```
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="#FF0000"
        android:pathData="M12,2L15,8H9L12,2Z"/> <!-- 예시 삼각형 경로 -->
</vector>

```

#### 3. NinePatchDrawable (패딩이 있는 크기 조절 가능 이미지)
- 특정 영역 보존하며 크기 조절 가능한 특수 유형의 비트맵
- 채팅 말풍선, 버튼 등의 늘어나는 UI 컴포넌트를 만드는데 유용

```
<nine-patch xmlns:android="http://schemas.android.com/apk/res/android"
    android:src="@drawable/chat_bubble.9.png"/>

```

#### 4. ShapeDrawable (커스텀 모양)
- XML 정의 커스텀 모양 생성에 사용
- 둥근 사각형, 타원 등의 버튼, 배경 및 커스텀 UI 컴포넌트에 유용

```
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <solid android:color="#FF5733"/>
    <corners android:radius="8dp"/>
</shape>

```

#### 5. LayerDrawable (여러 Drawable 쌓기)

- 여러 Drawable을 계층 구조로 결합하는데 사용
- 복잡한 UI 배경, 오버레이 효과 생성
