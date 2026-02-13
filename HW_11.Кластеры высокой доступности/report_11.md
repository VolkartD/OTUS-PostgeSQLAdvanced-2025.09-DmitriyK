# Кластеры высокой доступности
## Вариант 1. Кластер своими руками. Patroni + Etcd + HAProxy.
1. Создаем и настраиваем ВМ, 3 для partroni, 3 для etcd.
   ```
    sudo apt-get update -y && sudo apt-get upgrade -yml
    sudo apt install -y nano ssh mc
   ```
  
2. Установка настройка кластера etcd.
   
   a. Устанавливаем etcd на три машины.
    ```
    sudo apt-get install etcd-server -y && sudo apt-get install etcd-client -y
    etcd --version
    ```
      b. Настраиваем конфиг etcd.
   
   `sudo nano /etc/default/etcd`
   ```
    ETCD_NAME="etcd01"
    ETCD_DATA_DIR="/var/lib/etcd"
    ETCD_INITIAL_CLUSTER="etcd01=http://192.168.10.141:2380,etcd02=http://192.168.10.124:2380,etcd03=http://192.168.10.140:2380"
    ETCD_INITIAL_CLUSTER_STATE="new"
    ETCD_INITIAL_CLUSTER_TOKEN="pg-cluster"
    ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.10.141:2380"
    ETCD_ADVERTISE_CLIENT_URLS="http://192.168.10.141:2379"
    ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
    ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"


    ETCD_NAME="etcd02"
    ETCD_DATA_DIR="/var/lib/etcd"
    ETCD_INITIAL_CLUSTER="etcd01=http://192.168.10.141:2380,etcd02=http://192.168.10.124:2380,etcd03=http://192.168.10.140:2380"
    ETCD_INITIAL_CLUSTER_STATE="new"
    ETCD_INITIAL_CLUSTER_TOKEN="pg-cluster"
    ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.10.124:2380"
    ETCD_ADVERTISE_CLIENT_URLS="http://192.168.10.124:2379"
    ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
    ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"


    ETCD_NAME="etcd03"
    ETCD_DATA_DIR="/var/lib/etcd"
    ETCD_INITIAL_CLUSTER="etcd01=http://192.168.10.141:2380,etcd02=http://192.168.10.124:2380,etcd03=http://192.168.10.140:2380"
    ETCD_INITIAL_CLUSTER_STATE="new"
    ETCD_INITIAL_CLUSTER_TOKEN="pg-cluster"
    ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.10.140:2380"
    ETCD_ADVERTISE_CLIENT_URLS="http://192.168.10.140:2379"
    ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
    ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
   ```
   `systemctl restart etcd`
   
     c. проверяем кластер
 
 		
    `etcdctl --cluster=true endpoint health`
    
 	
    `ETCDCTL_ENDPOINTS=192.168.10.141:2379,192.168.10.124:2379,192.168.10.140:2379 etcdctl endpoint status -w table`
   
  ![1](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_11.Кластеры%20высокой%20доступности/source/Screenshot%202026-02-13%20115043.png)
    
3. Установка PostgreSQL настройка Patroni, создаем каталог для кластера.
   ```
	sudo apt install -y postgresql postgresql-contrib python3-pip python3-venv 
	sudo apt install -y patroni
	sudo systemctl stop postgresql
	sudo systemctl disable postgresql
	sudo nano /etc/patroni/config.yml
	sudo mkdir /var/lib/postgresql/16/data/cluster
	sudo chmod 750 /var/lib/postgresql/16/data/cluster
	sudo systemctl enable patroni
	sudo systemctl start patroni
	patronictl -c /etc/patroni/config.yml list
   ```
   Пример конфига патрони pg01, остальные конфиги - [pg02](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_11.Кластеры%20высокой%20доступности/source/patroni2.txt), [pg03](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_11.Кластеры%20высокой%20доступности/source/patroni2.txt)
```
   scope: pg-cluster2
namespace: /db/
name: pg01

restapi:
  listen: 0.0.0.0:8008
  connect_address: 192.168.10.106:8008

etcd3:
  hosts: 192.168.10.141:2379,192.168.10.124:2379,192.168.10.140:2379

bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
      parameters:
        wal_level: replica
        hot_standby: "on"
        max_wal_senders: 10
        max_replication_slots: 10
        wal_keep_size: 256MB

  initdb:
    - encoding: UTF8
    - data-checksums

  pg_hba:
    - host  all  all  0.0.0.0/0  md5
    - host  replication  all  0.0.0.0/0  md5

  users:
    admin:
      password: adminpass
      options:
        - createrole
        - createdb

postgresql:
  listen: 0.0.0.0:5432
  connect_address: 192.168.10.106:5432
  data_dir: /var/lib/postgresql/16/data/cluster
  bin_dir: /usr/lib/postgresql/16/bin
  authentication:
    replication:
      username: replicator
      password: replpass
    superuser:
      username: postgres
      password: postgrespass
   ```
4. 	Создаем и настраиваем ВМ haproxy
```
sudo apt install -y haproxy
sudo nano /etc/haproxy/haproxy.cfg
sudo systemctl restart haproxy
```
конфиг haproxy
```
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin
        stats timeout 30s
        user haproxy
        group haproxy
        daemon
        maxconn 256

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES2>
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
        log     global
        mode    tcp
        option  httplog
        option  dontlognull
        timeout connect 500
        timeout client  5000
        timeout server  5000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

frontend postgresql
    bind *:5432
    default_backend postgresql_nodes

backend postgresql_nodes
    option httpchk OPTIONS /master
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions

    server node1 192.168.10.106:5432 check port 8008
    server node2 192.168.10.142:5432 check port 8008
    server node3 192.168.10.138:5432 check port 8008

```
5. Проверем работу haproxy, подключаюсь через DBeaver с локальноко хоста на котором развернуты виртуалки.
   ![2](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_11.Кластеры%20высокой%20доступности/source/Screenshot%202026-02-13%20143029.png?raw=true)
7. Проверяем отказоустойчивость.
   
на текущем мастере, pg01:

  `sudo systemctl stop patroni`
  
   На pg02:
   
`patronictl -c /etc/patroni/config.yml list`

![3](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_11.Кластеры%20высокой%20доступности/source/Screenshot%202026-02-13%20143308.png?raw=true)

Видим что мастер переехал на pg03.
