- создать новый проект в Google Cloud Platform, Яндекс облако или на любых ВМ, докере  
- далее создать инстанс виртуальной машины с дефолтными параметрами  
- добавить свой ssh ключ в metadata ВМ  
- зайти удаленным ssh (первая сессия), не забывайте про ssh-add  
- поставить PostgreSQL  
- зайти вторым ssh (вторая сессия)  
- запустить везде psql из под пользователя postgres  
- выключить auto commit  
- сделать в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;  
- посмотреть текущий уровень изоляции: show transaction isolation level  
<span style="color:green">Уровень изоляции был read committed</span>
- начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции  
- в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');  
- сделать select * from persons во второй сессии  
- видите ли вы новую запись и если да то почему?  
<span style="color:green">Не вижу</span>
- завершить первую транзакцию - commit;  
- сделать select * from persons во второй сессии  
- видите ли вы новую запись и если да то почему?  
<span style="color:green">Вижу, потому что на уровне изоляции read committed возможно фантомное чтение</span>
- завершите транзакцию во второй сессии  
- начать новые но уже repeatable read транзакции - set transaction isolation level repeatable read;  
- в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');  
- сделать select * from persons во второй сессии  
- видите ли вы новую запись и если да то почему?  
<span style="color:green">Не вижу</span>
- завершить первую транзакцию - commit;  
- сделать select * from persons во второй сессии  
- видите ли вы новую запись и если да то почему?  
<span style="color:green">Не вижу</span>
- завершить вторую транзакцию  
- сделать select * from persons во второй сессии  
- видите ли вы новую запись и если да то почему? 
<span style="color:green">Вижу, до коммита не видел новую строку, т.к. при уровне изоляции repeatable read в PG не допускается фантомное чтение</span>