# Построение отказоустойчивого кластера патрони: PostgreSQL, Patroni, распределенное хранилище (ETCD), балансировщик (HaProxy), мониторинг (Prometheus), резервирование.

1. Разворачиваем ВМ: 3 Patroni, 3 etcd, 2 Haproxy, 1 Prometheus.
2. Обновляем и устанавливаем необходимые пакеты для дальнейшей работы:
   ```
   sudo apt-get update -y && sudo apt-get upgrade -y
   sudo apt install -y nano mc ssh network-manager
   ```
4. Устанавливаем и настраиваем кластер ETCD.
   
	3.1. Установка ETCD:
   
	`sudo apt-get install etcd-server -y && sudo apt-get install etcd-client -y`

	3.2. Проверяем версию:
   
	`etcd --version`

	3.3. Настраиваем конфиг запускаем ETCD:

   ```
   sudo nano /etc/default/etcd
   systemctl restart etcd
   ```
   [ETCD_conf](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/cfg/etcd.txt "Файл с конфигурацией 3-х хостов")
   
	3.4. Проверяем работу ECTD:
   
    ```
	etcdctl --cluster=true endpoint health
	ETCDCTL_ENDPOINTS=192.168.10.154:2379,192.168.10.155:2379,192.168.10.156:2379 etcdctl endpoint status -w table
    ```
    ![1](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/screenshots/etcd.png)
   
5. Patroni.
   
	4.1. Установка и настройка PostgreSQL и зависимости:
   
    `sudo apt install -y postgresql postgresql-contrib python3-pip python3-venv`

	4.2. Создаем каталог для кластера, выдаем права:
   
   ```
	sudo mkdir /var/lib/postgresql/16/data
	sudo mkdir /var/lib/postgresql/16/data/pg-cluster
	sudo chmod 750 /var/lib/postgresql/16/data/pg-cluster
   ```
   
	4.3. Установка patroni:
   
	`sudo apt install -y patroni`

	4.4. Отключаем PostgreSQL:
   
    ```
	sudo systemctl stop postgresql
	sudo systemctl disable postgresql
    ```
	
    4.5. Настраиваем конфиг patroni:
	
    `sudo nano /etc/patroni/config.yml`
   
   [Patroni_conf_1](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/cfg/patroni1.txt "текст config.yml")
   
   [Patroni_conf_2](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/cfg/patroni2.txt "текст config.yml")
   
   [Patroni_conf_3](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/cfg/patroni3.txt "текст config.yml")
   
  
	
    4.6. Запускаем patroni:
 
   ```
	sudo systemctl enable patroni
	sudo systemctl start patroni
   ```
   _При первом запуске кластера получил ошибку, неверно были указаны права на каталог для кластера_
![2](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/screenshots/Ошибка%20при%20запуске%20patroni.png)
   _Далее права поправил, назначил владельца пользователя Postgres_
   
    4.7. Проверяем:
  
    `patronictl -c /etc/patroni/config.yml list`

   ![3](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/screenshots/проверка%20patroni.png)


	  4.8. Скачиваем и разворачиваем тестовую БД:
   
   _Для теста использовал тестовую БД Авиаполетов в открытом досутпе от PostgresPRO_
  
   ```
	wget "https://edu.postgrespro.ru/demo-20250901-2y.sql.gz"
	gunzip -c demo-20250901-2y.sql.gz | psql -U postgres
   ```

	4.9. Проверяем размер БД:
  
   ```
	sudo su postgres
	psql
	SELECT pg_size_pretty(pg_database_size('demo'));
	 ```

7. Установка и настройка HaProxy.

	5.1. Устанавливаем haproxy:

	`sudo apt install -y haproxy`

	5.2. Настраиваем конфиг и запускаем:

   ```
    sudo nano /etc/haproxy/haproxy.cfg
	sudo systemctl restart haproxy
   ```
   
   [HaProxy_conf](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/cfg/HaProxy.txt "конфигурация HaProxy")
   

	5.3. Проверяем работу haproxy:

     с браузера подключаемся к хосту haproxy	http://192.168.10.150:7000/

     через Dbeaver подключаемся по адресу haproxy к postgresql

   ![4](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/screenshots/HaProxy%20проверка.png)

9. Установка и настойка Prometheus.
   
	6.1. Скачиваем пакет:

	`wget https://github.com/prometheus/prometheus/releases/download/v2.45.3/prometheus-2.45.3.linux-amd64.tar.gz`

	6.2. Создаем группу, пользователя, каталоги и настраиваем доступы:

    ```
	sudo groupadd --system prometheus
	sudo useradd -s /sbin/nologin --system -g prometheus prometheus
	sudo mkdir /etc/prometheus
	sudo mkdir /var/lib/prometheus
	tar -zxvf prometheus-2.45.3.linux-amd64.tar.gz
	cd prometheus-2.45.3.linux-amd64/
	sudo mv prometheus /usr/local/bin
	sudo mv promtool /usr/local/bin
	sudo mv consoles /etc/prometheus
	sudo mv console_libraries /etc/prometheus
	sudo mv prometheus.yml /etc/prometheus
	sudo chown prometheus:prometheus /usr/local/bin/prometheus
	sudo chown prometheus:prometheus /usr/local/bin/promtool
	sudo chown -R prometheus:prometheus /etc/prometheus/consoles
	sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
	sudo chown -R prometheus:prometheus /etc/prometheus
	sudo chown -R prometheus:prometheus /var/lib/prometheus
    ```

	6.3. Создаем службу, запускаем, проверяем:

    ```
	sudo nano /etc/systemd/system/prometheus.service
	sudo systemctl daemon-reload
	sudo systemctl start prometheus
	sudo systemctl enable prometheus
	sudo systemctl status prometheus
    ```
    [Prometheus_service](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/cfg/prometheus.txt)

	6.4.  Установка и настройка postgres_exporter:

    ```
	sudo useradd -rs /bin/false postgres_exporter
	wget https://github.com/prometheus-community/postgres_exporter/releases/download/v0.19.1/postgres_exporter-0.19.1.linux-amd64.tar.gz
	tar xvf postgres_exporter-0.19.1.linux-amd64.tar.gz
	sudo mv postgres_exporter-*/postgres_exporter /usr/local/bin/
	sudo -u postgres psql
	CREATE USER exporter WITH PASSWORD 'exporterpass';
	CONNECT ON DATABASE postgres TO exporter;
	GRANT pg_monitor TO exporter;
	sudo nano /etc/postgres_exporter.env
	sudo nano /etc/systemd/system/postgres_exporter.service
	sudo systemctl daemon-reload
	sudo systemctl enable postgres_exporter
	sudo systemctl start postgres_exporter
	curl http://192.168.10.151:9187/metrics
    ```

    [postgres_exporter_service](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/cfg/exporter.txt)

    6.5. Добавляем джобы в Prometheus:

     `sudo nano /etc/prometheus/prometheus.yml`
   
   [Prometheus_config](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/cfg/prometheus_cfg.txt)

   6.6. В браузере открываем Prometheus и смотрим метрики для проверки работы например:
   ```
   pg_database_size_bytes
   pg_roles_connection_limit
   ```

   ![5](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/screenshots/prometheus%20проверка.png)
   ![6](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/screenshots/проверка%20метрик%20прометеус1.png)
   ![7](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/screenshots/проверка%20метрик%20прометеус2.png)

   6.7. _Для мониторинга также хотел установиь и настроить дашборды в Grafana, но с толкнулся с проблемой что оф реcурсы не доступны в России на текущий момент._
![9](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/screenshots/Screenshot%202026-03-02%20170901.png)
![10](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/screenshots/Screenshot%202026-03-08%20113305.png)
   
7. Настраиваем резервирование БД.
   
   _Для хранения бекапов, и их создания выбрал сервер HaProxy, в идеале конечно иметь отдельный сервер, но не имсею возможности._
   
   7.1. Устанавливаем Cron если не установлен, устанавливаем утилиты клиента PostgreSQL:
   
   `sudo apt install -y postgresql-client`
   
   7.2. Создаем каталог для хранения бекапов:
   ```
   sudo mkdir /backups
   sudo chmod 750 /backups
   ```
   
   7.3. Создаем скрипт который будет бекапить БД:
   
   `sudo nano /backups/backup_bd.sh`
   
   [backup_sh](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/cfg/backup_db.txt "текст скрипта резервного копирования")
   
   7.4. Создаем задание в Cron с расписанием выполнения бекапа.
   
   7.5. Проверяем.
   ![8](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/screenshots/расписание%20и%20бекап.png)

8. keepalived.
   
   8.1. Устанавливаем на хосты haproxy и haproxy-2 keepalived:
   
   	`sudo apt install -y keepalived`

   8.2. Настраиваем конфиг, запускаем:

   ```
   sudo nano /etc/keepalived/keepalived.conf
   systemctl enable --now haproxy
   systemctl enable --now keepalived
   ```
[keepalived_cfg](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/cfg/keepalived_cfg.txt "Конфиг 2-ъ хостов")
   8.3. Проверяем, отключая мастер хост с VIP.
   мастер haproxy
![11](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/screenshots/Screenshot%202026-03-09%20133806.png)
   отключаем haproxy мастером становится haroxy-2
   ![12](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/FinalProject/screenshots/Screenshot%202026-03-09%20134258.png)
