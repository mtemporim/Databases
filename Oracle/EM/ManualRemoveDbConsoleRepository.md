[<-- Back to Enterprise Manager](https://github.com/mtemporim/Databases/blob/main/Oracle/EM/README.md)  


# Manually removing the DBConsole Repository

Theres some case that we have to remove the enterprime manager dbconsole repository, like after a backup restore, import etc..  

How to do this? 

>[!NOTE]
>
>This process must be done with the oracle user on the operating system "su - oracle"


On **shell**, open sqlplus* 
```bash
sqlplus / as sysdba 
```

In **sqlplus** Execute DBMS_AQADM.DROP_QUEUE_TABLE Procedure 
```sql
exec DBMS_AQADM.DROP_QUEUE_TABLE(
       queue_table=>'SYSMAN.MGMT_NOTIFY_QTABLE',
       force=>TRUE);
```

Next step should be droped the sysman account and management objects
Still in **sqlplus**, shutdown Oracle instance 
```sql
SHUTDOWN IMMEDIATE;
```
Start Oracle instance on restrict mode 
```sql
STARTUP RESTRICT;
```



