# Построение кластера Patroni.
1. Создаем виртуальные машины, 3 для etcd, 3 patroni, 1 haproxy.
   
   ![1](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_4.Высокая%20доступность%3A%20развертывание%20Patroni/source/Screenshot%202026-02-13%20153525.png)

3. Устанавливаем, настраиваем etcd.
   ```
   sudo apt-get update -y && sudo apt-get upgrade -y
   sudo apt install -y nano ssh mc
   sudo apt-get install etcd-server -y && sudo apt-get install etcd-client -y
   ```

  Проверяем: `etcd --version`
 
  Настраиваем конфиг etcd.
 
  `sudo nano /etc/default/etcd`
 
  [Config_etcd](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_4.Высокая%20доступность%3A%20развертывание%20Patroni/source/etcd.txt)

  ![2](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_4.Высокая%20доступность%3A%20развертывание%20Patroni/source/Screenshot%202026-02-13%20153944.png)

  `systemctl restart etcd`

  проверяем кластер
 
  `etcdctl --cluster=true endpoint health`
 
  `ETCDCTL_ENDPOINTS=192.168.10.141:2379,192.168.10.124:2379,192.168.10.140:2379 etcdctl endpoint status -w table`

  ![3](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_4.Высокая%20доступность%3A%20развертывание%20Patroni/source/Screenshot%202026-02-13%20154208.png)

3. Установка PostgreSQL настройка Patroni.

  ```
sudo apt install -y nano ssh mc
sudo apt-get update -y && sudo apt-get upgrade -y
sudo apt install -y postgresql postgresql-contrib python3-pip python3-venv
sudo apt install -y patroni
sudo systemctl stop postgresql
sudo systemctl disable postgresql
sudo nano /etc/patroni/config.yml
sudo chmod 750 /var/lib/postgresql/16/data/cluster
sudo mkdir /var/lib/postgresql/16/data/cluster
sudo systemctl enable patroni
sudo systemctl start patroni
```

[patroni_config1](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_4.Высокая%20доступность%3A%20развертывание%20Patroni/source/patroni1.txt)

[patroni_config2](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_4.Высокая%20доступность%3A%20развертывание%20Patroni/source/patroni2.txt)

[patroni_config3](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_4.Высокая%20доступность%3A%20развертывание%20Patroni/source/Screenshot%202026-02-13%20154208.png)

Проверяем:

  `patronictl -c /etc/patroni/config.yml list`

  ![4](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_4.Высокая%20доступность:%20развертывание%20Patroni/source/Screenshot%202026-02-13%20160111.png?raw=true)

4. Настраиваем haproxy.

  ```
sudo apt install -y haproxy
sudo nano /etc/haproxy/haproxy.cfg
sudo systemctl restart haproxy
```

![5](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_4.Высокая%20доступность:%20развертывание%20Patroni/source/Screenshot%202026-02-13%20154831.png?raw=true)

5. Проверем работу haproxy, подключаюсь через DBeaver с локальноко хоста на котором развернуты виртуалки.

![6](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_4.Высокая%20доступность:%20развертывание%20Patroni/source/Screenshot%202026-02-13%20155013.png?raw=true)

6. Проверяем отказоустойчивость.

  на текущем мастере `sudo systemctl stop patroni`

  ![7](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_4.Высокая%20доступность:%20развертывание%20Patroni/source/Screenshot%202026-02-13%20155449.png?raw=true)

  смотрим нового мастера ` patronictl -c /etc/patroni/config.yml list`.

  ![8](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_4.Высокая%20доступность:%20развертывание%20Patroni/source/Screenshot%202026-02-13%20155736.png?raw=true)
