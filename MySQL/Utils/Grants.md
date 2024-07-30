[<-- Back to Utils](https://github.com/mtemporim/Databases/tree/main/MySQL/Utils)


### Some queries that i use to grants

### It generate commands to grant DML and DDL for all schemas on instance except system databases
```sql
SELECT CONCAT('GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, ALTER ON `', schema_name, '`.* TO \'test\'@\'%\';') AS GrantCommand
FROM information_schema.schemata
WHERE schema_name NOT IN ('sys', 'performance_schema', 'mysql', 'information_schema');
```