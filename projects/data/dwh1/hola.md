---
# On-premise Data warehousing 

```sql
USE master;
--verificar si ya está creada la db previamente
IF EXISTS (SELECT 1 FROM sys.databases where name = 'datawarehouse1')
BEGIN 
	ALTER DATABASE datawarehouse1 SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
	DROP DATABASE datawarehouse1;
END;
--creacion de db
CREATE DATABASE	datawarehouse1;
--cambio a db
USE datawarehouse1;
--creacion de schemas

SELECT DB_NAME();
CREATE SCHEMA db_bronze;
CREATE SCHEMA db_silver;
CREATE SCHEMA db_gold;
 
```
