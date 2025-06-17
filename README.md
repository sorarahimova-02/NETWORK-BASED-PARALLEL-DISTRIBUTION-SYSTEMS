#NETWORK-BASED PARALLEL DISTRIBUTION SYSTEMS
This repository includes three practical SQL Server projects developed using SQL Server Management Studio (SSMS). Each project includes real-world SQL tasks such as query optimization, backup and recovery, and database security practices.

Project 1 – Performance Optimization and Monitoring
File: SQLQuery1.sql
Overview:
This project focuses on identifying slow-running queries, analyzing unused indexes, and creating helpful new indexes to improve overall system performance. Key steps include:

Top CPU-consuming queries:
SELECT TOP 10 
    total_worker_time / execution_count AS Avg_CPU_Time,
    execution_count,
    total_worker_time,
    SUBSTRING(qt.text, (qs.statement_start_offset/2)+1,
        ((CASE qs.statement_end_offset
          WHEN -1 THEN DATALENGTH(qt.text)
         ELSE qs.statement_end_offset END
         - qs.statement_start_offset)/2)+1)
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
ORDER BY Avg_CPU_Time DESC;

Long-running query analysis:
SELECT 
    OBJECT_NAME(i.object_id) AS TableName,
    i.name AS IndexName,
    i.index_id,
    s.user_seeks, s.user_scans, s.user_lookups, s.user_updates
FROM 
    sys.indexes AS i
LEFT JOIN 
    sys.dm_db_index_usage_stats AS s
    ON i.object_id = s.object_id AND i.index_id = s.index_id
WHERE 
    OBJECTPROPERTY(i.object_id,'IsUserTable') = 1
    AND i.name IS NOT NULL
ORDER BY 
    s.user_seeks + s.user_scans + s.user_lookups ASC;

Unused index detection
DROP INDEX ix_Colors_Archive ON Warehouse.Colors_Archive;

Index creation and deletion:

CREATE NONCLUSTERED INDEX IX_Orders_CustomerID_OrderDate
ON Sales.Orders (CustomerID, OrderDate);

DROP INDEX ix_Colors_Archive ON Warehouse.Colors_Archive;

Table-level disk usage inspection

🔹 Project 2 – Backup and Disaster Recovery Plan
📄 File: SQLQuery2.sql
🔍 Overview:
This project simulates critical database backup and restore operations including:

Full database backup:
2.1 Tam Yedek Alma'

BACKUP DATABASE WideWorldImporters
TO DISK = 'C:\Program Files\Microsoft SQL Server\MSSQL16.SQLEXPRESS\MSSQL\Backup\WWI_Full.bak'
WITH FORMAT, INIT, NAME = 'Tam Yedek';

Differential and transaction log backups:
--2.2 Fark Yedekleme

BACKUP DATABASE WideWorldImporters
TO DISK = 'C:\Program Files\Microsoft SQL Server\MSSQL16.SQLEXPRESS\MSSQL\Backup\WWI_Diff.bak'
WITH DIFFERENTIAL, INIT, NAME = 'Fark Yedek';



--2.3  Transaction Log Yedekleme

BACKUP LOG WideWorldImporters
TO DISK = 'C:\Program Files\Microsoft SQL Server\MSSQL16.SQLEXPRESS\MSSQL\Backup\WWI_Log.trn'
WITH INIT, NAME = 'Log Yedek';


Simulated table deletion and point-in-time recovery:
DROP TABLE FelaketTablosu;


-- Geri Yükleme (Restore)

USE master;
GO
ALTER DATABASE KurtarmaTestDB SET SINGLE_USER WITH ROLLBACK IMMEDIATE;

RESTORE DATABASE KurtarmaTestDB
FROM DISK = 'C:\Program Files\Microsoft SQL Server\MSSQL16.SQLEXPRESS\MSSQL\Backup\KurtarmaTestDB.bak'
WITH REPLACE;

ALTER DATABASE KurtarmaTestDB SET MULTI_USER;

-- Kontrol Et
SELECT * FROM KurtarmaTestDB.dbo.FelaketTablosu;

Scheduled automated backups

Verification of restore success with test databases:
ESTORE DATABASE KurtarmaTestDB_Test
FROM DISK = 'C:\Program Files\Microsoft SQL Server\MSSQL16.SQLEXPRESS\MSSQL\Backup\KurtarmaTestDB_Auto.bak'
WITH MOVE 'KurtarmaTestDB' TO 'C:\Program Files\Microsoft SQL Server\MSSQL16.SQLEXPRESS\MSSQL\DATA\KurtarmaTestDB_Test.mdf',
     MOVE 'KurtarmaTestDB_log' TO 'C:\Program Files\Microsoft SQL Server\MSSQL16.SQLEXPRESS\MSSQL\DATA\KurtarmaTestDB_Test.ldf',
     REPLACE;



🔹 Project 3 – Database Security and Access Control
📄 File: SQLQuery1 - Copy.sql
🔍 Overview:
This project demonstrates essential security techniques such as:
-- 1. SQL Server giriş hesabı oluştur
CREATE LOGIN guvenlik_user WITH PASSWORD = 'Guvenlik123!';

-- 2. Bu login'e bağlı kullanıcı oluştur
USE KurtarmaTestDB;
GO
CREATE USER guvenlik_user FOR LOGIN guvenlik_user;


-- 3. db_datareader rolüne ekle → sadece okuma izni verir
EXEC sp_addrolemember 'db_datareader', 'guvenlik_user'

SQL Server Authentication (login/user creation)

Transparent Data Encryption (TDE)

Column-level encryption (EncryptByPassPhrase)

SQL Injection simulation and prevention

Audit logging to track user activity
USE WideWorldImporters;
GO


CREATE DATABASE AUDIT SPECIFICATION DenetimSpec
FOR SERVER AUDIT KullaniciDenetimi
ADD (SELECT, INSERT, UPDATE, DELETE 
    ON Sales.Customers BY PUBLIC)
WITH (STATE = ON);



✅ Requirements
Microsoft SQL Server (2019 or later)

SQL Server Management Studio (SSMS)

Sufficient admin privileges to configure backups, security, and audit logs



