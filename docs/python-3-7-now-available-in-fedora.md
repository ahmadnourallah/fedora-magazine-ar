# لغة بايثون 3.7 متوفرة اﻵن في فيدورا

<img src='https://fedoramagazine.org/wp-content/uploads/2018/03/python37-945x400.jpg'/>

أُصدِرَت النسخة التجريبيَّة الثانية من بايثون الإصدار 3.7 في 28 شباط عام 2018. يحتوي هذا الإصدار على الكثير من الإصلاحات، وعلى وجه الخصوص، العديد من الميزات الجديدة المتاحة للاختبار. لا يتوفر الإصدار المسبق (التجريبي) من بايثون 3.7 على فيدورا المتدحرجة (Rawhide) وحسب، بل وعلى جميع إصدارات فيدورا الأخرى. أكمل القراءة لمعرفة المزيد.

## التثبيت والأساسيات

من السهل تثبيت بايثون 3.7 على نسخة فيدورا مدعومة، فقط قم بتشغيل هذا الأمر:

```
sudo dnf install python37
```

ومن ثم نَفِذ الأمر `python3.7` لتختبر الإصدارة. يمكنك أيضاً [إنشاء بيئة إفتراضية](https://developer.fedoraproject.org/tech/languages/python/python-installation.html#using-virtualenv) باستخدام الإصدار الجديد، أو إضافة `py37` إلى ملف [tox.ini](https://developer.fedoraproject.org/tech/languages/python/multiple-pythons.html#getting-it-and-running-it-all-with-tox) الخاص بمشروعك وبدء اختبار النسخة الطازجة من بايثون:

```
$ python3.7 -m venv venv
$ . venv/bin/activate
(venv) $ python --version
Python 3.7.0b2
```

لا توجد مكتبات إضافية لبايثون 3.7 مُحزَمِة في فيدورا بعد. بالرغم من ذلك، كل ما تريد يمكنك توفيره باستخدام [virtualenv و pip](https://developer.fedoraproject.org/tech/languages/python/python-installation.html#using-virtualenv):

```
(venv) $ python -m pip install requests  # or any other package
Collecting requests
  Using cached requests-2.18.4-py2.py3-none-any.whl
Collecting idna<2.7,>=2.5 (from requests)
  Using cached idna-2.6-py2.py3-none-any.whl
Collecting certifi>=2017.4.17 (from requests)
  Using cached certifi-2018.1.18-py2.py3-none-any.whl
Collecting urllib3<1.23,>=1.21.1 (from requests)
  Using cached urllib3-1.22-py2.py3-none-any.whl
Collecting chardet<3.1.0,>=3.0.2 (from requests)
  Using cached chardet-3.0.4-py2.py3-none-any.whl
Installing collected packages: idna, certifi, urllib3, chardet, requests
Successfully installed certifi-2018.1.18 chardet-3.0.4 idna-2.6 requests-2.18.4 urllib3-1.22
```

## ميزة بايثون 3.7 الجديدة: صنوف البيانات

إليك مثال على إحدى الميزات الرائعة لإصدار بايثون 3.7.

كم مرَّة كتبت `self.attribute = attribute` في الدالة `__init__`؟ سيكون جواب غالبيَّة مبرمجي بايثون: "العديد من المرات". مع استخدام `__repr__` ودعم الموازنات (comparisons)، يوجد الكثير من الطرق المتداولة لإنشاء صنف يحوي فقط على حفنة من البيانات. يحل مشروع [attrs](https://pypi.python.org/pypi/attrs/) الممتاز الكثير من هذه المشكلات. الآن مجموعة فرعية من أفكار مشروع attrs اختيرت بعناية ليتم وضعها في مكتبة قياسيَّة.

إليك مثال عن الميزة بشكل عملي:

```
from dataclasses import dataclass

@dataclass
class Point:
    x: float
    y: float
    z: float = 0.0

>>> p = Point(1.5, 2.5)
>>> print(p)
Point(x=1.5, y=2.5, z=0.0)
```

لاحظ أنَّ أنواع المتغيرات وضعت لغرض توثيقي فقط. لن يتم فحصها خلال تشغيل البرنامج. ولكن بالرغم من ذلك، سيتم فحصها باستخدام مدقق نوعي ساكن (static type checker) مثل [mypy](https://fedoramagazine.org/improve-python-projects-mypy/). صنوف البيانات (Data Classes) موثقة حالياً في [إقتراح بايثون التعزيزي رقم 557](https://www.python.org/dev/peps/pep-0557/). إذا أحسست أنَّ المكتبة البرمجيَّة ضيقة الأفق (محدودة)، فتذكر أنَّ ذلك مقصود. يمكنك دائماً التحوّل لاستخدام المكتبة [attrs](https://pypi.python.org/pypi/attrs/) إذا احتجت المزيد من الميزات.

هذه الميزة وغيرها من الميزات ستأتي مع الإصدار الجديد. سيصبح بايثون الإصدار 3.7 
[الإصدار الرئيسي في فيدورا 29](https://fedoraproject.org/wiki/Changes/Python3.7). إذا وجدت أي علّة، يرجى التبليغ عنها في [مبلّغ علل فيدورا](https://bugz.fedoraproject.org/python37) أو مباشرةً في [المنبع](https://bugs.python.org/). إذا كان لديك أي سؤال، أسأل المجموعة المختصة ببايثون على [python-devel@lists.fedoraproject.org](mailto:python-devel@lists.fedoraproject.org) أو قم بزيارة قناة IRC على خادم Freendoe على [#fedora-python](https://webchat.freenode.net/?channels=#fedora-python).

المصدر: [https://fedoramagazine.org/python-3-7-now-available-in-fedora/](https://fedoramagazine.org/python-3-7-now-available-in-fedora/)
