# Оконные функции

Данные по продажам авокадо 🥑  
**_Имеется таблица:_**   

- **_date_** – дата
- **_average_price_** – средняя цена одного авокадо
- **_total_volume_** –  количество проданных авокадо
- **_plu4046_** – количество проданных авокадо PLU* 4046
- **_plu4225_** – количество проданных авокадо PLU 4225
- **_plu4770_** – количество проданных авокадо PLU 4770
- **_total_bags_** – всего упаковок
- **_small_bags_** – маленькие упаковки
- **_large_bags_** – большие
- **_xlarge_bags_** – очень большие
- **_type_** – обычный или органический
- **_year_** – год
- **_region_** – город или регион (TotalUS – сразу по США)

В таблице находятся данные не за каждый день, а за конец каждой недели. Для каждой даты есть несколько наблюдений, отличающихся по типу авокадо и региону продажи. 
  ____________
*Давайте посмотрим на продажи авокадо в двух городах (NewYork, LosAngeles) и узнаем, 
сколько авокадо типа organic было продано в целом к концу каждой недели (накопительная сумма продаж), 
начиная с начала периода наблюдений (04/01/15).
Значения внутри окна сортируйте по дате, а саму таблицу отсортируйте по убыванию региона (сначала NY, потом LA) и по возрастанию даты.
В качестве ответа укажите объем продаж в Нью Йорке на 01/03/15 (без запятых).*
  
```sql
SELECT *
FROM 
    (
    SELECT 
        region,
        date,
        total_volume,
        sum(total_volume) OVER (PARTITION BY region
                                ORDER BY date
                                ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) volume
    FROM avocado
    WHERE region in ('NewYork', 'LosAngeles') 
                AND type = 'organic'
                AND date >= '01/04/15'::date

    ORDER BY region DESC, date
   ) as t
WHERE date = '03/01/15'::date
```
__________
*Теперь добавьте разбивку по каждому году (year). Таким образом, в конце февраля 2016 года объем составят уже не продажи за 2015 и январь-февраль 2016, а только за январь-февраль 2016.
Когда объемы продаж органических авокадо в Нью-Йорке превысили объемы продаж в Лос-Анджелесе?*

```sql
SELECT
    region,    
    date,   
    total_volume,    
    SUM(total_volume) OVER w AS volume    
FROM
    avocado  
WHERE region in ('NewYork', 'LosAngeles')
      AND type = 'organic'
WINDOW w AS
    (    
    PARTITION BY region , year  
    ORDER BY date ASC   
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW   
    )
ORDER BY
    region DESC,    
    date ASC
```
__________
*Посмотрим, когда объемы продаж обычных (conventional) авокадо резко падали по сравнению с предыдущей неделей. Возьмите данные по США в целом, посчитайте разницу между объемом продаж в неделю x (total_volume) и количеством проданных авокадо в течение предыдущей недели. Значения запишите в новый столбец week_diff.*

```sql
SELECT 
        date,
        total_volume,
        region,
        type,
        total_volume - lag(total_volume, 1) OVER (PARTITION BY region
                                                    ORDER BY date
                                                  ) week_diff
          
FROM avocado
WHERE type = 'conventional' AND region = 'TotalUS'
```
__________
*Посмотрим более подробно на объемы продаж авокадо в Нью-Йорке (NewYork) в 2018 году. Создайте колонку с разницей объемов продаж за неделю и за неделю до этого для каждого типа авокадо. Найдите день, когда продажи авокадо типа organic увеличились по сравнению с предыдущей неделей, а conventional – наоборот упали. Если таких дней несколько, то укажите их через запятую с пробелом, формат – 31/12/2020.*

```sql
SELECT 
        date,
        year,
        total_volume,
        type,
        total_volume - lag(total_volume, 1) OVER (PARTITION BY type
                                                    ORDER BY date
                                                  ) week_diff
          
FROM avocado
WHERE region = 'NewYork' and year = 2018
ORDER BY type
```
__________
*Теперь посчитайте скользящее среднее цены авокадо (average_price) в Нью-Йорке с разбивкой по типу авокадо. В качестве окна используйте текущую неделю и предыдущие две (обратите внимание, что в данной таблице в строках содержатся данные за неделю, а не за один день). Например 04/01/15, 11/01/15 и 18/01/15 для подсчета значения для 18/01/15.
В качестве ответа укажите полученное значение для обычных (conventional) авокадо за 17/04/16.*

```sql
SELECT 
        date,
        average_price ,
        region,
        type,
        AVG(average_price) OVER (PARTITION BY type
                            ORDER BY date
                            ROWS BETWEEN 2 PRECEDING AND CURRENT ROW ) rolling_price 
          
FROM avocado
WHERE region = 'NewYork'  
--and date = '04/17/16'::date
ORDER BY date
```
__________
**

```sql

```
__________
