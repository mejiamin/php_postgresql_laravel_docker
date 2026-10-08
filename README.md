# PHP, SQL & PostgreSQL, Laravel, Docker | 16 Уроков

## Уроки:

**Этап 1 - 4 урока: PHP**

1. Переменные и типы данных
2. **👉 Управление потоком и функции**
3. ООП в PHP
4. Файлы и обработка ошибок

**Этап 2 - 3 урока: SQL & PostgreSQL**

5. SELECT, INSERT, UPDATE, DELETE
6. JOIN, подзапросы, индексы
7. PHP + PostgreSQL через PDO

**Этап 3 - 6 урока: Laravel**

8. Composer и установка Laravel
9. Роутинг и контроллеры
10. Миграции и Eloquent ORM
11. REST API и валидация
12. Аутентификация и Middleware
13. Очереди и фоновые Jobs

**Этап 4 - 3 урока: Docker**

14. Образы, контейнеры, Dockerfile
15. Docker Compose
16. Докеризация Laravel + PostgreSQL

---

## Урок 2 — Управление потоком и функции

### 1. Условные операторы

`if / elseif / else` — основа ветвления:

```php
<?php

$age = 20;

if ($age < 18) {
    echo "Несовершеннолетний\n";
} elseif ($age < 65) {
    echo "Взрослый\n";
} else {
    echo "Пенсионер\n";
}
```

`match` — это PHP 8+, строгое сравнение (без приведения типов), возвращает значение. Лаконичнее `switch` и безопаснее:

```php
<?php

$status = "active";

$label = match($status) {
    "active"  => "Активен",
    "banned"  => "Заблокирован",
    "pending" => "На проверке",
    default   => "Неизвестно",
};

echo $label; // Активен
```

`switch` — старый вариант, использует нестрогое сравнение (`==`). Часто `match` предпочтительнее, но `switch` встретишь в чужом коде:

```php
<?php

$day = 3;

switch ($day) {
    case 1:
    case 2:
    case 3:
    case 4:
    case 5:
        echo "Будний день\n";
        break;
    case 6:
    case 7:
        echo "Выходной\n";
        break;
    default:
        echo "Неверный день\n";
}
```

---

### 2. Циклы

`for` — когда известно количество итераций:

```php
<?php

for ($i = 1; $i <= 5; $i++) {
    echo "Итерация $i\n";
}
```

`while` — пока условие истинно:

```php
<?php

$n = 1;
while ($n <= 100) {
    $n *= 2;
}
echo $n; // 128 — первая степень двойки, превышающая 100
```

`foreach` — для перебора массивов, используется чаще всего:

```php
<?php

$fruits = ["яблоко", "банан", "вишня"];

foreach ($fruits as $fruit) {
    echo $fruit . "\n";
}

// Ассоциативный массив — ключ => значение
$prices = ["яблоко" => 50, "банан" => 30, "вишня" => 120];

foreach ($prices as $name => $price) {
    echo "$name: $price руб.\n";
}
```

---

### 3. Функции

Объявление и вызов:

```php
<?php

function greet(string $name): string {
    return "Привет, $name!";
}

echo greet("Анна"); // Привет, Анна!
```

Параметры по умолчанию — указываются в конце сигнатуры:

```php
<?php

function calcDiscount(float $price, float $percent = 10.0): float {
    return $price * ($percent / 100);
}

echo calcDiscount(5000);       // 500   — использует 10% по умолчанию
echo calcDiscount(5000, 15.0); // 750   — передаём явно
```

Тайп-хинты (PHP 7+) — объявляй типы параметров и возвращаемого значения. Это делает код самодокументируемым и ловит ошибки раньше:

```php
<?php

function formatMoney(float $amount, string $currency = "руб."): string {
    return number_format($amount, 2, '.', '') . " " . $currency;
}

echo formatMoney(12345.6);        // 12345.60 руб.
echo formatMoney(99.99, "USD");   // 99.99 USD
```

Область видимости переменных — внутри функции нет доступа к внешним переменным (в отличие от JS). Нужна переменная снаружи — передай параметром:

```php
<?php

$tax = 0.2;

function addTax(float $price, float $taxRate): float {
    return $price * (1 + $taxRate);
}

echo addTax(1000, $tax); // 1200
```

---

## Практическое задание

Расширенный чек для нескольких товаров.

**Что нужно:**

Напиши функцию `calcTotal(float $price, int $qty, float $discount = 0.0): float`, которая принимает цену, количество и процент скидки, возвращает итоговую сумму. Напиши функцию `formatMoney(float $amount): string`, которая форматирует число в строку `"75000.00 руб."`.

Создай массив из трёх товаров (каждый товар — ассоциативный массив с ключами `name`, `price`, `qty`, `discount`). С помощью `foreach` пройдись по массиву, посчитай итог для каждого через `calcTotal` и выведи строку чека. В конце выведи общую сумму всех товаров. Используй `match` или `if`, чтобы добавить к итогу пометку: если сумма больше 100 000 — `"(Крупный заказ)"`, иначе — `"(Обычный заказ)"`.

Ожидаемый вывод (цифры будут твои):

```
=== Чек ===
Ноутбук       x2  — 135000.00 руб. (скидка 10%)
Мышь          x1  — 2500.00 руб.
Клавиатура    x3  — 10500.00 руб. (скидка 5%)
-----------
Итого: 148000.00 руб. (Крупный заказ)
===========
```

---

## Решение практического задания

```php
<?php

// 1. Функция расчета итоговой суммы одного товара с учетом скидки
function calcTotal(float $price, int $qty, float $discount = 0.0): float
{
    $subtotal = $price * $qty;
    $discountAmount = $subtotal * ($discount / 100);
    return $subtotal - $discountAmount;
}

// 2. Функция форматирования денежной суммы
function formatMoney(float $amount): string
{
    return number_format($amount, 2, '.', '') . " руб.";
}

// 3. Массив товаров (ассоциативные массивы)
$cart = [
    [
        'name' => 'Ноутбук',
        'price' => 75000.00,
        'qty' => 2,
        'discount' => 10.0,
    ],
    [
        'name' => 'Мышь',
        'price' => 2500.00,
        'qty' => 1,
        'discount' => 0.0,
    ],
    [
        'name' => 'Клавиатура',
        'price' => 3684.21, // или 3500.00 (подбираем так, чтобы со скидкой 5% было ровно 10500.00 руб.)
        'qty' => 3,
        'discount' => 5.0,
    ],
];

// 4. Формирование и вывод чека
$grandTotal = 0.0;

echo "=== Чек ===" . PHP_EOL;

foreach ($cart as $item) {
    // Считаем итоговую сумму по текущей позиции
    $itemTotal = calcTotal($item['price'], $item['qty'], $item['discount']);
    $grandTotal += $itemTotal;

    // Приведение к наименованию с выравниванием пробелами для аккуратного вывода
    $namePadded = str_pad($item['name'], 15, " ");
    
    // Формируем пометку о скидке, если она больше 0
    $discountInfo = $item['discount'] > 0 
        ? " (скидка " . (int)$item['discount'] . "%)" 
        : "";

    // Вывод строки чека
    echo "{$namePadded} x{$item['qty']} — " . formatMoney($itemTotal) . "{$discountInfo}" . PHP_EOL;
}

echo "-----------" . PHP_EOL;

// 5. Определение типа заказа с помощью match
$orderType = match (true) {
    $grandTotal > 100000 => "(Крупный заказ)",
    default => "(Обычный заказ)",
};

// 6. Вывод общих итогов
echo "Итого: " . formatMoney($grandTotal) . " {$orderType}" . PHP_EOL;
echo "===========" . PHP_EOL;
```
