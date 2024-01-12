## SQL

**Создание таблицы**  

CREATE TABLE  таблица 
             ( id          INT PRIMARY KEY AUTO_INCREMENT, 
               name    VARCHAR(30) ); 

**Вставка значений**  

INSERT INTO  таблица(поле1, поле2) 
             VALUES (значение1, значение2); 

**Выборка данных**  

SELECT столбцы FROM таблица;

**Выборка вычисляемых значений**  

SELECT title, amount, price, 
    IF(amount<4, price*0.5, price*0.7) AS sale
FROM book; 

**Выборка по условию**  

SELECT title, amount 
FROM book
WHERE amount BETWEEN 5 AND 14; 

SELECT title, price 
FROM book
WHERE author IN ('Булгаков М.А.', 'Достоевский Ф.М.');  

SELECT title 
FROM book
WHERE title LIKE 'Б%';  

**Выборка уникальных значений**  

SELECT DISTINCT author
FROM book;  


**Выборка данных, групповые функции MIN, MAX и AVG**  

SELECT author, 
    MIN(price)     AS Минимальная_цена, 
    MAX(price)         AS Максимальная_цена, 
    ROUND(AVG(price),2)        AS Средняя_цена
FROM book
GROUP BY author
HAVING SUM(price * amount) > 5000
ORDER BY Минимальная_цена DESC; 

WHERE и HAVING могут использоваться в одном запросе. При этом необходимо учитывать порядок выполнения SQL запроса на выборку на СЕРВЕРЕ: 
1.	FROM
2.	WHERE
3.	GROUP BY
4.	HAVING
5.	SELECT
6.	ORDER BY

_Пример_ Вывести максимальную и минимальную цену книг каждого автора, кроме Есенина, количество экземпляров книг которого больше 10.

SELECT author,
    MIN(price) AS Минимальная_цена,
    MAX(price) AS Максимальная_цена
FROM book
WHERE author <> 'Есенин С.А.'
GROUP BY author
HAVING SUM(amount) > 10;

_Пример_ Посчитать стоимость всех экземпляров каждого автора без учета книг «Идиот» и «Белая гвардия». В результат включить только тех авторов, у которых суммарная стоимость книг (без учета книг «Идиот» и «Белая гвардия») более 5000 руб. Вычисляемый столбец назвать Стоимость. Результат отсортировать по убыванию стоимости.

SELECT author,
    SUM(price * amount) AS Стоимость
FROM book
WHERE title <> 'Идиот' AND title <> 'Белая гвардия'
GROUP BY author
HAVING Стоимость > 5000


ORDER BY author DESC;
