# Manual de atualização do MySQL 8.0 para 8.4 LTS utilizando os binários (tarball)

[https://dev.mysql.com/doc/refman/8.4/en/upgrading.html]

---

> [!NOTE]
>**Premissa:** Este manual assume que o MySQL 8.0 foi instalado seguindo o procedimento descrito no manual **MySQL8-Install.md**, com a estrutura de diretórios, my.cnf e symlink `/mysql/mysql` já estabelecidos.
> O `my.cnf` existente **não precisa ser alterado** — todos os parâmetros foram validados como compatíveis com o MySQL 8.4.

---

## Pré-requisitos da atualização

Verificar os pacotes

```shell
dnf -y install libaio numactl-libs ncurses-compat-libs openssl
```

### Carregar a senha se root (temporariamente)

```shell
senha = Alesc01#
```

### Verificar a versão atual em execução

```shell
df -h | grep mysql
ls -ld /mysql/*

mysqld --version
mysql -u root -p$senha -e "SELECT VERSION();"
mysql -u root -p$senha -e "SHOW VARIABLES LIKE 'datadir';"
mysql -u root -p$senha -e "SHOW VARIABLES LIKE 'innodb_log_group_home_dir';"
mysql -u root -p$senha -e "SHOW VARIABLES LIKE 'gtid_mode';"
mysql -u root -p$senha -e "SHOW VARIABLES LIKE 'enforce_gtid_consistency';"
```

Confirmar que a versão é **8.0.x** antes de prosseguir.

### Verificar o status do serviço e confirmar saúde do banco

```shell
systemctl status mysqld
```

```shell
mysql -u root -p$senha -e "SHOW GLOBAL STATUS LIKE 'Uptime';"
mysql -u root -p$senha -e "SHOW GLOBAL STATUS LIKE 'Threads_connected';"
```

### Executar o verificador de compatibilidade pré-upgrade

O `mysqlcheck` analisa todas as tabelas em busca de incompatibilidades antes da atualização.

```shell
mysqlcheck -u root -p$senha --all-databases --check-upgrade
```

> Se retornar apenas `OK` em todas as tabelas, está pronto para prosseguir.  
> Qualquer `error` ou `Table upgrade required` deve ser corrigido antes de continuar.

### Verificar e purgar binlogs antigos (recomendado)

```shell
mysql -u root -p$senha -e "SHOW BINARY LOGS;"
mysql -u root -p$senha -e "PURGE BINARY LOGS BEFORE DATE_SUB(NOW(), INTERVAL 1 DAY);"
```

### Verificar se há transações XA pendentes

```shell
mysql -u root -p$senha -e "XA RECOVER;"
```

> Se retornar linhas, as transações XA devem ser resolvidas (commit ou rollback) antes de prosseguir.

---

## Backup físico antes da atualização

> [!IMPORTANT]
> Este passo é **obrigatório**. Realize o backup completo do datadir antes de qualquer alteração.

```shell
systemctl stop mysqld
```

Confirmar que o serviço parou completamente

```shell
systemctl status mysqld
```

Realizar o backup do datadir

Backup físico (datadir + binlogs + my.cnf)

```shell
mkdir -p /mysql/dump/pre-upgrade-mysql8.0-$(date +%F)

tar -czf /mysql/dump/pre-upgrade-mysql8.0-$(date +%F)/pre-upgrade-mysql8.0-$(date +%F).tar.gz \
  /mysql/data \
  /mysql/log \
  /mysql/innodb \
  /etc/my.cnf
```

Confirmar o backup

```shell
ls -lh /mysql/dump/pre-upgrade-mysql8.0-$(date +%F) | grep tar.gz
```

## Baixar e extrair o tarball do MySQL 8.4

1. Acesse a página de download do MySQL [https://downloads.mysql.com/], vá para a página de downloads do **MySQL Community Server**.
2. Em **Select Version:** Escolha a versão desejada do MySQL (por exemplo, 8.0 ou 8.4 LTS).
3. Em **Select Operating System:** Escolha a opção  "**Linux - Generic**" no menu suspenso.
4. Em **Select OS Version:** Escolha a versão e a arquitetura do sistema (por exemplo, x86, 64 bits, ARM, 64 bits).

Copiar o link para download e MD5 para validação do checksum

```shell
cd /tmp
```

Baixar o tarball (Exemplo: versão 8.4.x, glibc2.28, x86_64)

```shell
wget https://dev.mysql.com/get/Downloads/MySQL-8.4/mysql-8.4.8-linux-glibc2.28-x86_64.tar.xz
```

> Substituir pela versão mais recente disponível na página de downloads do MySQL Community Server.  
> Copie o MD5 exibido na página para validação do checksum.

Validar a integridade do arquivo  

Exemplo: EXPECTED="2183f24aeaef90d93905dcbd9215176c"

```shell
EXPECTED="<colar aqui o MD5 da página do MySQL>"
```

Exemplo ACTUAL=$(md5sum mysql-8.4.8-linux-glibc2.28-x86_64.tar.xz | awk '{print $1}')

```shell
ACTUAL=$(md5sum <Arquivo de instalação> | awk '{print $1}')
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

Extrair o tarball

```shell
tar -xJf mysql-8.4.8-linux-glibc2.28-x86_64.tar.xz
```

Mover o diretório da nova versão para /mysql

```shell
mv /tmp/mysql-8.4.8-linux-glibc2.28-x86_64 /mysql/
```

Confirmar que o diretório foi criado

```shell
ls /mysql/ | grep 8.4
```

---

## Atualizar o link simbólico

Esta é a etapa central do processo. O symlink `/mysql/mysql` aponta para o diretório da versão em uso. Basta redirecioná-lo para o 8.4.

Remover o symlink atual (que aponta para o 8.0)

```shell
unlink /mysql/mysql
```

Criar o novo symlink apontando para o 8.4

```shell
ln -s /mysql/mysql-8.4.8-linux-glibc2.28-x86_64 /mysql/mysql
```

Confirmar o novo symlink

```shell
ls -la /mysql/mysql
```

Saída esperada:

```text
lrwxrwxrwx. 1 root root XX ... /mysql/mysql -> /mysql/mysql-8.4.5-linux-glibc2.28-x86_64
```

Confirmar a estrutura de binários

```shell
ls /mysql/mysql/bin/mysqld        # deve existir
ls /mysql/mysql/share/english/    # deve conter errmsg.sys
ls /mysql/mysql/lib/plugin/       # deve conter *.so
```

Verificar a nova versão

```shell
mysqld --version
```

Deve retornar `mysqld  Ver 8.4.x`.

Ajustar o proprietário do novo diretório de binários

```shell
chown -R mysql:mysql /mysql/mysql
```

---

## Configurar os contextos SELinux para o novo diretório 8.4

Libera o symlink atualizado

```shell
semanage fcontext -a -t bin_t "/mysql/mysql"
restorecon -v /mysql/mysql
```

Libera o novo diretório de instalação do 8.4

```shell
semanage fcontext -a -t bin_t "/mysql/mysql-8.4.8-linux-glibc2.28-x86_64(/.*)?"
restorecon -R -v /mysql/mysql-8.4.8-linux-glibc2.28-x86_64
```

> Os contextos dos diretórios de dados, logs e innodb já foram configurados durante a instalação do 8.0 e **não precisam ser refeitos**.

---

## Validar o my.cnf com o novo binário

Antes de subir o serviço, validar que o 8.4 lê o my.cnf sem erros

```shell
mysqld --defaults-file=/etc/my.cnf --validate-config
```

> Nenhuma mensagem de `[ERROR]` deve aparecer. O warning sobre `binlog_format` não se aplica pois o parâmetro já foi removido do my.cnf.

---

## Atualizar a descrição do serviço systemd (opcional)

O arquivo de serviço funciona sem alterações pois o `ExecStart` aponta para `/mysql/mysql/bin/mysqld` via symlink. Caso queira atualizar a descrição:

```shell
sed -i 's/Description=MySQL 8 Server/Description=MySQL 8.4 LTS Server/' /etc/systemd/system/mysqld.service
```

Recarregar o systemd

```shell
systemctl daemon-reload
```

---

## Iniciar o MySQL 8.4

```shell
systemctl start mysqld
```

> [!IMPORTANT]
> No MySQL 8.4, o processo de upgrade do dicionário de dados e das tabelas do sistema é executado **automaticamente** no primeiro start. Não é necessário executar `mysql_upgrade` manualmente — esse comando foi removido no 8.4.

Monitorar o log durante a inicialização

```shell
tail -f /mysql/log/log_mysql.err
```

Aguardar as seguintes mensagens que confirmam o upgrade automático concluído:

```text
[System] [MY-013576] [InnoDB] InnoDB initialization has started.
[System] [MY-013577] [InnoDB] InnoDB initialization has ended.
[System] [MY-010931] [Server] /mysql/mysql/bin/mysqld: ready for connections.
```

Confirmar status do serviço

```shell
systemctl status mysqld
```

---

## Validação pós-upgrade

### Verificar a versão em execução

```shell
mysql -u root -p$senha -e "SELECT VERSION();"
```

Deve retornar `8.4.x`.

### Dentro do MySQL — confirmar variáveis de caminho

```sql
mysql -u root -p$senha -e "SHOW VARIABLES LIKE 'datadir';"
mysql -u root -p$senha -e "SHOW VARIABLES LIKE 'innodb_undo_directory';"
mysql -u root -p$senha -e "SHOW VARIABLES LIKE 'innodb_log_group_home_dir';"
mysql -u root -p$senha -e "SHOW VARIABLES LIKE 'innodb_doublewrite_dir';"
mysql -u root -p$senha -e "SHOW VARIABLES LIKE 'tmpdir';"
mysql -u root -p$senha -e "SHOW VARIABLES LIKE 'secure_file_priv';"
mysql -u root -p$senha -e "SHOW VARIABLES LIKE 'log_error';"
mysql -u root -p$senha -e "SHOW VARIABLES LIKE 'log_bin%';"
```

### Confirmar que as undo tablespaces estão no lugar certo

```sql
mysql -u root -p$senha -e "SELECT TABLESPACE_NAME, FILE_NAME FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'UNDO LOG';"
```

### Confirmar que o upgrade do schema interno foi concluído

```sql
mysql -u root -p$senha -e "SELECT SCHEMA_NAME, DEFAULT_CHARACTER_SET_NAME FROM INFORMATION_SCHEMA.SCHEMATA;"
```

```sql
mysql -u root -p$senha -e "SHOW DATABASES;"
```

### Verificar integridade das tabelas do sistema

```shell
mysqlcheck -u root -p$senha --all-databases
```

> Todas as tabelas devem retornar `OK`.

---

## Validação final da estrutura de arquivos

```shell
ls /mysql/data/               # ibdata1, mysql/, sys/, performance_schema/
ls /mysql/innodb/redo/        # deve conter diretório #innodb_redo
ls /mysql/innodb/undo/        # deve conter undo_001, undo_002
ls /mysql/innodb/doublewrite/ # deve conter arquivos #ib_*
ls /mysql/innodb/temp/        # deve conter ibtmp1 e diretório #innodb_temp
ls /mysql/log/                # deve conter log_mysql.err e mysql-bin.000001
```

---

## Pós-upgrade — limpeza do diretório 8.0

Após validar que o ambiente 8.4 está estável (recomendado aguardar pelo menos 48h em produção), o diretório da versão anterior pode ser removido.

```shell
rm -rf /mysql/mysql-8.0.45-linux-glibc2.28-x86_64
```

> [!IMPORTANT]
> Só execute a remoção após confirmar que **não há necessidade de rollback**. Enquanto o diretório 8.0 existir, o rollback é possível apenas redirecionando o symlink de volta.

### Rollback (se necessário antes de remover o 8.0)

```shell
systemctl stop mysqld
unlink /mysql/mysql
ln -s /mysql/mysql-8.0.45-linux-glibc2.28-x86_64 /mysql/mysql
semanage fcontext -a -t bin_t "/mysql/mysql"
restorecon -v /mysql/mysql
systemctl start mysqld
```

> [!IMPORTANT]
> O rollback só é possível se o datadir **não foi modificado** pelo 8.4 de forma incompatível. Uma vez que o MySQL 8.4 escreve no datadir (upgrade automático do dicionário), o rollback para o 8.0 pode não ser suportado. Por isso o backup do datadir antes da atualização é obrigatório.
