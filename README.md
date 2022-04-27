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
4. Стартуем кластер \
`pg_ctlcluster 14 main start` 
5. Подключаемся и создаем базу \
`psql` \
`create database mybase;` 
6. Переходим в созданную нами базу \
`\c mybase`

7. Создаем таблицу t1 \
`create table t1 as 
select 
  generate_series(1,10) as id,
  md5(random()::text)::char(10) as data;`
  
8. Создаем таблицу t2 \
`create table t2 as 
select 
  generate_series(1,10) as id,
  md5(random()::text)::char(10) as data;`
#### 9. Включаем wal_level для потоковой синхронизации (по дефолту установлена replica)
`ALTER SYSTEM SET wal_level = logical;`
### 10. Обязательно рестартуем кластер
`pg_ctlcluster 14 main restart`
#### 11. Создаем подписку на таблицу `t1`
`CREATE PUBLICATION t1_pub FOR TABLE t1;`

#### 12. Задаем пароль на подключение (123456) \
`\password`

