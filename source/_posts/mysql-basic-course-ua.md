---
title: Відеокурс "Основи MySQL"
date: 2023-11-22
tags:
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

   ##### Корисні посилання

   </summary>
     
   1) Пісочниці
   
   [http://sqlfiddle.com](http://sqlfiddle.com/) - проста пісочниця для тестування SQL запитів, спочатку в лівому вікні пишемо і запускаємо DDL запит (створення таблиць), потім в правому пишемо і виконуємо запити.
   
   [https://www.db-fiddle.com](https://www.db-fiddle.com/)  - більш складна пісочниця з можливістю групових сеансів.
   
   2) Матеріали
   
   [http://moonexcel.com.ua/уроки-sql-для-початківців-безкоштовно-онлайн_ua](http://moonexcel.com.ua/%D1%83%D1%80%D0%BE%D0%BA%D0%B8-sql-%D0%B4%D0%BB%D1%8F-%D0%BF%D0%BE%D1%87%D0%B0%D1%82%D0%BA%D1%96%D0%B2%D1%86%D1%96%D0%B2-%D0%B1%D0%B5%D0%B7%D0%BA%D0%BE%D1%88%D1%82%D0%BE%D0%B2%D0%BD%D0%BE-%D0%BE%D0%BD%D0%BB%D0%B0%D0%B9%D0%BD_ua) - непогані уроки по SQL українською мовою (багато реклами).
   
   https://acode.com.ua/sql-lessons/ - чудові уроки по SQL українською мовою і без реклами.
   
   3) Задачки
   
   [https://leetcode.com/problem-list/leetcode-curated-sql-70/?envType=featured-list](https://leetcode.com/problem-list/leetcode-curated-sql-70/?envType=featured-list&sorting=W3sic29ydE9yZGVyIjoiQVNDRU5ESU5HIiwib3JkZXJCeSI6IkRJRkZJQ1VMVFkifV0%3D&page=1) - список завдань по SQL, частково завдання платні, але є достатньо безкоштовних, різних рівнів складності.
</details>

<br/>
<br/>

<details>
   <summary>

   #### Дампи баз даних, що використовуються в завданнях

   </summary>

   {% gist 6050e4a21639d6fa973db898c7b26b1d smartphones-db-dump.sql %}
   {% gist 6050e4a21639d6fa973db898c7b26b1d cities-db-dump.sql %}
   {% gist 6050e4a21639d6fa973db898c7b26b1d product-store-db-dump.sql %}
   {% gist 6050e4a21639d6fa973db898c7b26b1d orders-db-dump.sql %}
</details>