# PHP, SQL & PostgreSQL, Laravel, Docker | 16 Уроков

## Уроки:

**Этап 1 - 4 урока: PHP**

1. Переменные и типы данных
2. Управление потоком и функции
3. ООП в PHP
4. **👉 Файлы и обработка ошибок**

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

## Урок 4 — Файлы и обработка ошибок

Это последний урок по чистому PHP. После него переходим к SQL и PostgreSQL.

### 1. Простые операции с файлами

Для большинства задач хватает двух функций:

```php
<?php

// Запись — создаёт файл если нет, перезаписывает если есть
file_put_contents('data.txt', "Hello, World!\n");

// Дозапись в конец файла
file_put_contents('data.txt', "Вторая строка\n", FILE_APPEND);

// Чтение — возвращает всё содержимое строкой
$content = file_get_contents('data.txt');
echo $content;

// Проверки перед работой
if (file_exists('data.txt')) {
  echo "Файл существует\n";
}

// Удалить файл
unlink('data.txt');
```

---

### 2. Построчное чтение (для больших файлов)

`file_get_contents` загружает файл целиком в память. Для больших файлов (логи, CSV на гигабайты) используй потоки:

```php
<?php

$handle = fopen('data.csv', 'r'); // r = read, w = write, a = append

if ($handle === false) {
  // fopen возвращает false если файл не открылся
  die("Не удалось открыть файл\n");
}

while (($line = fgets($handle)) !== false) {
  echo trim($line) . "\n"; // trim убирает \n в конце строки
}

fclose($handle); // всегда закрывай дескриптор
```

Запись через потоки:

```php
<?php

$handle = fopen('log.txt', 'a'); // a = append
fwrite($handle, date('Y-m-d H:i:s') . " — событие\n");
fclose($handle);
```

---

### 3. Работа с JSON

JSON — стандарт обмена данными между сервисами. В PHP это `json_encode` / `json_decode`:

```php
<?php

$data = [
  ['name' => 'Ноутбук', 'price' => 75000.0],
  ['name' => 'Мышь',    'price' =>  2500.0],
];

// Массив → JSON строка
$json = json_encode($data, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE);
file_put_contents('products.json', $json);

// JSON строка → массив (второй аргумент true = ассоциативный массив, не объект)
$loaded = json_decode(file_get_contents('products.json'), true);
var_dump($loaded[0]['name']); // string(7) "Ноутбук"

// Проверка ошибок декодирования
if (json_last_error() !== JSON_ERROR_NONE) {
  echo "Ошибка JSON: " . json_last_error_msg();
}
```

---

### 4. Исключения

Исключения — механизм обработки ошибок. Вместо `die()` или возврата `false` бросай исключение, а в вызывающем коде лови его:

```php
<?php

function divide(int $a, int $b): float
{
  if ($b === 0) {
    throw new \InvalidArgumentException("Делитель не может быть нулём");
  }
  return $a / $b;
}

try {
  echo divide(10, 2) . "\n";  // 5
  echo divide(10, 0) . "\n";  // бросает исключение
  echo "Эта строка не выполнится\n";
} catch (\InvalidArgumentException $e) {
  echo "Ошибка: " . $e->getMessage() . "\n";
} finally {
  // finally выполняется ВСЕГДА — и при ошибке, и без неё
  echo "Блок finally\n";
}
```

Несколько `catch` — ловят разные типы исключений:

```php
<?php

try {
  // ... какой-то код
} catch (\InvalidArgumentException $e) {
  echo "Неверный аргумент: " . $e->getMessage();
} catch (\RuntimeException $e) {
  echo "Ошибка выполнения: " . $e->getMessage();
} catch (\Throwable $e) {
  // Ловит всё остальное, включая Error (не только Exception)
  echo "Неизвестная ошибка: " . $e->getMessage();
}
```

---

### 5. Пользовательские исключения

Наследуй от стандартных классов, чтобы создавать смысловые исключения для своего кода:

```php
<?php

// Базовый класс для всех ошибок хранилища
class StorageException extends \RuntimeException {}

// Конкретные типы — можно ловить по отдельности
class FileNotFoundException extends StorageException {}
class FileCorruptedException extends StorageException {}

function readJson(string $path): array
{
  if (!file_exists($path)) {
    throw new FileNotFoundException("Файл не найден: $path");
  }

  $content = file_get_contents($path);
  $data    = json_decode($content, true);

  if (json_last_error() !== JSON_ERROR_NONE) {
    throw new FileCorruptedException("JSON повреждён: $path");
  }

  return $data;
}

try {
  $data = readJson('missing.json');
} catch (FileNotFoundException $e) {
  echo "Файл отсутствует: " . $e->getMessage();
} catch (FileCorruptedException $e) {
  echo "Данные битые: " . $e->getMessage();
}
```

---

### 6. strict_types

Добавляй это в начало каждого файла — включает строгую проверку типов. Без него PHP молча приведёт `"42"` к `int`, с ним — бросит `TypeError`:

```php
<?php
declare(strict_types=1); // всегда первой строкой после <?php

function add(int $a, int $b): int
{
  return $a + $b;
}

add(1, 2);     // OK
add(1, "2");   // TypeError: Argument #2 must be of type int, string given
add(1, 2.9);   // TypeError: Argument #2 must be of type int, float given
```

В Laravel `strict_types=1` — стандарт. Привыкай сейчас.

---

## Практическое задание

Создай файл `lesson4.php` — система хранения корзины в JSON.

**Что реализовать:**

Два кастомных исключения: `FileNotFoundException` и `FileCorruptedException` — оба наследуют `\RuntimeException`.

Класс `JsonStorage` с тремя методами: `write(string $path, array $data): void` — кодирует массив в JSON и пишет в файл, бросает `\RuntimeException` если запись не удалась. `read(string $path): array` — читает файл и декодирует JSON, бросает `FileNotFoundException` или `FileCorruptedException` в соответствующих ситуациях. `delete(string $path): void` — удаляет файл, бросает `FileNotFoundException` если его нет.

В основном коде: создай массив из трёх товаров (те же что в уроке 3 или новые), сохрани в `cart.json`, прочитай обратно и выведи список. Затем попробуй прочитать `ghost.json` (несуществующий) и поймай исключение. Затем удали `cart.json`.

Ожидаемый вывод:

```
Сохранено 3 товара в cart.json

Загружено из файла:
 - Ноутбук: 67500.00 руб.
 - Мышь: 2500.00 руб.
 - Клавиатура: 10500.00 руб.

[Ошибка] Файл не найден: ghost.json
cart.json удалён
```

---

## Решение практического задания

Покажу — но советую хотя бы пробежаться по коду и понять каждую строку, прежде чем двигаться дальше. SQL без понимания исключений будет труднее.

```php
<?php
declare(strict_types=1);

// --- Кастомные исключения ---

class FileNotFoundException extends \RuntimeException {}
class FileCorruptedException extends \RuntimeException {}

// --- Класс хранилища ---

class JsonStorage
{
  public function write(string $path, array $data): void
  {
    $json   = json_encode($data, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE);
    $result = file_put_contents($path, $json);

    // file_put_contents возвращает false при ошибке записи
    if ($result === false) {
      throw new \RuntimeException("Не удалось записать файл: $path");
    }
  }

  public function read(string $path): array
  {
    if (!file_exists($path)) {
      throw new FileNotFoundException("Файл не найден: $path");
    }

    $content = file_get_contents($path);
    $data    = json_decode($content, true);

    // json_decode возвращает null при невалидном JSON
    if (json_last_error() !== JSON_ERROR_NONE) {
      throw new FileCorruptedException("JSON повреждён: $path");
    }

    return $data;
  }

  public function delete(string $path): void
  {
    if (!file_exists($path)) {
      throw new FileNotFoundException("Файл не найден: $path");
    }

    unlink($path);
  }
}

// --- Основной код ---

$storage = new JsonStorage();

$cart = [
  ['name' => 'Ноутбук',    'price' => 75000.00, 'discount' => 10.0],
  ['name' => 'Мышь',       'price' =>  2500.00, 'discount' =>  0.0],
  ['name' => 'Клавиатура', 'price' => 11052.63, 'discount' =>  5.0],
];

// Сохранение
$storage->write('cart.json', $cart);
echo "Сохранено " . count($cart) . " товара в cart.json" . PHP_EOL;

// Чтение и вывод
echo PHP_EOL . "Загружено из файла:" . PHP_EOL;

$loaded = $storage->read('cart.json');

foreach ($loaded as $item) {
  $total = $item['price'] * (1 - $item['discount'] / 100);
  echo " - {$item['name']}: " . number_format($total, 2, '.', '') . " руб." . PHP_EOL;
}

// Попытка прочитать несуществующий файл
echo PHP_EOL;
try {
  $storage->read('ghost.json');
} catch (FileNotFoundException $e) {
  echo "[Ошибка] " . $e->getMessage() . PHP_EOL;
}

// Удаление
$storage->delete('cart.json');
echo "cart.json удалён" . PHP_EOL;
```

**Три вещи, которые важно понять:**

`file_put_contents` возвращает `false` при ошибке — не бросает исключение само по себе. Поэтому нужна явная проверка `=== false`. Строгое сравнение важно: `0` (записано 0 байт) тоже вернул бы `false` при нестрогом `==`.

`json_decode` возвращает `null` при невалидном JSON — но `null` это ещё не ошибка сама по себе, ведь можно было сохранить `null` намеренно. Поэтому проверяем через `json_last_error()`, который хранит код последней ошибки декодирования.

Оба кастомных исключения — пустые классы. Тело не нужно, всё нужное уже есть в `\RuntimeException`. Смысл их существования — в имени: код `catch (FileNotFoundException $e)` читается как документация.

Запусти, убедись что вывод совпадает — и переходим к Уроку 5: SQL и PostgreSQL.
