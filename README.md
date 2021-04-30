# mysql

настроить реплику

В материалах приложены ссылки на вагрант для репликации и дамп базы bet.dmp Базу развернуть на мастере и настроить так, чтобы реплицировались таблицы: | bookmaker | | competition | | market | | odds | | outcome

Настроить GTID репликацию x варианты которые принимаются к сдаче
рабочий вагрантафайл
скрины или логи SHOW TABLES
конфиги
пример в логе изменения строки и появления строки на реплике


В реpультате работ был подготволен vagrant файл, который запускает 2 вертуланые машины. и С помощью ansible конфигурируються до нужной нам схемы. 

# Проверка

На мастер сервере проверим server-id

```ruby
mysql> SELECT @@server_id;
+-------------+
| @@server_id |
+-------------+
|           1 |
+-------------+
1 row in set (0.00 sec)
```
 то же самое проверим на slave
 ```ruby
 mysql> SELECT @@server_id;
+-------------+
| @@server_id |
+-------------+
|           2 |
+-------------+
1 row in set (0.00 sec)
```
Убеждаемся что GTID включен:
```ruby
mysql> SHOW VARIABLES LIKE 'gtid_mode';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| gtid_mode     | ON    |
+---------------+-------+
1 row in set (0.00 sec)
```
проверим нашу базу зайдем на master
``` ruby
mysql> use bet
mysql> show tables;
+------------------+
| Tables_in_bet    |
+------------------+
| bookmaker        |
| competition      |
| events_on_demand |
| market           |
| odds             |
| outcome          |
| v_same_event     |
+------------------+
7 rows in set (0.00 sec)
```
```ruby
mysql> SELECT user,host FROM mysql.user where user='repl';
+------+------+
| user | host |
+------+------+
| repl | %    |
+------+------+
1 row in set (0.00 sec)
```
проверим нашу базу зайдем на slave
```ruby
[root@slave vagrant]# mysql

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| bet                |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> use bet
mysql> show tables
    -> ;
+---------------+
| Tables_in_bet |
+---------------+
| bookmaker     |
| competition   |
| market        |
| odds          |
| outcome       |
+---------------+
5 rows in set (0.00 sec) 
```
Вывод команды SHOW SLAVE STATUS\G на slave (здесь видно, что работает GTID-репликация):
```ruby
mysql> SHOW SLAVE STATUS\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.11.150
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 120467
               Relay_Log_File: slave-relay-bin.000002
                Relay_Log_Pos: 120680
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
            Replicate_Ignore_Table: bet.events_on_demand,bet.v_same_event
            Retrieved_Gtid_Set: 82881bcd-a9b2-11eb-a4ed-5254004d77d3:1-44
            Executed_Gtid_Set: 82881bcd-a9b2-11eb-a4ed-5254004d77d3:1-44
 ```
 
      
 
