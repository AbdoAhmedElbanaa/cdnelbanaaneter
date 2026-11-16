# دليل شامل لتعديل MiuiCamera.apk بواسطة MT Manager 📱

هذا الدليل يحتوي على جميع الأكواد الجاهزة للنسخ والاستبدال المباشر في برنامج **MT Manager** لحل مشكلة الكاميرا الأمامية، مشكلة كراش النظام الكلي (SIGSEGV chi_override_flush)، وجميع أخطاء اللوج.

---

## 📌 خطوات فتح الـ APK في MT Manager
1. افتح تطبيق **MT Manager** على الهاتف.
2. اضغط على ملف `MiuiCamera.apk` ثم اضغط على **View (عرض)**.
3. اضغط على أي ملف `classes.dex` (مثلاً `classes.dex`).
4. اختر **Dex Editor Plus (محرر Dex بلس)**.
5. ضع علامة صح على جميع ملفات الـ DEX (من classes.dex إلى classes7.dex) ثم اضغط **OK**.

---

## 1️⃣ التعديل الأول: منع كراش النظام القاتل (حل خطأ `chi_override_flush` SIGSEGV)

> **سبب المشكلة:** الكاميرا ترسل أمر `abortCaptures()` عند التبديل للكاميرا الأمامية، مما يؤدي لانهيار درايفر الكوالكوم `/vendor/lib64/hw/com.qti.chi.override.so` في كود الروم (RisingOS) وخروج إشارة `SIGSEGV` وموت خدمة الكاميرا بالكامل.

### 📍 مكان التعديل:
- اضغط على أيقونة البحث 🔍 واختر **Search (بحث)**.
- ابحث بنص: `abortCaptures E`
- نوع البحث: **Text**.
- سيظهر لك كلاس وحيد: `d.c.b.p4` (أو `MiCamera2.java`).

### ✂️ ما تفعله داخل الكلاس:
ابحث داخل الكلاس عن الدالة:
```smali
.method private J1()V
```
حدد الدالة بالكامل من `.method private J1()V` حتى نهايتها `.end method` واستبدلها بالكود التالي:

### 📋 كود الاستبدال (Copy & Paste):
```smali
.method private J1()V
    .registers 1

    return-void
.end method
```

---

## 2️⃣ التعديل الثاني: تعطيل خاصية `isSupportAbortCaptures` في إعدادات الأجهزة

### 📍 مكان التعديل:
- في شريط البحث 🔍، ابحث عن الكلاس: `com.mi.device.DataItemFeature`
- نوع البحث: **Class name**.
- افتح الكلاس الناتج `com.mi.device.DataItemFeature`.

### ✂️ ما تفعله داخل الكلاس:
ابحث داخل الكلاس عن الدالة:
```smali
.method public I4()Z
```
حدد الدالة بالكامل من `.method public I4()Z` حتى `.end method` واستبدلها بالكود التالي:

### 📋 كود الاستبدال (Copy & Paste):
```smali
.method public I4()Z
    .registers 1

    const/4 p0, 0x0

    return p0
.end method
```

---

## 3️⃣ التعديل الثالث (الأساسي): حل رفض الكاميرا الأمامية `createCaptureSession`

> **سبب المشكلة:** الكاميرا الأمامية في هواتف شاومي تطلب `InputConfiguration` وخصائص معالجة إضافية يرفضها نظام AOSP (مما يسبب خطأ `Unsupported set of inputs/outputs provided`).

### 📍 مكان التعديل:
- في شريط البحث 🔍، ابحث بنص: `3:cameraOpened2createCaptureSession@`
- نوع البحث: **Text**.
- افتح الكلاس الناتج: `d.c.a.n6.a.b.a` (أو `CompatibilityUtils.java`).

### ✂️ ما تفعله داخل الكلاس:
ابحث داخل الكلاس عن الدالة:
```smali
.method public static c(Landroid/hardware/camera2/CameraDevice;ILandroid/hardware/camera2/params/InputConfiguration;Ljava/util/List;Landroid/hardware/camera2/CaptureRequest;Landroid/hardware/camera2/CameraCaptureSession$StateCallback;Landroid/os/Handler;)V
```
حدد الدالة بالكامل من `.method public static c` حتى نهايتها `.end method` واستبدلها بالكود التالي:

### 📋 كود الاستبدال (Copy & Paste):
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

## 4️⃣ التعديل الرابع: تجاوز فحص اللانشر `Unknown caller`

### 📍 مكان التعديل:
- في شريط البحث 🔍، ابحث بنص: `checkCallerLegality: Unknown caller:`
- نوع البحث: **Text**.
- افتح الكلاس الناتج: `d.c.a.o3` (أو `CameraIntentManager.java`).

### ✂️ ما تفعله داخل الكلاس:
ابحث داخل الكلاس عن الدالة:
```smali
.method public a()Z
```
حدد الدالة بالكامل من `.method public a()Z` حتى `.end method` واستبدلها بالكود التالي:

### 📋 كود الاستبدال (Copy & Paste):
```smali
.method public a()Z
    .registers 2

    const/4 v0, 0x1

    return v0
.end method
```

---

## 5️⃣ التعديل الخامس: إيقاف كراش الشاشة القابلة للطي `device_posture`

### 📍 مكان التعديل:
- في شريط البحث 🔍، ابحث بنص: `device_posture`
- نوع البحث: **Text**.
- افتح الكلاس الناتج: `d.c.a.a6.f.k` (أو `FoldHelper.java`).

### ✂️ ما تفعله داخل الكلاس:
ابحث داخل الكلاس عن الدالة:
```smali
.method public static a()I
```
حدد الدالة بالكامل من `.method public static a()I` حتى `.end method` واستبدلها بالكود التالي:

### 📋 كود الاستبدال (Copy & Paste):
```smali
.method public static a()I
    .registers 1

    const/4 v0, 0x0

    return v0
.end method
```

---

## 💾 الحفظ والتثبيت (الخطوة الأخيرة)
1. اضغط على أيقونة **الحفظ (Save)** أعلى الشاشة داخل المحرر.
2. اضغط زر **رجوع (Back)**.
3. ستظهر رسالة تطلب حفظ التغييرات والخروج، اضغط **Save and exit**.
4. تأكد من تحديد خيار **Auto sign (توقيع تلقائي)** واضغط **OK** لتحديث وحفظ الـ APK.
5. قم بتثبيت ملف الـ APK وافتح الكاميرا وبدّل للأمامية!
