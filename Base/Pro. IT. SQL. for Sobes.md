Типичные вопросы на СОБЕСЕДОВАНИИ по SQL / Примеры задач и их решения
[[https://www.youtube.com/watch?v=GI2D3MAZBe0]]

id	name	class_item
1	Arpha	Arpha
2	accordeon	NULL
3	Drum	B
4	Royal	NULL
5	Tube	A
6	Piano	C

SELECT ID from table WHERE CLASS_ITEM <> 'A'

NULL is not in report too 
NULL only IS NOT or IS
______

Вопросы на собеседовании по SQL и ответы на них. Илья Хохлов. Часть 2
[[https://www.youtube.com/watch?v=dNNBKriXYdI]]

UNION vs UNION ALL : only uniq - all for 2 tables union
про alias в where - не сработает, но можно в orederby...
where vs having: having used in result report (in groupby...)
join: (inner (пересеч), right (всё из подключаяемой таблицы), left (всё из первой таблицы) , full (всё))
DML cmd: select, insert, update, delete
______

Вопросы по SQL и Базам Данных на интервью

Если нужно соеденить таблицы - ищем одинаковые в обеих таблицах
Если может не быть совпадений в таблицах - LEFT JOIN
INNER JOIN == JOIN
______

SQL. Решаем ТЕСТОВЫЕ ЗАДАНИЯ из AMAZON и FACEBOOK (подробный разбор)
[[https://www.youtube.com/watch?v=S9B43Ffiais]]

Сегодня вместе будем решать задачи по SQL с собеседований топовых технических компаний - Amazon и Facebook. Видео будет интересно как тем, кто учит SQL с нуля, так и тем, кто уже работает в IT и планирует проходить собеседования. 

Напишите, пожалуйста, если видео было полезным и стоит ли снимать ещё такие разборы?

Полезные ссылки:
[[https://www.stratascratch.com/]]
[[https://www.sql-ex.ru/]]

Таймкоды:
00:00 - Где искать задачи с собеседований IT-компаний?
00:53 - Решаем задачу из Amazon
09:36 - Решаем задачу из Facebook
20:44 - Задача со звездочкой из моей работы. Пишите ваши решения в комментарии!

Меня зовут Андрей -  я работаю продуктовым аналитиком в IT стартапе и на этом канале (Noukash) я рассказываю про IT и стартапы. Будут разборы профессий, советы, истории. Подписывайтесь и оставляйте комментарии)

Instagram: [[https://www.instagram.com/noukash/]]
ТГ Чат для общения: https://t.me/noukash_it

______

27 распространённых вопросов по SQL с собеседований и ответы на них
https://tproger.ru/articles/sql-interview-questions/

DELETE vs TRUNCATE: DML (del and can rollback), DDL (rebuild table)
COUNT
DISTINCT - uniq
DATE_ADD(DATE, INTERVAL)
AVG 

case for change table fields:
UPDATE table SET salary =
CASE
WHEN salary = 900 THEN 1000
ELSE 1500
END;

case for rename:
ALTER TABLE first_table RENAME second_table;