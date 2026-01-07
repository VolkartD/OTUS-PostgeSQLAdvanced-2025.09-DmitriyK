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
   ![1](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_6.Углубленный%20Анализ%20производительности/screenshots/Screenshot%202026-01-07%20052320.png?raw=true)
5. Оптимизируем PostgreSQL заранее посмотреть рекомендации в pgtune.
   ```
   sudo -su postgres
   psql
   ALTER SYSTEM SET max_connections = '48';
   ALTER SYSTEM SET shared_buffers = '512MB';
   ALTER SYSTEM SET effective_cache_size = '1536MB';
   ALTER SYSTEM SET maintenance_work_mem = '256MB';
   ALTER SYSTEM SET checkpoint_completion_target = '0.9';
   ALTER SYSTEM SET wal_buffers = '16MB';
   ALTER SYSTEM SET default_statistics_target = '500';
   ALTER SYSTEM SET random_page_cost = '1.1';
   ALTER SYSTEM SET effective_io_concurrency = '200';
   ALTER SYSTEM SET work_mem = '2730kB';
   ALTER SYSTEM SET huge_pages = 'off';
   ALTER SYSTEM SET min_wal_size = '4GB';
   ALTER SYSTEM SET max_wal_size = '16GB';
   \q
   exit
   sudo systemctl restart postgresql
   ```
6. Снова тестируем pgbench.
   `pgbench -c 30 -j 2 -T 60 test`
7.Для эксперимента меняем настройки хоста.
![2](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_6.Углубленный%20Анализ%20производительности/screenshots/Screenshot%202026-01-07%20053744.png?raw=true)
8. Снова тестируем pgbench.
    `pgbench -c 30 -j 2 -T 60 test`
9. Опять смотрим Pgtune.
    ![3](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_6.Углубленный%20Анализ%20производительности/screenshots/Screenshot%202026-01-07%20053947.png?raw=true)
10. Оптимизируем PostgreSQL по новым рекомендациям.
    ```
    sudo -su postgres
    psql
    ALTER SYSTEM SET max_connections = '50';
    ALTER SYSTEM SET shared_buffers = '2GB';
    ALTER SYSTEM SET effective_cache_size = '6GB';
    ALTER SYSTEM SET maintenance_work_mem = '1GB';
    ALTER SYSTEM SET checkpoint_completion_target = '0.9';
    ALTER SYSTEM SET wal_buffers = '16MB';
    ALTER SYSTEM SET default_statistics_target = '500';
    ALTER SYSTEM SET random_page_cost = '1.1';
    ALTER SYSTEM SET effective_io_concurrency = '200';
    ALTER SYSTEM SET work_mem = '10485kB';
    ALTER SYSTEM SET huge_pages = 'off';
    ALTER SYSTEM SET min_wal_size = '4GB';
    ALTER SYSTEM SET max_wal_size = '16GB';
    ALTER SYSTEM SET max_worker_processes = '4';
    ALTER SYSTEM SET max_parallel_workers_per_gather = '2';
    ALTER SYSTEM SET max_parallel_workers = '4';
    ALTER SYSTEM SET max_parallel_maintenance_workers = '2';
    \q
    exit
    sudo systemctl restart postgresql
    ```
11. Проводит тест pgbench.
    `pgbench -c 30 -j 2 -T 60 test`
    ![4](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_6.Углубленный%20Анализ%20производительности/screenshots/Screenshot%202026-01-07%20060122.png?raw=true)
    
