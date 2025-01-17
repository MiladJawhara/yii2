Моделі
======

Моделі є частиною архітектури [MVC](http://uk.wikipedia.org/wiki/Модель-вид-контролер).
Це об’єкти, які представляють бізнес-дані, бізнес-правила та бізнес-логіку.

Ви можете створювати класи моделей шляхом розширення класу [[yii\base\Model]] або його нащадків. Базовий клас
[[yii\base\Model]] підтримує багато корисних можливостей:

* [Атрибути](#attributes): представляють бізнес-дані та можуть бути доступними як звичайні властивості об’єкту
  або як елементи масиву;
* [Мітки атрибутів](#attribute-labels): визначають мітки, які використовуються при відображенні атрибутів;
* [Масове призначення](#massive-assignment): підтримується заповнення декількох атрибутів одним кроком;
* [Правила перевірки](#validation-rules): забезпечують ввід даних на основі оголошених правил перевірки;
* [Експортування даних](#data-exporting): дозволяє даним моделі бути експортованими у масиви з налаштуванням форматів.

Клас `Model` також є базовим класом для більш розширених моделей, таких як [Active Record](db-active-record.md).
Будь ласка, зверніться до відповідної документації для більш детальної інформації про ці розширені моделі.

> Info: Необов’язково створювати класи моделей на базі класу [[yii\base\Model]]. Однак, оскільки багато компонентів Yii
побудовані так, щоб підтримувати [[yii\base\Model]], краще використовувати його як базовий клас для моделей.


## Атрибути <span id="attributes"></span>

Моделі представляють бізнес-дані у вигляді *атрибутів*. Кожний атрибут є публічно доступною властивістю
моделі. Метод [[yii\base\Model::attributes()]] визначає, які атрибути має клас моделі.

Ви можете отримати доступ до атрибута як до звичайної властивості об’єкту:

```php
$model = new \app\models\ContactForm;

// "name" - це атрибут моделі ContactForm
$model->name = 'example';
echo $model->name;
```

Ви також можете отримувати доступ до атрибутів як до елементів масиву, завдяки підтримці
[ArrayAccess](https://www.php.net/manual/en/class.arrayaccess.php) та [ArrayIterator](https://www.php.net/manual/en/class.arrayiterator.php)
у класі [[yii\base\Model]]:

```php
$model = new \app\models\ContactForm;

// доступ до атрибутів як до елементів масиву
$model['name'] = 'example';
echo $model['name'];

// перебір атрибутів
foreach ($model as $name => $value) {
    echo "$name: $value\n";
}
```


### Визначення атрибутів <span id="defining-attributes"></span>

За замовчуванням, якщо ваш клас моделі успадкований безпосередньо від [[yii\base\Model]], усі його *не статичні публічні*
змінні є атрибутами. Наприклад, клас моделі `ContactForm`, наведений нижче, має чотири атрибути: `name`, `email`,
`subject` і `body`. Модель `ContactForm` використовується для репрезентації вхідних даних, отриманих з HTML-форми.

```php
namespace app\models;

use yii\base\Model;

class ContactForm extends Model
{
    public $name;
    public $email;
    public $subject;
    public $body;
}
```


Ви можете перевизначити метод [[yii\base\Model::attributes()]] для визначення атрибутів в інший спосіб. Метод повинен
повертати імена атрибутів у моделі. Наприклад, [[yii\db\ActiveRecord]] повертає
імена колонок відповідної таблиці бази даних як імена атрибутів. Також можливо знадобиться перевизначити
магічні методи, такі як `__get()` і `__set()`, щоб атрибути могли бути доступними як
звичайні властивості об’єкту.


### Мітки атрибутів <span id="attribute-labels"></span>

При відображенні значень чи при отриманні вводу для атрибутів, часто необхідно відобразити деякі написи, пов’язані з
атрибутами. Наприклад, якщо атрибут має ім’я `firstName`, ви можете відобразити мітку `First Name`, яка є більш зручною
для користувача при відображенні кінцевим користувачам у таких місцях як поля форми чи повідомлення про помилки.

Ви можете отримати мітку атрибуту викликом методу [[yii\base\Model::getAttributeLabel()|getAttributeLabel()]]. Наприклад,

```php
$model = new \app\models\ContactForm;

// відобразить "Name"
echo $model->getAttributeLabel('name');
```

За замовчуванням мітки атрибутів генеруються автоматично з імен атрибутів. Генерування виконується
методом [[yii\base\Model::generateAttributeLabel()|generateAttributeLabel()]]. Він перетворить імена змінних зі стилю
"camel-case" у декілька слів з першою літерою у кожному слові у верхньому регістрі. Наприклад, `username` стане
`Username`, а `firstName` стане `First Name`.

Якщо ви не хочете використовувати автоматично згенеровані мітки, ви можете перевизначити
[[yii\base\Model::attributeLabels()|generateAttributeLabel()]] для точного визначення міток атрибутів. Наприклад,

```php
namespace app\models;

use yii\base\Model;

class ContactForm extends Model
{
    public $name;
    public $email;
    public $subject;
    public $body;

    public function attributeLabels()
    {
        return [
            'name' => 'Your name',
            'email' => 'Your email address',
            'subject' => 'Subject',
            'body' => 'Content',
        ];
    }
}
```

Для додатків, що підтримують декілька мов, ви можливо захочете перекласти мітки атрибутів. Це можливо зробити
у методі [[yii\base\Model::attributeLabels()|attributeLabels()]] також, як показано нижче:

```php
public function attributeLabels()
{
    return [
        'name' => \Yii::t('app', 'Your name'),
        'email' => \Yii::t('app', 'Your email address'),
        'subject' => \Yii::t('app', 'Subject'),
        'body' => \Yii::t('app', 'Content'),
    ];
}
```

Ви можете навіть умовно визначати мітки атрибутів. Наприклад, на основі [сценарію](#scenarios), в якому модель
використовується, ви можете повертати різні мітки для одного й того ж атрибуту.

> Info: Строго кажучи, мітки атрибутів є частиною [представлень](structure-views.md). Але оголошення міток
у моделях є часто дуже зручним та призводить до чистоти коду та можливості його повторного використання.


## Сценарії <span id="scenarios"></span>

Модель може бути використана у різних *сценаріях*. Наприклад, модель `User` може бути використана для збору вводу при вході користувача,
але також вона може бути використана з метою реєстрації користувача. У різних сценаріях модель може використовувати різні
бізнес-правила та бізнес-логіку. Наприклад, атрибут `email` може бути обов’язковим у процесі реєстрації користувача,
але не потрібним у процесі входу користувача.

Модель використовує властивість [[yii\base\Model::scenario]] для того, щоб відслідковувати сценарій, в якому вона використовується.
За замовчуванням модель підтримує тільки один сценарій іменований `default`. Наступний код показує два шляхи
призначення сценарію моделі:

```php
// сценарій призначено як властивість
$model = new User;
$model->scenario = 'login';

// сценарій призначено через конфігурацію
$model = new User(['scenario' => 'login']);
```

За замовчуванням сценарії, які підтримуються моделлю, обумовлюються [правилами перевірки](#validation-rules) оголошеними
у моделі. Однак, ви можете змінити цю поведінку, перевизначивши метод [[yii\base\Model::scenarios()|scenarios()]]
як показано нижче:

```php
namespace app\models;

use yii\db\ActiveRecord;

class User extends ActiveRecord
{
    public function scenarios()
    {
        return [
            'login' => ['username', 'password'],
            'register' => ['username', 'email', 'password'],
        ];
    }
}
```

> Info: У вищенаведеному та у наступних прикладах класи моделей успадковані від [[yii\db\ActiveRecord]],
тому що використання декількох сценаріїв зазвичай відбувається з класами [Active Record](db-active-record.md).

Метод `scenarios()` повертає масив, ключами якого є імена сценаріїв, а значеннями - відповідні
*активні атрибути*. Активні атрибути можуть бути [масово призначеними](#massive-assignment) та підлягають
[перевірці](#validation-rules). У вищенаведеному прикладі, атрибути `username` і `password` є активними
у сценарії `login`; в той час як у сценарії `register`, також активним є атрибут `email` разом з `username` і `password`.

Стандартна реалізація методу `scenarios()` повертає усі сценарії, знайдені у правилах перевірки, оголошених
методом [[yii\base\Model::rules()||rules()]]. При перевизначенні методу [[yii\base\Model::scenarios()||scenarios()]],
якщо ви хочете ввести нові сценарії на додачу до тих сценаріїв, необхідно написати код подібний до наступного:

```php
namespace app\models;

use yii\db\ActiveRecord;

class User extends ActiveRecord
{
    public function scenarios()
    {
        $scenarios = parent::scenarios();
        $scenarios['login'] = ['username', 'password'];
        $scenarios['register'] = ['username', 'email', 'password'];
        return $scenarios;
    }
}
```

Можливості сценаріїв в основному використовуються при [перевірці](#validation-rules) та [масовому призначенні](#massive-assignment) атрибутів.
Однак, ви можете використовувати їх для інших цілей. Наприклад, ви можете оголошувати [мітки атрибутів](#attribute-labels)
по-різному, в залежності від поточного сценарію.


## Правила перевірки <span id="validation-rules"></span>

Дані для моделі, які отримуються від кінцевих користувачів, повинні пройти перевірку, щоб пересвідчитись, що вони задовольняють
певні правила (названі *правилами перевірки*, також відомі як *бізнес-правила*). Наприклад, дано модель `ContactForm`,
ви, можливо, хочете пересвідчитись, що всі атрибути заповнені та атрибут `email` містить коректну адресу електронної пошти.
Якщо значення для деяких атрибутів не задовольняють відповідні бізнес-правила, то будуть відображенні належні
повідомлення про помилки, щоб допомогти користувачу виправити їх.

Ви можете викликати [[yii\base\Model::validate()|validate()]] для перевірки отриманих даних. Даний метод використає
правила перевірки, оголошені у [[yii\base\Model::rules()|rules()]], для перевірки кожного необхідного атрибуту. При відсутності
помилок він поверне `true`. В іншому випадку, він збереже помилки у властивості [[yii\base\Model::errors]]
та поверне `false`. Наприклад:

```php
$model = new \app\models\ContactForm;

// заповнення атрибутів моделі даними від користувача
$model->attributes = \Yii::$app->request->post('ContactForm');

if ($model->validate()) {
    // усі дані є коректними
} else {
    // невдала перевірка: $errors - масив, що містить повідомлення про помилки
    $errors = $model->errors;
}
```


Для оголошення правил перевірки, пов’язаних з моделлю, необхідно перевизначити метод [[yii\base\Model::rules()|rules()]],
повернувши правила, які атрибути моделі повинні задовольнити. Наступний приклад показує правила перевірки, оголошені
для моделі `ContactForm`:

```php
public function rules()
{
    return [
        // атрибути name, email, subject і body є обов’язковими
        [['name', 'email', 'subject', 'body'], 'required'],

        // атрибут email повинен бути коректною адресою електронної пошти
        ['email', 'email'],
    ];
}
```

Правило може використовуватись для перевірки одного або кількох атрибутів, також атрибут може перевірятись одним або кількома правилами.
Будь ласка, зверніться до розділу [Перевірка вводу](input-validation.md) для більш детальної інформації про те, як оголошувати
правила перевірки.

Іноді необхідно, щоб правила застосовувались лише у певних [сценаріях](#scenarios). Для цього ви можете
визначати властивість `on` для правила, як наведено нижче:

```php
public function rules()
{
    return [
        // username, email і password є обов’язковими у сценарії "register"
        [['username', 'email', 'password'], 'required', 'on' => 'register'],

        // username і password є обов’язковими у сценарії "login"
        [['username', 'password'], 'required', 'on' => 'login'],
    ];
}
```

Якщо ви не визначите властивість `on`, то правило буде застосоване у всіх сценаріях. Правило називається
*активним правилом*, якщо воно може бути застосоване у поточному [[yii\base\Model::scenario|сценарії]].

Атрибут буде перевірятись тоді й лише тоді, якщо він є активним атрибутом, оголошеним у `scenarios()` і
пов’язаний з одним або кількома активними правилами, оголошеними у `rules()`.


## Масове призначення <span id="massive-assignment"></span>

Масове призначення - це зручний спосіб заповнення моделі даними, введеними користувачем, використовуючи один рядок коду.
Атрибути моделі заповнюються шляхом призначення вхідних даних безпосередньо властивості [[yii\base\Model::$attributes]].
Наступні два приклади коду є еквівалентними, обидва намагаються призначити дані з форми, представлені кінцевими користувачами,
атрибутам моделі `ContactForm`. Явно, що перший приклад, який використовує масове призначення, є набагато чистішим
й менш схильним до помилок, ніж другий:

```php
$model = new \app\models\ContactForm;
$model->attributes = \Yii::$app->request->post('ContactForm');
```

```php
$model = new \app\models\ContactForm;
$data = \Yii::$app->request->post('ContactForm', []);
$model->name = isset($data['name']) ? $data['name'] : null;
$model->email = isset($data['email']) ? $data['email'] : null;
$model->subject = isset($data['subject']) ? $data['subject'] : null;
$model->body = isset($data['body']) ? $data['body'] : null;
```


### Безпечні атрибути <span id="safe-attributes"></span>

Масові призначення застосовуються лише до, так званих *безпечних атрибутів*, які є атрибутами, що перелічені у
[[yii\base\Model::scenarios()]] для поточного [[yii\base\Model::scenario|сценарію]] моделі.
Наприклад, якщо модель `User` має нижченаведене оголошення сценарію, потім, коли поточним сценарієм
буде `login`, лише `username` і `password` можуть бути масово призначеними. Усі інші атрибути
будуть проігноровані.

```php
public function scenarios()
{
    return [
        'login' => ['username', 'password'],
        'register' => ['username', 'email', 'password'],
    ];
}
```

> Info: Причиною того, що масове призначення застосовується лише для безпечних атрибутів є необхідність
  контролювати те, які атрибути можуть змінюватись даними від кінцевого користувача. Наприклад, якщо модель `User`
  має атрибут `permission`, який визначає дозволи призначені користувачу, то необхідно бути впевненим,
  що цей атрибут може змінюватись лише адміністраторами через back-end інтерфейс.

Стандартна реалізація методу [[yii\base\Model::scenarios()|scenarios()]] повертає усі сценарії та атрибути,
знайдені у [[yii\base\Model::rules()|rules()]], якщо ви не перевизначили цей метод. Це означає, що атрибут є безпечним,
поки знаходиться в одному із активних правил перевірки.

З цієї причини, надається спеціальний валідатор з псевдонімом `safe`, що дозволяє оголошувати атрибут
безпечним без фактичної його перевірки. Наприклад, наступне правило оголошує обидва атрибути `title`
і `description` безпечними:

```php
public function rules()
{
    return [
        [['title', 'description'], 'safe'],
    ];
}
```


### Небезпечні атрибути <span id="unsafe-attributes"></span>

Як описано вище, метод [[yii\base\Model::scenarios()]] слугує двом цілям: вирішенню, які атрибути
повинні бути перевірені, і вирішенню, які атрибути є безпечними. У деяких рідких випадках, існує необхідність перевірити
атрибут, але не позначати його безпечним. Цього можна досягнути за допомогою префіксу знак оклику `!` у імені
атрибуту при його оголошені в `scenarios()`, як у атрибута `secret` в наведеному прикладі:

```php
public function scenarios()
{
    return [
        'login' => ['username', 'password', '!secret'],
    ];
}
```

Коли модель буде присутня у сценарії `login`, усі три атрибути будуть перевірені. Однак, лише атрибути
`username` і `password` можуть бути масово призначеними. Для призначення атрибуту `secret` вхідного значення
необхідно зробити це явно, як наведено нижче:

```php
$model->secret = $secret;
```


## Експортування даних <span id="data-exporting"></span>

Часто існує потреба експортувати моделі у різні формати. Наприклад, може виникнути необхідність в перетворенні набору
моделей у формат JSON або Excel. Процес експортування може бути розділений на два незалежні кроки.
Першим кроком, моделі перетворюються у масиви; другим кроком, масиви перетворюються у
відповідні формати. Ви можете зосередитись лише на першому кроці, оскільки другий крок можна виконати за допомогою
загальних форматтерів даних, одним з яких є [[yii\web\JsonResponseFormatter]].

Найпростіший спосіб перетворення моделі у масив - це використання властивості [[yii\base\Model::$attributes]].
Наприклад:

```php
$post = \app\models\Post::findOne(100);
$array = $post->attributes;
```

За замовчуванням властивість [[yii\base\Model::$attributes]] поверне значення *усіх* атрибутів,
оголошених у [[yii\base\Model::attributes()]].

Більш гнучкий та потужний спосіб перетворення моделі в масив - використання методу [[yii\base\Model::toArray()]].
Його типова поведінка така ж як у [[yii\base\Model::$attributes]]. Однак, він дозволяє обирати,
які елементи даних, що називаються *полями*, включати до масиву та як вони повинні бути форматовані.
Насправді, це стандартний спосіб експортування моделей при розробці веб-сервісів RESTful, як описано у
розділі [Форматування відповіді](rest-response-formatting.md).


### Поля <span id="fields"></span>

Поле - це просто іменований елемент у масиві, отриманому через виклик методу [[yii\base\Model::toArray()]]
моделі.

За замовчуванням імена полів є еквівалентними іменам атрибутів. Однак, можливо змінити цю поведінку за допомогою перевизначення
методів [[yii\base\Model::fields()|fields()]] і/або [[yii\base\Model::extraFields()|extraFields()]]. Обидва методи
повинні повертати перелік визначень полів. Поля, визначені у `fields()` є полями за замовчуванням, це означає, що
`toArray()` повертатиме ці поля за замовчуванням. Метод `extraFields()` визначає додатково доступні поля,
які також можуть бути повернутими через `toArray()`, якщо будуть зазначені у параметрі `$expand`. Наприклад,
наступний код буде повертати усі поля, визначені у `fields()`, а також поля `prettyName` і `fullAddress`,
якщо вони визначені у `extraFields()`:

```php
$array = $model->toArray([], ['prettyName', 'fullAddress']);
```

Ви можете перевизначити `fields()`, щоб додати, видалити, перейменувати або перевизначити поля. Значенням, яке повертатиме `fields()`,
повинен бути масив. Ключами масиву є імена полів, а значеннями масиву - відповідні
визначення полів, які можуть бути або іменами властивостей/атрибутів, або анонімними функціями, що повертають
відповідні значення полів. В окремому випадку, коли ім’я поля збігається з ім’ям його атрибуту,
ви можете пропустити ключ масиву. Наприклад:

```php
// точний перелік кожного поля найкраще використовувати тоді, коли ви хочете бути впевненим, що зміни
// у вашій таблиці БД або у атрибутах моделі не викликають змін ваших полів (для збереження зворотної сумісності API)
public function fields()
{
    return [
        // ім’я поля збігається з ім’ям атрибуту
        'id',

        // ім’я поля - "email", ім’я відповідного атрибуту - "email_address"
        'email' => 'email_address',

        // ім’я поля - "name", його значення визначене за допомогою анонімної PHP-функції
        'name' => function () {
            return $this->first_name . ' ' . $this->last_name;
        },
    ];
}

// відфільтровування деяких полів найкраще використовувати тоді, коли ви хочете
// успадкувати батьківську реалізацію та виключити деякі "чутливі" поля
public function fields()
{
    $fields = parent::fields();

    // вилучити поля, які містять конфіденційну інформацію
    unset($fields['auth_key'], $fields['password_hash'], $fields['password_reset_token']);

    return $fields;
}
```

> Warning: Оскільки за замовчуванням усі атрибути моделі будуть включені до масиву, що експортується, ви повинні
перевірити ваші дані, щоб бути впевненим, що вони не містять конфіденційної інформації. Якщо ж така інформація присутня,
вам потрібно перевизначити метод `fields()` та відфільтрувати її. У вищенаведеному прикладі були
відфільтровані поля `auth_key`, `password_hash` і `password_reset_token`.


## Кращі практики <span id="best-practices"></span>

Моделі є центральним місцем репрезентації бізнес-даних, бізнес-правил і бізнес-логіки. Вони часто потребують повторного
використання у різних місцях. В добре організованому додатку моделі зазвичай набагато більші, ніж
[контролери](structure-controllers.md).

В цілому, моделі

* можуть містити атрибути для репрезентації бізнес-даних;
* можуть містити правила перевірки для забезпечення коректності та цілісності даних;
* можуть містити методи, які реалізують бізнес-логіку;
* НЕ повинні безпосередньо отримувати доступ до запиту, сесії або будь-яких інших даних середовища. Ці дані повинні бути введені
  у моделі за допомогою [контролерів](structure-controllers.md);
* повинні уникати вкладеного HTML- або іншого презентаційного коду - це краще робити у [представленнях](structure-views.md);
* мають уникати завеликої кількості [сценаріїв](#scenarios) в одній моделі.

Як правило, ви можете враховувати останню рекомендацію, з наведених вище, тоді, коли розробляєте великі складні системи.
У цих системах моделі можуть бути дуже великими, оскільки вони використовуються у багатьох місцях й тому можуть містити
багато наборів правил і бізнес-логіки. Це часто перетворюється у кошмар під час супроводу коду моделі,
через те, що невелика зміна коду може зачіпати декілька різних місць. Щоб полегшити супровід коду моделі,
вам необхідно дотримуватися наступної стратегії:

* Визначити набір базових класів моделей, які є спільними для різних [додатків](structure-applications.md) або
  [модулів](structure-modules.md). Ці класи моделей повинні містити мінімальний набір правил та логіки, які
  є загальними для усіх додатків та модулів, які їх використовують.
* У кожному [додатку](structure-applications.md) або [модулі](structure-modules.md), який використовує модель,
  визначити конкретний клас моделі, що розширює відповідний базовий клас моделі. Конкретні класи моделей
  повинні містити правила та логіку, які є специфічними для додатка чи модуля.

Наприклад, у [розширеному шаблоні проекту](https://github.com/yiisoft/yii2-app-advanced/blob/master/docs/guide-uk/README.md), ви можете визначати базовий клас
моделі `common\models\Post`. Потім для front-end додатку ви визначаєте та використовуєте конкретний клас моделі
`frontend\models\Post`, який розширює клас `common\models\Post`. Аналогічно для back-end додатку,
ви визначаєте `backend\models\Post`. Подібна стратегія дає вам впевненість в тому, що код у `frontend\models\Post`
є специфічним тільки для front-end додатку, та якщо ви внесете будь-які зміни до нього, то не має потреби турбуватись, що
зміни можуть порушити back-end додаток.
