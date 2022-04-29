# Postgres
- Репликация на уровне таблиц (потоковая) 
- физическая репликация (синхронизация кластера - горизонтальное маштабирование) 
- Backup

## Схема сборки проекта
![](https://github.com/vedoff/postgres/blob/main/pict/Screenshot%20from%202022-04-29%2013-15-25.png)

### Установка postgres на ноды.
`vagrant up && ansible-playbook play.yml`

# Конфигурирование srv01
1. Заходим на сервер \
`vagrant ssh srv01` \
`sudo su - postgres` 
2. Создаем кластер если он не создан при установке\
`pg_createcluster -d /var/lib/postgresql/14/main 14 main` 
3. Проверяем \
`pg_lsclusters` 

4. Разрешаем доступ для синхронизации с определенных ip \
`vi /etc/postgresql/14/main/pg_hba.conf` 

`host    all    all    192.168.56.41/32   trust` \
`host    all    all    192.168.56.42/32   trust` 

Разрешаем слушать postgres на внешнем ip (локальная сеть) \
`vi /etc/postgresql/14/main/postgresql.conf` 

`listen_addresses = 'localhost, 192.168.56.40'`

Перезапускаем службу postgresql \
`ctrl d` \
`sudo systemctl restart postgresql*`

Проверяем что postgres слушает на локальном адресе и указано порту (5432) \
`ss -tunlp`

5. переходим обратно в кластер \
`sudo su - postgres`

6. Подключаемся и создаем базу \
`psql` \
`create database mybase;`

7. Переходим в созданную нами базу \
`\c mybase`

8. Создаем таблицу `t1` \
`create table t1 as 
select 
  generate_series(1,10) as id,
  md5(random()::text)::char(10) as data;`
  
9. Создаем таблицу `t2` \
`create table t2 as 
select 
  generate_series(1,10) as id,
  md5(random()::text)::char(10) as data;`
#### 10. Включаем wal_level для потоковой синхронизации (по дефолту установлена replica)
`ALTER SYSTEM SET wal_level = logical;`
### 11. Обязательно рестартуем кластер
`ctrl d` \
`sudo pg_ctlcluster 14 main restart`
#### 12. Создаем публикацию на таблицу `t1`
`sudo su - postgres` \
`psql` \
`\c mybase` \
`CREATE PUBLICATION t1_pub FOR TABLE t1;`

#### 13. Задаем пароль на подключение (123456)
`\password`

# Конфигурирование srv02
## Повторяем шаги с 1-3 5-11 и 13 для `srv02` в места пункта 12 для `t1` ->
###  -> создадим подписку к БД по Порту с Юзером и Паролем и Копированием данных=false
`CREATE SUBSCRIPTION t1_sub_srv01
CONNECTION 'host=192.168.56.40 port=5432 user=postgres password=123456 dbname=mybase' 
PUBLICATION t1_pub WITH (copy_data = false);`

## Создадим публикацию для `srv02` `t2` и подписку на эту таблицу на сервере srv01 
Заходим на сервер \
`vagrant ssh srv02` \
`sudo su - postgres` \
`\c mybase` \
`CREATE PUBLICATION t2_pub FOR TABLE t2;`

Разрешаем доступ для синхронизации с определенных ip \
`vi /etc/postgresql/14/main/pg_hba.conf` 

`host    all    all    192.168.56.40/32   trust` \
`host    all    all    192.168.56.42/32   trust` 

Разрешаем слушать postgres на внешнем ip (локальная сеть) \
`vi /etc/postgresql/14/main/postgresql.conf` 

`listen_addresses = 'localhost, 192.168.56.41'`

Перезапускаем службу postgresql \
Переходим под пользователя с правами `sudo` в нашем случае `vagrant` \
`ctrl d` \
`sudo systemctl restart postgresql*`

## Перейдем на сервер srv01 и выполним подписку на таблицу `t2` `srv02`
###  -> создадим подписку к БД по Порту с Юзером и Паролем и Копированием данных=false
`CREATE SUBSCRIPTION t2_sub_srv02
CONNECTION 'host=192.168.56.41 port=5432 user=postgres password=123456 dbname=mybase' 
PUBLICATION t2_pub WITH (copy_data = false);`

#### Проверим подписку
SELECT * FROM pg_stat_subscription \gx

![](https://github.com/vedoff/postgres/blob/main/pict/Screenshot%20from%202022-04-29%2013-33-54.png)
![](https://github.com/vedoff/postgres/blob/main/pict/Screenshot%20from%202022-04-29%2013-34-49.png)


# Конфигурирование srv03 
#### Повторим настройку ноды как для srv01/srv02
Пункт 4 для srv03

4. Разрешаем доступ для синхронизации с определенных ip \
`vi /etc/postgresql/14/main/pg_hba.conf` 

`host    all    all    192.168.56.40/32   trust` \
`host    all    all    192.168.56.41/32   trust` 

Так же дописывам в конец \
`host    replication    postgres    192.168.56.43/32    md5` 

Разрешаем слушать postgres на внешнем ip (локальная сеть) \
`vi /etc/postgresql/14/main/postgresql.conf` 

`listen_addresses = 'localhost, 192.168.56.42'`

Перезапускаем службу postgresql \
Переходим под пользователя с правами `sudo` в нашем случае `vagrant` \
`ctrl d` \
`sudo systemctl restart postgresql*`

###  -> создадим подписку на сервер `srv01`
`CREATE SUBSCRIPTION t1_sub_srv01_to_srv03
CONNECTION 'host=192.168.56.40 port=5432 user=postgres password=123456 dbname=mybase' 
PUBLICATION t1_pub WITH (copy_data = false);`

###  -> создадим подписку на сервер `srv02`
`CREATE SUBSCRIPTION t2_sub_srv02_to_srv03
CONNECTION 'host=192.168.56.41 port=5432 user=postgres password=123456 dbname=mybase' 
PUBLICATION t2_pub WITH (copy_data = false);`
### Результат
![](https://github.com/vedoff/postgres/blob/main/pict/Screenshot%20from%202022-04-29%2013-19-43.png)

# Конфигурирование сервера реплики backup
Разрешаем доступ для синхронизации с определенных ip \
`vi /etc/postgresql/14/main/pg_hba.conf` 

`host    all    all    192.168.56.42/32   md5` 

Разрешаем слушать postgres на внешнем ip (локальная сеть) \
`vi /etc/postgresql/14/main/postgresql.conf` 

`listen_addresses = 'localhost, 192.168.56.43'`

Удаляем данные из кластера, таким образом подготавливаем его к синхронизации \
`rm -rf /var/lib/postgresql/14/main`

Восстанавливаем папку базы `main` \
`cd /var/lib/postgresql/14/` \
`mkdir main` 

Обязательно высталяем права на каталог кластера, если будут выставлены права отличные от 700 или 750 то синхронизация не пройдет. \
Будет выдана ошибка с рекомендациями о выставлении прав на каталог кластера баз данных. \
![](https://github.com/vedoff/postgres/blob/main/pict/Screenshot%20from%202022-04-29%2013-11-50.png)

## `chmod go-rwx main` 

### Запускаем синхронизацию
`pg_basebackup -P -R -X stream -c fast -h 192.168.56.42 -D main`

В этой команде есть важный параметр -R. Он означает, что PostgreSQL-сервер также создаст пустой файл standby.signal. \
Несмотря на то, что файл пустой, само наличие этого файла означает, что этот сервер — реплика.
### Результат
![](https://github.com/vedoff/postgres/blob/main/pict/Screenshot%20from%202022-04-29%2013-16-19.png)


