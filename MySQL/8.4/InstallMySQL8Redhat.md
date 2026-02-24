# teste 

# Instalação do MySQL 8.4 no Red Hat Enterprise Linux 9.7

<br><br>  

## Índice

### Matriz de compatibilidade 

https://www.mysql.com/support/supportedplatforms/database.html

### Esquema de particionamento do MySQL 8.4

**Disk 1 - 200 GB**  

|Mounting Point|Size|
|:--|:--|
|boot     |1 GiB|
|boot efi |1 Gib|
|swap     |16 GiB|
|var      |60 GiB|
|home     |60 GiB|
|/        |63 GiB|
  
**Disk 2 - 200 GB**  

|Mounting Point|Size|
|:--|:--|
|/mysql/Data|50 GiB|
  
**Disk 3 - 50 GB**  

|Mounting Point|Size|
|:--|:--|
|/mysql/innodb/  (redo, undo, doublewrite)|200 GiB|

**Disk 4 - 50 GB**  

|Mounting Point|Size|
|:--|:--|
|/mysql/innodb/tmp|50 GiB|
  
**Disk 5 - 50 GB**  

|Mounting Point|Size|
|:--|:--|
|/mysql/audit|50 GiB|

**Disk 6 - 50 GB**  

|Mounting Point|Size|
|:--|:--|
|/mysql/log (binlog errorlog)|50 GiB|

**Disk 7 - 50 GB**  

|Mounting Point|Size|
|:--|:--|
|/mysql/dump/securefile (dump e securefile)|50 GiB|
|/mysql/scripts/|

### Registrar na conta Red Hat para utilizar os repositorios oficiais

```shellscript
subscription-manager register
```

### Anexar automaticamente a Developer Subscription

```shellscript
subscription-manager attach --auto
```

### Confirmar que a subscription está ativa

```shellscript
subscription-manager status
```

### Habilitar os repositórios corretos do RHEL 9

```shellscript
subscription-manager repos \
  --enable=rhel-9-for-x86_64-baseos-rpms \
  --enable=rhel-9-for-x86_64-appstream-rpms
```

### Confirmar se os repositorios foram referenciados corretamente

```shellscript
subscription-manager repos --list-enabled
```

### Limpar cache

```shellscript
dnf clean all
```

### Atualizar os pacotes

```shellscript
sudo dnf -y update
```

### Preparar o ambiente para o permissionamento de grupo para o usuário dbadmin 

Aplicar as permissões corretas, o 2 no 2770 garante que tudo criado dentro herda o grupo dbadmin, e o "setfacl" evita “permissão quebrada”

```shellscript
chmod 2770 /home/dbadmin
setfacl -m g:dbadmin:rwx /home/dbadmin
setfacl -d -m g:dbadmin:rwx /home/dbadmin
```

Criar o permissionamento do grupo para dbadmin

```shellscript
cat <<'EOF' | tee /etc/sudoers.d/dbadmin >/dev/null
%dbadmin ALL=(ALL) ALL
EOF
```

Ajustar as permissões minimas

```shellscript
chmod 440 /etc/sudoers.d/dbadmin
```

### Criação dos usuários administrativos

Criar o usuário

```shellscript
useradd -m <usuario>
usermod -aG dbadmin <usuario>
passwd <usuario>
passwd -e <usuario>
```

Padronização via .bashrc global, Criar editando o arquivo dbadmin.sh

```shellscript
cat > /etc/profile.d/dbadmin.sh <<'EOF'
if id -nG "$USER" | grep -qw "dbadmin"; then
  export PS1='[\u@DBA \W]\$ '
  alias ll='ls -lh'
fi
EOF
```

Aplicar as permissões corretas para o arquivo de profile

```shellscript
chmod 644 /etc/profile.d/dbadmin.sh
```





### Configurar o vim para exibir o tema desert como default para usuários logados com o root

```shellscript
cat > /root/.vimrc <<'EOF'
syntax on
colorscheme desert
EOF

chmod 600 /root/.vimrc
```

### Configurar o vim para exibir o tema desert como default para usuários DBAs logados pertencentes ao grupo dbadmin (logado com o usuário administrativo)

```shellscript
cat > /etc/vimrc.d/dbadmin.vim <<'EOF'
syntax on
colorscheme desert
EOF

chmod 644 /etc/vimrc.d/dbadmin.vim
```

### Criar o profile para aplicar padronização dos usuários pertencentes ao grupo dbadmin e a aplicação do tema desert para os respectivos

>[!IMPORTANT]
>Esse script roda no login bash e só afeta quem está no grupo dbadmin.

```shellscript
cat > /etc/profile.d/dbadmin.sh <<'EOF'
# Aplica somente para membros do grupo dbadmin
if id -nG "$USER" | grep -qw "dbadmin"; then
  export PS1='[\u@DBA \W]\$ '
  alias ll='ls -lh'
  export VIMINIT='source /etc/vimrc.d/dbadmin.vim'
fi
EOF

chmod 644 /etc/profile.d/dbadmin.sh
```

### Instalar os pacotes de aplicativos, ferramentas e utilitátios opcionais 

```shellscript
dnf -y install dnf-utils setroubleshoot setools policycoreutils-python-utils 
```

### Verifique se o sistema está registrado (obrigatório)

```shellscript
subscription-manager status
```

Deve aparecer:  
**Overall Status: Registered**  

### Habilitar o CodeReady Builder (CRB) para instalar o pacote aplicativo para o Epel (Extra Packages for Enterprise Linux)

```shellscript
subscription-manager repos --enable codeready-builder-for-rhel-9-x86_64-rpms
```

### Instalar o EPEL Release (URL oficial Fedora)

```shellscript
dnf install -y \
https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
```

### Atualizar o cache 

```shellscript
dnf clean all
dnf makecache
```

### Instalar o htop

```shellscript
dnf install -y htop
```



### Adicionando os discos para o MySQL

#### Após atachar o disco que terá o ponto de montagem unico em /mysql/data executar o seguinte comando 

```shellscript
lsblk
```

Ele deverá paracer algo com **sdb    8:16   0 200G  0 disk**


#### Criar o Physical Volume (PV)

```shellscript
pvcreate /dev/sdb
```

O que isso faz?  

- Inicializa o disco para uso pelo LVM
- Escreve metadados LVM no início do disco
- Não cria filesystem ainda

Verificar:

```shellscript
pvs
```

Saída esperada  
/dev/sdax......  
.  
.  
.  
**/dev/sdb   lvm2   200.00g**

#### Criar o Volume Group (VG)

```shellscript
vgcreate disk2-vg1 /dev/sdb
```

O que é um VG?

- É um pool de espaço
- Agrupa um ou mais discos (PVs)
- De onde saem os Logical Volumes

Verificar

```shellscript
vgs
```

Saida esperada  
disk1-vg1  
.  
.  
.  
**disk2-vg1   1   0   0 wz--n- <200.00g <200.00g**  

#### Criar o Logical Volume (LV)

```shellscript
lvcreate -n mysql_data -l 100%FREE disk2-vg1
```

O que isso faz?

- -n mysql_data → nome do LV
- -l 100%FREE → usa todo o espaço disponível do VG
- disk2-vg1 → VG de origem

Verificar 

```shellscript
lvs
```

Saída esperada  
swap  
.  
.  
.  
mysql_data  -wi-a----- <200.00g  

O caminho do dispositivo agora é: **/dev/disk2-vg1/mysql_data**


#### Criar o filesystem (XFS)

```shellscript
mkfs.xfs /dev/disk2-vg1/mysql_data
```

Por que XFS?

- Padrão do RHEL
- Excelente para I/O grande
- Recomendado para MySQL
- Suporta crescimento online

>[!IMPORTANT]
>⚠️ Esse comando apaga qualquer dado no LV (aqui está vazio, ok).

#### Criar o ponto de montagem

```shellscript
mkdir -p /mysql/data
```

#### Persistir no /etc/fstab (forma correta)

1. Obter o UUID

```shellscript
blkid /dev/disk2-vg1/mysql_data
```

Saída esperada   
/dev/disk2-vg1/mysql_data: **UUID="1a095ba2-2ec2-4ef8-9268-d7334f4a7875" TYPE="xfs"**

2. Editar /etc/fstab

```shellscript
vim /etc/fstab
```

Adicionar: 

UUID=1a095ba2-2ec2-4ef8-9268-d7334f4a7875 /mysql/data  xfs  defaults,noatime  0 0

Por que noatime?

- Evita escrita desnecessária
- Bom para banco de dados

#### Testar o fstab (OBRIGATÓRIO)


```shellscript
umount /mysql/data
mount -a
```

>[!NOTE]
>Se não der erro, está correto.





```shellscript
visudo
```
Localizar o seguinite trecho "**Allow root to run any commands anywhere**" e adicionar os  usuários administradores de acordo com o exemplo de  root ja disponível, salvar e sair do visudo"  
root         ALL=(ALL)       ALL  
<usuário1>   ALL=(ALL)       ALL  
<usuário12   ALL=(ALL)       ALL  

Criar o arquivo /etc/sudoers.d/admindba contendo a linha abaixo  
%admindba        ALL=(ALL)       ALL  

### Instalar os pacotes aplicativos para o Epel
sudo dnf -y install yum-utils epel-release 

### Instalar os pacotes de aplicativos opcionais, ferramentas e utilitátios opcionais 
sudo dnf install -y vim-enhanced wget bash-completion tcpdump setroubleshoot setools gcc.x86_64 net-tools tree htop lsof unzip bzip2 ncurses-compat-libs perl


#Configurar o schema de cores padrão do vim para o desert para o root (logado com o root)
echo -e "syntax on\ncolorscheme desert" >> /root/.vimrc


#Configurar o schema de cores padrão do vim para o desert para os usuários administrativo (logado com o usuário administrativo)
sudo echo -e "syntax on\ncolorscheme desert" >> /home/admindba/.vimrc



## Camila Vello Tamburim

##### Ela é muito Zé Roela

# Mas eu amo muito ela ♥