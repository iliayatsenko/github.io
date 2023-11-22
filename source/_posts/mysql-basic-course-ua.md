---
title: Відеокурс "Основи MySQL"
date: 2023-11-22
tags:
    - video
    - mysql
categories:
    - UA
---

Мав нагоду повикладати основи MySQL <!-- more --> на курсі від [БФ "Філософія добра"](https://phk.com.ua). Мій перший досвід викладання. Записи моїх занять:

#### 1) Створення бази і таблиці, типи даних
{% youtube hnoJVge6b9A %}

<br/>

#### 2) Створення індексів та зовнішніх ключів
{% youtube T4P7HYtpSAY %}

<br/>

#### 3) Запити, WHERE, вставка, оновленя та видаленя записів
{% youtube wFL03Szdi8o %}

<details>
   <summary>

   ##### Завдання
   
   </summary>

   {% gist 6050e4a21639d6fa973db898c7b26b1d 1-select-distinct.txt %}
   {% gist 6050e4a21639d6fa973db898c7b26b1d 2-where.txt %}
   {% gist 6050e4a21639d6fa973db898c7b26b1d 3-update.txt %}
   {% gist 6050e4a21639d6fa973db898c7b26b1d 4-insert.txt %}
   {% gist 6050e4a21639d6fa973db898c7b26b1d 5-in-between-like-regexp.txt %}
   {% gist 6050e4a21639d6fa973db898c7b26b1d 6-order-by-limit.txt %}

</details>

<br/>

#### 4) GROUP BY, агрегуючі функції, HAVING, підзапити
{% youtube 5wFFvPk2xLc %}

<details>
   <summary>

   ##### Завдання

   </summary>

   {% gist 6050e4a21639d6fa973db898c7b26b1d 7-group-by-aggregations-having.txt %}
</details>

<br/>

#### 5) EXISTS, JOIN, підзапити
{% youtube FgVGbz4h1lg %}

<details>
   <summary>

   ##### Завдання

   </summary>

   {% gist 6050e4a21639d6fa973db898c7b26b1d 8-subqueries.txt %}
   {% gist 6050e4a21639d6fa973db898c7b26b1d 9-join.txt %}
</details>

<br/>

#### 6) UNION, вбудовані функції, час і дата
{% youtube eJeEEurEEZ8 %}

<details>
   <summary>

   ##### Завдання

   </summary>

   {% gist 6050e4a21639d6fa973db898c7b26b1d 10-union.txt %}
   {% gist 6050e4a21639d6fa973db898c7b26b1d 11-functions.txt %}
</details>

<br/>
<br/>

<details>
   <summary>

   #### Дампи

   </summary>

   {% gist 6050e4a21639d6fa973db898c7b26b1d smartphones-db-dump.sql %}
   {% gist 6050e4a21639d6fa973db898c7b26b1d cities-db-dump.sql %}
   {% gist 6050e4a21639d6fa973db898c7b26b1d product-store-db-dump.sql %}
   {% gist 6050e4a21639d6fa973db898c7b26b1d orders-db-dump.sql %}
</details>