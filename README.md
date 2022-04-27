# Postgres
- Репликация на уровне таблиц (потоковая) 
- физическая репликация (синхронизация кластера - горизонтальное маштабирование) 
- Backup

## Схема сборки проекта
![](https://github.com/vedoff/postgres/blob/main/pict/Screenshot%20from%202022-04-27%2017-27-26.png)

### Конфигурирование srv01
`vagrant up` \
Заходим на сервер \
`vagrant ssh srv01` \
`sudo su - postgres` \
Создаем кластер \
`pg_createcluster -d /var/lib/postgresql/14/main 14 main` \
Проверяем \
`pg_lsclusters` \
Стартуем кластер \
`pg_ctlcluster 14 main start` \
Подключаемся и создаем базу \
`psql` \
`create database mybase;`

#### Создаем таблицу t1

`create table t1 as 
select 
  generate_series(1,10) as id,
  md5(random()::text)::char(10) as data;`

#### Создаем таблицу test2

`create table t2 as 
select 
  generate_series(1,10) as id,
  md5(random()::text)::char(5) as data;`
