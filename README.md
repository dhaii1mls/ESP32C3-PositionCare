# PositionCare – ESP32-C3 Firmware

نظام مراقبة ضغط وحركة المريض (للوقاية من قرح الفراش). يقرأ الجهاز **حساس الضغط (FSR)** و**حساس الحركة (MPU6050)** ويرسل القراءات كل ثانية عبر **MQTT** إلى الموقع/لوحة المتابعة.

*An ESP32-C3 firmware that reads pressure (FSR) and motion (MPU6050) data and publishes it over MQTT once per second.*

---

## المميزات

- قراءة ضغط منطقة العجز (sacrum) من حساس FSR وتحويلها إلى نسبة مئوية ومستوى (0 – 3)
- اكتشاف الحركة القوية عبر MPU6050 (الكشف التلقائي للعنوان `0x68` أو `0x69`)
- الاستمرار بإرسال بيانات الضغط حتى لو لم يستجب حساس الحركة
- إرسال البيانات بصيغة JSON كل **1 ثانية**
- حالة اتصال الجهاز (Online/Offline) عبر MQTT Last Will
- استقبال الأوامر من الموقع (حاليًا يطبع رسالة فقط)

## العتاد المطلوب

| المكوّن | الوصف |
|---|---|
| ESP32-C3 | المتحكم الرئيسي |
| FSR | حساس ضغط (Force Sensitive Resistor) |
| MPU6050 | حساس تسارع/حركة (I²C) |

### التوصيل

| الحساس | دبوس ESP32-C3 |
|---|---|
| FSR (قراءة تناظرية) | GPIO 2 |
| MPU6050 SDA | GPIO 9 |
| MPU6050 SCL | GPIO 8 |
| MPU6050 VCC / GND | 3.3V / GND |

> القراءة معكوسة في الكود (`4095 - rawRead`)، أي أن التوصيل مبني على تقسيم جهد يعطي قيمة أقل عند زيادة الضغط. عدّلها إذا كان توصيلك مختلفًا.

## المكتبات المطلوبة

ثبّتها من Arduino IDE ← Library Manager:

- `PubSubClient`
- `ArduinoJson` (الإصدار 6)
- `WiFi` و `Wire` (مضمّنة مع حزمة ESP32)

## الإعداد والتشغيل

1. ثبّت حزمة **ESP32** في Arduino IDE واختر اللوحة **ESP32C3 Dev Module**.
2. افتح الملف وعدّل بيانات الواي فاي:
   ```cpp
   const char* WIFI_SSID     = "YOUR_WIFI_NAME";
   const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";
   ```
3. (اختياري) غيّر `DEVICE_ID` إلى معرّف فريد لجهازك.
4. ارفع الكود وافتح Serial Monitor على سرعة **115200**.

> ⚠️ لا ترفع بيانات الواي فاي الحقيقية على GitHub.

## MQTT

- **الخادم:** `broker.hivemq.com` – المنفذ `1883` (خادم عام للتجربة، يُنصح بخادم خاص عند الاستخدام الفعلي)

| Topic | الاتجاه | الوصف |
|---|---|---|
| `positioncare/v1/<DEVICE_ID>/status` | الجهاز ← الموقع | بيانات الحالة (JSON) كل ثانية |
| `positioncare/v1/<DEVICE_ID>/online` | الجهاز ← الموقع | `1` متصل / `0` غير متصل (Retained) |
| `positioncare/v1/<DEVICE_ID>/cmd` | الموقع ← الجهاز | أوامر التحكم |

### مثال على رسالة الحالة

```json
{
  "v": 1,
  "id": "pc-4821",
  "name": "Patient 01",
  "room": "101",
  "uptime": 120,
  "still": 10,
  "stable": 10,
  "stillReq": 120,
  "stableReq": 120,
  "moving": false,
  "motionOk": true,
  "loaded": true,
  "alert": false,
  "muted": false,
  "alerts": 0,
  "zones": [
    { "id": "sacrum", "pct": 42.5, "lvl": 1 }
  ]
}
```

### مستويات الضغط (`lvl`)

| المستوى | النسبة |
|---|---|
| 0 | أقل من 5% |
| 1 | من 5% إلى أقل من 50% |
| 2 | من 50% إلى أقل من 85% |
| 3 | 85% فأكثر |

### منطق الحساسات

- `loaded = true` عندما تتجاوز قراءة FSR القيمة `1000` (من 4095).
- `moving = true` عندما يتجاوز الفرق بين التسارع الكلي و1g القيمة `0.15`.

## ملاحظات

- معاملات مثل `still` و`stable` و`stillReq` و`stableReq` قيم ثابتة حاليًا في الكود.
- دالة `callback` تستقبل الأوامر لكنها لا تنفّذ شيئًا بعد.

## الترخيص

أضف الترخيص الذي تريده (مثل MIT).
