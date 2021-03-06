# كيف تضيف خطوطاً جديدة إلى فيدورا

<img src='https://fedoramagazine.org/wp-content/uploads/2017/11/install-fonts-945x400.jpg'/>

تساعدك الخطوط (fonts) على التعبير عن أفكارك بطريقة إبداعية بواسطة تصميمات. تستطيع الخطوط نقل قكرتك إلى مستوى آخر سواءً كنت تشرح صورة، أو تصنع عرض تقديمي، أو تصمم بطاقة معايدة أو إعلاناً. من السهل الوقوع في حبهم نظراً لطبيعتهم الجميلة. لحسن الحظ، تجعل فيدورا عملية تثبيت الخطوط سهلة. دعنا نتعلم ذلك.

## التثبيت في فضاء النظام

عند تثبيتك لخط ما في فضاء النظام (system-wide) يصبح متاحاً لجميع المستخدمين. أفضل طريقة لفعل ذلك هي استخدام حزم RPM من مستودعات البرمجيات الرسمية.

قم بفتح *متجر البرمجيات* (Software) في توزيعة فيدورا محطة العمل خاصتك، أو أداة أخرى تستخدم المستودعات الرسمية. اختر تصنيف *الإضافات* (Add-ons) في مجموعة اللافتات المعروضة. ومن ثم اختر الخطوط. سترى قائمة من الخطوط المتوافرة مشابهة للصورة أدناه:

<img src='https://fedoramagazine.org/wp-content/uploads/2017/11/Software-fonts-768x576.png'/>

عندما تختار الخط المراد تحميله، ستظهر لك بعض التفاصيل عنه. بناءً على عدة أمور، قد تكون قادراً على رؤية شكل الخط عبر بعض الأمثلة النصيّة. انقر على زر *التثبيت* (install) لإضافة الخط للنظام. قد تستغرق عملية التثبيت بضعة لحظات لتكتمل، يعتمد هذا على سرعة النظام والشبكة لديك.

يمكنك أيضاً إزالة حزم الخطوط المُثبتة مسبقاً على نظامك (تكون هذه الخطوط مُعلمة بإشارة صح). ادخل إلى الخط المراد إزالته وانقر على زر *الإزالة* (remove) الموجود في منطقة تفاصيل الخط.

## التثبيت الشخصي (في فضاء المستخدم)

تعمل هذه الطريقة بشكل أفضل إذا كنت تملك خط قمت بتنزيله يملك صيغة متوافقة (مثل ttf .otf .ttc .pfa .pfb .pcf.). لا يجب تثبيت الملقحات الخطيّة هذه في فضاء النظام عبر وضعهم في مجلد النظام. الخطوط الغير مُحزمة من هذا النوع لا يمكن تحديثها تلقائياً. كما يمكن أن تتداخل أيضاً ببعض عمليات البرمجيات الأخرى لاحقاً. أفضل طريقة لتثبيت هذه الخطوط هي بوضعها في دليل المستخدم الحالي.

افتح مدير الملفات (التطبيق *Files*) في توزيعة فيدورا محطة العمل، أو أي مدير ملفات آخر من اختيارك. إذا كنت تستخدم مدير الملفات الإفتراضي (التطبيق *Files*)، فيمكنك الضغط على الإختصار *Ctrl+H* لعرض الملفات والمجلدات المخفيّة. ابحث عن المجلد *fonts.* وافتحه. إذا لم تجد هذا المجلد، قم بإنشاءه (تذكر أن تضع نقطة في بداية اسمه وأن تستخدم جميع الحروف بالحالة الصغيرة).

انسخ ملف الخط الذي نزلته وضعه في المجلد *fonts.* يمكنك إغلاق مدير الملفات الآن. افتح الطرفية واكتب الأمر التالي:

```
fc-cache
```

هذا الأمر سيقوم بإعادة بناء قاعدة الخطوط التي تساعد فيدورا على تحديد وتعليم الخطوط التي يمكن استخدامها. قد تحتاج أيضاً إلى إعادة تشغيل تطبيقات مثل إنكسكيب أو المكتب الحر (LibreOffice) إن أردت استخدام الخطوط الجديدة فيها. حالما تعيد تشغيل هذه التطبيقات، ينبغي أن تكون الخطوط الجديدة متاحة للاستخدام.

المصدر: [https://fedoramagazine.org/add-fonts-fedora](https://fedoramagazine.org/add-fonts-fedora)
