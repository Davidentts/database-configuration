# database-configuration
Инициализация кластера БД
     • Имя узла — pg118.
     • Имя пользователя — postgres0.
     • Директория кластера БД — $HOME/u23/zn16.
     • Кодировка, локаль — UTF8, английская
     • Перечисленные параметры задать через аргументы команды.
Конфигурация и запуск сервера БД
     • Способ подключения к БД — TCP/IP socket, номер порта 9077.
     • Остальные способы подключений запретить.
     • Способ аутентификации клиентов — по имени пользователя.
     • Настроить следующие параметры сервера БД: max_connections,
shared_buffers, temp_buffers, work_mem, checkpoint_timeout,
effective_cache_size, fsync, commit_delay. Параметры должны быть
подобраны в соответствии со сценарием OLAP: 5 пользователей, пакетная
запись данных в среднем по 30 МБ.
     • Директория WAL файлов — поддиректория в PGDATA.
     • Формат лог-файлов — csv.
     • Уровень сообщений лога — INFO.
     • Дополнительно логировать — контрольные точки.
Дополнительные табличные пространства и наполнение
     • Пересоздать шаблон template0 в новом табличном пространстве:
         ◦ $HOME/u05/znp08.
     • На основе template1 создать новую базу — theovermind4.
     • От имени новой роли (не администратора) произвести наполнение
существующих баз тестовыми наборами данных. Предоставить права по
необходимости. Табличные пространства должны использоваться по назначению.
     • Вывести список всех табличных пространств кластера и содержащиеся
в них объекты.

# Инициализация кластера
initdb --pgdata=$HOME/u23/zn16 --encoding=UTF-8 --locale=en_US --username=postgres0

![Вывод команды](/images/picture-1.png)

# Конфигурация и запуск сервер БД
## Откроем файл postgresql.conf и установим порт
![Символ "#" нужно убрать](/images/picture-2.png)

Далее переходим в файл pg_hba.conf
В этом файле настраиваем новый конфиг host, который будет отвечать за TCP/IP подключение с методом аутентификации ident:

![файл pg_hba.conf](/images/picture-3.png)

#Настройка параметров сервера
Теперь нам нужно настроить следуюущие параметры сервера БД: max_connections, shared_buffers, temp_buffers, work_mem, checkpoint_timeout, effective_cache_size, fsync, commit_delay. Параметры должны быть подобраны в соответствии со сценарием OLAP: 5 пользователей, пакетная запись данных в среднем по 30 МБ. Начнём по порядку:

max_connections = 5         #тут понятно из задания

shared_buffers = 1000MB     #берём 25% от имеющегося объёма оперативной памяти https://postgrespro.ru/docs/postgresql/9.6/runtime-config-resource#guc-shared-buffers

temp_buffers = 8MB          #Оставлю это значение по умолчанию

work_mem = 200МБ            #я полагаю, что сценарий OLAP может требовать множество операций сортировки или хэширования в сложных запросах

checkpoint_timeout = 5min   #оставляем это значение по умолчанию

effective_cache_size = 2GB  #рекомендуется испольщовать 50% Ram

fsync = on                  #Это даёт гарантию, что кластер баз данных сможет вернуться в согласованное состояние после сбоя оборудования или операционной системы

commit_delay = 0

log_destination = csvlog    #Включаем протоколирование в формате csv
log_filename = '---.csv'    #Меняем сразу расширение

logging_collector = on      #Включаем его, чтобы работало логирование
log_checkpoints = on        #Включаем логирование контрольных точек

log_min_messages = info     #Устанавливаем уровень сообщения логирование

### Директория WAL файлов — поддиректория в PGDATA
Файлы конфигурации и файлы данных, используемые кластером базы данных, традиционно хранятся вместе в каталоге данных кластера, который обычно называют PGDATA (https://postgrespro.ru/docs/postgresql/9.6/storage-file-layout)
Создаём поддиректорию для WAL файлов, перемещаем туда, а затем на прежнем месте создаём символьную ссылку.

mkdir wal_dir
cp -R pg_wal wal_dir
rm -r pg_wal/
ln -s /var/postgres/postgres0/u23/zn16/wal_dir/pg_wal/ pg_wal

#Запуск
pg_ctl start -D ./

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
