
```sql
SELECT ('SQL Server 2022') AS Product,
       SERVERPROPERTY('ProductVersion') AS 'Product Version', 
       SERVERPROPERTY('ProductLevel') AS 'Product Level', 
       SERVERPROPERTY('Edition') AS 'SQL Server Edition', 
       SERVERPROPERTY('EngineEdition') AS 'Engine Edition',
       SERVERPROPERTY('ProductUpdateLevel') AS 'Update Level',
       SERVERPROPERTY('ProductUpdateReference') AS 'Update Reference';
```
