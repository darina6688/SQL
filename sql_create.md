#  ПОДЗАПРОСЫ, ПРЕДСТАВЛЕНИЯ, СОЗДАНИЕ ТАБЛИЦ
## Данные для задач  
Напишите запрос для создания таблицы со следующими параметрами, также подобрав подходящий тип данных.
**Название таблицы:** reviews
**База данных:** test
**Столбцы:**
- **_listing_id_** – идентификатор объявления, может быть только положительным и целым числом, 32-битный тип данных
- **_id_** – идентификатор хозяина, может быть только положительным и целым числом, 32-битный тип данных
- **_created_at_** – дата со временем (2020-01-01 00:00:00), часовой пояс – 'Europe/Moscow'
- **_reviewer_id_** – идентификатор ревьюера, может быть только положительным и целым числом, 32-битный тип данных
- **_reviewer_name_** – имя того, кто оставил отзыв
- **_comments_** - текст отзыва
**Движок:** MergeTree
**Сортировка:** listing_id, id
  ____________ 
```sql
CREATE TABLE test.reviews
    (listing_id UInt32,    
    id UInt32,    
    created_at DateTime('Europe/Moscow'),   
    reviewer_id UInt32,    
    reviewer_name String,    
    comments String)   
ENGINE=MergeTree
ORDER BY(listing_id, id)
```
__________
*К вам пришел коллега с новостями: оказывается, в поле created_at будет записываться только дата, без времени, поэтому нужно изменить тип данных!
Напишите запрос, который модифицирует тип данных, и введите его в поле ниже без кавычек и лишних пробелов. 
Не забудьте указать базу данных test перед названием таблицы!*
```sql
ALTER TABLE test.reviews MODIFY COLUMN created_at date
```
______
*Предположим, ваш коллега вставил данные, но что-то перепутал. Часть строк с комментариями осталась совершенно пустой! 
Напишите запрос, который удалит из таблицы test.reviews те строки, где в столбце comments встречаются пустые значения ('').*
```sql
ALTER TABLE test.reviews DELETE WHERE comments=''
```
_________
*С помощью какого запроса можно создать обычное представление над таблицей test.reviews, которое будет содержать все записи из test.reviews, 
сгруппированные по reviewer_id с подсчитанным количеством отзывов (id) на каждого пользователя?*
```sql
CREATE VIEW test.reviews_number AS (SELECT reviewer_id, COUNT(id) reviews_count
                                    FROM test.reviews
                                    GROUP BY reviewer_id)
```
__________
*С помощью какого запроса можно создать новый столбец reviewer_score в таблице reviews после столбца reviewer_name?*
```sql
ALTER TABLE test.reviews ADD COLUMN reviewer_score UInt8 AFTER reviewer_name
```
_________
*Напишите запрос для добавления в таблицу test.reviews колонки price после колонки comments, 
которая может быть числом с плавающей точкой, 32-битный тип данных.*
```sql
ALTER TABLE test.reviews ADD COLUMN price Float32 AFTER comments
```
_________
*Напишите запрос, который удвоит price для всех строк с датой (created_at) после 2019-01-01.
Обратите внимание, что для сравнения с датой в формате '2019-01-01' можно не преобразовывать данные в колонке created_at.*
```sql
ALTER TABLE test.reviews UPDATE price=price*2 WHERE created_at>'2019-01-01'
```
