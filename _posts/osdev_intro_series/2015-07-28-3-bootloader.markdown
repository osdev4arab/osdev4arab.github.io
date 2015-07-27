---
layout: post
title: "٣- محمل الإقلاع bootloader"
date:   2015-07-28 08:02:00
tags: boot-loader hlt cli syntax boot cga display hello-world
comments: true
direction: rtl
author: iocoder
---

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
نقول للمجمع أن هذا البرنامج هو برنامج ١٦-بت، أي أنه برنامج للمعالج 8088 (أو 
80386 عندما يعمل في الوضع الحقيقي كما ذكرنا سابقا). السطر الأخير يوضح للمترجم أن 
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

محمل إقلاع Hello World:
----------------------------

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
الخلفية التي وراءه. يتم تمثيل لون المحرف بـ ٤ بت، ولون الخلفية بـ ٣ بت، أما 
البت المتبقى فيستخدم لتفعيل الوميض blinking.

ولأن لون المحرف يتم تمثيله بـ ٤ بت فإن عدد الألوان الممكنة هو ١٦ لون، ولذلك 
يسمى هذا الوضع (وضع ١٦ لون). ترقم الألوان من صفر إلى ١٥. لون المحرف ممكن أن يكون 
واحدا من الألوان الموجودة في الجدول أدناه. أما لون الخلفية يمكن فقط أن يكون واحد 
من أول ثمانية ألوان في الجدول.


<div dir="rtl">
<center>
<table border="2" width="800px" style="font-size:large; font-family: courier">

<tr>
<th style="text-align:center;">التسلسل</th>
<th style="text-align:center;">اللون</th>
<th style="text-align:center;">التسلسل</th>
<th style="text-align:center;">اللون</th>
<th style="text-align:center;">التسلسل</th>
<th style="text-align:center;">اللون</th>
<th style="text-align:center;">التسلسل</th>
<th style="text-align:center;">اللون</th>
</tr>

<tr>
<th style="text-align:center;">0</th>
<th style="text-align:center;background-color: #000000; color: white">أسود</th>
<th style="text-align:center;">4</th>
<th style="text-align:center;background-color: #AA0000; color: white">أحمر</th>
<th style="text-align:center;">8</th>
<th style="text-align:center;background-color: #555555; color: white">رصاصي</th>
<th style="text-align:center;">12</th>
<th style="text-align:center;background-color: #FF5555; color: white">أحمر<br/>فاتح</th>
</tr>

<tr>
<th style="text-align:center;">1</th>
<th style="text-align:center;background-color: #0000AA; color: white">أزرق</th>
<th style="text-align:center;">5</th>
<th style="text-align:center;background-color: #AA00AA; color: white">بنفسجي</th>
<th style="text-align:center;">9</th>
<th style="text-align:center;background-color: #5555FF; color: white">أزرق<br/>فاتح</th>
<th style="text-align:center;">13</th>
<th style="text-align:center;background-color: #FF55FF; color: white">بنفسجي<br/>فاتح</th>
</tr>

<tr>
<th style="text-align:center;">2</th>
<th style="text-align:center;background-color: #00AA00; color: white">أخضر</th>
<th style="text-align:center;">6</th>
<th style="text-align:center;background-color: #AA5500; color: white">بني</th>
<th style="text-align:center;">10</th>
<th style="text-align:center;background-color: #55FF55; color: black">أخضر<br/>فاتح</th>
<th style="text-align:center;">14</th>
<th style="text-align:center;background-color: #FFFF55; color: black">أصفر</th>
</tr>

<tr>
<th style="text-align:center;">3</th>
<th style="text-align:center;background-color: #00AAAA; color: white">سماوي</th>
<th style="text-align:center;">7</th>
<th style="text-align:center;background-color: #AAAAAA; color: white">رصاصي<br/>فاتح</th>
<th style="text-align:center;">11</th>
<th style="text-align:center;background-color: #55FFFF; color: black">سماوي<br/>فاتح</th>
<th style="text-align:center;">15</th>
<th style="text-align:center;background-color: #FFFFFF; color: black">أبيض</th>
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
