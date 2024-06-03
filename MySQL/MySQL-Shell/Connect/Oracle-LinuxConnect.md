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
### Syntax to connect by MySQL Shell connect string  
```bash
\connect user@192.168.0.10:3306
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

### Syntax to connect in specified script mode directly by Linux Shell (only Linux command line)  
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

### Syntax to connect in localhost directly by Linux Shell (only Linux command line)
#### Localhost
```bash
mysqlsh user@localhost:3306
```

#### 127.0.0.1
```bash
mysqlsh user@127.0.0.1:3306
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
> Exists some details on MySQL-Shell that be observed to who don't have worked with it yeat [Basic CMD-Overview](https://github.com/mtemporim/Databases/blob/main/MySQL/MySQL-Shell/Overview/BasicCMD-Overview.md)

