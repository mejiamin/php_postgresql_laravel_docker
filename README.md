# PHP, SQL & PostgreSQL, Laravel, Docker | 16 Уроков

## Уроки:

**Этап 1 - 4 урока: PHP**

1. Переменные и типы данных
2. Управление потоком и функции
3. **👉 ООП в PHP**
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

## Урок 3 — ООП в PHP

ООП — это способ организовать код вокруг объектов, которые сочетают данные и поведение. Вместо отдельных функций и массивов ты создаёшь классы, которые описывают сущности предметной области.

### 1. Класс и объект

Класс — это шаблон. Объект — экземпляр этого шаблона:

```php
<?php

class Product {
  // Свойства — данные объекта
  public string $name;
  public float $price;

  // Конструктор — вызывается при создании объекта
  public function __construct(string $name, float $price) {
    $this->name  = $name;
    $this->price = $price;
  }

  // Метод — поведение объекта
  public function getFormattedPrice(): string {
    return number_format($this->price, 2, '.', '') . " руб.";
  }
}

$laptop = new Product("Ноутбук", 75000.0);
echo $laptop->name;                  // Ноутбук
echo $laptop->getFormattedPrice();   // 75000.00 руб.
```

`$this` внутри класса — это ссылка на текущий объект.

---

### 2. Модификаторы доступа

`public` — доступно везде. `private` — только внутри класса. `protected` — внутри класса и его наследников. Правило хорошего тона: свойства делай `private`, доступ давай через методы:

```php
<?php

class Product {
  private string $name;
  private float  $price;
  private float  $discount; // в процентах

  public function __construct(
    string $name, 
    float $price, 
    float $discount = 0.0,
  ) {
    $this->name     = $name;
    $this->price    = $price;
    $this->discount = $discount;
  }

  // Геттеры — контролируемый доступ к данным
  public function getName(): string  { return $this->name; }
  public function getPrice(): float  { return $this->price; }

  // Бизнес-логика внутри класса
  public function getTotal(): float {
    return $this->price * (1 - $this->discount / 100);
  }

  public function hasDiscount(): bool {
    return $this->discount > 0.0;
  }

  public function getDiscountPercent(): float {
    return $this->discount;
  }
}

$mouse = new Product("Мышь", 2500.0);
$laptop = new Product("Ноутбук", 75000.0, 10.0);

echo $laptop->getTotal();    // 67500
echo $laptop->hasDiscount(); // 1 (true)
```

---

### 3. Магический метод __toString

Позволяет использовать объект как строку — в `echo` и конкатенации:

```php
<?php

class Product {
  // ... (свойства и конструктор как выше)

  public function __toString(): string {
    $total = number_format($this->getTotal(), 2, '.', '');
    $info  = $this->hasDiscount()
      ? " (скидка {$this->discount}%)"
      : "";
    return "{$this->name}: {$total} руб.{$info}";
  }
}

$laptop = new Product("Ноутбук", 75000.0, 10.0);
echo $laptop; // Ноутбук: 67500.00 руб. (скидка 10%)
```

---

### 4. Наследование

Дочерний класс получает все свойства и методы родителя и может расширять или переопределять их:

```php
<?php

class Product {
  public function __construct(
    protected string $name,
    protected float  $price
  ) {}

  public function getTotal(): float {
    return $this->price;
  }

  public function getName(): string { return $this->name; }
}

// Promoted properties в конструкторе (PHP 8) — сокращение выше
// protected string $name автоматически создаёт свойство и присваивает его

class DiscountedProduct extends Product {
  public function __construct(
    string $name,
    float  $price,
    private float $discount // добавляем новое свойство
  ) {
    parent::__construct($name, $price); // вызов конструктора родителя
  }

  // Переопределяем метод родителя
  public function getTotal(): float {
    return $this->price * (1 - $this->discount / 100);
  }

  public function getDiscount(): float { return $this->discount; }
}

$regular    = new Product("Мышь", 2500.0);
$discounted = new DiscountedProduct("Ноутбук", 75000.0, 10.0);

echo $regular->getTotal();    // 2500
echo $discounted->getTotal(); // 67500
```

---

### 5. Интерфейс

Интерфейс — это контракт: класс, реализующий интерфейс, обязан иметь все указанные методы. Это основа полиморфизма:

```php
<?php

interface Printable {
  public function toReceiptLine(): string;
}

class Product implements Printable {
  public function __construct(
    private string $name,
    private float  $price,
    private float  $discount = 0.0
  ) {}

  public function getTotal(): float {
    return $this->price * (1 - $this->discount / 100);
  }

  public function toReceiptLine(): string {
    $total = number_format($this->getTotal(), 2, '.', '');
    $info  = $this->discount > 0 ? " (скидка {$this->discount}%)" : "";
    return str_pad($this->name, 15) . "— {$total} руб.{$info}";
  }
}
```

---

## Практическое задание

Создай файл `lesson3.php`. Реализуй три класса.

**`Product`** — свойства `name`, `price`, `discount` (все `private`). Конструктор с тремя параметрами, `discount = 0.0` по умолчанию. Методы: `getName()`, `getTotal(): float` (цена со скидкой), `toReceiptLine(): string` (одна строка для чека).

**`Cart`** — хранит массив объектов `Product` в приватном свойстве `$items = []`. Методы: `addProduct(Product $product): void`, `getGrandTotal(): float` (сумма всех `getTotal()`), `printReceipt(): void` (выводит полный чек).

**`DiscountedCart`** — наследует `Cart`. Принимает в конструктор дополнительный `float $extraDiscount` — скидка на всю корзину целиком. Переопределяет `getGrandTotal()`: применяет `$extraDiscount` поверх суммы родителя. Переопределяет `printReceipt()`: добавляет строку с общей скидкой на корзину.

Ожидаемый вывод:

```
=== Обычная корзина ===
Ноутбук         — 67500.00 руб. (скидка 10%)
Мышь            — 2500.00 руб.
Клавиатура      — 10500.00 руб. (скидка 5%)
Итого: 80500.00 руб.

=== VIP корзина (доп. скидка 5%) ===
Ноутбук         — 67500.00 руб. (скидка 10%)
Мышь            — 2500.00 руб.
Итого без доп. скидки: 70000.00 руб.
Доп. скидка (5%): 3500.00 руб.
Итого: 66500.00 руб.
```

---

## Решение практического задания

```php
<?php

class Product
{
  // Constructor promotion (PHP 8) — свойства объявляются прямо в конструкторе
  public function __construct(
    private string $name,
    private float  $price,
    private float  $discount = 0.0
  ) {}

  public function getName(): string
  {
    return $this->name;
  }

  public function getTotal(): float
  {
    return $this->price * (1 - $this->discount / 100);
  }

  public function toReceiptLine(): string
  {
    $total        = number_format($this->getTotal(), 2, '.', '');
    $discountInfo = $this->discount > 0
      ? " (скидка {$this->discount}%)"
      : "";

    return str_pad($this->name, 15) . "— {$total} руб.{$discountInfo}";
  }
}

class Cart
{
  // protected — чтобы DiscountedCart мог обратиться к items
  // private здесь не подойдёт: дочерний класс его не увидит
  protected array $items = [];

  public function addProduct(Product $product): void
  {
    $this->items[] = $product;
  }

  public function getGrandTotal(): float
  {
    $total = 0.0;
    foreach ($this->items as $item) {
      $total += $item->getTotal();
    }
    return $total;
  }

  // protected — общий вспомогательный метод для дочерних классов
  // Это паттерн "Template Method": родитель предоставляет строительный блок
  protected function printItems(): void
  {
    foreach ($this->items as $item) {
      echo $item->toReceiptLine() . PHP_EOL;
    }
  }

  public function printReceipt(): void
  {
    echo "=== Обычная корзина ===" . PHP_EOL;
    $this->printItems();
    echo "Итого: " . number_format($this->getGrandTotal(), 2, '.', '')
      . " руб." . PHP_EOL;
  }
}

class DiscountedCart extends Cart
{
  public function __construct(private float $extraDiscount) {}

  public function getGrandTotal(): float
  {
    // parent:: — вызов метода родительского класса
    // Берём сумму до доп. скидки и применяем свою
    return parent::getGrandTotal() * (1 - $this->extraDiscount / 100);
  }

  public function printReceipt(): void
  {
    $subtotal       = parent::getGrandTotal();  // сумма БЕЗ доп. скидки
    $discountAmount = $subtotal * ($this->extraDiscount / 100);
    $grandTotal     = $this->getGrandTotal();   // сумма СО скидкой

    echo "=== VIP корзина (доп. скидка {$this->extraDiscount}%) ===" . PHP_EOL;
    $this->printItems();  // унаследованный метод Cart
    echo "Итого без доп. скидки: "
      . number_format($subtotal, 2, '.', '') . " руб." . PHP_EOL;
    echo "Доп. скидка ({$this->extraDiscount}%): "
      . number_format($discountAmount, 2, '.', '') . " руб." . PHP_EOL;
    echo "Итого: "                                
      . number_format($grandTotal, 2, '.', '')      . " руб." . PHP_EOL;
  }
}

// --- Использование ---

$cart = new Cart();
$cart->addProduct(new Product("Ноутбук",    75000.00, 10.0));
$cart->addProduct(new Product("Мышь",        2500.00));
$cart->addProduct(new Product("Клавиатура", 11052.63,  5.0));
$cart->printReceipt();

echo PHP_EOL;

$vipCart = new DiscountedCart(5.0);
$vipCart->addProduct(new Product("Ноутбук", 75000.00, 10.0));
$vipCart->addProduct(new Product("Мышь",     2500.00));
$vipCart->printReceipt();
```

**Три ключевые вещи, которые стоит понять прежде чем копировать:**

`protected` на `$items` — это намеренное решение. Если оставить `private`, `DiscountedCart` не увидит массив товаров и `printItems()` сломается. `protected` открывает доступ только для наследников, не для всего остального кода.

`parent::getGrandTotal()` в двух местах `DiscountedCart` — это не баг, это осознанно. Для `$subtotal` нужна сумма без доп. скидки (т.е. результат родительского метода), для `$grandTotal` — уже с ней. Если вызвать `$this->getGrandTotal()` оба раза, скидка применится дважды.

Constructor promotion — это синтаксический сахар PHP 8. `private float $extraDiscount` в конструкторе одновременно объявляет свойство, задаёт его тип и присваивает значение. Старый способ — три отдельных строки.
