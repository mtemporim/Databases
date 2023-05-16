[<-- Back to Install](https://github.com/mtemporim/Databases/tree/main/MySQL/MySQL-Shell/Connect)

[MySQL-Shell Connections Documentation](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-connections.html)


>[!TIP]
>
> Suppose that is logged on Linux Terminal


### Sintax 
´´´\connect <user>@<IP>:<port>´´´  On MySQL-Shell client command line 

´´´mysqlsh \c <user>@<IP>:<port>´´´ On Linux commnand line


### Call MySQL-Shell client by Linux command line to execute connect string 

```mysqlsh```

### Execute the connect string 

```\connect user@192.168.0.10:3306```


### Its possible to connect directly by Linux command line 

´´´mysqlsh \c user@192.168.0.10:3306´´´

### Exit/quit MySQL-Shell client
```\q``` or  ```\quit```


>[!IMPORTANT]
>
> Exists some details on MySQL-Shell that be observed to who don't have worked with it yeat [BasicCMD-Overview]()

