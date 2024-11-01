# Теория
**AggregatingMergeTree** - это движок, который предназначен для агрегации данных. Он позволяет группировать данные по определенному ключу и выполнять аггрегационные функции при вставке данных, такие как сумма или среднее значение, над группами данных.

Описание документации ClickHouse:

Движок, основанный на AggregatingMergeTree, наследует функциональность MergeTree, но изменяет логику слияния блоков данных. ClickHouse заменяет все строки с одинаковым первичным ключом (точнее, с одинаковым ключом сортировки) на одну (в пределах одного блока данных), которая хранит объединение состояний агрегатных функций.

Таблицы типа AggregatingMergeTree могут использоваться для инкрементальной агрегации данных, в том числе для агрегирующих материализованных представлений.

Немного сложно, но давайте разберемся на практике что все это означает. создадим таблицу аналогичную той которую мы создавали ранее, но с движком AggregatingMergeTree
```
CREATE TABLE sale (
    sale_date Date,
    product_id UInt32,
    product_category String,
    sale_amount Float64,
    sale_quantity SimpleAggregateFunction(max, UInt32),
    customer_id UInt32,
    store_id UInt32
) ENGINE = AggregatingMergeTree()
PARTITION BY toYYYYMM(sale_date)
ORDER BY (product_category)
```
Некоторые изменения в запросе, мы добавили SimpleAggregateFunction(max, UInt32), это означает лишь то что данный столбец должен выбирать max значение при агрегировании

Мы получили 3 строки которые были агрегированы, те колонки которые не имели дополнительную инструкцию по агрегации, были взяты из первого попавшегося значения (похоже на any)

Когда полезно использовать данный движок, в своей практике я использовал его для агрегирования таблицы с большим потоком данных, то есть мы получали много-много однотипных событий, но по факту нас не интересовали все эти события в сыром виде, тогда мы повесили данный движок и смогли сэкономить на хранении информации. 
# Задача
Создайте таблицу аналогичную предыдущей с движком AggregatingMergeTree и ключём сортировки product_category 

Укажите поля агрегации

- sale_amount как ``SUM``
- customer_id как ``any``

Поля таблицы:

- ``sale_date``: дата продажи
- ``product_id``: идентификатор продукта
- ``product_category``: категория продукта
- ``sale_amount``: сумма продажи 
- ``sale_quantity``: количество продаж
- ``customer_id``: идентификатор покупателя 
- ``store_id``: идентификатор магазина
- ``PARTITION BY``: по месяцам по полю sale_date
- ``ORDER BY``: product_category

После чего сгенерируйте данные и вставьте в таблицу, код для генерации данных. Вставка данных аналогично предыдущей задаче:
```sql
INSERT INTO <your_table_name>
FORMAT JSONEachRow
[{json_data}]
```
В качестве ответа укажите max(sale_amount) в полученной таблице и умножьте это на количество строк в таблице.
# Решение
```sql
CREATE TABLE sales_data
(
    sale_date Date,
    product_id UInt32,
    product_category String,
    sale_amount SimpleAggregateFunction(sum, Int64) DEFAULT 0,
    sale_quantity UInt32,
    customer_id SimpleAggregateFunction(any, UInt32) COMMENT 'идентификатор пользователя',
    store_id UInt32
)
ENGINE = AggregatingMergeTree()
PARTITION BY toYYYYMM(sale_date)
ORDER BY (product_category)

 

INSERT..

SELECT 
    max(sale_amount) AS MAX_SALE,
    MAX_SALE * COUNT(1) AS RESULT
FROM sales_data
 

```