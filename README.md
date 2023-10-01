# Установка PostgresQL

## Установка homebrew 

- Этот шаг необходимо выполнить, что бы установить СУБД PostgresQL без лишних заморочек

- homebrew - менеджер пакетов для ОС MacOS

```bash
     /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```


## Установка PostgresQL 14-той версии

- Этот шаг необходимо выполнить для того, что бы СУБД была локально установлена на ваш компьютер

```bash 
    brew install postgresql
```

### Возможные ошибки

- MacOS может не увидеть установленные утилиты и СУБД, исправить это просто - достаточно протсо обновить файлы PATH (./zshrc)

```bash
    export PATH="/usr/local/opt/postgresql/bin:$PATH"

    source ~/.zshrc
```

## Создание тестовой базы данных

- Присупим к созданию тестовой базы данных, для этого воспользуемся стандартной утилитой createdb

```bash
    createdb ИМЯ_БАЗЫ_ДАННЫХ
```

- Вместо "ИМЯ_БАЗЫ_ДАННЫХ" используйте свое название, в нашем случае - petrsu

```bash
    createdb petrsu
```

## Создание пользователя с правами суперпользователя

- Обычно Постгрес делает это сама, но так как мы устанавливали ее через homebrew а не запусали внутри докера, то пользователи не были созданы автоматически. Сейчас мы это исправим. 

```bash
    createuser ИМЯ_ПОЛЬЗОВАТЕЛЯ
```

- Данная команда создаст первичного пользователя (суперпользователя)

### Взможные проблемы

- Если версия MacOS выше Ventura, то пользователь будет создан без прав суперпользователя, это можно исправить используея утиллиту psql

```bash
psql -U ИМЯ_ПОЛЬЗОВАТЕЛЯ

CREATE USER postgres WITH PASSWORD 'ваш_пароль';

ALTER USER postgres WITH SUPERUSER;

\q
```

## Подключение к базе данных с правами суперпользователя

- Используем по Dbeaver (полностью бесплатное и мультиплатформенное) что бы подключиться к БД.

![Интерфейс DBEAVER](<static/Снимок экрана 2023-10-01 в 17.23.02.png>)

- В левом верхнем углу нажимем на вилку с зеленым плюсиком, открывается окошко выбора СУБД

![Alt text](<static/Снимок экрана 2023-10-01 в 17.24.01.png>)

- В поиске вводим "PostgresQL" и выбираем иконку со слоном

![Alt text](<static/Снимок экрана 2023-10-01 в 17.24.45.png>)

- Нажимаем кнопку далее и поподаем в окно настроек

![Alt text](<static/Снимок экрана 2023-10-01 в 17.25.19.png>)

- В поле "хость" ввводим 'localhost' - обозначение того что база данных развернута локально
- В поле "порт" вводим значение 5432
- В поле "База данных" вводим имя созданной ранее базы данных
- В поле "пользователь" вводим "postgres"
- в поле "пароль" вводим указанный ранее пароль
- Нажимаем кнопку "Проверить соединение"

![Alt text](<static/Снимок экрана 2023-10-01 в 17.27.31.png>)

- Должно появиться окшко "Соединено"
- Нажимаем на кнопку "Готово"
- Разворачиваем вкладку "petrsu" -> "Схемы" -> "public" -> "Таблицы"

- Нажимаем правой кнопкой мышки по "public" и выбираем пунк "Редактор SQL" -> "Открыть SQL скрипт"

- Поочередно вставляем и выполняем следующий код в открывшемся окне

```sql
-- Создаем таблицу для тестовых данных
CREATE TABLE test_table (
    id serial PRIMARY KEY,
    name VARCHAR (255),
    age INT
);

-- Вставляем тестовые данные
INSERT INTO test_table (name, age)
VALUES
    ('John Doe', 30),
    ('Jane Smith', 25),
    ('Bob Johnson', 35);
```

- Теперь у нас есть данные в базе, ура!
- 🇷🇺🇷🇺🇷🇺🇷🇺🇷🇺 СЛАВА РОССИИ!!! 🇷🇺🇷🇺🇷🇺🇷🇺🇷🇺


## Копирование базы данных по расписанию

- Создаем файл backup_script.sh

Вставляем следующий код:

```sh
#!/bin/bash

# Путь к каталогу, где будет храниться резервная копия
BACKUP_DIR="/Users/egorageev/Desktop/DB_HUESOSY/"

# Имя файла резервной копии
BACKUP_FILE="backup_$(date +%Y%m%d%H%M%S).sql"

# Имя вашей базы данных PostgreSQL
DATABASE_NAME="****"

# Пароль для пользователя PostgreSQL
PGPASSWORD="****"

# Выполнение резервного копирования с использованием pg_dump
pg_dump -U postgres -d "$DATABASE_NAME" > "$BACKUP_DIR/$BACKUP_FILE"

# Опционально: удаление старых резервных копий (например, если их слишком много)
# find "$BACKUP_DIR" -name "backup_*" -mtime +7 -exec rm {} \;
```

- Теперь сделаем файл исполняемым

```bash
    chmod +x backup.sh
```

- Теперь вызовем редактор вим для файла конфигурации cron

```
crontab -e
```

- Скопируем следующий фрагмент кода и вставим внутри нашего файла конфигурации

```nano
    0 2 * * * /полный/путь/к/backup.sh
```

- Вместо "/полный/путь/к/backup.sh" укажите полный путь к вашему файлу .sh

### Что такое CRON и с чем его едят

- Выражение щас обхясним
    - Первое поле 0: Это минуты. В данном случае, 0 означает "0 минут". То есть задача будет выполняться в начале каждого часа.
    - Второе поле - это часы. Задача будет выполняться во втором часу.
    - Третье поле *: Это дни месяца. Звездочка (*) означает "каждый день месяца". То есть задача будет выполняться в любой день месяца.
    - Четвертое поле *: Это месяцы. Звездочка (*) означает "каждый месяц". То есть задача будет выполняться в любом месяце.
    - Пятое поле *: Это дни недели. Звездочка (*) означает "каждый день". То есть задача будет выполняться каждый день.  
- Таким образом, данное cron выражение означает, что ваш скрипт backup.sh будет запускаться каждый день во втором часу утра (2:00 AM).


## Создадим дамп базы (резервную копию)

- Если вы не зафакапили и все сделали правильно, то просто откройте файл .backup_script.sh

```bash
    ./backup_script.sh
```

- В указааной внутри скрипта дирриктории появится файл backup_2023..etc.sql, это дамп нашей базы
- Теперь снесите нахуй созданную ранее табличку внутри базы, надеюсь что вы знаете как это сдлеать, мне лень расписывать
- Снесли? Супер! Теперь запустите файл дампа используя psql

```bash
    psql -U postgres -d имя_базы_данных -f имя_файла_дампа.sql
```

# УРА, ВАША БАЗЗА ВОСТАНОВЛЕНА, РЕЗЕРВНОЕ КОПИРОВАНИЕ НАСТРОЕНО! 

#### Зеленский лох!