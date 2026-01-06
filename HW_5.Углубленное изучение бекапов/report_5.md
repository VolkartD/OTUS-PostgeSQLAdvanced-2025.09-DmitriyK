# Изучение бекапов на примере WAL-G
1. Создаем стенд из 3-х хостов.
2. Устанавливаем PostgreSQL, nano, mc.
   `sudo apt install postgresql postgresql-contrib nano mc -y`
3. Настраиваем PostgreSQL на pg01.
```
sudo nano /etc/postgresql/16/main/pg_hba.conf
sudo nano /etc/postgresql/16/main/postgresql.conf
listen_addresses = '*'          # what IP address(es) to listen on;
wal_level = replica                     # minimal, replica, or logical
archive_mode = on               # enables archiving; off, on, or always
archive_command = '/usr/local/bin/wal-g wal-push \”%p\” >> /var/log/postgresql/archive_command.log 2>&1' # command to use to archive a WAL file
archive_timeout = 60            # force a WAL file switch after this	
restore_command = '/usr/local/bin/wal-g wal-touch \”%p\” >> /var/log/postgresql/archive_command.log 2>&1'
sudo systemctl restart postgresql
sudo systemctl status postgresql
```
4. Создаем тестовую БД на pg01.
```
sudo -su postgres
psql
Create Database loyalty;
pgbench -i -s 100 loyalty
```
5. Cоздаем ключи, добавляем хосты в доверенные.
```
sudo -u postgres ssh-keygen
sudo nano /var/lib/postgresql/.ssh/id_ed25519.pub
sudo -u postgres nano /var/lib/postgresql/.ssh/authorized_keys
```
6. Устанавливаем WAL-G
```
wget https://github.com/wal-g/wal-g/releases/download/v3.0.5/wal-g-pg-ubuntu-22.04-amd64.tar.gz
tar -xvf wal-g-pg-ubuntu-22.04-amd64.tar.gz
sudo mv wal-g-pg-ubuntu-22.04-amd64 /usr/local/bin/wal-g
```
7. Настройка конфига WAL-G pg01.
```
sudo su - postgres
nano /var/lib/postgresql/.walg.json
```
![1](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_5.Углубленное%20изучение%20бекапов/screenshots/Screenshot%202026-01-06%20212946.png?raw=true)
8. Создаем каталог для хранения бекапа pg01
`sudo -su postgres mkdir -p /var/lib/postgresql/sql_backup`
9. Создаем пользователя для wal-g.
```
sudo -su postgres
psql
create user backuper password 'qwe123' superuser createdb createrole replication;
```
10. Проверяем WAL-G делая бекап.
```
sudo -su postgres
/usr/local/bin/wal-g backup-push /var/lib/postgresql/16/main
```
![2](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_5.Углубленное%20изучение%20бекапов/screenshots/Screenshot%202026-01-06%20193552.png?raw=true)
11. Настройка конфига WAL-G pg02 и pg03.
```
sudo su - postgres
nano /var/lib/postgresql/.walg.json
```
![3](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_5.Углубленное%20изучение%20бекапов/screenshots/Screenshot%202026-01-06%20212952.png?raw=true)
12. Восстанеавливаем БД на pg02.
```
sudo su - postgres
/usr/local/bin/wal-g backup-fetch /var/lib/postgresql/16/main LATEST
```
![4](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_5.Углубленное%20изучение%20бекапов/screenshots/Screenshot%202026-01-06%20210015.png?raw=true)
13. Проверяем.
14. Создаем для теста таблицу.
```
sudo -su postgres
psql
create table test(id serial,dt timestamp default now());
```
15.Снова выполняем бекап на pg01
```
sudo -su postgres
/usr/local/bin/wal-g backup-push /var/lib/postgresql/16/main
```
16. Восстанеавливаем БД на pg03
```
sudo -su postgres
/usr/local/bin/wal-g backup-fetch /var/lib/postgresql/16/main LATEST
```
17. Проверяем катлог и бд
![5](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_5.Углубленное%20изучение%20бекапов/screenshots/Screenshot%202026-01-06%20210136.png?raw=true)
19. Удаляем стенд
