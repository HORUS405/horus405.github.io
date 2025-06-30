---
layout: post
status: publish
published: true
title: Stripped and Unstripped Binaries 
author: horus
categories: [Binary_Exploitation]
tags: [Binary_Exploitation , Arabic]
comments: true
image: 0x00.png
lang: ar
---

###     …  بِسْمِ اللَّـهِ الرَّحْمَـٰنِ الرَّحِيمِ  …


## المقدمة :
 في هذه السلسة بأذن الله سنشرح الBinary Exploitation techniques and Basics باللغة العربية قدر المستطاع لأثراء المحتوى العربي في مجال الامن السيبراني و سنبدأ اول مقال بشرح الفرق بين ال  stripped binary و ال unstripped binary و كيف نجد الmain function في stripped binaries 


---
### Stripped Vs Unstripped : 
تنقسم انواع الbinaries الى نوعان stripped و unstripped  . الunstripped binary هو عند تحويل الكود الي binary بطريقة الcompile العادية عن طريق هذا الامر مثلا `gcc code.c -o code` . اما عن الstripped binary هي عند تحويل الكود الي binary لكن بحذف الdebugging information لاخفاء اسامي الfunctions و تصعيب المهمة علي الreverse engineers و ايضا تخفيف حجم الbinary , يمكنك تحويل الكود الي stripped binaries عن طريق هذا الامر `gcc code.c -o code -s` الفلاج`s-` ده هو الي بيقول للcompiler شيل الdebugging information و انت بتعمل compile 
>compiling proccess : لما نعمل compile للكود بتاعنا الcompiler اوتوماتك بيحط symbols اسمها الdebugging information ديه بتساعد في عملية الdebugging اكتر و بتسهل الموضوع اما في حالة الstripped الcompiler مش بيحط الsymbols ديه 

 
---
## Testing : 
يلا نكتب كود بسيط جدا عشان نعرف بس الفرق بين الstripped و الunstripped و بيظهروا ازاي في الdisassembler ,
ده الكود :  
- ![](/assets/images/ray-so-export.png)

بعد ما كتبناه يلا نعمله compile مرة stripped و مرة unstripped عشان نشوف الفرق 
- ![](/assets/images/Pasted%20image%2020230706125445.png)
زي ما قولنا فعلا الstripped binary طلع مساحته اقل من الunstripped ,بس ده مش مهم اوي عشان نشوف المهم تعالوا نروح علي الdisassembler و المرة دي انا هستخدم ghidra .
لو فتحنا ghidra و روحنا على المكان الي بيتعرض فيه الfunctions ,في حالة الunstripped هنلاقي الmain function و باقي الfunctions عادي , اما في حالة الstripped مش هنلاقي اسم الmain function او اي اسماء لfunctions تانية, هنلاقي الfunctions بتبدي بأسم `_FUN` زي ما موضح في الصورة : 
- ![](/assets/images/Pasted%20image%2020230707081446.png)

هنعرف في الجزء الجي ده ازاي بقى نجيب الmain function لو قابلنا stripped binary 

---


## ?How To Find The Main Function

### Method 1 (ghidra) :
اسهل طريقة تقدر انك تجيب بيها الmain function هي انت تجيب الentrypoint و منها تجيب address الmain عن طريق انك تشوف اولargument في function اسمها `libc_start_main__` . 
بس ايه هي اصلا الfunction الي اسمها `libc_start_main__` اول function بتشتغل في الentry هي ال__libc_start_main i هي العقل لبرنامج الc الي شغال معانا بتعمل كل حاجة من اول الinitialization لحد الexit و لو شوفنا الdocumentation بتاعها هنلاقي ان اول حاجة بتاخدها هي الaddress بتاع الmain function 
- ![](/assets/images/Pasted%20image%2020230707080019.png)
 > entrypoint : الentrypoint هي نقطة البداية للbinary و هي الmemory address الي البرنامج بيبدأ منها  
 
يلا بقى نكمل مع ghidra عشان نجيب الmain function 
- ![](/assets/images/Pasted%20image%2020230707080412.png)
ولو روحنا للaddress ده فعلا هنلاقي الmain function بتاعتنا و كمان function الي عملناها و استخدمناها في الكود 
- ![](/assets/images/Pasted%20image%2020230707080631.png)

---

### Method 2 (gdb-\*) :

اسهل طريفة نجيبب منها الmain function في gdb هي اننا نشوف text section و نشوف اول function بيتعملها call الي هي ال libc_start_main__ و نجيب القيمة بتاعت اول argument  بيتحطلها و هو ده هيكون الaddress بتاع الmain function 
> ايه هو اصلا الsection الي اسمه .text ده ؟ 
> اي executable بيكون فيه sections و كل section بيكون مسؤول عن شيئ معين زي مثلا ال .text مسؤول انه يشيل الinstruction الي هي الكود او مثلا جزء ال.data مسؤول انه يشيل variables 

- ![](/assets/images/Pasted%20image%2020230707084142.png)

يلا بقى نرجع علي gdb و نجيب الmain function و ديه الخطوات الي همشي عليها 
نكتب الامر ده عشان نعرض الsections و الaddresses بتاعتها `info file`
- ![](/assets/images/Pasted%20image%2020230707085046.png)
نجيب الaddress بتاع الtext section و نشوف الinstructions الي فيه عن طريق الامر ده `x/20i <text-section-address`
- ![](/assets/images/Pasted%20image%2020230707085326.png)
ناخذ القيمة الي هتتمرر كargument لlibc_start_main__   (هي ديه الaddress بتاع الmain) و بعدين
نشوف بقى الmain function بتاعتنا و الfunctions الي كتبناها و استخدمناها 
- ![](/assets/images/Pasted%20image%2020230707085754.png)
## الخاتمة : 
و كدة الحمدالله يبقى خلصنا الشرح بتاعنا يارب تكونوا استفدتم لو عندك اي تعليق او شايف حاجة محتاجة تتعدل . 

تقدر تتواصل معايا عن طريق دول : 

twitter : https://twitter.com/Horus405

facebook : https://www.facebook.com/profile.php?id=100006298179182