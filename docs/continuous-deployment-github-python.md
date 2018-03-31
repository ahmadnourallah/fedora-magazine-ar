# إجراء عمليّة نشر متواصِل مع جيتهاب وبايثون

<img src='https://fedoramagazine.org/wp-content/uploads/2018/03/cd-github-python-945x400.jpg'/>

يُمكِن للمطورين إنشاء العديد من الخدمات المفيدة باستخدم خدمة الخطاطيف الشابكيّة (webhooks) من جيتهاب (GitHub)، فَمِن إجراء عمليّة تكامِل متواصل (CI) على برمجيّة جنكينز (Jenkins) إلى تموين (تحديد موارد - provisioning) الأجهزة في السحابة (cloud)، الاحتمالات غير محدودة. في هذا الدرس سنتعلم كيفيّة استخدام بايثون وإطار فلاسك (Flask) لبناء خدمة نشر متواصِل (continuous deployment) بسيطة.

مثال "النشر المتواصِل" الذي سنقوم به هنا هو عبارة عن تطبيق فلاسك بسيط فيه نقطة نهاية (endpoint) تستقبل الطلبات من خطاطيف جيتهاب الشابكيّة. بعد التحقق من كل طلب والتأكُّد أنَّه قادم من مستودع (repository) جيتهاب الصحيح، ستقوم الخدمة بسحب (pull) التغييرات إلى النسخة المحليّة من المستودع (الموجودة في الخادم الذي تعمل عليه الخدمة). وبهذه الطريقة، كل مرَّة يُدفَع إيداع جديد إلى مستودع جيتهاب البعيد (remote)، سيتم تحديث المستودع المحلي تلقائياً.

## إنشاء خدمة ويب بواسط فلاسك
من السهل بناء خدمات ويب صغيرة بواسط فلاسك. إليك مثال على هيكلة ملفات المشروع.

```
├── app
│   ├── __init__.py
│   └── webhooks.py
├── requirements.txt
└── wsgi.py

```

بدايةً، سنقوم بإنشاء التطبيق. شيفرة التطبيق توضع في الدليل *app*.

ينُشئ الملفان (<i>init</i><i>\_\_</i><i>.py</i><i>\_\_</i> و *webhooks.py*) تطبيق فلاسك. الأوّل يحتوي على شيفرة إنشاء التطبيق وعلى خيارات الإعداد (configuration). أما الثاني، فيحتوي على شيفرة نقطة النهاية، وهو المكان الذي سيستقبل فيه التطبيق البيانات من طلبات الجيتهاب.

إليك محتوى الملف *app/\_\_init\_\_.py*:

```python
import os
from flask import Flask

from .webhooks import webhook

def create_app():
 """ Create, configure and return the Flask application """

  app = Flask(__name__)
  app.config['GITHUB_SECRET'] = os.environ.get('GITHUB_SECRET')
  app.config['REPO_PATH'] = os.environ.get('REPO_PATH')
  app.register_blueprint(webhook)

  return(app)
```

تُنشئ الدالة متغيّرين إعداد:

<ul style='list-style-type: disc; list-style-position: inside;'>
  <li>المتغيّر <b>GITHUB_SECRET</b>: يحتوي على العبارة السريّة التي ستستخدم لتوثيق (للمصادقة - authenticate) طلبات الجيتهاب.</li>
  <li>المتغيّر <b>REPO_PATH</b>: يحتوي على مسار المستودع - المحلي - الذي سيُحدَث تلقائياً.</li>
</ul>

سنستخدم [المُخطّطات](http://flask.pocoo.org/docs/0.12/blueprints/) (blueprints) لتنظيم المشروع. يتيح استخدام المُخطّطات تجميع الواجهات البرمجيّة (APIs) بشكل منطقي، مما يجعل صيانة التطبيقات أسهل. استخدام المُخطّطات يُعتبَر عموماً شيء جيد.

إليك محتوى الملف *app/webhooks.py*:

```python
import hmac
from flask import request, Blueprint, jsonify, current_app 
from git import Repo

webhook = Blueprint('webhook', __name__, url_prefix='')

@webhook.route('/github', methods=['POST']) 
def handle_github_hook(): 
 """ Entry point for github webhook """

  signature = request.headers.get('X-Hub-Signature') 
  sha, signature = signature.split('=')

  secret = str.encode(current_app.config.get('GITHUB_SECRET'))

  hashhex = hmac.new(secret, request.data, digestmod='sha1').hexdigest()
  if hmac.compare_digest(hashhex, signature): 
    repo = Repo(current_app.config.get('REPO_PATH')) 
    origin = repo.remotes.origin 
    origin.pull('--rebase')

    commit = request.json['after'][0:6]
    print('Repository updated with commit {}'.format(commit))
  return jsonify({}), 200
```

ستُنشئ الشيفرة بدايةً مخططاً جديداً وتسميه *webhook*. ثم ستقوم بإضافة موِجه جديد لهذا المخطط باستخدام المُزخرِف (decorator) `route`. سيُستدعى هذا الموجه عندما يأتي أي طلب من النوع POST إلى الرابط *github/*.

### التحقق من الطلب

عندما تستقبل الخدمة طلبات على هذا الموجه، يجب عليها التحقق أولاً من أنَّ الطلب قادم من جيتهاب ومن المستودع الصحيح. يُرسِل موقع جيتهاب توقيع إلكتروني (signature) في ترويسة (header) الطلب *X-Hub-Signature*. يولَّد هذا التوقيع بواسطة كلمة سريّة (الموجودة في المتغيّر GITHUB_SECRET)، بعدها تُرجِع المكتبة [HMAC](https://en.wikipedia.org/wiki/HMAC) القيمة السداسيّة العشريّة (hex digest) من جسم الطلب، ومن ثم تُهشَر (hashed) بواسطة دالة الهاش (hash) `sha1`.
للتحقق من الطلب، تحتاج الخدمة إلى حساب التوقيع الإلكتروني محلياً ومقارنته مع التوقيع الآتي من ترويسة الطلب. هذا يتم عبر استخدام الدالة `hmac.compare_digest`.

### تحديث المستودع
بعد التحقق من الطلب، أصبح بالإمكان معالجته. يستخدم هذا الدرس الوحدة [GitPython](https://gitpython.readthedocs.io/en/stable/index.html) للعمل مع مستودع جيت. من الوحدة `GitPython` استدعينا الكائن `Repo` للوصول إلى المستودع البعيد المُسمى `origin` (اسم المتغيّر الذي حُفِظَت فيه جلسة الوصول للمستودع). ستقوم الخدمة بسحب آخر التغييرات محلياً من المستودع `origin`، كما ستستخدم أيضاً الخيار *rebase–* لتفادي مشكلات التداخل (merges).

ستطبع الدالة `print` نصاً لأغراض تنقيحيّة يحوي الهاش القصير للإيداع المُستلم من جسم الطلب. يشرح هذا المثال كيفيّة استخدام جسم الطلب. للمزيد من التفاصيل حول البيانات المتاحة في جسم الطلب، اطلع على [توثيق جيتهاب للمطورين](https://developer.github.com/v3/activity/events/types/#webhook-payload-example-26).

في النهاية، ستُعيد الخدمة سلسلة جسون (JSON) نصيّة فارغة ورقم الحالة (status code) 200. هذا يُخبر خادم خُطاف جيتهاب أنَّ الطلب اُستقبِل بنجاح.

## نشر الخدمة
سنستخدم خادم الويب [gunicorn](http://gunicorn.org/) لتشغيل الخدمة في هذا المثال. بدايةً قم بتثبيت الإعتماديات. يمكنك استخدم هذا الأمر [بصلاحية الجذر](https://fedoramagazine.org/howto-use-sudo/) على خادم فيدورا مدعوم:

```
sudo dnf install python3-gunicorn python3-flask python3-GitPython
```

الآن قم بتعديل الملف *wsgi.py* المُستخدم من قبل الخادم *gunicorn* لتشغيل الخدمة:

```python
from app import create_app

application = create_app()
```

لتجربة هذه الخدمة، قم باستنساخ (clone) [هذا المستودع](https://github.com/cverna/github_hook_deployment.git) أو مستودعك الخاص باستخدام هذا الأمر:

```
git clone https://github.com/cverna/github_hook_deployment.git /opt/
```

الخطوة التالية هي أن تقوم بإعداد متغيّرات البيئة (environment variables) المُستخدمة من قبل الخدمة. قم بتشغيل الأمرين التاليين:

```
export GITHUB_SECRET=asecretpassphraseusebygithubwebhook
export REPO_PATH=/opt/github_hook_deployment/
```

يستخدم هذا الدرس [مستودع الجيتهاب هذا](https://github.com/cverna/github_hook_deployment)، ولكن يمكنك إن أردت استخدام أي مستودع آخر تريد. وفي النهاية، قم بتشغيل خادم الويب باستخدام الأمرين التاليين:

```
cd /opt/github_hook_deployment/
gunicorn --bind 0.0.0.0 wsgi:application --reload
```

ستقوم هذه الخيارات ببدء خادم الويب على عنوان الشبكة 0.0.0.0، هذا يعني أنَّه سيستقبل الطلبات القادمة من أي جهاز (مضيف). يُستخدم الخيار *reload–* لإعادة تشغيل الخادم عند أي تغييرات تحصل على شيفرة الخدمة. وهنا يحصل سحر النشر المتواصل! في كل مرَّة يُستقبَل طلب من جيتهاب، ستُسحب آخر التحديثات إلى المستودع المحلي، وبالتالي سيكتشف الخادم هذه التغييرات، فسيقوم بإعادة تشغيل التطبيق تلقائياً.

**ملاحظة**: لتلقي الطلبات من جيتهاب، على خادم الويب (برمجيّة الخادم) أن يكون موجوداً على خادم (جهاز) يملك عنوان شبكة عمومي (public IP). كطريقة سهلة للقيام بذلك، يمكنك استخدام مزود السحابة المُفضَل لديك، مثل: DigitalOcean ،AWS ،Linode.

## إعداد مستودع الجيتهاب
آخر جزء من الدرس هو إعداد مستودع الجيتهاب لإرسال الخُطاف إلى الخدمة.

من إعدادات مستودعك، اختر الخيار *Webhook* من القائمة واضغط على *Add Webhook*. ادخل المعلومات التالية في الحقول:

<ul style='list-style-type: disc; list-style-position: inside;'>
  <li>الخيار <b>Payload URL</b> (عنوان الحمولة): رابط الخدمة، على سبيل المثال <i>http://public_ip_address:8000/github</i>.</li>
  <li>الخيار <b>Content type</b> (نوع المحتوى المُرسَل): اختر <i>application/json</i>.</li>
  <li>الخيار <b>Secret</b> (الكلمة السريّة): قيمة متغيّر البيئة <b>GITHUB_SECRET</b> الذي عرفناه سابقاً.</li>
</ul>

ثم اضغط على الزر *Add Webhook*.

<img src='https://fedoramagazine.org/wp-content/uploads/2018/03/Screenshot-2018-3-26-cverna-github_hook_deployment1.png'/>

سيُرسِل موقع جيتهاب الآن طلب في كل مرَّة تُدفَع تغييرات جديدة إلى الخادم.

## الخاتمة
يوضِّح لك هذا الدرس كيفيّة كتابة خدمة ويب تعتمد على فلاسك لاستقبال الطلبات من خدمة الخطاطيف الشابكيّة الخاصة بجيتهاب، وكيفيّة إجراء عمليّة نشر متواصِل. انطلاقاًَ من هذا الدرس، ينبغي أن تكون قادراً على بناء خدمات ويب مفيدة.
