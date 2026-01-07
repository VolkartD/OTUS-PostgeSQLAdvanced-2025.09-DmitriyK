#PostgreSQL в Minikube
1. Развернуть виртуальную машину.
2. Скачиваем бинарник.
  `curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64`
3. Устанавливаем пакет.
  `sudo install minikube-linux-amd64 /usr/local/bin/minikube`
4. Проверяем версию.
  `minikube version`
![1](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_7.PostgreSQL%20в%20Minikube/screenshots/Screenshot%202026-01-07%20182225.png?raw=true)
5. Установка kubectl.
   `sudo apt-get update && sudo apt-get install -y kubectl`
6. Установка зависимостей
   ```
   sudo apt install curl software-properties-common ca-certificates apt-transport-https -y
   wget -O- https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor | sudo tee /etc/apt/keyrings/docker.gpg > /dev/null
   echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu jammy stable"| sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   sudo apt update
   sudo apt install docker-ce -y
   sudo apt install postgresql-client
   ```
7. Запускаем minikube.
   ```
   minikube start
   minikube status
   kubectl get nodes
   ```
   ![2](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_7.PostgreSQL%20в%20Minikube/screenshots/Screenshot%202026-01-07%20183733.png?raw=true)

8. Создаем манифесты.
    `sudo nano demo-namespace.yaml`
   
   ![3](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_7.PostgreSQL%20в%20Minikube/screenshots/Screenshot%202026-01-07%20184553.png?raw=true)

   `sudo nano postgres-secret.yaml`
   
   ![4](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_7.PostgreSQL%20в%20Minikube/screenshots/Screenshot%202026-01-07%20185016.png?raw=true)

   `sudo nano postgres-service.yaml`
   
   ![5](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_7.PostgreSQL%20в%20Minikube/screenshots/Screenshot%202026-01-07%20185126.png?raw=true)

    `sudo nano postgres-statefulset.yaml`
   
   ![6](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_7.PostgreSQL%20в%20Minikube/screenshots/Screenshot%202026-01-07%20185256.png?raw=true)

    `sudo nano postgres-pvc.yaml`
   
   ![7](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_7.PostgreSQL%20в%20Minikube/screenshots/Screenshot%202026-01-07%20185342.png?raw=true)

    `sudo nano postgres-config.yaml`
   
   ![8](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_7.PostgreSQL%20в%20Minikube/screenshots/Screenshot%202026-01-07%20185452.png?raw=true)

10. Создаем namespace.
    ```
    kubectl apply -f demo-namespace.yaml
    kubectl get namespace
    ```
11. Создание Secret.
    ```
    kubectl apply -f postgres-secret.yaml
    kubectl -n test get secret
    ```
12. Создание конфигов.
    ```
    kubectl apply -f postgres-config.yaml
    kubectl -n test get configmap
    ```
13. Создание дисковой подсистемы.
    ```
    kubectl apply -f postgres-pvc.yaml
    kubectl -n test get pvc postgres-pvc
    ```
14. Создание statefulset.
    ```
    kubectl apply -f postgres-statefulset.yaml
    kubectl -n test get pods -l app=postgres
    ```
15. Создание сервиса.
    ```
    kubectl apply -f postgres-service.yaml
    kubectl -n test get svc postgres
    ```
16. Проброс порта.
    `kubectl port-forward -n test svc/postgres 5432:5432`
17. Подключение.
    `psql -h localhost -p 5432 -U otus_user -W -d otus`
    ![9](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_7.PostgreSQL%20в%20Minikube/screenshots/Screenshot%202026-01-07%20191631.png?raw=true)
18. Создаем тестовую таблицу.
    
    ![10](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_7.PostgreSQL%20в%20Minikube/screenshots/Screenshot%202026-01-07%20192513.png?raw=true)
    
20. Меняем манифест что б было создано 3 пода, переподнимаем.
    ```
    kubectl apply -f postgres-statefulset.yaml
    kubectl -n test get pods -l app=postgres
    ```
21. Проверяем что поднялись 3 пода.
    `kubectl get pods --all-namespaces`
    ![11](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_7.PostgreSQL%20в%20Minikube/screenshots/Screenshot%202026-01-07%20193705.png?raw=true)
22. Подключаемся, делаем select.
    ```
    psql -h localhost -p 5432 -U otus_user -W -d otus
    select * from mytest;
    ```
    ![12](https://github.com/VolkartD/OTUS-PostgeSQLAdvanced-2025.09-DmitriyK/blob/main/HW_7.PostgreSQL%20в%20Minikube/screenshots/Screenshot%202026-01-07%20194200.png?raw=true)
    
