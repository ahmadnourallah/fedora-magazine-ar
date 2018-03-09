# حسِّن مشاريع بايثون باستخدام mypy

<img src='https://fedoramagazine.org/wp-content/uploads/2018/02/mypy-945x400.jpg'/>

أداة *myapp* هي مدقق نوعي ساكن للغة بايثون. تجمع هذه الأداة بين فوائد الكتابة الديناميكية (dynamic typing) والكتابة الساكنة (static typing). هذا يجعلنا نتسائل عن معنى التحقق النوعي الساكن (static type checking) في بايثون؟ أكمل القراءة لتكتشف ذلك.

## ما هو النوع؟

يحتاج النظام لمعرفة مساحة الذاكرة المطلوبة ليتمكن البرنامج من تخزين البيانات. تستخدم لغات البرمجة *الأنواع* (types) لتحديد ذلك. يحدد كل نوع حجم الذاكرة الذي ينبغي على البرنامج حجزه لتخزين البيانات. بعض الأنواع الشائعة: العدد الصحيح (integer)، والعدد العشري (float)، والسلسلة النصية (string).

تتحقق اللغة المكتوبة ديناميكياً من الأنواع في وقت التشغيل. تملك لغة بايثون نظام أنواع "ضعيف (weak)"، أي أنَّ المُفسِر لا يفرض أي عملية تحقق من الأنواع.

تتحقق اللغة المكتوبة سكونياً من الأنواع بناءً على تحليلها للنص المصدري للبرنامج قبل تشغيله. فعندما ينجح البرنامج بعملية التحقق، يعني ذلك نجاحه في مجموعة من عمليات التحقق من سلامة الأنواع (type safety). يحدد التحقق النوعي الساكن الأخطاء المحتملة في شيفرة البرنامج قبل تشغيله. ولذلك برمجية *myapp* - والتي هي مدقق نوعي ساكن لتطبيقات بايثون - مفيدة.

## تثبيت وتشغيل أداة mypy

تثبيت الأداة سهل على فيدورا فهي محزمة في المستودعات الرسمية:

```
$ dnf install python3-mypy
```

سنقوم الآن بإنشاء تطبيق بايثون بسيط لفهم وإختبار آلية عمل هذه الأداة.

```python
class Person():
    def __init__(self,surname,firstname,age,job):
        self.surname = surname
        self.firstname = firstname
        self.age = age
        self.job = job


def display_doctors(persons):
    for person in persons:
        if person.job.lower()in['gp','dentist','cardiologist']:
            print(f'{person.surname} {person.name}')

mike = Person('Davis', 'Mike', '45', 'dentist')
john = Person('Roberts', 'John', 21, 'teacher')
lee = Person('Willams', 'Lee', 'gp', 56)

display_doctors(mike)
display_doctors([mike, john, 'lee'])
```

احفظ هذا المقتطف البرمجي في ملف باسم *testing_mypy.py*. قم الآن بتشغيل أداة *mypy* على هذا الملف:

```
$ mypy testing_mypy.py
```

لاحظ أنَّ أداة *myapp* متساهلة إفتراضياً، فعند تشغيلك للمثال أعلاه لن يتم إرجاع أيَّة أخطاء. هذا التساهل اﻹفتراضي مفيد للتطبيق الذي يحتوي على قاعدة شيفرات كبيرة، بحيث يمكنك استخدام هذه البرمجية تدريجياً في مشروعك.

### إضافة تلميحات للأنواع

مع إصدار بايثون 3.6 الإفتراضي في فيدورا 27 فما فوق، يمكنك إضافة تلميحات للأنواع (type hints) لشرح شيفرتك. ومن ثم تقوم أداة mypy بفحص التطبيق بناءً على هذه التمليحات. قم بفتح محررك لإضافة تلميحات للأنواع في الدالة *display_doctors* في المثال السابق:

```python
from typing import List
def display_doctors(persons: List[Person]) -> None:
    for person in persons:
    if person.job.lower()in['gp','dentist','cardiologist']:
        print(f'{person.surname} {person.name}')
```

أُضيفَت التلميحات التالية في المثال السابق:

<ul style='list-style-type: disc; list-style-position: inside;'>
  <li>التلميح <i>List[Person]</i>: يشرح أنَّ الدالة <i>display_doctors</i> تتوقع أنْ يُدخَل لها الكائن <i>Person</i> في قائمة.</li>
  <li>التلميح <i>-> None</i>: يشرح أنَّ الدالة ستعيد القيمة <i>None</i>.</li>
</ul>

دعنا نقوم بتشغيل mypy على البرمجية مجدداً:

```bash
$ mypy testing_mypy.py
test_mypy.py:19: error: "Person" has no attribute "name"
test_mypy.py:26: error: Argument 1 to "display_doctors" has incompatible type "Person"; expected "List[Person]"
test_mypy.py:27: error: List item 2 has incompatible type "str"; expected "Person"
```

أظهرت البرمجية بعض الأخطاء التي يمكن إصلاحها كما يلي:

<ul style='list-style-type: disc; list-style-position: inside;'>
  <li>الخطأ الأول في الدالة <i>print</i>، حيث يحاول البرنامج طباعة person.name، ولكن هذا المتغير غير موجود. بدلاً من ذلك، يجب أنْ يطبع البرنامج <i>person.firstname</i>.</li>
  <li>حدث الخطأ الثاني عند محاولة البرنامج لاستدعاء الدالة <i>display_doctors</i> لأول مرة. حيث أنَّ المعامل المتوقع هو الكائن <i>Person</i> في قائمة، لكن قد مُرِرَ في المثال الكائن <i>Person</i> وحسب.</li>
  <li>حدث الخطأ الأخير نتيجة خطأ في القائمة المُدخلة. فبدلاً من إضافة الكائن <i>lee</i> إلى القائمة، تم إدخاله كسلسلة نصية (<i>'lee'</i>).</li>
</ul>

فيما يلي تطبيق الإصلاحات المشروحة على البرنامج:

```python
def display_doctors(persons: List[Person]) -> None:
    for person in persons:
    if person.job.lower()in['gp','dentist','cardiologist']:
        print(f'{person.surname} {person.firstname}')
...
```

```python
display_doctors([mike])
display_doctors([mike, john, lee])
```

عدِّل الشيفرة كما شُرِح أعلاه وأعد تشغيل <i>mypy</i> مجدداً.

ستجد المزيد من الأخطاء في البرنامج. مهمتك كقارئ إيجادهم. انظر إلى استخدام التلميحات على الصنف (class) <i>Person</i>. جرب بنفسك أولاً. إن أردت التحقق من عملك، فيمكنك إيجاد البرنامج خالي من الأخطاء في [هذا المستودع](https://github.com/cverna/testing_mypy).

## الخاتمة

آمل أنْ تريك هذه المقدمة القصيرة عن أداة <i>mypy</i> بعض فوائد استخدامها على برمجيتك. تجعل هذه الأداة من السهل صيانة، وقراءة شيفرة البرمجيات. تُمكنُكَ أيضاً من إلتقاط الأخطاء قبل تشغيل الشيفرة. تتوفر الأداة أيضاً لتطبيقات بايثون الإصدار 2.7، ولكنها تستخدم صيغة مختلفة تعتمد على التعليقات.

المصدر: [https://fedoramagazine.org/improve-python-projects-mypy/](https://fedoramagazine.org/improve-python-projects-mypy/)
