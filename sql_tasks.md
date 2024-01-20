## Таблицы для задач  

  **installs** — содержит данные об установках приложения по дням.  
- **_DeviceID_** — идентификатор устройства, на которое было установлено приложение;  
- **_InstallationDate_** — дата установки приложения;  
- **_InstallCost_** — цена установки приложения в рублях;  
- **_Platform_** — платформа, на которой было установлено приложение (iOS/ Android);  
- **_Source_** — источник установки приложения (магазин приложения/ рекламная система/ переход с сайта).  

**events** — содержит данные о том, как активно пользователи просматривают товары в приложении по дням.  
- **_DeviceID_** — идентификатор устройства, на котором используется приложение;  
- **_AppPlatform_** — платформа, на которой используется приложение (iOS/ Android);  
- **_EventDate_** — дата, за которую собрана статистика;  
- **_events_** — количество просмотров всех товаров за этот день у этого DeviceID.  

 **checks** — содержит данные о покупках пользователей в приложении по дням  
- **_UserID_** — идентификатор пользователя;  
- **_Rub_** — суммарный чек пользователя на дату;  
- **_BuyDate_** — дата, за которую собрана статистика.  

**devices** — содержит информацию как об идентификаторах пользователей, так и их устройствах  
- **_DeviceID_** — идентификатор устройства;  
- **_UserID_** — идентификатор пользователя  

Особенность приложения заключается в том, что для просмотра товаров не обязательно авторизовываться. До момента авторизации про пользователя известен только его DeviceID — идентификатор устройства. При этом для совершения покупки логин обязателен. На моменте авторизации пользователю присваивается UserID, и тогда мы уже знаем два его идентификатора: DeviceID (устройство) и UserID (логин). Так как на этапах установки приложения и просмотра каталога пользователь еще может быть не авторизован, там мы сохраняем только DeviceID. Но так как покупки нельзя совершить без авторизации, то покупки сохраняются только с UserID. Для того чтобы просмотры и установки можно было объединить с покупками, нам нужна таблица соответствия DeviceID к UserID, то есть таблица devices  
__________
*У пользователя может быть два идентификатора – UserID и DeviceID. В таблице checks есть только UserID, в остальных – только DeviceID. Во вспомогательной таблице devices есть и UserID, и DeviceID.  Давайте с помощью JOIN дополним таблицу events (left) данными о UserID пользователей из таблицы devices (right). Для некоторых DeviceID не будет пары UserID из таблицы devices – подумайте, какой вид JOIN подойдет, чтобы не потерять те строки, где DeviceID есть в events, но нет в devices.  Укажите UserID из первой строки результирующей таблицы, используя сортировку по убыванию по полю DeviceID.*

![image](https://github.com/darina6688/SQL/assets/152012358/64106743-be5b-4a5a-b586-13b72b9ad375)
___________________
*пользователи пришедшие из какого источника совершили наибольшее число покупок. В качестве ответа выберите название Source, юзеры которого совершили больше всего покупок.  Для этого используйте UserID, DeviceID и Source из соответствующих таблиц. Считать уникальные значения здесь не нужно. Также заметьте, что покупки со стоимостью 0 рублей всё ещё считаются покупками.*

 ![image](https://github.com/darina6688/SQL/assets/152012358/21888597-0bdc-4b43-b8f9-9c5104d878cb)
____________________    
*Теперь выясним, сколько всего уникальных юзеров что-то купили в нашем приложении. Объедините нужные таблицы, посчитайте число уникальных UserID для каждого источника (Source), и в качестве ответа укажите число пользователей, пришедших из Source_7*

![image](https://github.com/darina6688/SQL/assets/152012358/32b0c855-f661-4406-86f7-51f3810f5520)
_____________________
SELECT 
    Source,
    SUM(Rub), 
    min(Rub),
    max(Rub),
    avg(Rub)   
FROM 
    checks c   
    JOIN devices d ON c.UserID = d.UserID  
    JOIN installs i ON d.DeviceID = i.DeviceID    
GROUP BY Source
ORDER BY Source
LIMIT 50 
_______________________
*task*

SELECT  count(distinct UserID)
FROM  default.checks c
      JOIN default.devices d ON c.UserID = d.UserID 
      JOIN test.installs i ON d.DeviceID = i.DeviceID
WHERE Source = 'Source7' 
GROUP BY Source
LIMIT 10

_________________________
*task*

SELECT 
    DeviceID    
FROM checks c   
     JOIN devices d ON c.UserID = d.UserID   
WHERE toStartOfMonth(CAST(BuyDate as date)) = '2019-10-01' 
ORDER BY DeviceID 
LIMIT 10

_______________________
*task*

SELECT 
    AppPlatform,
    Source,
    avg(events) as avg_events
FROM events    
     JOIN installs using (DeviceID)
GROUP BY AppPlatform, Source
ORDER BY avg_events DESC 
LIMIT 10 

_____________________________
*task*

SELECT Platform,
     count(Distinct DeviceID)
FROM installs
     LEFT SEMI JOIN events using (DeviceID)
GROUP BY Platform
LIMIT 10 

_________________________________
*task*

SELECT Platform,
       count(Distinct ev.DeviceID)/count(Distinct i.DeviceID)
FROM installs i 
     LEFT JOIN events ev ON i.DeviceID=ev.DeviceID 
GROUP BY Platform

___________________________
