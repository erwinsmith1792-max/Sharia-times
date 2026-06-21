# ShariaAstro Android

مشروع أندرويد أولي لتطبيق الأوقات الشرعية والاقترانات القمرية-الشمسية.

## الحالة الحالية

- تم إنشاء مشروع Android/Kotlin متعدد الوحدات:
  - `:core`: نواة الحسابات.
  - `:app`: واجهة Android/Compose عربية RTL.
- تم نقل نواة الحسابات إلى `core`.
- تم إنشاء باحث عددي للأحداث الشمسية:
  - الشروق الشرعي.
  - الغروب الشرعي.
  - الظهر الشرعي عندما يكون مركز الشمس على `Azimuth=180°` فوق الأفق.
  - منتصف الليل الشرعي عندما يكون مركز الشمس على `Azimuth=180°` تحت الأفق.
  - دخول صلاة الظهر عندما تلامس حافة الشمس خط `Azimuth=180°` من جهة الغرب.
- تم إنشاء حاسب الاقترانات القمرية-الشمسية وفق تعريف القوس العمودي على مسار الشمس اليومي الظاهري.
- تم إنشاء واجهة `EphemerisProvider`.
- تم إضافة تنفيذ أولي: `OrekitEphemerisProvider` اعتماداً على Orekit + JPL ephemerides.
- تم إنشاء واجهة `GeoResolver` مع عميل Nominatim أولي وربطه بزر بحث عن الإحداثيات في الواجهة.
- تم تنفيذ `TimeZoneMapResolver` لتحديد IANA TimeZone تلقائياً من خط العرض/الطول offline اعتماداً على Timezone Boundary Builder عبر مكتبة `timezonemap`.
- تم تحسين عرض النتائج في جداول منظمة بدل نص طويل:
  - جدول الأحداث الشرعية الشمسية.
  - جدول الوقت الشرعي للتوقيت المُدخل.
  - جدول أوقات الصلاة.
  - جدول الاقترانات والمدد بينها.
- تم فصل منطق الواجهة والحساب عن `MainActivity` بإضافة `ShariaAstroViewModel` وملف نماذج نتائج `ResultModels.kt`.
- أضيفت إعدادات النموذج الظاهري في الواجهة:
  - ارتفاع الموقع بالمتر.
  - درجة الحرارة.
  - الضغط الجوي.
  - تفعيل/تعطيل الانكسار الجوي القياسي.
- أضيف نظام تصدير النتائج ومشاركتها:
  - تصدير نصي TXT.
  - تصدير CSV للجداول.
  - تصدير JSON منظم.
  - مشاركة الملف عبر Android share sheet باستخدام FileProvider.
- أضيف سجل محلي للنتائج:
  - حفظ النتيجة الحالية.
  - عرض قائمة النتائج المحفوظة.
  - فتح نتيجة محفوظة وإعادة عرضها في لوحة النتائج.
  - حذف عنصر واحد أو مسح السجل كاملاً.
  - التخزين محلياً عبر SharedPreferences بصيغة JSON.
- أضيفت ميزة استيراد بيانات Orekit/JPL من الهاتف:
  - زر `استيراد orekit-data.zip` داخل التطبيق.
  - نسخ الملف إلى تخزين التطبيق الداخلي.
  - إعطاء الأولوية للملف المستورد على الملف المضمّن في assets.
  - إمكانية حذف الملف المستورد.
- بدأت المرحلة الخاصة بالاختبارات بإضافة اختبارات وحدة أولية لـ `ShariaTimeEngine` في `core/src/test`.
- تم إنشاء واجهة `TimeZoneResolver` لربط Timezone Boundary Builder أو خدمة timezone.

## البنية

```text
sharia_android/
  settings.gradle.kts
  build.gradle.kts
  app/
    build.gradle.kts
    src/main/...
  core/
    build.gradle.kts
    src/main/kotlin/ai/arena/shariaastro/
      ShariaTimeEngine.kt
      SolarAzimuthEventFinder.kt
      LegalSolarDayCalculator.kt
      LunarSolarConjunctionCalculator.kt
      GeoResolver.kt
```

## ما يلزم لإكمال النسخة العملية

1. تزويد التطبيق بملف بيانات Orekit:
   - ضع ملفاً باسم `app/src/main/assets/orekit-data.zip`.
   - يجب أن يحتوي على بيانات Orekit الأساسية وملف JPL DE ephemeris ثنائي مثل DE430/DE440 بصيغة يتعرف عليها Orekit.
   - الواجهة الحالية تتحقق من وجود الملف وتستخدم `OrekitEphemerisProvider` إذا وجد.
2. تحسين `TimeZoneResolver`:
   - التنفيذ الحالي offline عبر `TimeZoneMapResolver`.
   - يمكن لاحقاً إضافة cache دائم serialized أو مخدم وسيط لتقليل وقت أول بحث.
3. ربط ViewModel في تطبيق Android:
   - البحث عن المنطقة.
   - تحديد timezone.
   - حساب المراسي الشمسية.
   - حساب الوقت الشرعي والصلوات.
   - حساب الاقترانات السنوية.
4. إضافة اختبارات مقارنة بعد توفير ephemeris.

## تصدير النتائج

تظهر أزرار التصدير أسفل لوحة النتائج:

- `نص`: ملف TXT مقروء.
- `CSV`: مناسب للجداول وبرامج الجداول الحسابية.
- `JSON`: مناسب للأرشفة أو التكامل البرمجي.

ينشئ التطبيق الملف في cache داخلي ثم يفتحه عبر Android share sheet باستعمال `FileProvider`.

## استيراد بيانات Orekit/JPL من الهاتف

توجد بطاقة في أعلى التطبيق بعنوان `بيانات Orekit/JPL`، وفيها:

- حالة ملف البيانات الحالي.
- زر `استيراد orekit-data.zip` لاختيار الملف من ذاكرة الهاتف.
- زر `حذف الملف المستورد` للرجوع إلى الملف المضمّن داخل التطبيق إن وجد.

عند الاستيراد ينسخ التطبيق الملف إلى تخزينه الداخلي باسم `imported-orekit-data.zip`. لذلك لا يحتاج التطبيق إلى صلاحيات تخزين دائمة، ولا يتأثر إذا نقل المستخدم الملف الأصلي لاحقاً.

## السجل المحلي

تظهر لوحة `السجل المحلي للنتائج` أسفل النتائج. يستطيع المستخدم:

- حفظ النتيجة الحالية في السجل.
- فتح نتيجة محفوظة وإعادة عرضها.
- حذف نتيجة محددة.
- مسح السجل كاملاً.

يُخزن السجل محلياً في `SharedPreferences` باسم `sharia_astro_history`، وتُحفظ كل نتيجة كبنية JSON قابلة للاسترجاع.

## الاختبارات

أضيفت اختبارات وحدة أولية في:

```text
core/src/test/kotlin/ai/arena/shariaastro/
```

وتغطي حالياً:

- `ResultExportersTest` في وحدة `app`: تصدير النتائج إلى TXT/CSV/JSON.
- `UiResultJsonCodecTest` في وحدة `app`: ترميز/فك ترميز النتائج وعناصر السجل المحلي.
- `ShariaTimeEngineTest`: منطق الساعات الشرعية وأوقات الصلاة على يوم اصطناعي.
- `SolarAzimuthEventFinderTest`: الشروق/الغروب/Azimuth=180/دخول صلاة الظهر على نموذج شمسي اصطناعي.
- `LegalSolarDayCalculatorTest`: بناء مراسي اليوم الشرعي من الأحداث الفلكية.
- `TimeZoneAndDstTest`: التوقيت الصيفي عبر `java.time` وتحديد المنطقة الزمنية من الإحداثيات.
- `LunarSolarConjunctionCalculatorTest`: اختبار اصطناعي لحاسب الاقترانات والمدد بينها.

تشغيل الاختبارات من Android Studio أو:

```bash
./gradlew :core:test
```

## تشغيل المشروع

افتح المجلد `sharia_android` في Android Studio ثم نفّذ Gradle sync.

> ملاحظة: إذا ظهرت تحديثات لإصدارات Android Gradle Plugin أو Kotlin أو Compose، يمكن تحديثها من ملفي `build.gradle.kts`.
