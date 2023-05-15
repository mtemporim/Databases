
[<-- Back to mysqldump](https://github.com/mtemporim/Databases/tree/main/MySQL/mysqldump)

[mysqldump documentation](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html)

### mysqldump [options] > dump.sql

options:   
1.  -u = Explicit the user
1.  -p = Prompt to get password 
1.  -P = Specify the port of instance 
1.  -h = IP of server  

### Create a single database dump 
`mysqldump -u <user> -p -P 3306 -h <ip server> database_name > database_name.sql`

### Multiple MySQL databases dump
`mysqldump -u <user> -p -P 3306 -h <ip server> --databases database_name_a database_name_b > databases_a_b.sql`

### All MySQL databases dump
`mysqldump -u <user> -p -P 3306 -h <ip server> --all-databases > all_databases.sql`

### Create a compressed MySQL database dump
`mysqldump -u <user> -p -P 3306 -h <ip server> database_name | gzip > database_name.sql.gz`

### Create a dump with Timestamp
`mysqldump -u <user> -p -P 3306 -h <ip server> database_name > database_name-$(date +%Y%m%d).sql`