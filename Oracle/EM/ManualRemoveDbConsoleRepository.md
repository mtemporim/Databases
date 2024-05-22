[<-- Back to Enterprise Manager](https://github.com/mtemporim/Databases/blob/main/Oracle/EM/README.md)  


# Manually removing the DBConsole Repository

Theres some case that we have to remove the enterprime manager dbconsole repository, like after a backup restore, import etc..  

How to do this? 

>[!NOTE]
>
>This process must be done with the oracle user on the operating system "su - oracle"
 

**View DB Console status.**
```bash
emctl status dbconsole
```
**View agent status.**
```bash
emctl status agent
```
**Stop DB Console.**
```bash
emctl stop dbconsole
```
**Stop agent.**
```bash
emctl stop agent
```
**On shell, open sqlplus.**
```bash
sqlplus / as sysdba 
```
**In sqlplus Execute DBMS_AQADM.DROP_QUEUE_TABLE Procedure.**
```sql
exec DBMS_AQADM.DROP_QUEUE_TABLE(queue_table=>'SYSMAN.MGMT_NOTIFY_QTABLE',force=>TRUE);
```
## Next step should be droped the sysman account and management objects.
**Still in **sqlplus**, shutdown Oracle instance.** 
```sql
SHUTDOWN IMMEDIATE;
```
**Start Oracle instance on restrict mode.** 
```sql
STARTUP RESTRICT;
```
**Execute sysman.emd_maintenance.remove_em_dbms_jobs procedures**
```sql
EXEC sysman.emd_maintenance.remove_em_dbms_jobs;
```
>[!NOTE]
>
>**sysman.emd_maintenance.remove_em_dbms_jobs** It is a procedure to remove jobs from the database that have been scheduled by Oracle Enterprise Manager.


