# EggHeads

## 1)	Опишите, какие проблемы могут возникнуть при использовании данного кода

```php
$mysqli = new mysqli("localhost", "my_user", "my_password", "world");
// ПРОБЛЕМА: connection string в бизнес логике.
// ОПИСАНИЕ: Передавать строки подключения к БД следует в другом месте (на другом уровне, но точно не в файле где будут полученные результаты), в идеале использовать DI

$id = $_GET['id'];
// ПРОБЛЕМА: типизация и валидация
// ОПИСАНИЕ: ID получаем прямо из Query параметров, без типизации и валидации входящих данных. К тому же ID может не быть вовсе

$res = $mysqli->query('SELECT * FROM users WHERE u_id='. $id);
// ПРОБЛЕМА1: Prepared Statements
// ОПИСАНИЕ1: SQL injections
// ПРОБЛЕМА2: Не понятно что такое u_id
// ОПИСАНИЕ2: МБ в базе нет первичного ключа? А если это какой-то другой параметр, то название вообще не говорит о его назначении
// ПРОБЛЕМА3: Native SQL
// ОПИСАНИЕ3: Тут нет необходимости в обычном SQL запросе, так как это не сложный кастомный запрос, он только усложнит поддержку и рефакторинг
// ПРОБЛЕМА4: Result columns
// ОПИСАНИЕ4: Действительно ли нам необходимо получать все данные (*)?

$user = $res->fetch_assoc();
// Если u_id это не уникальный (первичный ключ), нужно использовать цикл для получения всех результатов
```

## 2)	Сделайте рефакторинг

```php
// Before:
$questionsQ = $mysqli->query('SELECT * FROM questions WHERE catalog_id='. $catId);
$result = array();
while ($question = $questionsQ->fetch_assoc()) {
    $userQ = $mysqli->query('SELECT name, gender FROM users WHERE id='. (int)$question[‘user_id’]);
    $user = $userQ->fetch_assoc();
    $result[] = array('question'=>$question, 'user'=>$user);
    $userQ->free();
}
$questionsQ->free();

// After:
$statement = $mysqli->prepare('SELECT questions.*, users.name, users.gender
    FROM questions
    LEFT JOIN users ON (questions.user_id = users.id)
    WHERE questions.catalog_id=?');
$statement->bind_param('i', $catId);
$statement->execute();
$sqlResult = $statement->get_result();
$result = array_map(static fn($row) => ['question' => $question, 'user' => $user], $sqlResult->fetch_assoc());
$sqlResult->free();
```

## 3)	Напиши SQL-запрос

```sql
SELECT
    users.name AS 'Counterparty name',
    users.phone AS 'Counterparty phone',
    SUM(orders.subtotal) AS 'Orders sum',
    AVG((orders.subtotal)) AS 'Average sum',
    MAX(orders.created) AS 'Last order date'
FROM users
LEFT JOIN orders ON (orders.user_id = users.id)
GROUP BY users.id
```
 
## 4) Напиши SQL-запросы

```sql
#1
SELECT DepartamentId, MAX(Salary)
FROM test_users2
GROUP BY test_users2.DepartamentId;

#2
SELECT name, lastName
FROM test_users2
WHERE DepartamentId = 3 AND Salary > 90000;

#3 Не совсем понятна суть вопроса, предположу, что для ускорения предыдущих двух запросов:
CREATE INDEX Departament3_Salary_index
    ON test_users2 (DepartamentId DESC, Salary DESC);

CREATE INDEX Departament_index
    ON test_users2 (DepartamentId);
```
 
## 5) Сделайте рефакторинг кода для работы с API чужого сервиса

```js
function printOrderTotal(responseString) {
    const responseJSON = JSON.parse(responseString);
    let orderSubtotal = 0;
        responseJSON.forEach(function(item){
        if (item.price !== undefined) {
            orderSubtotal += parseFloat(item.price);
        }
    });
        return orderSubtotal;
    }

    function getOutput(jsonString) {
        const total = printOrderTotal(jsonString);
        let outputSum = total === 0? 'Бесплатно': total + ' руб.';
        return 'Стоимость заказа: ' + outputSum;
    }

    console.log(getOutput('[{"price":0}]'));
    console.log(getOutput('[{"price":1}, {"price":7}]'));
    console.log(getOutput('[{"price":1.01}, {"price":101.448}]'));
```
