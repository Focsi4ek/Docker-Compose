# 🚀 Развертывание Joomla с использованием Docker Compose

**Автор:** Абрамов Даниил Сергеевич

Проект демонстрирует развертывание CMS **Joomla** с использованием **Docker Compose** и базы данных **MariaDB**.

В результате запускаются два контейнера:

- **Joomla** — веб-приложение
- **MariaDB** — база данных

После запуска сайт будет доступен по адресу:

[http://localhost:8082](http://localhost:8082)

---

## 📋 Требования

Перед началом работы необходимо убедиться, что установлен и запущен **Docker Desktop**.

Проверить существующие Docker Compose-проекты можно командой:

    docker compose ls

Также необходимо убедиться, что порт 8082 не занят другим приложением или контейнером.

---

## 📁 1. Создание проекта

Структура проекта:

    joomla-docker/
    ├── README.md
    ├── compose.yaml
    └── img/
        ├── 01-docker-compose-ps.png
        ├── 02-joomla-frontend.png
        ├── 03-joomla-admin-dashboard.png
        └── 04-mariadb-logs.png

Создание каталога проекта и файла конфигурации через терминал macOS:

    mkdir -p joomla-docker
    touch joomla-docker/compose.yaml
    cd joomla-docker

---

## ⚙️ 2. Настройка compose.yaml

Содержимое файла:

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
          - 8082:80
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

### Используемые сервисы

| Сервис | Docker-образ | Назначение |
|---|---|---|
| db | mariadb:11.5.2 | База данных |
| joomla | joomla:latest | CMS Joomla |

Порт 80 контейнера Joomla пробрасывается на локальный порт 8082.

> ⚠️ Пароли в данном проекте используются в учебных целях. В реальном проекте конфиденциальные данные рекомендуется хранить в файле .env или Docker Secrets.

---

## ▶️ 3. Запуск проекта

Запуск контейнеров в фоновом режиме:

    docker compose up -d

Проверка состояния контейнеров:

    docker compose ps

После успешного запуска контейнеры должны иметь статус Up.

### Результат запуска

![Статус Docker Compose](./img/01-docker-compose-ps.png)

---

## 🗄️ Проверка MariaDB

Для просмотра логов базы данных выполнить:

    docker compose logs db

В логах можно убедиться, что сервер MariaDB успешно запущен и готов принимать подключения.

![Логи MariaDB](./img/04-mariadb-logs.png)

---

## 🌐 4. Установка Joomla

После запуска контейнеров необходимо открыть браузер и перейти по адресу:

[http://localhost:8082](http://localhost:8082)

Откроется мастер установки Joomla.

### Шаг 1. Настройка сайта

На первом этапе необходимо указать:

- название сайта
- имя администратора
- логин администратора
- пароль администратора
- адрес электронной почты

### Шаг 2. Подключение к базе данных

Параметры подключения должны соответствовать значениям в compose.yaml.

| Параметр | Значение |
|---|---|
| Тип базы данных | MySQLi |
| Сервер базы данных | db |
| Имя пользователя | joomla_user |
| Пароль | joomla_password |
| Имя базы данных | joomla_db |

> В поле сервера базы данных необходимо указывать db, а не localhost, так как Joomla подключается к MariaDB через внутреннюю Docker-сеть.

### Шаг 3. Завершение установки

После заполнения всех параметров необходимо выполнить установку Joomla.

После завершения установки следует удалить директорию installation, если Joomla предложит это сделать.

---

## ✅ 5. Результат

После успешной установки пользовательская часть Joomla будет доступна по адресу:

[http://localhost:8082](http://localhost:8082)

### Главная страница Joomla

![Главная страница Joomla](./img/02-joomla-frontend.png)

### Панель администратора

Панель управления Joomla доступна по адресу:

[http://localhost:8082/administrator](http://localhost:8082/administrator)

![Панель администратора Joomla](./img/03-joomla-admin-dashboard.png)

---

## 🛠️ 6. Полезные команды

Все команды необходимо выполнять из директории проекта joomla-docker.

### Просмотр логов Joomla

    docker compose logs -f joomla

Для выхода из режима просмотра логов:

    Ctrl + C

### Просмотр состояния контейнеров

    docker compose ps

### Остановка контейнеров

    docker compose stop

При остановке контейнеров данные сохраняются.

### Запуск остановленных контейнеров

    docker compose start

### Перезапуск контейнеров

    docker compose restart

---

## 🗑️ 7. Удаление проекта

Для остановки контейнеров и удаления Docker volumes:

    docker compose down -v

Параметр -v удаляет тома db_data и joomla_data, поэтому данные сайта и базы данных будут потеряны.

После этого можно удалить рабочую директорию:

    cd ..
    rm -rf joomla-docker

---

## 🏗️ Архитектура проекта

                  ┌─────────────────────┐
                  │       Browser       │
                  └──────────┬──────────┘
                             │
                    localhost:8082
                             │
                             ▼
                  ┌─────────────────────┐
                  │       Joomla        │
                  │    joomla:latest    │
                  │       port 80       │
                  └──────────┬──────────┘
                             │
                      joomla-network
                             │
                             ▼
                  ┌─────────────────────┐
                  │       MariaDB       │
                  │   mariadb:11.5.2    │
                  │      port 3306      │
                  └─────────────────────┘

---

## 📌 Итог

В рамках проекта была развернута CMS **Joomla** с использованием Docker Compose.

Были настроены:

- контейнер Joomla
- контейнер MariaDB
- внутренняя Docker-сеть
- постоянное хранение данных через Docker volumes
- подключение Joomla к MariaDB
- доступ к сайту через локальный порт 8082
- доступ к административной панели Joomla

Использование Docker позволяет быстро развернуть Joomla без отдельной ручной установки PHP, Apache и MariaDB.

---

**Автор:** Абрамов Даниил Сергеевич