


Theres some case that we have to recreate the enterprime manager, like after a backup restore, import etc..  

How to recreate de Enterprise Manager 

>[!NOTE]
>
>This process must be done with the oracle user on the operating system "su - oracle"


**View DB Console status**
```bash
emctl status dbconsole
```
**View agent status**
```bash
emctl status agent
```
**Stop DB Console**
```bash
emctl stop dbconsole
```
**Stop agent**
```bash
emctl stop agent
```

-> Check db timezone and SO timezone

$ timedatectl

$ sqlplus / as sysdba
SELECT DBTIMEZONE FROM DUAL;

alter database set time_zone='-03:00';


-> Check spfile
cat $ORACLE_HOME/dbs/spfilesigrh.ora
-> Check pfile
cat $ORACLE_HOME/dbs/initsigrh.ora
-> Check env
vim .bash_profile
-> Check listener
cat  $ORACLE_HOME/network/admin/listener.ora
-> Check tnsnames
cat  $ORACLE_HOME/network/admin/tnsnames.ora
-> Edit hostname
vim /etc/hostname


-> Drop Enterprise Manager
$ sqlplus / as sysdba
exec DBMS_AQADM.DROP_QUEUE_TABLE(queue_table=>'SYSMAN.MGMT_NOTIFY_QTABLE',force=>TRUE);
SHUTDOWN IMMEDIATE;

STARTUP RESTRICT;

DECLARE
CURSOR c1 IS
SELECT owner, synonym_name name
FROM dba_synonyms
WHERE table_owner = 'SYSMAN';
BEGIN
FOR r1 IN c1
LOOP
IF r1.owner = 'PUBLIC' THEN
EXECUTE IMMEDIATE 'DROP PUBLIC SYNONYM '||r1.name;
ELSE
EXECUTE IMMEDIATE 'DROP SYNONYM '||r1.owner||'.'||r1.name;
END IF;
END LOOP;
END;
/

set pages 10
set head off
set lines 300
spool drop.sql
select 'drop public synonym '||object_name||';'
from dba_objects
where object_name like '%MGMT%'
and status = 'INVALID';

DROP ROLE mgmt_user;
DROP user mgmt_view cascade;
drop public synonym MGMT_TARGET_BLACKOUTS;
drop public synonym SETEMVIEWUSERCONTEXT;

ALTER SYSTEM DISABLE RESTRICTED SESSION;

$ rm -rf $ORACLE_HOME/nodename_dbuniquename
$ rm -rf $ORACLE_HOME/nodename_dbuniquename

-> Not mandatory deconfig dbcontrol
## emca -deconfig dbcontrol db -repos drop


-> recreate repository
emca -config all db -repos recreate -DBCONTROL_HTTP_PORT 5500

Database SID: &DATABASESID
Central agent home: &LITERAL_ORACLE_HOME_PATH (echo $ORACLE_HOME)

-> start and stop dbconsole
emctl start dbconsole
emctl stop dbconsole


-> edit emd.properties
vi $ORACLE_HOME/nodename_dbuniquename/sysman/config/emd.properties
## Change timezone: eg.: agentTZRegion=Brazil/East or agentTZRegion=US/Eastern

-> start dbconsole
emctl start dbconsole



References:
https://anargodjaev.wordpress.com/2015/01/07/how-to-manually-remove-dbconsole-repository/
https://docs.oracle.com/cd/E11882_01/server.112/e25494/dbcontrol.htm#ADMIN13414
https://www.fatihacar.com/blog/how-to-change-oracle-enterprise-manager-port-number/
http://luna-ext.di.fc.ul.pt/oracle11g/install.112/e10840/app_port.htm