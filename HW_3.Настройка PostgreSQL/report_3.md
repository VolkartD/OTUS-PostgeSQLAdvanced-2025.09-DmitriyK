# Настройка PostgreSQL
1. Установка и настройка виртуальной машины.
2. Установка PostgreSQL.
	`sudo apt-get install postresql -y`
3. Создаем БД с таблицей.
  ```
  CREATE DATABASE test;
  pgbench -i test 
  ```
4. Подключаем второй диск. ![1](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_3.Настройка%20PostgreSQL/screenshots/2.png?raw=true)
	- проверяем что диск добавился.
		`lsblk`
	- инициализируем диск.
		`sudo fdisk /dev/sdb`
	- делаем файловую систему.
		`sudo mkfs.ext4 /dev/sb1`
	- создаем раздел.
		`sudo mkdir /postgresql`
	- монтируем новый диск в раздел.
		`sudo mount /dev/sdb1 /postgresql`
![2](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_3.Настройка%20PostgreSQL/screenshots/6.png?raw=true)
![3](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_3.Настройка%20PostgreSQL/screenshots/7.png?raw=true)
5. Переносим БД на новый диск.
 	- останавливаем службу postgresql.
		`sudo systemctl stop postgresql`
	- копируем данные с сохранением прав.
		`sudo rsync -av /var/lib/postgresql/16/main/ /postgresql`
	- обновляем конфигурацию, меняем строку data_directory.
		`sudo nano /etc/postgresql/16/main/postgresql.conf`
	- запускаем службу.
	  `sudo systemctl start postgresql`
	- проверяем что данные сохранились.
	```
    sudo -su postgres
   psql
   \l
   ```
 ![4](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_3.Настройка%20PostgreSQL/screenshots/9.png?raw=true)
 ![5](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_3.Настройка%20PostgreSQL/screenshots/13.png?raw=true)
