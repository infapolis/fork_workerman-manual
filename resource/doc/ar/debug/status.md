يمكنك مراجعة حالة تشغيل Workerman باستخدام الأمر `start.php status` ، ويمكنك رؤية حالة تشغيل Workerman التالية مثل النموذج التالي:

```
----------------------------------------------GLOBAL STATUS----------------------------------------------------
إصدار Workerman: 3.5.13         إصدار PHP: 5.5.9-1ubuntu4.24
وقت البدء: 2018-02-03 11:48:20   تشغيل 112 أيام 2 ساعة   
متوسط الحمل: 0، 0، 0           حلقة الأحداث: \Workerman\Events\Event
4 عمال       11 عملية
worker_name      حالة الخروج          عدد مرات الخروج
ChatBusinessWorker 0                0
ChatGateway        0                0
Register           0                0
WebServer          0                0
----------------------------------------------PROCESS STATUS---------------------------------------------------
pid	memory  listening                worker_name        الاتصالات send_fail timers  total_request qps    حالة
18306	2.25M   لا شيء                     ChatBusinessWorker 5           0         0       11            0      [الخمول]
18307	2.25M   لا شيء                     ChatBusinessWorker 5           0         0       8             0      [الخمول]
18308	2.25M   لا شيء                     ChatBusinessWorker 5           0         0       3             0      [الخمول]
18309	2.25M   لا شيء                     ChatBusinessWorker 5           0         0       14            0      [الخمول]
18310	2M      websocket://0.0.0.0:7272 ChatGateway        8           0         1       31            0      [الخمول]
18311	2M      websocket://0.0.0.0:7272 ChatGateway        7           0         1       26            0      [الخمول]
18312	2M      websocket://0.0.0.0:7272 ChatGateway        6           0         1       21            0      [الخمول]
18313	1.75M   websocket://0.0.0.0:7272 ChatGateway        5           0         1       16            0      [الخمول]
18314	1.75M   text://0.0.0.0:1236      Register           8           0         0       8             0      [الخمول]
18315	1.5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [الخمول]
18316	1.5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [الخمول]
----------------------------------------------PROCESS STATUS---------------------------------------------------
الملخص	18M     -                        -                  54          0         4       138           0      [الملخص]
```
## شرح

### الحالة العامة

من هذا القسم يمكننا رؤية

إصدار Workerman```version:3.5.13```

وقت البدء ```2018-02-03 11:48:20``` ، وقت التشغيل ```112 أيام 2 ساعة```

حمولة الخادم ```load average: 0, 0, 0``` ، وهي متوسط الحمولة في النظام خلال الدقيقة الأخيرة، و5 دقائق الأخيرة، و 15 دقيقة الأخيرة على التوالي

مكتبة حدث الإدخال/ الإخراج المستخدمة، ```event-loop:\Workerman\Events\Event``` 

```4 workers``` (3 أنواع من العمليات، بما في ذلك عمليات ChatGateway و ChatBusinessWorker و Register وعملية WebServer)

```11 process``` ( 11 عملية بالمجموع)

``` worker_name``` (اسم عملية العامل)

 ``` exit_status``` (رمز حالة الخروج لعملية العامل)

 ``` exit_count``` ( عدد مرات الخروج بهذا الرمز)

بشكل عام، exit_status 0 يعني الخروج الطبيعي. إذا كانت قيمة غير ذلك، فإن العملية خرجت بشكل غير متوقع وستولّد رسالة خطأ مشابهة لـ  ```WORKER EXIT UNEXPECTED``` وسيتم تسجيل الخطأ في الملف المحدد في [Worker::logFile](worker/log-file.md).

**أشهر exit_status ومعانيهاالمتداولة عادة:**

* 0 : يعني الخروج الطبيعي، يمكن أن يظهر هذا الرمز بعد إعادة تشغيل ناعمة reload وهو أمر طبيعي. يجب ملاحظة أن استخدام exit أو die في كود الأعمال يؤدي إلى خروج طبيعي وظهور رسالة خطأ ```WORKER EXIT UNEXPECTED``` في workerman. 


* 9: يعني أن العملية قُتلت بواسطة إشارة SIGKILL. هذا الرمز يحدث أساسًا في حالات إيقاف وتشغيل ناعم reload، والسبب في ذلك هو أن العملية الفرعية لم تستجب لإشارة إعادة تحميل الرئيسية في الوقت المحدد (على سبيل المثال mysql وcurl وغيرها تنتظر لفترة طويلة أو حلقة تكرار ميتة في الأعمال وغير ذلك)، ويُلزم النسخ الاحتياطي القتل بواسطة إشارة SIGKILL. يجب الملاحظة أن استخدام أمر kill في سطر أوامر لينكس لإرسال إشارة SIGKILL لعملية فرعية يمكن أن يؤدي أيضًا إلى ظهور هذا الرمز.


* 11 : يُظهر أن PHP قد حدث core dump، وهذا عادة ما يكون ناجمًا عن استخدام إضافات غير مستقرة، يجب أن تعلق تلك الإضافات المُحددة في php.ini؛ وهناك أحيانا قليلة يكون ذلك ناجمًا عن خلل في PHP وفي هذه الحالة يجب ترقية PHP.


* 65280 : السبب وراء ظهور هذا الرمز هو خطأ فادح في الكود الأعمال مثل استدعاء دالة غير موجودة أو خطأ في الصيغة إلخ، وسيتم تسجيل المعلومات المحددة في الملف المحدد في [Worker::logFile](worker/log-file.md) ويمكن العثور على التفاصيل في الملف المحدد في [php.ini](https://php.net/manual/zh/ini.list.php) في [error_log](https://php.net/manual/zh/errorfunc.configuration.php#ini.error-log).


* 64000 : السبب وراء ظهور هذا الرمز هو أن الكود الأعمال قد أطلق استثناء ولم يتم التقاط هذا الاستثناء، وبالتالي أدى إلى خروج العملية. إذا كان workerman يعمل في وضع debug فإن مكدونة الاستثناء ستكون مطبوعة على الطرفية، وأما إذا كان يعمل في وضع الخلفية فإن مكدونة الاستثناء ستتم تسجيل في الملف المحدد في [Worker::stdoutFile](worker/stdout-file.md).


### حالة العمليات

**pid** : معرف العملية

**memory** : الذاكرة المستخدمة حالياً من قبل هذه العملية (باستثناء ذاكرة الملف التنفيذي لـ PHP)

**listening** : بروتوكول طبقة النقل وعنوان IP والمنفذ الذي يستمع عليه. إذا لم يكن يستمع إلى أي من الأبواب، سيتم عرض "لا شيء" 

**worker_name** : اسم الخدمة التي تشغلها هذه العملية، الرجاء الرجوع إلى [Worker class name attribute](worker/name.md)

**connections** : عدد الاتصالات TCP المستخدمة حالياً من قبل هذه العملية، تشمل الاتصالات مثيلات TcpConnection و AsyncTcpConnection. هذا القيمة هي القيمة الفعلية وليست تراكمية. يجب ملاحظة أنه إذا تم استدعاء إغلاق الاتصال ولم ينقص العداد المقابل، فقد يكون ذلك ناجمًا عن احتفاظ كود الأعمال بكائن $connection مما يؤدي إلى عدم تدمير مثيل الاتصال.


**total_request** : يُظهر كم من الطلبات استقبلت هذه العملية منذ بدء التشغيل. تشمل الطلبات الواردة من العملاء والطلبات الداخلية في Workerman، على سبيل المثال، الطلبات بين البوابة والعامل الأعمال في هندسة GateWay. هذه قيمة تراكمية.

**send_fail** : عدد مرات فشل هذه العملية في إرسال البيانات إلى العميل، الأسباب الشائعة للفشل تكون إغلاق اتصال العميل. إذا لم يكن هذا العدد صفرًا، فهذا يعتبر عادةً كونه حالة طبيعية، يُرجى الرجوع إلى [send_fail reasons in status](../faq/about-send-fail.md).

**timers** : عدد مؤقتات النشاط لهذه العملية (باستثناء الموقتات التي تم حذفها والتي تم تشغيلها مرة واحدة). يُرجى ملاحظة أن تلك الميزة تتطلب إصدار Workerman >= 3.4.7 وهي قيمة فعلية وليست تراكمية.

**qps** : عدد الطلبات الشبكية التي يتلقاها العملية كل ثانية، يُرجى ملاحظة أن تلك الميزة تتطلب إصدار Workerman >= 3.5.2 وهي قيمة فعلية وليست تراكمية.


**status** : حالة العملية، إذا كانت المعالجة تعتبر غير مشغولة "idle" أو مشغولة "busy". يُرجى ملاحظة أنه إذا كانت العملية مشغولة لفترة قصيرة من الوقت فإن ذلك يُعتبر ظاهرة عادية، أما إذا كانت العملية مشغولة بشكل دائم، فقد يكون ذلك ناتجًا عن انسداد في الأعمال أو حلقة تكرار ميتة في الأعمال ويجب التحقق من ذلك وفقًا للجزء [تصحيح العمليات المشغولة](busy-process.md).


## المبدأ
بعد تشغيل سكريبت status، سيقوم العملية الرئيسية بإرسال إشارة "SIGUSR2" إلى جميع عمليات العامل (worker)، ثم يدخل سكريبت status في مرحلة قصيرة من السبات للانتظار حتى تجمع جميع عمليات العامل البيانات الخاصة بها. في هذا الوقت، سيقوم العمليات العاملة الشاغرة بكتابة حالتها الحالية (عدد الاتصالات، وعدد الطلبات، إلخ) إلى ملفات القرص الخاصة، بينما ستنتظر العمليات العاملة التي تعمل في معالجة الطلبات التجارية حتى الانتهاء من معالجة الطلب قبل كتابة حالتها الخاصة. بعد السبات القصير، يبدأ سكريبت status في قراءة ملفات الحالة في القرص وعرض النتائج على واجهة السطر.

## ملاحظة
قد يظهر بعض العمليات كمشغولة أثناء عرض الحالات، وذلك بسبب تشغيلها في معالجة الطلبات (على سبيل المثال، تعليق المنطق التجاري لفترة طويلة في طلبات curl أو قاعدة البيانات، أو تشغيل حلقة كبيرة)، وبالتالي لا يمكن تقديم الحالة. يجب تفقد الشفرة التجارية لمعرفة سبب توقف الطلبات لفترة طويلة، وقيمة الوقت المستغرق للتقييم، وإذا لم تكن تستجيب للتوقعات، يجب إجراء فحص لشفرة العمل باستخدام الفقرة [تصحيح عمليات العمل المشغولة](busy-process.md).