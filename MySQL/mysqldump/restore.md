[<-- Back to mysqldump](https://github.com/mtemporim/Databases/tree/main/MySQL/mysqldump)

[mysqlpump documentation](https://dev.mysql.com/doc/refman/8.0/en/mysqlpump.html)

### mysql [options] database_name < dump.sql

options: 
1. -u = Explicit the user
1. -p = Prompt to get password 
1. -P = Specify the port of instance 
1. -h = IP of server  

### Restore a single MySQL database from a full MySQL dump 
```bash
mysql -u <user> -p -P 3306 -h <ip server> --one-database database_name < all_databases.sql
```

### Export and import a MySQL database in one command in other instance with another name 
```bash
mysqldump -u <user> -p -P 3306 -h <ip server> database_name | mysql -u <user> -p -P 3306 -h <ip other server> remote_database_name
```

