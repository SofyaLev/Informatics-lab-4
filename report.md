# Лабораторная работа 4

## Задание
Запустить в контейнере приложение “aafire”. Обратите внимание, что оно бесконечное и контейнер не будет автоматически отключаться.  
Приложить скриншот в процессе работы контейнера.  

Далее в рамках лабораторной работы необходимо самостоятельно настроить сеть между двумя контейнерами, также как в предыдущей работе вы настраивали связь между двумя виртуальными машинами.

## Решение
1. Устанавливаем Docker
2. Создадим папку с именем лабораторной работы `lab4` и перейдем в нее:
```
mkdir lab4 && cd lab4
```
3. Создаем файл с именем `Dockerfile` и открываем его в редакторе `nano` с помощью следующей команды:
```
touch Dockerfile && nano Dockerfile
```
4. Введем в файл следующий код:
```
FROM ubuntu:latest
RUN apt-get update && apt-get install -y libaa-bin fortune && apt-get install iputils-ping -y
```
![докерфайл](https://github.com/user-attachments/assets/9eb99499-594e-4a5d-822f-88a3be5d146f)

*Объяснение:*
- `FROM ubuntu:latest` — указывает образ `Ubuntu`, на основе которого будут работать контейнеры
- `RUN apt-get update && apt-get install -y libaa-bin fortune && apt-get install iputils-ping -y` указывает, какие команды необходимо выполнить в контейнере во время сборки образа:
  - устанавить пакет `libaa-bin` (содержит `aafire`).
  - устанавить `iputils-ping` (утилита `ping`).

5. В папке с файлом `Dockerfile` выполним команду:
```
sudo docker build -t aafire_image .
```
Получим:
![сборка образа](https://github.com/user-attachments/assets/142e89f3-5797-432a-8e65-e95e69e1ff0a)

*Объяснение:*
- `docker build` — собирает Docker-образ.
- `-t aafire_image` — задаёт имя образу.
- `.` — указывает, что `Dockerfile` находится в текущей директории.

Проверим успешность сборки:
```
sudo docker images
```
![проверка сборки образа](https://github.com/user-attachments/assets/b6ffd999-92d1-4b60-a6a4-6b3c1d56a719)

6. Запустим контейнеры с помощью следующих команд:
```
sudo docker run --name first_cont -it aafire_image
sudo docker run --name second_cont -it aafire_image
```

*Объяснение:*
- `docker run` — запускает новый контейнер.
- `--name <container_name>` — задаёт имя контейнеру.
- `-it` — запускает контейнер в интерактивном режиме.
- `aafire_image` — имя образа, на основе которого будет создан контейнер.
  
Увидим анимацию огня (она бесконечна, поэтому завершим процесс с помощью сочетания клавиш `Ctrl + C`):
![огонь1](https://github.com/user-attachments/assets/51d2c7fe-c7a6-467b-aa10-afeaff5f2076)

7. Проверим, что оба контейнера запущены:
```
sudo docker ps -a
```
![созданы оба контейнера](https://github.com/user-attachments/assets/afa3c22c-c279-420b-9d93-0add12a27413)

Если же они выключены, можно запустить их, используя команду:
```bash
sudo docker start first_cont second_cont
```

8. Создадим и подключим сеть:
```
sudo docker network create Network
```
Подключим контейнеры к сети:
```
sudo docker network connect Network first_cont
sudo docker network connect Network second_cont
```
Проверим подключение контейнеров:
```
sudo docker network inspect Network
```
Получим следующий результат:
![проверка сети1](https://github.com/user-attachments/assets/0c79830d-92bd-49b6-8a4f-1f4944863fa8)
![проверка сети2](https://github.com/user-attachments/assets/121a7411-5ea1-438a-b9b0-0c737f7ccc3f)

Важная для нас информация:
- В поле `Containers` должны быть данные о созданных контейнерах:
![image](https://github.com/user-attachments/assets/09bb48ca-9c8b-4e99-b53e-cc7c5f293228)
- IP адреса контейнеров в полях `IPv4Address`:
  - `first_cont` — `172.18.0.2`
  - `second_cont` — `172.18.0.3`

9. Проверим соединение. Для этого сначала запустим `bash` внутри `first_container`:
```
sudo docker exec -it first_cont bash
```
*Объяснение:*
- `docker exec` — запускает команду внутри работающего контейнера.
- `-it` — интерактивный режим.
- `bash` — запускает оболочку Bash внутри контейнера.

Проверим соединение с `second_cont`:
```
ping 172.18.0.3
```
![соединение со вторым](https://github.com/user-attachments/assets/096e0dd0-dcc2-4bf7-87f4-6e6a4ff96a33)


Аналогично проверяем доступ `second_cont` к `first_cont`:
![соединение с первым](https://github.com/user-attachments/assets/3b31b5ab-b721-49cd-8171-b80892db2ccc)

10. Выключим контейнеры и проверим, что их статус изменился:
```
sudo docker stop first_cont second_cont
```
```
sudo docker ps -a
```
![выкл контейнеров и проверка](https://github.com/user-attachments/assets/88e6eec4-9e3c-47ab-b218-724de6c8da78)

