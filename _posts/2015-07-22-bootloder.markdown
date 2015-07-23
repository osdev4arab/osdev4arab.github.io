---
title:  "محمل الإقلاع"
date:   2015-07-22 10:01:00
categories: osdev
tags: x86 8086 8088 80186 80286 80386 pentium real-mode protected-mode ibm pc
direction: rtl
comments: true
---

<center>
>
سلسلة مقدمة إلى تطوير نظم التشغيل
==================================
الرحلة الأولى: محمل الإقلاع
=========================
</center>

مرحباً بك في سلسة مقدمة إلى تطوير نظم التشغيل. في هذه السلسة سنتعلم معا أساسيات 
تطوير نظم تشغيل الكمبيوتر، وسنقوم ببناء نظام تشغيل صغير من البداية للكمبيوتر 
الشخصي.

نظام التشغيل هو برنامج الكمبيوتر المسؤول عن إدارة البرامج المختلفة، وإدارة 
النبيطة (العتاد)، وتوفير الواجهة بين النبيطة والبرمجيات. من أمثلة نظم التشغيل 
المشهورة نظام جنو لينوكس، نظام النوافذ، وعائلة نظم يونكس.

في أغلب الأحوال، أي شخص يستخدم الكمبيوتر فلابد أن يكون قد احتك بإحدى أنظمة 
الشتغيل المذكورة أعلاه. قد يشعر  المستخدم بأن نظام التشغيل هو أمر غامض، وذلك 
يدفعه إلى الفضول: كيف يتم عمل مثل تلك الأشياء؟ ومن هم المسؤولون عن تطوير نظم 
التشغيل تلك؟ وما الذي يتم بالظبط منذ أن أقوم بالضغط على زر التشغيل حتى نصل إلى 
سطح المكتب؟ وفي معظم الأحيان يكون هذا الفضول هو الدافع وراء تعلم تطوير نظم 
التشغيل وكتابة نظام تشغيل صغير كهواية. هدفنا في هذه السلسة هو توفير المعلومات 
اللازمة لإشباع هذه الرغبة.

المتطلبات القبلية:
--------------------

من أجل الاستمرار في هذه السلسلة، من الأفضل أن تكون على معرفة كافية ببرمجة 
الكمبيوتر. عملية تطوير نظام تشغيل هي في الواقع مشابهة لعملية تطوير أي برمجية 
أخرى. نحن هنا نستخدم نفس الأدوات التي يستخدمها نفس المبرمجين لتطوير برامجهم. قد 
يكون الفارق الوحيد هو أننا نقوم بكتابة نظام تشغيل. سنقوم بالتحدث عن الأدوات 
المستخدمة في الأجزاء اللاحقة.

من المهم أيضا أن تكون أتممت التمرينات الموجودة في سلسلة مقدمة إلى لغة التجميع 
للمعالج إنتل ٨٠٨٦. سنستخدم نفس المعالج في هذه السلسلة لتطوير نظام التشغيل 
التعليمي الذي سنقوم بكتابته سوياً.

ستحتاج أيضاً لنظام التشغيل GNU/Linux من أجل كتابة البرامج وترجمتها. يجب أن يحتوي 
النظام على binutils (للمجمع والرابط) وgcc (مترجم لغة سي) وmake (من أجل كتابة 
Makefiles). معظم التوزيعات تكون عليها تلك الأدوات جاهزة. ستحتاج إلى qemu من أجل 
محاكاة البرامج وقد نحتاج إلى بعض الأدوات الأخرى مثل hexedit.

جدول المحتويات:
-----------------

1. <a href="#intro">مدخل إلى الكمبيوتر الشخصي IBM</a>.
2. <a href="#firstprog">البرنامج الأول</a>.
3. <a href="#helloworld">محمل إقلاع "Hello World"</a>.

<a id="intro"></a>

١) مدخل إلى الكمبيوتر الشخصي IBM PC:
======================================

دأب الإنسان في صناعة أدوات تساعده في أداء العمليات الحسابية منذ ما قبل الميلاد. 
وكان حلم تشارلس باباج (١٧٩١-١٨٧١) هو بناء آلة ميكانيكية تقوم بمعالجة الحسابات 
بشكل أوتوماتيكي. سنستخدم مصطلح "الكمبيوتر الحديث" من أجل الإشارة لأجهزة الحاسوب 
التي بين أيدينا اليوم. يعتبر الكمبيوتر هارفارد مارك والكمبيوتر ENIAC من أوائل 
الحواسيب الحديثة.

وفي عام ١٩٤٥ قام العالم جون فون نويمان وآخرون بصياغة First Draft of a Report on 
the EDVAC، وفيه قام فون نويمان باقتراح معمارية للحاسب الآلي يكون البرنامج فيها 
مخزن في ذاكرة الحاسب (مبدأ البرنامج المخزن). في واقع الأمر فإن الكثير من 
الحواسيب اليوم هي في الأصل قائمة على عمارة فون نويمان وعلى مبدأ البرنامج المخزن. 
يوضح الشكل رقم ١ شكل العمارة التي اقترحها فون نويمان:


<p style="text-align:center">
<image src="/images/osdev/bootloader/vonneumann.png" />
<br />
شكل ١ – عمارة فون نويمان
</p>

من الشكل أعلاه يمكننا استنتاج أن الكمبيوتر القائم على عمارة فون نويمان يحتوي على 
المكونات الآتية:

1. وحدة التحكم: وهي المسئوولة عن جلب التعليمات والبيانات من الذاكرة وإرسال 
المعلومات إليها. تكون التعليمات مخزنة في الذاكرة مثلها مثل البيانات (على شكل 
أرقام). وتقوم وحدة التحكم بجلب التعليمات وفك تشفيرها بشكل آلي.
2. وحدة الحساب والمنطق: وهي الوحدة المسؤولة عن تنفيذ التعليمات. أما المركم 
(Accumulator) فهو مسجل (وحدة تخزين) يقوم بتخزين نتائج التعليمات أو المعلومات 
الأخرى بشكل مؤقت قبل إرسالها إلى مكان آخر أو قبل معالجتها.
3. وحدات الإدخال والإخراج: في الكمبيوتر الحديث تقوم وحدة التحكم بمواجهة وحدات 
الإدخال والإخراج بنفس الطريقة التي تواجه الذاكرة بها، وسنرى أمثلة لذلك لاحقاً. 
من أمثلة أجهزة الإدخال والإخراج هي وحدات العرض، الطرفيات، الطابعات، وماكينات 
الكتابة.
4. الذاكرة: وهي وحدة التخزين المسؤولة عن تخزين البرامج والبيانات.

وحدة التحكم ووحدة الحساب والمنطق معاً يمثلان ما نسميه بـ "المعالج". قد يكون في 
المعالج أكتر من مركم، وقد يكون في المعالج مسجلات أخرى غير المركم. يتصل المعالج 
بالذاكرة ووحدات الإدخال والإخراج عن طريق الناقل العام (System bus)، كما هو موضح 
بالشكل ٢:

<p style="text-align:center">
<image src="/images/osdev/bootloader/sysbus.png" />
<br />
شكل ٢ – الناقل العام
</p>

يتصل المعالج والذاكرة ووحدات الإدخال والإخراج ببعضهم البعض من خلال الناقل العام. 
عندما يقوم المعالج بجلب البيانات (أو التعليمات) من الذاكرة، يقوم بوضع عنوان 
البايت (أو الكلمة) المطلوبة في ناقل العناوين Address bus. ويقوم بإرسال إشارة 
"أنا أريد القراءة" من خلال ناقل التحكم Control bus. فتقوم الذاكرة في الحال بوضع 
البايت المطلوب في ناقل البيانات Data bus. تتم عملية الكتابة بنفس الأسلوب، الفارق 
هو أن المعالج هو الذي يقوم بوضع البيانات على ناقل البيانات، وتقوم الذاكرة 
بقراءتها وتخزينها.

يتم الاتصال بين المعالج ووحدات الإدخال والإخراج بنفس الطريقة. تظهر وحدات الإدخال 
والإخراج كأنها مجموعة من المسجلات، وكل مسجل له عنوان فريد. القراءة من هذه 
المسجلات تعطي حالة الجهاز، والكتابة على هذه المسجلات تحدد حالة الجهاز. فمثلا من 
المتوقع أن لوحة المفاتيح keyboard تقوم بتوفير مسجل، بقراءته يمكن معرفة آخر زر 
قام المستخدم بالضغط عليه. ومن المتوقع أن تقوم وحدة العرض بتوفير مسجلات، عند 
الكتابة عليها يتم إظهار أشكال معينة على الشاشة حسبما تم كتابته في المسجلات.

يجب أن تكون العناوين الخاصة بالمسجلات الخاصة بوحدات الإدخال والإخراج مختلفة عن 
العناوين الخاصة بكلمات الذاكرة. حيز العناوين هو مصطلح يشير إلى مجموعة العناوين 
التي يستطيع ناقل العناوين نقلها تبعا لسعته. فمثلا إذا كانت سعة ناقل العناوين 
تسمح له بنقل عناوين من صفر إلى ٩٩ فقط، فإن حيز العناوين هنا يبدأ عند الصفر 
وينتهي عند ٩٩ (١٠٠ عنوان فريد). يقوم مصمم الكمبيوتر (بالتحديد، من يقوم بتصميم 
العتاد) بتوزيع تلك العناوين بين الذاكرة ووحدات الإدخال والإخراج المختلفة. عندما 
يقوم المعالج بوضع عنوان معين على ناقل العناوين، يقوم كل جهاز بمقارنة هذا العنوان 
بمجموعة العناوين الخاصة به، والجهاز صاحب العنوان الموضوع هو فقط الذي يقوم بالرد، 
أما باقي الأجهزة فتقوم بتجاهل العملية لأنها لا تخصها.

تطور الكمبيوتر الرقمي:
-----------------------

مر الكمبيوتر الحديث بمراحل تطوير كثيرة منذ الأربعينات وحتى اليوم. ومازال 
الكمبيوتر قيد التطوير حتى اللحظة. الجدير بالذكر أنه في الأربعينات بدأت الحواسب 
في الانتقال من "حواسب إلكتروميكانيكية" إلى حواسب "كهربية". كانت الحواسب 
الالكتروميكانيكية في العادة تستخدم المرحل relay، أما الحواسيب الكهربية فكانت 
تستخدم الصمامات المفرغة Vacuum Tubes.

وكان هناك طريقتان لتمثيل (تشفير) البيانات في الحواسيب الكهربية: في الحواسيب 
الأنالوجية (التناظرية) يتم تمثيل البيانات بالتناظر مع فرق الجهد بين وحدة التمثيل 
والأرضي. أما في الحواسيب الرقمية (ديجيتال) يتم تخزين البيانات بالطريقة الرقمية. 
يمكننا القول بأنه منذ النصف الثاني من الأربعينيات بدأ مصممو الكمبيوتر في الاتجاه 
إلى الحواسيب الإلكترونية الرقمية والتي تستخدم نظام العد الثنائي لتمثيل البيانات 
الرقمية.

وباختراع المقحل (الترانزستور) عام ١٩٤٧ والذي استبدل الصمامات المفرغة، أصبحت الحواسب أصغر 
حجما وأقل تكلفة وأفضل من جهة الاعتمادية. من الحواسيب المشهورة في الستينيات 
الحاسوب IBM System/360 والحاسوب العلمي IBM 1620 الذي انتشر في الجامعات بشكل 
كبير.

أدى اختراع الدارات المدمجة إلى إمكانية تصنيع دوائر كاملة في نبائط صغيرة الحجم 
تسمى رقائق chips أو microchips واستخدامها لبناء أجهزة الكمبيوتر والأجهزة الأخرى. 
في النصف الثاني من الستينيات بدأت الكمبيوترات الصغيرة minicomputers في الظهور. 
ظهرت الحاجة إلى الـ minicomputer وذلك لإن الكمبيوتر وقتها كان يحتاج إلى تجهيزات 
وتكاليف باهظة. في العادة يستخدم الناس مصطلح mainframe (كمبيوتر مركزي) للإشارة 
إلى الكمبيوتر الكبير، ومصطلح الـ minicomputer للإشارة إلى الكمبيوتر الصغير.

من أمثلة الكمبيوترات الصغيرة هي الكمبيوتر PDP-7 و الكمبيوتر PDP-11 من DEC، وهما 
أول حواسب تم تطوير وتشغيل نظام التشغيل يونكس عليها في بداية السبعينيات.

أدى التطور السريع في صناعة الدارات المدمجة إلى إمكانية تصنيع معالج بالكامل في 
رقاقة microchip واحدة. المعالج الموضوع على microchip واحدة يسمى microprocessor 
(معالج صغري أو معالج دقيق).أول معالج صغري عرفه كوكب الأرض كان المعالج ٤٠٠٤ من إنتل.

وفي واقع الأمر، أحدث المعالج الصغري ثورة هائلة في السبعينيات، وأصبح الناس 
يتطلعون إلى ذلك اليوم حينما يكون هناك كمبيوتر في كل منزل. أدى المعالج الصغري إلى 
ظهور الميكروكمبيوتر Microcomputers وهي كمبيوترات أصغر من الكمبيوترات الصغيرة 
minicomputers وتكون في الأغلب قائمة على المعالج الصغري. من أمثلة الكمبيوترات 
الميكرووية في السبعينيات الكمبيوتر Altair من MITS، والكمبيوتر Apple II الذي كان 
قائما على المعالج ٦٥٠٢ من MOS. وبسبب انتشار صناعة الكمبيوترات الميكرووية، أدركت 
IBM أخيرا في بداية الثمانينات أهميته، وبدأت في عام ١٩٨١ خط انتاج جديد وهو 
الكمبيوتر الشخصي IBM PC.

<p style="text-align:center">
<image src="/images/osdev/bootloader/ibmpc.jpg" />
<br />
شكل ٣ – الكمبيوتر الشخصي IBM PC/XT
</p>

المواصفات التقنية للكمبيوتر IBM PC:
------------------------------------

- المعالج: استخدمت IBM المعالج ٨٠٨٨ من إنتل، وهو معالج مماثل تماما لـ ٨٠٨٦، إلا أن 
حجم ناقل البيانات فيه ٨ بت بدلا من ١٦. أي أن الكمبيوتر يمكنه ناقل بايت واحد كل 
مرة. لكن ما زال حجم الكلمة داخل المعالج ١٦ بت، أي أن المعالج يمكنه معالجة ٢ بايت 
في نفس الوقت. حجم ناقل العناوين ٢٠ بت، أي أن هناك ١ ميغا عنوان للذاكرة يمكن 
استخدامها، وبالتالي فإن أقصى حجم للذاكرة هو ١ ميغابايت وكان وقتها هذا الرقم كبير 
للغاية بالنسبة للميكروكمبيوتر.

- الذاكرة: أصدرت IBM اصدارات مختلفة من IBM PC تحتوي على ذواكر بحجم ١٦ كيلوبايت أو 
٦٤ كيلوبايت. واستخدمات رقاقات DRAM من أجل توفير هذا الحجم من الذاكرة.

- وحدات الإدخال والإخراج: احتوى الكمبيوتر على مجموعة من الوحدات المختلفة، مثل 
بطاقة العرض وذاكرةROM لتخزين بايوس (سيتم الحديث عنهم في الفصول المقبلة)، ووحدة 
للتحكم في محركات الأقراص المرنة Floppy Disk Controller، ووحدة للتحكم في لوحة 
المفاتيح Keyboard controller، ووحدات أخرى سيتم التحدث عنها في المقالات الآتية.

<p style="text-align:center">
<image src="/images/osdev/bootloader/org.png" />
<br />
شكل ٤ – التصميم العام لنظام الكمبيوتر الشخصي
</p>

<p style="text-align:center">
<image src="/images/osdev/bootloader/mainboard.png" />
<br />
شكل ٥ – اللوحة الأم للكمبيوتر الشخصي. هل يمكنك التعرف على المكونات الموجودة في 
شكل ٤؟
</p>

شهد الحاسوب الشخصي انتشارا كبيرا، وباتت IBM في تطوير نماذج أحدث من الكمبيوتر IBM 
PC. في عام ١٩٨٣ قامت IBM بإصدار الكمبيوتر IBM PC/XT والذي تميز بحجم ذاكرة أكبر 
يصل إلى ٦٤٠ كيلو بايت ومحرك قرص صلب داخلي. وفي عام ١٩٨٤ قامت IBM بإصدار 
الكمبيوتر IBM PC/AT حيث تم استبدال المعالج ٨٠٨٨ بالمعالج ٨٠٢٨٦، وأصبح من الممكن 
زيادة حجم الذاكرة إلى ١٦ ميغابايت. 

ومنذ بداية ظهور الكمبيوتر الشخصي IBM PC بدأت الشركات الأخرى في تصنيع حواسيب 
متوافقة مع الكمبيوتر IBM PC وهي ما تسمى PC clones. وبعد أن تركت IBM خط إنتاج 
الكمبيوتر الشخصي أصبحت تلك الحواسيب المتوافقة مع الكمبيوتر الشخصي IBM PC 
compatibles هي السائدة. الحواسيب الشخصية الموجودة اليوم هي في الحقيقة IBM PC 
compatibles. وبالتالي فإنها متوافقة بشكل كبير مع مواصفات الحواسيب الشخصية 
القديمة PC/XT وPC/AT.

وفي الواقع هذا هو سبب اختيارنا للكمبيوتر الشخصي IBM PC، لأن الكمبيوترات الشخصية 
الموجودة اليوم والتي تصنعها شركات مختلفة هي في واقع الأمر IBM PC compatibles، 
وهي الحواسيب الشخصية الأكثر انتشارا اليوم.

تستخدم الحواسيب الشخصية اليوم معالجات أكثر تقدما من ٨٠٨٨ و٨٠٢٨٦، وفي الأغلب فإن 
جميعها متوافقة مع المعالج ٨٠٣٨٦، وهو معالج ٣٢-بت وحجم الذاكرة فيه قد يصل إلى ٤ 
غيغا بايت (لأن ناقل العناوين حجمه ٣٢ بت). المعالج بنتيوم من أبرز المعالجات 
الموجودة اليوم، وهو متوافق مع المعالج ٨٠٣٨٦.

يدعم المعالج ٨٠٣٨٦ وضعان أساسيان للتشغيل: الوضع الحقيقي والوضع المحمي. في الوضع 
الحقيقي يكون ٨٠٣٨٦ متوافقا مع المعالج ٨٠٨٨ ذي ١٦-بت. أما في الوضع المحمي يعمل 
المعالج ٨٠٣٨٦ في وضع ٣٢-بت ويدعم إمكانيات أكثر تقدما مثل ترحيل الصفحات paging.

عند بدء تشغيل الكمبيوتر يبدأ المعالج ٨٠٣٨٦ في الوضع الحقيقي أي وضع ٨٠٨٨، ثم 
يقوم نظام التشغيل لاحقا بالتحول إلى الوضع المحمي. ولأن الكمبيوتر في الوضع 
الحقيقي يكون شبيها بشكل كبير بالكمبيوتر IBM PC/XT، فسنتحدث في هذه المقالة عن 
الكمبيوتر IBM PC/XT بشكل أساسي، وأنت تعلم جيدا عزيزي القارئ أن ما سيتم ذكره هنا 
يمكن تطبيقه بشكل مباشر على الكمبيوتر الشخصي الموجود في منزلك الآن أو في محل 
عملك. أي أنه يمكنك أن تتجاهل الآن الإمكانيات الجبارة الموجودة في حاسوبك، وتعتبره 
IBM PC/XT مؤقتاً حتى ننتقل لاحقا إلى ما هو أحدث في المقالات القادمة.

نظام العنونة في الكمبيوتر الشخصي IBM PC/XT:
--------------------------------------------

ذكرنا مسبقا أن حجم ناقل العناوين في الذي يسمح به المعالج ٨٠٨٨ هو ٢٠ بت، وبالتالي 
لدينا ١ ميغا عنوان نريد توزيعهم على الذاكرةRAM والوحدات الأخرى الموجودة شكل ٤.

إلا أنه في حقيقة الأمر، المعالج ٨٠٨٦ والمعالج ٨٠٨٨ يسمجان لوحدات الإدخال 
والأخراج بأن يتم عنونتها في حيز عناوين مختلف، يسمى حيز عناوين الإدخال والإخراج 
I/O address space.

أي أن هناك حيزان من العناوين في المعالج ٨٠٨٨، حيز عناوين للذاكرة memory address 
space وحجمه كما ذكرنا مسبقا ١ ميغا عنوان (من 0x00000 إلى 0xFFFFF)، وحيز عناوين 
الإدخال والإخراج I/O address space وحجمه ٦٤ كيلو عنوان (١٦ بت) (من 0x0000 إلى 
0xFFFF).

عندما يقوم المعالج بوضع عنوان ما على ناقل العناوين، يجب عليه تحديد ما إذا كان 
هذا العنوان تابعاً لحيز عناوين الذاكرة أم أنه تابعا لحيز عناوين الإدخال 
والإخراج. إذا كان العنوان الموضوع هو عنوان إدخال وإخراج، فإن العنوان يشغل فقط 
الـ ١٦ بت الأقل قيمة مكانية في ناقل العناوين، أما إذا كان عنوان ذاكرة، فإنه يشغل 
مجمل العشرين بت. ببساطة يتم تحديد نوع العنوان باستخدام ناقل التحكم.

ويحتوي الكمبيوتر الشخصي IBM PC/XT على أجهزة decoder خاصة تقوم بقراءة العنوان من 
ناقل العناوين وقراءة نوع العنوان من ناقل التحكم، ثم إرسال إشارة Chip Select إلى 
الجهاز الذي سيقوم بالرد. يوضح شكل ٦ خريطة عناوين الذاكرة في الكمبيوتر الشخصي IBM 
PC/XT.

<center>
<table border="2" width="500px">

<thead>
<tr>
<th style="text-align:center;">البداية</th>
<th style="text-align:center;">النهاية</th>
<th style="text-align:center;">الاستخدام</th>
</tr>
</thead>

<tbody>
<tr>
<td style="text-align:center;">0x00000</td>
<td style="text-align:center;">0x9FFFF</td>
<td style="text-align:center;">الذاكرة RAM (٦٤٠ كيلو بايت)</td>
</tr>

<tr>
<td style="text-align:center;">0xB8000</td>
<td style="text-align:center;">0xBFFFF</td>
<td style="text-align:center;">ذاكرة العرض Video RAM</td>
</tr>

<tr>
<td style="text-align:center;">0xFE000</td>
<td style="text-align:center;">0xFFFFF</td>
<td style="text-align:center;">الذاكرة ROM الخاصة بالبايوس</td>
</tr>
</tbody>

</table>
</center>

<center>
شكل ٦ – الأجزاء الأساسية من خريطة عناوين الذاكرة في الكمبيوتر الشخصي IBM PC/XT
</center>

نظام الإدخال والإخراج الأساسي BIOS:
---------------------------------

نظام الإدخال والإخراج الأساسي BIOS هو برنامج مثبت على ذاكرة ROM يعمل كـ firmware 
للكمبيوتر الشخصي. يظهر البرنامج في حيز العناوين من 0xFE000 إلى 0xFFFFF. عند 
تشغيل الكمبيوتر، يقوم المعالج ٨٠٨٨ بعمل RESET، ويبدأ تنفيذ التعليمات بدءاً من 
العنوان 0xFFFF0، وبالتالي فإن أول تعليمة يقوم المعالج ٨٠٨٨ بجلبها هي تعليمة من 
برنامج البايوس. وبالنظر إلى الشيفرة المصدرية الخاصة بالبايوس للكمبيوتر XT نجد أن 
العنوان 0xFFFF0 يحتوي على تعليمة JMP إلى 0xFE05B :

<pre>
<code>JMP 0xF000:0xE05B</code>
</pre>

وانطلاقا من هذا العنوان تبدأ البايوس في تشغيل الكمبيوتر وتجهيز النبيطة للعمل. 
وبعد إجراء اختبار بدء التشغيل الذاتي Power-on self-test أو ما نسميه POST، تقوم 
البايوس بعد ذلك بقراءة أول قطاع sector من القرص المرن أو الصلب... وهذا هو قطاع 
الإقلاع الذي سنقوم بكتابته اليوم.

حجم القطاع في الأقراص المرنة والأقراص الصلبة التي تعمل مع الكمبيوتر الشخصي ٥١٢ 
بايت. قطاع الأقلاع هو أول قطاع في القرص الذي سيقوم الكمبيوتر بالإقلاع منه. في 
الأغلب يكون نظام التشغيل مثبت على قرص صلب، وتقوم البايوس بتحميل قطاع الإقلاع من 
القرص الصلب تمهيدا لتحميل نظام التشغيل.

قطاع الإقلاع يحتوي على برنامج يسمى برنامج الإقلاع أو محمل الإقلاع. وظيفة برنامج 
الإقلاع هي تحميل نظام التشغيل من القرص إلى الذاكرة. لكل نظام تشغيل محمل إقلاع 
خاص به. في جنو-لينوكس محملات الإقلاع LILO وGRUB وSYSLINUX هي اﻷكثر انتشارا. كان 
للنظام MS-DOS محمل الإقلاع الخاص به والذي يقوم بتحميل IO.SYS إلى الذاكرة.

تقوم البايوس بتحميل قطاع الإقلاع إلى العنوان 0x7C00 (أنظر إلى خريطة الذاكرة 
أعلاه، إن هذا العنوان هو تابع لحيز العناوين الخاص بالذاكرة RAM)، ثم تقوم البايوس 
بعمل JMP إلى هذا العنوان ليبدأ محمل الإقلاع في التنفيذ.

------------------------------------------------------------------------------------

<a id="firstprog"></a>

٢) البرنامج الأول:
==================

ستتعلم في هذا الفصل كيف تكتب أول قطاع إقلاع خاص بك، وهو قطاع إقلاع بسيط يقوم 
بتحميل رقم مميز 0x1234 إلى المسجل AX ثم يتوقف باستخدام التعليمة HLT:

<pre>
<code>MOV AX, 0x1234   # load 0x1234 into AX.
CLI              # disable Interrupts.
HLT              # halt the processor.</code>
</pre>

التعليمةCLI تقوم بتعطيل المقاطعات. سنتحدث عن طلبات المقاطعة بالتفصيل في مقال 
لاحق. أما التعليمةHLT فهي تقوم بتوقيف المعالج حتى يأتي طلب مقاطعة. ولأن طلبات 
المقاطعة قد تم تعطيلها، فإن التعليمة HLT هنا ستتسبب في توقف المعالج نهائياً.

سنستخدم المجمع الخاص بجنو GNU assembler من أجل تجميع هذا البرنامج وتحويله إلى 
لغة الآلة. ولكن نحتاج لإضافة بعض السطور في بداية البرنامج حتى يقوم المجمع بترجمة 
البرنامج بشكل صحيح. تسمى تلك الأوامر بالتوجيهات directives، لأن الهدف منها هو 
توجيه المجمع للعمل بشكل معين، ولا يتم ترجمتها لتعليمات لغة الآلة لأنها لا تمثل 
جزءا من الكود الذي سيتم تنفيذه على المعالج.

<pre>
<code>.intel_syntax noprefix
.code16
.text</code>
</pre>

السطر الأول .intel_syntax noprefix يحدد الـ syntax الذي نستخدمه في كتابة 
البرنامج. هناك أكثر من syntax للغة التجميع الخاصة بمعالجات intel. الـ syntax 
الذي نستخدمه هنا هو نفسه المستخدم في المستندات الخاصة بإنتل. أما .code16 فكأننا 
نقول للمجمع أن هذا البرنامج هو برنامج ١٦-بت، أي أنه برنامج للمعالج ٨٠٨٨ (أو 
٨٠٣٨٦ عندما يعمل في الوضع الحقيقي كما ذكرنا سابقا). السطر الأخير يوضح للمترجم أن 
ما سيتم كتابته لاحقا هو متن البرنامج text، وليس بيانات data.

عند تحميل قطاع الإقلاع إلى العنوان 0x7C00، تقوم البايوس بفحص آخر ٢ بايت في 
القطاع، أي البايت رقم ٥١٠ ورقم ٥١١. يجب أن يحتويا على 0x55 و0xAA بالترتيب. تسمى 
تلك الأرقام بـ BIOS Boot Signature، ويجب أن يحتوي القطاع عليهما في آخر ٢ بايت 
وإلا فإنه يعتبر قطاع تالف وسترفض البايوس الإقلاع منه.

وبالتالي نحتاج الآن لكي نقول للمجمع أن البرنامج يجب أن يخرج في ملف حجمه ٥١٢ بايت 
(حجم القطاع)، وأن آخر ٢ بايت هما 0x55 و0xAA. يمكن استخدام التوجيه .org (يعد 
السطر الخاص بالتعليمة HLT) لعمل ذلك:

<pre>
<code>.org 510, 0x00   # move to 510 and fill the spaces with 0x00.
.word 0xAA55     # BIOS boot signature.</code>
</pre>

وهذا يعني أن المجمع سيقوم بملأ البايتات بعد التعليمة HLT بالرقم 0x00، حتى يصل 
عدد البايتات الإجمالية إلى ٥١٠، ثم بعد ذلك يقوم بوضع الكلمة 0xAA55. وبالتالي 
يصبح المجموع ٥١٢.

هذا هو شكل البرنامج كاملاً:

<pre>
<code>.intel_syntax noprefix
.code16
.text

MOV AX, 0x1234   # load 0x1234 into AX.
CLI              # disable Interrupts.
HLT              # halt the processor.

.org 510, 0x00   # move to 510 and fill the spaces with 0x00.
.word 0xAA55     # BIOS boot signature.</code>
</pre>

قم بحفظ البرنامج في ملف نصي باسم bootloader.s. ثم افتح الطرفية وانتقل إلى الدليل 
الموجود فيه bootloader.s. يستحسن إنشاء دليل جديد ووضع bootloader.s فيه. من 
الطرفية قم بكتابة الأمر التالي:

<pre><code>$ as -o bootloader.o bootloader.s</code></pre>

يقوم ذلك الأمر بتجميع البرنامج bootloader.s وإخراج الـ object file المسمى 
bootloader.o. الخطوة التالية هي إدخال bootloader.o إلى الرابط linker. الرابط هو 
عبارة عن برنامج يقوم بتجميع مخرجات المترجم والمكتبات وما إلى ذلك وربطهم سويا. في 
حالتنا هذه سنطلب من الرابط أيضاً إخراج الملف المخرج كـbinary file خالص بدلاً من 
تنسيق ELF، ويتم ذلك بكتابة هذا الأمر في الطرفية:

<pre><code>$ ld --oformat=binary -o bootloader.bin -e 0 bootloader.o</code></pre>

يقوم هذا الأمر بتشغيل الرابط، والذي سيقوم بربط الملف bootloader.o وتحويله من 
تنسيق ELF إلى تنسيق binary. أما -e فهي تستخدم لتعريف الرابط بعنوان أول تعليمة 
سيتم تنفيذها داخل الملف (كإزاحة بالنسبة لبداية الملف). الآن يمكنك استخدام ls من 
أجل رؤية التطورات:

<pre><code>$ ls -l</code></pre>

<p style="text-align:center">
<image src="/images/osdev/bootloader/shot1.png" />
</p>

قام الرابط بإخراج bootloader.bin، وحجمه ٥١٢ بايت كما وصفنا للأسمبلر. نريد الآن 
رؤية محتويات هذا الملف، يمكننا استخدام الأمر hexedit من أجل ذلك:

<pre><code>$ hexedit bootloader.bin</code></pre>

<p style="text-align:center">
<image src="/images/osdev/bootloader/shot2.png" />
</p>

الأرقام الموجودة في أول ٥ بايت هما في الحقيقة كود الهدف أو الأوبكود opcode الذي 
أخرجه المعالج assembler للتعليمات MOV وCLI وHLT. الرقم 0xB8 في أول بايت هو 
الأوبكود الخاص بالتعليمة MOV AX, some value، ويتبعه الـ value التي نريد وضعها في 
AX، وهي 0x1234 حيث يكون 0x34 هو البايت الأول (الأقل قيمة مكانية) والبايت 0x12 هو 
البايت الثاني (اﻷعلى قيمة مكانية). أما 0xFA فهي التعليمة CLI، والرقم 0xF4 هي 
التعليمة HLT. بعد ذلك قام الأسمبلر بملء الملف بالأصفار 0x00 حتى البايت 0x1FE 
(٥١٠) حيث قام بوضع 0x55 في البايت 0x1FE و0xAA في البايت 0x1FF وهو البايت الأخير.

لقد أصبح لدينا الآن محمل إقلاع جاهز. الخطوة التالية هي محاكاة قطاع الإقلاع 
باستخدام برامج محاكاة الكمبيوتر الشخصي. سنستخدم هنا البرنامج qemu. يمكنك تشغيل 
qemu بهذا الأمر البسيط:

<pre><code>$ qemu-system-i386 -hda bootloader.bin</code></pre>

يقوم هذا الأمر بتشغيل qemu وقراءة محمل الإقلاع من bootloader.bin. في الحقيقة فإن 
-hda تستخدم من أجل تحديد صورة قرص صلب hard disk image كي يستخدمها qemu كقرص صلب 
للماكينة الوهمية virtual machine. في حالتنا هذه يمكن اعتبار bootloader.bin بإنه 
قرص صلب يحتوي على قطاع واحد فقط، وهو قطاع الإقلاع. بعد تشغيل qemu تظهر الشاشة 
الآتية:

<p style="text-align:center">
<image src="/images/osdev/bootloader/shot3.png" />
</p>

لقد قامت البايوس من الإقلاع من الهارد ديسك، فتم تحميل البرنامج الخاص بنا إلى 
العنوان 0x7C00 وتشغيله، فقام البرنامج بوضع 0x1234 في AX ثم قام بعد ذلك بتوقيف 
المعالج تماما. نريد الآن أن نشاهد محتويات المسجل AX. قم بالضغط على Alt+Ctrl+2 
لفتح Qemu Console، ثم اكتب الأمر info registers. يقوم qemu بطباعة كل المسجلات 
على الشاشة. قم بالضغط على Ctrl+Up لتمرير الشاشة إلى الأعلى، حتى تتمكن من رؤية 
محتويات AX.

<p style="text-align:center">
<image src="/images/osdev/bootloader/shot4.png" />
</p>

مبروك. لقد انتهيت الآن من أول محمل إقلاع في سلسلتنا.

-------------------------------------------------------------------------------

<a id="helloworld"></a>

٣) محمل إقلاع Hello World!:
===========================

تعلمنا في الفصل السابق كيف نكتب محمل إقلاع للكمبيوتر الشخصي. الخطوة التالية هي 
جعل محمل الإقلاع يقوم بكتابة Hello World على شاشة العرض. 

في الكمبيوتر IBM PC/XT يمكن لبطاقة العرض CGA أو (Color graphics adapter) أن تعمل 
في عدد مختلف من اﻷوضاع. عند بدء الكمبيوتر تكون بطاقة العرض في وضع النصوص text 
mode، أي انها تعمل بشكل أساسي لإخراج نصوص على الشاشة. الوضع البديل هو وضع الرسوم 
graphics mode والذي تقوم فيه بإخراج رسومات بدلاً من نصوص. عند بدء الكمبيوتر تقوم 
البايوس بإجراء تمهيد initialization لبطاقة العرض CGA وتحويلها إلى وضع النصوص. 

في وضع النصوص تنقسم الشاشة إلى ٨٠ عموداً و٢٥ صفاً. أي ان عدد الخلايا يساوي ٢٠٠٠ 
خلية، وكل خلية تمثل محرفاً. الهدف هو كتابة !Hello World في الصف الأول، بدءاً من 
العمود الأول وانتهاءاً عند العمود الثاني عشر:

<div dir="ltr">
<center>
<table border="2" width="800px" style="font-size:large; font-family: courier">

<tr>
<th style="text-align:center"></td>
<th style="text-align:center">0</td>
<th style="text-align:center">1</td>
<th style="text-align:center">2</td>
<th style="text-align:center">3</td>
<th style="text-align:center">4</td>
<th style="text-align:center">5</td>
<th style="text-align:center">6</td>
<th style="text-align:center">7</td>
<th style="text-align:center">8</td>
<th style="text-align:center">9</td>
<th style="text-align:center">10</td>
<th style="text-align:center">11</td>
<th style="text-align:center">12</td>
<th style="text-align:center">...</td>
<th style="text-align:center">79</td>
</tr>

<tr>
<th style="text-align:center;">0</td>
<th style="text-align:center; background-color: blue; color: white">H</td>
<th style="text-align:center; background-color: blue; color: white">e</td>
<th style="text-align:center; background-color: blue; color: white">l</td>
<th style="text-align:center; background-color: blue; color: white">l</td>
<th style="text-align:center; background-color: blue; color: white">o</td>
<th style="text-align:center; background-color: blue; color: white"></td>
<th style="text-align:center; background-color: blue; color: white">W</td>
<th style="text-align:center; background-color: blue; color: white">o</td>
<th style="text-align:center; background-color: blue; color: white">r</td>
<th style="text-align:center; background-color: blue; color: white">l</td>
<th style="text-align:center; background-color: blue; color: white">d</td>
<th style="text-align:center; background-color: blue; color: white">!</td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
</tr>

<tr>
<th style="text-align:center;">1</td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
</tr>

<tr>
<th style="text-align:center;">.<br/>.<br/>.</td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
</tr>

<tr>
<th style="text-align:center;">24</td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
<th style="text-align:center; background-color: black"></td>
</tr>

</table>
</center>
<br/>
</div>

ذكرنا مسبقاً أن ذاكرة العرض Video RAM تظهر من العنوان 0xB8000 إلى العنوان 
0xBFFFF. كل محرف على الشاشة يتم تمثيله بـ ٢ بايت، بدءاً من 0xB8000. أي أن الكلمة 
عند 0xB8000-0xB8001 هي تمثيل للمحرف الأول في الصف الأول، والكلمة عند 
0xB8002-0xB8003 هي تمثل للمحرف الثاني في الصف الأول، وهكذا. وبالتالي مجموع 
البايتات المستخدم لوصف الشاشة هما ٤٠٠٠ بايت، بدءاً من 0xB8000.

كل محرف يتمثل بـ ٢ بايت، البايت الأول هو الـ ASCII code الخاص بالمحرف، والبايت 
التاني هو صفة المحرف attribute (أي لونه). يصف البايت التاني لون المحرف ولون 
الخلفية التي وراءه. يتم تمثيل لون المحرف بـ ٤ بايت، ولون الخلفية بـ ٣ بايت، أما 
البايت المتبقى فيستخدم لتفعيل الوميض blinking.

ولأن لون المحرف يتم تمثيله بـ ٤ بايت، فإن عدد الألوان الممكنة هو ١٦ لون، ولذلك 
يسمى هذا الوضع (وضع ١٦ لون). ترقم الألوان من صفر إلى ١٥. لون المحرف ممكن أن يكون 
واحدا من الألوان الموجودة في الجدول أدناه. أما لون الخلفية يمكن فقط أن يكون واحد 
من أول ثمانية ألوان في الجدول.


<div dir="ltr">
<center>
<table border="2" width="800px" style="font-size:large; font-family: courier">

<tr>
<th style="text-align:center;">Color</th>
<th style="text-align:center;">Index</th>
<th style="text-align:center;">Color</th>
<th style="text-align:center;">Index</th>
<th style="text-align:center;">Color</th>
<th style="text-align:center;">Index</th>
<th style="text-align:center;">Color</th>
<th style="text-align:center;">Index</th>
</tr>

<tr>
<th style="text-align:center;">0</th>
<th style="text-align:center;background-color: #000000; color: white">Black</th>
<th style="text-align:center;">4</th>
<th style="text-align:center;background-color: #AA0000; color: white">Red</th>
<th style="text-align:center;">8</th>
<th style="text-align:center;background-color: #555555; color: white">Gray</th>
<th style="text-align:center;">12</th>
<th style="text-align:center;background-color: #FF5555; color: white">Light<br/>Red</th>
</tr>

<tr>
<th style="text-align:center;">1</th>
<th style="text-align:center;background-color: #0000AA; color: white">Blue</th>
<th style="text-align:center;">5</th>
<th style="text-align:center;background-color: #AA00AA; color: white">Magenta</th>
<th style="text-align:center;">9</th>
<th style="text-align:center;background-color: #5555FF; color: white">Light<br/>Blue</th>
<th style="text-align:center;">13</th>
<th style="text-align:center;background-color: #FF55FF; color: white">Light<br/>Magenta</th>
</tr>

<tr>
<th style="text-align:center;">2</th>
<th style="text-align:center;background-color: #00AA00; color: white">Green</th>
<th style="text-align:center;">6</th>
<th style="text-align:center;background-color: #AA5500; color: white">Brown</th>
<th style="text-align:center;">10</th>
<th style="text-align:center;background-color: #55FF55; color: black">Light<br/>Green</th>
<th style="text-align:center;">14</th>
<th style="text-align:center;background-color: #FFFF55; color: black">Yellow</th>
</tr>

<tr>
<th style="text-align:center;">3</th>
<th style="text-align:center;background-color: #00AAAA; color: white">Cyan</th>
<th style="text-align:center;">7</th>
<th style="text-align:center;background-color: #AAAAAA; color: white">Light<br/>Gray</th>
<th style="text-align:center;">11</th>
<th style="text-align:center;background-color: #55FFFF; color: black">Light<br/>Cyan</th>
<th style="text-align:center;">15</th>
<th style="text-align:center;background-color: #FFFFFF; color: black">White</th>
</tr>

</table>
</center>
<br/>
</div>

<pre><code>Attributes Byte:
Bits 0..3 : Foreground Color
Bits 4..6 : Background Color
Bit 7: 0 = No Blinking, 1 = Enable Blinking</code></pre>

نريد رسم الحروف بلون خلفية أزرق (١) ولون الحروف أبيض (١٥)، وبالتالي تكون الـ 
bits كلها واحد ماعدا bit5 وbit6 وbit7، أي أن الـ attribute سيكون مساويا لـ 0x1F. 
يوضح الجدول أدناه شكل الـ Video RAM بعد الكتابة:

<div dir="rtl">
<center>
<table border="2" width="800px" style="font-size:large; font-family: sans-serif">

<tr>
<th style="text-align:center;">العنوان</th>
<th style="text-align:center;">القيمة</th>
<th style="text-align:center;">المعنى</th>
<th style="text-align:center;">الصف المقابل</th>
<th style="text-align:center;">العمود المقابل</th>
</tr>

<tr>
<td style="text-align:center;">0xB8000</td>
<td style="text-align:center;">'H'</td>
<td style="text-align:center;">الحرف 'H'</td>
<td style="text-align:center;" rowspan="2">الصف الأول</td>
<td style="text-align:center;" rowspan="2">العمود الأول</td>
</tr>

<tr>
<td style="text-align:center;">0xB8001</td>
<td style="text-align:center;">0x1F</td>
<td style="text-align:center;">خلفية زرقاء وأمامية بيضاء</td>
</tr>

<tr>
<td style="text-align:center;">0xB8002</td>
<td style="text-align:center;">'e'</td>
<td style="text-align:center;">الحرف 'e'</td>
<td style="text-align:center;" rowspan="2">الصف الأول</td>
<td style="text-align:center;" rowspan="2">العمود الثاني</td>
</tr>

<tr>
<td style="text-align:center;">0xB8003</td>
<td style="text-align:center;">0x1F</td>
<td style="text-align:center;">خلفية زرقاء وأمامية بيضاء</td>
</tr>

<tr>
<td style="text-align:center;">0xB8004</td>
<td style="text-align:center;">'l'</td>
<td style="text-align:center;">الحرف 'l'</td>
<td style="text-align:center;" rowspan="2">الصف الأول</td>
<td style="text-align:center;" rowspan="2">العمود الثالث</td>
</tr>

<tr>
<td style="text-align:center;">0xB8005</td>
<td style="text-align:center;">0x1F</td>
<td style="text-align:center;">خلفية زرقاء وأمامية بيضاء</td>
</tr>

<tr>
<td style="text-align:center;">0xB8006</td>
<td style="text-align:center;">'l'</td>
<td style="text-align:center;">الحرف 'l'</td>
<td style="text-align:center;" rowspan="2">الصف الأول</td>
<td style="text-align:center;" rowspan="2">العمود الرابع</td>
</tr>

<tr>
<td style="text-align:center;">0xB8007</td>
<td style="text-align:center;">0x1F</td>
<td style="text-align:center;">خلفية زرقاء وأمامية بيضاء</td>
</tr>

<tr>
<td style="text-align:center;">0xB8008</td>
<td style="text-align:center;">'o'</td>
<td style="text-align:center;">الحرف 'o'</td>
<td style="text-align:center;" rowspan="2">الصف الأول</td>
<td style="text-align:center;" rowspan="2">العمود الخامس</td>
</tr>

<tr>
<td style="text-align:center;">0xB8009</td>
<td style="text-align:center;">0x1F</td>
<td style="text-align:center;">خلفية زرقاء وأمامية بيضاء</td>
</tr>

<tr>
<td style="text-align:center;">0xB800A</td>
<td style="text-align:center;">' '</td>
<td style="text-align:center;">مسافة</td>
<td style="text-align:center;" rowspan="2">الصف الأول</td>
<td style="text-align:center;" rowspan="2">العمود السادس</td>
</tr>

<tr>
<td style="text-align:center;">0xB800B</td>
<td style="text-align:center;">0x1F</td>
<td style="text-align:center;">خلفية زرقاء وأمامية بيضاء</td>
</tr>

<tr>
<td style="text-align:center;">0xB800C</td>
<td style="text-align:center;">'W'</td>
<td style="text-align:center;">الحرف 'W'</td>
<td style="text-align:center;" rowspan="2">الصف الأول</td>
<td style="text-align:center;" rowspan="2">العمود السابع</td>
</tr>

<tr>
<td style="text-align:center;">0xB800D</td>
<td style="text-align:center;">0x1F</td>
<td style="text-align:center;">خلفية زرقاء وأمامية بيضاء</td>
</tr>

<tr>
<td style="text-align:center;">0xB800E</td>
<td style="text-align:center;">'o'</td>
<td style="text-align:center;">الحرف 'o'</td>
<td style="text-align:center;" rowspan="2">الصف الأول</td>
<td style="text-align:center;" rowspan="2">العمود الثامن</td>
</tr>

<tr>
<td style="text-align:center;">0xB800F</td>
<td style="text-align:center;">0x1F</td>
<td style="text-align:center;">خلفية زرقاء وأمامية بيضاء</td>
</tr>

<tr>
<td style="text-align:center;">0xB8010</td>
<td style="text-align:center;">'r'</td>
<td style="text-align:center;">الحرف 'r'</td>
<td style="text-align:center;" rowspan="2">الصف الأول</td>
<td style="text-align:center;" rowspan="2">العمود التاسع</td>
</tr>

<tr>
<td style="text-align:center;">0xB8011</td>
<td style="text-align:center;">0x1F</td>
<td style="text-align:center;">خلفية زرقاء وأمامية بيضاء</td>
</tr>

<tr>
<td style="text-align:center;">0xB8012</td>
<td style="text-align:center;">'l'</td>
<td style="text-align:center;">الحرف 'l'</td>
<td style="text-align:center;" rowspan="2">الصف الأول</td>
<td style="text-align:center;" rowspan="2">العمود العاشر</td>
</tr>

<tr>
<td style="text-align:center;">0xB8013</td>
<td style="text-align:center;">0x1F</td>
<td style="text-align:center;">خلفية زرقاء وأمامية بيضاء</td>

</tr>
<tr>
<td style="text-align:center;">0xB8014</td>
<td style="text-align:center;">'d'</td>
<td style="text-align:center;">الحرف 'd'</td>
<td style="text-align:center;" rowspan="2">الصف الأول</td>
<td style="text-align:center;" rowspan="2">العمود الحادي عشر</td>
</tr>

<tr>
<td style="text-align:center;">0xB8015</td>
<td style="text-align:center;">0x1F</td>
<td style="text-align:center;">خلفية زرقاء وأمامية بيضاء</td>
</tr>

<tr>
<td style="text-align:center;">0xB8016</td>
<td style="text-align:center;">'!'</td>
<td style="text-align:center;">الحرف '!'</td>
<td style="text-align:center;" rowspan="2">الصف الأول</td>
<td style="text-align:center;" rowspan="2">العمود الثاني عشر</td>
</tr>

<tr>
<td style="text-align:center;">0xB8017</td>
<td style="text-align:center;">0x1F</td>
<td style="text-align:center;">خلفية زرقاء وأمامية بيضاء</td>
</tr>

</table>
</center>
<br/>
</div>

إذن فإن مهمة محرك الإقلاع بوضوح هي نقل البايتات الموضحة في العمود الثاني من 
الجدول أعلاه، إلى العناوين الموضحة بالعمود الأول. ولعمل ذلك يجب أولا تحميل 
المسجل DS بالقيمة 0xB800 لكي يتسنى لنا إرسال البيانات إلى الـ memory segment 
الذي يبدأ من 0xB8000:

<pre><code>MOV AX, 0xB800              # move 0xB800 to AX
MOV DS, AX                  # set data segment to 0xB800</code></pre>

بعد ذلك يمكن نقل البايتات مباشرة:

<pre><code>MOV BYTE PTR [0x0000], 'H'  # move 'H'  to 0xB800:0x0000 (0xB8000)
MOV BYTE PTR [0x0001], 0x1F # move 0x1F to 0xB800:0x0001 (0xB8001)
MOV BYTE PTR [0x0002], 'e'  # move 'e'  to 0xB800:0x0002 (0xB8002)
MOV BYTE PTR [0x0003], 0x1F # move 0x1F to 0xB800:0x0003 (0xB8003)
MOV BYTE PTR [0x0004], 'l'  # move 'l'  to 0xB800:0x0004 (0xB8004)
MOV BYTE PTR [0x0005], 0x1F # move 0x1F to 0xB800:0x0005 (0xB8005)
MOV BYTE PTR [0x0006], 'l'  # move 'l'  to 0xB800:0x0006 (0xB8006)
MOV BYTE PTR [0x0007], 0x1F # move 0x1F to 0xB800:0x0007 (0xB8007)
MOV BYTE PTR [0x0008], 'o'  # move 'o'  to 0xB800:0x0008 (0xB8008)
MOV BYTE PTR [0x0009], 0x1F # move 0x1F to 0xB800:0x0009 (0xB8009)
MOV BYTE PTR [0x000A], ' '  # move ' '  to 0xB800:0x000A (0xB800A)
MOV BYTE PTR [0x000B], 0x1F # move 0x1F to 0xB800:0x000B (0xB800B)
MOV BYTE PTR [0x000C], 'W'  # move 'W'  to 0xB800:0x000C (0xB800C)
MOV BYTE PTR [0x000D], 0x1F # move 0x1F to 0xB800:0x000D (0xB800D)
MOV BYTE PTR [0x000E], 'o'  # move 'o'  to 0xB800:0x000E (0xB800E)
MOV BYTE PTR [0x000F], 0x1F # move 0x1F to 0xB800:0x000F (0xB800F)
MOV BYTE PTR [0x0010], 'r'  # move 'r'  to 0xB800:0x0010 (0xB8010)
MOV BYTE PTR [0x0011], 0x1F # move 0x1F to 0xB800:0x0011 (0xB8011)
MOV BYTE PTR [0x0012], 'l'  # move 'l'  to 0xB800:0x0012 (0xB8012)
MOV BYTE PTR [0x0013], 0x1F # move 0x1F to 0xB800:0x0013 (0xB8013)
MOV BYTE PTR [0x0014], 'd'  # move 'd'  to 0xB800:0x0014 (0xB8014)
MOV BYTE PTR [0x0015], 0x1F # move 0x1F to 0xB800:0x0015 (0xB8015)
MOV BYTE PTR [0x0016], '!'  # move '!'  to 0xB800:0x0016 (0xB8016)
MOV BYTE PTR [0x0017], 0x1F # move 0x1F to 0xB800:0x0017 (0xB8017)</code></pre>

وبالتالي يصبح الشكل العام للبرنامج هكذا:

<pre><code>.intel_syntax noprefix
.code16
.text

MOV AX, 0xB800              # move 0xB800 to AX
MOV DS, AX                  # set data segment to 0xB800

MOV BYTE PTR [0x0000], 'H'  # move 'H'  to 0xB800:0x0000 (0xB8000)
MOV BYTE PTR [0x0001], 0x1F # move 0x1F to 0xB800:0x0001 (0xB8001)
MOV BYTE PTR [0x0002], 'e'  # move 'e'  to 0xB800:0x0002 (0xB8002)
MOV BYTE PTR [0x0003], 0x1F # move 0x1F to 0xB800:0x0003 (0xB8003)
MOV BYTE PTR [0x0004], 'l'  # move 'l'  to 0xB800:0x0004 (0xB8004)
MOV BYTE PTR [0x0005], 0x1F # move 0x1F to 0xB800:0x0005 (0xB8005)
MOV BYTE PTR [0x0006], 'l'  # move 'l'  to 0xB800:0x0006 (0xB8006)
MOV BYTE PTR [0x0007], 0x1F # move 0x1F to 0xB800:0x0007 (0xB8007)
MOV BYTE PTR [0x0008], 'o'  # move 'o'  to 0xB800:0x0008 (0xB8008)
MOV BYTE PTR [0x0009], 0x1F # move 0x1F to 0xB800:0x0009 (0xB8009)
MOV BYTE PTR [0x000A], ' '  # move ' '  to 0xB800:0x000A (0xB800A)
MOV BYTE PTR [0x000B], 0x1F # move 0x1F to 0xB800:0x000B (0xB800B)
MOV BYTE PTR [0x000C], 'W'  # move 'W'  to 0xB800:0x000C (0xB800C)
MOV BYTE PTR [0x000D], 0x1F # move 0x1F to 0xB800:0x000D (0xB800D)
MOV BYTE PTR [0x000E], 'o'  # move 'o'  to 0xB800:0x000E (0xB800E)
MOV BYTE PTR [0x000F], 0x1F # move 0x1F to 0xB800:0x000F (0xB800F)
MOV BYTE PTR [0x0010], 'r'  # move 'r'  to 0xB800:0x0010 (0xB8010)
MOV BYTE PTR [0x0011], 0x1F # move 0x1F to 0xB800:0x0011 (0xB8011)
MOV BYTE PTR [0x0012], 'l'  # move 'l'  to 0xB800:0x0012 (0xB8012)
MOV BYTE PTR [0x0013], 0x1F # move 0x1F to 0xB800:0x0013 (0xB8013)
MOV BYTE PTR [0x0014], 'd'  # move 'd'  to 0xB800:0x0014 (0xB8014)
MOV BYTE PTR [0x0015], 0x1F # move 0x1F to 0xB800:0x0015 (0xB8015)
MOV BYTE PTR [0x0016], '!'  # move '!'  to 0xB800:0x0016 (0xB8016)
MOV BYTE PTR [0x0017], 0x1F # move 0x1F to 0xB800:0x0017 (0xB8017)

CLI                         # disable interrupts.
HLT                         # halt the processor.

.org 510, 0x00              # move to 510 and fill spaces with 0x00.
.word 0xAA55                # BIOS boot signature.</code></pre>

قم بإنشاء دليل فارغ واحفظ فيه الملف باسم bootloader.s. الآن نحن جاهزون لتجميع 
البرنامج وربطه كما فعلنا في الفصل السابق. بدلا من كتابة الأوامر التي كتبناها في 
الفصل السابق كل مرة، يمكننا عمل Makefile بالأوامر التي نريد تنفيذها:

<pre><code>all:
	as -o bootloader.o bootloader.s
	ld --oformat=binary -o bootloader.bin -e 0 bootloader.o
	qemu-system-i386 -hda bootloader.bin</code></pre>

قم بحفظ الملف باسم Makefile في نفس الدليل. قم بتشغيل الطرفية وانتقل إلى الدليل 
الذي قمت بإنشائه، ثم قم بكتابة الأمر التالي في الطرفية والذي يقوم بتنفيذ الـ 
Makefile، أي تجميع الملف bootloader.s وإخراج bootloader.bin ثم تشغيل المحاكي 
qemu.

<pre><code>$ make</code></pre>

بعد أن يبدأ qemu في التشغيل ستقوم البايوس بتحميل قطاع الإقلاع الذي كتبناه في 
التو، ليقوم بطباعة Hello World على الشاشة في الصف الأول.

<p style="text-align:center">
<image src="/images/osdev/bootloader/shot5.png" />
</p>

-------------------------------------------------------------------------------

<a id="ref"></a>

المراجع:
---------

<div dir="ltr">
<ul>
<li>Alan Clements. <i>Principles of Computer Hardware</i> (Forth Edition).</li>
<li><i>Personal Computer XT System – Technical Reference</i>, IBM.</li>
<li><i>80C186XL/80C188XL Microprocessor User's Manual</i>, Intel.</li>
</ul>
</div>
