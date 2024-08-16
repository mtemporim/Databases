[<-- Back to Utils](https://github.com/mtemporim/Databases/tree/main/MySQL/Utils)


### Some scripts that i use to automatize grants on MySQL

### It generate and apply commands to grant DML and DDL for all schemas on instance except system databases
```sql
#!/bin/bash

# Connect to MySQL
MYSQL_USER="root"
MYSQL_PASS="1QwK(y&QE&R!!b\%"
MYSQL_HOST="localhost"

# Users list 
DEV_USERS=("aab6329" "acf6330" "acb6334" "daf6327" "hmf6331" "tes6319")

# Command to list new schemas 
schemas=$(mysql -u${MYSQL_USER} -p${MYSQL_PASS} -h${MYSQL_HOST} -N -e "SHOW DATABASES;" | grep -Ev "(information_schema|performance_schema|mysql|sys)")

for schema in $schemas; do
  for user in "${DEV_USERS[@]}"; do
    mysql -u${MYSQL_USER} -p${MYSQL_PASS} -h${MYSQL_HOST} -e "GRANT EXECUTE, SELECT, SHOW VIEW, ALTER, ALTER ROUTINE, CREATE, CREATE ROUTINE, CREATE TEMPORARY TABLES, CREATE VIEW, DELETE, DROP, EVENT, INDEX, INSERT, REFERENCES, TRIGGER, UPDATE, LOCK TABLES ON ${schema}.* TO '${user}'@'%';"
  done
done

#  Apply changes 

mysql -u${MYSQL_USER} -p${MYSQL_PASS} -h${MYSQL_HOST} -e "FLUSH PRIVILEGES;"
```