[<-- Back to Dump](https://github.com/mtemporim/Databases/tree/main/MySQL/MySQL-Shell/Dump)

[util.loadDump documentation](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-utilities-load-dump.html)

### util.loadDump(url[, options])
`
shell-js> util.loadDump("/mnt/data/worlddump", {dryRun: true})
shell-js> util.loadDump("worlddump", {threads: 8, osBucketName: "hanna-bucket", osNamespace: "idx28w1ckztq"})
`

>[!IMPORTANT]
>
> The util.loadDump library are executed in MySQL Shell by Java Script interpreter, so to acces this option type \js 


>[!IMPORTANT]
>
> The backup of MySQL Shell made by util.dump*, create the backup in a folder including all data and metadata in separate files, to load you is needed this entire folder

### This example load a backup of schema/database who call "MyDatabaseName" in temporary directory with backup name "MybackupDatabaseName" into another MySQL instance

##### Windows
  ```bash
  util.loadDump("C:/Temp/MyDatabaseName/", {dryRun:false, showProgress:true, resetProgress:true, skipBinlog:true, threads:4, loadDdl:true, loadIndexes:true})
  ```

 
 ##### Linux
 ```bash
 util.loadDump("/tmp/MyDatabaseName/", {dryRun:false, showProgress:true, resetProgress:true, skipBinlog:true, threads:4, loadDdl:true, loadIndexes:true})
 ```
 

>[!IMPORTANT]
>
> if you run the command and show the following message *ERROR: The 'local_infile' global system variable must be set to ON in the target server, after the server is verified to be trusted.* You should access the RDBMS and run the following commands:

### This command show status of "local_infile" variable
```sql
show variables like 'local_infile';
```
+---------------+-------+ 
| Variable_name | Value | 
+---------------+-------+ 
| local_infile  | OFF   | 
+---------------+-------+ 


### It enable the variable "local_infile"
```sql
set global local_infile=1;
```

```sql
show variables like 'local_infile';
```
+---------------+-------+ 
| Variable_name | Value | 
+---------------+-------+ 
| local_infile  | ON    | 
+---------------+-------+ 


*This variable controls server-side LOCAL capability for LOAD DATA statements. Depending on the local_infile setting, the server refuses or permits local data loading by clients that have LOCAL enabled on the client side.*

[local_infile Documentation]([https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-utilities-load-dump.html](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_local_infile)https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_local_infile)







