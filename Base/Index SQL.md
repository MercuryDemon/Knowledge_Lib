24-02-2026 20:52
Tags: #z #lesson #SQL 

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

# Типы индексов в PostgreSQL:

## hashindex

- Hash-индекс - Эффективен только для операций равенства =. Работает быстрее чем b-tree при простых (=).
```SQL
CREATE INDEX idx_employees_name_hash
ON employees USING hash(full_name);
```


## B-tree 

- используется по умолчанию, если не выбран другой. Хорош для сравнений <, > и т.д. Подходит для уникальных ограничений(UNIQUE, PRIMARY KEY).
```SQL
CREATE INDEX idx_employees_salary
ON employees(salary);
```

## GiST 
(Generalized Search Tree) - Универсальный тип индекса. Подходит для полнотекстового поиска, геоданных, диапазонов. Используется с типами tsvector, range, geometry.
```SQL
CREATE INDEX idx_articles_content
ON articles USING gist(to_tsvector('russian', content));
```

## GIN 
(Generalized Inverted Index) - Идеален для поиска по множествам значений. Используется для JSONB, массивов и полнотекстового поиска.Позволяет искать вхождения внутри коллекций.
```SQL
CREATE INDEX idx_data_json
ON documents USING gin(data jsonb_path_ops);
```

## BRIN 
(Block Range Index) - Хранит данные не по каждой строке, а по блокам. Очень лёгкий и экономный по памяти. Подходит для больших таблиц с монотонными значениями (например, дата создания).
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


### Частичный индекс - Индекс для части строк по условию.
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

### Золотое правило
Индексы нужны для полей, которые часто используются в WHERE, JOIN и ORDER BY. Не ставьте индекс на каждое поле "на всякий случай" — это вредит производительности.

### Zero-Links
- [[00 SQL]]
- [[00 Обучение]]


### Links
- https://habr.com/en/companies/postgrespro/articles/328280/
- https://habr.com/en/companies/postgrespro/articles/330544/
- https://eax.me/postgresql-full-text-search/
- https://eax.me/pg-trgm/

