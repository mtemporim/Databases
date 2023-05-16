[<-- Back to Install](https://github.com/mtemporim/Databases/tree/main/MySQL/MySQL-Shell/Install)

[MySQL-Shell Install Documentation](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-install-linux-quick.html)


>[!IMPORTANT]
>
> This case is based on Oracle Linux 8 

### Install MySQL-Shell on Oracle Linux by prm

# Get MySQL-Shell for your version

[MySQL-Shell Download](https://dev.mysql.com/downloads/shell/)

### Move and rum the file downloaded to your server, in this case the file is on /tmp directory 

```rpm -ivh mysql-shell-8.0.33-1.el8.x86_64.rpm```

>[!NOTE]
>
> In case error occur of lib, instal this [lib package](https://objectstorage.sa-saopaulo-1.oraclecloud.com/n/gr7en3021aot/b/repo-git/o/libyaml-0.1.4-11.el7_0.x86_64.rpm)  


### Move and rum the lib file downloaded to your server, in this case the file is on /tmp directory to

```rpm -ivh libyaml-0.1.4-11.el7_0.x86_64.rpm```









