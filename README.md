# Домашнее задание к занятию "`Индексы`" - `Татаринцев Алексей`




### Задание 1

`Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.`

1. `Запускаю установленную баз данных sakila`
2. `Составляю запрос`

```
SELECT
    (SUM(INDEX_LENGTH) / SUM(DATA_LENGTH + INDEX_LENGTH)) * 100 AS index_to_table_ratio
FROM
    information_schema.TABLES
WHERE
    TABLE_SCHEMA = 'sakila';

```
![1](https://github.com/Foxbeerxxx/index/blob/main/img/img1.png)`



### Задание 2

`Выполните explain analyze следующего запроса:`
```
SELECT DISTINCT
    concat(c.last_name, ' ', c.first_name),
    SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title)
FROM
    payment p
JOIN
    rental r ON p.payment_date = r.rental_date
JOIN
    customer c ON r.customer_id = c.customer_id
JOIN
    inventory i ON r.inventory_id = i.inventory_id
JOIN
    film f ON i.film_id = f.film_id
WHERE
    DATE(p.payment_date) = '2005-07-30';
```

1. `Узкие места`
#
Использование DATE() в WHERE
DATE(p.payment_date) = '2005-07-30' является серьезной проблемой производительности. Функция DATE() применяется к каждому значению p.payment_date
#
p.payment_date = r.rental_date - Это нелогичный JOIN. Столбцы payment_date и rental_date имеют разное значение. Связь должна быть через корректные столбцы, скорее всего rental_id.
#
Вместо DATE(p.payment_date) = '2005-07-30' можно использовать диапазон.
#
Можно заменить JOIN на явные JOIN ON.


2. `Оптимизация`

```
EXPLAIN ANALYZE
SELECT DISTINCT
    concat(c.last_name, ' ', c.first_name),
    SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title)
FROM
    payment p
JOIN
    rental r ON p.rental_id = r.rental_id  
JOIN
    customer c ON r.customer_id = c.customer_id
JOIN
    inventory i ON r.inventory_id = i.inventory_id
JOIN
    film f ON i.film_id = f.film_id
WHERE
    p.payment_date >= '2005-07-30 00:00:00' AND p.payment_date < '2005-07-31 00:00:00';

CREATE INDEX idx_payment_payment_date ON payment (payment_date);
CREATE INDEX idx_rental_customer_id ON rental (customer_id);
CREATE INDEX idx_rental_inventory_id ON rental (inventory_id);
CREATE INDEX idx_customer_customer_id ON customer (customer_id);
CREATE INDEX idx_inventory_film_id ON inventory (film_id);
CREATE INDEX idx_film_film_id ON film (film_id);
CREATE INDEX idx_rental_rental_id ON rental (rental_id);
CREATE INDEX idx_payment_rental_id ON payment (rental_id);

```