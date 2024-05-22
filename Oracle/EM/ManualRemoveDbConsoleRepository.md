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
>[!NOTE]
>
>**DBMS_AQADM.DROP_QUEUE_TABLE** This procedure drops an existing queue table. All the queues in a queue table must be stopped and dropped before the queue table can be dropped

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

**Execute sysman.setEMUserContext procedures**
```sql
EXEC sysman.setEMUserContext('',5);
```
>[!NOTE]
>
>**sysman.setEMUserContext** It is a procedure that defines the context in which other OEM procedures will be executed, ensuring that they have the appropriate environment configured.

**Revoke DBA role granted to SYSMAN user**
```sql 
REVOKE dba FROM sysman;
```
**Execute follow sql block to remove all synonym associate**
```sql
DECLARE
CURSOR c1 IS
  SELECT owner, synonym_name name
  FROM dba_synonyms
    WHERE table_owner = 'SYSMAN';
BEGIN
  FOR r1 IN c1
  LOOP
    IF r1.owner = 'PUBLIC' 
    THEN
      EXECUTE IMMEDIATE 'DROP PUBLIC SYNONYM '||r1.name;
    ELSE
      EXECUTE IMMEDIATE 'DROP SYNONYM '||r1.owner||'.'||r1.name;
    END IF;
   END LOOP;
END;
/
```
**Drop the mgmt_user role**
```sql
DROP ROLE mgmt_user;
```

**Drop mgmt_view schema and all objects**
```sql
DROP user mgmt_view cascade;
```
**Drop SYSMAN user/schema and all objects**
```sql
DROP user sysman cascade;
```
**Disable restrict mode**
```sql  
ALTER SYSTEM DISABLE RESTRICTED SESSION;
```
Exit sqlplus  
**Drop existing dbcontrol**
```bash
emca -deconfig dbcontrol db -repos drop
```
>[!IMPORTANT]
>
>Reference:
>Master Note for Enterprise Manager Configuration Assistant (EMCA) in Single Instance Database Environment [**ID 1099271.1**]