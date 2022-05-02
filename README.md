# database-configuration
This repository describes how to configure the database

#Инициализация кластера
initdb --pgdata=$HOME/u23/zn16 --encoding=UTF-8 --locale=en_US --username=postgres0

![Вывод команды](/images/picture-1.png)

#Конфигурация и запуск сервер БД
##Откроем файл postgresql.conf и установим порт
![Символ "#" нужно убрать](/images/picture-2.png)

Переходим в файл pg_hba.conf
В этом файле настраиваем новый конфиг host, который будет отвечать за TCP/IP подключение с методом аутентификации ident.

![файл pg_hba.conf](/images/picture-3.png)
