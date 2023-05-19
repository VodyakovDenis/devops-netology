# site.yml

## Описание плейбука:

Название: Install ClickHouse

Handlers:

- Start clickhouse service - перезапускает сервис clickhouse-server.

Tasks:

- AptInst - устанавливает необходимые пакеты для работы с репозиториями APT.

- Add ClickHouse repository key - добавляет ключ репозитория ClickHouse.

- Add ClickHouse repository - добавляет репозиторий ClickHouse.

- Update APT cache - обновляет кэш APT.

- Install ClickHouse server and client - устанавливает сервер и клиент ClickHouse.

- Start ClickHouse server - запускает сервер ClickHouse.

- Flush handlers - сбрасывает обработчики.

- Create database - создает базу данных logs в ClickHouse. В случае ошибки выходит с ошибкой только при коде возврата, отличном от 0 и 82. Если выполнение команды прошло успешно, то помечает задачу как измененную.

### Парметры

В данном случае в отличии от базового плейбука под centos не задаются, устанавливается последний стабильный дистрибутив из репозитория.


# vector.yml

## Описание плейбука:

Название: Install Vector

Handlers: Отсутствуют

Tasks:

- Install tar - устанавливает пакет tar для работы с архивами.

- Vector distrib - загружает дистрибутив Vector по указанной ссылке.

- Unpack vector distrib - распаковывает загруженный дистрибутив в текущую директорию.

- Install vector - копирует исполняемый файл vector в /usr/local/bin/ и устанавливает ему права на выполнение.

- Check vector version - проверяет версию установленного Vector. Если версия не соответствует заданной, помечает задачу как измененную.

- Теги используются для группировки задач и упрощения запуска плейбука с выборочным выполнением задач.

### Параметры

**vector_version** - необходимо задать версию которую будем устанавливать.


# all.yml

Включает в себе плейбуки site.yml и vector.yml, но добавлены теги для запуска.
