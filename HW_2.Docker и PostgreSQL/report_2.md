# Работа с Docker и PostgreSQL
1. Создаем и настраиваем виртуальную машину VirtualBox.
2. Установка Docker Engine, проверка статуса.
   ```
   curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh  && rm get-docker.sh
   sudo systemctl status docker
   ```
3. Добавляем пользователя в группу Docker.
   `sudo usermod -aG docker $USER`
4. Создаем каталог /var/lib/postgres.
   `sudo mkdir /var/lib/postgres`
5. Создаем сеть Docker.
   `sudo docker network create db-net`
6. Скачиваем образ PostgreSQL 14.
   ```
   sudo docker pull postgres:14
   sudo docker image inspect postgres:14
   ```
7. Создаем контейнер БД.
   `sudo docker run -d --network db-net --name postgres -e POSTGRES_PASSWORD=postgres -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14`
8. Проверка конфига контейнера.
   `sudo grep -Ev '^$|^#' /var/lib/postgres/pg_hba.conf`
9. Подключаемся к контейнеру.
    `sudo docker exec -it postgres su - postgres "-c psql -h postgres -U postgres -W"`
10. Создаем БД, проверяем.
    ```
    Create Database bananaflow;
    \l
    ```
11. Создаем контейнер клиента.
    `sudo docker run -d --network db-net --name pg-client -e POSTGRES_PASSWORD=postgres postgres:14`
12. Подключаемся к БД в контейнере БД из контейнера клиента.
    ```
    sudo docker exec -it pg-client su - postgres "-c psql -h postgres -U postgres -W"
    \c bananaflow
    ```
13. Создаем таблицу.
    ```
    create table shipments(id serial, product_name text, quantity int, destination text);
    insert into shipments(product_name, quantity, destination) values('bananas', 1000, 'Europe');
    insert into shipments(product_name, quantity, destination) values('bananas', 1500, 'Asia');
    insert into shipments(product_name, quantity, destination) values('bananas', 2000, 'Africa');
    insert into shipments(product_name, quantity, destination) values('coffee', 500, 'USA');
    insert into shipments(product_name, quantity, destination) values('coffee', 700, 'Canada');
    insert into shipments(product_name, quantity, destination) values('coffee', 300, 'Japan');
    insert into shipments(product_name, quantity, destination) values('sugar', 1000, 'Europe');
    insert into shipments(product_name, quantity, destination) values('sugar', 800, 'Asia');
    insert into shipments(product_name, quantity, destination) values('sugar', 600, 'Africa');
    insert into shipments(product_name, quantity, destination) values('sugar', 400, 'USA');
    ```
14. Подключаемся к контейнеру с БД с ПК, использовал DBeaver. ![14](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_2.Docker%20и%20PostgreSQL/screenshots/14.png?raw=true)
15. Удаляем контейнер с БД, создаем его заново.
        ```
        sudo docker rm -f postgres
        sudo docker run -d --network db-net --name postgres -e POSTGRES_PASSWORD=postgres -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14
        ```
16. Проверяем что данные сохранились.
        ```
        sudo docker exec -it pg-client su - postgres "-c psql -h postgres -U postgres -W"
        \c bananaflow
        \dt
        SELECT * FROM shipments;
        ```
![16](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_2.Docker%20и%20PostgreSQL/screenshots/16.png?raw=true)
