# database-configuration
1. Инициализация кластера БД
     - Имя пользователя — postgres0.
     - Директория кластера БД — $HOME/u23/zn16.
     - Кодировка, локаль — UTF8, английская
     - Перечисленные параметры задать через аргументы команды.
2. Конфигурация и запуск сервера БД
     - Способ подключения к БД — TCP/IP socket, номер порта 9077.
     - Остальные способы подключений запретить.
     - Способ аутентификации клиентов — по имени пользователя.
     - Настроить следующие параметры сервера БД: max_connections,
shared_buffers, temp_buffers, work_mem, checkpoint_timeout,
effective_cache_size, fsync, commit_delay. Параметры должны быть
подобраны в соответствии со сценарием OLAP: 5 пользователей, пакетная
запись данных в среднем по 30 МБ.
     - Директория WAL файлов — поддиректория в PGDATA.
     - Формат лог-файлов — csv.
     - Уровень сообщений лога — INFO.
     - Дополнительно логировать — контрольные точки.
3. Дополнительные табличные пространства и наполнение
     - Пересоздать шаблон template0 в новом табличном пространстве:
         ◦ $HOME/u05/znp08.
     - На основе template1 создать новую базу — theovermind4.
     - От имени новой роли (не администратора) произвести наполнение
существующих баз тестовыми наборами данных. Предоставить права по
необходимости. Табличные пространства должны использоваться по назначению.
     - Вывести список всех табличных пространств кластера и содержащиеся
в них объекты.

# Инициализация кластера
### initdb [параметр...] [ --pgdata | -D ]каталог
Команда initdb создаёт новый кластер баз данных PostgreSQL. Кластер — это коллекция баз данных под управлением единого экземпляра сервера.

Инициализация кластера базы данных заключается в создании каталогов для хранения данных, формировании общих системных таблиц (относящихся ко всему кластеру, а не к какой-либо базе) и создании баз данных template1 и postgres. Впоследствии все новые базы создаются на основе шаблона template1 (все дополнения, установленные в template1 автоматически копируются в каждую новую базу данных). База postgres используется пользователями, утилитами и сторонними приложениями по умолчанию.

При попытке создать каталог для хранения данных initdb может столкнуться с нехваткой прав доступа, если этот каталог принадлежит суперпользователю root. В таком случае необходимо назначить пользователя базы данных владельцем этого каталога при помощи chown. Затем выполнить su для смены пользователя и дальнейшего выполнения initdb.

`initdb --pgdata=$HOME/u23/zn16 --encoding=UTF-8 --locale=en_US --username=postgres0`

![Вывод команды](/images/picture-1.png)
*Подробнее об этом читайте [тут](https://postgrespro.ru/docs/postgresql/9.6/app-initdb#:~:text=initdb%20%D0%B8%D0%BD%D0%B8%D1%86%D0%B8%D0%B0%D0%BB%D0%B8%D0%B7%D0%B8%D1%80%D1%83%D0%B5%D1%82%20%D0%BB%D0%BE%D0%BA%D0%B0%D0%BB%D0%B8%20%D0%B8%20%D0%BA%D0%BE%D0%B4%D0%B8%D1%80%D0%BE%D0%B2%D0%BA%D0%B8,%D0%BF%D1%80%D0%B8%20%D1%81%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%B8%D0%B8%20%D0%BD%D0%BE%D0%B2%D0%BE%D0%B9%20%D0%B1%D0%B0%D0%B7%D1%8B%20%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85.)*.

# Конфигурация и запуск сервер БД
### Настроем способ подключения
Для начала откроем файл **postgresql.conf**, установим порт, а также укажем все адреса через "*"
Обратите внимание, что комментарии "#" нужно убрать!

![Символ "#" нужно убрать](/images/picture-2.png "DELETE '#'")

Далее переходим в файл **pg_hba.conf**. В этом файле настраиваем новый конфиг *host*, который будет отвечать за TCP/IP подключение с методом аутентификации *ident*:

![файл pg_hba.conf](/images/picture-3.png)
Для простоты дальнейшей работы советую временно установить метод *trust*. Этот метод не требует никакой аутентификации для подключения, что является крайне не безопасным, поэтому не забудьте в конце это изменить!
*Подробнее об этом читайте [тут](https://postgrespro.ru/docs/postgresql/9.6/auth-pg-hba-conf).*

### Настройка параметров сервера
Теперь нам нужно настроить следуюущие параметры сервера БД:
  - max_connections
  - shared_buffers
  - temp_buffers
  - work_mem
  - checkpoint_timeout
  - effective_cache_size
  - fsync
  - commit_delay

Параметры должны быть подобраны в соответствии со сценарием **OLAP** : 5 пользователей, пакетная запись данных в среднем по 30 МБ. Начнём по порядку:
```
max_connections = 6         #максимальное число пользователей + суперпользователь (https://postgrespro.ru/docs/postgresql/9.6/runtime-config-connection#guc-max-connections)

shared_buffers = 1000MB     #берём 25% от имеющегося объёма оперативной памяти (https://postgrespro.ru/docs/postgresql/9.6/runtime-config-resource#guc-shared-buffers)

temp_buffers = 8MB          #оставлю это значение по умолчанию (https://postgrespro.ru/docs/postgresql/9.6/runtime-config-resource#guc-temp-buffers)

work_mem = 200МБ            #я полагаю, что сценарий OLAP может требовать множество операций сортировки или хэширования в сложных запросах (https://postgrespro.ru/docs/postgresql/9.6/runtime-config-resource#guc-work-mem)

checkpoint_timeout = 5min   #оставляем это значение по умолчанию (https://postgrespro.ru/docs/postgrespro/9.5/runtime-config-wal#guc-checkpoint-timeout)

effective_cache_size = 2GB  #рекомендуется использовать 50% Ram, так как это всего лишь представление планировщика (https://postgrespro.ru/docs/postgresql/9.6/runtime-config-query#guc-effective-cache-size)

fsync = on                  #это даёт гарантию, что кластер баз данных сможет вернуться в согласованное состояние после сбоя оборудования или операционной системы (https://postgrespro.ru/docs/postgrespro/9.5/runtime-config-wal#guc-fsync)

commit_delay = 0            #оставлю этот параметр по умолчанию (https://postgrespro.ru/docs/postgrespro/9.5/runtime-config-wal#guc-commit-delay)

log_destination = csvlog    #включаем протоколирование в формате csv (https://postgrespro.ru/docs/postgresql/9.6/runtime-config-logging#guc-log-destination)
log_filename = '---.csv'    #меняем сразу расширение (https://postgrespro.ru/docs/postgresql/9.6/runtime-config-logging#guc-log-filename)

logging_collector = on      #включаем его, чтобы работало логирование (https://postgrespro.ru/docs/postgresql/9.6/runtime-config-logging#guc-logging-collector)
log_checkpoints = on        #включаем логирование контрольных точек (https://postgrespro.ru/docs/postgresql/9.6/runtime-config-logging#guc-logging-collector)

log_min_messages = info     #устанавливаем уровень сообщения логирование (https://postgrespro.ru/docs/postgresql/9.6/runtime-config-logging#guc-log-min-messages)
```

### Директория WAL файлов — поддиректория в PGDATA
Файлы конфигурации и файлы данных, используемые кластером базы данных, традиционно хранятся вместе в каталоге данных кластера, который обычно называют [PGDATA](https://postgrespro.ru/docs/postgresql/9.6/storage-file-layout)
Создаём поддиректорию для WAL файлов, перемещаем туда, а затем на прежнем месте создаём символьную ссылку.
```
mkdir wal_dir
cp -R pg_wal wal_dir
rm -r pg_wal/
ln -s /var/postgres/postgres0/u23/zn16/wal_dir/pg_wal/ pg_wal
```
# Запуск и настройка табличных пространств
`pg_ctl start -D ./`

подключение
psql -h pg118 -p 9077 -U postgres0 -d postgres

Удаление template0
UPDATE pg_database SET datistemplate='false' WHERE datname='template0';
DROP DATABASE template0;

Создаём табличное пространство
CREATE TABLESPACE tm0 LOCATION '/var/postgres/postgres0/u05/znp08';
CREATE DATABASE template0 WITH TEMPLATE = template1 TABLESPACE = tm0;

На основе template1 создать новую базу — theovermind4.
CREATE DATABASE theovermind4 WITH TEMPLATE = template1;

От имени новой роли (не администратора) произвести наполнение
существующих баз тестовыми наборами данных. Предоставить права по
необходимости. Табличные пространства должны использоваться по назначению.
CREATE ROLE studentt LOGIN;

GRANT CONNECT ON DATABASE theovermind4 TO studentt;

psql -h pg118 -p 9077 -U studentt -d theovermind4
