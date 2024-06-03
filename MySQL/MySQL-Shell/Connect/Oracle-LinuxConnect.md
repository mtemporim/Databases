[<-- Back to Connect](https://github.com/mtemporim/Databases/tree/main/MySQL/MySQL-Shell/Connect)

[MySQL-Shell Connections Documentation](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-connections.html)


>[!TIP]
>
> Suppose that is logged on Linux Terminal


### Syntax 
```\connect <user>@<IP>:<port>```  *On MySQL-Shell client command line 

```mysqlsh \c <user>@<IP>:<port>``` *On Linux commnand line (bash)


### Call MySQL-Shell client by Linux command line to execute connect string 

```bash
mysqlsh
```

### Execute the connect string 

```bash
\connect user@192.168.0.10:3306
```
### Syntax to connect in specified script mode (only Linux command line)

#### SQL 
```bash
\connect user@192.168.0.10:3306  --sql
```

### Syntax to connect in localhost

#### Localhost
```bash
\connect user@localhost:3306
```

#### 127.0.0.1
```bash
\connect user@127.0.0.1:3306
```

### Syntax to connect in specified script mode (only Linux command line) directly by Linux Shell  
#### SQL  
```bash
mysqlsh \c user@192.168.0.10:3306 --sql
```

#### JavaScript 
```bash
mysqlsh \c user@192.168.0.10:3306 --js
```

#### Python 
```bash
mysqlsh \c user@192.168.0.10:3306 --py
```





### Exit/quit MySQL-Shell client
```bash
\q
``` 
or
```bash
\quit
```





>[!IMPORTANT]
>
> Exists some details on MySQL-Shell that be observed to who don't have worked with it yeat [BasicCMD-Overview](https://github.com/mtemporim/Databases/blob/main/MySQL/MySQL-Shell/Overview/BasicCMD-Overview.md)

