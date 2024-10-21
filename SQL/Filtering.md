<blockquote>Функции: WHERE, multiIf, LIKE </blockquote>

## Задача 1
В таблице titanic найти имя человека который

- Женщина
- Первый класс 
- Не выжила

В ответе указать человека с самым большим ID
## SQL-код
```sql
SELECT *
from titanic
where Sex = 'female' AND  Pclass = 1 AND Survived = 0
```
## Задача 2 
В таблице titanic разметить пассажиров и найти человека по условию

Создать дополнительный столбец chance_survived в котором

- Пассажир который женщина 1 или 2 класса мы будем считать **lucky** (больше шансов выжить)
- Пассажир который женщина 3 класса мы будем считать **fortunate** (очевидно что шансов меньше, но они есть)
- Для остальных просто оставьте **other**

Найти имя человека который

- В имени присутствовало ```Miss```
- Оканчивался текст ```a```
- Начинался текст с ```L```
  
В качестве ответа написать имя пассажира, который оказался lucky с самым большим PassengerId
## SQL-код
```sql
SELECT Pclass, Name, PassengerId,
multiIf(Pclass = 1, 'lucky', Pclass = 2, 'lucky', Pclass = 3, 'fortunate', 'other') AS chance_survived
FROM titanic
WHERE Name LIKE '%Miss%' AND Name LIKE '%a' AND Name LIKE 'L%' AND  chance_survived = 'lucky'
ORDER BY PassengerId DESC
LIMIT 1
```
