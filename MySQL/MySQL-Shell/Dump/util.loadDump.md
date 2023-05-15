[<-- Back to Dump](https://github.com/mtemporim/Databases/tree/main/MySQL/MySQL-Shell/Dump)

[util.loadDump documentation](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-utilities-load-dump.html)

### util.loadDump(url[, options])

shell-js> util.loadDump("/mnt/data/worlddump", {dryRun: true})
shell-js> util.loadDump("worlddump", {threads: 8, osBucketName: "hanna-bucket", osNamespace: "idx28w1ckztq"})

>[!IMPORTANT]
>
> Important: The util.loadDump library are executed in MySQL Shell by Java Script interpreter, so to acces this option type \js 


>[!IMPORTANT]
>
> Important 2: the backup of MySQL Shell made by util.dump*, create the backup in a folder including all data and metadata in separate files, to load you is needed this entire folder

### This example load a backup of schema/database who call "MyDatabaseName" in C:\Temp with backup name "MybackupDatabaseName" into another MySQL instance

##### Windows
  ```util.loadDump("C:/Temp/MyDatabaseName/", {dryRun:false, showProgress:true, resetProgress:true, skipBinlog:true, threads:4, loadDdl:true, loadIndexes:true})```

 
 ##### Linux
 ```util.loadDump("/tmp/MyDatabaseName/", {dryRun:false, showProgress:true, resetProgress:true, skipBinlog:true, threads:4, loadDdl:true, loadIndexes:true})```
 


