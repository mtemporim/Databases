[<-- Back to Enterprise Manager](https://github.com/mtemporim/Databases/tree/main/Oracle/11g/EM)


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
**Check db timezone and SO timezone**
```bash
timedatectl
```
**On shell, open sqlplus.**
```bash
sqlplus / as sysdba 
```
**In sqlplus check the DBTimezone.**
```sql
SELECT DBTIMEZONE FROM DUAL;
```
**If need, change DBTimezone(in my case Brazil UTC) 
```SQL
ALTER DATABASE SET TIME_ZONE='-03:00';
```

Exit of sqlplus

>[!IMPORTANT]
>
>Be sure that the OEM was unnistalled and DBControl removed

**Check pfile**
```bash
cat $ORACLE_HOME/dbs/initsigrh.ora
```
**check environment variables**
```bash
env
```
**Check listener**
```bash
cat  $ORACLE_HOME/network/admin/listener.ora
```
**Check tnsnames**
```bashs
cat  $ORACLE_HOME/network/admin/tnsnames.ora
```
**Check hostname**
```bash
hostname
```
>[!CAUTION]
>
>Make sure the listener is working ok, the environment variables, hostname

**Recreate repository**
```bash
emca -config all db -repos recreate -DBCONTROL_HTTP_PORT 5500
```

Database SID: <$ORACLE_SID>  -- Must be insert $ORACLE_SID value ```bash echo $ORACLE_SID```  
Central agent home: <$ORACLE_HOME> -- Must be insert $ORACLE_HOME value ```bash echo $ORACLE_SID```  



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