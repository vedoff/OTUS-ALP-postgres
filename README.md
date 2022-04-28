# Postgres
- Репликация на уровне таблиц (потоковая) 
- физическая репликация (синхронизация кластера - горизонтальное маштабирование) 
- Backup

## Схема сборки проекта
![](https://github.com/vedoff/postgres/blob/main/pict/Screenshot%20from%202022-04-27%2017-27-26.png)

### Конфигурирование srv01
`vagrant up` 
1. Заходим на сервер \
`vagrant ssh srv01` \
`sudo su - postgres` 
2. Создаем кластер \
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
sudo systemctl restart postgresql*

5. Стартуем кластер \
`pg_ctlcluster 14 main start`

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
`pg_ctlcluster 14 main restart`
#### 12. Создаем подписку на таблицу `t1`
`CREATE PUBLICATION t1_pub FOR TABLE t1;`

#### 13. Задаем пароль на подключение (123456)
`\password`

## Повторяем шаги с 1-11 и 13 для `srv02` в места пункта 12 для `t1` ->
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

#### Перейдем на сервер srv01 и выполним подписку на таблицу `t2` `srv02`
###  -> создадим подписку к БД по Порту с Юзером и Паролем и Копированием данных=false
`CREATE SUBSCRIPTION t2_sub_srv02
CONNECTION 'host=192.168.56.41 port=5432 user=postgres password=123456 dbname=mybase' 
PUBLICATION t2_pub WITH (copy_data = false);`

## Конфигурирование srv03 
#### Повторим настройку ноды как для srv01/srv02
Пункт 4 для srv03

4. Разрешаем доступ для синхронизации с определенных ip \
`vi /etc/postgresql/14/main/pg_hba.conf` 

`host    all    all    192.168.56.40/32   trust` \
`host    all    all    192.168.56.41/32   trust` \

Так же дописывам в конец \
`host    replication    postgres    192.168.56.43/32    md5` 

Разрешаем слушать postgres на внешнем ip (локальная сеть) \
`vi /etc/postgresql/14/main/postgresql.conf` 

`listen_addresses = 'localhost, 192.168.56.43'`

Перезапускаем службу postgresql \
Переходим под пользователя с правами `sudo` в нашем случае `vagrant` \
`ctrl d`
`sudo systemctl restart postgresql*`

###  -> создадим подписку на сервер `srv01`
`CREATE SUBSCRIPTION t1_sub_srv01_to_srv03
CONNECTION 'host=192.168.56.40 port=5432 user=postgres password=123456 dbname=mybase' 
PUBLICATION t1_pub WITH (copy_data = false);`

###  -> создадим подписку на сервер `srv02`
`CREATE SUBSCRIPTION t2_sub_srv02_to_srv03
CONNECTION 'host=192.168.56.41 port=5432 user=postgres password=123456 dbname=mybase' 
PUBLICATION t2_pub WITH (copy_data = false);`

## Конфигурирование сервера реплики backup





