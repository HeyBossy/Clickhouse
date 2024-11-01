#Теория
MergeTree (основной и самый важный движок) - это движок для хранения и обработки данных, который оптимизирован для работы с большими объемами данных и обеспечивает быстрый доступ к данным в определенном диапазоне дат. MergeTree может эффективно обрабатывать данные, поступающие в потоковом режиме, и может быстро выполнять запросы на выборку. На следующих степах я буду объяснять принцип работы движка, но возможно вам будет интересно заранее подготовиться к этому, необязательно, можете почитать про LSM деревья которые используются, хот ьи в обрезанном виду в ClickHouse.
#Задание
Задание делится на 2 части, создание необходимой таблицы и генерация данных + вставка в таблицу.

**Часть первая:**

В данном задании вам нужно будет создать таблицу с движком MergeTree со следующими параметрами.

Поля таблицы:
```sql
sale_date: дата продажи
product_id: идентификатор продукта
product_category: категория продукта
sale_amount: сумма продажи (добавьте значение по умолчанию DEFAULT 0 будет заполняться при вставке данных вместо NULL, также используется в некоторых SQL запросах например toInt64OrDefault)
sale_quantity: количество продаж
customer_id: идентификатор покупателя (добавьте комментарий с помощью такого кода COMMENT 'идентификатор пользователя')
store_id: идентификатор магазина
PARTITION BY: по месяцам по полю sale_date
ORDER BY: сначала по дате продажи, затем по идентификатору продукта и идентификатору покупателя
```
**Часть вторая:**

В полученную таблицу вставьте набор сгенерированных данных, скрипт генерации можете найти по данной ссылке. Чтобы вставить данные используйте такой код
```sql
INSERT INTO <your_table_name>
FORMAT JSONEachRow
[{json_data}]
```
Далее посчитайте сумму по полю sale_amount это и будет ответ. На следующем шаге нужно будет прислать код вашего решения.
# Решение
```sql
CREATE TABLE sales_data
(
    sale_date Date,
    product_id UInt32,
    product_category String,
    sale_amount Int64 DEFAULT 0,
    sale_quantity UInt32,
    customer_id UInt32 COMMENT 'идентификатор пользователя',
    store_id UInt32
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(sale_date)
ORDER BY (sale_date, product_id, customer_id);
 

 

INSERT INTO sales_data
FORMAT JSONEachRow

{"sale_date": "2020-01-06", "product_id": 2, "product_category": "pr2", "sale_amount": 24, "sale_quantity": 13, "customer_id": 10, "store_id": 74}, {"sale_date": "2020-01-27", "product_id": 8, "product_category": "pr2", "sale_amount": 3, "sale_quantity": 9, "customer_id": 23, "store_id": 60}, .......;

 

SELECT SUM(sale_amount)
FROM sales_data
```