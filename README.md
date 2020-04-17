# MySQL Trigger save query / MySQL Trigger salvar query
This repository shows how to create a table of logs in MySQL, saving all fields that were changed along with the executed query.<br>
Esse repositório mostra como  criar uma tabela de logs no MySQL , salvando todos os campos que foram alterados junto com a query executada.

## Getting Started / Vamos começar

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. <br>
Essas instruções fornecerão uma cópia do projeto em execução na sua máquina local para fins de desenvolvimento e teste.

### Prerequisites / Pré-requisitos

```
MySQL 5.1.29+
```

## Create tables / Criar tabelas. 

```sql

CREATE TABLE IF NOT EXISTS `mydb`.`user` (
  `id` INT(11) NOT NULL,
  `username` VARCHAR(16) NULL,
  `email` VARCHAR(255) NULL,
  `password` VARCHAR(32) NULL,
  `create_time` TIMESTAMP NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`));

CREATE TABLE IF NOT EXISTS `mydb`.`user_logs` (
  `id` INT(11) NOT NULL,
  `action` VARCHAR(45) NULL,
  `user_id` INT(11) NULL,
  `user_username` VARCHAR(16) NULL,
  `user_email` VARCHAR(255) NULL,
  `user_password` VARCHAR(32) NULL,
  `query` LONGTEXT NULL,
  `create_time` TIMESTAMP NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`));
```
![Image of BD](https://i.imgur.com/osVHLpa.png)<br>

Table users: Your table <br>
Table user_logs : Your table where all records of changes made to the tables are kept <b>user</b> <br>
<br>
Tabela users: Sua tabela <br>
Tabela user_logs : Sua tabela onde ficara guardado todos os registros de alteração feitas na tabelas <b>user</b> <br>

## Flush native tables from MySQL logs/  Liberar tabelas nativa de logs do MySQL
```sql
SET global general_log = 1;
SET global log_output = 'table';
```

## Create TRIGGER / Criando a TRIGGER:
### UPDATE
```sql
DELIMITER $$
DROP TRIGGER IF EXISTS `update_table_users`$$
CREATE TRIGGER `update_table_users` 
BEFORE UPDATE ON `user` 
FOR EACH ROW BEGIN 
SELECT argument INTO @tquery FROM mysql.general_log where thread_id = connection_id() and argument like 'UPDATE%' order by event_time desc limit 1;
INSERT INTO user_logs
SET  action = 'UPDATE',
     user_id = OLD.id,
     user_username = OLD.username,
     user_email = OLD.email,
     user_password = OLD.password,
     query = CONVERT( @tquery USING utf8 ),
     `create_time` = NOW();

END;
```

### INSERT
```sql
DELIMITER $$
DROP TRIGGER IF EXISTS `insert_table_users`$$
CREATE TRIGGER `insert_table_users` 
BEFORE INSERT ON `user` 
FOR EACH ROW BEGIN 
SELECT argument INTO @tquery FROM mysql.general_log where thread_id = connection_id() and argument like 'INSERT%' order by event_time desc limit 1;
INSERT INTO user_logs
SET  action = 'INSERT',
     user_id = OLD.id,
     user_username = OLD.username,
     user_email = OLD.email,
     user_password = OLD.password,
     query = CONVERT( @tquery USING utf8 ),
     `create_time` = NOW();

END;
```

### DELETE
```sql
DELIMITER $$
DROP TRIGGER IF EXISTS `delete_table_users`$$
CREATE TRIGGER `delete_table_users` 
BEFORE DELETE ON `user` 
FOR EACH ROW BEGIN 
SELECT argument INTO @tquery FROM mysql.general_log where thread_id = connection_id() and argument like 'DELETE%' order by event_time desc limit 1;
INSERT INTO user_logs
SET  action = 'DELETE',
     user_id = OLD.id,
     user_username = OLD.username,
     user_email = OLD.email,
     user_password = OLD.password,
     query = CONVERT( @tquery USING utf8 ),
     `create_time` = NOW();

END;
```

## Authors / Autor

* **Matteo Carminato** - *Initial work* - [CloudCrmm](http://cloudcrm.tech/)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
