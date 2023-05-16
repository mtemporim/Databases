[<-- Back to Dump](https://github.com/mtemporim/Databases/tree/main/MySQL/MySQL-Shell/Dump)

[util.dumpSchemas documentation](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-utilities-dump-instance-schema.html)

### util.dumpSchemas(schemas, outputUrl[, options])  

```
shell-js> util.dumpSchemas(["world"], "worlddump", {
       > "osBucketName": "hanna-bucket", "osNamespace": "idx28w1ckztq", 
       > "ocimds": "true", "compatibility": ["strip_definers", "strip_restricted_grants"]})
```


>[!IMPORTANT]
>
> The util.dump* library are executed in MySQL Shell by Java Script interpreter, so to acces this option type \js 

>[!IMPORTANT]
>
> The backup of MySQL Shell made by util.dump*, create the backup in a folder including all data and metadata in separate files


### This example create a backup of schema/database who call "MyDatabaseName" in temporary directory with backup name "MybackupDatabaseName", compressed using 4 thread, each thread is one cpu core

##### Windows
```util.dumpSchemas(["MyDatabaseName"], "C:/Temp/MybackupDatabaseName/", {dryRun:false, showProgress:true, consistent:false, compression:"zstd", threads:4})```

##### Linux
```util.dumpSchemas(["MyDatabaseName"], "/tmp/MybackupDatabaseName/", {dryRun:false, showProgress:true, consistent:false, compression:"zstd", threads:4})``` 




