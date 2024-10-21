```GROUP BY, NOT IN```
# Задание
Перечислите через запятую в порядке убывания: в какие годы топ 5 издателей по продажам за все время превышали продажи Global_Sales всех остальных более чем на 60 Ссылка на описание датасета. 

Подсказка: в комбинатор функций можно вставлять подзапросы sumIf( Global_Sales, publisher not in (SELECT ...))

План запроса 

Первый столбец - год 
Второй - продажи топ 5 издателей 
Третий - продажи кроме топ 5 издателей 
Четвертый - посчитайте разницу между 2 и 3 
Отсортируйте результаты по годам и оставьте только те что удовлетворяют условию >60
Создание таблицы:
```sql
CREATE TABLE video_game_sales (
    Rank UInt32,
    Name String,
    Platform String,
    Year String,
    Genre String,
    Publisher String,
    NA_Sales Float32,
    EU_Sales Float32,
    JP_Sales Float32,
    Other_Sales Float32,
    Global_Sales Float32
) ENGINE = Log
```
```sql
INSERT INTO video_game_sales SELECT * FROM url('https://raw.githubusercontent.com/dmitrii12334/clickhouse/main/vgsale', CSVWithNames, 'Rank UInt32,
    Name String,
    Platform String,
    Year String,
    Genre String,
    Publisher String,
    NA_Sales Float32,
    EU_Sales Float32,
    JP_Sales Float32,
    Other_Sales Float32,
    Global_Sales Float32');
```
Формат ответа:

Годы через запятую в порядке убывания.
# Ответ
```sql
SELECT 
    Year, 
    sumIf(Global_Sales, Publisher IN (
        SELECT Publisher 
        FROM default.video_game_sales 
        GROUP BY Publisher 
        ORDER BY SUM(Global_Sales) DESC 
        LIMIT 5
    )) AS top5,
    
    sumIf(Global_Sales, Publisher NOT IN (
        SELECT Publisher 
        FROM default.video_game_sales 
        GROUP BY Publisher 
        ORDER BY SUM(Global_Sales) DESC 
        LIMIT 5
    )) AS not_top5,
    
    (top5 - not_top5) AS diff
FROM 
    default.video_game_sales
GROUP BY 
    Year
HAVING 
    diff > 60
ORDER BY 
    Year DESC
 
