# Clean code PHP

Многое в этом документе честно [спизжено из репозитория jupeter/clean-code-php](https://github.com/jupeter/clean-code-php),
что-то дописывал [Sol](https://github.com/solbianca). 

## Содержание 
  1. [Введение](#введение)
  2. [Переменные](#переменные)
  3. [Функции](#функции)
  4. [Классы и объекты](#классы-и-объекты)
  5. [Общие советы](#общие-советы)

## Введение
Для команды, работающей над общим кодом, важно придерживаться общих стандартов и принципов. Они помогают гарантировать, что код понятен каждому разработчику, имеет меньше ошибок и легче подвержен изменению.

Каждый разработчик должен придерживаться принятого стиля, даже если он ему не нравится или он с ним не согласен. Данные принципы и рекомендации работают только тогда, когда вся команда их соблюдает.

Даже если разработчик работает над изолированным компонентом и никто кроме него туда код не пишет. Когда понадобится внести изменения, то разработчик может быть занят другой задачей, заболеть, уйти в отпуск или не работать в компании. И команде нужно будет поддерживать и дорабатывать этот данный код.


## Переменные

### Используйте осмысленные и говорящие имена

Имена переменных, функций, классов и тд должны отражать суть. Используйте целые слова - избегайте аббревиатур и сокращений, только если аббревиатура не является общеупотребимой, например URL и HTML. При использовании аббревиатуры, только первая буква пишется в верхнем регистре, остальные в нижнем.

Bad:

```php
$ymdstr = $date->format('y-m-d');
$usrTkn;
$preparedHTMLString;
$preparedHypertextMarkupLanguageString;
```

Good:

```php
$currentDate = $date->format('y-m-d');
$userToken;
$preparedHtmlString;
```

**[⬆ Назад к Содержанию](#содержание)**

### Давайте переменным имена, которые легко искать

Bad:

```php
// Что тут значит 448?
$result = $serializer->serialize($data, 448);
Good:

$json = $serializer->serialize($data, JSON_UNESCAPED_SLASHES | JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE);
Bad:

// Что значит 2 для роли?
$user->updateRole(2);
```

Good:

```php
$user->updateRole(User::ROLE_MODERATOR);

class User
{
    const ROLE_USER = 1;
    const ROLE_MODERATOR = 2;
    const ROLE_ADMIN = 4;
    const ROLE_SUPERADMIN = 8;
}
```

**[⬆ Назад к Содержанию](#содержание)**

### Избегайте ментального сопоставления

Не заставляйте тех, кто будет читать ваш код, переводить значения переменных. Лучше писать явно, чем неявно.

Bad

```php
$l = ['Austin', 'New York', 'San Francisco'];

for ($i = 0; $i < count($l); $i++) {
    $li = $l[$i];
    doStuff();
    // ...
    // много строчек кода
    // ...
    // Что же тут такое `$li`?
    dispatch($li);
}
```

Good

```php
$locations = ['Austin', 'New York', 'San Francisco'];

foreach ($locations as $location) {
    doStuff();
    // ...
    // много строчек кода
    // ...
    dispatch($location);
});
```

**[⬆ Назад к Содержанию](#содержание)**

### Не добавляйте ненужный контекст

Если имя вашего класса/объекта с чем-то у вас ассоциируется, не проецируйте эту ассоциацию на имя переменной.

Bad

```php
class Car
{
    public $carMake;
    public $carModel;
    public $carColor;
}
```

Good

```php
class Car
{
    public $make;
    public $model;
    public $color;
}
```

**[⬆ Назад к Содержанию](#содержание)**

### Не используйте числа с плавающей точкой где требуются целые значения

Никогда не доверяйте точности чисел с плавающей точкой до последней цифры, и не проверяйте напрямую их равенство.

[http://php.net/manual/ru/language.types.float.php](http://php.net/manual/ru/language.types.float.php)

**[⬆ Назад к Содержанию](#содержание)**

### Используйте DateTime класс
Этот класс инкапсулирует все функциональные возможности старых функций даты в один простой в использовании класс. Всегда используйте класс DateTime для создания, сравнения, изменения и отображения дат в PHP.

- Если явным образом не указать часовой пояс в конструкторе класса, то будет использован часовой пояс компьютера, на котором вызван код. Это может приводить к не ожидаемому поведению и ошибкам. Всегда указывайте временную зону UTC.
- Значение аргумента $timezone равно как и текущая временная зона не будут учитываться, если в качестве аргумента $time передается метка времени UNIX (например @946684800) или время, в котором временная зона уже содержится (например ```2010-01-28T15:00:00+02:00```).
- Передача нулевых дат (например, «0000-00-00», значение, обычно возвращаемое MySQL в качестве значения по умолчанию в столбце DateTime) для DateTime :: __ construct () приведет к бессмысленной дате, а не «0000-00-00»

Bad

```php
date_default_timezone_set('Australia/Currie');
echo date('Y-m-d H:i:s', time());
```

Good

```php
$date = new DateTime('now', new DateTimeZone('Australia/Currie'));
echo $date->format('Y-m-d H:i:s');
```

**[⬆ Назад к Содержанию](#содержание)**

## Функции

### Функции должны делать что-то одно

Это самое важное правило в разработке. Когда функции делают больше одной вещи, их труднее составлять, тестировать и обосновывать. А если вы можете наделить функции только какими-то одиночными действиями, то их будет легче рефакторить, а ваш код станет гораздо чище. Даже если вы не будете следовать никакой другой рекомендации, кроме этой, то всё равно опередите многих других разработчиков.

Bad

```php
function emailClients(array $clients): void
{
    foreach ($clients as $client) {
        $clientRecord = $db->find($client);
        if ($clientRecord->isActive()) {
            email($client);
        }
    }
}
```

Good

```php
function emailClients(array $clients): void
{
    $activeClients = activeClients($clients);
    array_walk($activeClients, 'email');
}

function activeClients(array $clients): array
{
    return array_filter($clients, 'isClientActive');
}

function isClientActive(int $client): bool
{
    $clientRecord = $db->find($client);
    return $clientRecord->isActive();
}
```

**[⬆ Назад к Содержанию](#содержание)**

### Используйте осмысленные и говорящие имена

Имена переменных, функций, классов и тд должны отражать суть. Используйте целые слова - избегайте аббревиатур и сокращений, только если аббревиатура не является общеупотребимой, например URL и HTML. При использовании аббревиатуры, только первая буква пишется в верхнем регистре, остальные в нижнем.

Bad

```php
genStr();
generateStr();
getHTMLStatistic();
getHypertextMarkupLanguageStatistic();
```

Good

```php
generateString();
getHtmlStatistic();
```

Bad

```php
class Email
{
    //...
    public function handle(): void
    {
        mail($this->to, $this->subject, $this->body);
    }
}

$message = new Email(...);
// Не понятно что делает метод handle(). Название метода не нисет конкретной семантики, просто "обработать".
$message->handle();
```

Good

```php
class Email 
{
    //...

     public function send(): void
    {
        mail($this->to, $this->subject, $this->body);
    }
}
	
$message = new Email(...);
// Семантика ясна и очевидна
$message->send();
```

**[⬆ Назад к Содержанию](#содержание)**

### Используйте аргументы по умолчанию

Bad

```php
createUser($role = null) {
    $role = $role ?: 'user';
    ...
}
```

Good

```php
createUser(string $role = 'user') {
    ...
}
```

**[⬆ Назад к Содержанию](#содержание)**

### Не передавайте слишком много аргументов в функции

Крайне важно ограничивать количество параметров функций, потому что это упрощает тестирование. Больше трёх аргументов ведёт к "комбинаторному взрыву", когда вам нужно протестировать кучу разных случаев применительно к каждому аргументу.

Bad

```php
function createMenu(string $title, string $body, string $buttonText, bool $cancellable): void
{
    // ...
}
}
```

Good

```php
class MenuConfig
{
    public $title;
    public $body;
    public $buttonText;
    public $cancellable = false;
}

$config = new MenuConfig();
$config->title = 'Foo';
$config->body = 'Bar';
$config->buttonText = 'Baz';
$config->cancellable = true;

function createMenu(MenuConfig $config): void
{
    // ...
}
```

**[⬆ Назад к Содержанию](#содержание)**

### Уменьшайте вложенность кода и поднимайте точки выхода

Bad

```php
function isShopOpen($day): bool
{
    if ($day) {
        if (is_string($day)) {
            $day = strtolower($day);
            if ($day === 'friday') {
                return true;
            } elseif ($day === 'saturday') {
                return true;
            } elseif ($day === 'sunday') {
                return true;
            } else {
                return false;
            }
        } else {
            return false;
        }
    } else {
        return false;
    }
}
```

Good

```php
function isShopOpen(string $day): bool
{
    if (empty($day)) {
        return false;
    }

    $openingDays = [
        'friday', 'saturday', 'sunday'
    ];

    return in_array(strtolower($day), $openingDays);
}
```

Bad

```php
function fibonacci(int $n)
{
    if ($n < 50) {
        if ($n !== 0) {
            if ($n !== 1) {
                return fibonacci($n - 1) + fibonacci($n - 2);
            } else {
                return 1;
            }
        } else {
            return 0;
        }
    } else {
        return 'Not supported';
    }
}
```

Good

```php
function fibonacci(int $n): int
{
    if ($n === 0) {
        return 0;
    }

    if ($n === 1) {
        return 1;
    }

    if ($n > 50) {
        throw new \Exception('Not supported');
    }

    return fibonacci($n - 1) + fibonacci($n - 2);
}
```

**[⬆ Назад к Содержанию](#содержание)**

### Избегайте условных выражений

Выше было рассмотрено правило чистого кода: функция должна делать что-то одно. Если у вас есть классы и функции, содержащие выражение if, то тем самым вы говорите своим пользователям, что функция делает больше одной вещи. Не забывайте — нужно оставить что-то одно.

Bad

```php
class Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        switch ($this->type) {
            case '777':
                return $this->getMaxAltitude() - $this->getPassengerCount();
            case 'Air Force One':
                return $this->getMaxAltitude();
            case 'Cessna':
                return $this->getMaxAltitude() - $this->getFuelExpenditure();
        }
    }
}
```

Good

```php
interface Airplane
{
    // ...

    public function getCruisingAltitude(): int;
}

class Boeing777 implements Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude() - $this->getPassengerCount();
    }
}

class AirForceOne implements Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude();
    }
}

class Cessna implements Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude() - $this->getFuelExpenditure();
    }
}
```

**[⬆ Назад к Содержанию](#содержание)**

### Не используйте флаги в качестве параметров функций

Флаги говорят вашим пользователям, что функции делают больше одной вещи. А они должны делать что-то одно. Разделяйте свои функции, если они идут по разным ветвям кода.

Bad

```php
function createFile(string $name, bool $temp = false): void
{
    if ($temp) {
        touch('./temp/'.$name);
    } else {
        touch($name);
    }
}
```

Good

```php
function createFile(string $name): void
{
    touch($name);
}

function createTempFile(string $name): void
{
    touch('./temp/'.$name);
}
```

**[⬆ Назад к Содержанию](#содержание)**

### Избегайте побочных эффектов

Функция может привносить побочные эффекты, если она не только получает значение и возвращает другое значение/значения, но и делает что-то ещё. Побочным эффектом может быть запись в файл, изменение глобальной переменной или случайная отправка всех ваших денег незнакомому человеку.

Но иногда побочные эффекты бывают нужны. Например, та же запись в файл. Лучше делать это централизованно. Не выбирайте несколько функций и классов, которые пишут в какой-то один файл, используйте для этого единый сервис. Единственный.

Главная задача — избежать распространённых ошибок вроде общего состояния для нескольких объектов без какой-либо структуры; использования изменяемых типов данных, которые могут быть чем-то перезаписаны; отсутствия централизованной обработки побочных эффектов. Если вы сможете это сделать, то будете счастливее подавляющего большинства других программистов.

Bad

```php
// Global variable referenced by following function.
// If we had another function that used this name, now it'd be an array and it could break it.
$name = 'Ryan McDermott';

function splitIntoFirstAndLastName(): void
{
    global $name;

    $name = explode(' ', $name);
}

splitIntoFirstAndLastName();

var_dump($name); // ['Ryan', 'McDermott'];
```

Good

```php
function splitIntoFirstAndLastName(string $name): array
{
    return explode(' ', $name);
}

$name = 'Ryan McDermott';
$newName = splitIntoFirstAndLastName($name);

var_dump($name); // 'Ryan McDermott';
var_dump($newName); // ['Ryan', 'McDermott'];
```

**[⬆ Назад к Содержанию](#содержание)**

### Не пишите в глобальные функции

Засорение глобалами — дурная привычка в любом языке, потому что вы можете конфликтовать с другой библиотекой, а пользователи вашего API не будут об этом знать, пока не получат исключение в production. Приведу пример: вам нужен конфигурационный массив. Вы пишете глобальную функцию вроде config(), но она может конфликтовать с другой библиотекой, пытающейся делать то же самое.

Bad

```php
function config(): array
{
    return  [
        'foo' => 'bar',
    ]
}
```

Good

```php
class Configuration
{
    private $configuration = [];

    public function __construct(array $configuration)
    {
        $this->configuration = $configuration;
    }

    public function get(string $key): ?string
    {
        return isset($this->configuration[$key]) ? $this->configuration[$key] : null;
    }
}
```

Загрузите конфигурацию и создайте экземпляр класса Configuration.

```php
$configuration = new Configuration([
    'foo' => 'bar',
]);
```

И теперь вы должны использовать экземпляр класса Configuration в своем приложении.

**[⬆ Назад к Содержанию](#содержание)**

### Инкапсулирование условных выражений

Bad

```php
if ($article->state === 'published') {
    // ...
}
```

Good

```php
if ($article->isPublished()) {
    // ...
}
```

**[⬆ Назад к Содержанию](#содержание)**

### Избегайте негативных условных выражений

Bad

```php
function isDOMNodeNotPresent(\DOMNode $node): bool
{
    // ...
}

if (!isDOMNodeNotPresent($node))
{
    // ...
}
```

Good

```php
function isDOMNodePresent(\DOMNode $node): bool
{
    // ...
}

if (isDOMNodePresent($node)) {
    // ...
}
```

**[⬆ Назад к Содержанию](#содержание)**

### Избегайте проверки типов (часть 1)

PHP не типизирован, т. е. ваши функции могут принимать аргументы любых типов. Иногда такая свобода даже мешает и возникает соблазн выполнять проверку типов в функциях. Но есть много способов этого избежать. Первое, что нужно учитывать, это согласованные API.

Bad

```php
function travelToTexas($vehicle): void
{
    if ($vehicle instanceof Bicycle) {
        $vehicle->peddleTo(new Location('texas'));
    } elseif ($vehicle instanceof Car) {
        $vehicle->driveTo(new Location('texas'));
    }
}
```

Good

```php
function travelToTexas(Traveler $vehicle): void
{
    $vehicle->travelTo(new Location('texas'));
}
```

**[⬆ Назад к Содержанию](#содержание)**

### Избегайте проверки типов (часть 2)

Если вы работаете с базовыми примитивами (вроде строковых, целочисленных) и массивами, то не можете использовать полиморфизм. Но если кажется, что вам всё ещё нужна проверка типов и вы используете PHP 7+, то примените объявление типов или строгий режим (strict mode). Это даст вам статичную типизацию поверх стандартного PHP-синтаксиса. Проблема ручной проверки типов в том, что её качественное выполнение подразумевает такое многословие, что полученная искусственная "типобезопасность" не компенсирует потери читабельности кода. Сохраняйте чистоту своего PHP, пишите хорошие тесты и проводите качественные ревизии кода. Или делайте всё это, но со строгим объявлением PHP-типов или в строгом режиме.

Bad

```php
function combine($val1, $val2): int
{
    if (!is_numeric($val1) || !is_numeric($val2)) {
        throw new \Exception('Must be of type Number');
    }

    return $val1 + $val2;
}
```

Good

```php
function combine(int $val1, int $val2): int
{
    return $val1 + $val2;
}
```

**[⬆ Назад к Содержанию](#содержание)**

### Разбивайте на блоки код внутри методов

Связанные строки кода должны быть сгруппированы в блоки, отделенные друг от друга, чтобы обеспечить лучшую читаемость. Определение «связанности» субъективно и зависит от контекста.

Bad

```php
public function doSomething () {
    if ($foo) {
        $bar = 1;
    }
    if ($spam) {
        $ham = 1;
    }
    if ($pinky) {
           $brain = 1;
    }
}
```

Good

```php
public function doSomething () {
    if ($foo) {
         $bar = 1;
    }

    if ($spam) {
            $ham = 1;
    }

    if ($pinky) {
            $brain = 1;
    }
}
```

**[⬆ Назад к Содержанию](#содержание)**

## Классы и объекты

### Композиция лучше наследования

Как говорится в известной книге "Шаблоны проектирования" Банды четырёх, по мере возможности нужно выбирать композицию, а не наследование. Есть много хороших причин использовать как наследование, так и композицию. Главная цель этой максимы заключается в том, если вы инстинктивно склоняетесь к наследованию, то постарайтесь представить, может ли композиция лучше решить вашу задачу. В каких-то случаях это действительно более подходящий вариант.

Вы спросите: "А когда лучше выбирать наследование?" Всё зависит от конкретной задачи, но можно ориентироваться на этот список ситуаций, когда наследование предпочтительнее композиции:

- Ваше наследование — это взаимосвязь is-a, а не has-a. Пример: Человек → Животное vs. Пользователь → Профиль пользователя (Profile).
- Вы можете повторно использовать код из базовых классов. (Люди могут двигаться, как животные.)
- Вы хотите внести глобальные изменения в производные классы, изменив базовый класс. (Изменение расхода калорий у животных во время движения.)

Bad

```php
class Employee 
{
    private $name;
    private $email;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->email = $email;
    }

    // ...
}

// Bad because Employees "have" tax data. 
// EmployeeTaxData is not a type of Employee

class EmployeeTaxData extends Employee 
{
    private $ssn;
    private $salary;

    public function __construct(string $name, string $email, string $ssn, string $salary)
    {
        parent::__construct($name, $email);

        $this->ssn = $ssn;
        $this->salary = $salary;
    }

    // ...
}
```

Good

```php
class EmployeeTaxData 
{
    private $ssn;
    private $salary;

    public function __construct(string $ssn, string $salary)
    {
        $this->ssn = $ssn;
        $this->salary = $salary;
    }

    // ...
}

class Employee 
{
    private $name;
    private $email;
    private $taxData;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->email = $email;
    }

    public function setTaxData(string $ssn, string $salary)
    {
        $this->taxData = new EmployeeTaxData($ssn, $salary);
    }

    // ...
}
```

**[⬆ Назад к Содержанию](#содержание)**

### Используйте инкапсуляцию объектов

В PHP можно задать для методов ключевые слова public, protected и private. С их помощью вы будете управлять изменением свойств объекта.

* Если вам нужно не только получать свойство объекта, то необязательно находить и менять каждый метод чтения (accessor) в кодовой базе.
* Благодаря set проще добавить валидацию.
* Можно инкапсулировать внутреннее представление.
* С помощью геттеров и сеттеров легко добавлять журналирование и обработку ошибок.
* При наследовании такого класса вы можете переопределить функциональность по умолчанию.
* Вы можете лениво загружать свойства объекта, например получая их с сервера.

Также это часть принципа Открытости/Закрытости, входящего в набор объектно ориентированных принципов проектирования.

Bad

```php
class BankAccount
{
    public $balance = 1000;
}

$bankAccount = new BankAccount();

// Buy shoes...
$bankAccount->balance -= 100;
```

Good

```php
class BankAccount
{
    private $balance;

    public function __construct(int $balance = 1000)
    {
      $this->balance = $balance;
    }

    public function withdrawBalance(int $amount): void
    {
        if ($amount > $this->balance) {
            throw new \Exception('Amount greater than available balance.');
        }

        $this->balance -= $amount;
    }

    public function depositBalance(int $amount): void
    {
        $this->balance += $amount;
    }

    public function getBalance(): int
    {
        return $this->balance;
    }
}

$bankAccount = new BankAccount();

// Buy shoes...
$bankAccount->withdrawBalance($shoesPrice);

// Get balance
$balance = $bankAccount->getBalance();
```

**[⬆ Назад к Содержанию](#содержание)**

### У объектов должны быть private/protected компоненты

* public методы и свойства наиболее опасны для изменений, так как они используются внешним кодом и мы не можем контролировать где и кто использует наши публичные методы и свойства. Поэтому изменения публичных методов и свойств может иметь непредсказуемые последствия.
* protected модификатор являются столь же опасными, как и public, поскольку они доступны в рамках любого дочернего класса. Это фактически означает, что разница между public и protected является только механизмом доступа, но гарантия инкапсуляции остается неизменной. Модификации в классе опасны для всех классов потомков.
* private модификатор гарантирует, что код опасен для изменения только в границах одного класса (вы защищены от модификаций, и у вас не будет Jenga эффекта). Но это не точно, смотри пункт про изменяемые/не изменяемые объекты.

Поэтому используйте private по умолчанию и public/protected, когда вам нужно предоставить доступ для внешних классов.

Для получения дополнительной информации вы можете прочитать сообщение в блоге, написанное Fabien Potencier.

Bad

```php
class Employee
{
    public $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }
}

$employee = new Employee('John Doe');
echo 'Employee name: '.$employee->name; // Employee name: John Doe
```

Good

```php
class Employee
{
    private $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }

    public function getName(): string
    {
        return $this->name;
    }
}

$employee = new Employee('John Doe');
echo 'Employee name: '.$employee->getName(); // Employee name: John Doe
```

**[⬆ Назад к Содержанию](#содержание)**

### Изменяемые и неизменяемые объекты (часть 1)

Неизменяемыми называются объекты, чьё состояние остаётся постоянным с момента их создания. Обычно такие объекты очень просты. Если в одном месте изменить объект, то в другом могут проявиться нежелательные побочные эффекты, которые с трудом поддаются отладке. Это может произойти где угодно: в сторонних библиотеках, в структурах языка и т. д. Использование неизменяемых объектов позволит избежать подобных неприятностей.

Bad

```php
class Money 
{
    private $amount;

    public function getAmount()
    {
        return $this->amount;
    }

    public function add($amount)
    {
        $this->amount += $amount;
        return $this;
    }
}

$userAmount = Money::USD(2);
/**
 * Марк собирается отправить Алексу 2 доллара. Комиссия составляет 3%,
 * и мы прибавляем её к основному переводу.
 */
$processedAmount = $userAmount->add($userAmount->getAmount() * 0.03);
/**
 * Получаем с карты Марка для последующего перевода 2 доллара + 3% комиссии
 */
$markCard->withdraw($processedAmount);
/**
 * Отправляем Алексу 2 доллара
 */
$alexCard->deposit($userAmount);
```

Класс Money изменяемый, вместо двух долларов Алекс получит 2 доллара и 6 центов (комиссия 3%). Причина в том, что ```$userAmount``` и ```$processedAmount``` ссылаются на один и тот же объект. В данном случае рекомендуется применить неизменяемый объект.

Good

```php
final class Money 
{
    private $amount;

    public function getAmount()
    {
        return $this->amount;
    }

    public function add($amount)
    {
        return new self($this->amount + $amount, $this->currency);
    }
}

$userAmount = Money::USD(2);
/**
 * Марк собирается отправить Алексу 2 доллара. Комиссия составляет 3%,
 * и мы прибавляем её к основному переводу.
 */
$processedAmount = $userAmount->add($userAmount->getAmount() * 0.03);
/**
 * Получаем с карты Марка для последующего перевода 2 доллара + 3% комиссии
 */
$markCard->withdraw($processedAmount);
/**
 * Отправляем Алексу 2 доллара
 */
$alexCard->deposit($userAmount);
```

В этот раз Алекс получит свои два доллара без комиссии, а с Марка правильно спишут эту сумму и комиссию.

**[⬆ Назад к Содержанию](#содержание)**

### Изменяемые и неизменяемые объекты (часть 2)

В php объекты передаются по ссылке. То есть если у вас один объект хранит в закрытом свойстве ссылку на другой объект, но доступен по геттеру, то внешнее окружение может изменить внутреннее состояние данного объекта. То есть инкапсуляция в данном случае не работает. Решить данную проблему можно с помощью неизменяемых объектов.

Bad

```php
class CalendarEvent 
{
    private $date;
    public function __construct(DateTime $date) 
    {
        $this->date = $date;
    }

    public function getDate() 
    {
        return $this->date;
    }
}

$calendarEvent = new CalendarEvent(new DateTime("2014-11-13"));
$nextWeeksSimilarCalendarEvent = new CalendarEvent($calendarEvent->getDate()->modify("+1 week"));
```

В данном случае у нас изменится дата для ```$nextWeeksSimilarCalendarEvent``` и для ```$calendarEvent``` так как мы изменили объект доступный о одной ссылке.

Good

```php
class CalendarEvent 
{
    private $date;
    public function __construct(DateTimeImmutable $date) 
    {
        $this->date = $date;
    }

    public function getDate() 
    {
        return $this->date;
    }
}

$calendarEvent = new CalendarEvent(new DateTimeImmutable("2014-11-13"));
$nextWeeksSimilarCalendarEvent = new CalendarEvent($calendarEvent->getDate()->modify("+1 week"));
```

Тут мы решили проблему, воспользовавшись неизменяемым объектом, который на все методы, изменяющие состояние, возвращает новый объект.

**[⬆ Назад к Содержанию](#содержание)**

## Общие советы

### Все берется из конфига

Ничего в приложении не должно быть прибито гвоздями/захардкожено. Все должно браться из конфига. Время жизни ключа в кеше, количество просмотров страниц без капчи и тд. Все это должно браться из конфига.

**[⬆ Назад к Содержанию](#содержание)**

### Убирайте мёртвый код

Он плох так же, как и дублирующий код. Не нужно держать его в кодовой базе. Если что-то не вызывается, избавьтесь от этого! Если что, мёртвый код можно будет достать из истории версий.

**[⬆ Назад к Содержанию](#содержание)**

### Пишите самодокументирующийся код и не злоупотребляй комментариями

Необходимо писать код так, что бы было понятно что он делает без дополнительных пояснений. Достигается это за счет говорящих имен переменных, функций, классов, объектов и т.д. За счет хорошей структуры кода. Код сам по себе является лучшим комментарием и документацией. Комментарии к коду надо писать только там, где это действительно необходимо.

Комментарии имеют две большие проблемы:

* Они устаревают. Код может быть написан и переписан множество раз, и если не следить за актуальностью комментариев то они становятся бесмыслеными. И поддержание комментариев в актуальном состоянии может требовать много сил и времени.
* Они не обязывают. Если комментарий к методу говорит, что он возвращает массив, это не значит что в какой-то момент не вернется null. Поэтому даже прочитав комментарий, нужно будет проверить данный метод, что он работает без сюрпризов. Либо смирится с тем, что ваш код периодически будет работать не так, как вы того ожидали и ловить плавающие баги.

**[⬆ Назад к Содержанию](#содержание)**

### Придерживайтесь единого стиля кода

Важно что бы команда придерживалась единого стиля кода. Проект должен быть написан в едином стиле.

В общем случае за основу можно взять psr-1/2:
[http://www.php-fig.org/psr/psr-1/](http://php.net/manual/ru/language.types.float.php)
[http://www.php-fig.org/psr/psr-2/](http://php.net/manual/ru/language.types.float.php)

В репозиторий код должен попадать отформатированным по принятому стандарту. То есть разработчик перед пушем обязан привести код к принятому кодстайлу. Не отформатированный код должен отклоняться ревьювером.

**[⬆ Назад к Содержанию](#содержание)**

### Практикуйте ревью кода
Ревью кода это очень мощный механизм, который направлен решать несколько задач:

- Обмен опытом. Проводящий ревью может узнать для себя новые вещи из чужого кода, а автор — из отзыва.
- Выход за границы своей зоны ответственности, позволяющий всем участникам команды анализировать разные части кода, ознакомляясь со всей кодовой базой.
- Качество кода. Автор кода может не заметить баг или ошибку логики, но что может увидеть ревьювер.
- Создание внутренней культуры команды. Ревью — это один из основных процессов, который подразумевает наличие командной культуры.
- 
Анализ кода друг друга — один из главных путей взаимодействия между разработчиками во время обычного рабочего дня. Это должен быть вдохновляющий процесс, чтобы все участники активно участвовали в нём. Ревью не должно восприниматься обузой. Обязанностью, от которой надо поскорее отделаться. От качества ревью напрямую зависит качество продукта.

Главное правило — целью ревью является нахождение проблем в коде перед тем как он будет слит в ветку, т.е. правильность. Самая распространенная ошибка у проводящего ревью смотреть на код, основываясь на том как бы это сделал он. Обычно существует несколько путей решения одного и того же. Помимо этого, есть миллион способов написать задуманное в виде кода. Задача проводящего ревью, не сделать так, чтобы написанный код был похож на то, чтобы он написал, нет — он никогда не будет таким. Его задача — удостовериться в том, что код, написанный автором корректен. Когда нарушается это правило, вы в итоге придете к эмоциям и переживаниям, что не является тем, к чему вы стремитесь.

**[⬆ Назад к Содержанию](#содержание)**

### Что не надо делать при ревью

- Не проводите ревью формально и с минимальными комментариями
- Не нужно всего лишь указывать на ошибки, ничего к этому не добавляя. Опишите возможные проблемы и решение. Помните, ревью это способ обмена опытом и коммуникации между разработчиками а не способ ткнуть кого-то лицом в грязь.
- Не думайте, что достаточно лишь «критиковать код, а не кодера»
- Это один из самых популярных советов: критикуйте код, а не кодера. Он хорош (если вы критикуете коллегу, а не его код, то у вас проблемы). Но этого недостаточно. Написание кода — творческий процесс, в который разработчик вкладывает сердце и душу. И если вы жестоко и тупо критикуете чей-то код, то тем самым вы критикуете самого человека. Нужно уметь критиковать. Найдите более позитивные способы анализа и вносите свои предложения, как улучшить код.
- Следите за тоном
- Не переходите на личности. Фразы вроде «Почему ты сделал именно так?» воспринимаются как нападки. Избегайте фраз, смысл которых заключается в «ты неправ». Всегда будьте сдержанны и дружелюбны, избегайте неприятных фраз, демонстрирующих раздражение и заставляющих людей чувствовать, что они плохо справляются.
- Не нарушайте правило «Никаких сюрпризов»
- Создайте для команды общее документированное соглашение о pull request и стандартах кода и во время ревью опирайтесь на него, а не на своё мнение.
- Не будьте непрошеным советчиком
- Сдерживайте себя и не предлагайте внести в код все изменения, которые вы хотели бы внести. Одна из задач ревью — найти и исправить баги и проблемы в структуре кода. А вовсе не заставить автора написать код так, как это сделали бы вы. В программировании зачастую есть множество решений одной и той же проблемы. И решение автора не становится плохим просто потому что вы бы сделали по-другому.

**[⬆ Назад к Содержанию](#содержание)**
