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
     - На основе template1 создать новую базу — theovermind4

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
  - [max_connections](https://postgrespro.ru/docs/postgresql/9.6/runtime-config-connection#guc-max-connections)
  - [shared_buffers](https://postgrespro.ru/docs/postgresql/9.6/runtime-config-resource#guc-shared-buffers)
  - [temp_buffers](https://postgrespro.ru/docs/postgresql/9.6/runtime-config-resource#guc-temp-buffers)
  - [work_mem](https://postgrespro.ru/docs/postgresql/9.6/runtime-config-resource#guc-work-mem)
  - [checkpoint_timeout](https://postgrespro.ru/docs/postgrespro/9.5/runtime-config-wal#guc-checkpoint-timeout)
  - [effective_cache_size](https://postgrespro.ru/docs/postgresql/9.6/runtime-config-query#guc-effective-cache-size)
  - [fsync](https://postgrespro.ru/docs/postgrespro/9.5/runtime-config-wal#guc-fsync)
  - [commit_delay](https://postgrespro.ru/docs/postgrespro/9.5/runtime-config-wal#guc-commit-delay)

Параметры должны быть подобраны в соответствии со сценарием **OLAP** : 5 пользователей, пакетная запись данных в среднем по 30 МБ. Начнём по порядку:
```
max_connections = 6         #максимальное число пользователей + суперпользователь

shared_buffers = 1000MB     #берём 25% от имеющегося объёма оперативной памяти

temp_buffers = 8MB          #оставлю это значение по умолчанию

work_mem = 200МБ            #я полагаю, что сценарий OLAP может требовать множество операций сортировки или хэширования в сложных запросах

checkpoint_timeout = 5min   #оставляем это значение по умолчанию

effective_cache_size = 2GB  #рекомендуется использовать 50% Ram, так как это всего лишь представление планировщика

fsync = on                  #это даёт гарантию, что кластер баз данных сможет вернуться в согласованное состояние после сбоя оборудования или операционной системы

commit_delay = 0            #оставлю этот параметр по умолчанию
```
Также нам нужно настроить формат лог-файлов, уровень сообщения лога и логирование контрольных точек. Для этого достаточно изменить следующие параметры:
- [log_destination](https://postgrespro.ru/docs/postgresql/9.6/runtime-config-logging#guc-log-destination)
- [log_filename](https://postgrespro.ru/docs/postgresql/9.6/runtime-config-logging#guc-log-filename)
- [logging_collector](https://postgrespro.ru/docs/postgresql/9.6/runtime-config-logging#guc-logging-collector)
- [log_checkpoints](https://postgrespro.ru/docs/postgresql/9.6/runtime-config-logging#guc-logging-collector)
- [log_min_messages](https://postgrespro.ru/docs/postgresql/9.6/runtime-config-logging#guc-log-min-messages)

```
log_destination = csvlog    #включаем протоколирование в формате csv

log_filename = '---.csv'      #меняем сразу расширение

logging_collector = on      #включаем его, чтобы работало логирование

log_checkpoints = on        #включаем логирование контрольных точек

log_min_messages = info     #устанавливаем уровень сообщения логирования
```

### Директория WAL файлов — поддиректория в PGDATA
Файлы конфигурации и файлы данных, используемые кластером базы данных, традиционно хранятся вместе в каталоге данных кластера, который обычно называют [PGDATA](https://postgrespro.ru/docs/postgresql/9.6/storage-file-layout).
Создаём поддиректорию для WAL файлов, перемещаем туда, а затем на прежнем месте создаём символьную ссылку.

*Прошу обратить внимание, что директория pg_wal и так является поддиректорией в PGDATA по умолчанию, поэтому в данном случае ничего не нужно было делать, однако, в качестве примера, я привёл создание ещё одной директории, в PGDATA, скопировал pg_wal туда, а затем создал символьную ссылку на прежнем месте.*
```
mkdir wal_dir
cp -R pg_wal wal_dir
rm -r pg_wal/
ln -s /var/postgres/postgres0/u23/zn16/wal_dir/pg_wal/ pg_wal
```
# Запуск и настройка табличных пространств
**pg_ctl** — это утилита для начальной инициализации, запуска, остановки, повторного запуска и управления кластером баз данных PostgreSQL. Подробнее об этом [тут](https://postgrespro.ru/docs/postgresql/9.6/app-pg-ctl).

`pg_ctl start -D ./`

**После этого подключаемся к БД с помощью [psql](https://postgrespro.ru/docs/postgresql/9.6/app-psql).**

### Удаление template0
По умолчанию удалить template0 нельзя. Однако в таблице pg_database есть два полезных флага для каждой базы данных: столбцы *datistemplate* и *datallowconn*. В данном случае нам нужен только *datistemplate*. datistemplate указывает на факт того, что база данных может выступать в качестве шаблона в команде CREATE DATABASE. Если сбросить этот флаг (установить false), то у нас появится возможность удалить эту базу данных. Сделать это можно при помощи UPDATE:
```
UPDATE pg_database SET datistemplate='false' WHERE datname='template0';
DROP DATABASE template0;
```
### Создаём табличное пространство
Табличные пространства в Postgres позволяют администраторам организовать логику размещения файлов объектов базы данных в файловой системе. Подробнее читайте [тут](https://postgrespro.ru/docs/postgrespro/9.5/manage-ag-tablespaces).

Создадим табличное пространство, а затем сразу же пересоздадим в нём template0 из template1, а также укажем табличное пространство по умолчанию:
```
CREATE TABLESPACE tm0 LOCATION '/var/postgres/postgres0/u05/znp08';
CREATE DATABASE template0 WITH TEMPLATE = template1 TABLESPACE = tm0;
```

### Создание роли
В этом пункте просто обращу внимание, что если вы хотите управлять базой данных, в котрой настроено другое табличное пространство, то вам необходимо добавлять права для этого табличного пространства. Подробнее об этом читайте [тут](https://postgrespro.ru/docs/postgresql/9.5/sql-grant).

*Пример создания роли:*
```
CREATE ROLE student LOGIN;
GRANT CONNECT ON DATABASE nameDB TO studentt;
```
