#Углубленный анализ производительности с использованием pgbench
1. Развернуть виртуальную машину.
   Стандартные установки 2 Гб ОЗУ, 1 проц, ссд.
2. Установить PostgreSQL.
   ```
   sudo apt-get update -y && sudo apt-get upgrade
   sudo apt install postgresql postgresql-contrib nano mc ssh -y
   ```
3. Создаем тестовую БД.
   ```
   sudo -su postgres
   psql
   CREATE DATABASE test;
   \q
   ```
4. Теструем производительность pgbench.
   ```
   pgbench -i -s 10 test
   pgbench -c 30 -j 2 -T 60 test
   ```
   ![1]()
