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

# Индексы в БД

![[Pasted image 20251015223220.png]]

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






















### Zero-Links
- [[00 SQL]]
- [[00 Обучение]]
- [[00 Data_Base]]


### Links
- https://www.youtube.com/watch?v=uGKIXTUjZbc&list=PLtPJ9lKvJ4oh5SdmGVusIVDPcELrJ2bsT
- 

