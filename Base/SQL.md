08-10-2025 19:09
Tags: #SQL #lesson #DB
# Что это такое

SQL - язык для работы с [[Базы данных (Data base)]] 

SQL - работает с реляционными базами данных

## Особенность реляционной модели

- В каждом поле хранится отдельные значения
- каждая сущность таблицы должны иметь значение в каждом поле(если значения нет - null)


## Почему SQL популярен

- Строгая математическая основа
- Стандартизация ANSI SQL
- Много людей знают SQL


# SELECT

синтаксис:
```PostgreSQL
SELECT * 
FROM niggers // *-извлекаем все данные, FROM-откуда и название таблицы данных
```
-- вместо * можно указать названия столбцов через запятую например `id, Name`

-- можно использовать оператор `AS`
`name AS item_name` тогда название столбца выведется `item_name`

-- если хотим получить только уникальные значения из столбца таблицы используем оператор `DISTINCT`
`SELECT DISTINCT(color) FROM niggers` =)

-- если хотим ограничить вывод нескольких первых результатов используем `LIMIT 10`
`SELECT DISTINCT(color) FROM niggers LIMIT 1` =)


# WHERE

оператор фильтрации данных (ГДЕ такоей-то столбец имеет значение сякое-то)
```PostgreSQL
SELECT * 
FROM niggers 
WHERE gender = 'mechanic'
```
операторы сравнения для фильтров:
![[Pasted image 20251013213440.png]]
логические операции:
![[Pasted image 20251013214041.png]]

или = OR
OR gender = 'male'
OR gender = 'mechanic'

# ORDER BY

сортировка сущностей по  столбцу
`SELECT * FROM niggers ORDER BY age
`
-- по дефолту сортируется по возрастанию ASC

-- если хотим по убыванию пишем DESC
`SELECT * FROM niggers ORDER BY age DESC`

# Создание таблиц
```PostgreSQL
CREATE TABLE niggers(
	id SERIAL PRIMARY KEY, //Создаем первичный ключ(в постгресе SERIAL автоматически инкрементится)
	name VARCHAR(100),
	gender VARCHAR(30),
	age INT,
	breed VARCHAR(30) // у последнего столбца не нужно ставить запятую
)
```
-- создали таблицу негров

основные(их дохуя) типы данных в SQL:
![[Pasted image 20251013215805.png]]


# Удаление таблицы

```PostgreSQL
DROP TABLE niggers
```

# Изменение таблицы

Чтобы чтото поменять в таблице - оператор `ALTER TABLE`
```PostgreSQL
ALTER TABLE niggers ADD COLUMN alive BOOLEAN;

ALTER TABLE niggers DROP COLUMN age;

ALTER TABLE niggers RENAME COLUMN name TO slave_name;

ALTER TABLE niggers RENAME TO slaves;
```

# Вставка и изменение данных в SQL

вставка данных:
```PostgreSQL
INSERT INTO niggers(name, gender, age) VALUES ('Osas','mechanic','31')
```

изменение данных:
```PostgreSQL
UPDATE niggers
SET name = 'Bomboclat',
	age = '99'
where id = 1
```

удаление данных:
```PostgreSQL
DELETE FROM niggers 
WHERE id = 2
```

удаление всех данных
```PostgreSQL
DELETE FROM niggers
```



# Группировка данных

чтобы сгрупировать по каким бибо значениям одного столбца используем GROUP BY
```PostgreSQL
SELECT gender, COUNT(*) FROM niggers
GROUP BY gender
```

многоуровневая группировка:

```PostgreSQL
SELECT gender, hair, COUNT(*) FROM niggers
GROUP BY gender, hair
```


# Агрегатные функции
![[Pasted image 20251015202356.png]]
## COUNT
счетчик
```PostgreSQL
COUNT(*) - считать * - все, или можно указать столбец по которому считать
```

## AVG
стреднее значение
```PostgreSQL
AVG(age)
```

пример:
```PostgreSQL
SELECT gender, AVG(age), SUM(age)/COUNT(*) AS averge // средний возраст нигеров
FROM niggers
GROUP BY gender
```
## MAX/MIN
максимальное значение/мнимальное значение
```PostgreSQL
MIN(age), MAX(age) - минимальный возраст
```

пример:
```PostgreSQL
SELECT gender, MIN(age), MAX(age) // мин/макс возраст нигеров
FROM niggers
GROUP BY gender
```

## SUM
сума значений
```PostgreSQL
SUM(age) - сумма всех значений из столбца - age
```

пример:
```PostgreSQL
SELECT gender, COUNT(*), SUM(age) // суммарные года прожитые всеми нигерами 
FROM niggers
GROUP BY gender
```


# HAVING
ключевое слово позволяющее включать условия для сортировки сгруппированных результатов.

пример:
```PostgreSQL
SELECT hair, COUNT(*) FROM niggers
WHERE genger='mechanic'
GROUP BY hair
HAVING COUNT(*) > 10
```
пример выведет группы с такими цветами волос, где негров больше 10

# Декомпазиция данных в базе

Мы можем создать столбец в таблице содержащий ссылки на данные из столбца в другой таблице. Такой столбей называется - внешний ключ/foreign key

- foreign key - может содержать тольок тот тип данных который реализует столбец связанной таблицы

Имеет смысл хранить в разных таблицах отдельные сущности
![[Pasted image 20251015213218.png]]

# Связывание таблиц

REFERANCES

пример:
```PostgreSQL
CREATE TABLE products(
	id PRIMARY KEY,
	name VARCHAR,
	type_id INT REFERANCES product_types(id) //связываем поля таблиц
	price int
	CONSTRAINT positive_price CHECK (price >= 0)
)
```
# Запрос данных из нескольких таблиц

Для обьединения выдачи данных из нескольких таблиц используется оператор  JOIN

пример
```PostgreSQL
SELECT produts.name, product_type.type_name
FROM products JOIN product_types
ON products.type_id = product_types.id // указываем условие обьединения
```
т.к. несколько таблиц мы пишем не только название столбца, а названаие_таблицы.название столбца

# Типы соединений в SQL

Внутреннее объединение - входят те строки для которых нашлись соответствующие строки из другой таблицы

Внешнее объединение - входят те строки для которых не нашлось соответсвующих значений, но они подходят под параметры вывода запроса

- левое (LEFT) внешнее обьединение распространяется на таблицу слева от ключевого слова JOIN
- правое (RIGHT) - справа от JOIN
- полное (FULL) распространяется на строки всех таблиц 
- перекрёстное (CROSS) каждое значение строеки справа для каждого значения строки слева ((условие не указывается))

# Схема базы данных

пример:
![[Pasted image 20251015215844.png]]

типы связи:
- один ко многим
- многие к одному
- многие ко многим

в SQL тип связи многие ко многим реализуется двумя столбцами в одной таблице, каждый из которых ведет к своей таблице

прмер запроса:
![[Pasted image 20251015220451.png]]


# Подзапросы

![[Pasted image 20251015220825.png]]
Запрос внутри запроса.

# Транзакции

транзакция - выполнение нескольких команд изменяющих базу, где необходимо чтобы все программы выполнились корректно, или не выполнение их вообще.


Причины не выполнения команд в БД
- Отказ СУБД (аппаратная проблема с сервером, программная ошибка в СУБД или ОС, нет места для записи данных)
- Отказ приложения пользователя(аппаратная проблема на клиенте, программная ошибкав приложении или ОС, пользователь прервал работу приложения)
- Потеря сетевого соединения клиент-сервер

Для начала отслеживания транзакции используем
```PostgreSQL
START TRANSACTION;

... // какието команды

COMMIT; / ROLLBACK;
```

для прекращения - 2 варианта
- COMMIT; - фиксация транзакции, изменения которые были выполнены командами записываются в БД
- ROLLBACK; - откат транзакции, если что-то где-то наебнулось, откатываем всё как было

в PostgreSQL - фиксация по умолчанию

# Ограничения в БД

ограничение - constraint - ограничивают некоторые изменения в базе для того, чтобы она оставалась в согласованном состояние

## Primary key

ограничение первичного ключа - столбец являющийся первичным ключем должен содержать не нулевое- уникальное значение. - ID

первичный ключ может содержать несколько полей(названий столбцов) 
записывается:
```PostgreSQL
PRIMARY KEY(product_id, order_id)
```

## NOT NULL

не пустые значения, содержать столбец должен

## UNIQUE

стобец должен содержать уникальные значение

## CHECK

Проверка значений с условием
пример 
```PostgreSQL
CREATE TABLE products(
	id PRIMARY KEY,
	name VARCHAR(100),
	type_id INT,
	price INT CHECK (price >= 0)
)
```
у ограничения можно указать имя
```PostgreSQL
CREATE TABLE products(
	id PRIMARY KEY,
	name VARCHAR(100),
	type_id INT,
	price INT 
	CONSTRAINT positive_price CHECK (price >= 0)
)
```


# Представления таблиц SQL

Представления - псевдонимы для запросов.

- представления не содержат данные, обращаются напрямую к исходным таблицам
- можно вместо ввода запроса вводить представление и запрос будет выполнен
- однако если структура БД поменяется придется переписывать все представления

как создать:
```PostgreSQL
CREATE VIEW customers_v id, name
	AS SELECT id, name FROM customers
```

Запрос к представлениям:
```PostgreSQL
SELECT * FROM customers_v //выведет все сущности таблицы предсавления
```

Зачем использовать?:
- ограничение доступа к данным
- псевдонимы для сложных запросов
- сокрытие реализации

## MATERIALIZED VIEW

- создается новая таблица из запроса в представлении, она хранит копии данных из исходных таблиц
- при обращении к представлению получаем данные из созданной таблицы представления
- потдерживают не все БД, PostgreSQL, Oracle - да
- данные из исходных таблиц не обновляются, чтобы обновлять матер. представ. запускаем:
```PostgreSQL
REFRESH MATERIALIZED VIEW products_v
```
 чтобы снести представление:
 ```PostgreSQL
DROP VIEW products_v

DROP MATERIALIZED VIEW products_v
 ```

## Индексы

Индекс — это специальная структура данных, которая ускоряет поиск строк в таблице по определённым столбцам.

- В PostgreSQL индексы создаются на уровне таблиц и автоматически поддерживаются при изменении данных (INSERT, UPDATE, DELETE).

- Но у него есть и минус - индексы занимают место и немного замедляют операции записи, но значительно ускоряют чтение.

пример создания/удаления индекса/создание уникального индекса

```SQL
CREATE INDEX idx_name
ON table_name

DROP INDEX idx_name

CREATE UNIQUE INDEX idx_unique_name
ON table_name
```

### Типы индексов в PostgreSQL:
- B-tree - используется по умолчанию, если не ывбрать другой. Хорош для сравнений <, > и т.д. Подходит для уникальных ограничений(UNIQUE, PRIMARY KEY).
```SQL
CREATE INDEX idx_employees_salary
ON employees(salary);
```

- Hash-индекс - Эввективен только для операций равенства =. Работает быстрее чем b-tree при простых =.
```SQL
CREATE INDEX idx_employees_name_hash
ON employees USING hash(full_name);
```

- GiST (Generalized Search Tree) - Универсальный тип индекса. Подходит для полнотекстового поиска, геоданных, диапазонов. Используется с типами tsvector, range, geometry.
```SQL
CREATE INDEX idx_articles_content
ON articles USING gist(to_tsvector('russian', content));
```

- GIN (Generalized Inverted Index) - Идеален для поиска по множествам значений. Используется для JSONB, массивов и полнотекстового поиска.Позволяет искать вхождения внутри коллекций.
```SQL
CREATE INDEX idx_data_json
ON documents USING gin(data jsonb_path_ops);
```

- BRIN (Block Range Index) - Хранит данные не по каждой строке, а по блокам. Очень лёгкий и экономный по памяти. Подходит для больших таблиц с монотонными значениями (например, дата создания).
```SQL
CREATE INDEX idx_logs_created
ON logs USING brin(created_at);
```


### Составной индекс - можно индексировать несколько колонок
```SQL
CREATE INDEX idx_orders_customer_date
ON orders(customer_id, order_date);
```
- Такой индекс ускоряет запросы, где фильтруются customer_id и order_date, но бесполезен для поиска только по order_date (если customer_id не участвует).


### Частичный индекс - Индекс для чсти строк по условию.
```SQL
CREATE INDEX idx_active_users
ON users(email)
WHERE is_active = TRUE;
```
- Полезно для таблиц, где большинство записей не нужны для поиска.


### Уникальный индекс - гарантирует уникальность значений в колонке или наборе колонок.
```SQL
CREATE UNIQUE INDEX idx_users_email
ON users(email);
```

-Фактически PRIMARY KEY и UNIQUE создают уникальные B-Tree индексы.


## Тразакции

Транзакция — это логическая единица работы с базой данных, которая объединяет один или несколько SQL-запросов в одно целое

Все операции внутри транзакции:

- выполняются целиком, либо

- откатываются полностью, если что-то пошло не так

пример:
```SQL

BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
ROLLBACK;
```
Если всё прошло успешно — COMMIT зафиксирует изменения.
Если одна из операций не удалась — можно сделать ROLLBACK и откатить всё.

Команды транзакций:
- BEGIN/START TRANSACTION - начало транзакции;
- COMMIT - зафиксировать изменения;
- ROLLBACK - отменить изменения;
- SAVEPOINT - создать промежуточную точку отката;
- ROLLBACK TO SAVEPOINT - откатиться к чекпоинту;
- RELEASE SAVEPOINT - удалить точку отката.

Транзакции подчиняются четырём ключевым принципам — ACID:

A-Atomicity - Атомарность. Все операции выполняются полностью, или не выполняются вовсе.
C-Consistency - Согласованность. После тарнзакции база остаётся в согласованном состоянии.
I-Isolation - Изоляция. Одноввременные изоляции не мешают друг другу.
D-Durability - Надежность. После Commit данные сохраняются даже при сбое. 

## Блокировки(mutex)
 
Когда несколько пользователей или процессов одновременно читают и изменяют данные, возникает риск конфликтов — например, два запроса пытаются обновить одну и ту же строку.
Чтобы избежать повреждения данных, PostgreSQL использует систему блокировок (locks).

Блокировка — это механизм, который ограничивает доступ других транзакций к объектам, пока текущая транзакция с ними работает.

### Виды блокировок:

- Row-level - блокируется одна строка (при UPDATE,DELETE SELECT... FOR UPDATE)
- Table-level - вся табица (при ALTER TABLE, DROP TABLE, CREATE INDEX)
- Advisory locks - пользовательские ручные блокировки)







### Zero-Links
- [[00 SQL]]
- [[00 Обучение]]
- [[00 Data_Base]]


### Links
- https://www.youtube.com/watch?v=uGKIXTUjZbc&list=PLtPJ9lKvJ4oh5SdmGVusIVDPcELrJ2bsT
- 

