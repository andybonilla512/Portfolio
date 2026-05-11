
---
# Bronze layer
## Overview

## Create database and schemas
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

## Create tables
```sql

USE datawarehouse1;
-- Crear schema si no existe
IF NOT EXISTS (
    SELECT * FROM sys.schemas WHERE name = 'db_bronze'
)
BEGIN
    EXEC('CREATE SCHEMA db_bronze');
END;
/*
 * 
 */
-- 
IF OBJECT_ID('db_bronze.crm_cust_info', 'U') IS NULL
BEGIN
	CREATE TABLE db_bronze.crm_cust_info (
		cst_id int,
		cst_key nvarchar(50),
		cst_firstname nvarchar(50),
		cst_lastname nvarchar(50),
		cst_marital_status nvarchar(50),
		cst_gndr nvarchar(50),
		cst_create_date date
	);
END;
--
IF OBJECT_ID('db_bronze.crm_prd_info', 'U') IS NULL
BEGIN
	CREATE TABLE db_bronze.crm_prd_info (
		prd_id int ,
		prd_key nvarchar(50),
		prd_nm nvarchar(50),
		prd_cost float,
		prd_line nvarchar(50),
		prd_start_dt date,
		prd_end_dt date
	);
END;
--
--DROP TABLE db_bronze.crm_sales_details;
IF OBJECT_ID('db_bronze.crm_sales_details', 'U') IS NULL
BEGIN
	CREATE TABLE db_bronze.crm_sales_details  (
		sls_ord_num nvarchar(50),
		sls_prd_key nvarchar(50),
		sls_cust_id int,
		sls_order_dt INT,
		sls_ship_dt INT,
		sls_due_dt INT,
		sls_sales float,
		sls_quantity int,
		sls_price float
	);
END;
--
IF OBJECT_ID('db_bronze.erp_cust_az12', 'U') IS NULL
BEGIN
	CREATE TABLE db_bronze.erp_cust_az12 (
		CID nvarchar(50),
		BDATE date,
		GEN nvarchar(50)
	);
END;
--
IF OBJECT_ID('db_bronze.erp_loc_a101', 'U') IS NULL
BEGIN
	CREATE TABLE db_bronze.erp_loc_a101 (
		CID nvarchar(50),
		CNTRY nvarchar(50)
	);
END;
--
IF OBJECT_ID('db_bronze.erp_px_cat_g1v2', 'U') IS NULL
BEGIN
	CREATE TABLE db_bronze.erp_px_cat_g1v2 (
		ID nvarchar(50),
		CAT nvarchar(50),
		SUBCAT nvarchar(50),
		MAINTENANCE nvarchar(50)
	);
END;
```


## Insert into tables
```sql
USE datawarehouse1;
/*==============================================================
    CREATE AUDIT SCHEMA
==============================================================*/
IF NOT EXISTS (
    SELECT *
    FROM sys.schemas
    WHERE name = 'db_audit'
)
BEGIN
    EXEC('CREATE SCHEMA db_audit');
END;
/*==============================================================
    CREATE LOG TABLE
==============================================================*/
IF OBJECT_ID('db_audit.etl_logs', 'U') IS NULL
BEGIN
    CREATE TABLE db_audit.etl_logs (
        log_id INT IDENTITY(1,1) PRIMARY KEY,
        process_name NVARCHAR(100),
        table_name NVARCHAR(100),
        load_start DATETIME,
        load_end DATETIME,
        duration_seconds INT,
        rows_inserted INT,
        status NVARCHAR(20),
        error_message NVARCHAR(MAX)
    );
END;
/*==============================================================
    CREATE OR ALTER PROCEDURE
==============================================================*/
CREATE OR ALTER PROCEDURE db_bronze.load_bronze
AS
BEGIN
    SET NOCOUNT ON;
    DECLARE @start_t_time DATETIME;
    DECLARE @end_t_time DATETIME;
	DECLARE @start_time DATETIME;
    DECLARE @end_time DATETIME;
    DECLARE @rows_inserted INT;
    PRINT '========================================';
    PRINT 'Loading Bronze Layer';
    PRINT '========================================';
	SET @start_t_time = GETDATE();
    /*==========================================================
        LOAD CRM - CUSTOMER INFO
    ==========================================================*/
    BEGIN TRY
        PRINT 'Loading crm_cust_info';
        SET @start_time = GETDATE();
        TRUNCATE TABLE db_bronze.crm_cust_info;
        BULK INSERT db_bronze.crm_cust_info
        FROM '/data/source_crm/cust_info.csv'
        WITH (
            FIRSTROW = 2,
            FIELDTERMINATOR = ',',
            TABLOCK
        );
        SET @rows_inserted = @@ROWCOUNT;
        SET @end_time = GETDATE();
        INSERT INTO db_audit.etl_logs (
            process_name,
            table_name,
            load_start,
            load_end,
            duration_seconds,
            rows_inserted,
            status,
            error_message
        )
        VALUES (
            'load_bronze',
            'crm_cust_info',
            @start_time,
            @end_time,
            DATEDIFF(SECOND, @start_time, @end_time),
            @rows_inserted,
            'SUCCESS',
            NULL
        );
        PRINT CAST(@rows_inserted AS VARCHAR) + ' rows inserted into crm_cust_info';
		PRINT 'Load duration: ' + CAST(DATEDIFF(SECOND, @start_time, @end_time) AS VARCHAR) + ' seconds';
    END TRY
    BEGIN CATCH
        INSERT INTO db_audit.etl_logs (
            process_name,
            table_name,
            load_start,
            load_end,
            duration_seconds,
            rows_inserted,
            status,
            error_message
        )
        VALUES (
            'load_bronze',
            'crm_cust_info',
            @start_time,
            GETDATE(),
            DATEDIFF(SECOND, @start_time, GETDATE()),
            0,
            'FAILED',
            ERROR_MESSAGE()
        );
        PRINT 'ERROR LOADING crm_cust_info';
        PRINT ERROR_MESSAGE();
    END CATCH;
    /*==========================================================
        LOAD CRM - PRODUCT INFO
    ==========================================================*/
    BEGIN TRY
        PRINT 'Loading crm_prd_info';
        SET @start_time = GETDATE();
        TRUNCATE TABLE db_bronze.crm_prd_info;
        BULK INSERT db_bronze.crm_prd_info
        FROM '/data/source_crm/prd_info.csv'
        WITH (
            FIRSTROW = 2,
            FIELDTERMINATOR = ',',
            TABLOCK
        );
        SET @rows_inserted = @@ROWCOUNT;
        SET @end_time = GETDATE();
        INSERT INTO db_audit.etl_logs (
            process_name,
            table_name,
            load_start,
            load_end,
            duration_seconds,
            rows_inserted,
            status,
            error_message
        )
        VALUES (
            'load_bronze',
            'crm_prd_info',
            @start_time,
            @end_time,
            DATEDIFF(SECOND, @start_time, @end_time),
            @rows_inserted,
            'SUCCESS',
            NULL
        );
        PRINT CAST(@rows_inserted AS VARCHAR) + ' rows inserted into crm_prd_info';
		PRINT 'Load duration: ' + CAST(DATEDIFF(SECOND, @start_time, @end_time) AS VARCHAR) + ' seconds';
    END TRY
    BEGIN CATCH
        INSERT INTO db_audit.etl_logs (
            process_name,
            table_name,
            load_start,
            load_end,
            duration_seconds,
            rows_inserted,
            status,
            error_message
        )
        VALUES (
            'load_bronze',
            'crm_prd_info',
            @start_time,
            GETDATE(),
            DATEDIFF(SECOND, @start_time, GETDATE()),
            0,
            'FAILED',
            ERROR_MESSAGE()
        );
        PRINT 'ERROR LOADING crm_prd_info';
        PRINT ERROR_MESSAGE();
    END CATCH;
    /*==========================================================
        LOAD CRM - SALES DETAILS
    ==========================================================*/
    BEGIN TRY
        PRINT 'Loading crm_sales_details';
        SET @start_time = GETDATE();
        TRUNCATE TABLE db_bronze.crm_sales_details;
        BULK INSERT db_bronze.crm_sales_details
        FROM '/data/source_crm/sales_details.csv'
        WITH (
            FIRSTROW = 2,
            FIELDTERMINATOR = ',',
            TABLOCK
        );
        SET @rows_inserted = @@ROWCOUNT;
        SET @end_time = GETDATE();
        INSERT INTO db_audit.etl_logs (
            process_name,
            table_name,
            load_start,
            load_end,
            duration_seconds,
            rows_inserted,
            status,
            error_message
        )
        VALUES (
            'load_bronze',
            'crm_sales_details',
            @start_time,
            @end_time,
            DATEDIFF(SECOND, @start_time, @end_time),
            @rows_inserted,
            'SUCCESS',
            NULL
        );
        PRINT CAST(@rows_inserted AS VARCHAR) + ' rows inserted into crm_sales_details';
		PRINT 'Load duration: ' + CAST(DATEDIFF(SECOND, @start_time, @end_time) AS VARCHAR) + ' seconds';
    END TRY
    BEGIN CATCH
        INSERT INTO db_audit.etl_logs (
            process_name,
            table_name,
            load_start,
            load_end,
            duration_seconds,
            rows_inserted,
            status,
            error_message
        )
        VALUES (
            'load_bronze',
            'crm_sales_details',
            @start_time,
            GETDATE(),
            DATEDIFF(SECOND, @start_time, GETDATE()),
            0,
            'FAILED',
            ERROR_MESSAGE()
        );
        PRINT 'ERROR LOADING crm_sales_details';
        PRINT ERROR_MESSAGE();
    END CATCH;
    /*==========================================================
        LOAD ERP - CUSTOMER
    ==========================================================*/
    BEGIN TRY
        PRINT 'Loading erp_cust_az12';
        SET @start_time = GETDATE();
        TRUNCATE TABLE db_bronze.erp_cust_az12;
        BULK INSERT db_bronze.erp_cust_az12
        FROM '/data/source_erp/cust_az12.csv'
        WITH (
            FIRSTROW = 2,
            FIELDTERMINATOR = ',',
            TABLOCK
        );
        SET @rows_inserted = @@ROWCOUNT;
        SET @end_time = GETDATE();
        INSERT INTO db_audit.etl_logs (
            process_name,
            table_name,
            load_start,
            load_end,
            duration_seconds,
            rows_inserted,
            status,
            error_message
        )
        VALUES (
            'load_bronze',
            'erp_cust_az12',
            @start_time,
            @end_time,
            DATEDIFF(SECOND, @start_time, @end_time),
            @rows_inserted,
            'SUCCESS',
            NULL
        );
        PRINT CAST(@rows_inserted AS VARCHAR) + ' rows inserted into erp_cust_az12';
		PRINT 'Load duration: ' + CAST(DATEDIFF(SECOND, @start_time, @end_time) AS VARCHAR) + ' seconds';
    END TRY
    BEGIN CATCH
        INSERT INTO db_audit.etl_logs (
            process_name,
            table_name,
            load_start,
            load_end,
            duration_seconds,
            rows_inserted,
            status,
            error_message
        )
        VALUES (
            'load_bronze',
            'erp_cust_az12',
            @start_time,
            GETDATE(),
            DATEDIFF(SECOND, @start_time, GETDATE()),
            0,
            'FAILED',
            ERROR_MESSAGE()
        );
        PRINT 'ERROR LOADING erp_cust_az12';
        PRINT ERROR_MESSAGE();
    END CATCH;
    /*==========================================================
        LOAD ERP - LOCATION
    ==========================================================*/
    BEGIN TRY
        PRINT 'Loading erp_loc_a101';
        SET @start_time = GETDATE();
        TRUNCATE TABLE db_bronze.erp_loc_a101;
        BULK INSERT db_bronze.erp_loc_a101
        FROM '/data/source_erp/loc_a101.csv'
        WITH (
            FIRSTROW = 2,
            FIELDTERMINATOR = ',',
            TABLOCK
        );
        SET @rows_inserted = @@ROWCOUNT;
        SET @end_time = GETDATE();
        INSERT INTO db_audit.etl_logs (
            process_name,
            table_name,
            load_start,
            load_end,
            duration_seconds,
            rows_inserted,
            status,
            error_message
        )
        VALUES (
            'load_bronze',
            'erp_loc_a101',
            @start_time,
            @end_time,
            DATEDIFF(SECOND, @start_time, @end_time),
            @rows_inserted,
            'SUCCESS',
            NULL
        );
        PRINT CAST(@rows_inserted AS VARCHAR) + ' rows inserted into erp_loc_a101';
		PRINT 'Load duration: ' + CAST(DATEDIFF(SECOND, @start_time, @end_time) AS VARCHAR) + ' seconds';
    END TRY
    BEGIN CATCH
        INSERT INTO db_audit.etl_logs (
            process_name,
            table_name,
            load_start,
            load_end,
            duration_seconds,
            rows_inserted,
            status,
            error_message
        )
        VALUES (
            'load_bronze',
            'erp_loc_a101',
            @start_time,
            GETDATE(),
            DATEDIFF(SECOND, @start_time, GETDATE()),
            0,
            'FAILED',
            ERROR_MESSAGE()
        );
        PRINT 'ERROR LOADING erp_loc_a101';
        PRINT ERROR_MESSAGE();
    END CATCH;
    /*==========================================================
        LOAD ERP - PRODUCT CATERY
    ==========================================================*/
    BEGIN TRY
        PRINT 'Loading erp_px_cat_g1v2';
        SET @start_time = GETDATE();
        TRUNCATE TABLE db_bronze.erp_px_cat_g1v2;
        BULK INSERT db_bronze.erp_px_cat_g1v2
        FROM '/data/source_erp/px_cat_g1v2.csv'
        WITH (
            FIRSTROW = 2,
            FIELDTERMINATOR = ',',
            TABLOCK
        );
        SET @rows_inserted = @@ROWCOUNT;
        SET @end_time = GETDATE();
        INSERT INTO db_audit.etl_logs (
            process_name,
            table_name,
            load_start,
            load_end,
            duration_seconds,
            rows_inserted,
            status,
            error_message
        )
        VALUES (
            'load_bronze',
            'erp_px_cat_g1v2',
            @start_time,
            @end_time,
            DATEDIFF(SECOND, @start_time, @end_time),
            @rows_inserted,
            'SUCCESS',
            NULL
        );
        PRINT CAST(@rows_inserted AS VARCHAR) + ' rows inserted into erp_px_cat_g1v2';
		PRINT 'Load duration: ' + CAST(DATEDIFF(SECOND, @start_time, @end_time) AS VARCHAR) + ' seconds';
    END TRY
    BEGIN CATCH
        INSERT INTO db_audit.etl_logs (
            process_name,
            table_name,
            load_start,
            load_end,
            duration_seconds,
            rows_inserted,
            status,
            error_message
        )
        VALUES (
            'load_bronze',
            'erp_px_cat_g1v2',
            @start_time,
            GETDATE(),
            DATEDIFF(SECOND, @start_time, GETDATE()),
            0,
            'FAILED',
            ERROR_MESSAGE()
        );
        PRINT 'ERROR LOADING erp_px_cat_g1v2';
        PRINT ERROR_MESSAGE();
    END CATCH;
    SET @end_t_time = GETDATE();
    PRINT '========================================';
    PRINT 'Bronze Layer Load Finished';
	PRINT 'ETL start date: ' + CAST(@start_t_time AS VARCHAR)
	PRINT 'ETL end date: ' + CAST(@end_t_time AS VARCHAR)
	PRINT 'Full ETL duration: ' + CAST(DATEDIFF(SECOND, @start_t_time, @end_t_time) AS VARCHAR) + ' seconds';
    PRINT '========================================';
END;
/*==============================================================
    EXECUTE PROCEDURE
==============================================================*/
EXEC db_bronze.load_bronze;
/*==============================================================
    VALIDATE LOGS
==============================================================*/
SELECT *
FROM db_audit.etl_logs
ORDER BY log_id DESC;

```
