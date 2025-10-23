
> Краткое описание: этот гайд содержит основные теоретические понятия и примеры, которые понадобятся для выполнения заданий из файла `УБД 3 Бурукин.docx` (выборки, сортировки, агрегирования, оконные функции и т.д.).

---

## 1. Общая классификация команд SQL

- **DDL (Data Definition Language)** — команды определения структуры данных: `CREATE`, `ALTER`, `DROP`, `TRUNCATE` и т.п. Используются для создания/изменения таблиц, схем, индексов.
    
- **DML (Data Manipulation Language)** — команды для работы с данными: `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `MERGE`.
    
- **DCL (Data Control Language)** — команды управления доступом: `GRANT`, `REVOKE`.
    
- **TCL (Transaction Control Language)** — управление транзакциями: `BEGIN`, `COMMIT`, `ROLLBACK` (в разных СУБД используются свои ключевые слова).
    

> В заданиях используются в основном DML (`SELECT`).

---

## 2. Базовый синтаксис `SELECT`

```sql
SELECT [выражения | столбцы]
FROM <таблица> -- или JOIN нескольких таблиц
WHERE <условие>
GROUP BY <столбцы>
HAVING <условие для групп>
ORDER BY <столбцы> [ASC | DESC];
```

Порядок логической обработки (важно для понимания):

1. `FROM` (включая `JOIN`)
    
2. `WHERE` (фильтрация строк)
    
3. `GROUP BY` (формирование групп)
    
4. `HAVING` (фильтрация групп)
    
5. `SELECT` (вычисление выражений и агрегатов)
    
6. `ORDER BY` (сортировка результата)
    
7. `LIMIT` / `OFFSET` (если есть — ограничение числа строк)
    

---

## 3. `ORDER BY` — сортировка

- Формат: `ORDER BY col1 ASC, col2 DESC, col3`.
    
- По умолчанию `ASC` (возрастание). `DESC` — по убыванию.
    
- Можно сортировать одновременно по нескольким колонкам — это используется в задании (например, `ORDER BY StateProvinceID ASC, TaxRate ASC, Name ASC`).
    

Пример:

```sql
SELECT * FROM sales_tax_rate
ORDER BY StateProvinceID ASC, TaxRate ASC, Name ASC;
```

---

## 4. `WHERE` — фильтрация строк

- Логические операторы: `=`, `<>` / `!=`, `>`, `<`, `>=`, `<=`.
    
- Комбинация: `AND`, `OR`, `NOT`.
    
- `IN` — перечисление значений: `col IN (v1, v2, ...)`.
    
- `BETWEEN` — диапазон (включительно): `col BETWEEN low AND high`.
    
- `IS NULL` / `IS NOT NULL` — проверка NULL.
    
- `LIKE` — шаблон: `%` — любое количество символов, `_` — один символ.
    

Примеры (аналогично заданиям):

```sql
-- NULL или не в списке:
SELECT FirstName, LastName
FROM Person.Person
WHERE Title IS NULL OR Title NOT IN ('Mr.', 'Ms.')
ORDER BY LastName ASC;

-- диапазон по числовому полю:
SELECT ProductID, OrderQty, UnitPrice
FROM purchase_order_detail
WHERE LineTotal BETWEEN 10000 AND 20000
ORDER BY ProductID ASC;

-- поиск подстроки в имени:
SELECT Color, SUM(ListPrice)
FROM product
WHERE Color IS NOT NULL AND Name LIKE '%tain%'
GROUP BY Color
ORDER BY Color;
```

**Важное про NULL:** сравнение `NULL = NULL` возвращает `UNKNOWN`. Для проверки NULL используйте `IS NULL`.

---

## 5. Агрегатные функции и `GROUP BY`

- Агрегатные функции: `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`.
    
- Если в `SELECT` есть агрегатные функции и одновременно указаны поля без агрегатов, поля без агрегатов должны быть в `GROUP BY`.
    
- `HAVING` применяется для фильтрации уже агрегированных групп.
    

Пример группировки и агрегирования:

```sql
SELECT ModifiedDate,
       AVG(Bonus) AS AverageBonusByDate,
       SUM(SalesYTD) AS TotalSales
FROM sales_person
GROUP BY ModifiedDate
ORDER BY ModifiedDate;
```

Для подсчёта общего количества строк:

```sql
SELECT COUNT(*) AS TotalProductCount
FROM product;
```

---

## 6. Оконные (window) функции — `OVER(...)`

Оконные функции позволяют вычислять агрегаты в контексте «окна» строк, не сворачивая их в группы — результат сохраняет исходное число строк.

Структура: `AGG(...) OVER (PARTITION BY col1, col2 ORDER BY colX ROWS BETWEEN ... )`.

- `PARTITION BY` — разделяет набор строк на группы (аналог GROUP BY для окна).
    
- `ORDER BY` внутри `OVER` задаёт порядок в окне (нужно для ранжирования и некоторых frame-оперций).
    

Примеры из заданий:

```sql
-- сумма продаж по территории, но оставить строки по-прежнему:
SELECT SalesYTD, SUM(SalesQuota) OVER (PARTITION BY TerritoryID) AS SumQuotaByTerritory
FROM sales_person;

-- минимальное значение бонуса по территории, но вернуть строку для каждого SalesPerson:
SELECT TerritoryID, MIN(Bonus) OVER (PARTITION BY TerritoryID) AS MinBonusByTerr
FROM sales_person
ORDER BY TerritoryID ASC;

-- использование WHERE вместе с оконной функцией (фильтрация до/после зависит от логики)
SELECT MAX(Bonus) OVER (PARTITION BY TerritoryID) AS MaxBonusWindow
FROM sales_person
WHERE BusinessEntityID IN (288, 277, 280);
```

**Важно:** `WHERE` выполняется до оконных агрегатов. Если нужно отфильтровать по результату оконной функции — используйте `QUALIFY` (в некоторых СУБД) или вложенный запрос / CTE.

---

## 7. Алиасы и понятность вывода

- Алиасы колонок: `AS alias` (можно опустить `AS` в большинстве СУБД).
    
- Алиасы таблиц: `FROM sales_person sp` — удобно при `JOIN` и когда имён длинные.
    

Пример:

```sql
SELECT sp.SalesYTD, SUM(sp.SalesQuota) OVER (PARTITION BY sp.TerritoryID) AS SumQuota
FROM sales_person AS sp;
```

---

## 8. Практические советы по написанию запросов

1. Всегда проверяйте NULL-значения (`IS NULL`), особенно в условиях и в `ORDER BY`.
    
2. Для поиска подстроки используйте `LIKE '%text%'`, помните про регистрозависимость (в некоторых СУБД — важно).
    
3. Чтобы не получать неожиданные результаты при группировке, явно указывайте `GROUP BY` и используйте алиасы для агрегатов.
    
4. При сложных выражениях разбивайте запрос на CTE (`WITH ...`) — так проще отлаживать.
    
5. Если нужно ограничить результат — используйте `LIMIT` / `OFFSET` или `FETCH FIRST` (в зависимости от СУБД) в конце.
    
6. Пользуйтесь индексами (на практике) — сортировка и фильтрация по индексированным полям быстрее.
    

---

## 10. Быстрая карта соответствия теории и примеры

(Раздел с прямыми ссылками на номера заданий удалён — в документе теперь только общие объяснения и шаблоны запросов.)

## 11. Полезные примеры-шаблоны (копировать и использовать)

```sql
-- 1) Поиск с несколькими условиями и сортировкой
SELECT col1, col2
FROM schema.table
WHERE col3 IS NOT NULL
  AND col4 NOT IN ('A', 'B')
ORDER BY col2 ASC, col1 DESC;

-- 2) Группировка и агрегат
SELECT category, COUNT(*) AS cnt, SUM(price) AS total
FROM schema.products
WHERE price > 0
GROUP BY category
HAVING SUM(price) > 1000
ORDER BY total DESC;

-- 3) Оконная функция
SELECT id, value, SUM(value) OVER (PARTITION BY category) AS sum_by_cat
FROM schema.table;
```

---

## 12. Заключение

Этот гайд покрывает ключевые концепции, нужные для выполнения типичных заданий: понимание отличий DDL/DML, базовый синтаксис `SELECT`, работа с `WHERE`, `ORDER BY`, `GROUP BY`, агрегатами и оконными функциями. Если хотите — могу подготовить подсказки и примеры конкретных запросов или сгенерировать `.md` файл для скачивания.

---

_Конец гайда._  
Полезные примеры-шаблоны (копировать и использовать)

```sql
-- 1) Поиск с несколькими условиями и сортировкой
SELECT col1, col2
FROM schema.table
WHERE col3 IS NOT NULL
  AND col4 NOT IN ('A', 'B')
ORDER BY col2 ASC, col1 DESC;

-- 2) Группировка и агрегат
SELECT category, COUNT(*) AS cnt, SUM(price) AS total
FROM schema.products
WHERE price > 0
GROUP BY category
HAVING SUM(price) > 1000
ORDER BY total DESC;

-- 3) Оконная функция
SELECT id, value, SUM(value) OVER (PARTITION BY category) AS sum_by_cat
FROM schema.table;
```

---