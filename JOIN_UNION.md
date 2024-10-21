```JOIN, UNION, WITH```
# Задача 1
Ваша задача преобразовать ```Age``` в таблице ```Титаник``` с помощью функции выше и посчитать сколько будет строк с пропущенными значениями, считайте по полю ```Age```. Для проверки является ли строка ```NULL``` воспользуйтесь функцией ```isNull(...)```
которая вернет только ```NULL``` строки.  Если вы попытаетесь, посчитать таким образом ```count(Age)``` то ответом будет 0 Так как ClickHouse не берет во внимание строки с ```NULL``` значением. Чтобы получить корректный ответ воспользуйтесь функцией ```ifNull(x,alt)```

# Решение
```sql
SELECT count(*)
FROM default.titanic
WHERE isNull(toFloat64OrNull(Age))
```
# Задача 2
**Описание задачи:**

Для начала простой запрос на JOIN. У нас есть таблица с выручкой в рублях и таблица с курсом USD-RUB. Объедините данные и посчитайте выручку в долларах. 

Скрипт для генерации данных 
```sql
CREATE TABLE revenue_rub 
(
    date	String,
    revenue_rub	UInt64
)engine=Log

INSERT INTO revenue_rub (*) VALUES
	('2023-04-01', '2461712'),
	('2023-04-02', '1904536'),
	('2023-04-03', '3021203'),
	('2023-04-04', '3217854'),
	('2023-04-05', '2312401'),
	('2023-04-06', '2522456'),
	('2023-04-07', '3271421'),
	('2023-04-08', '2708432'),
	('2023-04-09', '2785278'),
	('2023-04-10', '2716433'),
	('2023-04-11', '2815574'),
	('2023-04-12', '2876831'),
	('2023-04-13', '2015123'),
	('2023-04-14', '1802406'),
	('2023-04-15', '2698974'),
	('2023-04-16', '2473162'),
	('2023-04-17', '2346825'),
	('2023-04-18', '2927760'),
	('2023-04-19', '2367126'),
	('2023-04-20', '2457581'),
	('2023-04-21', '2062530'),
	('2023-04-22', '1898335'),
	('2023-04-23', '2177096'),
	('2023-04-24', '2235702'),
	('2023-04-25', '2212442'),
	('2023-04-26', '2189182'),
	('2023-04-27', '2165921'),
	('2023-04-28', '2142661'),
	('2023-04-29', '2119400'),
	('2023-04-30', '2096140');
CREATE TABLE usd_rub 
(
    date	String,
    usdrub	Float64
) engine=Log

INSERT INTO usd_rub(*) VALUES
	('2023-04-01', '80.3'),
	('2023-04-02', '81.3'),
	('2023-04-03', '81.6'),
	('2023-04-04', '81.6'),
	('2023-04-05', '81.3'),
	('2023-04-06', '81.7'),
	('2023-04-07', '81.6'),
	('2023-04-08', '81.9'),
	('2023-04-09', '81.5'),
	('2023-04-10', '81.4'),
	('2023-04-11', '81.8'),
	('2023-04-12', '81.4'),
	('2023-04-13', '82'),
	('2023-04-14', '82'),
	('2023-04-15', '81.6'),
	('2023-04-16', '81.1'),
	('2023-04-17', '81.4'),
	('2023-04-18', '79.8'),
	('2023-04-19', '79.5'),
	('2023-04-20', '78.7'),
	('2023-04-21', '77.6'),
	('2023-04-22', '77.1'),
	('2023-04-23', '77.2'),
	('2023-04-24', '76.2'),
	('2023-04-25', '75.6'),
	('2023-04-26', '75'),
	('2023-04-27', '74.4'),
	('2023-04-28', '73.8'),
	('2023-04-29', '73.2'),
	('2023-04-30', '72.6');
```
**Формат ответа:**

В ответ впишите только целую часть без округления. 
# Решение
```sql
WITH join_df AS (
    SELECT *
    FROM revenue_rub
    INNER JOIN usd_rub USING (date)
)
SELECT SUM(revenue_rub / usdrub) AS revenue_usd
FROM join_df
```
# Задача 3
**Описание задачи:**

Давайте оценим как менялись глобальные продаже от года к году для приставок ```PS3, PS2, X360, Wii```. 

Для этого нужно выполнить следующие шаги. Посчитать какие были продажи за каждый год по каждой платформе,  отфильтровать строки с пустым значением колонки ```Year``` Затем нужно продублировать данный запрос и объединить два одинаковых запроса друг с другом, так чтобы данные за предыдущий год были в текущем. Ссылка на описание датасета. 

У вас возникнут проблемы с датой, воспользуйтесь функцией которую мы изучил ранее ```parseDateTimeBestEffort```

В задаче нужно использовать ```ASOF JOIN``` для объединения данных. Также у вас может возникнуть сложность при объединении если вы воспользуетесь таким синтаксисом ```using(id, dt)``` в данной задаче нужно использовать ```ASOF JOIN``` с таким синтаксисом ```ON ...=... and ... > или < ...```  Последнее условие указывает на то по какому принципу объединять данные которые не совпадают.
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
**Формат ответа:**

После чего просто возьмите сумму от столбца разницы текущего и предыдущего года. У вас получится отрицательное число, впишите ответ по модулю округленный до целого числа. Важно, отфильтруйте все строки где предыдущий год равен 0

# Решение
```sql
WITH filtering_Year AS (
    SELECT 
        Platform, 
        Year, 
        (sumIf(NA_Sales, Platform IN ['PS3', 'PS2', 'X360', 'Wii']) + 
         sumIf(EU_Sales, Platform IN ['PS3', 'PS2', 'X360', 'Wii']) + 
         sumIf(JP_Sales, Platform IN ['PS3', 'PS2', 'X360', 'Wii']) + 
         sumIf(Other_Sales, Platform IN ['PS3', 'PS2', 'X360', 'Wii']) + 
         sumIf(Global_Sales, Platform IN ['PS3', 'PS2', 'X360', 'Wii'])) AS Total_Sales_Sum
    FROM default.video_game_sales
    WHERE Year != ''  
    AND Year != '0'  
    GROUP BY Platform, Year
    ORDER BY Platform, Year
)
SELECT 
    round(abs(SUM(a.Total_Sales_Sum - b.Total_Sales_Sum))) AS Total_Sales_Difference  
FROM (
    SELECT 
        Platform, 
        toInt32(Year) AS Year,  
        Total_Sales_Sum
    FROM filtering_Year
) AS a
ASOF LEFT JOIN (
    SELECT 
        Platform, 
        toInt32(Year) AS Year,  
        Total_Sales_Sum
    FROM filtering_Year
) AS b
ON a.Platform = b.Platform AND b.Year < a.Year  
WHERE b.Total_Sales_Sum != 0 -- Выходит 402..
```
# Задача 4
**Описание задачи:**


В данном задании нам нужно будет вычислить вероятность выжить на Титанике для различных групп пассажиров и выяснить сколько людей из тех у кого были самые маленькие шансы на спасение в итоге выжил. Задание большое! Шаги которые нужно выполнить для этого:
# Анализ данных о пассажирах Титаника

## Шаг 1. Подготовка данных

Для начала нужно выделить столбцы, по которым будем считать вероятность выживания пассажира: `Pclass`, `Sex`, `Age`.

Для поля `Age` нам нужно выполнить следующие действия:
- Преобразовать тип в `Float`.
- Заполнить пустые значения, используя среднее значение (`avg`), игнорируя пустые значения.

На выходе мы должны получить таблицу с такими полями:
- `Pclass`
- `Sex`
- `Age`
- `Survived`

## Шаг 2. Округление возраста

Для поля `Age` воспользуемся функцией `roundAge()` для округления до различных возрастных групп.

## Шаг 3. Расчет вероятности выживания

Посчитаем вероятность выжить по полям `Pclass`, `Sex`, `Age`. Вероятность выживания считается по формуле:

$$
\text{prob} = \frac{\text{count(SurvivedUser)}}{\text{count(AllUser)}}
$$

Где:
- `count(SurvivedUser)` — количество выживших пассажиров в конкретной группе.
- `count(AllUser)` — общее количество пассажиров в группе.

## Шаг 4. Добавление столбца классификации

Создадим дополнительный столбец `pss`, в котором пассажиры с вероятностью выживания (`prob`) больше 0.1 будут называться `lucky`, а остальные — `other`:

```sql
CASE WHEN prob > 0.1 THEN 'lucky' ELSE 'other' END AS pss
```
# Решение
```sql
WITH  
-- получаем вероятности
get_prob AS (
    SELECT 
        Pclass, 
        Sex, 
        roundAge(ifNull(toFloat32OrNull(Age), (SELECT avg(toFloat32OrNull(Age)) FROM default.titanic))) AS Age,
        count() AS total_passengers,
        countIf(Survived = 1) AS survived_passengers,
        round(survived_passengers / total_passengers, 2) AS prob,
        multiIf(round(survived_passengers / total_passengers, 2) > 0.1, 'lucky', 'other') AS pss
    FROM 
        default.titanic
    GROUP BY 
        Pclass, 
        Sex, 
        Age
),
-- тут джойн
final_data AS (
    SELECT 
        t.Pclass, 
        t.Sex, 
        t.Survived,
        roundAge(ifNull(toFloat32OrNull(t.Age), (SELECT avg(toFloat32OrNull(Age)) FROM default.titanic))) AS Age,
        g.pss
    FROM 
        default.titanic t
    INNER JOIN 
        get_prob g
    ON 
        t.Pclass = g.Pclass
        AND t.Sex = g.Sex
        AND roundAge(ifNull(toFloat32OrNull(t.Age), (SELECT avg(toFloat32OrNull(Age)) FROM default.titanic))) = g.Age
)
--посдчет other
SELECT 
    countIf(Survived = 1 AND pss = 'other') AS other_survived
FROM 
    final_data -- выходит 46
```
