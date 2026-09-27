# 🚀 Развертывание Joomla с использованием Docker Compose

**Автор:** Абрамов Даниил Сергеевич

Проект демонстрирует развертывание CMS **Joomla** в контейнерах Docker с использованием **Docker Compose** и базы данных **MariaDB**.

В результате будут запущены два контейнера:

- 🌐 **Joomla** — веб-приложение;
- 🗄️ **MariaDB** — база данных.

Joomla будет доступна по адресу:

```text
http://localhost:8082
```

---

## 📋 Требования

Перед началом работы необходимо установить и запустить:

- Docker Desktop;
- Docker Compose.

Проверить наличие уже запущенных Compose-проектов можно командой:

```bash
docker compose ls
```

Также рекомендуется убедиться, что порт `8082` не используется другим приложением.

---

## 📁 1. Создание проекта

Структура проекта:

```text
joomla-docker/
├── compose.yaml
└── img/
    ├── 01_Terminal_Status_Abramov_D_S.png
    ├── 02_Joomla_Frontend_Abramov_D_S.png
    ├── 03_Joomla_Admin_Abramov_D_S.png
    └── 04_MariaDB_Logs_Abramov_D_S.png
```

Создадим каталог проекта и файл конфигурации:

```bash
mkdir -p joomla-docker
touch joomla-docker/compose.yaml
cd joomla-docker
```

---

## ⚙️ 2. Настройка `compose.yaml`

В файл `compose.yaml` необходимо добавить следующую конфигурацию:

```yaml
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
```

### Используемые сервисы

| Сервис | Образ | Назначение |
|---|---|---|
| `db` | `mariadb:11.5.2` | Хранение данных Joomla |
| `joomla` | `joomla:latest` | CMS Joomla |

Порт контейнера Joomla `80` пробрасывается на локальный порт `8082`:

```text
localhost:8082 → container:80
```

> ⚠️ Пароли в данном файле приведены в учебных целях. В реальном проекте конфиденциальные данные рекомендуется хранить в `.env` или Docker Secrets.

---

## ▶️ 3. Запуск проекта

Запуск контейнеров в фоновом режиме:

```bash
docker compose up -d
```

Проверка состояния контейнеров:

```bash
docker compose ps
```

После успешного запуска оба сервиса должны иметь статус `Up`.

### Результат

![Статус Docker-контейнеров](img/01_Terminal_Status_Abramov_D_S.png)

---

## 🗄️ Проверка MariaDB

Для просмотра логов базы данных выполните:

```bash
docker compose logs db
```

В логах можно убедиться, что MariaDB успешно запущена и готова принимать подключения.

![Логи MariaDB](img/04_MariaDB_Logs_Abramov_D_S.png)

---

## 🌐 4. Установка Joomla

После запуска контейнеров необходимо открыть браузер и перейти по адресу:

```text
http://localhost:8082
```

Откроется веб-установщик Joomla.

### Шаг 1. Настройка сайта

Необходимо указать:

- название сайта;
- имя администратора;
- логин администратора;
- пароль;
- адрес электронной почты.

### Шаг 2. Подключение к базе данных

Параметры должны соответствовать значениям из `compose.yaml`:

| Параметр | Значение |
|---|---|
| Тип базы данных | `MySQLi` |
| Сервер базы данных | `db` |
| Имя пользователя | `joomla_user` |
| Пароль | `joomla_password` |
| Имя базы данных | `joomla_db` |

> В качестве адреса сервера необходимо указывать именно `db`, а не `localhost`, поскольку Joomla подключается к MariaDB через внутреннюю Docker-сеть.

### Шаг 3. Завершение установки

После проверки параметров необходимо выполнить установку Joomla.

По завершении установки следует удалить директорию `installation`, если установщик предложит это сделать.

---

## ✅ Результат

После успешной установки будет доступна пользовательская часть сайта:

```text
http://localhost:8082
```

![Главная страница Joomla](img/02_Joomla_Frontend_Abramov_D_S.png)

Также можно перейти в панель администратора:

```text
http://localhost:8082/administrator
```

![Панель администратора Joomla](img/03_Joomla_Admin_Abramov_D_S.png)

---

## 🛠️ 5. Полезные команды

Все команды необходимо выполнять из каталога:

```text
joomla-docker
```

### Просмотр логов Joomla

```bash
docker compose logs -f joomla
```

Для выхода из режима просмотра логов:

```text
Ctrl + C
```

### Остановка контейнеров

```bash
docker compose stop
```

Данные при этом сохраняются в Docker volumes.

### Повторный запуск

```bash
docker compose start
```

### Просмотр состояния

```bash
docker compose ps
```

### Перезапуск сервисов

```bash
docker compose restart
```

---

## 🗑️ 6. Удаление проекта

Для остановки контейнеров и удаления созданных volumes:

```bash
docker compose down -v
```

> ⚠️ Параметр `-v` удаляет тома `db_data` и `joomla_data`, поэтому данные сайта и базы данных будут потеряны.

После этого можно удалить директорию проекта:

```bash
cd ..
rm -rf joomla-docker
```

---

## 🏗️ Архитектура проекта

```text
                    ┌──────────────────┐
                    │     Browser      │
                    └────────┬─────────┘
                             │
                    http://localhost:8082
                             │
                             ▼
                 ┌──────────────────────┐
                 │      Joomla          │
                 │    joomla:latest     │
                 │       port 80        │
                 └──────────┬───────────┘
                            │
                     Docker Network
                   joomla-network
                            │
                            ▼
                 ┌──────────────────────┐
                 │      MariaDB         │
                 │   mariadb:11.5.2     │
                 │      port 3306       │
                 └──────────────────────┘
```

---

## 📌 Итог

В рамках проекта была развернута CMS **Joomla** с использованием **Docker Compose**.

Были настроены:

- отдельный контейнер Joomla;
- отдельный контейнер MariaDB;
- внутренняя Docker-сеть;
- постоянное хранение данных через Docker volumes;
- доступ к Joomla через локальный порт `8082`;
- подключение Joomla к MariaDB внутри Docker-сети.

Проект позволяет быстро развернуть готовую среду Joomla без ручной установки PHP, веб-сервера и СУБД.

---

**Автор:** Абрамов Даниил Сергеевич