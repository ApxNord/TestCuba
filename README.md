# Ответы на 1-ое тестовое задание
**1. В БД mysql  есть одна единственная таблица «abc» с полями: id (int), name (varchar), cnt (int). 
В таблице содержится порядка 10 млн записей. Что нужно сделать, что быстро работали следующие запросы (3 разных случая)? 
Все остальные факторы кроме скорости чтения не критичны.
SELECT * FROM abc WHERE name = 'xxx' AND cnt = yyy
SELECT * FROM abc WHERE cnt = xxx AND name LIKE 'yyy%'
SELECT * FROM abc ORDER BY cnt ASC**

Из рекомендаций, которые можно применить:
Стараться не использовать * в запросе, при указании конкретных столбцов запрос не будет тратить время на проверку и создание этих столбцов.
Разбить таблицу на отдельные разделы по какому-нибудь признаку, т.е. осуществить партицирование таблицы.
Так же можно создать составные индексы. Для первого запроса - создать составной индекс для полей name и cnt, для второго - cnt и name, для третьего - только cnt.
К примеру:
```sql CREATE INDEX index_name_cnt ON abc (name, cnt);
CREATE INDEX index_cnt_name ON abc (cnt, name);
CREATE INDEX index_cnt ON abc (cnt);
```
***
**2. У вас есть задача обработки большого объема данных в PHP (например, парсинг CSV-файлов объемом в несколько гигабайт). 
Как вы будете обрабатывать этот файл, чтобы избежать превышения лимита памяти, и какие функции PHP и подходы для этого будете использовать?**

Если это CSV-файл, можно воспользоваться функцией fgetcsv. Так будет постепенно считываться каждая строка файла, а не весь файл сразу.
Пример:
```php $handle = fopen("file.csv", "r");
if ($handle !== false) {
    while (($data = fgetcsv($handle)) !== false) {
        // Обработка данных
    }
    fclose($handle);
}
```
***
**3. Есть код и запрос к БД, чтобы вы в нем изменили? Почему?:
$DB->query("SELECT * FROM abc WHERE id=" . $_GET['id']);**

Здесь наблюдается проблема SQL-инъекции. Т.к. параметр $_GET['id'] ничем не экранирован и вставляется в запрос на прямую, то в этот параметр может прийти опасное значение.
Например '2; drop table ...'. Такие запросы лучше создавать через параметризированные запросы. Пример:
```php $query = $DB->prepare("SELECT * FROM abc WHERE id = :id");
$query->execute(['id' => $_GET['id']]);
$result = $query->fetchAll();
```
***
**4. В БД есть таблица заказов (orders) с полями:
date - дата оформления заказа
customer_name - имя клиента
order_price - сумма заказа

Напиши sql запросы для выборки:**
Запрос, который покажет сколько денег принес каждый отдельно взятый покупатель с группировкой по месяцам.
```sql
SELECT customer_name, MONTH(date) AS month, SUM(order_price) AS total_price
FROM orders
GROUP BY customer_name, YEAR(date), MONTH(date)
ORDER BY customer_name, YEAR(date), MONTH(date);
```
Запрос, который выведет  имена клиентов, у которых суммарные покупки за весь период превысили 10 тыс. руб. и одновременно никогда не было заказов менее 500 руб.
```sql
SELECT customer_name
FROM orders
GROUP BY customer_name
HAVING SUM(order_price) > 10000 AND MIN(order_price) >= 500;
```
***
**5. Есть две javascript-функции:
function f(a,b) { return a+b }
и
var f = function(a,b) { return a+b }
Есть ли между ними разница? Если есть то какая?**

Сами функции выполняют одинаковые действия. только в первом случае идет классическое объявление функции f. Ее можно вызвать в любой точке программы, даже до ее объявления.
Во втором случае объявлена анонимная функция и результат ее выполнения передана переменной f. Обращаться к этой переменной можно только после ее объявления.
***
**6. Чем принципиально отличаются между собой условия LEFT JOIN и INNER JOIN в sql? Какой вариант JOIN может дать потенциально больше результатов (строк) и почему?**

Оба условия осуществляют объединение двух табилц, сопоставляя строки для которых выполняется условие соединения. Только в случае с LEFT JOIN, таблица слева
(которая указана первая в запросе) вернет все строки, даже если нет соответствующего значения из правой таблицы (в результатирующую таблицу будет передано значение null в столбец).
Поэтому LEFT JOIN вернет больше строк.
***
**7. Вам нужно реализовать консольный php-скрипт на сервере под Unix, который бы выводил каждые 15 секунд фразу «Hello». После вывода «Hello» скрипт всегда завершает работу. 
Напишите этот скрипт и пошагово расскажите, что нужно сделать, чтобы выполнялись исходные условия.**

Создать скрипт, например hello.php, и разместить в него эти строки:
```php
<?
while(true) {
    echo "Hello\n";
    sleep(15);
    break;
}
```
Чтобы скрипт выполнялся каждые 15 секунд, можно воспользоваться cron-таблицей и в ней добавить строку выполнения:
``` * * * * * /путь_к_скрипту/hello.php ```
Дополнительно нужно еще убедиться, что у скипта есть права на выполнение. Для этого можно выполнить команду:
``` chmod +x hello.php ```
***
**8. Напишите на php функцию для распределения рублевой скидки по купону на все товары в корзине пропорционально стоимости товара. 
На входе в функцию передаются два параметра: размер скидки в рублях (!) и массив из цен товаров, на выходе тот же массив цен, 
но уже с учетом скидки: distribute_discount(int $discount, array $prices) → return array $prices;**
```php
<?
function distribute_discount(int $discount, array $prices) : array {
    // Получаю сумму всех цен
    $total_price = array_sum($prices);

    // Подсчет доли скидки
    $discount_per_price = $discount / $total_price;

    foreach ($prices as $key => $price) {
        $discount_amount = round($price * $discount_per_price, 2); // Лучше еще округленить до двух знаков после запятой
        $prices[$key] = $price - $discount_amount;
    }

    return $prices;
}
```
