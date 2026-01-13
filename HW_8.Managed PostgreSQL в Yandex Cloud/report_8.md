# Managed Service for PostgreSQL.
1. Регистрируемся в YndexCloud.
2. Создаем консоль.
3. Создаем каталог или используем дефолтный.
4. Идем во вкладку "Все сервисы", ищем кластер Managed Service for PostgreSQL.
  ![1](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_8.Managed%20PostgreSQL%20в%20Yandex%20Cloud/screenshots/Screenshot%202026-01-13%20091707.png?raw=true)
  ![2](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_8.Managed%20PostgreSQL%20в%20Yandex%20Cloud/screenshots/Screenshot%202026-01-13%20091718.png?raw=true)
5. Выставляем требуемые настройки кластера.
  ![3](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_8.Managed%20PostgreSQL%20в%20Yandex%20Cloud/screenshots/Screenshot%202026-01-13%20092353.png?raw=true)
6. Создаем кластер.
7. Подключаемся через yndex WebSQL, создаем тестовую таблицу.
  ![5](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_8.Managed%20PostgreSQL%20в%20Yandex%20Cloud/screenshots/Screenshot%202026-01-13%20093447.png?raw=true)
  ![6](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_8.Managed%20PostgreSQL%20в%20Yandex%20Cloud/screenshots/Screenshot%202026-01-13%20093846.png?raw=true)
8. В настройках хоста копируем FQDN.
9. Заходим в Dbeaver, создаем новое подключение.

    ![7](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_8.Managed%20PostgreSQL%20в%20Yandex%20Cloud/screenshots/Screenshot%202026-01-13%20101035.png?raw=true)

10. Проверяем подключение.
  ![8](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_8.Managed%20PostgreSQL%20в%20Yandex%20Cloud/screenshots/Screenshot%202026-01-13%20101140.png?raw=true)
11. Подключаемся через psql:
    11.1. Устанавливаем PostgreSQL для Windows версии 16. Command Line Tools.
    11.2. Устанавливаем сертификат.
      `mkdir $HOME\.postgresql; curl.exe -o $HOME\.postgresql\root.crt https://storage.yandexcloud.net/cloud-certs/CA.pem`
    11.3. Устанавливаем переменные окружения.
      `$Env:PGSSLMODE="verify-full"; $Env:PGTARGETSESSIONATTRS="read-write"`
12. Подключаемся к БД
      ```
            & "C:\Program Files\PostgreSQL\17\bin\psql.exe" `
              >>     --host=rc1b-41vfljs6hs6sia0a.mdb.yandexcloud.net `
              >>     --port=6432 `
              >>     --username=otus_user `
              >>     otus_db
      ```
13. Вводим пароль пользователя.
14. Проверяем.
  ![9](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_8.Managed%20PostgreSQL%20в%20Yandex%20Cloud/screenshots/Screenshot%202026-01-13%20103941.png?raw=true)
15. Удаляем кластер.
