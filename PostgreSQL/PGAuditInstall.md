# Instalação do PGAudit para POstgreSQL 15 No Oracle Linux 9.5

## Identificar o repositório baixar e isntalar o PGAudit

### Instalar o repositório PGDG

```shellscript
dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

### 2. Desabilitar o módulo padrão do postgresql do OL9 para evitar conflito

```shellscript
dnf -qy module disable postgresql
```

### 3. Atualizar o cache

```shellscript
dnf makecache
```

### 4. Buscar o pgaudit agora

```shellscript
dnf search pgaudit
```

### 5. Identificar qual é a versão do PGAudit para a versão correspondente do PostgreSQL (Nosso caso a PostgreSQL 15 o arquivo pgaudit17_15)

```shellscript
dnf search pgaudit
```

### 6. Instalar o PGAudit

```shellscript
dnf install pgaudit17_15 -y
```

### 7. Identificar o caminho para "Data Directory" para obtermos a localização do arquivo postgresql.conf

```shellscript
/opt/postgresql/15.13/bin/psql -U postgres -c "SHOW data_directory;"
```

### 8. Para o nosso caso onde o Data Directory apontou para  /pgsql/data/<app_dir>, será executado o comando:

```shellscript
vi /pgsql/data/<app_dir>/postgresql.conf
```

>[!IMPORTANT]
>app_dir é a minha aplicação que omiti o nome, o diretório será outro no caso da instalação default

### 9. Localizar o parâmetro shared_preload_libraries e ajuste caso não existir adicione as seguintes linhas ao final do arquivo

```ini
shared_preload_libraries = 'pgaudit'
pgaudit.log = 'all'
pgaudit.log_relation = on
pgaudit.log_parameter = on
```

### 10. Salve o arquivo, O PostgreSQL está sendo executado a partir de /opt/postgresql/15.13/bin/ (instalação customizada), porém o pgaudit17_15 foi instalado pelo PGDG em outro caminho. Precisamos localizar onde o .so foi instalado:

```shellscript
find / -name "pgaudit.so" 2>/dev/null
```

No nosso caso  
**/usr/pgsql-15/lib/pgaudit.so**  

O pgaudit.so está em /usr/pgsql-15/lib/ mas o  PostgreSQL está em /opt/postgresql/15.13/. São duas instalações diferentes.  
É preciso copiar a biblioteca para o diretório correto:

### 11. Confirmar o caminho correto do lib da instalação

Localizar as libs

```shellscript
find /opt/postgresql/15.13 -name "*.so" | head -5
```

### 12. Copiar o pgaudit.so para o diretório de instalação customizado

```shellscript
cp /usr/pgsql-15/lib/pgaudit.so /opt/postgresql/15.13/lib/
```

Confirmar a copia  

```shellscript
ls -la /opt/postgresql/15.13/lib/pgaudit.so
```

### 13. Identifica o servico do systemd e reiniciar o PostgreSQL

```shellscript
systemctl list-units | grep postgres
systemctl stop postgresql.service
systemctl start postgresql.service
```