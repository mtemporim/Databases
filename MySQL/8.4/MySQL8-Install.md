# Manual de instalação do MySQL utilizando os binários (tarball)

[https://dev.mysql.com/doc/refman/8.0/en/binary-installation.html]

## Pré-requisitos do sistema

### Verificar dependências necessárias

```shell
dnf -y install libaio numactl-libs ncurses-compat-libs openssl
```

Criar o usuário e grupo mysql

```shell
groupadd -r mysql
useradd -r -g mysql -s /sbin/nologin -d /mysql mysql
```

Verificar se todos os discos estão nmontados

```shell
lsblk
df -h | grep mysql
```

Criar os  diretórios principais

```shell
mkdir -p /mysql/data
mkdir -p /mysql/innodb/{redo,undo,doublewrite,temp}
mkdir -p /mysql/audit
mkdir -p /mysql/log
mkdir -p /mysql/dump/securefile
mkdir -p /mysql/scripts

chown -R mysql:mysql /mysql
chmod -R 750 /mysql
chmod 770 /mysql/dump/securefile
```

### Ajustes de kernel (mínimo recomendado)

```shell
cat > /etc/sysctl.d/99-mysql.conf << 'EOF'
vm.swappiness = 1
vm.overcommit_memory = 1
fs.aio-max-nr = 1048576
EOF

sysctl --system
```

### Limites de arquivos

```shell
cat > /etc/security/limits.d/mysql.conf << 'EOF'
mysql soft nofile 1048576
mysql hard nofile 1048576
EOF
```

### Baixar e extrair o tarball

1. Acesse a página de download do MySQL [https://downloads.mysql.com/], vá para a página de downloads do **MySQL Community Server**.
2. Em **Select Version:** Escolha a versão desejada do MySQL (por exemplo, 8.0 ou 8.4 LTS).
3. Em **Select Operating System:** Escolha a opção  "**Linux - Generic**" no menu suspenso.
4. Em **Select OS Version:** Escolha a versão e a arquitetura do sistema (por exemplo, x86, 64 bits, ARM, 64 bits).

Copiar o link para download e MD5 para validação do checksum

```shell
cd /tmp
```

Baixar o tarball (versão 8.0.45, glibc2.28, x86_64) 

```shell
wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.45-linux-glibc2.28-x86_64.tar.xz
```

Cole aqui o MD5 que você está vendo na página do MySQL de onde foi copiado o link para o download  
EXPECTED="ebe0a06e5f0125c0265eb3576b4c01a4"

```shell
EXPECTED="<colar aqui>"
```

Execute esse comando informando o nome do arquivo baixado  
ACTUAL=$(md5sum mysql-8.0.45-linux-glibc2.28-x86_64.tar.xz | awk '{print $1}')

```shell
ACTUAL=$(<nome do arquivo baixado> | awk '{print $1}')
```

```shell
if [ "$ACTUAL" = "$EXPECTED" ]; then
    echo " "
    echo "✅ MD5 OK! Arquivo íntegro e pronto para usar."
    echo " "
else
    echo " "
    echo "❌ MD5 não bate! Apague o arquivo e baixe novamente."
    echo " "

fi
```

Caso o arquivo estiver ok, extrair

```shell
tar -xJf mysql-8.0.45-linux-glibc2.28-x86_64.tar.xz
```

Após a extração do tarball em /tmp, mover o diretório completo da versão para /mysql instalada.

```shell
mv /tmp/mysql-8.0.45-linux-glibc2.28-x86_64 /mysql/
```

Criar um link simbólico /mysql/mysql apontando para o diretório da versão

```shell
ln -s /mysql/mysql-8.0.45-linux-glibc2.28-x86_64 /mysql/mysql
```

Confirmar estrutura

```shell
ls /mysql/mysql/bin/mysqld        # deve existir
ls /mysql/mysql/share/english/    # lc_messages_dir — deve conter errmsg.sys
ls /mysql/mysql/lib/plugin/       # plugin_dir — deve conter *.so
```

### Configurar o PATH e variáveis de ambiente

Adicionar ao PATH do sistema

```shell
cat >> /etc/profile.d/mysql.sh << 'EOF'
export PATH=/mysql/mysql/bin:$PATH
export MYSQL_HOME=/mysql/mysql
EOF

source /etc/profile.d/mysql.sh
```

Confirmar

```shell
which mysqld
mysqld --version
```

### Instalar o my.cnf

Criar o arquivo /etc/my.cnf

```shell
cat > /etc/my.cnf << 'EOF'
[mysqld]
# =============================================================================
# CORE / PATHS - DIRETÓRIOS E ARQUIVOS BASE
# =============================================================================
basedir                         = /mysql/mysql
datadir                         = /mysql/data
innodb_data_home_dir            = /mysql/data/
pid-file                        = /run/mysqld/mysqld.pid
socket                          = /mysql/data/mysql.sock
plugin_dir                      = /mysql/mysql/lib/plugin
lc_messages_dir                 = /mysql/mysql/share

# =============================================================================
# INNODB PATHS
# =============================================================================

innodb_directories              = /mysql/data;/mysql/innodb

innodb_log_group_home_dir       = /mysql/innodb/redo
innodb_redo_log_capacity        = 2G                            

innodb_undo_directory           = /mysql/innodb/undo            

innodb_doublewrite_dir          = /mysql/innodb/doublewrite     
innodb_doublewrite              = ON                            

innodb_tmpdir                   = /mysql/innodb/temp            
innodb_temp_tablespaces_dir     = /mysql/innodb/temp            
tmpdir                          = /mysql/innodb/temp            
innodb_temp_data_file_path      = /mysql/innodb/temp/ibtmp1:12M:autoextend:max:10G  

# =============================================================================
# CHARACTER SET / COLLATION
# =============================================================================
character_set_server            = utf8mb4
collation_server                = utf8mb4_general_ci

# =============================================================================
# CONEXÃO E SEGURANÇA
# =============================================================================
server-id                       = 1
user                            = mysql
port                            = 3306
default_storage_engine          = innodb                        
max_allowed_packet              = 64M                           
max_connections                 = 300
connect_timeout                 = 30
secure-file-priv                = /mysql/dump/securefile        
skip-mysqlx                                                     

# =============================================================================
# PERFORMANCE E MEMÓRIA (otimizado para 16 GB RAM | SSD/NVMe)
# =============================================================================
innodb_buffer_pool_size         = 10G                           
innodb_buffer_pool_instances    = 8                             
innodb_log_buffer_size          = 32M                           
innodb_file_per_table           = ON                             
innodb_flush_method             = O_DIRECT                      
innodb_flush_log_at_trx_commit  = 1                             
innodb_io_capacity              = 1000                          
innodb_io_capacity_max          = 2000                          
innodb_stats_on_metadata        = OFF                             

thread_cache_size               = 50                            
table_open_cache                = 4000                          
table_definition_cache          = 2000                          

tmp_table_size                  = 128M                          
max_heap_table_size             = 128M                          
sort_buffer_size                = 4M                            
join_buffer_size                = 4M                            
read_buffer_size                = 2M                            
read_rnd_buffer_size            = 4M                            

# =============================================================================
# SERVER LOGS
# =============================================================================
log-error                       = /mysql/log/log_mysql.err      
log-timestamps                  = SYSTEM                        
log_error_verbosity             = 3                             

log_output                      = FILE                          

general_log                     = OFF
general_log_file                = /mysql/audit/general.log

slow_query_log                  = OFF
slow_query_log_file             = /mysql/audit/slow_query.log
long_query_time                 = 7                             

# =============================================================================
# BINARY LOG
# =============================================================================
log-bin                         = /mysql/log/mysql-bin          
log-bin-index                   = /mysql/log/mysql-bin          
sync_binlog                     = 1                             
binlog_expire_logs_seconds      = 432000                        

# =============================================================================
# REPLICAÇÃO E GTID
# =============================================================================
gtid_mode                       = ON                           
enforce_gtid_consistency        = ON                           





[client]
port                            = 3306
socket                          = /mysql/data/mysql.sock
EOF
```

Ajustar as permissões

```shell
chown root:root /etc/my.cnf
chmod 644 /etc/my.cnf
```

Verificar se o MySQL consegue ler sem erros de sintaxe

```shell
mysqld --defaults-file=/etc/my.cnf --validate-config
```

Se retornar sem mensagens de erro, está pronto. Qualquer linha problemática aparece aqui antes de inicializar o banco.

### Inicializar o banco de dados

Adicionar o arquivo de log de erro do MySQL antes do initialize para evitar erro de permissão no primeiro start.

```shell
touch /mysql/log/log_mysql.err
chown mysql:mysql /mysql/log/log_mysql.err
chmod 640 /mysql/log/log_mysql.err
```

Ajustar dono do basedir

```shell
chown -R mysql:mysql /mysql/mysql
```

Inicializar — este comando cria o datadir, undo, redo, doublewrite  
A senha temporária do root será gravada no log de erro

```shell
mysqld --defaults-file=/etc/my.cnf --initialize --user=mysql
```

Verificar a senha temporária gerada

```shell
grep 'temporary password' /mysql/log/log_mysql.err
```

### Configurar os contextos e permissões para o SELinux

Libera leitura do symlink pelo systemd

```shell
semanage fcontext -a -t bin_t "/mysql/mysql"
restorecon -v /mysql/mysql
```

Libera todo o diretório da instalação (binários, libs, share etc.)

```shell
semanage fcontext -a -t bin_t "/mysql/mysql-8.0.45-linux-glibc2.28-x86_64(/.*)?"
restorecon -R -v /mysql/mysql-8.0.45-linux-glibc2.28-x86_64
```

Libera os diretórios de dados/logs para o mysqld_t

```shell
semanage fcontext -a -t mysqld_db_t "/mysql/data(/.*)?"
semanage fcontext -a -t mysqld_db_t "/mysql/innodb(/.*)?"
semanage fcontext -a -t mysqld_log_t "/mysql/log(/.*)?"
semanage fcontext -a -t mysqld_log_t "/mysql/audit(/.*)?"
restorecon -R -v /mysql/data /mysql/innodb /mysql/log /mysql/audit
```

### Configurar o serviço systemd

Criar o arquivo de serviço

```shell
cat > /etc/systemd/system/mysqld.service << 'EOF'
[Unit]
Description=MySQL 8 Server
After=network.target

[Service]
Type=notify
User=mysql
Group=mysql
ExecStart=/mysql/mysql/bin/mysqld --defaults-file=/etc/my.cnf
ExecStop=/mysql/mysql/bin/mysqladmin --defaults-file=/etc/my.cnf shutdown
TimeoutSec=600
Restart=on-failure
LimitNOFILE=1048576
LimitNPROC=1048576
ProtectSystem=full
ProtectHome=true
PrivateTmp=true
NoNewPrivileges=true
RuntimeDirectory=mysqld
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
EOF
```

Recarregar systemd e habilitar o serviço

```shell
systemctl daemon-reload
systemctl enable mysqld
systemctl start mysqld
systemctl status mysqld
```

### Etapa de pós-inicialização

Login com a senha temporária

```shell
mysql -u root -p
```

(cole a senha temporária capturada no grep acima)

### Dentro do MySQL

Trocar a senha temporária (obrigatório antes de qualquer outra coisa)

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Alesc01#';
```

Confirmar variáveis de caminho estão corretas

```sql
SHOW VARIABLES LIKE 'datadir';
SHOW VARIABLES LIKE 'innodb_undo_directory';
SHOW VARIABLES LIKE 'innodb_log_group_home_dir';
SHOW VARIABLES LIKE 'innodb_doublewrite_dir';
SHOW VARIABLES LIKE 'tmpdir';
SHOW VARIABLES LIKE 'secure_file_priv';
SHOW VARIABLES LIKE 'log_error';
SHOW VARIABLES LIKE 'log_bin%';
```

Confirmar que os arquivos das undo tablespaces foram criados no lugar certo

```sql
SELECT TABLESPACE_NAME, FILE_NAME FROM INFORMATION_SCHEMA.FILES
WHERE FILE_TYPE = 'UNDO LOG';
```

### Validação final da estrutura de arquivos

Confirmar que os arquivos foram criados nos discos corretos

```shell
ls /mysql/data/           # ibdata1, mysql/, sys/, performance_schema/
ls /mysql/innodb/redo/    # deve conter diretório #innodb_redo
ls /mysql/innodb/undo/    # deve conter undo_001, undo_002
ls /mysql/innodb/doublewrite/  # deve conter arquivos #ib_*
ls /mysql/innodb/temp/    # deve conter ibtmp1 e diretório #innodb_temp
ls /mysql/log/            # deve conter log_mysql.err e mysql-bin.000001
```
