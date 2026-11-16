# الدليل النهائي الشامل لتعديل MiuiCamera.apk 📱
### حل مشكلة الكاميرا الأمامية وانهيار النظام (SIGSEGV Chi Override Crash)

هذا الدليل تم إعداده ليكون شاملاً وسريعاً للتطبيق المباشر عبر برنامج **MT Manager** على الهاتف، حيث يمكنك نسخ الأكواد ولصقها مباشرة دون الحاجة لكتابة أي سطر يدوي.

---

## ⚡ الخطوة التمهيدية: فتح الـ APK في محرر MT Manager
1. افتح تطبيق **MT Manager** وتوجه إلى ملف `MiuiCamera.apk`.
2. اضغط على ملف التطبيق ثم اختر **View (عرض)**.
3. اضغط على أي ملف باسم `classes.dex` (مثلاً `classes.dex`).
4. اختر **Dex Editor Plus (محرر Dex بلس)**.
5. **مهم جداً:** ضع علامة صح (✔) أمام **جميع ملفات classes.dex** (من classes.dex إلى classes7.dex) ثم اضغط **OK**.

---

## 📋 جدول التعديلات السريعة (5 تعديلات حاسمة)

| # | اسم الكلاس / الملف | دالة التعديل | أرقام الأسطر في الملف الأصلي | الهدف من التعديل |
|---|---|---|---|---|
| **1** | `d.c.b.p4` (classes2.dex) | `private J1()V` | **من السطر 4519 إلى 4615** | حل كراش النظام الكلي `chi_override_flush SIGSEGV` |
| **2** | `com.mi.device.DataItemFeature` (classes7.dex) | `public I4()Z` | **من السطر 3294 إلى 3304** | تعطيل طلب تفريغ الكاميرا `isSupportAbortCaptures` |
| **3** | `d.c.a.n6.a.b.a` (classes.dex) | `public static c(...)V` | **من السطر 99 إلى 220** | تشغيل الكاميرا الأمامية وإلغاء الـ Streams المرفوضة |
| **4** | `d.c.a.o3` (classes.dex) | `public a()Z` | **من السطر 1367 إلى 1612** | تخطي فحص اللانشر ليعمل التطبيق مع أي لانشر |
| **5** | `d.c.a.a6.f.k` (classes.dex) | `public static a()I` | **من السطر 37 إلى 70** | منع كراش الشاشات المطوية `SettingNotFoundException` |

---

## 1️⃣ التعديل الأول: إيقاف كراش النظام القاتل (`chi_override_flush SIGSEGV`)

> **المشكلة:** عند التبديل للكاميرا الأمامية، يرسل التطبيق أمر `abortCaptures()`، مما يجعل درايفر الكاميرا الخاص بكوالكوم في النظام (`com.qti.chi.override.so`) ينهار ويوقف خدمة الكاميرا بالكامل على روم RisingOS.

### 📍 طريقة البحث:
- اضغط على أيقونة البحث 🔍 في MT Manager.
- اكتب في خانة البحث: `abortCaptures E`
- اختر نوع البحث: **Text**.
- اضغط **OK**، ثم افتح الكلاس الناتج: `d.c.b.p4` (أو `MiCamera2`).

### ✂️ الاستبدال (من السطر 4519 إلى 4615):
اذهب للسطر 4519 أو ابحث عن الدالة:
```smali
.method private J1()V
```
حدد الدالة بالكامل من `.method private J1()V` حتى `.end method` واستبدلها بالكود التالي:

```smali
.method private J1()V
    .registers 1

    return-void
.end method
```

---

## 2️⃣ التعديل الثاني: تعطيل خاصية `isSupportAbortCaptures` على مستوى الجهاز

### 📍 طريقة البحث:
- اضغط على أيقونة البحث 🔍.
- اكتب في خانة البحث: `com.mi.device.DataItemFeature`
- اختر نوع البحث: **Class name**.
- افتح الكلاس الناتج: `com.mi.device.DataItemFeature`.

### ✂️ الاستبدال (من السطر 3294 إلى 3304):
اذهب للسطر 3294 أو ابحث عن الدالة:
```smali
.method public I4()Z
```
حدد الدالة بالكامل من `.method public I4()Z` حتى `.end method` واستبدلها بالكود التالي:

```smali
.method public I4()Z
    .registers 1

    const/4 p0, 0x0

    return p0
.end method
```

---

## 3️⃣ التعديل الثالث: تشغيل الكاميرا الأمامية وقبول التدفقات القياسية (`createCaptureSession`)

> **المشكلة:** الكاميرا الأمامية تطلب `InputConfiguration` وإعدادات `SessionParameters` خاصة بخوارزميات شاومي ترفضها رومات AOSP مما يسبب `Unsupported set of inputs/outputs provided`. هذا التعديل يجعل الكاميرا الأمامية (ID = 1) تعمل بالوضع القياسي النظيف.

### 📍 طريقة البحث:
- اضغط على أيقونة البحث 🔍.
- اكتب في خانة البحث: `3:cameraOpened2createCaptureSession@`
- اختر نوع البحث: **Text**.
- افتح الكلاس الناتج: `d.c.a.n6.a.b.a` (أو `CompatibilityUtils`).

### ✂️ الاستبدال (من السطر 99 إلى 220):
اذهب للسطر 99 أو ابحث عن الدالة:
```smali
.method public static c(Landroid/hardware/camera2/CameraDevice;ILandroid/hardware/camera2/params/InputConfiguration;Ljava/util/List;Landroid/hardware/camera2/CaptureRequest;Landroid/hardware/camera2/CameraCaptureSession$StateCallback;Landroid/os/Handler;)V
```
حدد الدالة بالكامل حتى `.end method` واستبدلها بالكود التالي:

```smali
.method public static c(Landroid/hardware/camera2/CameraDevice;ILandroid/hardware/camera2/params/InputConfiguration;Ljava/util/List;Landroid/hardware/camera2/CaptureRequest;Landroid/hardware/camera2/CameraCaptureSession$StateCallback;Landroid/os/Handler;)V
    .registers 10

    invoke-virtual {p0}, Landroid/hardware/camera2/CameraDevice;->getId()Ljava/lang/String;
    move-result-object v0
    const-string v1, "1"
    invoke-virtual {v1, v0}, Ljava/lang/String;->equals(Ljava/lang/Object;)Z
    move-result v0
    if-eqz v0, :cond_front_ok
    const/4 p1, 0x0
    const/4 p2, 0x0
    const/4 p4, 0x0
    :cond_front_ok

    new-instance v0, Landroid/hardware/camera2/params/SessionConfiguration;
    if-nez p6, :cond_handler
    const/4 p6, 0x0
    goto :goto_cfg
    :cond_handler
    new-instance v1, Ld/c/a/n6/a/b/a$a;
    invoke-direct {v1, p6}, Ld/c/a/n6/a/b/a$a;-><init>(Landroid/os/Handler;)V
    move-object p6, v1
    :goto_cfg
    invoke-direct {v0, p1, p3, p6, p5}, Landroid/hardware/camera2/params/SessionConfiguration;-><init>(ILjava/util/List;Ljava/util/concurrent/Executor;Landroid/hardware/camera2/CameraCaptureSession$StateCallback;)V
    if-eqz p2, :cond_skip_input
    invoke-virtual {v0, p2}, Landroid/hardware/camera2/params/SessionConfiguration;->setInputConfiguration(Landroid/hardware/camera2/params/InputConfiguration;)V
    :cond_skip_input
    if-eqz p4, :cond_skip_params
    invoke-virtual {v0, p4}, Landroid/hardware/camera2/params/SessionConfiguration;->setSessionParameters(Landroid/hardware/camera2/CaptureRequest;)V
    :cond_skip_params
    invoke-virtual {p0, v0}, Landroid/hardware/camera2/CameraDevice;->createCaptureSession(Landroid/hardware/camera2/params/SessionConfiguration;)V
    return-void
.end method
```

---

## 4️⃣ التعديل الرابع: تجاوز فحص اللانشر (`Unknown caller`)

> **المشكلة:** يظهر في اللوج: `checkCallerLegality: Unknown caller: com.google.android.apps.nexuslauncher`. هذا التعديل يسمح بفتح التطبيق من أي لانشر دون قيود.

### 📍 طريقة البحث:
- اضغط على أيقونة البحث 🔍.
- اكتب في خانة البحث: `checkCallerLegality: Unknown caller:`
- اختر نوع البحث: **Text**.
- افتح الكلاس الناتج: `d.c.a.o3` (أو `CameraIntentManager`).

### ✂️ الاستبدال (من السطر 1367 إلى 1612):
اذهب للسطر 1367 أو ابحث عن الدالة:
```smali
.method public a()Z
```
حدد الدالة بالكامل من `.method public a()Z` حتى `.end method` واستبدلها بالكود التالي:

```smali
.method public a()Z
    .registers 2

    const/4 v0, 0x1

    return v0
.end method
```

---

## 5️⃣ التعديل الخامس: إيقاف كراش إعدادات الشاشة (`device_posture`)

> **المشكلة:** يظهر في اللوج: `android.provider.Settings$SettingNotFoundException: device_posture`. هذا التعديل يمنع البحث عن إعداد وضعية الطي في الهاتف العادي.

### 📍 طريقة البحث:
- اضغط على أيقونة البحث 🔍.
- اكتب في خانة البحث: `device_posture`
- اختر نوع البحث: **Text**.
- افتح الكلاس الناتج: `d.c.a.a6.f.k` (أو `FoldHelper`).

### ✂️ الاستبدال (من السطر 37 إلى 70):
اذهب للسطر 37 أو ابحث عن الدالة:
```smali
.method public static a()I
```
حدد الدالة بالكامل من `.method public static a()I` حتى `.end method` واستبدلها بالكود التالي:

```smali
.method public static a()I
    .registers 1

    const/4 v0, 0x0

    return v0
.end method
```

---

## 💾 الحفظ والتثبيت (الخطوات النهائية):
1. بعد الانتهاء من التعديلات الخمسة، اضغط على أيقونة **الحفظ (Save)** في أعلى الشاشة.
2. اضغط زر **رجوع (Back)** للخروج من محرر الأكواد.
3. ستظهر نافذة تطلب منك تأكيد الحفظ والخروج، اضغط على **Save and exit**.
4. تأكد تماماً من وضع علامة صح على خيار **Auto sign (توقيع تلقائي)**.
5. اضغط **OK** وانتظر حتى يقوم MT Manager بحفظ وتوقيع ملف الـ APK.
6. قم بتثبيت التطبيق على هاتفك وجرب الكاميرا الأمامية وستعمل معك بسلاسة وبدون أي انهيار للنظام!
