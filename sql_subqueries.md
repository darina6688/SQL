# Подзапросы  

Данные для следующих задач: Airbnb в Берлине. Не забывайте о преобразовании столбцов к нужному типу данных!
**_Имеются следующие таблицы:_**   

**1. listings** – информация о жилье, включая полные описания, характеристики и средние оценки в отзывах; поскольку столбцов очень много, нужные перечислены в текстах самих задач  

**2. calendar_summary** – информация о доступности и цене того или иного жилья по дням  
- **_listing_id_** – идентификатор жилья (объявления)  
- **_date_** – дата  
- **_available_** – доступность жилья в данный день (t/f)  
- **_price_** – цена за ночь  

**3. reviews** – отзывы  
- **_listing_id_** –  идентификатор жилья  
- **_id_** – id отзыва  
- **_date_** – дата отзыва  
- **_reviewer_id_** – id ревьюера (автора отзыва)  
- **_reviewer_name_** – имя автора  
- **_comments_** – сам отзыв  
  ____________
*Сначала оставьте только те объявления, в которых оценка на основе отзывов выше среднего, а число отзывов в месяц составляет строго меньше трёх. 
Затем отсортируйте по убыванию две колонки: сначала по числу отзывов в месяц, потом по оценке. В качестве ответа укажите id объявления из первой строки.*
  
```sql
SELECT 
    id,
    toFloat64OrNull(review_scores_rating) as rsr,   --приводим колонку к численному виду
    reviews_per_month
FROM
    listings
WHERE
    rsr > (SELECT AVG(toFloat64OrNull(review_scores_rating))
                            FROM listings)
    and reviews_per_month < 3
ORDER BY reviews_per_month DESC, rsr DESC
```
__________
*Посчитайте среднее расстояние до центра города и выведите идентификаторы объявлений о сдаче отдельных комнат, для которых расстояние оказалось меньше среднего. Результат отсортируйте по убыванию, тем самым выбрав комнату, которая является наиболее удаленной от центра, но при этом расположена ближе, чем остальные комнаты в среднем.* 

```sql
SELECT 
    id,
    host_id,
    geoDistance(13.4050, 52.5200, toFloat64OrNull(longitude), toFloat64OrNull(latitude)) as dist    --приводим широту и долготу к численному виду

FROM
    listings
WHERE room_type = 'Private room' and dist < (SELECT AVG(geoDistance(13.4050, 52.5200, toFloat64OrNull(longitude), toFloat64OrNull(latitude)))
                                            FROM listings
                                            WHERE room_type = 'Private room')   
ORDER BY dist DESC
```
Вариант с WITH:
```sql

WITH (SELECT AVG(geoDistance(13.4050, 52.5200, toFloat64OrNull(longitude), toFloat64OrNull(latitude)))
                                            FROM listings
                                            WHERE room_type = 'Private room') as avg_dist
SELECT 
    id,
    host_id,
    avg_dist,
    geoDistance(13.4050, 52.5200, toFloat64OrNull(longitude), toFloat64OrNull(latitude)) as dist    
FROM
    listings
WHERE room_type = 'Private room' and dist < avg_dist    
ORDER BY dist DESC
```
__________
Представим, что вы планируете снять жилье в Берлине на 7 дней, используя более хитрые фильтры, чем предлагаются на сайте.
Отберите объявления из таблицы listings, которые:
- находятся на расстоянии от центра меньше среднего (здесь нам пригодится запрос из предыдущего задания)
- обойдутся дешевле 100$ в день (price с учетом cleaning_fee, который добавляется к общей сумме за неделю, т.е его нужно делить на кол-во дней)
- имеют последние отзывы (last_review), начиная с 1 сентября 2018 года
- имеют WiFi в списке удобств (amenities)  
Отсортируйте полученные значения по убыванию review_scores_rating (не забудьте перевести строку к численному виду) и в качестве ответа укажите host_id из первой строки.  

*В этом задании пригодится подзапрос из предыдущего шага. 
И функция `multiSearchAnyCaseInsensitive(колонка, ['искомая_подстрока'])`, которая ищет заданную подстроку в строке вне зависимости от регистра и выдает 0 в случае, если подстрока не была найдена.*  
  *Подсказки:*
1. привести колонки к **численному виду** (использовав `toFloat64OrNull()`)
2. правильно указать цену за уборку `cleaning fee`(указана за неделю)
3. отобрать объявления, которые обойдутся **дешевле** 100$ в день
4. для этого очистить колонки с ценой от лишних символов `replaceRegexpAll(price, '[$,]', '')` и привести их к численному виду
5. отобрать объявления, где в списке удобств (`amenities`) указан WiFi *multiSearchAnyCaseInsensitive*
6. отсортировать полученные значения **по убыванию** `review_scores_rating` (приведя перед этим колонку к численному виду)
7. отобрать последние отзывы (`last_review`), начиная с `2018-09-01`

```sql
SELECT 
    id,
    host_id,
    toFloat64OrNull(review_scores_rating) as rsr,
    price
FROM
    (SELECT 
        id,
        host_id,
        geoDistance(13.4050, 52.5200, toFloat64OrNull(longitude), toFloat64OrNull(latitude)) as dist,
        review_scores_rating,
        price,
        amenities,
        last_review,
        cleaning_fee
    FROM
        listings
    WHERE room_type = 'Private room' and dist < (SELECT AVG(geoDistance(13.4050, 52.5200, toFloat64OrNull(longitude), toFloat64OrNull(latitude)))
                                                FROM listings
                                                WHERE room_type = 'Private room')   
    )
    
WHERE (toFloat64OrNull(replaceRegexpAll(price, '[$,]', '')) + toFloat64OrNull(replaceRegexpAll(cleaning_fee, '[$,]', '')) / 7) < 100
    AND multiSearchAnyCaseInsensitive(amenities, ['wifi'])!=0 
    AND last_review >= '2018-09-01'
    
ORDER BY rsr DESC  
```
__________
Давайте найдем в таблице calendar_summary те доступные (available='t') объявления, у которых число отзывов от уникальных пользователей в таблице reviews выше среднего.  
NB! Для простоты будем считать, что отзыв — это уникальный посетитель на уникальное жилье, не учитывая возможные повторные отзывы от того же посетителя.  
Для этого с помощью конструкции WITH посчитайте среднее число уникальных reviewer_id из таблицы reviews на каждое жильё, потом проведите джойн таблиц calendar_summary и reviews по полю listing_id (при этом из таблицы calendar_summary должны быть отобраны уникальные listing_id, отфильтрованные по правилу available='t'). Результат отфильтруйте так, чтобы остались только записи, у которых число отзывов от уникальных людей выше среднего. Отсортируйте результат по возрастанию listing_id

?```sql
WITH (SELECT AVG(COUNT(DISTINCT reviewer_id))
      FROM reviews
      GROUP BY listing_id)  as avg_rev
      
SELECT 
    listing_id,
    COUNT(reviewer_id)  as cnt_rev 
FROM 
    (SELECT    
    	listing_id,
    	COUNT(reviewer_id)
    FROM calendar_summary    
    WHERE available='t'     
    GROUP BY listing_id
    HAVING COUNT(DISTINCT reviewer_id) > avg_rev
    ) AS c
JOIN 
    (SELECT  
        listing_id, 
    	COUNT(DISTINCT(reviewer_id))    
    FROM reviews    
    GROUP BY listing_id    
    ) AS r  
ON c.listing_id = r.listing_id
ORDER BY listing_id
```
__________
Используйте таблицу checks и разделите всех покупателей на сегменты:  
NB! Правые границы берутся не включительно, например, чек в 10 рублей будет относиться к сегменту С  
- А — средний чек покупателя менее 5 ₽  
- B — средний чек покупателя от 5-10 ₽  
- C — средний чек покупателя от 10-20 ₽  
- D — средний чек покупателя от 20 ₽  

Отсортируйте результирующую таблицу по возрастанию UserID 

```sql
SELECT
    UserID,    
    CASE   
        WHEN avg(Rub) < 5 THEN 'A'        
        WHEN avg(Rub) >= 5 AND avg(Rub) < 10 THEN 'B'        
        WHEN avg(Rub) >= 10 AND avg(Rub) < 20 THEN 'C'        
        ELSE 'D'        
    END AS segment    
FROM checks
GROUP BY UserID
ORDER BY UserID
```
__________
*Используйте предыдущий запрос как подзапрос и посчитайте, сколько клиентов приходится на каждый сегмент и сколько доходов он приносит. Отсортируйте результат по убыванию суммы доходов на сегмент и в качестве ответа укажите наибольшую сумму.*

```sql
SELECT
    segment,    
    COUNT(DISTINCT UserID),    
    sum(Rub)    
FROM
    checks   
JOIN
    (SELECT    
        UserID,    
        CASE    
            WHEN avg(Rub) < 5 THEN 'A'        
            WHEN avg(Rub) >= 5 AND avg(Rub) < 10 THEN 'B'        
            WHEN avg(Rub) >= 10 AND avg(Rub) < 20 THEN 'C'       
            ELSE 'D'       
        END AS segment    
    FROM checks    
    GROUP BY UserID    
    ) as user_segm    
USING (UserID)
GROUP BY segment
ORDER BY sum(Rub) DESC
```
__________
Вернемся к таблице AirBnb. Предположим, что в выборе жилья нас интересует только два параметра: наличие кухни (kitchen) и гибкой системы отмены (flexible), причем первый в приоритете.  
Создайте с помощью оператора CASE колонку с обозначением группы, в которую попадает жилье из таблицы listings:  
- 'good', если в удобствах (amenities) присутствует кухня и система отмены (cancellation_policy) гибкая  
- 'ok', если в удобствах есть кухня, но система отмены не гибкая  
- 'not ok' во всех остальных случаях  

Результат отсортируйте по новой колонке по возрастанию, установите ограничение в 5 строк, в качестве ответа укажите host_id первой строки.
Обратите внимание, что cancellation_policy - это отдельная колонка, по ней необходимо смотреть систему отмены*

```sql
SELECT 
    host_id,    
    CASE
        WHEN multiSearchAnyCaseInsensitive(amenities, ['kitchen'])!=0 AND cancellation_policy = 'flexible' THEN 'good'   
        WHEN multiSearchAnyCaseInsensitive(amenities, ['kitchen'])!=0 AND cancellation_policy != 'flexible' THEN 'ok'    
    ELSE 'not ok'
    END as qual_group
FROM listings    
ORDER BY qual_group
```
__________

