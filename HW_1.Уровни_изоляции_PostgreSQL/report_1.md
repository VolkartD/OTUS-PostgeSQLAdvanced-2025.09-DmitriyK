# Работа с уровнями изоляции транзакции в PostgreSQL
1. Создание проекта.
	- создание виртуальной машины.
2. добавление ssh-ключа. 
	- создание ssh-ключей.
	- подключение к виртуальной машине с помощью ssh-ключа.
3. Установка PostgreSQL.
	- проверка доступных версий: 
		`apt-cache search PostgreSQL`
	- установка PostgreSQL:
		`sudo apt-get install PostgreSQL`
4. Подключение к PoststgreSQL.
	- подключение через ssh-ключи к ВМ.
	- подключение к PostgreSQL: 
		`sudo -su postgres pqsl`
	- аналогично запускаем вторую сессию.
5. Работа с транзакциями.
	- проверяем AUTOCOMMIT:
		`\echo :AUTOCOMMIT`
	- выключаем:
		`\set AUTOCOMMIT off`
	- в первой сессии создаем таблицу с данными согласно инструкции.
```
create table shipments(id serial, product_name text, quantity int, destination text);
insert into shipments(product_name, quantity, destination) values('bananas', 1000, 'Europe');
insert into shipments(product_name, quantity, destination) values('coffee', 500, 'USA');
commit;
```
6. Изучение уровней изоляции.
	- проверяем уровень изоляции:
		`show transaction isolation level;`
	- выполняем `select * from shipments;` во второй сессии, получаем ошибку так как в первой сессии транзакция не закончилась, и таблица заблокирована.
	- завершаем транзакцию в первой сессии, повторяем запрос во второй, так как в первой сессии транзакция успешно завершилась и данные были добавлены, теперь во второй сессии они таблица доступна и они видны.
7. Эксперименты с уровнем изоляции repeatable read.
	- устанавливаем уровень изоляции в сессиях:
		`set transaction isolation level repeatable read;`
	- в первой сессии начинаем транзакцию:
		`insert into shipments(product_name, quantity, destination) values('bananas', 2000, 'Africa');`
	- во второй сессии смотрим таблицу:
		`select * from shipments;`
	- новая запись не видна, так как транзакция в первой сессии не была завершена, но ошибки о блокировки таблицы тоже нет, так как уровень изоляции repeatable read обеспечивает что повторное чтения  одних данных вернет тот же результат,что предотвращает аномалии "грязного" чтения, неповторяющегося чтения и фантомного чтения. После завершения транзакции комитом в первой сессии, и повторном селекте во второй сессии, новые данные видны.
