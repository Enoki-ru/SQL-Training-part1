
Доброго времени суток. Сегодня мы будем с вами проходить много, очень много тестов  с тренажера по sql, который мне предложила Яндекс Стажировка перед тем как я буду проходить с ними интервью. Тренажер этот онлайновый, находится по ссылке 
$$https://sql-ex.ru/$$
*Let's go* как говорят у меня на будущей родине, погнали.

Допустим, у нас есть база данных "Компьютерная фирма", которая содержить несколько таблиц.
![[Pasted image 20230121161814.png]]

---
## Задания

**Задание: 1 ([Serge I](https://sql-ex.ru/users_page.php?username=Serge%20I): 2002-09-30)**  
Найдите номер модели, скорость и размер жесткого диска для всех ПК стоимостью менее 500 дол. Вывести: model, speed и hd

По заданию мы должны запросы делать именно pc таблице. Тут всё просто, 
```sql
select model, speed, hd from pc where price < 500
```
![[Pasted image 20230121162037.png]]

**Задание: 2 ([Serge I](https://sql-ex.ru/users_page.php?username=Serge%20I): 2002-09-21)**  
Найдите производителей принтеров. Вывести: maker

По заданию обращаться будем именно к основной таблице `product`
Но если мы просто попытаемся вывести те строки где `type = 'Printer'`
т.е. запрос будет
```sql
select maker from product where type = 'Printer'
```
То мы получим
![[Pasted image 20230121162323.png]]

Как видите, некоторые производители встречаются несколько раз у разных товаров. Тк нам нужны только уникальные названия (т.е. без повторений), то используем параметр
`distinct` чтобы оставлять только уникальные наименования
```sql
select distinct maker from product where type = 'Printer'
```
![[Pasted image 20230121162657.png]]

**Задание: 3 ([Serge I](https://sql-ex.ru/users_page.php?username=Serge%20I): 2002-09-30)**  
Найдите номер модели, объем памяти и размеры экранов ПК-блокнотов, цена которых превышает 1000 дол.

По аналогии с первым номером, только в другую таблице смотреть нужно
```sql
select model, ram, screen from laptop where price > 1000
```
![[Pasted image 20230121162938.png]]


**Задание: 4 ([Serge I](https://sql-ex.ru/users_page.php?username=Serge%20I): 2002-09-21)**  
Найдите все записи таблицы Printer для цветных принтеров.

Тут что-то новенькое. Теперь нам нужно вывести уже все строки, если в строке coror будет стоять $y$ 
*Для вывода всех строк таблицы используется `*` *
```sql
select * from printer where color = 'y'
```
![[Pasted image 20230121163128.png]]

**Задание: 5 ([Serge I](https://sql-ex.ru/users_page.php?username=Serge%20I): 2002-09-30)**  
Найдите номер модели, скорость и размер жесткого диска ПК, имеющих 12x или 24x CD и цену менее 600 дол.

Смотрим в таблицу ПК. Понимаю, это пока всё однотипно и понятно, так что вот просто ответ

```sql
select model, speed, hd from PC where (cd='12x' or cd='24x') and (price<600)
```

![[Pasted image 20230121212610.png]]

**Задание: 6 ([Serge I](https://sql-ex.ru/users_page.php?username=Serge%20I): 2002-10-28)**  
Для каждого производителя, выпускающего ПК-блокноты c объёмом жесткого диска не менее 10 Гбайт, найти скорости таких ПК-блокнотов. Вывод: производитель, скорость.

А вот это задача потруднее, так как у нас эти названия находятся в разных таблицах. Поэтому Нам нужно не просто вывести данные, а объединить их из разных таблица. Но как это сделать?
1) Нам нужно будет указать уникальные имена производителей, поэтому пишем distinct
2) Нам нужно будет объединить таблицы в одну, используя уникальные номера для объединения (номера моделей)
Таким образом запрос выглядит так
```sql
SELECT DISTINCT product.maker, laptop.speed FROM product INNER JOIN Laptop ON product.model=Laptop.model where laptop.hd >= 10
```
![[Pasted image 20230121215211.png]]

**Задание: 7 ([Serge I](https://sql-ex.ru/users_page.php?username=Serge%20I): 2002-11-02)**  
Найдите номера моделей и цены всех имеющихся в продаже продуктов (любого типа) производителя B (латинская буква).

Итак, какие есть идеи решения
1) Можно попробовать выбирать сначала для каждой таблицы те строки, где модели находятся в главной таблице, но с maker='B'.
 Тоесть как-то так
```sql
SELECT model,price FROM PC WHERE model IN (SELECT model FROM Product WHERE maker='B' and type = 'PC')
```

 ![[Pasted image 20230121221132.png]]
И даже вы скажете, что здесь видны повторения уже на этом этапе, то используя UNION объединение строк, повторения сами уйдут.
Таким образом решение будет выглядеть как-то так:
```SQL
SELECT model, price 
FROM PC 
WHERE model IN (SELECT model 
 FROM Product 
 WHERE maker = 'B' AND 
 type = 'PC'
 )
UNION
SELECT model, price 
FROM Laptop 
WHERE model IN (SELECT model 
 FROM Product 
 WHERE maker = 'B' AND 
 type = 'Laptop'
 )
UNION
SELECT model, price 
FROM Printer 
WHERE model IN (SELECT model 
 FROM Product 
 WHERE maker = 'B' AND 
 type = 'Printer'
)
```
![[Pasted image 20230121221427.png]]
Так и оказалось! Однако мне не очень нравится этот метод, он очень громоздкий. Давайте поэтому сначала объединим все строки PC Laptop Printer, а потом уже будем искать всё что нам нужно.
```sql
SELECT * FROM (SELECT model, price 
 FROM PC
 UNION
 SELECT model, price 
 FROM Laptop
 UNION
 SELECT model, price 
 FROM Printer
 ) AS a
WHERE a.model IN (SELECT model 
 FROM Product 
 WHERE maker = 'B'
 )
```
Прочитаем наш запрос.
*Выберем все значения из таблицы А (Таблица А содержит модели и цену из Laptop, Printer, PC) где модели в этой таблице находятся в таблице Б (таблица Б содержит модели из основной таблица Product, где производитель назван 'B')*
![[Pasted image 20230121222030.png]]
Также получилось! И более компактно


**Задание: 8 ([Serge I](https://sql-ex.ru/users_page.php?username=Serge%20I): 2003-02-03)**  
Найдите производителя, выпускающего ПК, но не ПК-блокноты.

*Тут я долго боролся, пытаясь поработать в разными таблицами, как вдруг понял, что можно использовать команду исключения EXCEPT. А смысл здесь такой. Находим сначала все уникальные названия производителей, которые имеют товары PC. А потом из них исключаем EXCEPT тех произв-й, которые имеют товары Laptop*
```sql
SELECT DISTINCT p.maker FROM Product p WHERE p.type='PC'
EXCEPT
SELECT DISTINCT p.maker FROM Product p WHERE p.type='Laptop'
```
![[Pasted image 20230121224824.png]]

Можете потом почитать FAQ для этой задачи, там всё сложно и неправильно к тому же но ладно http://www.sql-tutorial.ru/book_exercise_8.html 

**Задание: 9 ([Serge I](https://sql-ex.ru/users_page.php?username=Serge%20I): 2002-11-02)**  
Найдите производителей ПК с процессором не менее 450 Мгц. Вывести: Maker


Значит, объединим по номерам моделей две таблицы, чтобы узнать, у какого подходящего по скорости проца какой производитель, и выведем уникальных производителей )
```sql
SELECT DISTINCT p.maker FROM product p INNER JOIN PC pc ON p.model=pc.model
WHERE speed >=450

```
![[Pasted image 20230121225723.png]]

**Задание: 10 ([Serge I](https://sql-ex.ru/users_page.php?username=Serge%20I): 2002-09-23)**  
Найдите модели принтеров, имеющих самую высокую цену. Вывести: model, price

*Чтобы это решить, мы должны вывести модель и цены тех принтеров, цена которых равна максимальному значению. А само максимально значение можно найти, создав отдельно таблицу со стобцом MAX(price), созданной из этой же таблицы Принтеров*

```sql
SELECT model, price FROM Printer
WHERE price = (SELECT MAX(price) FROM Printer)

```
![[Pasted image 20230121230619.png]]