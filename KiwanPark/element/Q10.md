## ✅ Q10. BroadcastReceiver란 무엇인가요?

### 📌 개요
`BroadcastReceiver`는 시스템 또는 앱에서 발생한 **브로드캐스트 메시지**(`Intent`)를 수신해 처리하는 컴포넌트입니다. 이벤트 중심 구조로, 사용자나 시스템의 변화에 반응할 수 있게 해줍니다.

---

### 🎯 목적
- 시스템 이벤트 수신: 부팅 완료, 네트워크 변경, 배터리 부족 등
- 앱 간 이벤트 통신: 특정 이벤트를 다른 앱/컴포넌트에 전달

---

### 🧩 유형

| 유형         | 설명                           | 등록 위치               |
|--------------|--------------------------------|--------------------------|
| 정적 등록     | 앱이 실행 중이지 않아도 수신 가능  | `AndroidManifest.xml`    |
| 동적 등록     | 앱 실행 중일 때만 유효            | `registerReceiver()` 코드 |

---

### 📝 선언 (정적)
```xml
<receiver android:name=".MyBroadcastReceiver" android:exported="false">
    <intent-filter>
        <action android:name="android.intent.action.BATTERY_LOW" />
    </intent-filter>
</receiver>
```

---

### ⚙️ 등록 (동적)
**Dynamic Registration in Activity**  
Android 13(Tiramisu) 이상 대응

```kotlin
val receiver = MyBroadcastReceiver()
val intentFilter = IntentFilter(Intent.ACTION_BATTERY_LOW)

// Android Tiramisu (API 33) 이상에서는 RECEIVER_EXPORTED 또는 RECEIVER_NOT_EXPORTED 플래그 필요
val flags = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
    Context.RECEIVER_NOT_EXPORTED
} else {
    0 // 플래그 없음
}
registerReceiver(receiver, intentFilter, flags)

// Activity나 컴포넌트가 소멸될 때 등록 해제 필수
override fun onDestroy() {
    unregisterReceiver(receiver)
    super.onDestroy()
}
```

---

### ⚠️ 유의사항

#### 🔄 생명주기 관리
- 동적 등록 시 생명주기를 고려해 **`unregisterReceiver()`**를 적절히 호출해야 메모리 누수를 방지할 수 있습니다.

#### 🚫 백그라운드 실행 제한 (API 26+)
- Android 8.0 이상에서는 대부분의 **암시적 브로드캐스트가 제한**됩니다.
- 이 경우에는 `Context.registerReceiver()` 외에 `JobScheduler` 또는 `WorkManager`와 함께 사용해야 합니다.

#### 🔐 보안
- 민감한 정보가 포함된 브로드캐스트는 권한으로 보호해야 하며, `android:permission` 또는 `registerReceiver()`의 권한 인자를 사용해 무단 접근을 방지해야 합니다.

---

### 💡 사용 사례 (코드 포함)

#### 1. 네트워크 연결 변경 감지
```kotlin
class NetworkReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        if (ConnectivityManager.CONNECTIVITY_ACTION == intent.action) {
            Log.d("NetworkReceiver", "네트워크 상태가 변경되었습니다.")
        }
    }
}
```

#### 2. SMS 수신 감지
```kotlin
class SmsReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        val bundle = intent.extras
        val pdus = bundle?.get("pdus") as? Array<*>
        pdus?.forEach {
            val sms = SmsMessage.createFromPdu(it as ByteArray)
            Log.d("SmsReceiver", "SMS 내용: ${sms.displayMessageBody}")
        }
    }
}
```

#### 3. 충전 상태 감지
```kotlin
class PowerReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        when (intent.action) {
            Intent.ACTION_POWER_CONNECTED -> Log.d("PowerReceiver", "충전기 연결됨")
            Intent.ACTION_POWER_DISCONNECTED -> Log.d("PowerReceiver", "충전기 분리됨")
        }
    }
}
```

#### 4. 커스텀 브로드캐스트 전송 및 수신
```kotlin
// 송신
val intent = Intent("com.example.ACTION_CUSTOM")
intent.putExtra("data", "커스텀 메시지입니다.")
sendBroadcast(intent)

// 수신
class CustomReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        val message = intent.getStringExtra("data")
        Log.d("CustomReceiver", "수신한 메시지: $message")
    }
}
```

---

### 📚 요약

| 항목       | 요약 설명                                  |
|------------|---------------------------------------------|
| 정의       | 시스템/앱 이벤트에 반응하는 컴포넌트         |
| 용도       | 네트워크, 배터리, 커스텀 이벤트 수신용       |
| 등록       | 정적: Manifest / 동적: 코드 내 등록          |
| 제한       | Android 8+에서 암시적 브로드캐스트 제한 존재  |
| 주의사항   | 생명주기 해제 필수, 보안 설정 권장           |

---

### 💬 실전질문
### Q) 브로드캐스트의 유형에는 어떤 것이 있으며, 기능 및 사용 측면에서 시스템 브로드캐스트와 커스텀 브로드캐스트는 어떤 차이가 있나요?

#### 📌 1. 브로드캐스트의 분류

브로드캐스트는 **발신 주체**와 **등록 방식**에 따라 다음과 같이 나뉩니다:

| 분류 기준 | 유형 | 설명 |
|-----------|------|------|
| 발신 주체 기준 | **① 시스템 브로드캐스트** | OS 또는 시스템 앱에서 발생시킴 |
|               | **② 커스텀 브로드캐스트** | 개발자가 명시적으로 전송 |
| 등록 방식 기준 | **③ 정적 등록** | Manifest에 등록, 앱 꺼져 있어도 수신 가능 (제한 있음) |
|               | **④ 동적 등록** | 코드로 등록, 앱 실행 중에만 수신 가능 |

> 본 질문은 **① 시스템 브로드캐스트 vs ② 커스텀 브로드캐스트** 비교를 중심으로 설명합니다.

---

#### 📊 2. 시스템 vs 커스텀 브로드캐스트 비교

| 항목 | 시스템 브로드캐스트 | 커스텀 브로드캐스트 |
|------|----------------------|----------------------|
| **정의** | OS에서 자동 발생하는 이벤트 | 앱 내부/간 커뮤니케이션을 위한 개발자 정의 이벤트 |
| **예시** | `BOOT_COMPLETED`, `BATTERY_LOW`, `SMS_RECEIVED` | `"com.example.ACTION_CUSTOM"` |
| **등록 방식** | 주로 정적/동적 모두 가능 (단, API 26+는 제한) | 주로 동적 등록 |
| **보안 제어** | 일부는 시스템 권한 필요 | `android:permission` 등으로 보호 가능 |
| **수신 범위** | 모든 앱 또는 권한 허용된 앱 | 명시적으로 범위 제어 가능 |
| **제한 사항** | Android 8.0 이상에서 일부 브로드캐스트는 Manifest 등록 불가 | 시스템 제약 없음 (단, 퍼포먼스 주의) |
| **사용 목적** | 시스템 상태 감지 → 앱 대응 | 앱 내 이벤트 전파, 모듈 간 decoupling |

---

#### 🧠 3. 핵심 차이 설명

- **전송 주체의 차이**  
  시스템 브로드캐스트는 Android OS 또는 시스템 서비스가 전송합니다. 반면, 커스텀 브로드캐스트는 개발자가 앱 내부 로직에서 직접 `sendBroadcast()`로 전송합니다.

- **보안/권한 측면**  
  시스템 브로드캐스트는 일부 브로드캐스트 수신 시 시스템 권한 요구(`RECEIVE_SMS`, `READ_PHONE_STATE` 등), 커스텀 브로드캐스트는 `intent-filter`나 권한 선언을 통해 직접 제어 가능합니다.

- **사용 목적**  
  시스템: 외부 이벤트(네트워크, 충전 등)에 대응  
  커스텀: 앱 내부/모듈 간 decoupled 구조를 위한 이벤트 분산

---

#### 💡 4. 코드 예시

##### ✅ 시스템 브로드캐스트 수신 (예: 충전기 연결 감지)

```kotlin
class PowerReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        when (intent.action) {
            Intent.ACTION_POWER_CONNECTED -> Log.d("Receiver", "충전기 연결됨")
        }
    }
}

// 등록
val filter = IntentFilter(Intent.ACTION_POWER_CONNECTED)
registerReceiver(PowerReceiver(), filter)
```

---

##### ✅ 커스텀 브로드캐스트 송신 및 수신

**1) 전송**
```kotlin
val intent = Intent("com.example.ACTION_CUSTOM")
intent.putExtra("key", "value")
sendBroadcast(intent)
```

**2) 수신**
```kotlin
class CustomReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        val value = intent.getStringExtra("key")
        Log.d("CustomReceiver", "수신된 값: $value")
    }
}
val filter = IntentFilter("com.example.ACTION_CUSTOM")
registerReceiver(CustomReceiver(), filter)
```

---

#### 📝 요약

- **시스템 브로드캐스트**는 **OS에서 발생**하는 이벤트로, 앱은 이를 감지하여 동작합니다.
- **커스텀 브로드캐스트**는 **앱 내부나 앱 간 통신**을 위해 개발자가 정의하고 전송합니다.
- Android 8.0 이상에서는 **정적 브로드캐스트 등록이 제한**되므로, `registerReceiver()`와 백그라운드 제약을 함께 고려한 설계가 필요합니다.
