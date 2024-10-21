# Clickhause
Уроки Кликхауса ( в основном SQL)

Мой гугл док с описнаием функций SQL
https://docs.google.com/document/d/1PWSHJNFfmrYEIgtiFuFpfPoaJLhKcUKbvTjAI-z8k7Q/edit?usp=sharing 

Запуск контнейнеров (кликхаус и веб-клиент)
```
docker run -d --name clickhouse-play -p 8123:8123 -p 3000:3000 -p 9000:9000 clickhouse/clickhouse-server
```
```
docker run -d -p 8080:80 spoonest/clickhouse-tabix-web-client
```
