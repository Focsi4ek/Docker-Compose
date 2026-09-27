# Развертывание CMS Joomla с использованием Docker Compose
**Автор:** Абрамов Даниил Сергеевич

**Joomla!** — бесплатная система управления контентом (CMS) с открытым исходным кодом, написанная на PHP и JavaScript. В данном проекте система используется в связке с реляционной базой данных **MariaDB**.

Перед началом работы необходимо убедиться, что Docker Desktop запущен, и нет конфликтующих контейнеров, занимающих нужные порты:
```shell
docker compose ls


1. Создание каталога проекта

Структура проекта:

joomla-docker/
├── compose.yaml
└── img/
    ├── 01-docker-compose-ps.png
    ├── 02-joomla-frontend.png
    ├── 03-joomla-admin-dashboard.png
    └── 04-mariadb-logs.png


Создание каталога проекта и пустого файла конфигурации через терминал (macOS):

mkdir -p joomla-docker && touch joomla-docker/compose.yaml && cd joomla-docker


2. Файл конфигурации compose.yaml

Содержимое файла конфигурации:

services:
  db:
    image: mariadb:11.5.2
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: example_root_password
      MYSQL_DATABASE: joomla_db
      MYSQL_USER: joomla_user
      MYSQL_PASSWORD: joomla_password
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - joomla-network

  joomla:
    depends_on:
      - db
    image: joomla:latest
    ports:
      - "8082:80"
    restart: unless-stopped
    environment:
      JOOMLA_DB_HOST: db:3306
      JOOMLA_DB_USER: joomla_user
      JOOMLA_DB_PASSWORD: joomla_password
      JOOMLA_DB_NAME: joomla_db
    volumes:
      - joomla_data:/var/www/html
    networks:
      - joomla-network

networks:
  joomla-network:

volumes:
  db_data:
  joomla_data:


3. Запуск проекта и проверка статуса

Запуск контейнеров в фоновом режиме:

docker compose up -d


Проверка статуса поднятых контейнеров:

docker compose ps


![](01_docker_compose_ps.png) 


Проверка готовности базы данных к подключениям:

docker compose logs db


![](04_mariadb_logs.png) 


4. Процесс установки и настройки Joomla

Переход к веб-установщику Joomla осуществляется в браузере по адресу: http://localhost:8082

Этапы настройки:

1. Задание названия сайта и создание учетной записи администратора.
2. Настройка подключения к базе данных строго в соответствии с compose.yaml:
  ⚬ Тип базы данных: MySQLi
  ⚬ Имя сервера баз данных: db
  ⚬ Имя пользователя: joomla_user
  ⚬ Пароль к БД: joomla_password
  ⚬ Имя базы данных: joomla_db
3. Установка платформы.
4. Удаление директории installation по завершении процесса.

Результаты установки:

![](02_joomla_frontend.png)


![](03_joomla_admin_dashboard.png) 

5. Полезные команды управления

Находясь в директории joomla-docker, можно управлять проектом:

⚬ Просмотр логов приложения Joomla в реальном времени (Ctrl+C для выхода):
  docker compose logs -f joomla
  
⚬ Приостановка работы контейнеров (без удаления данных):
  docker compose stop
  
⚬ Запуск приостановленных контейнеров:
  docker compose start
  

6. Удаление проекта

Для полного удаления проекта, остановки контейнеров и очистки томов с данными (volumes):

docker compose down -v


Удаление рабочей директории:

cd ..
rm -rf joomla-docker