# test_task
Тестовое задание


Задание 4.
```sql
SELECT o.*
FROM orders o
JOIN payments p ON o.id = p.order_id
WHERE o.status != 'paid'
  AND p.status = 'paid';
```
