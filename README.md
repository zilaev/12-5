# Решение домащнего задания к занятию 12.5. «Индексы» - Зилаев Денис



### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.
```
SELECT table_schema as DB_name
	,CONCAT(ROUND((SUM(index_length))*100/(SUM(data_length+index_length)),2),'%') '% of index'
FROM information_schema.TABLES where TABLE_SCHEMA = 'sakila'
```
![screen](https://github.com/zilaev/12-5/blob/main/percent.png)

### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

Ответ:
После анализа исходного запроса было выявляено, что наиболее узким местом в предлагаемом запросе является то, 
что оконная функция обрабатывает излишние таблицы а именно inventory, rental и film. Т.к. нужно посчитать сумму платежей покупателей 
за конкретную дату, обработка и присоединение этих таблиц не имеет смысла т.к. дальше данные не используются.
Все необходимые данные есть в таблицах payment и customer, соответственно, остальные таблицы можно исключить.
Предлагается оптимизировать запрос следующим образом:
```explain analyze
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id)
from payment p, customer c
where date(p.payment_date) = '2005-07-30' and p.customer_id = c.customer_id 
```
Сравним, к примеру actual time:
actual time исходного запроса на моей системе составляет 9018.977
actual time оптимизированного запроса составляет 11,796.


