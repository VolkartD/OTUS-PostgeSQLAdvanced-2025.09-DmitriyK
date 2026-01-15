#
1. Выбираем два облака для сравнения. YandexCloud и SberCloud.
2. Регистрируемся.
3. Создаем managed PostgreSQL YandexCloud.
   3.1. Добавлем новое облако для проекта в организации.
   ![1](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_9.Выбор%20облака/screenshots/Screenshot%202026-01-15%20070154.png?raw=true)
   
   3.2. Идем во "все сервисы" находим Managed Service for PostgreSQL, создаем кластер.
   ![2](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_9.Выбор%20облака/screenshots/Screenshot%202026-01-15%20070311.png?raw=true)
   3.3. Настраиваем необходимые параметры кластера.
   ![3](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_9.Выбор%20облака/screenshots/Screenshot%202026-01-15%20070414.png?raw=true)
   3.4. Фиксируем стоимость.
   
   ![4](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_9.Выбор%20облака/screenshots/Screenshot%202026-01-15%20070810.png?raw=true)
   
   3.5. Создаем.
5. Создаем managed PostgreSQL в SberCloud.
   
   4.1. В Сервисах выбираем managed PostgreSQL.
   
   ![5](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_9.Выбор%20облака/screenshots/Screenshot%202026-01-15%20070927.png?raw=true)
   
   4.2. Нажимаем создать кластер.
   
   4.3. Настраиваем будущий кластер аналогично кластеру в яндексе для сравнения.
   
   ![6](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_9.Выбор%20облака/screenshots/Screenshot%202026-01-15%20071058.png?raw=true)
   
   ![7](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_9.Выбор%20облака/screenshots/Screenshot%202026-01-15%20071300.png?raw=true)
   
   ![8](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_9.Выбор%20облака/screenshots/Screenshot%202026-01-15%20071449.png?raw=true)
   
   4.4. Фиксируем стоимость.
   
   ![9](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_9.Выбор%20облака/screenshots/Screenshot%202026-01-15%20111728.png?raw=true)
   
   4.5. Создаем.
7. Проверяем подключение к кластерам с локальной машины.
   
   5.1. В Yandex Cloud.
   
     5.1.1. Устанавливаем PostgreSQL для Windows версии 17. Command Line Tools.
   
     5.1.2. Устанавливаем сертификат.
   
     `mkdir $HOME\.postgresql; curl.exe -o $HOME\.postgresql\root.crt https://storage.yandexcloud.net/cloud-certs/CA.pem`
   
     5.1.3. Устанавливаем переменные окружения:
   
     `$Env:PGSSLMODE="verify-full"; $Env:PGTARGETSESSIONATTRS="read-write"`
   
     5.1.4. Подключаемся к БД
   
     ```
     & "C:\Program Files\PostgreSQL\17\bin\psql.exe" `
 			--host=rc1b-sbh5m9lf9funulf0.mdb.yandexcloud.net `
			--port=6432 `
			--username=otus_user `
			otus_db
     ```
     
     ![10](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_9.Выбор%20облака/screenshots/Screenshot%202026-01-15%20072402.png?raw=true)
   
     5.1.5. Вводим пароль пользователя.
   
     5.1.6. Выполняем создание тестовой таблицы.
   
     ```
     CREATE TABLE test_table (
			id SERIAL PRIMARY KEY,
			name VARCHAR(50),
			value INTEGER,
			created_at TIMESTAMP DEFAULT NOW()
			);

			INSERT INTO test_table (name, value) VALUES
			('test1', 100),
			('test2', 200),
			('test3', 300);
     ```
     
     5.1.7. Проверяем.
   
     `SELECT * FROM test_table;`
   
     ![11](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_9.Выбор%20облака/screenshots/Screenshot%202026-01-15%20072933.png?raw=true)
   
    5.2. В SberCloud.
   
     5.2.1. Подключение доступно только с внутренней ВМ,
   
     ![12](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_9.Выбор%20облака/screenshots/Screenshot%202026-01-15%20074026.png?raw=true)
   
   поэтому создаем ВМ с публичным адресом, генерируем ssh ключ.
   
     ![13](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_9.Выбор%20облака/screenshots/Screenshot%202026-01-15%20074137.png?raw=true)
   
     ![14](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_9.Выбор%20облака/screenshots/Screenshot%202026-01-15%20074243.png?raw=true)
   
     ![15](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_9.Выбор%20облака/screenshots/Screenshot%202026-01-15%20074941.png?raw=true)
   
     5.2.2. Подключаемся по ssh к ВМ.
   
     ![16](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_9.Выбор%20облака/screenshots/Screenshot%202026-01-15%20075553.png?raw=true)
   
     5.2.3. устанавливаем консольный клиент postgresql.
   
     ```
      sudo apt-get update
      sudo apt-get install postgresql
     ```
     
     5.2.4. Подключаемся к БД.
   
     `psql -h 10.0.0.6 -p 5432 -U dbadmin -d otus_db_test`
   
     5.2.5. Выполняем создание тестовой таблицы.
   
     ```
     CREATE TABLE test_table (
			id SERIAL PRIMARY KEY,
			name VARCHAR(50),
			value INTEGER,
			created_at TIMESTAMP DEFAULT NOW()
			);

			INSERT INTO test_table (name, value) VALUES
			('test1', 100),
			('test2', 200),
			('test3', 300);
     ```
     
     5.2.6. Проверяем.
   
     `SELECT * FROM test_table;`
     ![17](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_9.Выбор%20облака/screenshots/Screenshot%202026-01-15%20104241.png?raw=true)

10. Удаляем кластеры.
11. Вывод:
    
    В сравнении более комфортным показался YandexCloud, проще настроить подключение, проще развертывание. 
  
    Так же создание кластера в YandexCloud происходит быстрей, в данной работе создание создание кластера в YandexCloud заняло около 5 минут, тогда как аналогичный по характеристикам в SberCloud около 20 минут, для теста пересоздал кластер несколько раз. 
  
    И в SberCloud нет возможности сделать публичный доступ к БД, это безопасней, но менее удобно, в этом вопросе стоит исходить из требований к проекту.
  	К плюсам SberCloud могу отнести более гибкую настройку кластера и ВМ, а так-же, ИМХО _интерфейс в SberCloud более понятный и дружественный_.
    
    В ценовой категории кластер в SberCloud дешевле, но развернуть ВМ с публичным адресом уже сравнивает ценовые показатели.
  
  *P.S. Думаю если разница в цене и будет критичной, то на высоконагруженых системах, на тестовых стендах с минимальными параметрами сказать что выгодней именно в цене сложно.*
